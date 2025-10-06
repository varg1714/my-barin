---
source: https://mp.weixin.qq.com/s/R2XXYFoR67VsiGwT6XM96A
create: 2024-08-09 13:48
read: false
---

![](https://mmbiz.qpic.cn/mmbiz_gif/AAQtmjCc74DZeqm2Rc4qc7ocVLZVd8FOASKicbMfKsaziasqIDXGPt8yR8anxPO3NCF4a4DkYCACam4oNAOBmSbA/640?wx_fmt=gif)

**目录**

一、前言

二、核心概念

    1. 日志复制状态机

    2. Leader、Follower、Candidate

三、Why Elixir

四、选主实现

    1. 任期（Term）

    2. 选举计时（Election Timer）

    3. 消息类型

    4. 状态机框架

    5. 选举计时

    6. 拉票

    7. 处理拉票消息  

    8. 计票

五、Leader 的工作

六、日志复制

    1. 从客户端行为开始

    2. 数据结构

    3. 日志仓库

    4. 日志复制分步实现

    5. 日志复制安全性讨论

    6. Rethink 集群选主

    7. Leader 提交过往任期的日志讨论

七、用户状态机  

    1. 什么是用户状态机？

    2. 定义状态机行为

    3. 应用状态机与 Raft 日志的交互

八、编写一个分布式 KVDB

    1. 状态机实现

    2. 完整的 Raft-base KVDB

    3. 验证

九、总结

**一**

**前言**

Raft 算法是一种分布式一致性算法，由 Diego Ongaro 和 John Ousterhout 在 2013 年提出。它主要用于分布式系统中，保证系统中的数据在多个节点间保持一致性。

Raft 算法被广泛应用于众多分布式系统中，尤其是在需要强一致性保证的场景中，例如：

*   **分布式存储系统**：如 ETCD、Consul 等键值存储系统，它们利用 Raft 算法来保证数据的强一致性和高可用性。
    
*   **分布式数据库**：一些分布式数据库管理系统（DBMS），如 CockroachDB 等。
    
*   **分布式锁服务**：例如 Google 的 Chubby 以及微软的 Azure Service Bus 等。

得物的多个内部中间件也是使用 Raft 算法作为多分片一致性的保证。

长期以来，大部分开发者都是将 Raft 作为一个黑盒使用，只知道它能保证多分片的一致性，对其运行原理也停留在纸面。当面临 Raft 性能调优或者奇怪的 Raft 问题排障的时候则束手无策。

费曼说过：“**What I cannot create, I do not understand**。” 我们中国先贤也强调 “**知行合一，以致良知**”。如果我们不能亲手编写一次 Raft 算法，对这个东西就不能算作理解。

**二**

**核心概念**

在着手开始写之前，我们先介绍几个 Raft 算法中的核心概念。

**日志复制状态机**

如果说什么是分布式系统理论的基石，那一定 “日志”。此处的日志与我们平时在应用中打印的给人阅读的信息不同，它是**最简单的存储抽象，是按时间排序的 append-only 的、有序的记录序列**。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7YSnG2emnibFasWLx2CnuLE4ztpLiajIvyoCW2qdXHcGYBMNicDhy3Oe7A/640?wx_fmt=png&from=appmsg)

“日志” 揭示了系统当下正在发生的事实。

关于日志的更深入理解，可以参考博客 _The Log: What every software engineer should know about real-time data's unifying abstraction_，非常全面深刻。

当进入分布式的领域，日志就需要 “状态机” 的辅助：

*   如果两个相同的确定性过程以相同的状态开始并以相同的顺序获得相同的输入，它们将产生相同的输出并以相同的状态结束。
    
*   确定性意味着处理不依赖于时间，并且不会让任何其他输入影响其结果。例如，一个程序的输出受到线程特定执行顺序或调用 gettimeofday 或其他一些不可重复的东西的影响，通常最好被认为是非确定性的。
    
*   进程的状态是在处理结束时保留在机器上的任何数据，无论是在内存中还是在磁盘上。

关于以相同的顺序获得相同的输入的一点应该会引起人们的注意——这就是日志的用武之地。这是一个非常直观的概念：**如果你将两段确定性代码提供给相同的输入日志，它们将产生相同的输出**。

所以 Raft 最重要的任务就是解决如何在分布式系统中使多个副本的日志数据达成一致的问题。

**Leader、Follower、Candidate**

为了实现上述目标，Raft 使用**强领导**模型，即要求集群中的一个副本充当 Leader，其他副本充当 Follower。

Leader 负责根据客户端请求采取行动生成命令，将命令复制给 Follower，并将响应返回给客户端。

多个副本根据选举算法推选合法的 Leader。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7sicLT7p52ftQoMpDibkNjyanOkBfCMxLAULrSE9YaLouK9CUFGhS2D2A/640?wx_fmt=png&from=appmsg)

这个方案解决了分布式系统中的以下问题：

*   **容错性（Fault Tolerance）**：分布式系统中可能会出现节点故障，包括宕机、网络分区等问题。通过这些角色的分工和协作，系统可以在一定数量的节点出现故障时仍然正常运行。
    
*   **领导者选举（Leader Election）**：在分布式系统中，通常需要一个节点（Leader）来协调其他节点（Followers）的工作，以确保一致性和效率。Leader 负责管理日志复制、决策提议等任务。当 Leader 失效时，系统需要能够选举出新的 Leader。
    
*   **一致性（Consensus）**：为了保证系统状态的一致性，需要所有节点就某个值或状态达成共识。这通常通过一系列的投票和通信过程来实现，其中 Candidate 角色在选举过程中起到关键作用。
    
*   **去中心化（Decentralization）**：在去中心化的系统中，没有中央权威来决定系统状态。这些角色的引入有助于在没有中心节点的情况下实现一致性和决策。

这种模型有其优点和缺点：

*   显著的优点是简单。数据总是从领导者流向 Leader，只有 Leader 响应客户端请求 (工程实践中有例外)。这使得 Raft 集群更容易分析、测试和调试。
    
*   缺点是性能——因为集群中只有一台服务器与客户端通信，这在客户端活动激增的情况下可能成为瓶颈。当然这个可以通过一些工程手段在一致性和吞吐性能中做出平衡，例如我们可以放弃强一致的要求，从 Follower 中读取数据，减轻 Leader 的负担，关于一致性的部分，可以参考文章《共识、线性一致性、顺序一致性、最终一致性、强一致性概念区分》。

**三**

**Why Elixir**

本系列中介绍的 Raft 实现是用 Elixir 编写的。在笔者看来，Elixir 具有三个优势，使其成为本系列和网络服务的有前途的实现语言：

*   Raft 框架需要大量的网络编程，而 Elixir 基于 Erlang 在网络编程方面体验一骑绝尘，甚至自带 RPC 实现；
    
*   Elixir 自带一个功能强大的 Shell，开发调试的难度大大降低；
    
*   Raft 这类算法需要大量的并发编程，而并发编程在 Elixir 中就像呼吸一样简单；
    
*   Erlang 语言的 OTP 框架大大降低的服务器编程的心智负担，常见的服务端编程场景都有合适的解决方案。

