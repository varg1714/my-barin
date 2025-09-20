---
source: https://www.javadoop.com/post/java-thread-pool
create: 2025-09-20 16:21
read: false
knowledge: false
---
[回首页](/)

# 深度解读 java 线程池设计思想及源码实现

[](https://api.javadoop.com/post/download?name=java-thread-pool)

发布时间: 2017-09-05

我相信大家都看过很多的关于线程池的文章，基本上也是面试的时候必问的，如果你在看过很多文章以后，还是一知半解的，那希望这篇文章能让你真正的掌握好 Java 线程池。

本文一大重点是源码解析，同时会有少量篇幅介绍线程池设计思想以及作者 Doug Lea 实现过程中的一些巧妙用法。本文还是会一行行关键代码进行分析，目的是为了让那些自己看源码不是很理解的同学可以得到参考。

线程池是非常重要的工具，如果你要成为一个好的工程师，还是得比较好地掌握这个知识，很多线上问题都是因为没有用好线程池导致的。即使你为了谋生，也要知道，这基本上是面试必问的题目，而且面试官很容易从被面试者的回答中捕捉到被面试者的技术水平。

本文略长，建议在 pc 上阅读，边看文章边翻源码（Java7 和 Java8 都一样），建议想好好看的读者抽出至少 30 分钟的整块时间来阅读。当然，如果读者仅为面试准备，可以直接滑到最后的**总结**部分。

## [](#toc_9)总览

开篇来一些废话。下图是 java 线程池几个相关类的继承结构：

![](https://assets.javadoop.com/imgs/20510079/java-thread-pool/1.jpg)

先简单说说这个继承结构，Executor 位于最顶层，也是最简单的，就一个 execute(Runnable runnable) 接口方法定义。

ExecutorService 也是接口，在 Executor 接口的基础上添加了很多的接口方法，所以**一般来说我们会使用这个接口**。

然后再下来一层是 AbstractExecutorService，从名字我们就知道，这是抽象类，这里实现了非常有用的一些方法供子类直接使用，之后我们再细说。

然后才到我们的重点部分 ThreadPoolExecutor 类，这个类提供了关于线程池所需的非常丰富的功能。

另外，我们还涉及到下图中的这些类：

![](https://assets.javadoop.com/imgs/20510079/java-thread-pool/others.png)

同在并发包中的 Executors 类，类名中带字母 s，我们猜到这个是工具类，里面的方法都是静态方法，如以下我们最常用的用于生成 ThreadPoolExecutor 的实例的一些方法：

```
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

另外，由于线程池支持**获取线程执行的结果**，所以，引入了 Future 接口，RunnableFuture 继承自此接口，然后我们最需要关心的就是它的实现类 FutureTask。到这里，记住这个概念，在线程池的使用过程中，我们是往线程池提交任务（task），使用过线程池的都知道，我们提交的每个任务是实现了 Runnable 接口的，其实就是先将 Runnable 的任务包装成 FutureTask，然后再提交到线程池。这样，读者才能比较容易记住 FutureTask 这个类名：它首先是一个任务（Task），然后具有 Future 接口的语义，即可以在将来（Future）得到执行的结果。

当然，线程池中的 BlockingQueue 也是非常重要的概念，如果线程数达到 corePoolSize，我们的每个任务会提交到等待队列中，等待线程池中的线程来取任务并执行。这里的 BlockingQueue 通常我们使用其实现类 LinkedBlockingQueue、ArrayBlockingQueue 和 SynchronousQueue，每个实现类都有不同的特征，使用场景之后会慢慢分析。想要详细了解各个 BlockingQueue 的读者，可以参考我的前面的一篇对 BlockingQueue 的各个实现类进行详细分析的文章。

把事情说完整：除了上面说的这些类外，还有一个很重要的类，就是定时任务实现类 ScheduledThreadPoolExecutor，它继承自本文要重点讲解的 ThreadPoolExecutor，用于实现定时执行。不过本文不会介绍它的实现，我相信读者看完本文后可以比较容易地看懂它的源码。

以上就是本文要介绍的知识，废话不多说，开始进入正文。

## [](#toc_10)Executor 接口

```
/* 
 * @since 1.5
 * @author Doug Lea
 */
public interface Executor {
    void execute(Runnable command);
}
```

我们可以看到 Executor 接口非常简单，就一个 `void execute(Runnable command)` 方法，代表提交一个任务。为了让大家理解 java 线程池的整个设计方案，我会按照 Doug Lea 的设计思路来多说一些相关的东西。

我们经常这样启动一个线程：

```
new Thread(new Runnable(){
  // do something
}).start();
```

用了线程池 Executor 后就可以像下面这么使用：

```
Executor executor = anExecutor;
executor.execute(new RunnableTask1());
executor.execute(new RunnableTask2());
```

如果我们希望线程池同步执行每一个任务，我们可以这么实现这个接口：

```
class DirectExecutor implements Executor {
    public void execute(Runnable r) {
        r.run();// 这里不是用的new Thread(r).start()，也就是说没有启动任何一个新的线程。
    }
}
```

我们希望每个任务提交进来后，直接启动一个新的线程来执行这个任务，我们可以这么实现：

```
class ThreadPerTaskExecutor implements Executor {
    public void execute(Runnable r) {
        new Thread(r).start();  // 每个任务都用一个新的线程来执行
    }
}
```

我们再来看下怎么组合两个 Executor 来使用，下面这个实现是将所有的任务都加到一个 queue 中，然后从 queue 中取任务，交给真正的执行器执行，这里采用 synchronized 进行并发控制：

```
class SerialExecutor implements Executor {
    // 任务队列
    final Queue<Runnable> tasks = new ArrayDeque<Runnable>();
    // 这个才是真正的执行器
    final Executor executor;
    // 当前正在执行的任务
    Runnable active;
  
    // 初始化的时候，指定执行器
    SerialExecutor(Executor executor) {
        this.executor = executor;
    }
 
    // 添加任务到线程池: 将任务添加到任务队列，scheduleNext 触发执行器去任务队列取任务
    public synchronized void execute(final Runnable r) {
        tasks.offer(new Runnable() {
            public void run() {
                try {
                    r.run();
                } finally {
                    scheduleNext();
                }
            }
        });
        if (active == null) {
            scheduleNext();
        }
    }

    protected synchronized void scheduleNext() {
        if ((active = tasks.poll()) != null) {
            // 具体的执行转给真正的执行器 executor
            executor.execute(active);
        }
    }
}
```

当然了，Executor 这个接口只有提交任务的功能，太简单了，我们想要更丰富的功能，比如我们想知道执行结果、我们想知道当前线程池有多少个线程活着、已经完成了多少任务等等，这些都是这个接口的不足的地方。接下来我们要介绍的是继承自 `Executor` 接口的 `ExecutorService` 接口，这个接口提供了比较丰富的功能，也是我们最常使用到的接口。

## [](#toc_11)ExecutorService

一般我们定义一个线程池的时候，往往都是使用这个接口：

```
ExecutorService executor = Executors.newFixedThreadPool(args...);
ExecutorService executor = Executors.newCachedThreadPool(args...);
```

因为这个接口中定义的一系列方法大部分情况下已经可以满足我们的需要了。

那么我们简单初略地来看一下这个接口中都有哪些方法：

```
public interface ExecutorService extends Executor {

    // 关闭线程池，已提交的任务继续执行，不接受继续提交新任务
    void shutdown();

    // 关闭线程池，尝试停止正在执行的所有任务，不接受继续提交新任务
    // 它和前面的方法相比，加了一个单词“now”，区别在于它会去停止当前正在进行的任务
    List<Runnable> shutdownNow();

    // 线程池是否已关闭
    boolean isShutdown();

    // 如果调用了 shutdown() 或 shutdownNow() 方法后，所有任务结束了，那么返回true
    // 这个方法必须在调用shutdown或shutdownNow方法之后调用才会返回true
    boolean isTerminated();

    // 等待所有任务完成，并设置超时时间
    // 我们这么理解，实际应用中是，先调用 shutdown 或 shutdownNow，
    // 然后再调这个方法等待所有的线程真正地完成，返回值意味着有没有超时
    boolean awaitTermination(long timeout, TimeUnit unit)
            throws InterruptedException;

    // 提交一个 Callable 任务
    <T> Future<T> submit(Callable<T> task);

    // 提交一个 Runnable 任务，第二个参数将会放到 Future 中，作为返回值，
    // 因为 Runnable 的 run 方法本身并不返回任何东西
    <T> Future<T> submit(Runnable task, T result);

    // 提交一个 Runnable 任务
    Future<?> submit(Runnable task);

    // 执行所有任务，返回 Future 类型的一个 list
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
            throws InterruptedException;

    // 也是执行所有任务，但是这里设置了超时时间
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
            throws InterruptedException;

    // 只有其中的一个任务结束了，就可以返回，返回执行完的那个任务的结果
    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
            throws InterruptedException, ExecutionException;
  
    // 同上一个方法，只有其中的一个任务结束了，就可以返回，返回执行完的那个任务的结果，
    // 不过这个带超时，超过指定的时间，抛出 TimeoutException 异常
    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
            throws InterruptedException, ExecutionException, TimeoutException;
}
```

这些方法都很好理解，一个简单的线程池主要就是这些功能，能提交任务，能获取结果，能关闭线程池，这也是为什么我们经常用这个接口的原因。

## [](#toc_12)FutureTask

在继续往下层介绍 ExecutorService 的实现类之前，我们先来说说相关的类 FutureTask。

```
Future      Runnable
   \           /
    \         /
   RunnableFuture
          |
          |
      FutureTask
      
FutureTask 通过 RunnableFuture 间接实现了 Runnable 接口，
所以每个 Runnable 通常都先包装成 FutureTask，
然后调用 executor.execute(Runnable command) 将其提交给线程池
```

我们知道，Runnable 的 void run() 方法是没有返回值的，所以，通常，如果我们需要的话，会在 submit 中指定第二个参数作为返回值：

```
<T> Future<T> submit(Runnable task, T result);
```

其实到时候会通过这两个参数，将其包装成 Callable。它和 Runnable 的区别在于 run() 没有返回值，而 Callable 的 call() 方法有返回值，同时，如果运行出现异常，call() 方法会抛出异常。

```
public interface Callable<V> {
   
    V call() throws Exception;
}
```

在这里，就不展开说 FutureTask 类了，因为本文篇幅本来就够大了，这里我们需要知道怎么用就行了。

下面，我们来看看 `ExecutorService` 的抽象实现 `AbstractExecutorService` 。

## [](#toc_13)AbstractExecutorService

AbstractExecutorService 抽象类派生自 ExecutorService 接口，然后在其基础上实现了几个实用的方法，这些方法提供给子类进行调用。

这个抽象类实现了 invokeAny 方法和 invokeAll 方法，这里的两个 newTaskFor 方法也比较有用，用于将任务包装成 FutureTask。定义于最上层接口 Executor 中的 `void execute(Runnable command)` 由于不需要获取结果，不会进行 FutureTask 的包装。

需要获取结果（FutureTask），用 submit 方法，不需要获取结果，可以用 execute 方法。

下面，我将一行一行源码地来分析这个类，跟着源码来看看其实现吧：

Tips: invokeAny 和 invokeAll 方法占了这整个类的绝大多数篇幅，读者可以选择适当跳过，因为它们可能在你的实践中使用的频次比较低，而且它们不带有承前启后的作用，不用担心会漏掉什么导致看不懂后面的代码。

```
public abstract class AbstractExecutorService implements ExecutorService {

    // RunnableFuture 是用于获取执行结果的，我们常用它的子类 FutureTask
    // 下面两个 newTaskFor 方法用于将我们的任务包装成 FutureTask 提交到线程池中执行
    protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
        return new FutureTask<T>(runnable, value);
    }

    protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
        return new FutureTask<T>(callable);
    }

    // 提交任务
    public Future<?> submit(Runnable task) {
        if (task == null) throw new NullPointerException();
        // 1. 将任务包装成 FutureTask
        RunnableFuture<Void> ftask = newTaskFor(task, null);
        // 2. 交给执行器执行，execute 方法由具体的子类来实现
        // 前面也说了，FutureTask 间接实现了Runnable 接口。
        execute(ftask);
        return ftask;
    }

    public <T> Future<T> submit(Runnable task, T result) {
        if (task == null) throw new NullPointerException();
        // 1. 将任务包装成 FutureTask
        RunnableFuture<T> ftask = newTaskFor(task, result);
        // 2. 交给执行器执行
        execute(ftask);
        return ftask;
    }
  
    public <T> Future<T> submit(Callable<T> task) {
        if (task == null) throw new NullPointerException();
        // 1. 将任务包装成 FutureTask
        RunnableFuture<T> ftask = newTaskFor(task);
        // 2. 交给执行器执行
        execute(ftask);
        return ftask;
    }

    // 此方法目的：将 tasks 集合中的任务提交到线程池执行，任意一个线程执行完后就可以结束了
    // 第二个参数 timed 代表是否设置超时机制，超时时间为第三个参数，
    // 如果 timed 为 true，同时超时了还没有一个线程返回结果，那么抛出 TimeoutException 异常
    private <T> T doInvokeAny(Collection<? extends Callable<T>> tasks,
                            boolean timed, long nanos)
        throws InterruptedException, ExecutionException, TimeoutException {
        if (tasks == null)
            throw new NullPointerException();
        // 任务数
        int ntasks = tasks.size();
        if (ntasks == 0)
            throw new IllegalArgumentException();
        // 
        List<Future<T>> futures= new ArrayList<Future<T>>(ntasks);
      
        // ExecutorCompletionService 不是一个真正的执行器，参数 this 才是真正的执行器
        // 它对执行器进行了包装，每个任务结束后，将结果保存到内部的一个 completionQueue 队列中
        // 这也是为什么这个类的名字里面有个 Completion 的原因吧。
        ExecutorCompletionService<T> ecs =
            new ExecutorCompletionService<T>(this);
        try {
            // 用于保存异常信息，此方法如果没有得到任何有效的结果，那么我们可以抛出最后得到的一个异常
            ExecutionException ee = null;
            long lastTime = timed ? System.nanoTime() : 0;
            Iterator<? extends Callable<T>> it = tasks.iterator();

            // 首先先提交一个任务，后面的任务到下面的 for 循环一个个提交
            futures.add(ecs.submit(it.next()));
            // 提交了一个任务，所以任务数量减 1
            --ntasks;
            // 正在执行的任务数(提交的时候 +1，任务结束的时候 -1)
            int active = 1;

            for (;;) {
                // ecs 上面说了，其内部有一个 completionQueue 用于保存执行完成的结果
                // BlockingQueue 的 poll 方法不阻塞，返回 null 代表队列为空
                Future<T> f = ecs.poll();
                // 为 null，说明刚刚提交的第一个线程还没有执行完成
                // 在前面先提交一个任务，加上这里做一次检查，也是为了提高性能
                if (f == null) {
                    if (ntasks > 0) {
                        --ntasks;
                        futures.add(ecs.submit(it.next()));
                        ++active;
                    }
                    // 这里是 else if，不是 if。这里说明，没有任务了，同时 active 为 0 说明
                    // 任务都执行完成了。其实我也没理解为什么这里做一次 break？
                    // 因为我认为 active 为 0 的情况，必然从下面的 f.get() 返回了
                    
                    // 2018-02-23 感谢读者 newmicro 的 comment，
                    //  这里的 active == 0，说明所有的任务都执行失败，那么这里是 for 循环出口
                    else if (active == 0)
                        break;
                    // 这里也是 else if。这里说的是，没有任务了，但是设置了超时时间，这里检测是否超时
                    else if (timed) {
                        // 带等待的 poll 方法
                        f = ecs.poll(nanos, TimeUnit.NANOSECONDS);
                        // 如果已经超时，抛出 TimeoutException 异常，这整个方法就结束了
                        if (f == null)
                            throw new TimeoutException();
                        long now = System.nanoTime();
                        nanos -= now - lastTime;
                        lastTime = now;
                    }
                    // 这里是 else。说明，没有任务需要提交，但是池中的任务没有完成，还没有超时(如果设置了超时)
                    // take() 方法会阻塞，直到有元素返回，说明有任务结束了
                    else
                        f = ecs.take();
                }
                /*
                 * 我感觉上面这一段并不是很好理解，这里简单说下。
                 * 1. 首先，这在一个 for 循环中，我们设想每一个任务都没那么快结束，
                 *     那么，每一次都会进到第一个分支，进行提交任务，直到将所有的任务都提交了
                 * 2. 任务都提交完成后，如果设置了超时，那么 for 循环其实进入了“一直检测是否超时”
                       这件事情上
                 * 3. 如果没有设置超时机制，那么不必要检测超时，那就会阻塞在 ecs.take() 方法上，
                       等待获取第一个执行结果
                 * 4. 如果所有的任务都执行失败，也就是说 future 都返回了，
                       但是 f.get() 抛出异常，那么从 active == 0 分支出去(感谢 newmicro 提出)
                         // 当然，这个需要看下面的 if 分支。
                 */
              
              
              
                // 有任务结束了
                if (f != null) {
                    --active;
                    try {
                        // 返回执行结果，如果有异常，都包装成 ExecutionException
                        return f.get();
                    } catch (ExecutionException eex) {
                        ee = eex;
                    } catch (RuntimeException rex) {
                        ee = new ExecutionException(rex);
                    }
                }
            }// 注意看 for 循环的范围，一直到这里
          
            if (ee == null)
                ee = new ExecutionException();
            throw ee;
        
        } finally {
            // 方法退出之前，取消其他的任务
            for (Future<T> f : futures)
                f.cancel(true);
        }
    }

    public <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException {
        try {
            return doInvokeAny(tasks, false, 0);
        } catch (TimeoutException cannotHappen) {
            assert false;
            return null;
        }
    }

    public <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                           long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException {
        return doInvokeAny(tasks, true, unit.toNanos(timeout));
    }

    // 执行所有的任务，返回任务结果。
    // 先不要看这个方法，我们先想想，其实我们自己提交任务到线程池，也是想要线程池执行所有的任务
    // 只不过，我们是每次 submit 一个任务，这里以一个集合作为参数提交
    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException {
        if (tasks == null)
            throw new NullPointerException();
        List<Future<T>> futures = new ArrayList<Future<T>>(tasks.size());
        boolean done = false;
        try {
            // 这个很简单
            for (Callable<T> t : tasks) {
                // 包装成 FutureTask
                RunnableFuture<T> f = newTaskFor(t);
                futures.add(f);
                // 提交任务
                execute(f);
            }
            for (Future<T> f : futures) {
                if (!f.isDone()) {
                    try {
                        // 这是一个阻塞方法，直到获取到值，或抛出了异常
                        // 这里有个小细节，其实 get 方法签名上是会抛出 InterruptedException 的
                        // 可是这里没有进行处理，而是抛给外层去了。此异常发生于还没执行完的任务被取消了
                        f.get();
                    } catch (CancellationException ignore) {
                    } catch (ExecutionException ignore) {
                    }
                }
            }
            done = true;
            // 这个方法返回，不像其他的场景，返回 List<Future>，其实执行结果还没出来
            // 这个方法返回是真正的返回，任务都结束了
            return futures;
        } finally {
            // 为什么要这个？就是上面说的有异常的情况
            if (!done)
                for (Future<T> f : futures)
                    f.cancel(true);
        }
    }

    // 带超时的 invokeAll，我们找不同吧
    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                         long timeout, TimeUnit unit)
        throws InterruptedException {
        if (tasks == null || unit == null)
            throw new NullPointerException();
        long nanos = unit.toNanos(timeout);
        List<Future<T>> futures = new ArrayList<Future<T>>(tasks.size());
        boolean done = false;
        try {
            for (Callable<T> t : tasks)
                futures.add(newTaskFor(t));

            long lastTime = System.nanoTime();

            Iterator<Future<T>> it = futures.iterator();
            // 每提交一个任务，检测一次是否超时
            while (it.hasNext()) {
                execute((Runnable)(it.next()));
                long now = System.nanoTime();
                nanos -= now - lastTime;
                lastTime = now;
                // 超时
                if (nanos <= 0)
                    return futures;
            }

            for (Future<T> f : futures) {
                if (!f.isDone()) {
                    if (nanos <= 0)
                        return futures;
                    try {
                        // 调用带超时的 get 方法，这里的参数 nanos 是剩余的时间，
                        // 因为上面其实已经用掉了一些时间了
                        f.get(nanos, TimeUnit.NANOSECONDS);
                    } catch (CancellationException ignore) {
                    } catch (ExecutionException ignore) {
                    } catch (TimeoutException toe) {
                        return futures;
                    }
                    long now = System.nanoTime();
                    nanos -= now - lastTime;
                    lastTime = now;
                }
            }
            done = true;
            return futures;
        } finally {
            if (!done)
                for (Future<T> f : futures)
                    f.cancel(true);
        }
    }

}
```

到这里，我们发现，这个抽象类包装了一些基本的方法，可是像 submit、invokeAny、invokeAll 等方法，它们都没有真正开启线程来执行任务，它们都只是在方法内部调用了 execute 方法，所以最重要的 execute(Runnable runnable) 方法还没出现，需要等具体执行器来实现这个最重要的部分，这里我们要说的就是 ThreadPoolExecutor 类了。

鉴于本文的篇幅，我觉得看到这里的读者应该已经不多了，大家都习惯了快餐文化。我写的每篇文章都力求让读者可以通过我的一篇文章而对相关内容有全面的了解，所以篇幅不免长了些。

## [](#toc_14)ThreadPoolExecutor

ThreadPoolExecutor 是 JDK 中的线程池实现，这个类实现了一个线程池需要的各个方法，它实现了任务提交、线程管理、监控等等方法。

我们可以基于它来进行业务上的扩展，以实现我们需要的其他功能，比如实现定时任务的类 ScheduledThreadPoolExecutor 就继承自 ThreadPoolExecutor。当然，这不是本文关注的重点，下面，还是赶紧进行源码分析吧。

首先，我们来看看线程池实现中的几个概念和处理流程。

我们先回顾下提交任务的几个方法：

```
public Future<?> submit(Runnable task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<Void> ftask = newTaskFor(task, null);
    execute(ftask);
    return ftask;
}
public <T> Future<T> submit(Runnable task, T result) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task, result);
    execute(ftask);
    return ftask;
}
public <T> Future<T> submit(Callable<T> task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task);
    execute(ftask);
    return ftask;
}
```

一个最基本的概念是，submit 方法中，参数是 Runnable 类型（也有 Callable 类型），这个参数不是用于 new Thread(**runnable**).start() 中的，此处的这个参数不是用于启动线程的，这里指的是**任务**，任务要做的事情是 run() 方法里面定义的或 Callable 中的 call() 方法里面定义的。

初学者往往会搞混这个，因为 Runnable 总是在各个地方出现，经常把一个 Runnable 包到另一个 Runnable 中。请把它想象成有个 Task 接口，这个接口里面有一个 run() 方法。

我们回过神来继续往下看，我画了一个简单的示意图来描述线程池中的一些主要的构件：

![](https://assets.javadoop.com/imgs/20510079/java-thread-pool/pool-1.png)

当然，上图没有考虑队列是否有界，提交任务时队列满了怎么办？什么情况下会创建新的线程？提交任务时线程池满了怎么办？空闲线程怎么关掉？这些问题下面我们会一一解决。

我们经常会使用 `Executors` 这个工具类来快速构造一个线程池，对于初学者而言，这种工具类是很有用的，开发者不需要关注太多的细节，只要知道自己需要一个线程池，仅仅提供必需的参数就可以了，其他参数都采用作者提供的默认值。

```
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

