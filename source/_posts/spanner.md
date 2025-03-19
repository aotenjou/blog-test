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

![](https://pic1.imgdb.cn/item/67daad4088c538a9b5c10c7b.png)

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

![](https://pic1.imgdb.cn/item/67dab4d788c538a9b5c10e3e.png)

Spanner 的数据模型不是纯粹关系型的，**它的行必须有名称。**更准确地说，每个表都需要有**包含一个或多个主键列的排序集合**。

这种需求，让 Spanner 看起来仍然有点像键值存储: 主键形成了一个行的名称，每个表都定义了从主键列到非主键列的映射。

当一个行存在时，必须要求已经给行的一些键定义一些值(即使是 NULL)。采用这种结构是很有用的，因为这可以让**应用通过选择键来控制数据的局部性**。


#### 高层架构

![](https://pic1.imgdb.cn/item/67dabbb788c538a9b5c110fc.png)

考虑图中3个数据中心的A、B、C(存在于不同的地域中)，数据存储在分片中,包含了数据库的行和一些键值对，例如A的分片中有键A-M的。
现将该分片复制到B、C数据中心中，即使整个数据中心出现故障也可以继续。**位于不同数据中心的spanserver形成了Paxos Group**， 如Figure3中所示，也就是一个Paxos Group中的副本可以存在于不同数据中心中。三个键A-M的shard是通过日志复制使状态机去更新键值对。

###### 设计原因

- 多个分片是为了获得更好的并行性
- 每个分片有一个Paxos组用于复制，若a-c的距离远复制开销大，也只需要a、b之间获得majority即可
- 数据中心的容错，速率过慢
- 副本靠近客户端，客户可以获得高性能的只读事务

###### 挑战

1. 读取副本产生的最新的写入：追求更强的一致性，在ZooKeeper中实现的fast-read是弱的线性一致性。本地副本需要读到最新的写，在本文中将会用到。
2. 跨分片事务：支持跨分片的事务，例如转账事务中一个分片是一个账户、另一个分片是目标账户，当我们要执行转账操作时可以像事务一样执行，并且具有ACID的语义。
3. 事务是可序列化的：读取多条记录的事务必须是可序列化的。但是本地碎片可能反映已提交事务的不同子集。

#### True Time


| function    | return                               |
| ----------- | ------------------------------------ |
| TT.before() | true if t has definitely not arrived |
| TT.now()    | TTinterval: [earliest, latest]       |
| TT.after()  | true if t has definitely passed      |

spanner中，TrueTime API可以提供全球统一的时间。

- TT.now()可以获得一个绝对时间**TTinterval**，是一个区间。这个值和UnixTime是相同的，同时还能够得到一个误差e。
- TT.after(t)和TT.before(t)是基于TT.now()实现的。

TrueTime API实现靠的是**GPS和原子钟**。之所以要用两种技术来处理，是因为导致这两个技术的失败的原因是不同的。
GPS会有一个天线，电波干扰会导致其失灵。原子钟很稳定。当GPS失灵的时候，原子钟仍然能保证在相当长的时间内，不会出现偏差。
实际部署的时候。每个数据中心需要部署一些Master机器，其他机器上需要有一个slave进程来从Master同步。有的Master用GPS，有的Master用原子钟。



#### 并发控制

时间戳管理
Spanner使用TrueTime来控制并发，实现外部一致性。支持以下几种事务。 

- 读写事务：读写操作的集合。 
- 只读事务：只有读操作，并且提供的snapshot isolation的功能，保证强一致性。
- 快照读,只读事务由客户端提供时间戳：可以读取stale data。
- 快照读,客户端提供时间范围。

![](https://pic1.imgdb.cn/item/67dac17788c538a9b5c113a9.png)

上表是Spanner现在支持的事务。单独的写操作都被实现为读写事务；单独的非快照被实现为只读事务。
事务总有失败的时候，如果失败，对于这两种操作会自己重试，无需应用自己实现重试循环。

**时间戳的设计大大提高了只读事务的性能。**事务开始的时候，要声明这个事务里没有写操作，只读事务可不是一个简单的没有写操作的读写事务。**它会用一个系统时间戳去读**，所以对于同时的其他的写操作是没有Block的(只读事务是lock free的)。而且只读事务可以在任意一台已经更新过的replica上面读。
对于快照读操作，可以**读取以前的数据**，需要客户端指定一个时间戳或者一个时间范围。Spanner会找到一个已经充分更新好的replica上读取。
还有一个有趣的特性的是，对于只读事务，如果执行到一半，该replica出现了错误。客户端没有必要在本地缓存刚刚读过的时间，因为是根据时间戳读取的。只要再用刚刚的时间戳读取，就可以获得一样的结果。

#### 读写事务

###### 没有时间戳的读写事务

![](https://pic1.imgdb.cn/item/67daca0488c538a9b5c115c4.png)

A、B是两个不同的数据中心，客户端最开始的请求的读操作并不是事务操作，目的是为了找到Paxos Group的leader，并且Leader在锁表中分配该数据的锁，当client获得返回值时进行提交(commit)，该实现与Lab3中的读操作服务本质相同。
后续执行二段提交部分，提交完成后并释放锁。这种实现与分布式事务中实现的2PC+2PL的实现方法相同，只不过加入了Paxos的容错设计。

###### 分配时间戳给读写事务
事务的读写将会用到二段锁，当所有的锁都已经获得以后，在任何锁被释放之前(也就是持有所有锁期间)，就可以给事务分配时间戳。
对于一个给定的事务，Spanner会**为事务分配时间戳，**这个时间戳是Paxos分配给Paxos 操作的，它代表了事务提交的时间。

Spanner 依赖下面这些单调性:在每个Paxos组内，Spanner会以单调增加的顺序给每个Paxos写操作分配时间戳，即使在跨越多个领导者时也是如此。一个单个的领导者副本，可以很容易地以单调增加的方式分配时间戳。在多个领导者之间就会强制实现彼此隔离的不连贯:一个领导者必须只能分配属于它自己租约时间区间内的时间戳。要注意到，一旦一个时间戳S被分配，Smax就会被增加到s，从而保证彼此隔离性(不连贯性)。

- $s_i$:一个读写事务的时间戳，当Si被分配，Smax就会增长到S.
- $T_i$:表示一个事务
- $e_i$:代表一个事务的一些事件如开始或结束
- $e_i^{server}$:表示写事务Ti的Commit请求到达Coordinator的时间
- $tabs$:一个时间的绝对时间



**External consistency** -> 不变性(invariant)：如果事务T2开始发生在T1提交事务之前，T2的时间戳就必须大于T1的提交事务的时间戳-------tabs($e_1^{commit}$)<tabs($e_2^{start}$)=>S1 < S2

**Start**：为一个事务 Ti 担任协调者的领导者分配一个提交时间戳 $s_i$，不会小于TT.now().latest 的值，TT.now().latest的值是在$e_i^{server}$事件之后计算得到的。要注意，担任参与者的领导者， 在这里不起作用。第 4.2.1 节描述了这些担任参与者的领导者是如何参与下一条规则的实现的。

**Commit Wait**：担任协调者的领导者，必须确保客户端不能看到任何被Ti提交的数据，直到 TT.after($s_i$)为真。提交等待，就是要确保$s_i$会比$T_i$的绝对提交时间小。

Commit Wait 的证明如下图所示(只针对于R/W 事物)

- commit wait：T1的时间戳小于其commit提交开始的绝对时间
- assumption：T2的开始时间大于T1 commit的时间
- causality： T2开始的时间小于或等于其commit消息到coordinator的时间
- start：commit消息到达coordinator的时间小于等于T2的时间戳则事务开始
- transitivity：可以得到s1发生在s2之前

![](https://pic1.imgdb.cn/item/67dacc1388c538a9b5c11656.png)

###### 只读事务:高性能读

- 快速的读从本地的(邻近的)分片中
- 不需要二段锁
- 不需要二段提交
**正确性**
- Serializeble: 根据序列化每个操作都有一个先后顺序，所以我们将之前的三个事务进行一个排序：R/W1（T1） R/O  R/W2（T3）（T2） ,T3必须看见T1的写入
- External consistency(外部一致性)  Serializable+Real time，与线性一致性相似(都是强一致性)只不过外部一致性是事务级别的属性
示例：下一个事务必须看见上一个事务的写入 
1. 如果事务T1再另一个事务T2开始之前提交, 则T1提交的时间戳会小于T2提交的时间戳
2. 如果T1< T2  ，则T2 必须看见T1的写入

**Bad Plan** 
只读操作读取最新提交的值由于R/O操作是不加锁的，T3在只读操作时Ry会读取到T2提交的值，因此违背了事务的隔离性。

![](https://pic1.imgdb.cn/item/67dace1b88c538a9b5c116d4.png)










