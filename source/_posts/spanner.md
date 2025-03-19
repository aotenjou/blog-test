---
title: spanner
date: 2025-03-19 14:34:41
tags:
- 论文

---
<!-- more -->
## bigtable + google file system + mapreduce （4）
## Spanner 全球化分布式数据库

#### Spanner部署

Spanner的部署叫做*universe*，可以让Spanner全球性的管理数据，谷歌中使用了三套Spanner部署分别用于:测试、研发、上线 

![](https://pic1.imgdb.cn/item/67da8a8588c538a9b5c0ff66.png)

如上图Spanner server的结构图中，Spanner由多个**Zone(管理部署的单元、物理隔离的单元)**组成。一个数据中心中可以有一个或者多个Zone。
上图中的一个Zone有一个**zonemaster(分配数据给spanserver)**和100-1000个**spanserver(服务与客户端)**，用户使用**localtion proxies**定位每个Zone中分配数据服务的**spanservers**
- Universe master: 主要是一个控制台，它显示了关于 zone 的各种状态信息，可以用于相互之间的调试
- Placement driver: 会周期性地与 spanserver 进行交互，来发现那些需要被转移的数据，或者是为了满足新的副本约束条件，或者是为了进行负载均衡

#### Spanserver软件栈与副本部署

![](https://pic1.imgdb.cn/item/67da8aac88c538a9b5c0ff72.png)

###### 组成部分：

- Colossus:放置tablet的文件系统,GFS的升级版 
- tablet:类似于bigtable中的tablet，实现如下映射`(key:string, timestamp:int64)->string`  
- Paxos:共识算法与raft类似 
- replica:Paxos的上层状态机 
- locktable:实现并发控制,与分布式事务(分布式事务——悲观并发)的二阶段锁相同 
- Transaction manager：每个Paxos组中的支持分布式事务的软件

#### 结构分析

universe的结构中展示了多个Zone中都含有100-1000个Spanserver，Zone可以理解为一个区域(地域)的数据中心。
而每个SpanServer都是由一个Paxos状态机和Paxos协议组成(但是一个Paxos Group的Paxos副本不一定在一个Zone中),不同Zone之间的数据复制也是通过Paxos Group

作者：Ember
链接：https://zhuanlan.zhihu.com/p/597208058
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

Spanner于bigtable不同之处在于，Spanner分配Timestamp给数据作为版本号，是其更像是一个多版本数据库，而不是键值对的存储，tablet的状态存储在B-tree类似的文件与WAL中tablet的上层则是一个paxos的状态机方便复制(Paxos的一些细节我也没有研究过，复制过程可以参考RaftLAb2的实现。每次写操作都需要写入两次：
1. 写入tablet的log中 
2. 写入Paxos的日志中。

**写操作必须在领导者上初始化 Paxos 协议，读操作可以直接从底层的任何副本的 tablet 中访问状态信息**，只要这个副本足够新。
**副本的集合被称为一个 Paxos group**。

对于每个是领导者的副本而言，每个 spanserver 会实现一个**锁表**来实现并发控制。
对于那些**需要同步**的操作，比如事务型的读操作，需要获得锁表中的锁，而其他类型的操作则可以不理会锁表。

对于每个Paxos Group中都会有**Leader会开启Transaction manager**功能，每个拥有该功能的副本就称为一个Participant Leader，其余副本称为Participant slave。
如果只有一个Paxos组如果一个事务只包含一个 Paxos 组(对于许多事务而言都是如此)，它就可以绕过事务管理器，因为锁表和 Paxos 二者一起可以保证事务性。
如果一个事务包含了多于一个（multi） Paxos 组，那些组的领导者之间会彼此协调合作完成两阶段提交。其中**一个参与者组，会被选为协调者**，该组的 participant leader 被称为 coordinator leader，该组的 participant slaves 被称为 coordinator slaves。

#### directory

Spanner 将**具有公共前缀的键称为 directory**。
目前一个 directory 是数据存放的基本单位。**属于一个目录的所有数据，都具有相同的副本配置。**
 当数据在不同的 Paxos 组之间进行移动时，会一个目录一个目录地转移，如上图所示。
 Spanner 可能会**1.移动一个目录从而减轻一个 Paxos 组的负担，2.也可能会把那些被频繁地一起访问的目录都放置到同一个组中，3.或者会把一个目录转移到距离访问者更近的地方**。当客户端操作正在进行时，也可以进行目录的转移。
directory 是**数据复制和placement配置的基本单位**。spanner中c小单位也是 directory，同时提供方法 MoveDir 可以手动将一个 directory 移动到指定的zone。

#### 数据模型

spanner的行模型是`(key:string, timestamp:int64) -> row content`，
跟big table的模型最大的不同是这里**强化了row**的概念，不再突出column。
这样spanner的**timestamp是赋给整行数据的，是有物理意义的**，这使得spanner更像一个实现多版本并发的数据库，而在big table中，timestamp仅仅用于保存多个版本的key-value，跟并发完全无关；
这也是为什么spanner称自己为semi-relational 数据库，而big table只称自己是semi-structure 数据库的原因。

