这里先不说有什么区别，它们最终都会导向这个构造方法：

```
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        // 这几个参数都是必须要有的
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
      
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

基本上，上面的构造方法中列出了我们最需要关心的几个属性了，下面逐个介绍下构造方法中出现的这几个属性：

*   corePoolSize
    
    核心线程数，不要抠字眼，反正先记着有这么个属性就可以了。
    
*   maximumPoolSize
    
    最大线程数，线程池允许创建的最大线程数。
    
*   workQueue
    
    任务队列，BlockingQueue 接口的某个实现（常使用 ArrayBlockingQueue 和 LinkedBlockingQueue）。
    
*   keepAliveTime
    
    空闲线程的保活时间，如果某线程的空闲时间超过这个值都没有任务给它做，那么可以被关闭了。注意这个值并不会对所有线程起作用，如果线程池中的线程数少于等于核心线程数 corePoolSize，那么这些线程不会因为空闲太长时间而被关闭，当然，也可以通过调用 `allowCoreThreadTimeOut(true)`使核心线程数内的线程也可以被回收。
    
*   threadFactory
    
    用于生成线程，一般我们可以用默认的就可以了。通常，我们可以通过它将我们的线程的名字设置得比较可读一些，如 Message-Thread-1， Message-Thread-2 类似这样。
    
*   handler：
    
    当线程池已经满了，但是又有新的任务提交的时候，该采取什么策略由这个来指定。有几种方式可供选择，像抛出异常、直接拒绝然后返回等，也可以自己实现相应的接口实现自己的逻辑，这个之后再说。
    

除了上面几个属性外，我们再看看其他重要的属性。

Doug Lea 采用一个 32 位的整数来存放线程池的状态和当前池中的线程数，其中高 3 位用于存放线程池状态，低 29 位表示线程数（即使只有 29 位，也已经不小了，大概 5 亿多，现在还没有哪个机器能起这么多线程的吧）。我们知道，java 语言在整数编码上是统一的，都是采用补码的形式，下面是简单的移位操作和布尔操作，都是挺简单的。

```
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));