这个项目的开发中，笔者借鉴了这个生产级别的开源 Raft 库——龙舟的实现细节，如果对生产级的 Raft 实现感兴趣，可以阅读龙舟的代码。

我们的实现大致代码结构如下：

```
.                                                                                                                                                                                                                                                    
├── README.md                                                                                                                                                                                                                                        
├── Taskfile.yaml                                                                                                                                                                                                                                    
├── lib                                                                                                                                                                                                                                              
│   └── ex_raft                                                                                                                                                                                                                                      
│       ├── config.ex                                                                                                                                                                                                                                
│       ├── core               # 各个状态下的处理逻辑                                                                                                                                                                                                                      
│       │   ├── candidate.ex                                                                                                                                                                                                                         
│       │   ├── common.ex                                                                                                                                                                                                                            
│       │   ├── follower.ex                                                                                                                                                                                                                          
│       │   ├── free.ex                                                                                                                                                                                                                              
│       │   ├── leader.ex                                                                                                                                                                                                                            
│       │   └── prevote.ex                                                                                                                                                                                                                           
│       ├── debug.ex                                                                                                                                                                                                                                 
│       ├── exception.ex                                                                                                                                                                                                                             
│       ├── guards.ex                                                                                                                                                                                                                                
│       ├── log_store        # 日志存储，本节暂时不用                                                                                                                                                                                                                 
│       │   ├── cub.ex                                                                                                                                                                                                                               
│       │   └── inmem.ex                                                                                                                                                                                                                             
│       ├── log_store.ex                                                                                                                                                                                                                             
│       ├── message_handlers      # 各个状态下的消息处理                                                                                                                                                                                                                 
│       │   ├── candidate.ex                                                                                                                                                                                                                         
│       │   ├── follower.ex                                                                                                                                                                                                                          
│       │   ├── leader.ex                                                                                                                                                                                                                            
│       │   └── prevote.ex                                                                                                                                                                                                                           
│       ├── mock              # 日志复制状态机，本节暂时不用                                                                                                                                                                                                                       
│       │   └── statemachine.ex                                                                                                                                                                                                                      
│       ├── models              # 几个分片常用的数据结构                                                                                                                                                                                                                     
│       │   ├── replica.ex                                                                                                                                                                                                                           
│       │   └── replica_state.ex                                                                                                                                                                                                                     
│       ├── pb                      # 消息的数据结构                                                                                                                                                                                                                 
│       │   ├── ex_raft.pb.ex                                                                                                                                                                                                                        
│       │   ├── ex_raft.proto                                                                                                                                                                                                                        
│       │   └── gen.sh                                                                                                                                                                                                                               
│       ├── remote                 # 节点间通信客户端                                                                                                                                                                                                                  
│       │   ├── client.ex                                                                                                                                                                                                                            
│       │   └── erlang.ex                                                                                                                                                                                                                            
│       ├── remote.ex                                                                                                                                                                                                                                
│       ├── replica.ex            # 分片进程                                                                                                                                                                                                                    
│       ├── serialize.ex                                                                                                                                                                                                                             
│       ├── server.ex             # 节点间以及和客户端通信的服务端                                                                                                                                                                                                                   
│       ├── statemachine.ex                                                                                                                                                                                                                          
│       ├── typespecs.ex                                                                                                                                                                                                                             
│       └── utils                     # 工具集                                                                                                                                                                                                               
│           ├── buffer.ex                                                                                                                                                                                                                            
│           └── uvaint.ex                                                                                                                                                                                                                            
├── mix.exs                                                                                                                                                                                                                                          
├── mix.lock                                                                                                                                                                                                                                         
├── test                                                                                                                                                                                                                                             
│   ├── ex_raft_test.exs                                                                                                                                                                                                                             
│   ├── log_store.exs                                                                                                                                                                                                                                
│   └── test_helper.exs                                                                                                                                                                                                                              
└── tmp
```

**四**

**选主实现**

这一节中，我们暂时不考虑日志复制的细节，专注实现选主逻辑。

从论文 _In Search of an Understandable Consensus Algorithm_（《寻找一种易于理解的一致性算法（扩展版）》，以下称为 “论文”）中，可以得知：“选主” 这个过程本身也是个状态机（注意区分此处的状态机和日志复制状态机不是一回事）：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7TVczOqtLl7C2RQ8KAEZicu2tu2b67mvnrygF89TQhYtGUU6iakLya1uA/640?wx_fmt=png&from=appmsg)

在典型的稳态场景中，集群中的一台服务器是 Leader，而所有其他服务器都是 Follower。

虽然我们希望事情永远这样保持下去，但 Raft 的目标是容错，所以我们会讨论一些故障场景：例如其中一些服务器 crash、网络分区等。

为了实现选主的逻辑，我们要引入一些概念：

**任期（Term）**

论文中的解释：任期（Term）是分布式系统中共识算法中的一个关键概念，尤其是在领导者选举（Leader Election）和共识达成过程中。任期是时间上的一个划分，用于组织和同步分布式系统中的事件，确保系统中的所有节点能够在一个明确的时间段内就某个领导者或一系列决策达成一致。

简单来说，任期标明了 Leader 的服役阶段。

任期这个概念有以下特性：

*   **任期越大，话语权越大**：本地任期较小的总是服从任期更高的消息来源。
    
*   **每次选举，任期都会 + 1**：确保不会多个选举过程计票混乱。
    
*   **一山不容二虎**：同个任期内最多只有一个 Leader 被选出。

**选举计时（Election Timer）**

每个 Follower 都会在本地维持一个选举计时器，在选举计时器到期前，如果没有收到 Leader 的日志或者心跳，Follower 就会认为 Leader 出了意外，那么就会开始发起选举。这里有几个常见问题：

**Q：会不会有多个 Follower 同时成为候选人？**

A：会的，为了避免这种情况出现，我们给选举计时器添加一些随机区间，这是 Raft 简单的原因之一。Raft 使用这种随机化来降低多个 Follower 同时进行选举的机会。但是即使他们同时成为候选人，任何给定任期内也只有一个人会被选为领导人。在极少数情况下，如果选票分裂，没有候选人能获胜，将进行新的选举。

**Q：如果 Follower 与集群断开连接（分区）怎么办？**

A：这就是网络分区的隐蔽性，因为 Follower 无法区分谁被分区了。这种情况 Follower 会开始选举。但这种情况下 Follower 不会得到任何选票。这个节点可能会在候选状态中持续自旋（每隔一段时间重新启动一个新的选举），直到重新连接到集群。

**消息类型**

Raft 中两个节点的通信类型多种多样，不过涉及到选主的部分较为简单，只有下图添加注释这 4 种消息类型：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7zs9D30YCDDE8r16IFml5A5A2YHkJM4IicPrhXob7sE1kJSejmWyTOzQ/640?wx_fmt=png&from=appmsg)

**状态机框架**

从上述介绍中可以看出，选主这个过程涉及多个状态的轮转，每个状态下对不同消息做不同的反应，所以非常自然我们选择状态机的方式实现。

