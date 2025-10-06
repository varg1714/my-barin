---
source: https://mp.weixin.qq.com/s/PsRA9kB6KE9EB17-G-kOlA
create: 2025-08-04 22:48
read: true
knowledge: true
knowledge-date: 2025-08-11
tags:
  - Redis
---
![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97bCSICdmD5k5Jibib2IpkhLFtkOzvD2504vSVCg0ricAJ8DBknMIDgGN2c7LmRtoAwLDibBkZTMCkALA/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp&randomid=uhpx8hlk)

![](https://mmbiz.qpic.cn/mmbiz_gif/VY8SELNGe96srmm5CxquJGSP4BbZA8IDLUj8l7F3tzrm8VuILsgUPDciaDLtvQx78DbkrhAqOJicxze5ZUO5ZLNg/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&tp=webp&randomid=0rksa1wo)

  

Redis （ Remote Dictionary Server）： 是一个开源的内存数据库，提供了一个高性能的键值 (key-value) 存储系统，常用于缓存、消息队列、会话存储等应用场景。Redis 在腾讯内部应用相当普遍，也是技术面试中常见的考题方向，你可能经常使用到 Redis 服务，但往往都是使用 get、set 等命令，没有多想过这些命令是如何执行的？本文为你解答清楚。

  

关注腾讯云开发者，一手技术干货提前解锁👇

  

🕓6 月 26 日（周四晚）19:30，四位架构专家将从技术与实践的角度，深入探讨 AI 时代的企业 IT 服务管理，从业务流程变革到理论落地的最佳案例，展望未来 10 年的 IT 服务行业前景，带你一窥行业变革的数字密码。参与直播互动还可抽取定制「腾讯公仔」，快点击预约直播吧👇

我一开始想这个问题的时候的答案是： 客户端发送命令给服务端， 服务端收到执行之后再处理将命令执行结果返回给客户端。那更细节的过程呢？

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HJGew5BqNwuSdtWupxYyiaICGFyCkJuiaeV4LcHAKDiaAcicxibe76X1rAPrLvCIb6Zs1UF6G2QnTlibw/640?wx_fmt=png&from=appmsg&randomid=0qx2pts4)

那么在了解一条 Redis 命令是如何执行之前，我们首先来看看 Redis 的架构。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HJGew5BqNwuSdtWupxYyiaMJ1hgSCbVgBKWqrstEdBicgJOjAicb0F6DCU6dMwRbSATsxSibPYg4aEA/640?wx_fmt=png&from=appmsg&randomid=svmht5cx)

Redis 的部署架构（宏观视角）

1.  单机模式
    
    基础部署形态，单节点运行，无高可用保障，适用于开发测试或非核心场景。
    
2.  主从复制（Master-Slave）
    
    数据从主节点同步至多个从节点，提供读写分离能力，增强读性能与数据冗余。
    
3.  哨兵模式（Sentinel）
    
    在主从复制基础上，引入监控节点自动故障转移，实现高可用（HA），当主节点故障时自动选举新主节点。
    
4.  集群模式（Cluster）
    
    分布式架构，数据分片存储在多个节点，支持自动故障转移与水平扩展，适用于海量数据与高并发场景。
    

Redis 的核心组件（微观视角）

1.  事件驱动引擎
    
    基于 I/O 多路复用（如 epoll）实现的高性能网络模型，单线程处理并发请求，避免线程切换开销。
    
2.  命令处理层
    
    解析客户端命令，执行对应操作逻辑（如 GET/SET/HSET），支持丰富的数据结构（String、Hash、List 等）。
    
3.  内存管理系统
    
    负责内存分配与回收，支持内存淘汰策略（如 LRU、LFU），优化内存使用效率。
    
4.  持久化模块
    
    - RDB（快照）：定期生成二进制快照文件，恢复速度快但可能丢失部分数据。
    
    - AOF（日志）：追加写命令到日志文件，数据安全性高，支持重写机制压缩日志。
    
