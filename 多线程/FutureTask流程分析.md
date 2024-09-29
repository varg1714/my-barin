#多线程 #源码阅读 

# 1. FutureTask 类图

![](https://r2.129870.xyz/img/20220502220541.png)

# 2. FutureTask 流程分析

## 2.1. run 方法调用流程

![[Excalidraw/多线程/FutureTask.md#^group=W4z6N-R80g68r-o81MhO4|run方法流程]]

## 2.2. get 方法调用流程

![[Excalidraw/多线程/FutureTask.md#^group=eBMxZU15QaALZYzS5Vwsc|get调用流程]]

# 3. FutureTask 疑问记录

## 3.1. FutureTask 为何又自己实现了一个阻塞队列？

FutureTask 自己实现阻塞队列，没有使用 AQS 相关的流程，也没有使用如 Condition 之类的一些辅助类，个人认为有以下原因：
- 其它组件太重了，FutureTask **只需要维持一个队列记录等待的线程即可就可**，不需要其它复杂的并发控制。Condition 需要执行的线程持有同一把锁，ReentrantLock 中的共享锁需要指定共入线程的数量，非共享锁仅一个线程可以进入，对于未知数量线程调用且无需锁控制的 FutureTask 来说是不合适的。