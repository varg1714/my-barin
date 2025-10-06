---
source: https://juejin.cn/post/7359577911990894627
create: 2025-09-23 16:48
read: true
knowledge: true
knowledge-date: 2025-09-23
tags:
  - 分布式
  - 系统架构
  - 多线程
summary: "[[RRateLimiter 实现原理]]"
---
```
大家好，我是MK，一位近9年的后端JAVA coder。
身边coder朋友、同学包括我都没有写技术博客习惯；秉着改变、进步、尝试的想法开启了我的第一篇博客文章。
本人是JAVA后端老油条、入门级（产品、前端、测试、运维、数据分析、RPA工程师、AI工具玩家），自导自演从原型开始从0到1统筹项目开发。
后续计划用技术博客作为技术分享笔记，边梳理边分享，互相交流，互相学习，如不嫌弃点波关注。越努力越幸运，期待轻舟已过万重山！
```

`本次讲解Redisson版本：3.16.1`

## 一、RRateLimiter 限流使用

```java
Config config = new Config();
config.useSingleServer().setAddress("redis://127.0.0.1:6379");
RedissonClient redissonClient = Redisson.create(config);
//限流key
String limiterKey = "myRateLimiter";
//根据redissonClient获取RRateLimiter，并设置限流key
RRateLimiter rateLimiter = redissonClient.getRateLimiter(limiterKey);
//设置限流参数
//rateType：OVERALL全局限流、PER_CLIENT当前客户端应用限流（即单机限流，自动加上当前JAVA客户端ID做标记）
//rate：限流数量（总许可证数），即时间窗口内超出此数量则限流
//rateInterval：时间窗口间隔，即多少个时间单位内作为时间窗口
//rateIntervalUnit：时间窗口时间单位 
//以下举例配置说明：全局限流、2分钟时间窗口间隔最多支持4个访问数量
boolean trySetRateResult = rateLimiter.trySetRate(RateType.OVERALL, 4, 2, RateIntervalUnit.MINUTES);
//尝试申请许可证
//permits：本次申请的许可证数，不填时默认为1
boolean tryAcquireResult = rateLimiter.tryAcquire(1);
//申请成功执行业务代码
if(tryAcquireResult){
    doBusiness();
}

// 关闭 RedissonClient 连接
redissonClient.shutdown();
```

## 二、源码分析

### （一）redissonClient.getRateLimiter(limiterKey)

```java
//进入Redisson.java，获取RRateLimiter时传入限流key并使用默认命令处理器
@Override
public RRateLimiter getRateLimiter(String name) {
    return new RedissonRateLimiter(commandExecutor, name);
}
```

### （二）rateLimiter.trySetRate(RateType.OVERALL, 4, 2, RateIntervalUnit.MINUTES)

```java
//进入RedissonRateLimiter.java，设置限流配置参数
//lua脚本解释：设置rate、interval、type等参数时，原来存在则不设置
//此处无法满足当限流配置参数改动时覆盖并重置限流功能，下篇文章将讲解重写此方法解决配置不同重置限流问题
@Override
public RFuture<Boolean> trySetRateAsync(RateType type, long rate, long rateInterval, RateIntervalUnit unit) {
    return commandExecutor.evalWriteAsync(getRawName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN,
            "redis.call('hsetnx', KEYS[1], 'rate', ARGV[1]);"
          + "redis.call('hsetnx', KEYS[1], 'interval', ARGV[2]);"
          + "return redis.call('hsetnx', KEYS[1], 'type', ARGV[3]);",
            Collections.singletonList(getRawName()), rate, unit.toMillis(rateInterval), type.ordinal());
}
```

### （三）rateLimiter.tryAcquire(1)