5.  监控与统计系统
    
    提供运行状态监控（如内存使用、QPS）、慢查询分析、性能指标采集等功能，辅助运维优化。
    

这些核心组件的作用：

<table><tbody><tr><td data-colwidth="180" width="277"><section><span leaf="">架构层级</span></section></td><td data-colwidth="149" width="141"><section><span leaf="">核心功能</span></section></td><td data-colwidth="230" width="230"><section><span leaf="">关键技术特性</span></section></td></tr><tr><td data-colwidth="180" width="277"><section><span leaf="">单机模式</span></section></td><td data-colwidth="149" width="141"><section><span leaf="">基础运行单元</span></section></td><td data-colwidth="230" width="230"><section><span leaf="">单线程内存操作</span></section></td></tr><tr><td data-colwidth="180" width="277"><section><span leaf="">主从副本</span></section></td><td data-colwidth="149" width="141"><section><span leaf="">数据冗余与读写分离</span></section></td><td data-colwidth="230" width="230"><section><span leaf="">异步复制、PSYNC 机制</span></section></td></tr><tr><td data-colwidth="180" width="277"><section><span leaf="">哨兵集群</span></section></td><td data-colwidth="149" width="141"><section><span leaf="">故障自动转移</span></section></td><td data-colwidth="230" width="230"><section><span leaf="">监控选举、配置传播</span></section></td></tr><tr><td data-colwidth="180" width="277"><section><span leaf="">分片集群</span></section></td><td data-colwidth="149" width="141"><section><span leaf="">横向扩展数据存储</span></section></td><td data-colwidth="230" width="230"><section><span leaf="">哈希槽分区、Gossip 协议</span></section></td></tr></tbody></table>

主要的名词解释：

*   Redis 客户端：与 Redis 服务器交互的程序或工具（如 Jedis、Lettuce、redis-cli 等）； 能够通过 TCP 协议连接 Redis 服务器（默认端口 6379），使用 RESP（Redis Serialization Protocol）协议通信，发送命令（如 SET, GET）并接收响应。
    
*   事件驱动层：单线程 Reactor 模式（6.0 后引入多线程 IO）； 组成为：文件事件处理器：通过 I/O 多路复用（如 epoll/kqueue）监听套接字；时间事件处理器：处理定时任务（如过期键清理）。
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe96KXwx2BLy4ZO6QFkibV3VibVASWrXuvB6bicawBiczHjsCxVOAokrvAveOvjq4nt5xat4rTFvZBNXqVw/640?wx_fmt=png&from=appmsg&randomid=r1d87e3j)
    
*   命令层：解析客户端请求（将 RESP 协议转换为内存数据结构）、校验命令合法性、执行命令逻辑（Get、Set 等）。
    
*   内存分配 / 回收：使用 jemalloc/tcmalloc 代替系统 malloc（减少碎片），内存淘汰策略（LRU/LFU/random/TTL 等）。
    
*   RDB 与 AOF：Redis 提供的持久化策略，以保证数据可靠性。
    
*   副本（Replaction）：Redis 通过副本，实现【主 - 从】运行模式，是故障切换的基石，用于提高系统运行可靠性，支持读写分离、提高性能。
    
*   哨兵（Sentinel）：哨兵用于支持故障时，主从节点自动切换。哨兵为 Redis 高可用提高了保证。
    
*   集群（Cluster）：Redis 基于数据分片，支持横向拓展的一种高性能模式。主节点负责数据存储，从节点作备份。
    
*   监控与统计：Redis 提供监控信息和性能分析工具，包括内存使用（used_memory）、命令统计 (commandstats) 等。
    

核心模块：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe96KXwx2BLy4ZO6QFkibV3VibVIm4Y42YIgn7cVZoP56S5HDiaP43B1uVNnLXSUHAE3Z6UDpuImp2mAlg/640?wx_fmt=png&from=appmsg&randomid=m6sgj08g)

在分析命令是如何执行之前，我们需要关注 Redis 最核心的模块 - 事件驱动， 事件驱动也是 Redis 高性能的基石。

