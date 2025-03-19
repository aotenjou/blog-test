---
title: raft
date: 2025-03-14 11:06:11
tags:
- 论文
---
 <!-- more -->
## understandable共识算法：RAFT

##### 相比其他分布式共识算法，raft的最大特点是**易于理解**。
1. 问题分解：raft把共识算法分为三个子问题：
	1. 领导者选举（leader election）
	2. 日志复制（log replication）
	3. 安全性（safety）
2. 状态简化：对算法作出限制，减少状态数量和可能产生的变动。

## 1. 复制状态机
![](https://pic1.imgdb.cn/item/67d3cfa088c538a9b5bd2596.png)
**相同初始状态+相同输入=相同结束状态**。
- 多个节点上，从**相同的初始状态开始**，执行相同的命令会产生**相同的最终状态**。

raft中，leader将用户端请求（command）封装到一个个log entries中，再将他们复制到所有follower节点，然后大家按相同顺序应用log entity中的command，则大家结束状态肯定一致。

可以说，使用共识算法就是为了实现复制状态机。
分布式场景下各节点间，通过共识算法保证命令序列的一致，从而使得**状态一致**。
即便副本放入**不同的数据结构**，只要初始数据相同并发给相同的命令，同一时刻从两个副本中独到的结果也一样。（实现HTAP（OLTP+OLAP））。

## 2. 状态简化
![](https://pic1.imgdb.cn/item/67d3cfc088c538a9b5bd2619.png)
任何时刻，每各界点都是**leader/follower/candidate**状态之一。
只需要考虑**状态切换**，不需考虑**共存与互相影响**。
![](https://pic1.imgdb.cn/item/67d440a588c538a9b5bdb4c6.png)
raft把时间分割成任意长度的**任期（term）**，任期用连续的整数标记。

每个任期从一次选举开始。
某些情况，一次选举无法选出leader（两节点票数相同），以没有leader结束，新的任期（包含一次新的选举）会很快重新开始。Raft保证一个任期内，leader数不超过1。

#### 节点间使用**RPC**通信。
两种主要RPC：
- RequestVote RPC：请求投票，由candidate在选举期间发起。
- AppendEntries RPC：追加条目，由leader发起，用来**复制日志+提供一种心跳机制**。

通信时会**交换当前任期号**，如果一个服务器当前任期号比其他的小，会将自己任期号更新为较大的值。如果candidate/leader发现**任期号过期**，会立刻回到**follower状态**。如果节点接收到一个包含过期任期号的请求，会直接拒绝这个请求。

## 3.领导者选举
 
内部**心跳机制**：
- 如果存在leader，周期性地向所有follower发送心跳，来维持自己的地位。
- 如果follower一段时间没有收到心跳，就认为系统中没有可用leader，开始选举。

**选举过程**：
1. follower先**增加当前任期号**，并转换到**candidate**状态。
2. 再投票给自己，并且并行地向集群中其他服务器节点发送**投票请求（RequestVote RPC）**
3. 结果：
- **超过半数选票**赢得选举——>成为leader开始发送心跳。
- 其他节点赢得选举—>收到**新leader**心跳后，若新leader任期号**不小于**自己当前任期，则从candidate返回follower。
- 一段时间后没有任何获胜者（得票过于分散）—>每个candidate在自己投票选举**超时**后**增加任期号并开启新一轮投票**。
——>即：收到新leader心跳或收到超过半数选票就不会发起新选举。
- 随机选举超时时间约为**150-300ms**。

**请求投票RPC**


	//请求投票RPC Request
	type RequestVoteRequest struct{
		int term；//自己当前任期号
		int candidateID；//自己ID
		int lastLogIndex；//自己最后一个日志号
		int lastLogTerm；//自己最后一个日志的任期
	}
	//请求投票RPC Response
	type RequestVOteResponse struct{
		int term；//自己当前任期号
		bool voteGranted//自己会不会投票给这个candidate
	}

论文中称请求为argument，响应称result。

- 对于没有成为candidate的follower节点，同一个任期会按照**先来先得原则**投出选票。
- 最后一个日志任期：**安全性子问题**。


## 4. 日志复制
Leader被选出后，为客户端请求提供服务。
客户端连接leader：三种情况
1. 新leader直接服务。
2. 连接到follower：获取到leader的ID。
3. 宕机节点：寻找其他节点。
只要超过半数节点可用，client就能提供服务。

Leader收到客户端指令后，会把指令作为**新条目追加到日志中**。
- 一条日志中要有三个信息：
	- 状态机指令
	- leader任期号
	- 日志号（日志索引）
日志号+任期号 才能唯一确定一个日志。

leader并行发送**AppendEntries RPC**给follower,让他们复制该条目。当该条目被**超过半数**follower复制后，leader就可以在**本地执行该执行**并**把结果返回客户端**。

本地执行指令，即leader应用日志与状态机这一步称作**提交**。

复制过程中，leader/follower随时都有崩溃/缓慢的可能性。Raft在有宕机的情况下应该支持日志复制，并保证每个副本日志顺序一致（保证复制状态机实现）。具体有三种情况：
1. follower因为某些原因**没有给leader相应**，leader会**不断重发**追加条目请求**AppendEntries RPC**，哪怕leader已经回复客户端的情况。
2. 若有**follower崩溃后恢复**，此时raft追加条目的**一致性检查**生效，保证follwer能**按顺序恢复崩溃后缺失的日志**。
3. **leader宕机**，崩溃的leader可能复制了日志的**部分follower但还没有提交**，而选出的新leader又可能不具备这些日志，这样就**有部分follower中的日志和新leader的日志不相同**。这种情况下，leader通过**强制follower复制他的日志**来解决不一致的问题，意味着**follower中与leader冲突的日志条目会被新leader日志条目覆盖**（因为没有提交，所以不违背外部一致性）。

Raft**一致性检查**：
leader在每个发往follower的追加条目RPC中，会放入**前一个日志条目的索引位置和任期号**。若follower在它日志中找不到前一个日志，它就会拒绝此日志，leader收到follower的拒绝后，会**发送前一个日志条目**，从而**逐渐向前定位到follower第一个缺失的日志**。
		
可以优化来减少被拒绝的AppendEntries RPC个数。

通过以上机制，leader被选举之后**不需要任何特殊操作**就能使日志恢复到一致状态。
leader只需要正常操作，日志就能在**恢复AppendEntries一致性检查失败的时候自动趋于一致**。

leader**永远不会覆盖/删除自己的日志条目（AppendOnly），只能追加**。就能保证一致性特性：
- 只要过半服务器正常运行，raft就能接受、复制、应用新的日志条目。
- 正常情况下，新的日志条目可以在一个RPC来回中被复制给集群中的过半机器。
- 单个运行慢的follower不会影响整体性能。

**追加rpc**

	//追加日志RPC Request
	type AppendEntriesRequest struct{
		int term；//自己当前任期号
		int leaderID；//leader（自己的）ID
		int prevLogIndex；//前一个日志的日志号
		int prevLogTerm；//前一个日志的任期号
		byte[] entries；//当前日志体
		int leaderCommit；//leader已提交日志号
	}
	//追加日志RPC Response
	type AppendEntriesReponse struct{
		int term；//自己当前任期号
		bool success；//如果该follower包含前一个日志，则返回true
	}


如果leaderCommit>commitIndex,那么把commitIndex设为min（leaderCommit,Index of last new entry）

## 5. 安全性问题

**领导者选举** 和**日志复制（有序，无空洞）** 其实已经涵盖了共识算法的全程，但不能完全保证**每一个状态机会按*相同顺序*执行相同命令**。

所以补充规则保证算法在各类宕机问题下不出错：
1. Leader宕机处理：选举限制。
2. Leader宕机处理：新leader是否提交之前任期内的日志条目。
3. Follower和Candidate宕机处理。
4. 时间于可用性限制。

#### 1. leader宕机处理：选举限制

如果一个follower落后了leader若干条日志（**但没有漏一整个任期**），那么下次选举中按照领导者选举的规则，它依然可能当选leader,但当选leader后永远无法不上之前缺失的那部分日志，从而造成状态机之间不一致。

所以要对领导者选举增加一个限制，**保证选出来的leader一定包含了之前各任期的所有被提交的日志条目**

RequestVote RPC执行了这样的限制：**RPC中包含了candidate的日志信息，如果投票者自己的日志比candidate还*新*，它会拒绝掉该投票请求**。
raft通过比较两份日志中最后一条日志条目索引值和任期号来定义谁的日志比较新。
- 如果最后任期号不同：任期号大的日志“新”。
- 任期号相同：日志长的“新”。

#### 2. leader宕机处理：新leader是否提交之前任期内的日志条目

一旦当前任期内某个日志条目已经存储到**过半**的服务器节点上，leader就知道该日志条目可以被提交了。

follower的提交触发：下一个AppendEntries RPC：**心跳/新日志**。
单点提交<-->集群提交

如果某个leader在提交某个日志条目之前崩溃了，以后的leader会试图完成该日志条目的**复制**（而非提交），不能通过心跳提交老日志。
*raft永远不会通过计算副本数目的方式来提交之前任期内的日志条目。*
只有leader**当前任期内的日志条目**才通过计算副本数目的方式来提交。

一旦当前任期的某个日志条目一这种方式被提交，那么由于日志匹配特性，之前所有日志条目也都会被间接地提交。

#### 3. Follower和candidate宕机处理
- follower或candidate崩溃，那么后续发送给他们的RequestVote/AppendEntries RPC都会失效。
- Raft通过**无限的重试来处理这种失败**。如果崩溃的机器重启，那么这些RPC会成功完成。
- 如果**一个服务器 在完成了一个RPC但没有响应的时候崩溃了，它重启之后会再次受到同样请求（raft的RPC都是幂等的）**。

#### 4.时间与可用性限制
- raft算法整体不依赖客观时间，如果因为其他因素造成后发RPC先到，也不会影响raft的正确性。
- 只要整个系统满足下面时间要求，Raft就能选举出并维持一个稳定的leader：
- **广播时间（broadcastTime）<<选举超时时间（electionTimeOut）<<平均故障时间（MTBF）**
广播时间和平均故障时间是系统决定，但选举超时时间是我们自己选择。
Raft的RPC需要接受并将信息落盘，广播时间大约**0.5-20ms**，取决于存储技术。选举超时时间可能**10-500ms**。

## 6. 集群成员变更

需要**改变集群配置**的时候（增删节点/替换宕机机器/改变复制程度），raft可以**配置变更自动化**。

自动化配置变更机制最大的难点是**保证转换过程中不会出现同一任期的两个leader**，因为转换期间整个集群可能划分为**两个独立的大多数**。

所以配置采用了一种**两阶段**的方法。
集群先切换到一个过渡的配置，称为**联合一致（joint consensus）**。
1. 第一阶段，leader发起$C_{old,new}$，使整个集群进入**联合一致状态**。这时，**所有RPC都要在新旧两个配置中都达到大多数才算成功**。
2. 第二阶段，leader发起$C_{new}$，使整个集群进入新配置状态。这时，所有RPC只要在新配置下能达到大多数就算成功。

这时有三种可能：
![](https://pic1.imgdb.cn/item/67d3d12388c538a9b5bd2b71.png)
1. Leader在$C{old,new}$未提交时宕机。
2. Leader在$C{old,new}$已提交，$C_{new}$未发起时宕机。
3. Leader在$C_{new}$，已发起时宕机。
宗旨是避免**脑裂问题**

**leader在选出来时天然具有所有日志，避免复写，日志只能从leader流向follower**。