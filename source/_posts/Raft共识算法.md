---
title: Raft共识算法
date: 2020-03-21 16:45:12
tags:
---

### Raft共识算法

## 官网博客： [Raft Consensus Algorithm](https://raft.github.io/)

#### 节点的状态

- Leader

- Follower

- Candidate

- 在系统运行正常的时候只有Leader和Follower两种状态的节点。一个Leader节点，其他的节点都是Follower。

- Candidate是系统运行不稳定时期的中间状态，当一个Follower对Leader的的心跳出现异常，就会转变成Candidate，Candidate会去竞选新的Leader，它会向其他节点发送竞选投票，如果大多数节点都投票给它，它就会替代原来的Leader，变成新的Leader，原来的Leader会降级成Follower。

#### RPC

Raft协议在选举阶段交互的RPC有两类：RequestVote和AppendEntries。

- RequestVote是用来向其他节点发送竞选投票。
- AppendEntries是当该节点得到更多的选票后，成为Leader，向其他节点确认消息。

#### 选举流程

Raft采用心跳机制触发Leader选举。系统启动后，全部节点初始化为Follower，term为0.节点如果收到了RequestVote或者AppendEntries，就会保持自己的Follower身份。如果一段时间内没收到AppendEntries消息直到选举超时，说明在该节点的超时时间内还没发现Leader，Follower就会转换成Candidate，自己开始竞选Leader。一旦转化为Candidate，该节点立即开始下面几件事情：

- 1、增加自己的term。
- 2、启动一个新的定时器。
- 3、给自己投一票。
- 4、向所有其他节点发送RequestVote，并等待其他节点的回复。

如果在这过程中收到了其他节点发送的AppendEntries，就说明已经有Leader产生，自己就转换成Follower，选举结束。

如果在计时器超时前，节点收到多数节点的同意投票，就转换成Leader。同时向所有其他节点发送AppendEntries，告知自己成为了Leader。

每个节点在一个term内只能投一票，采取先到先得的策略，Candidate前面说到已经投给了自己，Follower会投给第一个收到RequestVote的节点。每个Follower有一个计时器，在计时器超时时仍然没有接受到来自Leader的心跳RPC, 则自己转换为Candidate, 开始请求投票，就是上面的的竞选Leader步骤。

如果多个Candidate发起投票，每个Candidate都没拿到多数的投票（Split Vote），那么就会等到计时器超时后重新成为Candidate，重复前面竞选Leader步骤。

Raft协议的定时器采取随机超时时间，这是选举Leader的关键。每个节点定时器的超时时间随机设置，随机选取配置时间的1倍到2倍之间。由于随机配置，所以各个Follower同时转成Candidate的时间一般不一样，在同一个term内，先转为Candidate的节点会先发起投票，从而获得多数票。多个节点同时转换为Candidate的可能性很小。即使几个Candidate同时发起投票，在该term内有几个节点获得一样高的票数，只是这个term无法选出Leader。由于各个节点定时器的超时时间随机生成，那么最先进入下一个term的节点，将更有机会成为Leader。连续多次发生在一个term内节点获得一样高票数在理论上几率很小，实际上可以认为完全不可能发生。一般1-2个term类，Leader就会被选出来。