1、Reactor 模式实现

```
// 核心数据结构：aeEventLoop
typedef struct aeEventLoop {
    int maxfd;                // 最大文件描述符
    aeFileEvent *events;      // 注册的文件事件数组
    aeTimeEvent *timeEventHead; // 时间事件链表
    aeFiredEvent *fired;      // 触发事件数组
} aeEventLoop;
```

2. 事件处理的三要素

```
1. 事件注册 → 2. 多路复用监听 → 3. 事件派发
```

Redis 注册的事件可以分为： 文件事件（fileEvent） 和 时间事件 （timeEvent）

<table><tbody><tr><td data-colwidth="150" width="277"><font><span leaf="">事件类型</span></font><font></font></td><td data-colwidth="141" width="141"><font></font><font><span leaf="">触发条件</span></font><font></font></td><td data-colwidth="203" width="230"><font></font><font><span leaf="">处理函数示例</span></font><font></font></td><td data-colwidth="157" width="223"><font></font><font><span leaf="">注册时机</span></font><font></font></td></tr><tr><td data-colwidth="150" width="277"><font></font><font><span leaf="">文件事件</span></font><font></font></td><td data-colwidth="141" width="141"><section><span leaf="">套接字可读 / 可写</span></section></td><td data-colwidth="203" width="230"><section><span leaf="">acceptTcpHandler</span></section></td><td data-colwidth="157" width="223"><section><span leaf="">服务启动 / 新建连接时</span></section></td></tr><tr><td data-colwidth="150" width="277"><font></font><font><span leaf="">时间事件</span></font><font></font></td><td data-colwidth="141" width="141"><section><span leaf="">定时或周期性任务</span></section></td><td data-colwidth="203" width="230"><section><span leaf="">serverCron（核心周期函数）</span></section></td><td data-colwidth="157" width="223"><section data-mpa-action-id="mb69cqogxxp"><span leaf="">服务初始化时</span></section></td></tr></tbody></table>

3、关键代码执行流

a. 新建连接事件在 Redis 启动时注册，当 Redis 收到新建连接请求后，会调用 acceptTcpHandler

```
void initServer(void) {
    if (createSocketAcceptHandler(&server.ipfd, acceptTcpHandler) != C_OK) {
        serverPanic("Unrecoverable error creating TCP socket accept handler.");
    }
}
```

b. 读事件处理函数 readQueryFromClient，在新建连接时注册。写事件处理函数 sendReplyToClient 在发送执行结果时注册。

```
// 读事件处理函数。新建连接时注册
connSetReadHandler(conn, readQueryFromClient);
// 写事件处理函数。单次事件循环，无法发完数据时注册
connSetWriteHandler(c->conn, sendReplyToClient)
```

c. 在 Redis 启动后，进入事件循环 aeMain

```
void aeMain(aeEventLoop *eventLoop) {
    eventLoop->stop = 0;
    while (!eventLoop->stop) {
        // 事件循环处理函数
        // 关注读、写、时间事件
        aeProcessEvents(eventLoop, AE_ALL_EVENTS|
                                   AE_CALL_BEFORE_SLEEP|
                                   AE_CALL_AFTER_SLEEP);
    }
}
```

d. 单次事件循环 aeProcessEvents 函数简化后，执行流程如下。

