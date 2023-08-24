#多线程 #源码阅读

# 1. Condition 类图

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20220501154209.png)

Condition 对象有 await，signal 等方法。Object 有 wait，notify 等方法。这两者之间区别如下：
1.  Condition 支持不响应中断
2.  Condition 可以支持多个等待队列，new 多个 Condition 即可 (即多种等待条件，而 wait 只能支持一种)

如上图可见，Condition 常用于 juc 包中各个容器的实现。对于 BlockingQueue 就是依赖于 Condition 实现的，因此核心逻辑就在于看队列容量决定 await 和 signal 的调用实际，对于 BlockingQueue 这部分的流程分析后续有空再绘图表示。

# 2. Condition 的流程分析

![Condition流程.drawio.svg](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202308250211226.svg)

# 3. Condition 疑问记录

## 3.1. Condition 何时唤醒阻塞中的线程

当其它线程调用 signal 方法后，会从 Condition 等待队列中将节点移动到 AQS 等待队列，从而进行锁的抢占。抢占到锁的线程继续按原有的代码执行，未抢占到的线程继续阻塞，等待其它线程释放锁。
**这里需要注意的一点是别忽略 Condition 是仅有一个线程可以进入的！**就算 Condition 调用了 `signalAll();` 语句，也只是意味着线程可以都开始抢占锁，但是还是需要在 AQS 阻塞队列中按 FIFO 特性进行获取锁的。
也就是意味着 `Signal()` 仅仅意味着线程可以开始抢占锁了，但是能不能够抢占到还是需要看锁获取情况的。

## 3.2. Condition 中何时会抛出 InterruptedException 异常？

当线程被唤醒检查中断标记位时，若发现当前线程被中断过且状态仍为 CONDITION，说明还未进入 AQS 队列就被异常唤醒了，此时会标记成 THROW_IE 状态，后续会抛出InterruptedException 异常。
