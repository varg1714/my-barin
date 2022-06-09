#多线程 #源码阅读 

# CountDownLatch 类图

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20220501190616.png)

# CountDownLatch 的流程分析

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20220609172349.svg)

**对于 Semaphore 类，其实现逻辑和 CountDownLatch 基本一致。**

# CountDownLatch 疑问记录

## CountDownLatch 是如何唤醒所有等待的线程的？

首先当 countDown () 方法调用后，会判断头节点的 waitStatus 状态，若为 SIGNAL，则认为后续有阻塞的节点，因此开始执行唤醒动作。

当头节点被唤醒后，将后一个节点置为新的头节点，同样再按上面的策略执行，直到后续所有节点被唤醒为止。在头节点唤醒下一个节点后，会将自身 waitStatus 状态改为 PROPAGATE，一个传播状态，标记已经将唤醒通知传递给后续节点了，此时唤醒操作完成。

## CountDownLatch 中唤醒时为什么以下 null 的判断会继续执行

```java

private void setHeadAndPropagate(Node node, int propagate) {
	Node h = head; // Record old head for check below
	setHead(node);
	/*
	 * Try to signal next queued node if:
	 *   Propagation was indicated by caller,
	 *     or was recorded (as h.waitStatus either before
	 *     or after setHead) by a previous operation
	 *     (note: this uses sign-check of waitStatus because
	 *      PROPAGATE status may transition to SIGNAL.)
	 * and
	 *   The next node is waiting in shared mode,
	 *     or we don't know, because it appears null
	 *
	 * The conservatism in both of these checks may cause
	 * unnecessary wake-ups, but only when there are multiple
	 * racing acquires/releases, so most need signals now or soon
	 * anyway.
	 */
	if (propagate > 0 || h == null || h.waitStatus < 0 ||
		(h = head) == null || h.waitStatus < 0) {
		Node s = node.next;
		// 这里 s == null 为何会继续往下执行？
		if (s == null || s.isShared())
			doReleaseShared();
	}
}

```

这里的 `s == nul` 的判断不是代表下一个节点为 null，而是在将节点添加到阻塞队列中还未构建完成而导致的中间状态。将节点添加到阻塞队列的代码如下：

```java

private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}

```

可以看到，在这里时先进行了 `node.prev = t;` 前驱节点的赋值，而后驱节点是通过 CAS 指令加自旋操作完成的，这不是一个原子操作。因此可能出现前驱节点已经完成赋值而后驱节点还未完成赋值的情况。因此上述的 `s == nul` 的判断是处理这种情况的。