```
int aeProcessEvents(aeEventLoop *eventLoop, int flags)
{
    int processed = 0, numevents;
    if (eventLoop->maxfd != -1 ||
        ((flags & AE_TIME_EVENTS) && !(flags & AE_DONT_WAIT))) {
        // 事件触发前执行函数 beforeSleep
        if (eventLoop->beforesleep != NULL && flags & AE_CALL_BEFORE_SLEEP)
            eventLoop->beforesleep(eventLoop);

        // 获取触发事件
        numevents = aeApiPoll(eventLoop, tvp);
        // 事件触发后执行函数 afterSleep
        if (eventLoop->aftersleep != NULL && flags & AE_CALL_AFTER_SLEEP)
            eventLoop->aftersleep(eventLoop);

        // 循环处理事件
        for (j = 0; j < numevents; j++) {
            aeFileEvent *fe = &eventLoop->events[eventLoop->fired[j].fd];
            // 执行读事件回调函数 rfileProc
            if (fe->mask & mask & AE_READABLE)
                fe->rfileProc(eventLoop,fd,fe->clientData,mask);
            // 执行写事件回调函数 wfileProc
            if (fe->mask & mask & AE_WRITABLE)
                fe->wfileProc(eventLoop,fd,fe->clientData,mask);
        }
    }

    // 时间事件
    if (flags & AE_TIME_EVENTS)
        processed += processTimeEvents(eventLoop);
    return processed;
}
// 其中 beforeSleep 函数。在每次事件触发前，会执行一些特定功能。
```

补充：

1.  beforeSleep 核心作用：
    
    - 将命令回复缓冲区数据写入客户端（handleClientsWithPendingWrites）。
    
    - 集群模式下发送心跳包。
    
    - Module 系统的事件钩子执行。
    
2.  serverCron 时间事件：
    
    - 每 100ms 执行一次（可配置）。
    
    - 执行过期键清理、持久化触发、主从重连、集群故障检测等。
    
3.  写事件注册策略：
    
    - 延迟注册：默认不注册写事件，仅在输出缓冲区满时注册（installClientWriteHandler）。
    
    - 一次性触发：发送完成后立即取消写事件监听，避免空转。
    
4.  多线程 IO 扩展（Redis 6.0+）：
    
    - 主线程：仍负责命令执行和事件调度。
    
    - IO 线程：在 aftersleep 阶段处理解析后的命令，通过 postponeClientRead 分流读操作。
    

了解了事件驱动后，我们现在来看，一条 Redis 命令是如何执行的。

我给出示意图，方便理解。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe96KXwx2BLy4ZO6QFkibV3VibVKyzUcmz1LwweCnqTJGkyrp6I9j0km7xHvFBiciawNDHagDgKicaH4Vvkg/640?wx_fmt=png&from=appmsg&randomid=4p62ry95)

1、 建立连接阶段：

客户端发起请求，由 Redis 事件驱动模块 ae 接收。ae 是一个基于 IO 多路复用的 while 无限循环。ae 模块在接收连接请求后，会触发「新建连接事件」，由 「acceptTcpHandler」 函数执行。该函数负责接收连接、新建连接，以及初始化 client 数据结构。

```
// 核心代码路径
aeMain() → aeProcessEvents() → acceptTcpHandler() → createClient() → connSetReadHandler(conn, readQueryFromClient)
```

2. 读 and 解析 阶段：

Redis 收到命令后，触发 ae 模块「读事件」，进入「readQueryFromClient」执行流程。该流程判断是否启用 IO 多线程，选择以下两条分支之一。

*   若启用，则主线程将该连接客户端加入「clients_pending_read」读就绪队列，并将客户端 flag 标记为「CLIENT_PENDING_READ」，表示可读。下一次循环时，会将 clients_pending_read 队列分发给 IO 线程和主线程，执行读取请求、解析命令等操作。最终，由主线程执行命令。
    
*   若未启用，则主线程「独自」执行读取命令、解析命令、执行命令、发送结果等全部流程。
    

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe96KXwx2BLy4ZO6QFkibV3VibV3BkJQqujp2g8EKozOO845X7wxFpf0U05jCicfe4qGKC5KchLibGk4ibEg/640?wx_fmt=png&from=appmsg&randomid=ygs6qtwn)

其中，解析命令流程，会解析客户端发来的请求字符串。具体为以下两个步骤。

*   找到命令对应的执行函数，放到 client->cmd->proc 中。
    
*   解析参数，放到 client->argv、client->argc 中。
    

Redis 所有命令的执行函数，保存在「redisCommandTable」中。SET 命令对应为「setCommand」。