// 这里 COUNT_BITS 设置为 29(32-3)，意味着前三位用于存放线程状态，后29位用于存放线程数
// 很多初学者很喜欢在自己的代码中写很多 29 这种数字，或者某个特殊的字符串，然后分布在各个地方，这是非常糟糕的
private static final int COUNT_BITS = Integer.SIZE - 3;

// 000 11111111111111111111111111111
// 这里得到的是 29 个 1，也就是说线程池的最大线程数是 2^29-1=536870911
// 以我们现在计算机的实际情况，这个数量还是够用的
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

// 我们说了，线程池的状态存放在高 3 位中
// 运算结果为 111跟29个0：111 00000000000000000000000000000
private static final int RUNNING    = -1 << COUNT_BITS;
// 000 00000000000000000000000000000
private static final int SHUTDOWN   =  0 << COUNT_BITS;
// 001 00000000000000000000000000000
private static final int STOP       =  1 << COUNT_BITS;
// 010 00000000000000000000000000000
private static final int TIDYING    =  2 << COUNT_BITS;
// 011 00000000000000000000000000000
private static final int TERMINATED =  3 << COUNT_BITS;

// 将整数 c 的低 29 位修改为 0，就得到了线程池的状态
private static int runStateOf(int c) { return c & ~CAPACITY; }
// 将整数 c 的高 3 为修改为 0，就得到了线程池中的线程数
private static int workerCountOf(int c) { return c & CAPACITY; }

private static int ctlOf(int rs, int wc) { return rs | wc; }

/*
 * Bit field accessors that don't require unpacking ctl.
 * These depend on the bit layout and on workerCount being never negative.
 */

private static boolean runStateLessThan(int c, int s) {
    return c < s;
}

private static boolean runStateAtLeast(int c, int s) {
    return c >= s;
}

