---
title: Zookeeper
date: 2020-03-21 16:46:13
tags:
---

# Zookeeper

## 简介

分布式系统协调工具，简单说 zookeeper = 文件系统 + 通知机制

## Znode类型

- 永久节点
  
  - 连接断开也存在

- 永久编号节点
  
  - zk自动编号

- 临时节点
  
  - 连接断开即删除

- 临时编号节点

## 三种节点类型

- leader
  
  - 负责进行投票的发起和决议，更新系统状态

- follower
  
  - 用于接收客户端请求并返回结果，在选举过程中参与投票

- observer
  
  - 可接受客户端请求，将写请求转发给leader节点，不参与投票过程，只同步leader状态，observer的目的是扩展系统，提高读取速度

## watcher

- 简介：zk允许客户端在指定节点上注册一些watcher，并且在特定事件触发的时候，zk服务端会将事件通知到感兴趣的客户端上去，该机制是zk实现分布式协调服务的重要特性

- 特性
  
  - client只会收到一次watchevent
  
  - getData，exists，getchildren可以设置监控
  
  - 注册：getData，exists，getChildren
  
  - 触发：create，delete，setData

- 实现原理
  
  - watcher时轻量级的，其实就是本地jvm的callback，服务端只是存了是否有设置watcher的布尔类型
  - 服务端dataTree发生变化后通过调用triggerWatch来触发watch回调, 通过服务端保存的ServerCnxn来进行event的process，最终将event通过序列化发送到客户端ClientCnxn，客户端，客户端根据类型、path来从watcherManager中获取响应的watcher，然后异步调用watcher的process来处理回调
  - 客户端的watcher注册是在客户端发送注册信息给server后收到响应后再注册到watcherManager的map中去的，然后客户端收到服务端的event后通过上述步骤回调watcher的process方法

## ACL

zk采用ACL（AccessControlLists）策略进行权限控制，类似与unix文件系统的权限控制，有如下5中权限

- create: 创建子节点的权限

- read：获取节点数据和子节点列表的权限

- write：更新节点数据的权限

- delete：删除子节点的权限

- admin：设置节点acl的权限

create和delete这两种权限都是针对子节点的权限控制

## zk的特点

- 顺序一致性
  
  - 从同一客户端发起的事务请求，最终会严格地按照顺序被应用到zk中去

- 原子性
  
  - 所有事务的处理结果在整个集群中所有机器上的应用情况是一致的，就是说集群中的所有机器要么都成功应用了某一个事务，要么都没有应用

- 单一系统镜像
  
  - 无论客户端连接到哪个服务器，看到的数据都是一致的

- 可靠性
  
  - 一旦请求被应用，更改的结果就被被持久化，直到被下一次更改覆盖

## 工作状态

- LOOKING：正在寻找leader

- LEADING：当领导中

- FOLLOWING：苦逼的下属

## 如何保证数据的一致性

- 重新选举leader之后的数据同步：选出leader后，leader需要和其他节点进行数据同步，完成数据同步的leader才能成为真正的leader，当超过一半的follower和observer同步完成后才算完成同步。（新leader选举完成后，leader会将自身最大的proposal的事务id，zxid发送给其他的follower节点，其他节点会根据leader的消息进行回退或数据同步操作最终目的要保证集群中所有节点的数据副本一致）

- 处理完事务请求的leader与follower保持同步：事务请求全部由leader处理，当leader收到请求后，将请求事务转化为事务proposal，由于leader会为每一个follower创建一个队列，将该事务放入响应队列，保证事务的顺序性。之后会由队列中顺序向其他节点广播该提案，follower收到后会将其以事务的形式写入到本地日志中，并向leader发送返回ack，leader等待其他follower的回复，当收到半数以上的响应时，leader会向其他节点发送commit消息，同时leader自身提交该提案，当follower将数据同步完成（事务被过半的follower提交）后，leader会将该follower加入到真正可用的follower列表中

## 数据同步流程

- leader等待server连接

- follower连接leader，将自身最大的zxid发送给leader

- leader根据收到的zxid确认同步点，然后发送同步信息给follower

- 完成同步后通知follower已经时uptodate状态

- 此时follower就可以重新接受客户端的请求进行服务了

## 写流程

- 客户端将请求放入outgoingquque，然后被发送至服务端

- follower收到请求后发送给leader，leader收到后加入等待队列等待被commit

- leader发送给各个follower

- follower收到leader的提案

- 写入事务日志，记录快照

- 发送ack给leader

- 统计ack数，然后发送commit给所有follower、通知observer

- follwer收到后开始commit

- 然后操作zk节点，最后返回给客户端

## ZAB

##### 总览：多个Follower。单个Leader，所有Proposal由leader发起，保证事务提交的顺序。

##### 事务提交：类似二阶提交，

1. Leader向多个Follower提交提案

2. Follower收到后如果可以执行则返回ACK

3. 收到超过半数ACK后则发送commit消息给follower，follwer执行提案，本次成功

##### Leader选举：

1. 每个参与者广播自己的选票，vote【zxid，sid，electionEpoch】

2. 收到其他参与者的选票，vote【zxid‘，sid’，electionEpoch'】
   
   1. 如果electionEpoch‘ < 自己的选举论数，则忽略其他参与者的选票
   
   2. 如果electionEpoch’ > electionEpoch，则更新electionEpoch进行下一轮选票
   
   3. 如果zxid' > zxid, 则更新sid为 sid'; 若zxid相等，如果sid‘ > sid，则sid=sid’，投出更新后的选票

3. 某个参与者发现自己获得了超过半数的选票，且经过短暂的等待后，若没有其他leader产生，则自己变为leader

4. 选举后follower根据自己记录的leader信息与leader建立连接

5. leader会接收follower的连接、计算新的epoch值，通知统一epoch值、数据同步（通知follower同步方式，follower根据结果进行相应操作）、启动zk对外服务

##### Leader同步

由于上述选举过程保证了leader必然拥有最大的zxid，leader只需要向follower同步自己的历史提案即可

1. DIFF差异化同步：此时follower的zxid与当前leader的zxid一致，不需要同步

2. TRUNC同步：此时follower的zxid大于leader记录的最大commitId，需要follower回滚,follower收到后后回滚snapLog，然后将snapLog加载到自己的DataTree

3. SNAP同步：全量同步，follower收到后从leader那获取全量DataTree保存

### 2PC

1. 发起者向所有参与者提交事务。参与者执行并记录Undo log，返回ACK
2. 发起者收到所有的ACK后，发送COMMIT, 参与者释放资源。  
   2.1 否则发送ABORT, 参与者undo 事务。

#### 缺点

1. 同步阻塞，收到事务后需要阻塞直到事务完成或取消
2. 单点，发起者单点
3. 脑裂，部分节点可能没收到部分请求导致数据不一致。

### 3PC

1. 先发送 canCommit 询问是否能执行事务。
2. 可以则发出 preCommit，参与者执行事务，记录Undo log, 返回ACK
3. 发起者收到所有ACK，发送COMMIT, 参与者释放资源。  
   3.1 否则发送ABORT，参与者undo 事务。

如果参与者收到preCommit 后，超时未收到COMMIT 或 ABORT, 则继续执行事务。

#### 优于2PC

1. 阻塞范围变小。
2. 在某些故障下，可以继续执行。
