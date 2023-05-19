#多线程 #源码阅读 

# 1. ThreadLocal 的类关系

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20220429173759.png)

ThreadLocal 类组织关系如下，在 Thread 中引用了 ThreadLocalMap 对象，而 ThreadLocalMap 对象是在 ThreadLocal 中声明定义的。

在 ThreadLocalMap 中的 Entry 对象以弱引用的形式关联 ThreadLocal 对象：

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
	/** The value associated with this ThreadLocal. */
	Object value;

	Entry(ThreadLocal<?> k, Object v) {
		super(k);
		value = v;
	}
}
```

# 2. ThreadLocal 源码分析

## 2.1. ThreadLocal 赋值流程

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20220609172537.svg)

# 3. ThreadLocal 的疑问记录

## 3.1. ThreadLocal 中为何 Entry 对象要使用弱引用？

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20220429175922.png)

如上所示，当 ThreadLocal 存在两个引用，一个为 ThreadLocal 声明赋值时的引用，一个为 ThreadLocalMap 中的 key 引用。ThreadLocal 声明引用为强引用，若 ThreadLocalMap 中的 key 同样为强引用，那么 ThreadLocal 对象只能等待两个引用都失效后才可回收。**即意味着 ThreadLocalMap 线程不中止，该区域永远无法回收**。

线程声明周期与 ThreadLocal 对象的声明周期往往不是相同的，当线程所属线程池时此现象尤为明显。所以若不将线程中的 ThreadLocalMap 的 key 声明为弱引用，那么该 ThreadLocal 永远无法回收。当声明为弱引用时，ThreadLocal 对象无强引用指向，GC 操作发生时该对象就可以被回收了。

## 3.2. ThreadLocalMap 对象为何要在 Thread 中维护？

个人认为有以下几点原因：
1. 若维护在 ThreadLocal 对象中，则需要考虑线程并发读写的问题。
2. ThreadLocal 对象本身定义就是归属于每一个线程，因此线程私有也是合理的。