而 Erlang OTP 有个自带的状态机实现框架：gen_statem，有了这个利器，可以写出很清晰的状态机逻辑：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7C2YDlbr1m271vtNGO9KUQHdH2AeYdmseTlZc22dxpicBxXuaDuBia7Xg/640?wx_fmt=png&from=appmsg)

**选举计时**

我们这里的计时方式也是借鉴了 “龙舟” 的风格，即维护一个单位时间很小（1s）的本地时钟，其他超时事件全部用该时钟的整数倍来计算，这种方式配置和编码起来较为简单。

我们在代码中需要判断 localTick 的次数超出了 electionTimeout，即可将自身状态转变为 Candidate 开启选主。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7aiapoAlXkpVaN04t1uGK9GdiaqaugUcUNkR2DVX1RKEBVApUK6ZNeNoA/640?wx_fmt=png&from=appmsg)

**拉票**

开启投票前 Candidate 需要做这几件事：

*   将自身的 Term+1；
    
*   投自己一票。

完成这些后，就将携带新 Term 的消息发给所有其他的节点。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7qXJRQm9P6FjiaeMEjgTKDqspKZCJzUR1057EHnmT7whQrajyoO2Fxlg/640?wx_fmt=png&from=appmsg)

需要注意的是在这个 Raft 实现中，消息发送全部采用单向流，消息的发送和处理是隔离开的。

具体实现中则是使用了 Erlang 自带的 RPC，简化了自己开发通信协议的成本。

**处理拉票消息**

由于 Raft 中任期概念的设计，其他角色在接收到拉票消息时需要考虑以下几种情况：

*   **request_vote 消息携带的 Term 大于本地 Term**

这种情况较为简单，做 Follower 即可。

需要注意的是，在遇到较大 Term 而转换自身状态的情况下，不用重置选举计时器，原因在注释中写明了：

```
Not to reset the electionTick value to avoid the risk of having the local node not being to campaign at all. if the local node generates the tick much slower than other nodes (e.g. bad config, hardware clock issue, bad scheduling, overloaded etc), it may lose the chance to ever start a campaign unless we keep its electionTick value here.
```

简而言之就是防止某个节点一直处于 Follower 角色，失去选举的机会。

代码中 cast_pipein 这个函数是指在变更状态为 Follower 后，继续处理这个 request_vote 消息。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7No7KetTKJ479zueXvcF6drtSbLh7nicxILQG12pvWIKLFA9ib5L9We4Q/640?wx_fmt=png&from=appmsg)

*   **request_vote 消息携带的 Term 小于本地 Term**

这种情况，直接丢掉消息就行，任期小的消息在 Raft 算法中是不用处理掉的。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7KYBh0ELiawMzQ5xZI4ZmicicqYRc5C942Rm9Qphk1u8wuIJApne7JjXbw/640?wx_fmt=png&from=appmsg)

*   **request_vote 消息携带的 Term 等于本地 Term**

这种情况就是在情况 1 转变本地 Term 的后续，当任期一致时，投不投票也有讲究。

Raft 算法中规定，如下情况可以投赞成票：

*   这个任期中当前节点谁都没投过，这个消息来源第一个来拉票的，那就投给消息来源节点；
    
*   这个任期中当前节点投过了，而且我投的就是同个节点，那就不介意再投一次。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7rLdU7IL5wLHgA9C7a2C6ye7w1233wgg9Jia0oklMiaQoqGvV1bBbpWLw/640?wx_fmt=png&from=appmsg)

**计票**

Candidate 在散出 request_vote 的消息后，会开始等待其他节点的回执，这时候有 3 种情况：

*   **收到了更高 Term 的心跳或者其他消息**

这种情况说明集群中已经存在一个更高任期的节点，那也不需要继续选举了，直接默认该消息来源节点为 Leader。这里和上文中处理更高 Term 的 request_vote 消息类似。

*   **收到了半数以上的同意选票**

大部分节点同意了当前的选举要求，那该节点就成为这个任期的合法 Leader：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7gcSH45licGdtzyyFr7JWCnT3uOCD7lcA2iaOemrCsZttA1GeG0LTL1aA/640?wx_fmt=png&from=appmsg)

*   **在下个选举周期到来前还没收到半数选票**

Raft 算法不会给 Candidate 无限的计票窗口，如果一直没有收到足够的选票，就清零计票，重置 Term+1，再次发起选举，重复上述的过程，直到有个合法的 Leader 诞生。

接着上面网络分区的场景讨论，相信许多 Raft 系统的维护人员会发现，如果某个离群的节点在重新回归后会立即成为 Leader，就是这个原因，离群的节点在不停的自旋选举过程中将自己的 Term 抬高了很多，一但回到集群就会因为 Term 优势成为 Leader。为了解决这个问题，Raft 有个 prevote 的补丁，这里暂不展开讨论。

**五**

**Leader 的工作**

当 Candidate 成功成为 Leader 后，就需要承担 Leader 的责任。在 Raft 算法中，Leader 需要持续的向 Follower 发送心跳消息表明自己的存在。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7JnicVH8zHVicsab9aHfpc2XvdojP4gtMnHhDyKibVXtxqd951busDHsBQ/640?wx_fmt=png&from=appmsg)

Follower 收到心跳后，就可以重置自己的选举计时，起码在这个选举周期内，集群中有个 Leader 了，不用开启选举。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7U2DhLIUIrp1kLiaNQQMWpM6RhK266DvoRYkIib53UwsGOrib7HjvgHHQA/640?wx_fmt=png&from=appmsg)

Leader 收到心跳回执后，需要标记下 Follower 的状态，这个状态在集群可用性处理和后续的日志复制中有大用。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7ibQjJ9G21iaUxGaZMsD918s3Qv5bXZbASGw2nkcfq8zORBTEyic9DfwJw/640?wx_fmt=png&from=appmsg)

**六**

**日志复制**

上面的概念介绍中我们已经阐述过，Raft 本质就是多个副本的日志数据达成一致的解决方案。 我们已经实现了选主能力，但是尚未涉及到日志的部分。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7L3X3KK0PtH5eCp7YYicBrd5SVxqUiaBaJDxO6fWMibQJ5uc8ibC62JKjrQ/640?wx_fmt=png&from=appmsg)

接下来，我们将为选主集群添加日志复制能力。为了实现这个目标，我们需要一些新的数据结构和接口协议，同时也会对选主部分逻辑做增强的安全检查处理。

**从客户端行为开始**

Raft 集群的客户端会和集群产生交互，而交互的具体内容则是一个个具体的指令（Command）。例如一个 Raft 架构的 KVDB，客户端会向集群发送类似 PUT foo bar 的写请求或者是 GET foo 的读请求，这里的请求都可以算作指令的范畴。

Raft 集群在接收到客户端的指令后，根据论文第 5.3 节，会做如下的动作：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7xiaBhZ9aKENNONdN9cNq9F1VhNWRfVktrxJcvZmxOQKBiauKd5CibmK1Q/640?wx_fmt=png&from=appmsg)

