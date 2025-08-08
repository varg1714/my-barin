---
source: https://mp.weixin.qq.com/s/Fqn-797GpsSKdVztzthSOg
create: 2025-08-04 22:48
read: true
knowledge: true
tags:
  - Redis
---
![](http://mmbiz.qpic.cn/mmbiz_gif/AAQtmjCc74DZeqm2Rc4qc7ocVLZVd8FOASKicbMfKsaziasqIDXGPt8yR8anxPO3NCF4a4DkYCACam4oNAOBmSbA/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&randomid=8z1rs68h)

**目录**

一、背景

二、Redis6.0 多线程 IO 概述

 1.  参数与配置

 2. 执行流程概述

三、源码分析

 1.  初始化

 2. 读数据流程

 3. 写数据流程

 4. 多线程 IO 动态暂停与开启

四、性能对比

 1.  测试环境

 2. Redis 版本

 3. 压测命令

 4. 统计结果

 5. 结论

五、6.0 多线程 IO 不足

六、总结

**一**

 **背景**

使用过 Redis 的同学肯定都了解过一个说法，说 Redis 是单线程模型，那么实际情况是怎样的呢？

其实，我们常说 Redis 是单线程模型，**是指 Redis 采用单线程的事件驱动模型，只有并且只会在一个主线程中执行 Redis 命令操作**，这意味着它在处理请求时不使用复杂的上下文切换或锁机制。尽管只是单线程的架构，但 Redis 通过非阻塞的 I/O 操作和高效的事件循环来处理大量的并发连接，性能仍然非常高。

然而在 Redis4.0 开始也引入了一些后台线程执行异步淘汰、异步删除过期 key、异步执行大 key 删除等任务，然后，在 Redis6.0 中引入了多线程 IO 特性，将 Redis 单节点访问请求从 10W 提升到 20W。

而在去年 Valkey 社区发布的 Valkey8.0 版本，在 I/O 线程系统上进行了重大升级，特别是异步 I/O 线程的引入，使主线程和 I/O 线程能够并行工作，可实现最大化服务吞吐量并减少瓶颈，使得 Valkey 单节点访问请求可以提升到 100W。

**那么在 Redis6.0 和 Valkey8.0 中多线程 IO 是怎么回事呢？是否改变了 Redis 原有单线程模型？**

*   2024 年，Redis 商业支持公司 Redis Labs 宣布 Redis 核心代码的许可证从 BSD 变更为 RSALv2，明确禁止云厂商提供 Redis 托管服务，这一决定直接导致社区分裂。
    
*   为维护开源自由，Linux 基金会联合多家科技公司（包括 AWS、Google、Cloud、Oracle 等）宣布支持 Valkey，作为 Redis 的替代分支。
    
*   Valkey8.0 系 Valkey 社区发布的首个主要大版本。
    
*   最新消息，在 Redis 项目创始人 antirez 今年加入 Redis 商业公司 5 个月后，Redis 宣传从 Redis8 开始，Redis 项目重新开源。
    

本篇文章主要介绍 Redis6.0 多线程 IO 特性。

**二**

 **Redis6.0 多线程 IO 概述**

Redis6.0 引入多线程 IO，但多线程部分只是用来处理网络数据的读写和协议解析，执行命令仍然是单线程。默认是不开启的，需要进程启动前开启配置，并且在运行期间无法通过 config set 命令动态修改。

**参数与配置**

多线程 IO 涉及下面两个配置参数：

```
# io-threads 4  IO 线程数量
# io-threads-do-reads no  读数据及数据解析是否也用 IO 线程
```

*    io-threads 表示 IO 线程数量， io-threads 设置为 1 时（代码中默认值），表示只使用主线程，不开启多线程 IO。因此，若要配置开启多线程 IO，需要设置 io-threads 大于 1，但不可以超过最大值 128。
    
*   但在默认情况下，Redis 只将多线程 IO 用于向客户端写数据，因为作者认为通常使用多线程执行读数据的操作帮助不是很大。如果需要使用多线程用于读数据和解析数据，则需要将参数 io-threads-do-reads 设置为 yes 。
    
*   此两项配置**参数在 Redis 运行期间无法通过 config set 命令修改，并且开启 SSL 时，不支持多线程 IO 特性。**
    
*   若机器 CPU 将至少超过 4 核时，则建议开启，并且至少保留一个备用 CPU 核，使用超过 8 个线程可能并不会有多少帮助。
    

**执行流程概述**

Redis6.0 引入多线程 IO 后，读写数据执行流程如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AWMkTUB1CTPH37fRAaSojQJ3LrFErPacbkWcdQqgLjhcr3d2vshM1jdjEqHQiaiblnTbCpLXGElib9w/640?wx_fmt=png&from=appmsg&randomid=tudha79d)

