---
source: https://blog.csdn.net/LCBUSHIHAHA/article/details/125954347
create: 2024-02-19 14:19
read: true
knowledge: true
knowledge-date: 2025-10-19
tags:
  - Java
  - 框架原理
summary: "[[Guava RateLimiter 实现]]"
---

# 常见限流算法 与 Guava RateLimiter 实现_guava 滑动窗口 - CSDN 博客

## 1. 常见的限流算法

### 1.1. 固定窗口计数限流、

固定窗口限流是指定一个时间段，规定固定时间段内允许的请求数，这个时间段超过指定请求数后就会开始限流。但是两个窗口临界的地方会出现允许流量暴增的情况。在每分钟允许 100 个请求的情况下，在 1 分钟时间点，发生了 200 个请求。  

![](https://img-blog.csdnimg.cn/37cf92e79a6244009a65bfed85b42c25.png)

### 1.2. 滑动窗口计数限流

为了解决临界点的问题，使用滑动的窗口来计数。窗口滑动的间隔越短，时间窗口的临界突变问题发生的概率也就越小，不过只要有时间窗口的存在，还是有可能发生时间窗口的临界突变问题。

![](https://img-blog.csdnimg.cn/b3af5eb2a5b9437d95ef850a1a59668a.png)

### 1.3. 漏桶算法

往漏桶中以任意速率流入水，以固定的速率流出水。当水超过桶的容量时，会被溢出，也就是丢弃。因为桶的容量不变，保证了整体的速率。  
在正常流量下漏铜算法没有问题，**但是面对突发流量，漏铜还是固定速率就不是很合理，这个时候我们是希望系统尽量快的处理请求，提升用户体验**（在不影响系统正常运行的情况下）。  

![](https://img-blog.csdnimg.cn/2f1c7e04265b4d50bcc548ad5e6abdfe.png)

### 1.4. [令牌桶算法](https://so.csdn.net/so/search?q=%E4%BB%A4%E7%89%8C%E6%A1%B6%E7%AE%97%E6%B3%95&spm=1001.2101.3001.7020)

根据限流大小，以固定的速率往[令牌桶](https://so.csdn.net/so/search?q=%E4%BB%A4%E7%89%8C%E6%A1%B6&spm=1001.2101.3001.7020)中放入令牌。如果令牌桶满了，超过令牌桶容量的限制，就会丢弃。  
系统接收一个用户的请求，都会先去令牌桶取一个令牌，拿到令牌才会去处理业务逻辑，没有取到则放弃请求。由于令牌桶算法会存储一些令牌，所以可以应对一些突发流量。  

![](https://img-blog.csdnimg.cn/602b6ab10c3243158845e257e3d3318a.png)

## 2. [Guava](https://so.csdn.net/so/search?q=Guava&spm=1001.2101.3001.7020) RateLimiter 实现概述

Google Guava 的限流器是**令牌桶算法**实现。  
设计要点

*   令牌生成与存储
*   突发流量与预热

### 2.1. 令牌生成与存储

按照令牌桶的算法描述：`以固定的速率往令牌桶中放入令牌`。

*   需要首先有一个桶来存放令牌，这个桶可以是一个队列。
*   需要有一个线程来定时的往这个桶放令牌。

如果按照上面的思路实现令牌桶算法，如果应用中每个接口都单独设置限流器，则会有大量的限流器被创建，意味着会创建大量的线程以及大量存储令牌的队列。轻则频繁上下文切换应用性能降低，重则导致内存溢出。  
Guava 的 RateLimiter 就是用了一种技巧避免使用线程和队列，思路和 Redis 在获取 key 的时候判断是否过期的策略有一点点类似，RateLimiter 也是在获取令牌的时候才去处理新增令牌的逻辑。  
RateLimiter 中会记录一下信息：

*   多久产生一个令牌。如果 1 秒限制 10 次访问，那么对应 1/10s 会产生一个令牌。
*   下次发放令牌的时间。
*   上次获取令牌时存储了多少令牌。
*   最大可存储令牌数

当获取令牌时，会去计算当前时间与当前存储的`下次发放令牌时间`的差值 /`多久产生一个令牌`得到从上次获取令牌到当前时间一共可以产生多少令牌，然后将可存储的令牌数存储在一个`double`字段。由访问线程去驱动令牌计算，这样就省去了使用专门线程去放令牌，简化了令牌存储方案。

### 2.2. 突发流量与预热

Google 的限流器实现在对原本令牌桶算法进行了一些扩展。对突发流量的支持这个原本令牌桶算法就支持，但是预热没有。Google 的 RateLimiter 这两个功能对应来个实现类`SmoothBursty`和`SmoothWarmingUp`。

#### 2.2.1. SmoothBursty

SmoothBursty 用来处理突发流量的限流器。它比较简单，就是使用当前存储的一些令牌来应对突发流量。

#### 2.2.2. SmoothWarmingUp 设计

**SmoothWarmingUp 作用与使用场景**  
SmoothWarmingUp 用来处理应用启动时，一些缓存还没有生成等导致处理不了正常情况下能抗住的请求量。所以就需要在刚启动应用时每秒发放的令牌数要比正常情况下要小。比如正常可以每秒处理 5000 个请求，但是刚启动时只能处理 500 个，如果这个时候 5000 个请求打过来，应用刚启动就会挂掉。

**SmoothWarmingUp 原理**  
SmoothWarmingUp 它并没有去设计一个按启动时间来动态修改令牌发放速度的算法，而是通过当前令牌桶中剩余的令牌数来实现的。令牌桶中剩余的令牌越多，说明应用就越久没有人访问，也就是越冷，所以获取令牌需要的时间就越久。

```
^ throttling
              |
        cold  +                  /
     interval |                 /.
              |                / .
              |               /  .   ← "warmup period" is the area of the trapezoid between
              |              /   .     thresholdPermits and maxPermits
              |             /    .
              |            /     .
              |           /      .
       stable +----------/  WARM .
     interval |          .   UP  .
              |          . PERIOD.
              |          .       .
            0 +----------+-------+--------------→ storedPermits
              0 thresholdPermits maxPermits
```

上图中 x 轴表示令牌桶中的令牌数，y 轴表示获取令牌需要等待的时间。  
maxPermits 表示令牌桶能存储最大的令牌数。  
stable interval 表示多久产生一个令牌，也就是正常情况下令牌的发放速度。  
**合着一起看就是当令牌桶中的令牌数为 x 时，那么获取一个令牌需要等待的时间为 y**。

*   当系统最冷的时候 x 在`maxPermits`。
*   当系统最热的时候 x 在 0 处。
*   当系统请求逐渐变多，x 的值会渐渐的左移直到 0，当系统逐渐没有请求，x 的值会逐渐右移，直到为`maxPermits`。

在`SmoothWarmingUp`有一些假设条件。

*   `WARM UP PERIOD`表示预热的时间，其在函数图像中的面积是一个梯形（积分知识）。
*   梯形的面积是旁边矩形（x 轴：0 -> thresholdPermits 和 y 轴：0 -> stable interval 围成的面积）的两倍。  
    由上面两个假设，我们已知预热时间（由用户设定）和 stable interval(通过用户设定计算得出) 可以计算出以下的值。

```java
//预热时间=2倍的矩形面积
warmupPeriod = 2  * stableInterval  *  thresholdPermits
//由此可以推出
thresholdPermits=  = 0.5 * warmupPeriod / stableInterval
//预热时间=梯形面积
warmupPeriod = 0.5 * (stableInterval + coldInterval) * (maxPermits - thresholdPermits)
//由此推出maxPermits
maxPermits = thresholdPermits + 2.0 * warmupPeriod / (stableInterval + coldInterval)
```

从图中还可以看出，在系统最冷的时候，获取一个令牌所花的时间是正常情况下令牌发放速度的三倍。这里要注意，在`SmoothBursty`中获取令牌桶中存储的令牌时不用等待的，也就是说在`SmoothWarmingUp`中，从令牌桶中获取令牌最快也得等待`stable interval`，最慢为 3*`stable interval`。为什么是 3 倍呢？因为程序里面写死了。

```java
public static RateLimiter create(double permitsPerSecond, long warmupPeriod, TimeUnit unit) {warmupPeriod);
    return create(
        permitsPerSecond, warmupPeriod, unit, 3.0, SleepingStopwatch.createFromSystemTimer());
  }
```

## 3. Guava RateLimiter 源码分析

### 3.1. RateLimiter 成员变量

```java
//用来计时
private final SleepingStopwatch stopwatch;
```

### 3.2. SmoothRateLimiter 成员变量

```java
//当前令牌桶中存储的令牌数
  double storedPermits;
  //最大可以存储的令牌数
  double maxPermits;
  //令牌发放的间隔时间
  double stableIntervalMicros;
  //下个发放令牌的时间点
  private long nextFreeTicketMicros = 0L;
```

`SmoothBursty`和`SmoothWarmingUp`继承了`SmoothRateLimiter`所以它们都有上面的这些成员变量，它们也是实现整个算法的核心。

## 4. SmoothBursty 源码分析

### 4.1. 成员变量

```java
//存储最大突发秒数，默认是1S，通过这个最大突发秒数计算最大可以存储多少令牌
final double maxBurstSeconds;
```

### 4.2. 构造函数

```java
SmoothBursty(SleepingStopwatch stopwatch, double maxBurstSeconds) {
	//计时器
      super(stopwatch);
      //存储最大突发秒数赋值
      this.maxBurstSeconds = maxBurstSeconds;
 }
```

### 4.3. 创建

```java
static RateLimiter create(double permitsPerSecond, SleepingStopwatch stopwatch) {
  	
    RateLimiter rateLimiter = new SmoothBursty(stopwatch, 1.0 /* maxBurstSeconds */);
   	//设置速度
    rateLimiter.setRate(permitsPerSecond);
    return rateLimiter;
  }
```

```java
public final void setRate(double permitsPerSecond) {
    checkArgument(
        permitsPerSecond > 0.0 && !Double.isNaN(permitsPerSecond), "rate must be positive");
        //加锁，保证线程安全
    synchronized (mutex()) {
  		//设置令牌发送速度
      doSetRate(permitsPerSecond, stopwatch.readMicros());
    }
  }
```

```java
com.google.common.util.concurrent.SmoothRateLimiter#doSetRate(double, long)
 @Override
  final void doSetRate(double permitsPerSecond, long nowMicros) {
  	//如果到了可以发放令牌的时间则会更新令牌数
    resync(nowMicros);
    //计算多少微秒会产生一个令牌
    double stableIntervalMicros = SECONDS.toMicros(1L) / permitsPerSecond;
    this.stableIntervalMicros = stableIntervalMicros;
    //这是一个abstract方法，SmoothBursty，SmoothWarmingUp会有不同的实现，从而实现不同的功能
    doSetRate(permitsPerSecond, stableIntervalMicros);
}
void resync(long nowMicros) {
    //如果当前时间大于下次发放令牌的时间
    // if nextFreeTicket is in the past, resync to now
    if (nowMicros > nextFreeTicketMicros) {
      //coolDownIntervalMicros 有不同实现
        //计算出堆积了多少令牌。离上次发发放令牌到现在的时间除以 多少微秒会产生一个令牌
        //第一次计算的时候会返回一个无限大的值，因为stableIntervalMicros还没有被赋值。
        //不过没有关系,因为在doSetRate还会再设置一次
      double newPermits = (nowMicros - nextFreeTicketMicros) / coolDownIntervalMicros();
      storedPermits = min(maxPermits, storedPermits + newPermits);
      nextFreeTicketMicros = nowMicros;
    }
  }
```

```java
com.google.common.util.concurrent.SmoothRateLimiter.SmoothBursty#doSetRate
void doSetRate(double permitsPerSecond, double stableIntervalMicros) {
      double oldMaxPermits = this.maxPermits;
      //设置最大允许存储的令牌数，也就是能应对多少突发流量
      maxPermits = maxBurstSeconds * permitsPerSecond;
      if (oldMaxPermits == Double.POSITIVE_INFINITY) {
        // if we don't special-case this, we would get storedPermits == NaN, below
        storedPermits = maxPermits;
      } else {
        storedPermits =
        	//这个初始值很关键，对于SmoothWarmingUp它的初始值为最大可存储的令牌数，用于预热
            (oldMaxPermits == 0.0)
                ? 0.0 // initial state
                    //这个生效是在动态的修改permitsPerSecond时生效
                    //maxPermits扩大后，等比的扩大storedPermits
                : storedPermits * maxPermits / oldMaxPermits;
      }
    }
```

以上的代码就是构建 SmoothBursty 的全部逻辑。可以看到主要是设置了多久发放一个令牌，下次发放令牌的时间初始值，当前存储了多少令牌的初始值以及最大能存储多少令牌。分析完构建后，就再来看看是如何利用这几个参数实现限流的。

### 4.4. 获取令牌

获取令牌有不同的方法：`acquire`，`tryAcquire`。他们各自还有不同的重载，不过实现的主要逻辑差不多。

#### 4.4.1. acquire 方法

```java
public double acquire(int permits) {
  	//计算需要等待多长时间才能获取到指定的令牌数
    long microsToWait = reserve(permits);
    //睡microsToWait
    stopwatch.sleepMicrosUninterruptibly(microsToWait);
    //返回经过了多少秒才获取到令牌
    return 1.0 * microsToWait / SECONDS.toMicros(1L);
  }
  	//计算需要等待多长时间才能获取到指定的令牌数
   final long reserve(int permits) {
    checkPermits(permits);
    //保证线程安全
    synchronized (mutex()) {
      return reserveAndGetWaitLength(permits, stopwatch.readMicros());
    }
  }
  
  final long reserveAndGetWaitLength(int permits, long nowMicros) {
    long momentAvailable = reserveEarliestAvailable(permits, nowMicros);
    return max(momentAvailable - nowMicros, 0);
  }
```

```java
@Override
  final long reserveEarliestAvailable(int requiredPermits, long nowMicros) {
    resync(nowMicros);
    long returnValue = nextFreeTicketMicros;
    //计算出storedPermits能支持多少个，剩余的就需要freshPermits来支持
    double storedPermitsToSpend = min(requiredPermits, this.storedPermits);
    double freshPermits = requiredPermits - storedPermitsToSpend;
    //将获取storedPermits话费的时间和获取freshPermits的时间合并得到最终等待的时间
    long waitMicros =
        storedPermitsToWaitTime(this.storedPermits, storedPermitsToSpend)
            + (long) (freshPermits * stableIntervalMicros);
    //设置下次发放令牌时间
    this.nextFreeTicketMicros = LongMath.saturatedAdd(nextFreeTicketMicros, waitMicros);
    //减少storePermits消耗的令牌数
    this.storedPermits -= storedPermitsToSpend;
    return returnValue;
  }
```

storedPermitsToWaitTime() 方法是实现`SmoothWarmingUp`与`SmoothBursty`的核心方法。`SmoothBursty`很简单，就是直接返回 0，表示从令牌桶中获取令牌不需要付出任何代价。但是`SmoothWarmingUp`就会复杂一些。

```java
long storedPermitsToWaitTime(double storedPermits, double permitsToTake) {
      return 0L;
    }
```

#### 4.4.2. tryAcquire 方法获取令牌

```java
public boolean tryAcquire() {
    return tryAcquire(1, 0, MICROSECONDS);
  }
public boolean tryAcquire(int permits, long timeout, TimeUnit unit) {
    //将时间转换为微秒
    long timeoutMicros = max(unit.toMicros(timeout), 0);
    //确保permits大于0
    checkPermits(permits);
    long microsToWait;
    //下面的代码进入临界区
    synchronized (mutex()) {
      //时间是一个相对时间
      long nowMicros = stopwatch.readMicros();
      //判断最大等待时间到来前是否可以获取到permits，如果不可以则直接返回
      if (!canAcquire(nowMicros, timeoutMicros)) {
        return false;
      } else {
        microsToWait = reserveAndGetWaitLength(permits, nowMicros);
      }
    }
    //睡眠需要等待的时间
    stopwatch.sleepMicrosUninterruptibly(microsToWait);
    return true;
  }

private boolean canAcquire(long nowMicros, long timeoutMicros) {
    return queryEarliestAvailable(nowMicros) - timeoutMicros <= nowMicros;
  }
//直接返回下次发放令牌的时间
@Override
  final long queryEarliestAvailable(long nowMicros) {
    return nextFreeTicketMicros;
  }
```

以上就是`SmoothBursty`获取令牌的全部逻辑，`SmoothWarmingUp`的时间与`SmoothBursty`的逻辑基本一样，只有初始化和从令牌桶获取令牌的逻辑不一样。

## 5. SmoothWarmingUp 源码分析

在看下面分析前，可以回顾一下上面的函数图象，下面的成员变量以及初始化会和它又很大关系。

```
^ throttling
              |
        cold  +                  /
     interval |                 /.
              |                / .
              |               /  .   ← "warmup period" is the area of the trapezoid between
              |              /   .     thresholdPermits and maxPermits
              |             /    .
              |            /     .
              |           /      .
       stable +----------/  WARM .
     interval |          .   UP  .
              |          . PERIOD.
              |          .       .
            0 +----------+-------+--------------→ storedPermits
              0 thresholdPermits maxPermits
```

### 5.1. 成员变量

```java
//预热时间
	private final long warmupPeriodMicros;
	//斜率
    private double slope;
	//图像中的thresholdPermits
    private double thresholdPermits;
    //因子数，固定为3
    private double coldFactor;
```

### 5.2. 构造方法

```java
SmoothWarmingUp(
        SleepingStopwatch stopwatch, long warmupPeriod, TimeUnit timeUnit, double coldFactor) {
      super(stopwatch);
      //预热时间
      this.warmupPeriodMicros = timeUnit.toMicros(warmupPeriod);
      this.coldFactor = coldFactor;
    }
```

### 5.3. 创建

```java
//创建时会设置预热时间
 public static RateLimiter create(double permitsPerSecond, Duration warmupPeriod) {
    return create(permitsPerSecond, toNanosSaturated(warmupPeriod), TimeUnit.NANOSECONDS);
  }

  public static RateLimiter create(double permitsPerSecond, long warmupPeriod, TimeUnit unit) {
    checkArgument(warmupPeriod >= 0, "warmupPeriod must not be negative: %s", warmupPeriod);
    //coldFactor固定为3
    return create(
        permitsPerSecond, warmupPeriod, unit, 3.0, SleepingStopwatch.createFromSystemTimer());
  }

 static RateLimiter create(
      double permitsPerSecond,
      long warmupPeriod,
      TimeUnit unit,
      double coldFactor,
      SleepingStopwatch stopwatch) {
    RateLimiter rateLimiter = new SmoothWarmingUp(stopwatch, warmupPeriod, unit, coldFactor);
    //这个和SmoothBurst调用相同的方法，但是里面的doSetRate会有差异。
    rateLimiter.setRate(permitsPerSecond);
    return rateLimiter;
  }
```

```java
@Override
    void doSetRate(double permitsPerSecond, double stableIntervalMicros) {
      double oldMaxPermits = maxPermits;
      double coldIntervalMicros = stableIntervalMicros * coldFactor;
      //这里的推导
      //长方形的面积是 warmupPeriod/2，也就是梯形面积的一半，是因为 coldFactor 是硬编码的 3
      //梯形面积为 warmupPeriod，而长方形面积为 stableInterval * thresholdPermits
      //warmupPeriod = 2 * stableInterval * thresholdPermits
      //所以 thresholdPermits = 0.5 * warmupPeriod / stableInterval
      thresholdPermits = 0.5 * warmupPeriodMicros / stableIntervalMicros;
      //梯形面积计算公式
      //warmupPeriod = 0.5 * (stableInterval + coldInterval) * (maxPermits - thresholdPermits)
      //maxPermits = thresholdPermits + 2.0 * warmupPeriod / (stableInterval + coldInterval)
      maxPermits =
          thresholdPermits + 2.0 * warmupPeriodMicros / (stableIntervalMicros + coldIntervalMicros);
      slope = (coldIntervalMicros - stableIntervalMicros) / (maxPermits - thresholdPermits);
      if (oldMaxPermits == Double.POSITIVE_INFINITY) {
        // if we don't special-case this, we would get storedPermits == NaN, below
        storedPermits = 0.0;
      } else {
        storedPermits =
            (oldMaxPermits == 0.0)
                    //初始化为最大的storedPermits，这样就是冷启动
                ? maxPermits // initial state is cold
                : storedPermits * maxPermits / oldMaxPermits;
      }
    }
```

### 5.4. 获取令牌

获取令牌的令牌与 SmoothBursty 基本一致，唯一有区别的地方就是从令牌桶中获取令牌的逻辑。这个方面也有提到，看漏了的可以回看。

```java
@Override
    long storedPermitsToWaitTime(double storedPermits, double permitsToTake) {
      double availablePermitsAboveThreshold = storedPermits - thresholdPermits;
      long micros = 0;
      // measuring the integral on the right part of the function (the climbing line)
      if (availablePermitsAboveThreshold > 0.0) {
        double permitsAboveThresholdToTake = min(availablePermitsAboveThreshold, permitsToTake);
        // TODO(cpovirk): Figure out a good name for this variable.
        double length =
            permitsToTime(availablePermitsAboveThreshold)
                + permitsToTime(availablePermitsAboveThreshold - permitsAboveThresholdToTake);
        micros = (long) (permitsAboveThresholdToTake * length / 2.0);
        //减去从梯形获取的令牌，剩下的从矩形获取
        permitsToTake -= permitsAboveThresholdToTake;
      }
      // measuring the integral on the left part of the function (the horizontal line)
      //梯形的面积计算完后，计算矩形的面积（如果矩形已经够用了，可能permitsToTake会为0）
      micros += (long) (stableIntervalMicros * permitsToTake);
      return micros;
    }
```

## 6. 总结

Guava 使用了一种巧妙的方式避免使用额外的线程发放令牌，以及使用额外的容器存放每一个令牌，大大提升了程序的性能。不过 Guava 的限流器只是针对单机，如果需要实现分布式的限流则需要另外的中间件了。

参考资料  
Guava 源码