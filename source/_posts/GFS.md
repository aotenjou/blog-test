---
title: GFS
date: 2025-03-15 14:08:30
tags:
- 论文
---
 <!-- more -->


## bigtable + google file system + levelDB （2）

## Google file system：最著名的分布式文件系统。

##### 普通文件系统接口
基本接口：
**创建create，删除delete，打开open，关闭close，读取read，写入write。**
拓展接口：
**生成快照snapshot,修改update,追加append**

修改<=>删除+写入？

GFS**支持追加不支持修改**：追加操作更容易保持一致性。

##### 文件系统演变过程

**单机文件系统——A——>分布式文件系统——B——>GFS。**

A：1.文件如何分散存储？自动扩容缩容？2.如何知道文件存储在哪台机器？3.如何保证服务器故障时文件不损坏不丢失？4.多副本一致性？

B：1.大文件（GB）存储？2.多机器自动监控容灾与恢复？3.快速顺序读/追加写？


### 1.GFS整体架构

- 三种节点：**GFS client,GFS master,GFS chunkserver**.

1. client：维持**专用接口**，与应用交互。
2. master：维持**元数据**，统一管理chunk**位置&租约**。
3. chunkserver：**存储数据**。
![](https://pic1.imgdb.cn/item/67d523bc88c538a9b5be7393.png)

### 2.存储设计：A1——分割存储,B1——更大chunk+配套一致性策略。

GFS不选择以文件为单位进行存储，而是分为**chunk（64MB）**。
- GFS需要存储文件都偏大（几GB），大chunk有效减少系统内部寻址和交互次数。
- 大chunk意味client在**一个chunk可能多次操作**：复用TCP连接，节省网络开销。
- 大chunk意味**减小chunk数量**，从而节省元数据存储开销，相当于**节省系统内最珍贵的内存资源**。

但更大chunk也意味着可能**多线程——>单chunk：热点问题**。GFS在一致性设计做出性能妥协。

**一个文件：三个副本在三个chunkserver**，调度保证server间负载均衡。

### 3.master设计：A1——master单点增减，调整chunk元数据,A2——根据master中文件->chunk->chunk位置的映射。

文件位置等信息——>**元数据metadata**

两种元数据节点设计：

|        | 单中心节点             | 分布式中心节点              |
| ------ | ----------------- | -------------------- |
| 优点     | 实现难度低，容易保持一致性     | 实现难度高，一致性难以保证，可靠性难验证 |
| 缺点     | 单点可能成为系统瓶颈        | 无瓶颈，可扩展性强            |
| 方案工作重心 | 缩减元数据，减少单master压力 | 设计分布式元数据管理功能，并验证可靠性  |

GFS使用**单中心节点**。

存储三类元数据：
1. 所有文件/chunk的namespace【持久化】
2. 文件到chunk的映射【持久化】
3. 每个chunk位置【不持久化】：master重启时，可以从各个chunkserver处收集chunk位置信息。

所以GFS读取文件过程：**文件名->文件对应所有chunk名->所有chunk位置->对应chunkserver读取chunk。**

![](https://pic1.imgdb.cn/item/67d52a0188c538a9b5be833b.png)

优化：master不成为系统瓶颈，减轻压力：
1. GFS**数据流**不经过master,直接client和chunkserver交互。（**控制流和数据流分离，控制流才经过master**）
2. GFS的client缓存master元数据，大多数情况不需要访问master。
3. 避免master**内存**成为瓶颈：节省内存：增大chunk大小来节省chunk数量，对元数据定制化压缩...

可以把所有元数据放在master内存。
64MB的chunk,元数据小于64bit。


### 4.高可用设计：A3——masterWAL和主备、chunk多副本,B2——主备切换：chubby,租约，副本位置、数量：master
高可用问题通用做法是**共识算法**：raft/paxos.

GFS借鉴**主备**思想，为元数据和文件数据单独设计高可用方案。
Google文件量大->GFS机器总数很多->个别机器宕机频繁->节点宕机等小问题，GFS自动解决：**自动切换主备（B2）**。
**数据/服务高可用 Logic**与**物理节点宕机 Physic**的区别。

##### （1）master高可用->metadata高可用
master三类元数据中，**namespace**和**文件与chunk**的对应关系，只在master中存在，必须持久化+高可用。

GFS正在使用的master称为**primary master**，此外维持一个**shadow master**作为备份。
master运行时，对元数据所有修改操作，都要先**记录日志（WAL，write ahead log）**，再真正修改内存中元数据。primary实时向shadow同步WAL。
只有同步完成，元数据修改操作才算成功。**以日志代替数据块**。

**生成新增元数据日志并写入本地磁盘——>WAL传递shadow master——>反馈后正式修改primary master内存。**

实现自动切换：
- master宕机时通过**chubby（见bigtable）**识别并切换shadow，**秒级**。（与mysql主备机制很像）

##### （2）chunk高可用

**文件拆为chunk——>每个chunk三个副本。**
文件数据高可用**以chunk为维度来保持**。

GFS唯一中心节点master可以发挥作用，维持chunk副本信息。

GFS保证chunk高可用的思路：
**每个chunk的三个副本都写入完成才视为写入完成——>一个chunk所有副本都有完整数据——>一个chunkserve宕机，仍然有另外两个副本**
**——>如果这个宕机副本一段时间没有回复，master会在另一个chunkserver重建一个副本，始终把chunk副本维持在3个。**

GFS维护每个chunk**校验和**，读取时进行数据校验。如果校验和不匹配，chunkserver会反馈给master,选择其他副本读取并重建chunk副本。
为了减少对master压力，GFS采用一种**租约（lease）**机制，文件读写权限下放给某个chunk副本。授权后称该副本为**primary**，租约生效一段时间，对这个chunk的写操作有该副本负责。
**租约大约60s，主备之决定控制流走向，不影响数据流**。

副本放置：
- 副本复制：某副本所在server宕机后，chunk副本数小于预期（3），新增一个chunk副本。
- 负载均衡：某server负载过高，将副本放在另外server,**append new,delete old**。

**总之，选择策略：**
1. 新副本server利用率低
2. 新副本所在server最近创建副本不多：防止某server瞬间增加大量副本，**成为热点**。
3. chunk其他副本不在**同一机架**。保证机架/机房级别高可用。

### 4.GFS读写：B3——三写一读，流水线+控制流数据流分离+append+就近读取。

读写流程<->一致性机制v

需求：
**读：快速** 可以读落后版本，但不能错。
**写：** 改写overwrite，追加append
**改：正确** 不在意性能。（在意改为append）
**追加：快速** 可以允许一定异常，但追加数据不能丢失。

#### 写入

- 三个副本都完成后返回结果
1. 流水线技术
2. 数据流与控制流分离技术

![](https://pic1.imgdb.cn/item/67d538d588c538a9b5beada3.png)

就近传递。primary replica唯一确定写入顺序，保持副本一致性。
如果一个写入涉及多个chunk,client把他们分为多个写入。
**三写一读**

#### 改写和追加
改写：分布式操作，保证一致性代价很大。
推荐**追加**。

#### 读取
**clinet读取,查看缓存中有没有元数据信息——>（if n）请求master获取metadata并缓存——>计算offset对应chunk**
**——>client向最近server发送读请求，若server没有所需chunk,说明缓存失效，请求master获取最新元数据。**
**——>读取时获取chunk校验和**

### 5.GFS一致性模型：A4——四种方式保证一致性。

1. 一个chunk所有副本写入顺序一致：数据流控制流分离，控制流由primary发出，副本写入由primary到secondary.
2. chunk版本号检测chunk副本是否出现过宕机。失效副本不再写入，master不再记录该副本歇息（client刷新缓存时同步），GC自动回收。
3. master定期检查chunk副本的校验和。
4. 更多的追加，更少的改写。



多个client,写入并发，副本可能不一致。
一致性大体三个层次：**inconstant,consistent,defined**。
- consistent：一致的。无论从哪个副本读取，结果都一样。
- defined：已定义的。文件发生修改操作后，读取一致，client可以看到最新修改内容（consistent+与用户最新写入一致）。

**串行改写成功：defined**。
**写入失败：inconsistent**。重试一定次数使无法写入。
**并发改写成功：consistent 不同undefined**。对单个改写，成功意味着副本间一致。
但并发改写可能涉及多个chunk,改写执行顺序不一定相同，有可能造成应用读取不到预期。
**追加写成功：defined interspersed with inconsistent**。已定义，可能存在副本间不一致（追加重复执行导致）。

追加一致性的额外限制：
1. 单次append不超过64MB（1chunk）
2. 文件最后一个chunk不足以提供此次追加所需空间，则用**padding**填满，新增chunk。

从而限制append都在最后一个chunk,保证追加**原子性**。

**重复追加很好解决**。校验，长度。。。

![](https://pic1.imgdb.cn/item/67d540bf88c538a9b5bec2ea.png)

### 其他
**快照snapshot机制：**
COW（Copy On Write，写时复制）
**回收租约，停止chunk写入——>拷贝元数据生成快照文件——>增加chunk引用计数——>master正常授权租约，允许chunk写入**

**GC机制：**
客户端直接删除的文件，丢失修改操作而失效的副本，checksum校验失败而失效的副本。