**流程简述**

1.  主线程负责接收建立连接请求，获取 socket 放入全局等待读处理队列。
    
2.  主线程处理完读事件之后，通过 RR（Round Robin）将这些连接分配给这些 IO 线程，也会分配给主线程自己。
    
3.  主线程先读取分配给自己的客户端数据，然后阻塞等待其他 IO 线程读取 socket 完毕。
    
4.  IO 线程将请求数据读取并解析完成（这里只是读数据和解析、并不执行）。
    
5.  主线程通过单线程的方式执行请求命令。
    
6.  主线程通过 RR（Round Robin）将回写客户端事件分配给这些 IO 线程，也会分配给主线程自己。
    
7.  主线程同样执行部分写数据到客户端，然后阻塞等待 IO 线程将数据回写 socket 完毕。
    

**设计特点**

1.  IO 线程要么同时在读 socket，要么同时在写，不会同时读和写。
    
2.  IO 线程只负责读写 socket 解析命令，不负责命令执行。
    
3.  主线程也会参与数据的读写。
    

**三**

 **源码分析**

多线程 IO 相关源代码都在源文件 networking.c 中最下面。

**初始化**

主线程在 main 函数中调用 InitServerLast 函数，InitServerLast 函数中调用 initThreadedIO 函数，在 initThreadedIO 函数中根据配置文件中的线程数量，创建对应数量的 IO 工作线程数量。

```c
/* Initialize the data structures needed for threaded I/O. */
void initThreadedIO(void) {
    io_threads_active = 0; /* We start with threads not active. */
    
    /* Don't spawn any thread if the user selected a single thread:
     * we'll handle I/O directly from the main thread. */
    if (server.io_threads_num == 1) return;
    
    if (server.io_threads_num > IO_THREADS_MAX_NUM) {
        serverLog(LL_WARNING,"Fatal: too many I/O threads configured. "
                             "The maximum number is %d.", IO_THREADS_MAX_NUM);
        exit(1);
    }
   
    /* Spawn and initialize the I/O threads. */
    for (int i = 0; i < server.io_threads_num; i++) {
        /* Things we do for all the threads including the main thread. */
        io_threads_list[i] = listCreate();
        if (i == 0) continue; /* Thread 0 is the main thread. */
       
        /* Things we do only for the additional threads. */
        pthread_t tid;
        pthread_mutex_init(&io_threads_mutex[i],NULL);
        io_threads_pending[i] = 0;
        pthread_mutex_lock(&io_threads_mutex[i]); /* Thread will be stopped. */
        if (pthread_create(&tid,NULL,IOThreadMain,(void*)(long)i) != 0) {
            serverLog(LL_WARNING,"Fatal: Can't initialize IO thread.");
            exit(1);
        }
        io_threads[i] = tid;
    }
}
```

*   如果 io_threads_num 的数量为 1，则只运行主线程， io_threads_num 的 IO 线程数量不允许超过 128。
    
*   序号为 0 的线程是主线程，因此实际的工作线程数目是 io-threads - 1。
    

**初始化流程**

*   为包括主线程在内的每个线程分配 list 列表，用于后续保存待处理的客户端。
    
*   为主线程以外的其他 IO 线程初始化互斥对象 mutex，但是立即调用 pthread_mutex_lock 占有互斥量，将 io_threads_pending[i] 设置为 0，接着创建对应的 IO 工作线程。
    
*   占用互斥量是为了创建 IO 工作线程后，可暂时等待后续启动 IO 线程的工作，因为 IOThreadMain 函数在 io_threads_pending[id] == 0 时也调用了获取 mutex，所以此时无法继续向下运行，等待启动。
    
*   在 startThreadedIO 函数中会释放 mutex 来启动 IO 线程工作。何时调用 startThreadedIO 打开多线程 IO，具体见下文的「多线程 IO 动态暂停与开启」。
    