*   如果当前节点不是 Leader，则会把 Command 转发至 Leader；
    
*   Leader 接收到指令后会先写入本地的日志仓库，然后向所有 Follower 发送**日志复制 (AppendEntries， 本实现中叫 Replicate) 请求；**
    
*   一旦 Leader 确认该 Command 已充分复制（即集群半数以上节点承认它们的日志仓库中成功复制了该 Command ），该 Command 就会被 commit；
    
*   Leader 会更新它的 commit 水位，在 Raft 中 commit 的日志即被认为是安全的日志，Leader 会在随后的心跳或者日志复制请求中携带 commit 水位，Follower 在收到心跳后更新本地 commit 水位；
    
*   各个节点会选择自己觉得合适的时机将已经 commit 的日志 apply 至状态机，这里时机的选择会很大影响 Raft 实现的吞吐和 RT。

这里是论文中提供的日志复制的简单例子：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L79nkTQ1g4eAnLaefan0mu5gibicUibzH3HFTicjoGyZQUxSj3p6w0ibd9xMQ/640?wx_fmt=png&from=appmsg)

按照论文的论述，这种日志复制的机制有 2 个特点：

1. 如果在不同的日志中的两个条目拥有相同的索引和任期号，那么他们存储了相同的指令。
2. 如果在不同的日志中的两个条目拥有相同的索引和任期号，那么他们之前的所有日志块也全部相同。

简单来说，如果 2 个节点中某个日志 Entry 的 index 和 Term 一样，那么从集群启动到日志记录的当下，集群中所有节点的状态是完全一致的。

这 2 个特点在论文中被定义为 “**日志匹配特性**”，由这个特性保证了 “**集群状态可预测性**”。

**数据结构**

为了实现日志复制功能，我们需要扩展一下数据结构。

**Entry**

Entry 即一个个日志块，每个 Entry 会携带一个 Command，为了让 Command 可以在集群中传播，Command 会序列化为一个 binary。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7Eu34ASZ6O0H72skKTp4zY6XBibUicKNBmiaANOaWGlfic0mQy74p3cGia4Q/640?wx_fmt=png&from=appmsg)

这里的日志类型中本篇会新增用到以下两个，其余的暂时用不上：

*   append_entries/append_entries_resp: 日志复制 / 日志复制回执
    
*   propose：提案

**Message**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7wcml8fdAhKlv68rgtgA5yjibCbNRHOb0Lqp9nZX77icXEbB8Fc7HSCpg/640?wx_fmt=png&from=appmsg)

Message 结构中，我们扩展了如下字段：

*   log_term/log_index：当前节点日志复制的最新进度；
    
*   commit: 已提交的日志水位；
    
*   hint：日志复制异常时携带的水位信息（这种情况一般会触发日志的覆盖）。

其他字段暂时用不到。

**日志仓库**

为了实现日志复制，每个节点都需要维护一个自己的日志仓库用于存储 Raft 的 Entry，我们为日志仓库抽象了如下的 API：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7KicIpv0QAUNYaUksO1JJnL31zTwQEcCLr1Hmclmd5wHfEb6cBjMt7Fg/640?wx_fmt=png&from=appmsg)

这里同样采取 API 和实现分离的设计方式，具体的实现是可拔插的。（按照 Elixir 的编程习惯，这里用 Protocol 的方式来定义更优雅）

在我们这个版本的实现中，我们采用了 Cubdb 作为日志仓库的实现基础。(Cubdb 是一个以 B + 树文件存储为基础的嵌入式 KVDB)。具体的实现方式可以参考代码。使用这个存储主要是为了简单考虑，如果考虑性能的话可以模仿龙舟的方式使用 wal 和 memcache 结合的方式，大大提升日志存储的吞吐性能。

**日志复制分步实现**

**发起提案**

这里提案（Propose）即客户端对 Raft 集群发起的指令（Raft 中读指令要比写指令复杂的多，读指令涉及到一致性相关讨论，感兴趣的可以搜索 Raft 中 read_index 的内容）。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L76qDMg63XlXvloHCEZEFFtN4Zstjj9nNsfhEgibdd0sPYqnEFthLBNoA/640?wx_fmt=png&from=appmsg)

接下来就是根据当前节点的不同角色来判断：

*   **如果当前节点是 Candidate**: 可以直接拒绝这个提案，毕竟集群都没 Ready。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7UYoOOtf5KNe7gAF4Jib8Q7Tve3v1aEO4iaMYlZjhA7ZGibukxHwAV3faA/640?wx_fmt=png&from=appmsg)

*   **如果节点为 Follower**: 就把提案转发到 Leader。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L768v4AGJG3RQibI2KvCBXyS3ia5WKtEKeXC5F3cCld2ucwHksibmvg7jqw/640?wx_fmt=png&from=appmsg)

*   **如果节点为 Leader**：进入提案处理流程，本实现中就是向本地节点发一条 propose 处理消息，这么做是为了复用 Term 异常处理等逻辑。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7EvWfD4pVfGGMbPZeGEEB9icE1FDPxW6Wz4oK4g84QWuL8pOxicRKYFcw/640?wx_fmt=png&from=appmsg)

**leader 复制提案**

Leader 在接收到 propose 后做如下动作：

*   从 propose 中提取需要复制的 Entries；
    
*   将 Entries 添加至本地日志仓库；
    
*   维护本地的日志水位；
    
*   如果日志复制前后日志水位发生变化，说明有新日志，就会开启日志复制广播。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7pb9eKQ0icCE1WAz410h6iaXVrB7p2jjZnYJhicXnKmKNpsJpA5MsumSOQ/640?wx_fmt=png&from=appmsg)

**Leader 广播日志复制消息**

这一过程比较简单，不过需要注意的是 Leader 会维护每个 Follower 的复制水位，这个水位是通过日志复制和心跳共同维护的。

日志复制会从每个 Follower 的复制水位开始，直到 Leader 的最新日志或者最大复制体积。

发送的日志复制请求会携带 Leader 记录的复制水位 index 和水位对应的 Term，这俩数据是非常重要的，Follower 会根据这两个标定来检查日志是否安全。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L79liaXHLQq96QATQQe7My7Aa6Erxd29Yz0wpzQia9yfaRnBQ7UnADD4bQ/640?wx_fmt=png&from=appmsg)

**Follower 处理日志复制请求**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7PXmicNqrW8OfXTfa3yDrDWuNCphe7SX0QkddSoqia02SEkchftKsXZug/640?wx_fmt=png&from=appmsg)

Follower 在接到日志复制消息后有如下情形：

*   **消息中日志复制起点 log_index 小于 Follower 本地已提交水位 (commit index)**

这种情形说明 Leader 的复制水位已经落后太多，没必要做更多的动作，只需要回复 Leader 最新的 commit 水位即可。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7icfmKlzwUFLr8tS1YsIl3icW9Dcb7W0uWsdxEa5rXMyxqicKfMJINIdbw/640?wx_fmt=png&from=appmsg)

*   **消息中****日志****复制起点 log_index 小于等于 Follower 的最新日志进度**

