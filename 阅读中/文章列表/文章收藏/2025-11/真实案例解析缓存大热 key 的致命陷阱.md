---
source: https://mp.weixin.qq.com/s/-r6U2mwEAyiIzMS_zTn3-g
create: 2025-11-02 16:29
read: true
knowledge: true
knowledge-date: 2025-11-02
tags:
  - 系统运维
  - 系统架构
summary: "[[京东缓存大 key 故障分析]]"
---
### 引言

在现代软件架构中，缓存是提高系统性能和响应速度的重要手段。然而，如果不正确地使用缓存，可能会导致严重的线上事故，尤其是缓存的大热 key 问题更是老生常谈。本文将探讨一个常见但容易被忽视的问题：缓存大热 key 和缓存击穿问题。我们将从一个真实案例入手，分析其原因，并提供解决方案和预防措施。

### 案例描述

某系统在双十一大促期间，遇到了一个严重的线上事故。业务人员在创建一个大型活动，该大型活动由于活动条件和活动奖励比较多，导致生成的缓存内容非常大。活动上线后，系统就开始出现各种异常告警，核心 UMP 监控可用率由 100% 持续下降到 20%，系统访问 Redis 的调用次数和查询性能也断崖式下降，后续更是产生连锁反应影响了其他多个核心接口的可用率，导致整个系统服务不可用。

### 原因分析

在这个系统中，为了提高查询活动的性能，我们开发团队决定使用 Redis 作为缓存系统。将每个活动信息作为一个 key-value 存储在 Redis 中。由于业务需要，有时候业务运营人员也会创建一个非常庞大的活动，来支撑双十一期间的各种玩法。针对这种庞大的活动，我们开发团队也提前预料到了可能会出现的大 key 和热 key 问题，所以在查询活动缓存之前增加了一层本地 jvm 缓存，本地 jvm 缓存 5 分钟，缓存失效后再去回源查询 Redis 中的活动缓存，本以为会万无一失，没想到最后还是出了问题。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXoLSUOZrH3sDactJKAQeo7hMaFrw6nEU49gj4n0BKmE9Jj9mu7JLp1ib42a9abCHpVibBicwib3iacQPQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

查询方法伪代码

```
ActivityCache present = activityLocalCache.getIfPresent(activityDetailCacheKey);
if (present != null) {
    ActivityCache activityCache = incentiveActivityPOConvert.copyActivityCache(present);
    return activityCache
}
ActivityCache remoteCache = getCacheFromRedis(activityDetailCacheKey);
activityLocalCache.put(activityDetailCacheKey, remoteCache);
return remoteCache;

```

查询活动缓存流程如上图所示，为什么加了本地缓存还是出了问题？  
这里其实就存在着第一个缓存陷阱：缓存击穿问题。首先解释一下什么是缓存击穿；缓存击穿（Cache    Miss）是指在高并发的系统中，如果某个缓存键对应的值在缓存中不存在（即缓存失效），那么所有请求都会直接访问后端数据库，导致数据库的负载瞬间增加，可能会引发数据库宕机或服务不可用的情况。所以在本次事故里边，运营人员审批活动上线的一瞬间，活动缓存只是写入到了 Redis 缓存中，但是本地缓存还都是空的，所以此时就会有大量请求来同时访问 Redis。  
按照以往经验，Redis 缓存都是纯内存操作，查询性能可以满足大量请求同时查询活动缓存，就在此时我们却陷入了第二个缓存陷阱：网络带宽瓶颈；Redis 的高并发性能毋庸置疑，但是我们却忽略了一个大 key 和热 key 对网络带宽的影响，本次引发问题的大热 key 大小达到了 1.5M，经过事后了解京东 Redis 对单分片的网络带宽也有限流，默认 200M，根据换算，该热 key 最多只能支持 133 次的并发访问。所以就在活动上线的同一时刻，加上缓存击穿的影响，迅速达到了 Redis 单分片的带宽限流阈值，导致 Redis 线程进入阻塞状态，以至于所有的业务服务器都无法查询 Redis 缓存成功，最终引发了缓存雪崩效应。

### 解决方案

为了解决这个问题，我们开发团队采取了以下措施：

1.  **大 key 治理**：更换缓存对象序列化方法，由原来的 JSON 序列化调整为 Protostuff 序列化方式。治理效果：缓存对象大小由 1.5M 减少到了 0.5M。
    
2.  **使用压缩算法**：在存储缓存对象时，再使用压缩算法（如 gzip）对数据进行压缩, 注意设置压缩阈值，超过一定阈值后再进行压缩，以减少占用的内存空间和网络传输的数据量。压缩效果：500k 压缩到了 17k。
    
3.  **缓存回源优化**：本地缓存 miss 后回源查询 Redis 增加线程锁，减少回源 Redis 并发数量。
    
4.  **监控和优化 Redis 配置**：定期监控 Redis 网络传输情况，根据实际情况调整 Redis 的限流配置，以确保 Redis 的稳定运行。
    

治理后业务伪代码如下：

```
ActivityCache present = activityLocalCache.get(activityDetailCacheKey, key -> getCacheFromRedis(key));
if (present != null) {                
    return present;
}

```

```
/**
* 查询二进制缓存
*
* @param activityDetailCacheBinKey
* @return
*/
private ActivityCache getBinCacheFromJimdb(String activityDetailCacheBinKey) {
    List<byte[]> activityByteList = slaveCluster.hMget(activityDetailCacheBinKey.getBytes(),"stock".getBytes());
    if (activityByteList.get(0) != null && activityByteList.get(0).length > 0) {
        byte[] decompress = ByteCompressionUtil.decompress(activityByteList.get(0));
        ActivityCache activityCache = ProtostuffUtil.deserialize(decompress, ActivityCache.class);
        if (activityCache != null) {
            if (activityByteList.get(1) != null && activityByteList.get(1).length > 0) {
                activityCache.setAvailableStock(Integer.valueOf(new String(activityByteList.get(1))));
            }
            return activityCache;
        }
    }
return null;

```

### 预防措施

为了避免类似的问题再次发生，开发团队采取了以下预防措施：

1.  **设计阶段考虑缓存策略**：在系统设计阶段，充分考虑缓存的使用场景和数据特性，避免盲目使用大 key 缓存。
    
2.  **进行压力测试和性能评估**：在上线前，进行充分的压力测试和性能评估，模拟高并发和大数据量的情况，及时发现和解决潜在问题。
    
3.  **定期进行系统优化和升级**：随着业务的发展和技术的进步，定期对系统进行优化和升级，引入新的技术和工具来提高系统的性能和稳定性。
    

### 结论

缓存大 key 和热 key 是缓存使用中常见的陷阱，千万不要心存侥幸，否则会引发严重的线上事故。通过本文的案例分析和解决方案，我们希望能够帮助读者更好地理解和应对这个问题。记住，合理使用缓存是提高系统性能的关键，而不是简单地将所有数据都存储在缓存中