**IO 线程主函数**

IO 线程主函数代码如下所示：

```c
void *IOThreadMain(void *myid) {
    /* The ID is the thread number (from 0 to server.iothreads_num-1), and is
     * used by the thread to just manipulate a single sub-array of clients. */
    long id = (unsigned long)myid;
    char thdname[16];
   
    snprintf(thdname, sizeof(thdname), "io_thd_%ld", id);
    redis_set_thread_title(thdname);
    redisSetCpuAffinity(server.server_cpulist);
   
    while(1) {
        /* Wait for start */
        for (int j = 0; j < 1000000; j++) {
            if (io_threads_pending[id] != 0) break;
        }
       
        /* Give the main thread a chance to stop this thread. */
        if (io_threads_pending[id] == 0) {
            pthread_mutex_lock(&io_threads_mutex[id]);
            pthread_mutex_unlock(&io_threads_mutex[id]);
            continue;
        }
       
        serverAssert(io_threads_pending[id] != 0);
        
        if (tio_debug) printf("[%ld] %d to handle\n", id, (int)listLength(io_threads_list[id]));
        
        /* Process: note that the main thread will never touch our list
         * before we drop the pending count to 0. */
        listIter li;
        listNode *ln;
        listRewind(io_threads_list[id],&li);
        while((ln = listNext(&li))) {
            client *c = listNodeValue(ln);
            if (io_threads_op == IO_THREADS_OP_WRITE) {
                writeToClient(c,0);
            } else if (io_threads_op == IO_THREADS_OP_READ) {
                readQueryFromClient(c->conn);
            } else {
                serverPanic("io_threads_op value is unknown");
            }
        }
        listEmpty(io_threads_list[id]);
        io_threads_pending[id] = 0;
       
        if (tio_debug) printf("[%ld] Done\n", id);
    }
}
```

从 IO 线程主函数逻辑可以看到：

*   如果 IO 线程等待处理任务数量为 0，则 IO 线程一直在空循环，因此后面主线程给 IO 线程分发任务后，需要设置 IO 线程待处理任务数 io_threads_pending[id] ，才会触发 IO 线程工作。
    
*   如果 IO 线程等待处理任务数量为 0，并且未获取到 mutex 锁，则会等待获取锁，暂停运行，由于主线程在创建 IO 线程之前先获取了锁，因此 IO 线程刚启动时是暂停运行状态，需要等待主线程释放锁，启动 IO 线程。
    
*   IO 线程待处理任务数为 0 时，获取到锁并再次释放锁，是为了让主线程可以暂停 IO 线程。
    
*   只有 io_threads_pending[id] 不为 0 时，则继续向下执行操作，根据 io_threads_op 决定是读客户端还是写客户端，从这里也可以看出 IO 线程要么同时读，要么同时写。
    

**读数据流程**

**主线程将待读数据客户端加入队列**

当客户端连接有读事件时，会触发调用 readQueryFromClient 函数，在该函数中会调用 postponeClientRead。

```c
void readQueryFromClient(connection *conn) {
    client *c = connGetPrivateData(conn);
    int nread, readlen;
    size_t qblen;
    
    /* Check if we want to read from the client later when exiting from
     * the event loop. This is the case if threaded I/O is enabled. */
    if (postponeClientRead(c)) return;
    ......以下省略
}

/* Return 1 if we want to handle the client read later using threaded I/O.
 * This is called by the readable handler of the event loop.
 * As a side effect of calling this function the client is put in the
 * pending read clients and flagged as such. */
int postponeClientRead(client *c) {
    if (io_threads_active &&
        server.io_threads_do_reads &&
        !ProcessingEventsWhileBlocked &&
        !(c->flags & (CLIENT_MASTER|CLIENT_SLAVE|CLIENT_PENDING_READ)))
    {
        c->flags |= CLIENT_PENDING_READ;
        listAddNodeHead(server.clients_pending_read,c);
        return 1;
    } else {
        return 0;
    }
}
```

如果开启多线程，并且开启多线程读（io_threads_do_reads 为 yes），则将客户端标记为 CLIENT_PENDING_READ，并且加入 clients_pending_read 列表。