这种情况说明 Leader 发来的日志可能与 Follower 的日志有**部分重叠**，这个时候要非常谨慎，需要仔细对比 Leader 的日志和本地日志是否匹配。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7iacMr7aRxO9lZNnGXGSsbicicd1fkfyoF95gIXxUEWichGZZ2BZqIYYCuw/640?wx_fmt=png&from=appmsg)

这里的日志匹配的过程分为这么几步：

*   判断日志复制起点的任期和本地是否一致，如果不一致立即拒绝复制；
    
*   如果任期一致，从消息中找到第一个与本地日志块不一致的日志 Entry 即为匹配起点，不一致指的是任期和 index 任意一个不一致；
    
*   当找到匹配起点后，后续所有日志都是用 Leader 的日志覆盖本地。

为何需要在日志复制时做这么严格的任期检查呢？这个问题先保留，后面再细致讨论。

*   **日志复制起点大于 Follower 的最新日志进度**

这种情况就比较简单了，日志都跳号了，肯定不能接受，直接回绝即可。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7fHh97xsDUTk45oaftRkGICiaG2gGib56347ib84stHaiaiaojmjljZMtaHw/640?wx_fmt=png&from=appmsg)

**Leader 处理复制回执**

Leader 在接收了 Follower 的日志复制回执后会有如下情形：

*   **复制成功**

这种为正常情况，Leader 会更新 Follower 的水位，并且发起一次 commit 的尝试。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7RPwUt5hibdJu2FoZ1uSKibYrkIqHoK3ogsWBONsO7nxPLc0JhsfxxPXA/640?wx_fmt=png&from=appmsg)

commit 的请求按照论文，必须选择至少半数复制成功的安全水位。

这里取安全水位的做法参考了龙舟的技巧，每次收到回执后将所有节点的 match 水位排序后取中位数即为半数同意的安全水位，非常巧妙。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7W9R4HS1TrLjpyc0ib91CTKLEfmlUW5XLON5b3ZYnmXkaOj6U0KsU0bg/640?wx_fmt=png&from=appmsg)

值得注意的是，在 commit 之前我们还特地做了一次 Term 的检查工作，这里是 Raft 的安全性检查，后文中会详细讨论。

*   **复制失败**

异常情况下，按照论文的描述，Leader 不用做日志回滚，只需要调整复制的起点水位即可。复制的起点水位取本地记录的 match 水位和 Follower 返回的合理水位中的较小值（重复了没有问题，不连续才是大问题）。

如果这个过程中 Follower 的日志已经脏了，这个安全水位开始的日志可以覆盖纠正 Follower 的日志。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7U2rF590180ac3WiaGTkjvibDodSYoiaEg6ic9qxgDjmFv3ZXjQwoLpY6gg/640?wx_fmt=png&from=appmsg)

**日志复制安全性讨论**

在上节的分步实现中，我们多次做了日志的任期检查：

*   Follower 在接收到日志复制消息后，会对比复制消息的 Term 和本地日志 Term 是否一致；
    
*   Leader 在收到日志复制成功回执后 commit 之前，会校验 commit 水位的 Term 和当前 Term 是否一致（即不允许跨任期提交）；

为什么 Raft 需要不厌其烦地做 Term 和 index 的检查呢？

在 Raft 论文中有如下论述：

在正常的操作中，Leader 和 Follower 的日志保持一致性，所以日志复制 RPC 的一致性检查从来不会失败。

然而，Leader 意外崩溃的情况会使得日志处于不一致的状态（老的 Leader 可能还没有完全复制所有的日志块）。这种不一致问题会在 Leader 和 Follower 的一系列崩溃下加剧。

下图展示了 Follower 的日志可能和新 Leader 不同。Follower 可能会丢失一些在新的 Leader 中存在的日志块，也可能拥有一些新 Leader 没有的日志块，或者两者都发生。丢失或者多出日志块可能会持续多个任期。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7RwYZCl0sCJ2HQYPbXic90wYvCH6M5lcBhjia1MiaNricqiaNqI4Ut78Z5WA/640?wx_fmt=png&from=appmsg)

当一个 Leader 成功当选时，Follower 可能是任何情况（a-f）。每一个 Box 表示是一个 Entry；里面的数字表示任期号。Follower 可能会缺少一些 Entries（a-b），可能会有一些未被提交的 Entries（c-d），或者两种情况都存在（e-f）。

例如，场景 f 可能会这样发生，某节点在任期 2 的时候是 Leader，已附加了一些 Entries 到自己的日志仓库中，但在提交之前就崩溃了；很快这个机器就被重启了，在任期 3 重新被选为 Leader，并且又增加了一些 Entries 到自己的日志仓库中；在任期 2 和任期 3 的日志被提交之前，这个节点又宕机了，并且在接下来的几个任期里一直处于宕机状态。

在 Raft 算法中，Leader 是通过强制 Follower 直接复制自己的日志来处理不一致问题的。这意味着在 Follower 中的冲突的日志块会被 Leader 的日志覆盖。

要使得 Follower 的日志进入和自身一致的状态，Leader 必须找到最后两者达成一致的地方，然后删除 Follower 从那个点之后的所有日志块（水位重置），并发送自己在水位之后的日志给 Follower。

所有的这些操作都在进行日志复制 RPC 的一致性检查时完成。Leader 针对每一个 Follower 维护了一个 **nextIndex**，这表示下一个需要发送给 Follower 的日志块的索引。当一个 Leader 刚刚被选出的时候，它初始化所有的 nextIndex 值为自己的最后一条日志的 index 加 1（图中的 11）。

如果一个 Follower 的日志和 Leader 不一致，那么在下一次的日志复制 RPC 时的一致性检查就会失败。在被 Follower 拒绝之后，Leader 就会减小 nextIndex 值并进行重试。最终 nextIndex 会在某个位置使得 Leader 和 Follower 的日志达成一致。当这种情况发生，日志复制 RPC 就会成功，这时就会把 Follower 冲突的日志块全部删除并且加上 Leader 的日志。一旦附加日志 RPC 成功，那么 Follower 的日志就会和 Leader 保持一致，并且在接下来的任期里一直继续保持。

通过这种机制，Leader 在获得权力的时候就不需要任何特殊的操作来恢复一致性。他只需要进行正常的操作，然后日志就能在**日志复制 RPC——日志复制回复处理**这一过程中自动趋于一致。Leader 从来不会覆盖或者删除自己的日志。

**Rethink 集群选主**

在上面的篇幅中，我们讨论过集群选主的过程，一个节点只要拿到集群半数以上的 request_vote 的成功回执即可成为 Leader，而且投票的原则非常简单，只要这个任期我没投过票或者我投过相同的节点即可赞成投票。

不过在添加日志复制后这里会出现一个小问题，例如在下图中：

如果 f 节点发起选举并得到了大多数成员认可，那么集群大部分节点就会面临日志回退，已经 “安全” 的日志都变得不再安全，这不符合 Raft 的设计初衷，所以我们需要在投票时添加一个机制防止类似 f 节点这类比较 “旧” 的节点当选为 Leader。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7RwYZCl0sCJ2HQYPbXic90wYvCH6M5lcBhjia1MiaNricqiaNqI4Ut78Z5WA/640?wx_fmt=png&from=appmsg)