private static boolean isRunning(int c) {
    return c < SHUTDOWN;
}
```

上面就是对一个整数的简单的位操作，几个操作方法将会在后面的源码中一直出现，所以读者最好把方法名字和其代表的功能记住，看源码的时候也就不需要来来回回翻了。

在这里，介绍下线程池中的各个状态和状态变化的转换过程：

*   RUNNING：这个没什么好说的，这是最正常的状态：接受新的任务，处理等待队列中的任务
*   SHUTDOWN：不接受新的任务提交，但是会继续处理等待队列中的任务
*   STOP：不接受新的任务提交，不再处理等待队列中的任务，中断正在执行任务的线程
*   TIDYING：所有的任务都销毁了，workCount 为 0。线程池的状态在转换为 TIDYING 状态时，会执行钩子方法 terminated()
*   TERMINATED：terminated() 方法结束后，线程池的状态就会变成这个

RUNNING 定义为 -1，SHUTDOWN 定义为 0，其他的都比 0 大，所以等于 0 的时候不能提交任务，大于 0 的话，连正在执行的任务也需要中断。

看了这几种状态的介绍，读者大体也可以猜到十之八九的状态转换了，各个状态的转换过程有以下几种：

*   RUNNING -> SHUTDOWN：当调用了 shutdown() 后，会发生这个状态转换，这也是最重要的
*   (RUNNING or SHUTDOWN) -> STOP：当调用 shutdownNow() 后，会发生这个状态转换，这下要清楚 shutDown() 和 shutDownNow() 的区别了
*   SHUTDOWN -> TIDYING：当任务队列和线程池都清空后，会由 SHUTDOWN 转换为 TIDYING
*   STOP -> TIDYING：当任务队列清空后，发生这个转换
*   TIDYING -> TERMINATED：这个前面说了，当 terminated() 方法结束后

上面的几个记住核心的就可以了，尤其第一个和第二个。

另外，我们还要看看一个内部类 Worker，因为 Doug Lea 把线程池中的线程包装成了一个个 Worker，翻译成工人，就是线程池中做任务的线程。所以到这里，我们知道**任务是 Runnable（内部变量名叫 task 或 command），线程是 Worker**。

Worker 这里又用到了抽象类 AbstractQueuedSynchronizer。题外话，AQS 在并发中真的是到处出现，而且非常容易使用，写少量的代码就能实现自己需要的同步方式（对 AQS 源码感兴趣的读者请参看我之前写的几篇文章）。

```
private final class Worker
    extends AbstractQueuedSynchronizer
    implements Runnable {
    private static final long serialVersionUID = 6138294804551838833L;

    // 这个是真正的线程，任务靠你啦
    final Thread thread;
    
    // 前面说了，这里的 Runnable 是任务。为什么叫 firstTask？因为在创建线程的时候，如果同时指定了
    // 这个线程起来以后需要执行的第一个任务，那么第一个任务就是存放在这里的(线程可不止执行这一个任务)
    // 当然了，也可以为 null，这样线程起来了，自己到任务队列（BlockingQueue）中取任务（getTask 方法）就行了
    Runnable firstTask;
    
    // 用于存放此线程完成的任务数，注意了，这里用了 volatile，保证可见性
    volatile long completedTasks;

    // Worker 只有这一个构造方法，传入 firstTask，也可以传 null
    Worker(Runnable firstTask) {
        setState(-1); // inhibit interrupts until runWorker
        this.firstTask = firstTask;
        // 调用 ThreadFactory 来创建一个新的线程
        this.thread = getThreadFactory().newThread(this);
    }

    // 这里调用了外部类的 runWorker 方法
    public void run() {
        runWorker(this);
    }

    ...// 其他几个方法没什么好看的，就是用 AQS 操作，来获取这个线程的执行权，用了独占锁
}
```

前面虽然啰嗦，但是简单。有了上面的这些基础后，我们终于可以看看 ThreadPoolExecutor 的 execute 方法了，前面源码分析的时候也说了，各种方法都最终依赖于 execute 方法：

```
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
  
    // 前面说的那个表示 “线程池状态” 和 “线程数” 的整数
    int c = ctl.get();
  
    // 如果当前线程数少于核心线程数，那么直接添加一个 worker 来执行任务，
    // 创建一个新的线程，并把当前任务 command 作为这个线程的第一个任务(firstTask)
    if (workerCountOf(c) < corePoolSize) {
        // 添加任务成功，那么就结束了。提交任务嘛，线程池已经接受了这个任务，这个方法也就可以返回了
        // 至于执行的结果，到时候会包装到 FutureTask 中。
        // 返回 false 代表线程池不允许提交任务
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    // 到这里说明，要么当前线程数大于等于核心线程数，要么刚刚 addWorker 失败了
  
    // 如果线程池处于 RUNNING 状态，把这个任务添加到任务队列 workQueue 中
    if (isRunning(c) && workQueue.offer(command)) {
        /* 这里面说的是，如果任务进入了 workQueue，我们是否需要开启新的线程
         * 因为线程数在 [0, corePoolSize) 是无条件开启新的线程
         * 如果线程数已经大于等于 corePoolSize，那么将任务添加到队列中，然后进到这里
         */
        int recheck = ctl.get();
        // 如果线程池已不处于 RUNNING 状态，那么移除已经入队的这个任务，并且执行拒绝策略
        if (! isRunning(recheck) && remove(command))
            reject(command);
        // 如果线程池还是 RUNNING 的，并且线程数为 0，那么开启新的线程
        // 到这里，我们知道了，这块代码的真正意图是：担心任务提交到队列中了，但是线程都关闭了
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // 如果 workQueue 队列满了，那么进入到这个分支
    // 以 maximumPoolSize 为界创建新的 worker，
    // 如果失败，说明当前线程数已经达到 maximumPoolSize，执行拒绝策略
    else if (!addWorker(command, false))
        reject(command);
}
```

对创建线程的错误理解：如果线程数少于 corePoolSize，创建一个线程，如果线程数在 [corePoolSize, maximumPoolSize] 之间那么可以创建线程或复用空闲线程，keepAliveTime 对这个区间的线程有效。

从上面的几个分支，我们就可以看出，上面的这段话是错误的。

上面这些一时半会也不可能全部消化搞定，我们先继续往下吧，到时候再回头看几遍。

这个方法非常重要 addWorker(Runnable firstTask, boolean core) 方法，我们看看它是怎么创建新的线程的：

```
// 第一个参数是准备提交给这个线程执行的任务，之前说了，可以为 null
// 第二个参数为 true 代表使用核心线程数 corePoolSize 作为创建线程的界限，也就说创建这个线程的时候，
// 		如果线程池中的线程总数已经达到 corePoolSize，那么不能响应这次创建线程的请求
// 		如果是 false，代表使用最大线程数 maximumPoolSize 作为界限
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // 这个非常不好理解
        // 如果线程池已关闭，并满足以下条件之一，那么不创建新的 worker：
        // 1. 线程池状态大于 SHUTDOWN，其实也就是 STOP, TIDYING, 或 TERMINATED
        // 2. firstTask != null
        // 3. workQueue.isEmpty()
        // 简单分析下：
        // 还是状态控制的问题，当线程池处于 SHUTDOWN 的时候，不允许提交任务，但是已有的任务继续执行
        // 当状态大于 SHUTDOWN 时，不允许提交任务，且中断正在执行的任务
        // 多说一句：如果线程池处于 SHUTDOWN，但是 firstTask 为 null，且 workQueue 非空，那么是允许创建 worker 的
        // 这是因为 SHUTDOWN 的语义：不允许提交新的任务，但是要把已经进入到 workQueue 的任务执行完，所以在满足条件的基础上，是允许创建新的 Worker 的
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
               firstTask == null &&
               ! workQueue.isEmpty()))
            return false;

        for (;;) {
            int wc = workerCountOf(c);
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            // 如果成功，那么就是所有创建线程前的条件校验都满足了，准备创建线程执行任务了
            // 这里失败的话，说明有其他线程也在尝试往线程池中创建线程
            if (compareAndIncrementWorkerCount(c))
                break retry;
            // 由于有并发，重新再读取一下 ctl
            c = ctl.get();
            // 正常如果是 CAS 失败的话，进到下一个里层的for循环就可以了
            // 可是如果是因为其他线程的操作，导致线程池的状态发生了变更，如有其他线程关闭了这个线程池
            // 那么需要回到外层的for循环
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }

    /* 
     * 到这里，我们认为在当前这个时刻，可以开始创建线程来执行任务了，
     * 因为该校验的都校验了，至于以后会发生什么，那是以后的事，至少当前是满足条件的
     */
  
    // worker 是否已经启动
    boolean workerStarted = false;
    // 是否已将这个 worker 添加到 workers 这个 HashSet 中
    boolean workerAdded = false;
    Worker w = null;
    try {
        final ReentrantLock mainLock = this.mainLock;
        // 把 firstTask 传给 worker 的构造方法
        w = new Worker(firstTask);
        // 取 worker 中的线程对象，之前说了，Worker的构造方法会调用 ThreadFactory 来创建一个新的线程
        final Thread t = w.thread;
        if (t != null) {
            // 这个是整个线程池的全局锁，持有这个锁才能让下面的操作“顺理成章”，
            // 因为关闭一个线程池需要这个锁，至少我持有锁的期间，线程池不会被关闭
            mainLock.lock();
            try {

                int c = ctl.get();
                int rs = runStateOf(c);

                // 小于 SHUTTDOWN 那就是 RUNNING，这个自不必说，是最正常的情况
                // 如果等于 SHUTDOWN，前面说了，不接受新的任务，但是会继续执行等待队列中的任务
                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    // worker 里面的 thread 可不能是已经启动的
                    if (t.isAlive())
                        throw new IllegalThreadStateException();
                    // 加到 workers 这个 HashSet 中
                    workers.add(w);
                    int s = workers.size();
                    // largestPoolSize 用于记录 workers 中的个数的最大值
                    // 因为 workers 是不断增加减少的，通过这个值可以知道线程池的大小曾经达到的最大值
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            // 添加成功的话，启动这个线程
            if (workerAdded) {
                // 启动线程
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        // 如果线程没有启动，需要做一些清理工作，如前面 workCount 加了 1，将其减掉
        if (! workerStarted)
            addWorkerFailed(w);
    }
    // 返回线程是否启动成功
    return workerStarted;
}
```

简单看下 addWorkFailed 的处理：

```
// workers 中删除掉相应的 worker
// workCount 减 1
private void addWorkerFailed(Worker w) {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        if (w != null)
            workers.remove(w);
        decrementWorkerCount();
        // rechecks for termination, in case the existence of this worker was holding up termination
        tryTerminate();
    } finally {
        mainLock.unlock();
    }
}
```

回过头来，继续往下走。我们知道，worker 中的线程 start 后，其 run 方法会调用 runWorker 方法：

```
// Worker 类的 run() 方法
public void run() {
    runWorker(this);
}
```

继续往下看 runWorker 方法：

```
// 此方法由 worker 线程启动后调用，这里用一个 while 循环来不断地从等待队列中获取任务并执行
// 前面说了，worker 在初始化的时候，可以指定 firstTask，那么第一个任务也就可以不需要从队列中获取
final void runWorker(Worker w) {
    // 
    Thread wt = Thread.currentThread();
    // 该线程的第一个任务(如果有的话)
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        // 循环调用 getTask 获取任务
        while (task != null || (task = getTask()) != null) {
            w.lock();          
            // 如果线程池状态大于等于 STOP，那么意味着该线程也要中断
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                // 这是一个钩子方法，留给需要的子类实现
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    // 到这里终于可以执行任务了
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    // 这里不允许抛出 Throwable，所以转换为 Error
                    thrown = x; throw new Error(x);
                } finally {
                    // 也是一个钩子方法，将 task 和异常作为参数，留给需要的子类实现
                    afterExecute(task, thrown);
                }
            } finally {
                // 置空 task，准备 getTask 获取下一个任务
                task = null;
                // 累加完成的任务数
                w.completedTasks++;
                // 释放掉 worker 的独占锁
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        // 如果到这里，需要执行线程关闭：
        // 1. 说明 getTask 返回 null，也就是说，队列中已经没有任务需要执行了，执行关闭
        // 2. 任务执行过程中发生了异常
        // 第一种情况，已经在代码处理了将 workCount 减 1，这个在 getTask 方法分析中会说
        // 第二种情况，workCount 没有进行处理，所以需要在 processWorkerExit 中处理
        // 限于篇幅，我不准备分析这个方法了，感兴趣的读者请自行分析源码
        processWorkerExit(w, completedAbruptly);
    }
}
```

我们看看 getTask() 是怎么获取任务的，这个方法写得真的很好，每一行都很简单，组合起来却所有的情况都想好了：

```
// 此方法有三种可能：
// 1. 阻塞直到获取到任务返回。我们知道，默认 corePoolSize 之内的线程是不会被回收的，
//      它们会一直等待任务
// 2. 超时退出。keepAliveTime 起作用的时候，也就是如果这么多时间内都没有任务，那么应该执行关闭
// 3. 如果发生了以下条件，此方法必须返回 null:
//    - 池中有大于 maximumPoolSize 个 workers 存在(通过调用 setMaximumPoolSize 进行设置)
//    - 线程池处于 SHUTDOWN，而且 workQueue 是空的，前面说了，这种不再接受新的任务
//    - 线程池处于 STOP，不仅不接受新的线程，连 workQueue 中的线程也不再执行
private Runnable getTask() {
    boolean timedOut = false; // Did the last poll() time out?
  
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);
        // 两种可能
        // 1. rs == SHUTDOWN && workQueue.isEmpty()
        // 2. rs >= STOP
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            // CAS 操作，减少工作线程数
            decrementWorkerCount();
            return null;
        }

        boolean timed;      // Are workers subject to culling?
        for (;;) {
            int wc = workerCountOf(c);
            // 允许核心线程数内的线程回收，或当前线程数超过了核心线程数，那么有可能发生超时关闭
            timed = allowCoreThreadTimeOut || wc > corePoolSize;

            // 这里 break，是为了不往下执行后一个 if (compareAndDecrementWorkerCount(c))
            // 两个 if 一起看：如果当前线程数 wc > maximumPoolSize，或者超时，都返回 null
            // 那这里的问题来了，wc > maximumPoolSize 的情况，为什么要返回 null？
            //    换句话说，返回 null 意味着关闭线程。
            // 那是因为有可能开发者调用了 setMaximumPoolSize() 将线程池的 maximumPoolSize 调小了，那么多余的 Worker 就需要被关闭
            if (wc <= maximumPoolSize && ! (timedOut && timed))
                break;
            if (compareAndDecrementWorkerCount(c))
                return null;
            c = ctl.get();  // Re-read ctl
            // compareAndDecrementWorkerCount(c) 失败，线程池中的线程数发生了改变
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
        // wc <= maximumPoolSize 同时没有超时
        try {
            // 到 workQueue 中获取任务
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();
            if (r != null)
                return r;
            timedOut = true;
        } catch (InterruptedException retry) {
            // 如果此 worker 发生了中断，采取的方案是重试
            // 解释下为什么会发生中断，这个读者要去看 setMaximumPoolSize 方法。
          
            // 如果开发者将 maximumPoolSize 调小了，导致其小于当前的 workers 数量，
            // 那么意味着超出的部分线程要被关闭。重新进入 for 循环，自然会有部分线程会返回 null
            timedOut = false;
        }
    }
}
```

到这里，基本上也说完了整个流程，读者这个时候应该回到 execute(Runnable command) 方法，看看各个分支，我把代码贴过来一下：

```
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
  
    // 前面说的那个表示 “线程池状态” 和 “线程数” 的整数
    int c = ctl.get();
  
    // 如果当前线程数少于核心线程数，那么直接添加一个 worker 来执行任务，
    // 创建一个新的线程，并把当前任务 command 作为这个线程的第一个任务(firstTask)
    if (workerCountOf(c) < corePoolSize) {
        // 添加任务成功，那么就结束了。提交任务嘛，线程池已经接受了这个任务，这个方法也就可以返回了
        // 至于执行的结果，到时候会包装到 FutureTask 中。
        // 返回 false 代表线程池不允许提交任务
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    // 到这里说明，要么当前线程数大于等于核心线程数，要么刚刚 addWorker 失败了
  
    // 如果线程池处于 RUNNING 状态，把这个任务添加到任务队列 workQueue 中
    if (isRunning(c) && workQueue.offer(command)) {
        /* 这里面说的是，如果任务进入了 workQueue，我们是否需要开启新的线程
         * 因为线程数在 [0, corePoolSize) 是无条件开启新的线程
         * 如果线程数已经大于等于 corePoolSize，那么将任务添加到队列中，然后进到这里
         */
        int recheck = ctl.get();
        // 如果线程池已不处于 RUNNING 状态，那么移除已经入队的这个任务，并且执行拒绝策略
        if (! isRunning(recheck) && remove(command))
            reject(command);
        // 如果线程池还是 RUNNING 的，并且线程数为 0，那么开启新的线程
        // 到这里，我们知道了，这块代码的真正意图是：担心任务提交到队列中了，但是线程都关闭了
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // 如果 workQueue 队列满了，那么进入到这个分支
    // 以 maximumPoolSize 为界创建新的 worker，
    // 如果失败，说明当前线程数已经达到 maximumPoolSize，执行拒绝策略
    else if (!addWorker(command, false))
        reject(command);
}
```

上面各个分支中，有两种情况会调用 reject(command) 来处理任务，因为按照正常的流程，线程池此时不能接受这个任务，所以需要执行我们的拒绝策略。接下来，我们说一说 ThreadPoolExecutor 中的拒绝策略。

```
final void reject(Runnable command) {
    // 执行拒绝策略
    handler.rejectedExecution(command, this);
}
```

此处的 handler 我们需要在构造线程池的时候就传入这个参数，它是 RejectedExecutionHandler 的实例。

RejectedExecutionHandler 在 ThreadPoolExecutor 中有四个已经定义好的实现类可供我们直接使用，当然，我们也可以实现自己的策略，不过一般也没有必要。

```
// 只要线程池没有被关闭，那么由提交任务的线程自己来执行这个任务。
public static class CallerRunsPolicy implements RejectedExecutionHandler {
    public CallerRunsPolicy() { }
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            r.run();
        }
    }
}

// 不管怎样，直接抛出 RejectedExecutionException 异常
// 这个是默认的策略，如果我们构造线程池的时候不传相应的 handler 的话，那就会指定使用这个
public static class AbortPolicy implements RejectedExecutionHandler {
    public AbortPolicy() { }
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        throw new RejectedExecutionException("Task " + r.toString() +
                                             " rejected from " +
                                             e.toString());
    }
}