然后 readQueryFromClient 函数中就立即返回，主线程没有执行从客户端连接中读取的数据相关逻辑，读取了客户端数据行为等待后续各个 IO 线程执行。

**主线程分发并阻塞等待**

主线程在 beforeSleep 函数中会调用 handleClientsWithPendingReadsUsingThreads 函数。

```c
/* When threaded I/O is also enabled for the reading + parsing side, the
 * readable handler will just put normal clients into a queue of clients to
 * process (instead of serving them synchronously). This function runs
 * the queue using the I/O threads, and process them in order to accumulate
 * the reads in the buffers, and also parse the first command available
 * rendering it in the client structures. */
int handleClientsWithPendingReadsUsingThreads(void) {
    if (!io_threads_active || !server.io_threads_do_reads) return 0;
    int processed = listLength(server.clients_pending_read);
    if (processed == 0) return 0;
   
    if (tio_debug) printf("%d TOTAL READ pending clients\n", processed);
   
    /* Distribute the clients across N different lists. */
    listIter li;
    listNode *ln;
    listRewind(server.clients_pending_read,&li);
    int item_id = 0;
    while((ln = listNext(&li))) {
        client *c = listNodeValue(ln);
        int target_id = item_id % server.io_threads_num;
        listAddNodeTail(io_threads_list[target_id],c);
        item_id++;
    }
    
    /* Give the start condition to the waiting threads, by setting the
     * start condition atomic var. */
    io_threads_op = IO_THREADS_OP_READ;
    for (int j = 1; j < server.io_threads_num; j++) {
        int count = listLength(io_threads_list[j]);
        io_threads_pending[j] = count;
    }
   
    /* Also use the main thread to process a slice of clients. */
    listRewind(io_threads_list[0],&li);
    while((ln = listNext(&li))) {
        client *c = listNodeValue(ln);
        readQueryFromClient(c->conn);
    }
    listEmpty(io_threads_list[0]);
    
    /* Wait for all the other threads to end their work. */
    while(1) {
        unsigned long pending = 0;
        for (int j = 1; j < server.io_threads_num; j++)
            pending += io_threads_pending[j];
        if (pending == 0) break;
    }
    if (tio_debug) printf("I/O READ All threads finshed\n");
   
    /* Run the list of clients again to process the new buffers. */
    while(listLength(server.clients_pending_read)) {
        ln = listFirst(server.clients_pending_read);
        client *c = listNodeValue(ln);
        c->flags &= ~CLIENT_PENDING_READ;
        listDelNode(server.clients_pending_read,ln);
        
        if (c->flags & CLIENT_PENDING_COMMAND) {
            c->flags &= ~CLIENT_PENDING_COMMAND;
            if (processCommandAndResetClient(c) == C_ERR) {
                /* If the client is no longer valid, we avoid
                 * processing the client later. So we just go
                 * to the next. */
                continue;
            }
        }
        processInputBuffer(c);
    }
    return processed;
}
```

*   先检查是否开启多线程，以及是否开启多线程读数据（io_threads_do_reads），未开启直接返回。
    
*   检查队列 clients_pending_read 长度，为 0 直接返回，说明没有待读事件。
    
*   遍历 clients_pending_read 队列，通过 RR 算法，将队列中的客户端循环分配给各个 IO 线程，包括主线程本身。
    
*   设置 io_threads_op = IO_THREADS_OP_READ，并且将 io_threads_pending 数组中各个位置值设置为对应各个 IO 线程分配到的客户端数量，如上面介绍，目的是为了使 IO 线程工作。
    
*   主线程开始读取客户端数据，因为主线程也分配了任务。
    
*   主线程阻塞等待，直到所有的 IO 线程都完成读数据工作。
    
*   主线程执行命令。
    

**IO 线程读数据**

在 IO 线程主函数中，如果 io_threads_op == IO_THREADS_OP_READ ，则调用 readQueryFromClient 从网络中读取数据。

**IO 线程读取数据后，不会执行命令。**

在 readQueryFromClient 函数中，最后会执行 processInputBuffer 函数，在 processInputBuffe 函数中，如 IO 线程检查到客户端设置了 CLIENT_PENDING_READ 标志，则不执行命令，直接返回。

