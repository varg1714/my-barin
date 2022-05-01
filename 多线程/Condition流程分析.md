#多线程 

# Condition 类图

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20220501154209.png)

Condition 对象有 await，signal 等方法。Object 有 wait，notify 等方法。这两者之间区别如下：
1.  Condition 支持不响应中断
2.  Condition 可以支持多个等待队列，new 多个 Condition 即可 (即多种等待条件，而 wait 只能支持一种)

如上图可见，Condition 常用于 juc 包中各个容器的实现。

# Condition 的流程分析

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20220501154420.png)

# Condition 疑问记录

## Condition 何时唤醒阻塞中的线程

当其它线程调用 signal 方法后，会从 Condition 等待队列中将节点移动到 AQS 等待队列，从而进行锁的抢占。

## Condition 中何时会抛出 InterruptedException 异常？

当线程被唤醒检查中断标记位时，若发现当前线程被中断过且状态仍为 CONDITION，说明还未进入 AQS 队列就被异常唤醒了，此时会标记成 THROW_IE 状态，后续会抛出InterruptedException 异常。