```
struct redisCommand redisCommandTable[] = {
    ...
    {"set",setCommand,-3,
     "write use-memory @string",
     0,NULL,1,1,1,0,0,0},
    ...
}
```

前面我们提到，每次事件循环，Redis 会执行预处理函数「beforeSleep」，该函数内会将 clients_pending_read 读就绪队列进行分发。具体调用函数如下：

```
int handleClientsWithPendingReadsUsingThreads(void) {
    // 未开启 IO 线程，直接返回
    if (!server.io_threads_active || !server.io_threads_do_reads) return 0;
    ...
    // 否则，分发「读」就绪队列到线程私有队列 io_threads_list[target_id] 中
    while((ln = listNext(&li))) {
        client *c = listNodeValue(ln);
        int target_id = item_id % server.io_threads_num;
        listAddNodeTail(io_threads_list[target_id],c);
        item_id++;
    }
    ...
    // 主线程执行 io_threads_list[0] 任务
    listRewind(io_threads_list[0],&li);
    while((ln = listNext(&li))) {
        client *c = listNodeValue(ln);
        readQueryFromClient(c->conn);
    }
    listEmpty(io_threads_list[0]);
    // 主线程等待其它 IO 线程执行任务
    while(1) {
        unsigned long pending = 0;
        for (int j = 1; j < server.io_threads_num; j++)
            pending += getIOPendingCount(j);
        if (pending == 0) break;
    }
    while(listLength(server.clients_pending_read)) {
        ...
        // 主线程，执行命令（已读取完成，解析好的命令）。
        if (processPendingCommandsAndResetClient(c) == C_ERR) {
            continue;
        }
        ...
    }
    return processed;
}
```

该函数遍历 clients_pending_read 「读」就绪队列，将「读」任务分发给 IO 线程和主线程的任务队列「io_threads_list」。收到任务后，IO 线程和主线程进入「readQueryFromClient」执行流程。注意，本次执行 readQueryFromClient 前，client 状态已被设置为 「CLIENT_PENDING_READ」 ，所以执行时，client 不会再次加入任务队列，而是进入真正的执行流程。

3、 执行命令阶段

```
int processPendingCommandsAndResetClient(client *c)
processPendingCommandsAndResetClient()
├── processCommand()          // 命令校验（权限/内存/集群等）
├── call()                    // 执行命令前钩子（monitor/watch）
├── c->cmd->proc(c)          // 实际执行命令（如setCommand）
├── propagate()              // 主从复制/AOF传播
└── addReply()               // 响应处理
```

其中，c->cmd->proc 用来执行真正的命令 setCommand。

执行完命令后，主线程进入最后一步「addReply」，调用 prepareClientToWrite，将执行结果，加入 「clients_pending_write」 写就绪队列中，等待返回客户端。

```
void addReply(client *c, robj *obj) {
    // 加入 clients_pending_write 写就绪队列
    if (prepareClientToWrite(c) != C_OK) return;
    ...
}
```

在进入下一次事件循环时，beforeSleep 函数，将 clients_pending_write 写就绪队列，分发给 IO 线程和主线程。执行函数如下：

```
int handleClientsWithPendingWritesUsingThreads(void) {
    // 如果开启 IO 线程或者客户端连接很少
    // 主线程直接同步发送结果
    if (server.io_threads_num == 1 || stopThreadedIOIfNeeded()) {
        return handleClientsWithPendingWrites();
    }

    ...
    // 否则，分发 clients_pending_write 给 IO 线程和主线程执行
    while((ln = listNext(&li))) {
        int target_id = item_id % server.io_threads_num;
        // 添加到线程任务队列
        listAddNodeTail(io_threads_list[target_id],c);
        item_id++;
    }
    ...
    // 主线程处理分配给自己的任务，这里是同步执行
    listRewind(io_threads_list[0],&li);
    while((ln = listNext(&li))) {
        client *c = listNodeValue(ln);
        // 直接发送给客户端
        writeToClient(c,0);
    }

    // 等待 IO 线程执行完毕
    while(1) {
        unsigned long pending = 0;
        for (int j = 1; j < server.io_threads_num; j++)
            pending += getIOPendingCount(j);
        if (pending == 0) break;
    }
    // 如果同步写数据，没有写完，则注册写事件
    // 在下一次事件循环中触发
    listRewind(server.clients_pending_write,&li);
    while((ln = listNext(&li))) {
        client *c = listNodeValue(ln);
        // 注册写事件
        if (clientHasPendingReplies(c) &&
                connSetWriteHandler(c->conn, sendReplyToClient) == AE_ERR)
        {
            freeClientAsync(c);
        }
    }
    listEmpty(server.clients_pending_write);
}
```