```java
private <T> RFuture<T> tryAcquireAsync(RedisCommand<T> command, Long value) {
    return commandExecutor.evalWriteAsync(getRawName(), LongCodec.INSTANCE, command,
           //获取限流配置中rate、interval、type参数，限流配置存于limiterKey中
            "local rate = redis.call('hget', KEYS[1], 'rate');"
          + "local interval = redis.call('hget', KEYS[1], 'interval');"
          + "local type = redis.call('hget', KEYS[1], 'type');"
          + "assert(rate ~= false and interval ~= false and type ~= false, 'RateLimiter is not initialized')"
          
          //{limiterKey}:value作为key，存储剩余许可证数量
          + "local valueName = KEYS[2];"
          //{limiterKey}:permits作为key，记录所有许可发出的时间戳及当次申请的许可证个数
          + "local permitsName = KEYS[4];"
          //单机限流，则valueName、permitsName使用带有当前客户端标记作为key，如：{limiterKey}:value:b474c7d5-862c-4be2-9656-f4011c269d54
          + "if type == '1' then "
              + "valueName = KEYS[3];"
              + "permitsName = KEYS[5];"
          + "end;"

          + "assert(tonumber(rate) >= tonumber(ARGV[1]), 'Requested permits amount could not exceed defined rate'); "
          //获取当前剩余许可证数量
          + "local currentValue = redis.call('get', valueName); "
          //当剩余许可证数量存在时
          + "if currentValue ~= false then "
                  //获取zset中已过期的许可证记录
                 + "local expiredValues = redis.call('zrangebyscore', permitsName, 0, tonumber(ARGV[2]) - interval); "
                 //统计已过期的许可证记录有多少个许可证，注意一个许可证记录可能存在多个许可证数量（当次传入申请几个许可证数量则为几个）
                 + "local released = 0; "
                 + "for i, v in ipairs(expiredValues) do "
                      + "local random, permits = struct.unpack('fI', v);"
                      + "released = released + permits;"
                 + "end; "
                 //当过期许可证数量大于0时，则表示有可以回收的许可证，则删除许可证记录并重新把过期的许可证数量重新回收加入剩余许可证数currentValue
                 + "if released > 0 then "
                      + "redis.call('zremrangebyscore', permitsName, 0, tonumber(ARGV[2]) - interval); "
                      + "currentValue = tonumber(currentValue) + released; "
                      + "redis.call('set', valueName, currentValue);"
                 + "end;"
                 //剩余许可证数量小于本次申请的许可证数量，则访问被拒绝，返回下个许可证需要等待多长时间
                 + "if tonumber(currentValue) < tonumber(ARGV[1]) then "
                     + "local nearest = redis.call('zrangebyscore', permitsName, '(' .. (tonumber(ARGV[2]) - interval), '+inf', 'withscores', 'limit', 0, 1); "
                     + "return tonumber(nearest[2]) - (tonumber(ARGV[2]) - interval);"
                 //剩余许可证数量大于等于本次申请的许可证数量，则可以访问，添加本次申请记录到permitsName中，剩余许可证数量减去本次申请的许可证数量
                 + "else "
                     + "redis.call('zadd', permitsName, ARGV[2], struct.pack('fI', ARGV[3], ARGV[1])); "
                     + "redis.call('decrby', valueName, ARGV[1]); "
                     + "return nil; "
                 + "end; "
          //当剩余许可证数量不存在时，即是首次进入，则设置限流信息
          + "else "
                 //设置当前剩余许可证数量为配置中的最大许可证数量
                 + "redis.call('set', valueName, rate); "
                 //设置本次需要申请的许可证记录到permitsName中，ARGV[2]即当前时间戳作为zse排序字段
                 //ARGV[3]：8字节的随机值，ARGV[1]：本次申请许可证数量，拼接打包后作为zset值
                 + "redis.call('zadd', permitsName, ARGV[2], struct.pack('fI', ARGV[3], ARGV[1])); "
                 //剩余许可证数量减去本次申请的许可证数量
                 + "redis.call('decrby', valueName, ARGV[1]); "
                 + "return nil; "
          + "end;",
          //Arrays.asList中的为key，在lua脚本中以KEY[1]、KEY[2]等获取对应值，KEY[1]表示获取第一个值
          //Arrays.asList后面的参数都为传入参数值，在lua脚本中可用ARGV[1]、ARGV[2]等获取对应位置值
            Arrays.asList(getRawName(), getValueName(), getClientValueName(), getPermitsName(), getClientPermitsName()),
            value, System.currentTimeMillis(), ThreadLocalRandom.current().nextLong());
}
```

### tryAcquireAsync 等待逻辑

```java
private void tryAcquireAsync(long permits, RPromise<Boolean> promise, long timeoutInMillis) {
    long s = System.currentTimeMillis();
    RFuture<Long> future = tryAcquireAsync(RedisCommands.EVAL_LONG, permits);
    future.onComplete((delay, e) -> {
        if (e != null) {
            promise.tryFailure(e);
            return;
        }
        
        if (delay == null) {
            //delay就是lua返回的 还需要多久才会有令牌
            promise.trySuccess(true);
            return;
        }
        
        //没有手动设置超时时间的逻辑
        if (timeoutInMillis == -1) {
            //延迟delay时间后重新执行一次拿令牌的动作
            commandExecutor.getConnectionManager().getGroup().schedule(() -> {
                tryAcquireAsync(permits, promise, timeoutInMillis);
            }, delay, TimeUnit.MILLISECONDS);
            return;
        }
        
        //el 请求redis拿令牌的耗时
        long el = System.currentTimeMillis() - s;
        //如果设置了超时时间，那么应该减去拿令牌的耗时
        long remains = timeoutInMillis - el;
        if (remains <= 0) {
            //如果拿令牌的时间比设置的超时时间还要大的话直接就false了
            promise.trySuccess(false);
            return;
        }
        //比如设置的的超时时间为1s，delay为1500ms，那么1s后告知失败
        if (remains < delay) {
            commandExecutor.getConnectionManager().getGroup().schedule(() -> {
                promise.trySuccess(false);
            }, remains, TimeUnit.MILLISECONDS);
        } else {
            long start = System.currentTimeMillis();
            commandExecutor.getConnectionManager().getGroup().schedule(() -> {
                //因为这里是异步的，所以真正再次拿令牌之前再检查一下过去了多久时间。如果过去的时间比设置的超时时间大的话，直接false
                long elapsed = System.currentTimeMillis() - start;
                if (remains <= elapsed) {
                    promise.trySuccess(false);
                    return;
                }
                //再次拿令牌
                tryAcquireAsync(permits, promise, remains - elapsed);
            }, delay, TimeUnit.MILLISECONDS);
        }
    });
}
```