```c
            ......省略
/* If we are in the context of an I/O thread, we can't really
             * execute the command here. All we can do is to flag the client
             * as one that needs to process the command. */
            if (c->flags & CLIENT_PENDING_READ) {
                c->flags |= CLIENT_PENDING_COMMAND;
                break;
            }
            ...... 省略
```

  

**写数据流程**

命令处理完成后，依次调用：  

addReply-->prepareClientToWrite-->clientInstallWriteHandler，将待写客户端加入队列 clients_pending_write。

```c
void clientInstallWriteHandler(client *c) {
    /* Schedule the client to write the output buffers to the socket only
     * if not already done and, for slaves, if the slave can actually receive
     * writes at this stage. */
    if (!(c->flags & CLIENT_PENDING_WRITE) &&
        (c->replstate == REPL_STATE_NONE ||
         (c->replstate == SLAVE_STATE_ONLINE && !c->repl_put_online_on_ack)))
    {
        /* Here instead of installing the write handler, we just flag the
         * client and put it into a list of clients that have something
         * to write to the socket. This way before re-entering the event
         * loop, we can try to directly write to the client sockets avoiding
         * a system call. We'll only really install the write handler if
         * we'll not be able to write the whole reply at once. */
        c->flags |= CLIENT_PENDING_WRITE;
        listAddNodeHead(server.clients_pending_write,c);
    }
}
```

在 beforeSleep 函数中调用 handleClientsWithPendingWritesUsingThreads。

```c
int handleClientsWithPendingWritesUsingThreads(void) {
    int processed = listLength(server.clients_pending_write);
    if (processed == 0) return 0; /* Return ASAP if there are no clients. */
    
    /* If I/O threads are disabled or we have few clients to serve, don't
     * use I/O threads, but thejboring synchronous code. */
    if (server.io_threads_num == 1 || stopThreadedIOIfNeeded()) {
        return handleClientsWithPendingWrites();
    }
    
    /* Start threads if needed. */
    if (!io_threads_active) startThreadedIO();
   
    if (tio_debug) printf("%d TOTAL WRITE pending clients\n", processed);
    
    /* Distribute the clients across N different lists. */
    listIter li;
    listNode *ln;
    listRewind(server.clients_pending_write,&li);
    int item_id = 0;
    while((ln = listNext(&li))) {
        client *c = listNodeValue(ln);
        c->flags &= ~CLIENT_PENDING_WRITE;
        int target_id = item_id % server.io_threads_num;
        listAddNodeTail(io_threads_list[target_id],c);
        item_id++;
    }
   
    /* Give the start condition to the waiting threads, by setting the
     * start condition atomic var. */
    io_threads_op = IO_THREADS_OP_WRITE;
    for (int j = 1; j < server.io_threads_num; j++) {
        int count = listLength(io_threads_list[j]);
        io_threads_pending[j] = count;
    }
    
    /* Also use the main thread to process a slice of clients. */
    listRewind(io_threads_list[0],&li);
    while((ln = listNext(&li))) {
        client *c = listNodeValue(ln);
        writeToClient(c,0);
    }
    listEmpty(io_threads_list[0]);
   
    /* Wait for all the other threads to end their work. */
    while(1) {
        unsigned long pending = 0;
        for (int j = 1; j < server.io_threads_num; j++)
            pending += io_threads_pending[j];
        if (pending == 0) break;
    }
    if (tio_debug) printf("I/O WRITE All threads finshed\n");
    
    /* Run the list of clients again to install the write handler where
     * needed. */
    listRewind(server.clients_pending_write,&li);
    while((ln = listNext(&li))) {
        client *c = listNodeValue(ln);
       
        /* Install the write handler if there are pending writes in some
         * of the clients. */
        if (clientHasPendingReplies(c) &&
                connSetWriteHandler(c->conn, sendReplyToClient) == AE_ERR)
        {
            freeClientAsync(c);
        }
    }
    listEmpty(server.clients_pending_write);
    return processed;
}
```

1.  判断 clients_pending_write 队列的长度，如果为 0 则直接返回。
    
2.  判断是否开启了多线程，若只有很少的客户端需要写，则不使用多线程 IO，直接在主线程完成写操作。
    
3.  如果使用多线程 IO 来完成写数据，则需要判断是否先开启多线程 IO（因为会动态开启与暂停）。
    