论文中有如下论述：

Raft 使用投票的方式来阻止一个 Candidate 赢得选举，除非这个 Candidate 包含了所有已经提交的日志块。Candidate 为了赢得选举必须联系集群中的大部分节点，这意味着每一个已经提交的日志块在这些服务器节点中肯定存在于至少一个节点上。如果 Candidate 的日志至少和大多数的服务器节点一样新（这个新的定义会在下面讨论），那么他一定持有了所有已经提交的日志块。请求投票（RequestVote）RPC 实现了这样的限制：

RPC 中包含了 Candidate 的日志信息，然后投票人会拒绝掉那些日志没有自己新的投票请求。

简单来说就是：投票的时候如果拉票人的日志没我本地的新，那我就不能投票给你。

代码实现：

*   发起投票：发起投票时 Candidate 需要在 request_vote 消息中携带本地的日志任期和最新的日志 index；

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7uegBno7lPqB8KHtEBLRkGibpLth5MXQCkeFHTT5ZPM4GzqugV3ZbfRA/640?wx_fmt=png&from=appmsg)

*   处理投票：其他节点处理 request_vote 消息时，需要检查拉票消息的日志是不是要比本地更新（日志的 Term 大于本地或者同个 Term 下的日志 index 大于本地）。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7og1vqpoGzX9LvfZDuIYU4YhvwtOUn12gyDicKEOmj0kBVYDtkicSICww/640?wx_fmt=png&from=appmsg)

**Leader 提交过往任期的日志讨论**

在日志复制的第 5 步中，Leader 获取了半数以上的安全水位执行提交，不过提交前判断了是否与当前任期一致，如果出现跨任期的情况，就会暂缓这次提交，为什么需要这一步呢？

论文有如下论述：

对 Leader 来说，只要当前任期的日志被存储到了大多数的服务器上，那么这个日志是可以被提交的。

如果 Leader 在提交日志块之前崩溃了，未来后续的 Leader 会继续尝试复制这条日志块。然而，一个 Leader 不能断定一个之前任期里的日志块被保存到大多数服务器上的时候就一定已经提交了。

下图 8 展示了一种情况，一条已经被存储到大多数节点上的老日志块，也依然有可能会被未来的 Leader 覆盖掉。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7cGFb9PMSiaIbqvY9PEs79LMIUVydlriaaghMIupAQknef5KVO1ey9LcQ/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7cGFb9PMSiaIbqvY9PEs79LMIUVydlriaaghMIupAQknef5KVO1ey9LcQ/640?wx_fmt=png&from=appmsg)

如图的时间序列展示了为什么 Leader 无法决定对老任期号的日志块进行提交。

(a) S1 是 Leader，部分 (Follower) 复制了 index=2 的日志块。

(b) S1 崩溃了，然后 S5 在任期 3 里通过 S3、S4 和自己的选票赢得选举，然后从客户端接收了一条不一样的日志块放在了索引 2 处。

(c) S5 又崩溃了；S1 重新启动，选举成功，开始复制日志。这时来自任期 2 的那条日志已经被复制到了集群中的大多数机器上，但是还没有被提交。

(d) 如果 S1 又崩溃了，S5 可以重新被选举成功（通过来自 S2、S3 和 S4 的选票），然后覆盖了他们在 index=2 处的日志。

(e) 如果在崩溃之前，S1 把自己主导的新任期里产生的日志块复制到了大多数机器上，那么在后面任期里面这些新的日志块就会被提交（因为 S5 就不可能选举成功）。这样在同一时刻就同时保证了，前的所有老的日志块就会被提交。

简单来说就是：过往任期的 “安全” 水位不一定是安全的，完全可能在 Leader 切换后被覆盖，只有当前任期的 “安全” 水位是安全的，因为其他日志不一致的节点没法在该任期内当选 Leader 只能被覆盖。

**七**

**用户状态机**

前面的篇幅中我们已经介绍了 Raft 的 2 个核心部分：

*   集群选主
    
*   日志复制

但是现在的 Raft 集群还没法集成任何业务，因为缺了和业务息息相关的一环——**用户状态机**。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7ib2N2aee3DFmsMTico7derUVqT4eXO0dOojN09BwHTToO1fNawVfNFMw/640?wx_fmt=png&from=appmsg)

接下来，我们将完成 Raft 拼图的最后一块，将业务能力集成，并尝试用这个完全体的 Raft 制作一个分布式的 KVDB。

**什么是用户状态机？**

在 Raft 一致性算法中，**业务状态机**（也称为应用状态机或用户状态机）指的是实际执行业务逻辑的组件。当 Raft 处理完日志条目（通常是客户端发送的命令或操作请求）并通过选举和日志复制机制达成一致后，这些日志条目会被提交给状态机进行处理。

业务状态机是确定性的，这意味着对于任何给定的输入，它总是产生相同的结果，无论在哪个节点上运行。这个特性保证了，只要日志条目的顺序相同，所有节点上的状态机经过相同的序列化输入后会达到相同的状态。因此，即使集群中发生故障，只要有一个存活的节点，就可以通过复制其状态来恢复其他节点，从而保持整个系统的状态一致。

在 Raft 中，业务状态机的职责包括：

*   执行命令：解析并执行由 Raft 提交的日志条目中的命令或操作。
    
*   更新状态：根据命令的结果更新内部状态。
    
*   返回结果：将操作结果返回给客户端或其他组件。
    
*   持久化状态（optional）：将关键状态写入磁盘，防止数据丢失。
    
*   快照：定期创建状态机的快照，减少日志的大小，提高性能。

需要注意的是，Raft 算法本身不关心业务状态机的具体实现细节，它只负责保证日志条目的顺序一致性和可用性。这使得 Raft 能够适用于各种不同的分布式系统和应用场景，例如数据库、缓存、配置管理系统等。

**定义状态机行为**

参照上面的概念定义，我们可以定义用户状态机的行为：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7395XUC6RydrSOMRLGfibnC3QtXINH2wm9DweUniad277qc4k2hcsumibQ/640?wx_fmt=png&from=appmsg)

我们的状态机约定有如下 API：

*   开启、关闭状态机；
    
*   执行 Raft 日志中的业务命令；
    
*   对应用状态机执行当前状态的读命令；
    
*   准备快照的检查点，用于快照保存；
    
*   保存快照；
    
*   加载快照。

Raft 中是这么定义 “快照” 的，后文有更加详细的解释：

快照（Snapshot）是一种重要的机制，用于减少系统的状态大小并提高性能。在 Raft 中，快照的主要目的是记录系统的一个完整状态点，这样就不需要从大量的日志条目中重新计算整个状态。

**应用状态机与 Raft 日志的交互**

应用状态机提供的各个 API 是和 Raft 的日志复制紧密相关的，我们分开讨论每个 API 的使用时机。

**执行 Raft 日志中的业务命令**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L75evhWdQmrZTXkZywVqBCx2kE21Ap0S4xk3libK3uFCAhsO1wdt52jHw/640?wx_fmt=png&from=appmsg)