4、响应发送阶段

最终，IO 线程和主线程，通过 writeToClient 函数，将命令执行结果发送给客户端。

-End-

原创作者｜唐浩雲

感谢你读到这里，不如关注一下？👇

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95UnhD9f7ia4T3ufXM1liaxxffiaEy41n0icohEC2qDS05icapaN4iaTVfsClibPRmqOjNW6q33PZicAVoSOg/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp&randomid=h7bt0wtf)

📢📢来领开发者专属福利！点击下方图片直达👇  

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe96fpHIJvruyicw5Q9fM7qjaupkTNfa2IDoe6Jm7TqVwngJS9TkMIMXhTvyTiaLr7OVTicXP3bDU6utHg/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp&randomid=d5f9kycg)

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe975eiakGydXqTICibuXvLhyqN5sicc7ia7Cvb8nJGK2gjavrfIIYr5oicm20W8hFPvUdSm8UTzzWiaFco9Q/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp&randomid=ujf4upse)

  
文章中提到的 Redis 的几种部署架构，你在工作中用到过哪一种？欢迎评论留言补充。我们将选取 1 则优质的评论，送出腾讯云定制文件袋套装 1 个（见下图）。7 月 3 日中午 12 点开奖。  

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe96Ad6VYX3tia1sGJkFMibI6902he72w3I4NqAf7H4Qx1zKv1zA4hGdpxicibSono28YAsjFbSalxRADBg/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp&randomid=53yptxwg)

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe979Bb4KNoEWxibDp8V9LPhyjmg15G7AJUBPjic4zgPw1IDPaOHDQqDNbBsWOSBqtgpeC2dvoO9EdZBQ/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp&randomid=c2uh5qgb)

[

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95YfT6jRntOjzVAiaM05yMozOqLgIAJFEmJKSRePvFxXAWM5Fuib0cyRebVBcO4GmMDuGsRjKWW4Ziaw/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp&randomid=xsdxq8z6)

](https://mp.weixin.qq.com/s?__biz=MzI2NDU4OTExOQ==&mid=2247689809&idx=1&sn=f01be1c6883682be79ac68cff76fdaf3&scene=21#wechat_redirect)

[

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95YfT6jRntOjzVAiaM05yMozo4LiaIAhpn8qLfwtXPvWr5vIcvAeT7Nwg2THvXbFxYnDY9tn2emmoww/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp&randomid=0et46djx)

](https://mp.weixin.qq.com/s?__biz=MzI2NDU4OTExOQ==&mid=2247689738&idx=1&sn=4b711afe876c8221d91eacd100e0f32a&scene=21#wechat_redirect)

[

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95YfT6jRntOjzVAiaM05yMozPR17ZYOzpnx46wu8DosqtwW7Bqp9enhVt8AEmbe0Jx6tZORUUTTHhg/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp&randomid=x9les572)

](https://mp.weixin.qq.com/s?__biz=MzI2NDU4OTExOQ==&mid=2247690045&idx=1&sn=57cddf2d9128ad2290191407027a0562&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95pIHzoPYoZUNPtqXgYG2leyAEPyBgtFj1bicKH2q8vBHl26kibm7XraVgicePtlYEiat23Y5uV7lcAIA/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp&randomid=2lznsb26)