4.  遍历 clients_pending_write 队列，通过 RR 算法，循环将所有客户端分配给各个 IO 线程，包括主线程自身。
    
5.  设置 io_threads_op = IO_THREADS_OP_WRITE，并且将 io_threads_pending 数组中各个位置值设置为对应的各个 IO 线程分配到的客户端数量，目的是为了使 IO 线程工作。
    
6.  主线程开始写客户端数据，因为主线程也分配了任务，写完清空任务队列。
    
7.  阻塞等待，直到所有 IO 线程完成写数据工作。
    
8.  再次遍历所有客户端，如果有需要，为客户端在事件循环上安装写句柄函数，等待事件回调。
    

**多线程 IO 动态暂停与开启**

从上面的写数据的流程中可以看到，在 Redis 运行过程中多线程 IO 是会动态暂停与开启的。

在上面的写数据流程中，先调用 stopThreadedIOIfNeeded 函数判断是否需要暂停多线程 IO，当**等待写的客户端数量低于线程数的 2 倍时，会暂停多线程 IO，**否则就会打开多线程。

```c
int stopThreadedIOIfNeeded(void) {
    int pending = listLength(server.clients_pending_write);
    
    /* Return ASAP if IO threads are disabled (single threaded mode). */
    if (server.io_threads_num == 1) return 1;
   
    if (pending < (server.io_threads_num*2)) {
        if (io_threads_active) stopThreadedIO();
        return 1;
    } else {
        return 0;
    }
}
```

在写数据流程 handleClientsWithPendingWritesUsingThreads 函数中，stopThreadedIOIfNeeded 返回 0 的话，就会执行下面的 startThreadedIO 函数，开启多线程 IO。

```c
void startThreadedIO(void) {
    serverAssert(server.io_threads_active == 0);
    for (int j = 1; j < server.io_threads_num; j++)
        pthread_mutex_unlock(&io_threads_mutex[j]);
    server.io_threads_active = 1;
}

void stopThreadedIO(void) {
    /* We may have still clients with pending reads when this function
     * is called: handle them before stopping the threads. */
    handleClientsWithPendingReadsUsingThreads();
    serverAssert(server.io_threads_active == 1);
    for (int j = 1; j < server.io_threads_num; j++)
        pthread_mutex_lock(&io_threads_mutex[j]);
    server.io_threads_active = 0;
}
```

从上面的代码中可以看出：

*   开启多线程 IO 是通过释放 mutex 锁来让 IO 线程开始执行读数据或者写数据动作。
    
*   暂停多线程 IO 则是通过加锁来让 IO 线程暂时不执行读数据或者写数据动作，此处加锁后，IO 线程主函数由于无法获取到锁，因此会暂时阻塞。
    

**四**

 **性能对比**

**测试环境**

两台物理机配置：CentOS Linux release 7.3.1611(Core) ，12 核 CPU1.5GHz，256G 内存（free 128G）。

**Redis 版本**

使用 Redis6.0.6，多线程 IO 模式使用线程数量为 4，即 io-threads 4 ，参数 io-threads-do-reads 分别设置为 no 和 yes ，进行对比测试。

**压测命令**

  

  

  

  

  

  

  

  

  

  

  

  

  

  

```c
redis-benchmark -h 172.xx.xx.xx -t set,get -n 1000000 -r 100000000 --threads ${threadsize} -d ${datasize} -c ${clientsize}

单线程 threadsize 为 1，多线程 threadsize 为 4
datasize为value 大小，分别设置为 128/512/1024
clientsize 为客户端数量，分别设置为 256/2000
如：./redis-benchmark -h 172.xx.xx.xx -t set,get -n 1000000 -r 100000000 --threads 4 -d 1024 -c 256
```

  

  

  

  

  

  

  

  

  

  

**统计结果**

当 io-threads-do-reads 为 no 时，统计图表如下所示（c 2000 表示客户端数量为 2000）。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AWMkTUB1CTPH37fRAaSojQNzfFIYoLcibWeYjrc8ibTKNWftZv4HRjU1klCXE1iaHmWt3DOywciadGHw/640?wx_fmt=png&from=appmsg&randomid=idro7i0e)

