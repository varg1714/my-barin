#多线程 #源码阅读 

# ReentrantLock 类图

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20220430224029.png)

ReentrantLock 实现有两种，分为公平锁与非公平锁。两者的却别在于允不允许新插入的线程直接抢占锁：公平锁需要线程排队，而非公平锁则允许线程直接去抢占锁而无需判断当前队列中是否有其他线程在等待锁。

公平锁 FairSync 与非公平锁 NoFairSync 都依赖了 AbstractQueuedSynchronized 这个类，也就是我们常说的 AQS。

# ReentrantLock 执行流程分析

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20220501130609.png)

# ReentrartLock 疑问记录

## ReentrartLock 中使用的 Node 节点各个状态值作用

```java

static final class Node {
	/** Marker to indicate a node is waiting in shared mode */
	static final Node SHARED = new Node();
	/** Marker to indicate a node is waiting in exclusive mode */
	static final Node EXCLUSIVE = null;

	/** waitStatus value to indicate thread has cancelled */
	static final int CANCELLED =  1;
	/** waitStatus value to indicate successor's thread needs unparking */
	static final int SIGNAL    = -1;
	/** waitStatus value to indicate thread is waiting on condition */
	static final int CONDITION = -2;
	/**
	 * waitStatus value to indicate the next acquireShared should
	 * unconditionally propagate
	 */
	static final int PROPAGATE = -3;

	/**
	 * Status field, taking on only the values:
	 *   SIGNAL:     The successor of this node is (or will soon be)
	 *               blocked (via park), so the current node must
	 *               unpark its successor when it releases or
	 *               cancels. To avoid races, acquire methods must
	 *               first indicate they need a signal,
	 *               then retry the atomic acquire, and then,
	 *               on failure, block.
	 *   CANCELLED:  This node is cancelled due to timeout or interrupt.
	 *               Nodes never leave this state. In particular,
	 *               a thread with cancelled node never again blocks.
	 *   CONDITION:  This node is currently on a condition queue.
	 *               It will not be used as a sync queue node
	 *               until transferred, at which time the status
	 *               will be set to 0. (Use of this value here has
	 *               nothing to do with the other uses of the
	 *               field, but simplifies mechanics.)
	 *   PROPAGATE:  A releaseShared should be propagated to other
	 *               nodes. This is set (for head node only) in
	 *               doReleaseShared to ensure propagation
	 *               continues, even if other operations have
	 *               since intervened.
	 *   0:          None of the above
	 *
	 * The values are arranged numerically to simplify use.
	 * Non-negative values mean that a node doesn't need to
	 * signal. So, most code doesn't need to check for particular
	 * values, just for sign.
	 *
	 * The field is initialized to 0 for normal sync nodes, and
	 * CONDITION for condition nodes.  It is modified using CAS
	 * (or when possible, unconditional volatile writes).
	 */
	volatile int waitStatus;

	/**
	 * Link to predecessor node that current node/thread relies on
	 * for checking waitStatus. Assigned during enqueuing, and nulled
	 * out (for sake of GC) only upon dequeuing.  Also, upon
	 * cancellation of a predecessor, we short-circuit while
	 * finding a non-cancelled one, which will always exist
	 * because the head node is never cancelled: A node becomes
	 * head only as a result of successful acquire. A
	 * cancelled thread never succeeds in acquiring, and a thread only
	 * cancels itself, not any other node.
	 */
	volatile Node prev;

	/**
	 * Link to the successor node that the current node/thread
	 * unparks upon release. Assigned during enqueuing, adjusted
	 * when bypassing cancelled predecessors, and nulled out (for
	 * sake of GC) when dequeued.  The enq operation does not
	 * assign next field of a predecessor until after attachment,
	 * so seeing a null next field does not necessarily mean that
	 * node is at end of queue. However, if a next field appears
	 * to be null, we can scan prev's from the tail to
	 * double-check.  The next field of cancelled nodes is set to
	 * point to the node itself instead of null, to make life
	 * easier for isOnSyncQueue.
	 */
	volatile Node next;

	/**
	 * The thread that enqueued this node.  Initialized on
	 * construction and nulled out after use.
	 */
	volatile Thread thread;

	/**
	 * Link to next node waiting on condition, or the special
	 * value SHARED.  Because condition queues are accessed only
	 * when holding in exclusive mode, we just need a simple
	 * linked queue to hold nodes while they are waiting on
	 * conditions. They are then transferred to the queue to
	 * re-acquire. And because conditions can only be exclusive,
	 * we save a field by using special value to indicate shared
	 * mode.
	 */
	Node nextWaiter;

	/**
	 * Returns true if node is waiting in shared mode.
	 */
	final boolean isShared() {
		return nextWaiter == SHARED;
	}

	Node() {    // Used to establish initial head or SHARED marker
	}

	Node(Thread thread, Node mode) {     // Used by addWaiter
		this.nextWaiter = mode;
		this.thread = thread;
	}

	Node(Thread thread, int waitStatus) { // Used by Condition
		this.waitStatus = waitStatus;
		this.thread = thread;
	}
}

```

在 Node 节点中的值，waitStatus 用来跟踪当前线程的状态，初始创建时为默认值 0。在 AQS 中当该线程被取消后变为 CANCELLED，位于阻塞队列中等待唤醒时为 SIGNAL。CONDITION 和 PROPAGATE 分别用于 Condition 类控制中使用。

## 为什么 AQS 要使用双向链表来保存线程节点信息？

1. 在线程自旋过程中，需要判断前驱节点是否为头节点，若为头节点则自身无需阻塞而直接抢占锁，所以需要 pre 引用指向前节点。

## 为什么要使用 SIGNAL 来表示阻塞队列中的线程状态？

SIGNAL 标记的节点表示该节点正在等待一个资源，所以如果当前节点发现前驱节点是 SIGNAL 状态的话，那么自身就可以挂起了。如果不挂起的话由于唤醒会优先唤醒前驱节点，自己在队列中排队空占 CPU 是耗费资源的。因此在 AQS 中当前节点会将前驱节点状态改为 SIGNAL，表示前驱节点优先抢占锁，而自身需要等待直到被唤醒。

```java

private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
	int ws = pred.waitStatus;
	if (ws == Node.SIGNAL)
		/*
		 * This node has already set status asking a release
		 * to signal it, so it can safely park.
		 */
		return true;
	if (ws > 0) {
		/*
		 * Predecessor was cancelled. Skip over predecessors and
		 * indicate retry.
		 */
		do {
			node.prev = pred = pred.prev;
		} while (pred.waitStatus > 0);
		pred.next = node;
	} else {
		/*
		 * waitStatus must be 0 or PROPAGATE.  Indicate that we
		 * need a signal, but don't park yet.  Caller will need to
		 * retry to make sure it cannot acquire before parking.
		 */
		compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
	}
	return false;
}

```