// 不做任何处理，直接忽略掉这个任务
public static class DiscardPolicy implements RejectedExecutionHandler {
    public DiscardPolicy() { }
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    }
}

// 这个相对霸道一点，如果线程池没有被关闭的话，
// 把队列队头的任务(也就是等待了最长时间的)直接扔掉，然后提交这个任务到等待队列中
public static class DiscardOldestPolicy implements RejectedExecutionHandler {
    public DiscardOldestPolicy() { }
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            e.getQueue().poll();
            e.execute(r);
        }
    }
}
```

到这里，ThreadPoolExecutor 的源码算是分析结束了。单纯从源码的难易程度来说，ThreadPoolExecutor 的源码还算是比较简单的，只是需要我们静下心来好好看看罢了。

## [](#toc_15)Executors

这节其实也不是分析 Executors 这个类，因为它仅仅是工具类，它的所有方法都是 static 的。

*   生成一个固定大小的线程池：

```
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

最大线程数设置为与核心线程数相等，此时 keepAliveTime 设置为 0（因为这里它是没用的，即使不为 0，线程池默认也不会回收 corePoolSize 内的线程），任务队列采用 LinkedBlockingQueue，无界队列。

过程分析：刚开始，每提交一个任务都创建一个 worker，当 worker 的数量达到 nThreads 后，不再创建新的线程，而是把任务提交到 LinkedBlockingQueue 中，而且之后线程数始终为 nThreads。

*   生成只有**一个线程**的固定线程池，这个更简单，和上面的一样，只要设置线程数为 1 就可以了：

```
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

*   生成一个需要的时候就创建新的线程，同时可以复用之前创建的线程（如果这个线程当前没有任务）的线程池：

```
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

核心线程数为 0，最大线程数为 Integer.MAX_VALUE，keepAliveTime 为 60 秒，任务队列采用 SynchronousQueue。

这种线程池对于任务可以比较快速地完成的情况有比较好的性能。如果线程空闲了 60 秒都没有任务，那么将关闭此线程并从线程池中移除。所以如果线程池空闲了很长时间也不会有问题，因为随着所有的线程都会被关闭，整个线程池不会占用任何的系统资源。

过程分析：我把 execute 方法的主体黏贴过来，让大家看得明白些。鉴于 corePoolSize 是 0，那么提交任务的时候，直接将任务提交到队列中，由于采用了 SynchronousQueue，所以如果是第一个任务提交的时候，offer 方法肯定会返回 false，因为此时没有任何 worker 对这个任务进行接收，那么将进入到最后一个分支来创建第一个 worker。之后再提交任务的话，取决于是否有空闲下来的线程对任务进行接收，如果有，会进入到第二个 if 语句块中，否则就是和第一个任务一样，进到最后的 else if 分支创建新线程。

```
int c = ctl.get();
// corePoolSize 为 0，所以不会进到这个 if 分支
if (workerCountOf(c) < corePoolSize) {
    if (addWorker(command, true))
        return;
    c = ctl.get();
}
// offer 如果有空闲线程刚好可以接收此任务，那么返回 true，否则返回 false
if (isRunning(c) && workQueue.offer(command)) {
    int recheck = ctl.get();
    if (! isRunning(recheck) && remove(command))
        reject(command);
    else if (workerCountOf(recheck) == 0)
        addWorker(null, false);
}
else if (!addWorker(command, false))
    reject(command);
```

SynchronousQueue 是一个比较特殊的 BlockingQueue，其本身不储存任何元素，它有一个虚拟队列（或虚拟栈），不管读操作还是写操作，如果当前队列中存储的是与当前操作相同模式的线程，那么当前操作也进入队列中等待；如果是相反模式，则配对成功，从当前队列中取队头节点。具体的信息，可以看我的另一篇关于 BlockingQueue 的文章。

## [](#toc_16)总结

我一向不喜欢写总结，因为我把所有需要表达的都写在正文中了，写小篇幅的总结并不能真正将话说清楚，本文的总结部分为准备面试的读者而写，希望能帮到面试者或者没有足够的时间看完全文的读者。

1.  java 线程池有哪些关键属性？
    
    corePoolSize，maximumPoolSize，workQueue，keepAliveTime，rejectedExecutionHandler
    
    corePoolSize 到 maximumPoolSize 之间的线程会被回收，当然 corePoolSize 的线程也可以通过设置而得到回收（allowCoreThreadTimeOut(true)）。
    
    workQueue 用于存放任务，添加任务的时候，如果当前线程数超过了 corePoolSize，那么往该队列中插入任务，线程池中的线程会负责到队列中拉取任务。
    
    keepAliveTime 用于设置空闲时间，如果线程数超出了 corePoolSize，并且有些线程的空闲时间超过了这个值，会执行关闭这些线程的操作
    
    rejectedExecutionHandler 用于处理当线程池不能执行此任务时的情况，默认有**抛出 RejectedExecutionException 异常**、**忽略任务**、**使用提交任务的线程来执行此任务**和**将队列中等待最久的任务删除，然后提交此任务**这四种策略，默认为抛出异常。
    
2.  说说线程池中的线程创建时机？
    
    1.  如果当前线程数少于 corePoolSize，那么提交任务的时候创建一个新的线程，并由这个线程执行这个任务；
    2.  如果当前线程数已经达到 corePoolSize，那么将提交的任务添加到队列中，等待线程池中的线程去队列中取任务；
    3.  如果队列已满，那么创建新的线程来执行任务，需要保证池中的线程数不会超过 maximumPoolSize，如果此时线程数超过了 maximumPoolSize，那么执行拒绝策略。
    
    * 注意：如果将队列设置为无界队列，那么线程数达到 corePoolSize 后，其实线程数就不会再增长了。因为后面的任务直接往队列塞就行了，此时 maximumPoolSize 参数就没有什么意义。
    
3.  Executors.newFixedThreadPool(…) 和 Executors.newCachedThreadPool() 构造出来的线程池有什么差别？
    
    细说太长，往上滑一点点，在 Executors 的小节进行了详尽的描述。
    
4.  任务执行过程中发生异常怎么处理？
    
    如果某个任务执行出现异常，那么执行任务的线程会被关闭，而不是继续接收其他任务。然后会启动一个新的线程来代替它。
    
5.  什么时候会执行拒绝策略？
    
    1.  workers 的数量达到了 corePoolSize（任务此时需要进入任务队列），任务入队成功，与此同时线程池被关闭了，而且关闭线程池并没有将这个任务出队，那么执行拒绝策略。这里说的是非常边界的问题，入队和关闭线程池并发执行，读者仔细看看 execute 方法是怎么进到第一个 reject(command) 里面的。
    2.  workers 的数量大于等于 corePoolSize，将任务加入到任务队列，可是队列满了，任务入队失败，那么准备开启新的线程，可是线程数已经达到 maximumPoolSize，那么执行拒绝策略。
    

因为本文实在太长了，所以我没有说执行结果是怎么获取的，也没有说关闭线程池相关的部分，这个就留给读者吧。

本文篇幅是有点长，如果读者发现什么不对的地方，或者有需要补充的地方，请不吝提出，谢谢。

（全文完）

## 历史评论 (121)

Tips: 由于评论系统已经采用 Github Issues，所以原来的评论功能不再可用，这里仅仅是备份显示原来的数据