当 io-threads-do-reads 为 yes 时，统计图表如下所示（c 256 表示客户端数量为 256）。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AWMkTUB1CTPH37fRAaSojQicfOK6VyGkEVVRXTQPhlLQyib2g5kVnC0xQIZiahfPdETUGDCtibuHRd9g/640?wx_fmt=png&from=appmsg&randomid=a19khifi)

**结论**

使用 redis-benchmark 做 Redis6 单线程和多线程简单 SET/GET 命令性能测试：

1.  从上面可以看到 GET/SET 命令在设置 4 个 IO 线程时，QPS 相比于大部分情况下的单线程，性能几乎是翻倍了。
    
2.  连接数越多，多线程优势越明显。
    
3.  value 值越小，多线程优势越明显。
    
4.  使用多线程读命令比写命令优势更加明显，当 value 越大，写命令越发没有明显的优势。
    
5.  参数 io-threads-do-reads 为 yes，性能有微弱的优势，不是很明显。
    
6.  总体来说，以上结果基本符合预期，结果仅作参考。
    

**五**

 **6.0 多线程 IO 不足**

尽管引入多线程 IO 大幅提升了 Redis 性能，但是 Redis6.0 的多线程 IO 仍然存在一些不足：

*   CPU 核心利用率不足：当前主线程仍负责大部分的 IO 相关任务，并且当主线程处理客户端的命令时，IO 线程会空闲相当长的时间，同时值得注意的是，主线程在执行 IO 相关任务期间，性能受到最慢 IO 线程速度的限制。
    
*   IO 线程执行的任务有限：目前，由于主线程同步等待 IO 线程，线程仅执行读取解析和写入操作。如果线程可以异步工作，我们可以将更多工作卸载到 IO 线程上，从而减少主线程的负载。
    
*   不支持带有 TLS 的 IO 线程。
    

最新的 Valkey8.0 版本中，通过引入异步 IO 线程，将更多的工作转移到 IO 线程执行，同时通过**批量预读取内存数据**减少内存访问延迟，大幅提高 Valkey 单节点访问 QPS，单个实例每秒可处理 100 万个请求。我们后续再详细介绍 Valkey8.0 异步 IO 特性。

**六**

 **总结**

Redis6.0 引入多线程 IO，但多线程部分只是用来处理网络数据的读写和协议解析，**执行命令仍然是单线程**。通过开启多线程 IO，并设置合适的 CPU 数量，可以提升访问请求一倍以上。

Redis6.0 多线程 IO 仍然存在一些不足，没有充分利用 CPU 核心，在最新的 Valkey8.0 版本中，引入异步 IO 将进一步大幅提升 Valkey 性能。

**往期回顾**

1. [得物社区活动：组件化的演进与实践](https://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247540215&idx=1&sn=72a0573520a8032d33b622f25bdd0671&scene=21#wechat_redirect)

2. [从 CPU 冒烟到丝滑体验：算法 SRE 性能优化实战全揭秘｜得物技术](https://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247540022&idx=1&sn=4d3ae7c51496890132816285d4754551&scene=21#wechat_redirect)

3.[CSS 闯关指南：从手写地狱到 “类” 积木之旅｜得物技术](https://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247539650&idx=1&sn=acd5ac023cac17e50bc1a46355e0eac5&scene=21#wechat_redirect)

4. [以细节诠释专业，用成长定义价值——对话 @孟同学 ｜得物技术](https://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247539238&idx=1&sn=4fcfbfecbafe02c10172b7832416465c&scene=21#wechat_redirect)

5. [大语言模型的训练后量化算法综述 | 得物技术](https://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247538986&idx=1&sn=b6b82a790a3c696bce27704472e799b2&scene=21#wechat_redirect)

文 / 竹径

关注得物技术，每周一、三更新技术干货

要是觉得文章对你有帮助的话，欢迎评论转发点赞～

未经得物技术许可严禁转载，否则依法追究法律责任。

“

**扫码添加小助手微信**

如有任何疑问，或想要了解更多技术资讯，请添加小助手微信：

![](https://mmbiz.qpic.cn/mmbiz_jpg/AAQtmjCc74AWMkTUB1CTPH37fRAaSojQgDIibZCoR5W5vw4QnGRz54GzncGJJKLXd5h2vzbnAUWgcsPSUtA0ndw/640?wx_fmt=jpeg&from=appmsg&randomid=b6vdea8m)