Raft 的状态中保存了 2 个水位：

*   commit_index: 一个上上篇文章中讨论的复制的安全进度（集群半数以上的复制水位）；
    
*   last_applied: 已经复制到应用状态机的水位。

当 commit_index>last_applied 且 commit_index 任期与当前任期一致（原因见上篇文章 Leader 提交过往任期的日志讨论）时，即可将（last_applied, commit_index] 这个范围的日志应用到应用状态（即调用 update 方法）。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7CgXJ9OQVEqwqtNvUYTO1VUQdxLqsKJDHs8eROvf16tk7drNU9RwHqQ/640?wx_fmt=png&from=appmsg)

而执行 apply 的时机就很有讲究了，各个 Raft 框架的实现各不相同，合理的 apply 时机和整个 Raft 框架的性能息息相关，开发者需要自己在框架的吞吐、提案的延迟中取一个合理的折中。我们这里的实现较为简单，定时 apply 到 statemachine 即可。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7CrLibI1pAicsFnpxU5tYhS8OnQ6fvJNxE9eS8piaKFPWufhfSq1OHicGIA/640?wx_fmt=png&from=appmsg)

**保存快照**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7AgYG6CibiaU9aOHEDt67Bgo5Y8ZMWoIibQjmc7c4U8sLLd39SnR2GNpbA/640?wx_fmt=png&from=appmsg)

快照（Snapshot）是一个重要的优化机制，用于处理日志文件不断增长所带来的存储和性能问题。随着系统的运行，Raft 需要记录越来越多的状态转换指令，这些指令以日志条目的形式存储，以便在需要时能够重放这些指令来重建系统状态。

然而，随着日志的增长，以下问题逐渐显现：

*   **存储成本上升**：因为需要保存更多的日志条目。
    
*   **性能下降**：特别是在服务器重启或新成员加入集群时，需要重放大量的日志条目来恢复状态，这可能导致长时间的延迟。

为了解决这些问题，Raft 引入了快照机制。快照是对系统当前状态的一个完整拷贝，它包含了从系统启动到快照生成时刻的所有状态信息。一旦快照创建完成，它所代表的那一刻之前的所有日志条目就可以被安全地删除，因为快照包含了那些日志条目所能提供的所有信息。

以下是快照在 Raft 中的一些关键作用：

*   **状态压缩**：快照可以显著减少存储需求，因为它取代了之前所有的日志条目。
    
*   **状态恢复**：当服务器重启或者新节点加入集群时，可以从最新的快照恢复状态，而无需重放所有日志条目。
    
*   **日志同步**：如果某个节点的日志与领导者不同步，领导者可以向该节点发送快照，帮助其快速更新至最新状态。

快照的生成时机通常是根据预设的策略，比如日志条目达到一定数量或存储使用量超过阈值。

在 Raft 中，快照的生成和传输过程需要谨慎设计，以确保数据一致性和系统的稳定性。

在我们这个玩具实现中，就不实现复杂的自动快照了，实现一个手动快照入口即可。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7O1OUSaedgL65o5R4RYz1af50ThcMoZ7KwU1ibjQoGCKIYxu9h9aLFmg/640?wx_fmt=png&from=appmsg)

快照的构成包括 2 个部分：

*   **Raft 集群当前状态**：包括各个节点的 id、任期、保持快照时的最新 index 和集群节点构成等，这部分我们称之为 **snapshot_metadata。**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7az1ibwfI5cJ7XTmk6tTuRialNo8R9dxBrsgvKHiarOqVA1ZzicCHrLtuAg/640?wx_fmt=png&from=appmsg)

*   **用户状态机想要持久化的数据**：这部分数据由业务自己实现管理。

Raft 无法知道用户状态机想要持久化的数据，所以在保存快照时，携带 snapshot_metadata 调用用户状态机的快照 API。这里我们设计了很简单的快照格式，用一个 4 字节的 header 保存 snapshot_meta 的长度，方便恢复的时候做长度切割。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7LGhd2XZKCHm2h2qGaWpNrict50PLNdZoAmGCqLicZksnXlCA6n62W2eA/640?wx_fmt=png&from=appmsg)

我们的玩具实现中只是单纯的存在当前节点磁盘，生产中可以根据实际的业务需要将快照存储在三方存储（例如 s3）中方便管理。

**快照恢复**

既然有快照保存，那就一定有快照恢复，快照恢复有以下几种情形：

*   **集群节点重启**：需要加载本地快照恢复到最近的持久化进度，后续再从进度处重复日志达到一致性；
    
*   **新节点加入集群**：由于新节点没有本地快照，需要从远端拉取快照，可以从 Leader 处获取也可以从单独的快照服务中获取。

第二种情形较为复杂，我们就先不讨论了，这里只实现第一种情况：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L75X9ayq7LiazEUKwMrmNabq36QKop9ljhIXiaKPWoL8cPh5QC0g7V6Buw/640?wx_fmt=png&from=appmsg)

快照恢复的过程就是快照制作的反过程，我们从存储（磁盘）上读出快照数据，parse 为业务快照和 metadata，其中:

*   metadata 可以用于恢复集群成员、任期、最新 index 等数据；
    
*   业务快照则可以调用 statemachine 的 load_snapshot api 恢复业务数据。

**八**

**编写一个分布式 KVDB**

至此，我们已经介绍完了 Raft 中核心的概念，为了更直观的感受用 Raft 的功能，我们选择一个很简单的业务场景来验证——KVDB。

我们的玩具 KVDB 将支持以下能力：

*   添加 kv 对，如果相同 key 就覆盖；
    
*   删除 kv 对；
    
*   读某个 key 对应的 val。

**状态机实现**

为了实现这个 KVDB，我们首先按照业务状态机的业务约定实现状态机接口。

这里的 kv 数据全部存储在内存 map 中，update 和 read 方法即对该 map 的读写。

快照实现也非常简单，将所有的 kv 对 dump 为一个 json，快照加载的时候读取这个 json 恢复所有 kv 即可。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L75QekdunOVC1oNSDGTkqFltBkFEexYR7ia2rVKKNZfbrg6mBrf9RCibWQ/640?wx_fmt=png&from=appmsg)

**完整的 Raft-base KVDB**

有了这个状态机，我们就可以将我们编写的 Raft 框架和这个状态机联合起来，组成一个 Raft-kv。

我们简单组织一下上层的 API：

*   show：展示当前集群结构；
    
*   put：写入一个 kv；
    
*   delete：删除一个 key；
    
*   read：读取一个 key；
    
*   save_snapshot：快照落盘。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7ic7l8hdgIhJcOJ5s31LqgmUbVIzn6CA8zMXwxQALoQmcicmd8e33Uvibw/640?wx_fmt=png&from=appmsg)

从代码可知，这里的写入和删除都是对 Raft 集群发起一个提案，而读则是采用的较为简单的 “脏读” 策略，直接读的本地内存（如果想要更为严格的一致性，需要实现 Raft 提供的**一致性读 API**）。