*   ![](https://assets.javadoop.com/headimg/4.png)
    
    oneleaf2024-07-12 17:19:00
    
    // 添加成功的话，启动这个线程 if (workerAdded) { // 启动线程 t.start(); workerStarted = true; }
    
    为啥这里不需要放入锁里面呢？如果这个 t 已经被其他线程启动了呢？
    
*   ![](https://avatars0.githubusercontent.com/u/36094031?v=4)
    
    jorksz2024-04-07 22:13:10
    
    第三次刷，随着工作经验增加，也有了更深的理解。
    
*   ![](https://avatars.githubusercontent.com/u/10616763?v=4)
    
    xiaozhi2024-02-05 16:45:07
    
    反复学习
    
*   ![](https://avatars0.githubusercontent.com/u/36094031?v=4)
    
    jorksz2023-10-18 22:46:40
    
    反复学习
    
*   ![](https://avatars.githubusercontent.com/u/70273033?v=4)
    
    zhangfanMemory2023-09-27 14:54:21
    
    反复学习
    
*   ![](https://avatars.githubusercontent.com/u/75552222?v=4)
    
    fxz2023-09-24 13:26:49
    
    fxz 来过
    
*   ![](https://avatars0.githubusercontent.com/u/55621390?v=4)
    
    suntang13142023-08-07 23:15:54
    
    留下👣
    
*   ![](https://avatars.githubusercontent.com/u/16587845?v=4)
    
    Jack2023-04-02 21:08:25
    
    非常感谢，非常赞
    
*   ![](https://assets.javadoop.com/headimg/2.png)
    
    Zero-ONE2022-10-21 14:16:00
    
    好强大，分析很好，kf3b3qgre 完美
    
*   ![](https://assets.javadoop.com/headimg/19.png)
    
    Eric2022-09-19 16:18:54
    
    Java 11+ addWorker() 最开始的几行代码的策略发生了变化
    
*   ![](https://assets.javadoop.com/headimg/13.png)
    
    fei2022-01-02 22:26:35
    
    有没有一种可能，当前线程数为 n + 1, 核心线程数为 n, 有两个线程同时判断当前线程 > coreSize, timed 为 true，然后 poll 时间到了，同时返回 null，同时执行线程回收动作，导致当前线程数少于 n
    
    *   ![](https://avatars1.githubusercontent.com/u/31086581?v=4)
        
        ZhZGod2022-08-23 17:29:47
        
        if ((wc> maximumPoolSize || (timed && timedOut)) && (wc > 1 || workQueue.isEmpty())) { if (compareAndDecrementWorkerCount(c)) return null; continue; } getTask 里减少线程数里 用 cas 了
        
*   ![](https://assets.javadoop.com/headimg/8.png)
    
    Steven2021-12-15 08:14:03
    
    作者分析的很细致，其实可以出一本掘金小册子，让更多的人看到
    
*   ![](https://avatars0.githubusercontent.com/u/36094031?v=4)
    
    jorksz2021-12-11 11:57:24
    
    mark, 过了一遍，还需要好好吸收
    
*   ![](https://assets.javadoop.com/headimg/1.png)
    
    xiaoger2021-12-10 15:29:26
    
    牛皮
    
*   ![](https://assets.javadoop.com/headimg/10.png)
    
    TW2021-07-09 16:53:22
    
    看完了，害得花时间消化消化，讲的特好！
    
*   ![](https://avatars.githubusercontent.com/u/19384323?v=4)
    
    朝阳 2021-06-22 07:50:13
    
    ```
    if (! isRunning(recheck) && remove(command))
            reject(command);
        // 1
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    ```
    
    感谢大佬分享，请教一下 1 处线程数 ==0 才去创建非核心线程，这块没有理解，不是应该 workerCount < 非核心线程数就创建非核心线程吗
    
    *   ![](https://avatars.githubusercontent.com/u/30865646?v=4)
        
        EricWong2022-04-25 10:23:23
        
        /* * Proceed in 3 steps: * * 1. If fewer than corePoolSize threads are running, try to * start a new thread with the given command as its first * task. The call to addWorker atomically checks runState and * workerCount, and so prevents false alarms that would add * threads when it shouldn't, by returning false. * * 2. If a task can be successfully queued, then we still need * to double-check whether we should have added a thread * (because existing ones died since last checking) or that * the pool shut down since entry into this method. So we * recheck state and if necessary roll back the enqueuing if * stopped, or start a new thread if there are none. * * 3. If we cannot queue task, then we try to add a new * thread. If it fails, we know we are shut down or saturated * and so reject the task. */
        
    *   ![](https://avatars.githubusercontent.com/u/30865646?v=4)
        
        EricWong2022-04-25 10:34:26
        
        这块的整体代码是有前置条件的，首先 ctl 值是代表线程池的运行状态 & 工作线程数数 这个 ctl 源码里面有一大块的注释，可以先看一下然后试着理解。
        
        *   running 状态 并且向 workeQuene 中添加了任务成功，存在并发的情况，`java int recheck = ctl.get();` recheck 是又获取了一遍线程池的 ctl， ==0 代表线程池中还没有工作线程，所以初始化了一个非核心线程来保证线程池当中最少有一个活跃线程
        
*   ![](https://assets.javadoop.com/headimg/19.png)
    
    yang2021-03-03 19:00:45
    
    作者强啊 如果整理成一本 pdf 的话 就好了
    
*   ![](https://assets.javadoop.com/headimg/18.png)
    
    bo2020-12-18 16:49:24
    
    真的好
    
*   ![](https://assets.javadoop.com/headimg/7.png)
    
    哈哈 2020-10-10 20:05:53
    
    ~ 意味着按位取反
    
*   ![](https://assets.javadoop.com/headimg/11.png)
    
    bvcn2020-08-23 09:11:04
    
    买单号，上单号网 [www.danhw.com](http://www.danhw.com)
    
*   ![](https://assets.javadoop.com/headimg/16.png)
    
    chenwenjie2020-07-20 11:45:39
    
    图好像裂了
    
    *   ![](https://avatars3.githubusercontent.com/u/33995610?v=4)
        
        mimajiushi2020-07-20 11:51:36
        
        [https://www.javadoop.com/post/java-thread-pool](https://www.javadoop.com/post/java-thread-pool) 访问图没事，[https://javadoop.com/post/java-thread-pool](https://javadoop.com/post/java-thread-pool) 访问图就没了
        
*   ![](https://avatars1.githubusercontent.com/u/64183978?v=4)
    
    boychenfei2020-07-08 10:54:43
    
    博主你好： RUNNING 定义为 -1，SHUTDOWN 定义为 0，其他的都比 0 大，所以等于 0 的时候不能提交任务，大于 0 的话，连正在执行的任务也需要中断。 RUNNING 的值是 3 个 1 和 29 个 0 为什么说 RUNNING 定义是 - 1 呢？
    
    *   ![](https://avatars3.githubusercontent.com/u/10348256?v=4)
        
        zhaoyd2020-08-15 15:19:48
        
        -1 的二进制是 11111111111111111111111111111111（32 个 1） RUNNING = -1 << 29 也就是 11100000000000000000000000000000（3 个 1、29 个 0）
        
*   ![](https://avatars3.githubusercontent.com/u/28783897?v=4)
    
    chenshuaikang2020-06-30 13:37:27
    
    博主，可以转载你这篇文章吗
    
*   ![](https://assets.javadoop.com/headimg/15.png)
    
    天上蹲 2020-06-08 15:31:45
    
    谢谢博主
    
*   ![](https://assets.javadoop.com/headimg/8.png)
    
    appreciate2020-06-06 16:52:51
    
    把队列队头的任务 (也就是等待了最长时间的) 直接扔掉，然后提交这个任务到等待队列中 public static class DiscardOldestPolicy implements RejectedExecutionHandler { } 这一块我理解的是抛弃掉队列中头一个任务( e.getQueue().poll();)，也就是抛弃下一个即将执行的任务，这个等待时间最长的任务是不是有问题？
    
    *   ![](https://avatars.githubusercontent.com/u/33395907?v=4)
        
        starkZH2021-02-23 11:04:55
        
        我也觉得有点问题，应该不是等待时间最长的吧
        
*   ![](https://assets.javadoop.com/headimg/5.png)
    
    abc2020-05-15 02:13:09
    
    不得不承认，这比某些工具书有深度多了。看完就一个字，痛快，透彻
    
*   ![](https://avatars3.githubusercontent.com/u/26666752?v=4)
    
    zouyu2020-05-15 00:48:58
    
    应届生六月份就要去公司报到了，看博主的文章看得是很爽啊，不知道到时候工作能不能经常用到这些知识。
    
*   ![](https://assets.javadoop.com/headimg/11.png)
    
    苏苏 2020-03-29 14:27:26
    
    对于 newCachedThreadPool() 那段分析我有点疑惑。newCachedThreadPool() 既然使用了 SynchronousQueue，根据源码，其实是使用的 SynchronousQueue 中的非公平模式 TransferStack，那么下面第二个 if 语句, workQueue.offer(command) 不应该是一直失败的吗？
    
    ```
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    ```
    
    因为根据 SynchronousQueue 的 offer(E e) 方法，其实是调用了 transfer.transfer(e, true, 0)，点进去来到 TransferStack 的 transfer(E e, boolean timed, long nanos) 方法，对应下面这一段代码
    
    ```
    for (;;) {
                    SNode h = head;
                    if (h == null || h.mode == mode) {  
                        if (timed && nanos <= 0) {      //一直会进到这里来，然后判断之后返回null值
                            if (h != null && h.isCancelled())
                                casHead(h, h.next);     
                            else
                                return null;
                        } else if (casHead(h, s = snode(s, e, h, mode))) {
                            ...
                        }
                    } else if (!isFulfilling(h.mode)) { 
                        ...
                    } else {                            
                        ...
                    }
                }
    ```
    
    所以 workQueue.offer(command) 会一直返回 false。所以对于 cachedPool，会一直进到这里来创建 worker。
    
    ```
    else if (!addWorker(command, false))
                reject(command);
    ```
    
    不知道我的理解是不是错了...
    
    *   ![](https://avatars1.githubusercontent.com/u/45305519?v=4)
        
        Kyrie172020-04-09 11:01:36
        
        层主有理解了吗，我现在也和你一样感觉是不断创建新线程，不会复用诶。
        
        *   ![](https://avatars1.githubusercontent.com/u/45305519?v=4)
            
            Kyrie172020-04-10 17:21:59
            
            哦哦，理解了，getTask() 中
            
            ```
            ...
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();
            ...
            ```
            
            在线程空闲的时候会插入第一个取的任务结点。
            
    *   ![](https://avatars.githubusercontent.com/u/33395907?v=4)
        
        starkZH2021-02-23 11:47:19
        
        创建完的 worker 会在 getTask 循环 workQueue.take，此时再 offer 任务进来就会成功了
        
*   ![](https://assets.javadoop.com/headimg/11.png)
    
    苏苏 2020-03-29 11:55:47
    
    getTask() 源码跟我的不一样，我是 jdk1.8_131，楼主上面的是哪个版本的源码
    
*   ![](https://avatars1.githubusercontent.com/u/29819766?v=4)
    
    wangboxes2020-03-10 11:25:07
    
    相见恨晚啊
    
*   ![](https://avatars2.githubusercontent.com/u/30142368?v=4)
    
    doubleyao2019-10-31 17:38:29
    
    非常赞
    
*   ![](https://assets.javadoop.com/headimg/8.png)
    
    wade2019-10-03 00:55:24
    
    赞
    
*   ![](https://assets.javadoop.com/headimg/15.png)
    
    d2019-09-25 18:33:02
    
    如果某个任务执行出现异常，那么执行任务的线程会被关闭，而不是继续接收其他任务。然后会启动一个新的线程来代替它。
    
    有个地方好像有点问题，在 runWorker 中，task 并不会抛出异常，因为 task 是 FutureTask，它把异常都 catch 然后吞掉了，所以这里的一堆 catch 没有用，异常需要处理的话，是在`afterExecute(task, thrown);`里面处理的。然后由于并不会进到这几个 catch 里面，所以也不会开一个新的线程来代替它。
    
    *   ![](https://avatars1.githubusercontent.com/u/14758664?v=4)
        
        HongJie2019-09-29 13:34:00
        
        你说的确实有道理，回头我再仔细看一下，为什么这里要捕获异常。
        
        因为 Doug Lea 的代码，几乎总是完美的，他写 catch 的地方，一定是有原因的。
        
    *   ![](https://avatars2.githubusercontent.com/u/39616107?v=4)
        
        wezhyn2019-10-20 18:17:56
        
        ThreadPoolExecutor 并非所有 task 都是通过 `submit()`来提交的，还可以直接运行 `execute(Runnable)`, 这个时候 task 就并非是可靠的
        
*   ![](https://assets.javadoop.com/headimg/2.png)
    
    hiker2019-08-22 11:49:57
    
    感谢大佬。 打卡，翻来覆去的看了看多边，能力有限算是马马虎虎理解了。 个人愚见啊，大佬要是把 processWorkerExit(Worker w, boolean completedAbruptly) 方法的解释也写出来 ，感觉会更加完整，或者这么说，读者能把这个方法也仔细读读，能更好的理解整个池子的运行流程。因为我是读了那个方法才理解了，线程的回收情况，和 corePoolSize，maximumPoolSize 的真正含义。
    
*   ![](https://avatars0.githubusercontent.com/u/24283818?v=4)
    
    houxudong2019-06-17 17:58:41
    
    感谢作者！比某些人出的书好太多了。
    
*   ![](https://avatars0.githubusercontent.com/u/22957350?v=4)
    
    宋维飞 2019-05-20 16:12:48
    
    太厉害了 点赞
    
*   ![](https://assets.javadoop.com/headimg/7.png)
    
    Hello_nogy2019-04-13 23:53:02
    
    原来上上个回答就有，哈哈哈哈哈，懂了
    
*   ![](https://assets.javadoop.com/headimg/11.png)
    
    Hello_nogy2019-04-13 23:50:34
    
    java.util.concurrent.ThreadPoolExecutor#runWorker 请教一下，这个方法里的 Thread wt 和 w.thread 不应该是同一个线程吗？下面在 while 循环中 w.lock(); 的意义是什么？反正都在一个线程里
    
*   ![](https://assets.javadoop.com/headimg/5.png)
    
    ciao2019-04-12 13:12:36
    
    为啥创建线程池的时候不马上创建线程，而是等任务来一个就创建线程？ 为什么不是马上把线程创建好放到池子里面，连接池不就是这样的么？
    
    *   ![](https://avatars1.githubusercontent.com/u/14758664?v=4)
        
        HongJie2019-04-12 14:15:48
        
        你的想法也没错，这么搞是为了节约资源，另外，线程池提供了以下方法满足你的需求：
        
        ```
        public boolean prestartCoreThread() {
             return workerCountOf(ctl.get()) < corePoolSize &&
                addWorker(null, true);
        }
        ```
        
        *   ![](https://avatars0.githubusercontent.com/u/37655156?v=4)
            
            Ciao2019-04-18 09:27:19
            
            我问了别的博主，说平时开发也不调用这个方法，那不是第一次用的线程池的时候不很慢么？ 还有当开启的线程数大于核心线程数并且小于最大线程数时，不是就开始创建新的线程放到线程池里吗。超过时间单位后就销毁最大线程数减去核心线程数的线程数量么？ 怎么是超过核心线程数后 不是创建新线程去执行任务 还是把任务放到阻塞队列。。 等核心线程有空闲的时候再去阻塞队列拿？ 只跟我之前理解的不一样呀。。。
            
            *   ![](https://avatars1.githubusercontent.com/u/14758664?v=4)
                
                HongJie2019-04-18 10:19:30
                
                似乎你对 Java 线程池的理解有点偏差，你可以尝试先看完源码，然后再提出你的疑问。
                
                另外，`prestartCoreThread()` 在很多地方都会用到，一般会在应用初始化的时候完成。
                
*   ![](https://assets.javadoop.com/headimg/18.png)
    
    Deacon2019-04-10 14:20:30
    
    建议博主加上线程池关闭相关方法的解析，比如 shutdown 和 shutdownNow，因为我看源码的时候有个位置没搞明白，就是在 runWorker 方法里面调用 getTask 从 workQueue 中拿到任务之后，为什么要使用 w.lock 将任务的执行过程加锁 (博主的文章中没有强调) runWorker 部分代码：
    
    while (task != null || (task = getTask()) != null) { w.lock(); // 加锁
    
    后来在网上找了几篇文章都说到了线程池的关闭策略，这两者结合在一起就比较好理解了，简单而言就是 Worker 本身就是一把独占非重入锁，shutdown 方法无法中断正在执行任务中的线程 (该线程已经调用了 w.lock 拿到了独占状态，shutdown 方法在中断线程之前会调用 w.tryLock，此时 tryLock 会失败，无法发送中断信号)，而 shutdownNow 则比较粗暴，不管线程是否正在执行任务，都会发起中断请求，这也符合接口对两个方法语义描述
    
    public void shutdown() { final ReentrantLock mainLock = this.mainLock; mainLock.lock(); try {checkShutdownAccess(); advanceRunState(SHUTDOWN); // 中断空闲线程，空闲线程就是等待从任务队列中拿任务的线程，即没有获取到 Worker 锁 interruptIdleWorkers(); onShutdown(); // hook for ScheduledThreadPoolExecutor } finally { mainLock.unlock(); } tryTerminate(); }
    
    // 中断空闲线程 private void interruptIdleWorkers(boolean onlyOne) { final ReentrantLock mainLock = this.mainLock; mainLock.lock(); try { for (Worker w : workers) { Thread t = w.thread; if (!t.isInterrupted() && w.tryLock()) { //w.tryLock 尝试获取锁，成功之后才会发起中断请求 try { t.interrupt(); } catch (SecurityException ignore) { } finally { w.unlock(); } } if (onlyOne) break; } } finally { mainLock.unlock(); } }
    
*   ![](https://assets.javadoop.com/headimg/9.png)
    
    wbice2019-04-03 20:59:03
    
    lz 讲的太细了，干货满满，纠正了我之前一些错误的观点，比如认为先创建线程到 max 个数后，才将 task 添加到队列中。
    
*   ![](https://avatars0.githubusercontent.com/u/33517683?v=4)
    
    adv123456782019-04-03 08:55:55
    
    invokeAll 方法里这个注释好像错了，是提交所有任务，且每提交一个任务就检查有没有超时 // 提交一个任务，检测一次是否超时 while (it.hasNext()) { execute((Runnable)(it.next())); long now = System.nanoTime(); nanos -= now - lastTime; lastTime = now; // 超时 if (nanos <= 0) return futures; }
    
    *   ![](https://avatars1.githubusercontent.com/u/14758664?v=4)
        
        HongJie2019-04-03 09:07:15
        
        不是注释错误，是表述方式的问题，因为这段代码的关注点在 **超时** 上。
        
*   ![](https://avatars2.githubusercontent.com/u/3206693?v=4)
    
    ygmyth2019-04-02 16:12:38
    
    嗯, 看到了. 应该是这里 ```  
    Worker(Runnable firstTask) { setState(-1); // inhibit interrupts until runWorker this.firstTask = firstTask; this.thread = getThreadFactory().newThread(this); }
    
    ```
    传给线程的runnable 是 当前worker
    ```
    
*   ![](https://avatars2.githubusercontent.com/u/3206693?v=4)
    
    ygmyth2019-04-02 15:40:03
    
    回过头来，继续往下走。我们知道，worker 中的线程 start 后，其 run 方法会调用 runWorker 方法：
    
    ```
    // Worker 类的 run() 方法
    public void run() {
        runWorker(this);
    }
    ```
    
    在 addWorker 中调用的是 worker 的 thread t.start 方法, 调用 runWorker 的 run 方法是 Woker 类的, 为什么 t.start 后会调用 Worker.run 方法呢
    
    *   ![](https://avatars1.githubusercontent.com/u/14758664?v=4)
        
        HongJie2019-04-02 16:05:06
        
        `t.start()` 会启动线程 t，然后执行其 run() 方法
        
    *   ![](https://avatars.githubusercontent.com/u/33395907?v=4)
        
        starkZH2021-02-23 11:49:57
        
        因为 worker 在创建线程时是将自己作为 Runnable 任务传递给线程的。所以线程在 start 后会调用 worker 的 run 方法。
        
        ```
        Worker(Runnable firstTask) {
                    setState(-1); // inhibit interrupts until runWorker
                    this.firstTask = firstTask;
                    this.thread = getThreadFactory().newThread(this);
                }
        ```
        
*   ![](https://assets.javadoop.com/headimg/7.png)
    
    大王 2019-03-08 16:45:57
    
    如果当前线程数已经达到 corePoolSize，那么将提交的任务添加到队列中，等待线程池中的线程去队列中取任务. 这段没怎么搞懂
    
    *   ![](https://avatars1.githubusercontent.com/u/14758664?v=4)
        
        HongJie2019-03-08 17:03:44
        
        一定是因为你没好好看完
        
*   ![](https://assets.javadoop.com/headimg/8.png)
    
    DTWENMi2019-02-27 01:25:26
    
    厉害，厉害
    
*   ![](https://assets.javadoop.com/headimg/8.png)
    
    okali2019-01-05 04:34:50
    
    很赞
    
*   ![](https://assets.javadoop.com/headimg/4.png)
    
    hanc2018-12-30 07:29:47
    
    try { // 到 workQueue 中获取任务 Runnable r = timed ? workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) : workQueue.take(); if (r != null) return r; timedOut = true; } catch (InterruptedException retry) { // 如果此 worker 发生了中断，采取的方案是重试 // 解释下为什么会发生中断，这个读者要去看 setMaximumPoolSize 方法， // 如果开发者将 maximumPoolSize 调小了，导致其小于当前的 workers 数量， // 那么意味着超出的部分线程要被关闭。重新进入 for 循环，自然会有部分线程会返回 null timedOut = false; } } 这个地方的 InterruptedException 异常应该是 queue 的 pool 和 take 方法用的都是可中断锁的原因吧。
    
    *   ![](https://avatars1.githubusercontent.com/u/22166069?v=4)
        
        mujianping2021-03-30 17:27:40
        
        worker 线程从阻塞队列取 task 的时候如果发生中断的话, pool 和 take 方法会响应中断并抛异常, 指的是 worker 线程
        
*   ![](https://assets.javadoop.com/headimg/12.png)
    
    辉火流星 2018-12-21 00:34:03
    
    看了很多次 还是有些地方 不明白 ，不过收获颇多 ，感谢作者
    
*   ![](https://assets.javadoop.com/headimg/13.png)
    
    123123123122018-12-18 06:59:53
    
    12312312312312321312
    
*   ![](https://assets.javadoop.com/headimg/20.png)
    
    靖远小师兄 2018-12-12 06:38:44
    
    博主很细心啊, 我一步一步看到这个里来的, 先看的 aqs 写的很不错, 现在基本上只是看了个大概, 很多东西理解不深, 后面慢慢消化. 非常感谢
    
*   ![](https://assets.javadoop.com/headimg/10.png)
    
    小 2018-11-29 11:04:22
    
    // 如果失败，说明当前线程数已经达到 maximumPoolSize，执行拒绝策略 else if (!addWorker(command, false)) reject(command); 这里是不是少了一种情况, 如果在 addworker 中线程池的状态被其他线程改成 > shutdown 是不是也会返回 false
    
*   ![](https://assets.javadoop.com/headimg/12.png)
    
    辉火流星 2018-10-12 08:11:50
    
    博主厉害极了
    
*   ![](https://assets.javadoop.com/headimg/14.png)
    
    guzhan2018-09-30 05:40:16
    
    感谢楼主的分析。
    
*   ![](https://assets.javadoop.com/headimg/13.png)
    
    我不吃甜食 2018-09-30 05:38:44
    
    博主你好，看完上文后我还有一个问题。Worker 设计为 extends AQS，并且是非 可重入的，这样的设计初衷我没太理解；按照 api doc，设计为锁是为了保护正在运行任务的 Thread 不被中断，而是那些 idle 的 Thread 才能中断；但是我觉得在 runWorker()方法里，当一个 Thread 获取到任务 (getTask()!=null) 并开始运行这个任务后，无论外界怎样操作，比如调用 setCorePoolSize() 中断了该线程 (因为不可重入，实际上无法中断这个 Thread)，该线程也没有去响应这个中断呀，能响应中断的 Thread 都是在 getTask() 中阻塞的线程。所以我一直觉得将 Worker 设计为锁，作用不大；不知这个怎样理解？
    
    *   ![](https://assets.javadoop.com/headimg/13.png)
        
        我不吃甜食 2018-09-30 07:15:05
        
        我又想了想，理解了😂
        
        *   ![](https://assets.javadoop.com/headimg/20.png)
            
            不候鸟 2018-11-01 08:39:19
            
            worker 的意义主要是为了每一个工作的线程加一个单独的锁把，如果专门创建一个 worker 类，里面包含一个自定义的独占锁和 thread 类，也是一样的效果？
            
        *   ![](https://assets.javadoop.com/headimg/12.png)
            
            NewHongjay2018-11-23 02:12:24
            
            这里我也不太懂，为什么要继承 AQS，麻烦能解答下吗？谢谢
            
            *   ![](https://assets.javadoop.com/headimg/13.png)
                
                我不吃甜食 2018-12-07 09:41:21
                
                我的理解是这样的：中断一个线程只是将这个线程的中断标记位设置了一下，该线程可以不理会这个中断 (也可能完成任务后检查这个标记位，若打标记，此处会影响线程的行为)。所以中断一个正在运行的线程，实际上对这个正在运行的线程没有大的影响，但是 java api doc 里建议不要给正在运行的线程设置中断标记位，这里的锁我觉得是这个规范的作用，这里实际的行为不管有没有锁都一样。
                
                *   ![](https://assets.javadoop.com/headimg/4.png)
                    
                    hanc2019-01-01 04:12:33
                    
                    这是因为外部有可能会在线程池中调用其他方法中断 Worker，为了保证任务完整被执行，必须在获得锁的情况下才能中断 worker。 interruptIdleWorkers(shutdown 以及回收空闲 worker 线程时候会调用 例如 tryTerminate()方法)，会尝试获取锁 (tryLock) 中获取锁然后中断线程，如果获取不到锁 也就不会中断线程，让任务执行完成。 interruptWorkers 方法在 shutdownNow 方法中会调用，也就是状态从 shutdown ->> stop，如果线程不处于中断状态会中断线程。 上面说的中断线程都只是将线程声明为中断状态，然后在 getTask()方法中会有一个捕获 InterruptedException 的异常然后将超时状态变为 false 然后回收线程。 详细的可以看看代码哈
                    
        *   ![](https://assets.javadoop.com/headimg/2.png)
            
            老王子 2018-11-23 10:20:03
            
            没理解为什么需要设计成锁？能否解释一下?
            
            *   ![](https://assets.javadoop.com/headimg/13.png)
                
                我不吃甜食 2018-12-07 09:41:30
                
                同上
                
            *   ![](https://avatars.githubusercontent.com/u/33395907?v=4)
                
                starkZH2021-02-23 12:06:26
                
                个人感觉应该是为了严谨吧。加锁之后可以保证一个 worker 同一时刻只会被一个线程调用，同时也可以保证对 completedTasks 操作的原子性。
                
*   ![](https://assets.javadoop.com/headimg/7.png)
    
    janghao2018-09-06 07:24:37
    
    在代码中 atomicinteger get 之后通常会进入一个判断流程在这之间 如果被其他线程修改值之后不会有问题吗 这样即使是 atomicinteger 属性是 volatile 修饰也不会起作用的 虽然我也认为线程池本身是线程安全但是还是理解不上去
    
*   ![](https://assets.javadoop.com/headimg/16.png)
    
    jianghao2018-09-06 05:41:15
    
    首先感谢博主的讲解，我有一个问题 ThreadPoolExecutor 这个类怎么保重的线程安全？类中加锁的粒度很细，会不会造成线程不同步问题？虽然代码中线程数和线程状态用 AtomicInteger 来控制，但是它的值是不断在变化和不断的重新读取值，多线程调用时难免会造成数据不一致问题？ 语言组织的不好，见谅
    
*   ![](https://assets.javadoop.com/headimg/16.png)
    
    jianghao2018-08-30 05:40:07
    
    博主，请问可以转发吗？
    
*   ![](https://assets.javadoop.com/headimg/11.png)
    
    多普 2018-08-26 03:29:20
    
    大佬 , 我想把这篇打印成 pdf , ctrl+p 出来的图片很不清楚 可不可以调整呀...
    
*   ![](https://assets.javadoop.com/headimg/19.png)
    
    悟空 2018-08-17 22:40:54
    
    博主你好！“对创建线程的错误理解：如果线程数少于 corePoolSize，创建一个线程，如果线程数在 [corePoolSize, maximumPoolSize] 之间那么可以创建线程或复用空闲线程，keepAliveTime 对这个区间的线程有效。” 从上面的几个分支，我们就可以看出，上面的这段话是错误的。 看了几遍就是没发现那里错的，麻烦博主解释一下。谢谢
    
    *   ![](https://avatars0.githubusercontent.com/u/30643093?v=4)
        
        阿狸 2021-01-06 17:26:35
        
        如果线程数在 [corePoolSize, maximumPoolSize] 之间那么我们应该判断队列的状态吧！
        
    *   ![](https://avatars.githubusercontent.com/u/52889249?v=4)
        
        orzwalker2022-11-29 02:18:12
        
        *   1. 少于 core，创建新线程执行任务
        *   2. 大于 core，任务入队列等待执行（此时不创建多余线程）
        *   3. **入队列失败，再创建新线程执行任务**，并不是说 [core, max] 的线程是来一个任务就创建一个线程
        
*   ![](https://assets.javadoop.com/headimg/9.png)
    
    melanch2018-08-07 01:33:49
    
    楼主好文, 整个博客都很好. 点赞
    
*   ![](https://assets.javadoop.com/headimg/5.png)
    
    link2018-08-04 05:59:53
    
    为博主点赞
    
*   ![](https://assets.javadoop.com/headimg/7.png)
    
    Nthan2018-04-11 06:26:50
    
    大佬，为什么 ThreadPoolExecutor 的 submit(Runnable task, T result) 方法，为什么 task 是 Runnable 的，还需要设置 result？这个会在什么场景使用呢？你也说到需要获取结果（FutureTask），用 submit 方法，不需要获取结果，可以用 execute 方法。但为什么 AbstractExecutorService#newTaskFor(Runnable runnable, T value) 会有这个方法呢？
    
    *   ![](https://assets.javadoop.com/headimg/17.png)
        
        算算算算了吧 2018-12-07 00:09:09
        
        我提一个思路，不一定正确：
        
        1、纯粹用 execute() ，如果不增加其他消息通知的机制，上层是不知道这个任务执行到什么程度，但是如果使用 submit(Runnable task, T result)，那么上层可以有机会得知任务执行到什么程度。
        
        2、execute() 跟 submit(Runnable task）没什么区别，只是额外提供多了一个能力。
        
        3、猜测：execute() 作者设计会 public ，可能更注重于允许用户重写，而非直接调用。
        
*   ![](https://assets.javadoop.com/headimg/17.png)
    
    小小白 2018-04-02 06:11:53
    
    有个疑问哈~ 执行拒绝策略的第一个时机写到：“workers 的数量达到了 corePoolSize（任务此时需要进入任务队列），任务入队成功，与此同时线程池被关闭了，而且关闭线程池并没有将这个任务出队，那么执行拒绝策略。” 但是，前面 execut 代码里面的注释： // 如果线程池已不处于 RUNNING 状态，那么移除已经入队的这个任务，并且执行拒绝策略 意思是出队成功，然后执行拒绝策略的
    
    感觉有一点点矛盾，我的理解是：只要线程池被关闭就要执行拒绝策略，但是在执行之前先将该任务出队。任务没有出队应该不是执行拒绝策略的原因吧 希望楼主大佬解惑~
    
*   ![](https://assets.javadoop.com/headimg/17.png)
    
    小小白 2018-03-31 06:00:53
    
    太赞啦~ 向楼主学习！
    
*   ![](https://assets.javadoop.com/headimg/4.png)
    
    xuefei.lin2018-03-26 03:19:18
    
    博主威武！！！
    
*   ![](https://assets.javadoop.com/headimg/9.png)
    
    余谦 2018-03-23 00:32:03
    
    假如 core 线程数为 5，这 5 个线程全部启动，之后没有任务，那么是不是这个五个线程就一直在 getTask 方法（死循环）里面，一直占用 cpu 资源？
    
    *   ![](https://assets.javadoop.com/headimg/13.png)
        
        我不吃甜食 2018-09-30 02:24:53
        
        是 park 了，不是死循环；此时是不会消耗 cpu 资源的；
        
        *   ![](https://assets.javadoop.com/headimg/17.png)
            
            算算算算了吧 2018-12-06 23:54:42
            
            park 应该是阻塞的意思吧？ 我理解 getTask（） 过程中，这几条线程取不到任务会一直阻塞。
            
            *   ![](https://assets.javadoop.com/headimg/13.png)
                
                我不吃甜食 2018-12-07 09:28:39
                
                是阻塞，和 sleep 的效果一样
                
*   ![](https://assets.javadoop.com/headimg/9.png)
    
    王 2018-03-16 03:25:42
    
    博主，请教图片的类图效果怎么出来的。谢谢。
    
*   ![](https://assets.javadoop.com/headimg/14.png)
    
    Leo2018-03-10 04:25:48
    
    刚接触线程池，在公司，从 10 点多，断断续续看到了现在，觉得懂了个大概 6,7 成，以后会继续关注，给博主一个赞
    
    *   ![](https://avatars3.githubusercontent.com/u/48586563?v=4)
        
        naughtyman2019-10-28 13:27:49
        
        看 6 个小时也是牛皮
        
*   ![](https://assets.javadoop.com/headimg/6.png)
    
    英强 2018-03-07 08:39:54
    
    是我没看仔细。。。没问题的。。。?
    
*   ![](https://assets.javadoop.com/headimg/6.png)
    
    英强 2018-03-07 08:38:19
    
    首先，感谢作者分享，写得真的好。然后，挑个刺这注释是不是有问题
    
    // 两种可能 // 1. rs == SHUTDOWN && workQueue.isEmpty() // 2. rs >= STOP if (rs>= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) { ```
    
    *   ![](https://assets.javadoop.com/headimg/14.png)
        
        ling2018-09-28 07:56:55
        
        rs >= SHUTDOWN && (rs>= STOP || workQueue.isEmpty()) => (rs >= SHUTDOWN && rs >= STOP) || (rs >= SHUTDOWN && workQueue.isEmpty()) => (rs >= STOP) || (rs >= SHUTDOWN && workQueue.isEmpty())
        
*   ![](https://assets.javadoop.com/headimg/7.png)
    
    xupeng.zhang2018-03-06 23:54:20
    
    可以啊
    
*   ![](https://assets.javadoop.com/headimg/5.png)
    
    zxp2018-03-05 13:33:37
    
    博主，请教个问题，讲 Executors.newFixedThreadPool 那里，讲 keepAliveTime(keepAliveTime 设置为 0（因为这里它是没用的，即使不为 0，线程池默认也不会回收 corePoolSize 内的线程)；然后讲 Executors.newCachedThreadPool 那里，如果线程空闲了 60 秒都没有任务，那么将关闭此线程并从线程池中移除。同样底层都是用 ThreadPoolExecutor，只是用的队列不同，但是 keepAliveTime 的效果却不一样？想问一下线程是什么时候会被回收？
    
    *   ![](https://assets.javadoop.com/headimg/17.png)
        
        算算算算了吧 2018-12-06 23:50:26
        
        回收的核心主要是在 getTask() 里面。
        
        如果 return null；这条线程就执行完了。
        
        文章里面也说了，有两种情况会 return null。
        
        一个是线程数目是否大于最大容量，另一个是是否发生超时。你提到的 Executors.newFixedThreadPool 跟 Executors.newCachedThreadPool，在 getTask() 里面主要体现为 timed 的不一样。
        
        所以一个的线程永远不被回收（不设置 allowCoreThreadTimeOut（true））, 另外一个超时会被回收掉。
        
*   ![](https://assets.javadoop.com/headimg/11.png)
    
    newmicro2018-02-15 01:47:42
    
    首先，大赞博主的文章！昨天刚发现这个博客，正在持续学习中！然后，想讨论下 doInvokeAny 中 active == 0 分支存在的问题：博主说 "因为我认为 active 为 0 的情况，必然从下面的 f.get() 返回了"，但是 f.get() 可能抛出异常，此时就会导致循环继续被执行而不是 return，如果没有 active == 0 分支进行 break 可能出现死循环。不知理解的是否正确。
    
*   ![](https://assets.javadoop.com/headimg/6.png)
    
    小木 2018-01-19 14:21:39
    
    为博主点赞
    
*   ![](https://assets.javadoop.com/headimg/16.png)
    
    MindHacks03222018-01-19 07:03:25
    
    写的挺有深度的，看了好几遍，每遍都有新收获！
    
*   ![](https://assets.javadoop.com/headimg/3.png)
    
    徐涛 2018-01-14 10:24:37
    
    博主，你的文章我都很喜欢，从中学到了很多东西，希望自己能像你一样，用心做事写文章。
    
*   ![](https://assets.javadoop.com/headimg/4.png)
    
    bin2018-01-14 06:28:12
    
    博主的线程池部分和 spring 源码都很认真的看了，讲的比书上通俗易懂，但是重要的点都有讲到，真的很 nice，是我见过最好的博客之一了
    
*   ![](https://assets.javadoop.com/headimg/12.png)
    
    陈澄 2018-01-06 07:04:20
    
    非常棒，博主辛苦了
    
*   ![](https://assets.javadoop.com/headimg/1.png)
    
    zdd2017-12-29 03:45:38
    
    干货啊
    
*   ![](https://assets.javadoop.com/headimg/19.png)
    
    刘启航 2017-12-21 13:40:00
    
    终于找到一个良心博客站了
    
*   ![](https://assets.javadoop.com/headimg/6.png)
    
    唐尤华 2017-12-17 03:04:41
    
    非常赞的文章, 楼主辛苦了
    
*   ![](https://assets.javadoop.com/headimg/3.png)
    
    Crow2017-11-07 14:01:31
    
    这才是真正的干货
    
*   ![](https://assets.javadoop.com/headimg/13.png)
    
    未晓 2017-11-03 11:01:45
    
    我竟然看完了?
    
*   ![](https://assets.javadoop.com/headimg/2.png)
    
    Catyuki2017-10-30 05:16:46
    
    辛苦作者了 您的文章都是好文, 长虽长, 但反复看几遍后, 只能送上 1024 个赞
    

展开更多历史评论