## 三、源码关键点

### （一）相关 Key 分析

<table><thead><tr><th>Key</th><th>说明</th><th>保存相关参数</th></tr></thead><tbody><tr><td>limiterKey</td><td>存储当前限流配置，rateLimiter.trySetRate(RateType.OVERALL, 4, 2, RateIntervalUnit.MINUTES)</td><td>rate：总许可证数；interval：使用 rateInterval、rateIntervalUnit 算出时间窗口毫秒数；type：0 为全局、1 为单机</td></tr><tr><td>{limiterKey}:value</td><td>存储剩余许可证数量</td><td></td></tr><tr><td>{limiterKey}:permits</td><td>已发出申请成功记录，使用 zset 存储</td><td>Score：申请成功时间戳（毫秒）；Member：8 字节的随机值 + 本次申请成功记录许可证数量（即 tryAcquire 传值数量），如：13r155pk、2 进行拼接及处理</td></tr></tbody></table>

#### 1. limiterKey 截图：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18a60a0a34434ac395af533eacb130d4~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1887&h=392&s=45303&e=png&b=fefefe)

#### 2. {limiterKey}:value 截图：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb04253be12449e88b0dce921dc7b04b~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1876&h=413&s=39597&e=png&b=fefefe)

#### 3. {limiterKey}:permits 截图：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e98f961c573047b9a24a118b64af6760~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1864&h=474&s=70439&e=png&b=fdfdfd)

## 四、限流执行流程图

### （一）主流程

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9730996f71a84a38bf9785cdeed5101b~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1765&h=2631&s=224035&e=png&b=f3f4f5)

### （二）tryAcquire 流程

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2156e9ec65ff494fb87ee02072e0b792~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2218&h=4290&s=542625&e=png&b=f3f4f5)

## 五、总结

1.  **令牌桶机制**：RRateLimiter 基于令牌桶算法实现限流，该算法在固定的时间窗口内控制允许通过的请求量。令牌桶中会以固定的速率生成令牌，请求需要消耗令牌才能通过。如果令牌桶中无可用令牌，则请求将被限制。
2.  **时间窗口控制（滑动时间窗口算法）**：利用 Redis 的键过期机制来实现时间窗口的控制。这意味着每个时间窗口开始时都会重置令牌桶，确保每个时间窗口内的请求量受到限制。
3.  **非公平性**：RRateLimiter 的非公平性指的是它不会保证请求的处理顺序。在一个时间窗口内，先到达的请求可能会因为缺少令牌而被延迟处理，而后到的请求可能因为令牌充足而得到快速响应。这种机制适用于那些不需要严格按请求到达顺序处理的场景。
4.  **参数设置**：使用`trySetRate`方法可以设置限流参数，包括令牌的生成速率和桶的大小。这为开发者提供了灵活的限流策略配置能力。
5.  **分布式特性**：由于 RRateLimiter 是基于 Redis 实现的，因此它天生具有分布式特性。可以在多个节点上共享相同的限流状态，从而实现对分布式系统中所有节点的统一限流管理。
6.  **Lua 脚本**：RRateLimiter 底层使用 Lua 脚本来执行一些逻辑，这样可以提高性能并减少网络延迟。Lua 脚本的使用使得 RRateLimiter 能够高效地在 Redis 服务器端执行复杂的限流逻辑。
7.  **应用场景**：RRateLimiter 适用于需要对服务或 API 进行访问频率控制的场合，如防止系统过载、防止资源滥用等。

## 六、下期彩蛋

### （一）研究源码后发现如下问题：

1.  **限流配置变更时不生效**：当二次调用 rateLimiter.trySetRate() 修改不同限流参数 rate、rateInterval、rateIntervalUnit 时，`配置不更新`，无法更新最新限流方案；
2.  **申请许可证大于配置最大许可证数时源代码 lua 脚本报错**： 如 rateLimiter.tryAcquire(5)，rateLimiter.trySetRate() 中的 rate 为 4 时，源代码 lua 脚本抛异常：ERR Error running script (call to f_873b6a3c0be138fbd885d12401aef7b45672df07): @user_script:1: user_script:1:...；
3.  **RRateLimiter 未实现固定时间窗口算法**：当业务场景需要使用`固定时间窗口算法`时无法直接使用。

### （二）下期预告

```
拓展改造RRateLimiter实现以下功能：
 1. 重写rateLimiter.trySetRate()方法，支持限流配置参数变更时自动覆盖旧限流配置并重置限流；
 2. 拓展RRateLimiter实现固定时间窗口算法（按自然时间单位）。
```