接下来就是激动人心的时刻，我们将把从选主到状态机所有的代码串联起来，检验 Raft 是否能正常工作。

**验证**

**Raft 集群组建**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L77ztMPRyTYSiciapFIOn8MVYicBoJiahqsM04dnoIREwicUDRfpuPLa0bnZA/640?wx_fmt=png&from=appmsg)

这里我们在本地启动 3 个 Erlang 进程，每个进程有个独立的 node id 和 port，各个 node 通过 Erlang 虚拟机相互发现通信。

配置文件中，我们将这 3 个 node 组成一个 Raft 集群。

从图中的 show 命令执行结果可知，3 个节点成功组成了稳态的 Raft 集群，节点 1 为 Leader，2 和 3 为 Follower。

**读写作业**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L7kyyv332VFrGicO64PbAeOzKWuXU4oD10qRjbO7yPicy6LERJGJI44AdQ/640?wx_fmt=png&from=appmsg)

*   在 1 节点写入 foo1:bar1；
    
*   在 2 和 3 节点读取 foo1，均可以读出结果 bar1；
    
*   在 2 节点删除 foo1，在 1 节点和 3 节点均无法继续读出 foo1。  

由结果可知，我们的写入和删除提案都被 3 个节点复制执行成功。

**快照落盘与恢复**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L728S1n75zdADk6yfLYs7C4zyxwibCqH2NwAl11tKnaMXo3IqzicleyxaQ/640?wx_fmt=png&from=appmsg)

*   在 1 节点写入 foo1:bar1 和 foo2:bar2 两个 kv；
    
*   执行快照落盘操作，将一个 json 存入本地磁盘；
    
*   重启该 node，该节点从 Leader 转换为 Follower；
    
*   读取 foo1，得到结果 bar1；
    
*   读取 foo2，得到结果 bar2；  

由实验结果可知，我们的快照制作和快照恢复功能生效了，节点可以从磁盘中恢复状态和集群达到一致性。

**九**

**总结**

本文深入探讨了 Raft 日志复制算法的核心概念和实现细节。从日志复制状态机的基础，到 Leader、Follower、Candidate 的角色定义，我们逐步揭开了 Raft 算法的神秘面纱。我们选择了 Elixir 作为实现语言，展示了其在构建高效、可扩展的分布式系统中的潜力。

通过分步实现日志复制，我们了解了如何发起提案、Leader 如何复制提案以及 Follower 如何处理日志复制请求。安全性讨论让我们认识到了在分布式系统中保证数据一致性的重要性。此外，我们还探讨了 Raft 算法在集群选主和用户状态机中的常见问题和陷阱，以及如何通过定义状态机行为来执行业务命令。

在实践层面，我们学习了如何编写一个分布式 KVDB，并验证了其正确性。这包括了 Raft 集群的组建、读写操作的执行、以及快照的落盘与恢复。这些实际操作不仅加深了我们对 Raft 算法的理解，也为我们提供了宝贵的实践经验。

本文仅浅尝辄止地了解了一下 Raft 的核心知识，制作了一个玩具的 Raft 实现，而生产级别的 Raft 框架需要考虑的问题更多了。感兴趣的开发者可以在本文的基础上进阶思考一下这些问题：

*   如何实现一致性读？
    
*   如何避免网络分区的节点重新回到集群立即成为 Leader？
    
*   如何选择日志 commit 和 apply 的时机可以让 Raft 框架获得最理想的吞吐？
    
*   如何实现动态的节点增删？

随着分布式理论的发展，业界也对 Raft 做了很多的扩展和升级，例如：

*   通过 Raft 集群的分级并联获取更高的硬件资源利用率；
    
*   分拆日志匹配的 job 为一个无状态的服务，增加 Leader 的水平日志处理能力；
    
*   增加 Observer 角色（只复制不参与选主）横向扩展集群的读能力。

更多奇思妙想可以参考 ACM 上的论文。

随着技术的不断进步，我们有理由相信 Raft 算法及其在分布式系统中的应用将会更加广泛。在未来，我们期待看到更多创新的实现和应用场景，以解决日益复杂的分布式计算问题。让我们拭目以待，共同见证分布式技术的未来发展。

**引用：  
**

*   https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying
    
*   https://zhuanlan.zhihu.com/p/618127949
    
*   https://github.com/maemual/raft-zh_cn/blob/master/raft-zh_cn.md#%E5%AF%BB%E6%89%BE%E4%B8%80%E7%A7%8D%E6%98%93%E4%BA%8E%E7%90%86%E8%A7%A3%E7%9A%84%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%E6%89%A9%E5%B1%95%E7%89%88
    
*   https://github.com/lucaong/cubdb
    
*   https://dl.acm.org/action/doSearch?AllField=raft

**往期回顾**

1. [链路级资损防控之资损字段防控实践｜得物技术](http://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247527972&idx=1&sn=24fea420cd6d981b3bba7c6d9bd7da9f&chksm=c1613b7bf616b26df9d51705e58f99568d9aa81d06a91c3cfa7e2eb9250686146c134725ec76&scene=21#wechat_redirect)  
2. [前端打包工具 Mako 架构解析｜得物技术](http://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247527699&idx=1&sn=b9457248f09d54a154478f87682d7798&chksm=c161384cf616b15a56d1ac2b01d9dc06ebd476d4008414601ec36a1260f5bd570819615c6471&scene=21#wechat_redirect)  
3. [基于 Rspack 实现大仓应用构建提效实践｜得物技术](http://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247527650&idx=1&sn=abb64fd1a1b3c72b5b8181ce8a4fd0d9&chksm=c16139bdf616b0abaccb01477d7f82bcdfe1f4df2707c6ccf755d94ad478f96be2a9b5580a4a&scene=21#wechat_redirect)  
4. [星愿森林的互动玩法揭秘｜得物技术](http://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247527623&idx=1&sn=1eb79e88e2a2db776462507e430d7bd4&chksm=c1613998f616b08efd9bcd2cf1749369c938a1f614dad7511a9a7fe9cae51cd971a03e08dc59&scene=21#wechat_redirect)  
5. [StarRocks 跨集群迁移最佳实践｜得物技术](http://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247527446&idx=1&sn=7454faec48e966fafaae2e26829e6a56&chksm=c1613949f616b05f1defbbd62dcbe9c33f03b4b43e9b28bb03b7fb3fafe5c5032e90298abf33&scene=21#wechat_redirect)

文 / 古月方块

关注得物技术，每周一、三、五更新技术干货

要是觉得文章对你有帮助的话，欢迎评论转发点赞～

未经得物技术许可严禁转载，否则依法追究法律责任。

“

**扫码添加小助手微信**

如有任何疑问，或想要了解更多技术资讯，请添加小助手微信：

![](https://mmbiz.qpic.cn/mmbiz_jpg/AAQtmjCc74DiaFyTgdHwM2FvxdQHQQ3L71KPvOf4t11u6gfL9QYJyIR4Rom8YLxs6h1rlsEQObfhnQSiaoT7u3IQ/640?wx_fmt=jpeg&from=appmsg)