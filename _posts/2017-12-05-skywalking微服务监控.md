# Raft协议

------

## 说明
分布式存储系统通常通过维护多个副本来进行容错，提高系统的可用性。要实现这个目标，就必须要解决分布式存储系统的最核心问题: 维护多个副本的一致性。

一致性(consensus): 它是构建具有容错性(fault-tolerant)的分布式系统的基础。在一个具有一致性的集群里面，同一时刻所有的节点对存储在其中的某个值都有相同的结果，即对其共享的存储保持一致。集群具有自动恢复的性质，当少数节点失效的时候不影响集群的正常工作，当大多数集群中的节点失效的时候，集群则会停止服务(不会返回一个错误的结果)

一致性用来保证即使在部分副本宕机的情况下，系统仍然能正常对外提供服务。一致性协议通常基于replicated state machines，即所有节点都从同一个state出发，都经过同样的一些操作序列(log)，最后到达同样的state。


## 架构

![raft-01.png](https://aaron-13.github.io/images/raft-01.png)

系统中每个节点有三个组件:

+ 状态机: 当我们说一致性的时候，就是说要保证这个状态机的一致性。状态机会从log里面取出所有命令，然后执行一遍，得到的结果就是我们对外提供的保证了一致性的数据。

+ log: 保存了所有修改的记录

+ 一致性模块: 一致性模块算法就是用来保证写入log的命令的一致性，这也是raft算法核心的内容。


## 协议内容
Raft协议将一致性协议的核心内容分拆成为几个关键阶段，以简化流程，提高协议的可理解性。

### Leader election
Raft协议的每个副本都会处于三种状态之一: Leader, Follower,Candidate

+ Leader: 所有请求的矗立着，Leader副本接受client的更新请求，本地处理后再同步至多个副本。

+ Follower: 请求的被动更新者，从Leader接受更新请求，然后写入本地日志文件

+ Candidate: 如果Follower副本在一段时间内没有收到Leader副本的心跳，则判断leader可能已经故障，此时启动选主进程，此时副本会变成Candidate状态，直到选主结束


时间被分为很多连续的随机长度的term，term有唯一的id。每个term一开始就进行选主:

	+ Follower将自己维护的current_term_id加1
	+ 然后将自己的状态转成Candidate
	+ 发送RequestVoteRPC消息(带上current_term_id)给其他所有server

这个过程会有三个结果:
	+ 自己被选成主。当收到majority的投票后，状态切成leader，并且定期给其他的所有server发心跳信息(不带log的 AppendEntriesRPC)以告诉对方自己是current_term_id所表示的term的leader。每个term最多只有一个leader，term id 作为logical clock，在每个RPC消息中都会带上，用于检测过期的消息。当一个server收到的RPC消息中的rpc_term_id比本地的current_term_id更大时，就更新current_term_id为rpc_term_id，并且如果当前的state为Leader或candidate，将自己的状态改为Follower。如果rpc_term_id比本地current_term_id更小，则拒绝这个RPC消息。

	+ 别人成为了主。当Candidator在等待投票的过程中，收到了大于或者等于本地的current_term_id的声明对方是leader的AppendEntriesRPC时，则将自己的state切成follower，并且更新本地的current_term_id

	+ 没有选出主。当投票被瓜分，没有任何一个candidate收到majority的vote时，没有leader被选出。这种情况下，每个candidate等待的投票的过程就超时了，接着candidates都会将本地current_term_id再加1，放起RequestVoteRPC进行新一轮Leader election。


投票策略	
	+ 每个节点只会给每个term投一票，具体的是否同意和后续的Safety有关

	+ 当投票被瓜分后，所有的candidate同时超时，然后有可能进入新一轮的票数被瓜分，为了避免这种情况，Raft采用一种简单的方法: 每个candidate的election timeout从150ms-300ms之间取随机，那么第一个超时的candidate就可以发起新一轮的leader election，带着最大的term_id给其他所有server发送RequestVoteRPC消息，从而自己成为leader。


## Log Replication

当leader被选出来后，就可以接收客户端发来的请求了，每个请求包含一条需要被replicated state machines执行的命令。Leader会把它作为一个log entry append到日志中，然后给其他的server发AppendEntriesRPC请求。当Leader确定一个log entry被safely replicated了，就apply这条log entry到状态机中然后返回结果给客户端。如果某个Follower宕机了或者运行的很慢，或者网络丢包了，则会一直给这个Follower发AppendEntriesRPC直到日志一致。

当一条日志是commited时，Leader才可以将它应用到状态机中。Raft保证一条commited的log entry已经持久化了并且会被所有的节点执行。

当一个新的Leader被选出来时，它的日志和其他的Follower的日志可能不一样，这个时候，就需要一个机制来保证日志的一致性。一个新的Leader产生时，集群状态可能如下:

![raft-02.png](https://aaron-13.github.io/images/raft-02.png)

新leader产生后，就以leader上的log为准，其他的follower要么少数据，要么多数据，或者既少了又多了数据。

因此，需要有一种机制来让leader和follower对log达成一致，leader会为每个follower维护一个nextIndex，表示leader给各个follower发送的下一条log entry在log中的index，初始化为leader的最后一条log entry的下一个位置。leader给follower发送AppendEntriesRPC消息，带着(term_id,(nextIndex-1)),term_id(即nextIndex-1)这个槽位的log entry的term__id，follower接收到AppendEntriesRPC，会从自己的log中找是不是存在这样的log entry，如果不存在，就给leader恢复拒绝消息，然后leader则将nextIndex减1，再重复，直到AppendEntriesRPC消息被接收。

leader和b为例:
初始化，nextIndex为11，leader给b发送AppendEntriesRPC(6,10),b在自己log的10槽位中没有找到term_id为6的log entry，给leader回应一个拒绝消息。接着，leader将nextIndex减1，变为9，然后发送AppendEntriesRPC(6,9),以此循环，直到发送AppendEntriesRPC(4,4)，b接收到消息。随后，leader就可以从槽位5开始给b推送日志了


## Safety
+ 哪些follower有资格成为leader？
Raft保证被选为新leader的节点拥有所有已提交的log entry，这与ViewStampedReplication不同，后者不需要这个保证，而是通过其他机制从follower拉去自己没有的提交的日志记录

这个保证是在RequestVoteRPC阶段做的，candidate在发送RequestVoteRPC时，会带上自己的最后一条日志记录的term_id和index，其他节点收到消息时，如果发现自己的日志比RPC请求中携带的更新，拒绝投票。日志比较的原则是，如果本地的最后一条log entry的term id 更大，则更新，如果term id 一样大，则日志更多的更大(index)

+ 哪些日志记录被认为commited？
1. leader正在replicated当前term(即term 2)的日志记录给其它follower，一旦leader确认这条log entry被majority写盘了，这条log entry就被认为是commited。

2. leader正在replicate更早的term的log entry给其它follower。


**只允许主节点提交包含当前term的日志**


## Log Compaction
Raft采用对整个系统进行snapshot来处理，snapshot之前的日志都可以丢弃，Snapshot技术在Chubby和ZooKeeper系统中都有采用。

Raft使用的方案是: 每个副本独立的对自己的系统状态进行Snapshot，并且只能对已经提交的日志记录(已经应用到状态机)进行snapshot

Snapshot中包含以下内容:
+ 日志元数据，最后一条commited log entry的(log index, last_included_term)

+ 系统状态机: 存储系统当前状态

Snapshot的缺点就是不是增量的，即使内存中某个值没有变，下次做snapshot的时候同样会被dump到磁盘。当leader需要发给某个follower的log entry被丢弃了，leader会将snapshot发给落后太多的follower。当新加进一台机器时，也会发送snapshot给它，发送snapshot使用新的RPC。

做snapshot有一些需要注意的性能点: 1. 不要做太频繁，否则消耗磁盘的带宽  2.不要做的太不频繁，否则一旦节点重启需要回放大量日志，影响可用性。系统推荐当日志到达某个固定的大小做一次snapshot。 3. 做一次snapshot可能耗时过长，会影响正常log entry的replicate，这个可以通过使用copy-on-write的技术来避免snapshot过程影响正常log entry的replicate


## 集群拓扑变化
集群拓扑变化的意思是在运行过程中多副本集群的结构性变化，如增加/减少副本数，节点替换等。

Raft协议定义时也考虑了这种情况，从而避免由于下线老集群上线新集群而引起的系统不可用。Raft也是利用上面的Log Entry和一致性协议来实现这个功能。

假设老集群配置用Cold表示，新集群用Cnew表示，这个集群拓扑变化的流程如下:
1. 当集群成员配置改变时，leader收到人工发出的重配置命令从cold切成cnew

2. leader副本在本地生成一个新的Log entry，其内容是cold U cnew，代表当前时刻新旧拓扑配置共存，写入本地日志，同时将该log entry推送至其他Follower节点。

3. Follower副本收到log entry后更新本地日志，并且此时就以该配置作为自己了解的全局拓扑结构

4. 如果多数Follower确认了cold U cnew这条日志的时候，Leader就commit这条log entry

5. 接下来leader生成一条新的log entry，其内容是全新的配置cnew，同样将该log entry写入本地日志，同时推送到Follower上

6. Follower收到新的配置日志cnew后，将其写入日志，并且从此刻起，就以该新的配置作为系统拓扑，并且如果发现自己不在cnew这个配置中就会自动退出

7. leader收到多数Follower的确认消息后，给客户端发起命令执行成功的消息


**异常分析**

+ 如果Leader的cold U cnew尚未推送到Follower，Leader就挂了，此时选出新的Leader并不包含这条日志，此时新的Leader依然使用cold作为全局拓扑配置

+ 如果leader的cold U new 推送到大部分的Follower后就挂了，此时选出的新的leader可能是cold也可能是cnew的某个Follower

+ 如果leader在推送cnew配置的过程中挂了，那么和2一样，新选出来的leader可能是cold也可能是cnew中的某一个，那么此时客户端继续执行一次改变配置的命令即可

+ 如果大多数的Follower确认了cnew这个消息后，那么接下来即使leader挂了，新选出来的leader也肯定是位于cnew这个配置中的，因为有Raft的协议保证。


**不能直接从cold切换到cnew，因为如果直接切换，可能会产生多主**
