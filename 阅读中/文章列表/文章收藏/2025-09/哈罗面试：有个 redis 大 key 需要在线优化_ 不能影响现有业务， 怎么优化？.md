---
source: https://mp.weixin.qq.com/s/ckplRMtP4bv7pdYoyYqW4A
create: 2025-09-11 21:10
read: true
knowledge: true
knowledge-date: 2025-09-12
tags:
  - Redis
summary: "[[Redis 大 key 在线迁移方案]]"
---
FSAC 未来超级架构师

架构师总动员  
实现架构转型，再无中年危机

# 尼恩说在前面：

在 40 岁老架构师 尼恩的**读者交流群** (50+) 中，最近有小伙伴拿到了一线互联网企业如得物、阿里、滴滴、极兔、有赞、shein 希音、shopee、百度、网易的面试资格，遇到很多很重要的面试题：

MySQL 的慢查询、如何监控、如何排查？

你知道如何排查慢 sql 嘛?

你知道如何对 mysql 进行优化嘛?

mysql 慢查询如何定位分析？哪些情况会导致慢查询？

没开 sql 慢查询日志，怎么发现慢 sql？

前几天 小伙伴面试 哈罗，遇到了这个问题。但是由于 没有回答好，导致面试挂了。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WOlF0ykBHHicM94ZGic77Kk99wFyPuf5yacPG6YCWLEe3AibiaicgeyKS8BictibR22qBNLES8dPlzicpXnjA/640?from=appmsg&watermark=1#imgIndex=0)

小伙伴面试完了之后，来求助尼恩。

那么，遇到 这个问题，该如何才能回答得很漂亮，才能 让面试官刮目相看、口水直流。

所以，尼恩给大家做一下系统化、体系化的梳理，使得大家内力猛增，可以充分展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer 直提”。

当然，这道面试题，以及参考答案，也会收入咱们的 《[尼恩 Java 面试宝典](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》V145 版本 PDF 集群，供后面的小伙伴参考，提升大家的 3 高 架构、设计、开发水平。

最新《尼恩 架构笔记》《尼恩高并发三部曲》《尼恩 Java 面试宝典》的 PDF，请关注本公众号【技术自由圈】获取，后台回复：领电子书

## 一、什么是 Big Key?

**通俗易懂的讲，Big Key 就是某个 key 对应的 value 很大，占用的 redis 空间很大，本质上是大 value 问题。**

key 往往是程序可以自行设置的，value 往往不受程序控制，因此可能导致 value 很大。

redis 中这些 Big Key 对应的 value 值很大，在序列化 / 反序列化过程中花费的时间很大，因此当我们操作 Big Key 时，通常比较耗时，这就可能导致 redis 发生阻塞，从而降低 redis 性能。

BigKey 指以 Key 的大小和 Key 中成员的数量来综合判定。

下面，尼恩 用几个实际的例子对大 Key 的特征进行描述：

*   Key 本身的数据量过大：一个 String 类型的 Key，它的值为 5MB
    
*   Key 中的成员数过多：一个 ZSET 类型的 Key，它的成员数量为 10000 个
    
*   Key 中成员的数据量过大：一个 Hash 类型的 Key，它的成员数量虽然只有 1000 个但这些成员的 Value 值总大小为 100MB
    

尼恩提示：在实际业务中，大 Key 的判定仍然需要根据 Redis 的实际使用场景、业务场景来进行综合判断。通常都会以数据大小与成员数量来判定。

## 二、Big Key 产生的场景？

**1、redis 数据结构使用不恰当**

将 Redis 用在并不适合其能力的场景，造成 Key 的 value 过大，如使用 String 类型的 Key 存放大体积二进制文件型数据。

**2、未及时清理垃圾数据**

没有对无效数据进行定期清理，造成如 HASH 类型 Key 中的成员持续不断的增加。即一直往 value 塞数据，却没有删除机制，value 只会越来越大。

**3、对业务预估不准确**

业务上线前规划设计考虑不足没有对 Key 中的成员进行合理的拆分，造成个别 Key 中的成员数量过多。

**4、明星、网红的粉丝列表、某条热点新闻的评论列表**

假设我们使用 List 数据结构保存某个明星 / 网红的粉丝，或者保存热点新闻的评论列表，因为粉丝数量巨大，热点新闻因为点击率、评论数会很多，这样 List 集合中存放的元素就会很多，可能导致 value 过大，进而产生 Big Key 问题。

## 三、Big Key 的危害？

### 危害 1：阻塞请求

Big Key 对应的 value 较大，我们对其进行读写的时候，需要耗费较长的时间，这样就可能阻塞后续的请求处理。

Redis 的核心线程是单线程，单线程中请求任务的处理是串行的，前面的任务完不成，后面的任务就处理不了。

### 危害 2：内存增大

读取 Big Key 耗费的内存比正常 Key 会有所增大，如果不断变大，可能会引发 OOM（内存溢出），

内存增大  达到 redis 的最大内存 maxmemory 设置值， 会 引发写阻塞， 或导致  重要 Key 被逐出。

### 危害 3：阻塞网络

单 value 较大时， 读取 会占用服务器网卡较多带宽，

自身变慢的同时， 可能会影响该服务器上的其他 Redis 实例或者应用。

### 危害 4：影响主从同步、主从切换

删除一个大 Key， 造成主库较长时间的阻塞并引发同步中断或主从切换。

## 四、生产场景的 bigkey 扫描

结合 scan + 定时任务的方式，  在 吞吐量的低峰期，进行扫描

发现了 bigkey， 可以及时的进行 运维 告警， 发送  邮件通知或者    钉钉企业信息

参见下面的文章：

[滴滴一面：BigKey 问题很致命，如何排查和处理？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247491622&idx=1&sn=86e2e9190a0f92beff8871a515a08b95&scene=21#wechat_redirect)

## 五、在线优化

## 5.1、 step1： 大 key 拆分

假设大 Key 为 `user:info:all（Hash 类型）

```
redis.hset(“user:info:all", uid.toString(), JSON.toJSONString(info));
```

存储 100 万用户信息， map key 为用户 ID，value 为用户详情

**优化方案：**

按用户 ID 哈希分片，拆分为 100 个小 Key，如 user:info:0~user:info:99，

100 万用户 ,  每个 Key 存储约 1 万用户。

**分片公式：**

`shard_id = hash(uid) % 100`，通过用户 ID 计算所属分片。

## 5.2、 step2：  “同步双写”  ，确保新老数据一致性

在应用层实现 “写新 Key 的同时，也写老 Key”，确保数据一致性：

```
// 伪代码：双写逻辑public void updateUserInfo(Long uid, UserInfo info) {    // 1. 计算分片ID，写入新Key    int shardId = Math.abs(uid.hashCode() % 100);    string newKey = "user:info:" + shardId;    redis.hset(newKey, uid.toString(), JSON.toJSONString(info));    // 2. 同时写入老Key（兼容旧逻辑）    string oldKey = "user:info:all";    redis.hset(oldKey, uid.toString(), JSON.toJSONString(info));}
```

老 Key 继续服务现有请求，新 Key 逐步积累完整数据， 避免数据丢失。

## 5.3、 step3：渐进式迁移，实现 新 key、老 key 一致性

用**后台任务 + 分批迁移**的方式，将老 Key 数据迁移到新 Key，实现 新 key、老 key 一致性

使用 scan ，避免一次性操作阻塞 Redis：

```
# 伪代码：Python后台迁移脚本import redisimport timer = redis.Redis(host='localhost', port=6379, db=0)old_key = "user:info:all"shard_count = 100cursor = 0while True:    # 1. 渐进式扫描老Key（每次扫描1000个field，避免阻塞）    cursor, fields = r.hscan(old_key, cursor=cursor, count=1000)    if not fields:        break  # 迁移完成    # 2. 按分片写入新Key    shard_data = {}    for uid, info in fields.items():        shard_id = hash(uid) % shard_count        shard_key = f"user:info:{shard_id}"        if shard_key not in shard_data:            shard_data[shard_key] = []        shard_data[shard_key].append((uid, info))    # 3. 批量写入新Key（pipeline减少网络往返）    with r.pipeline() as pipe:        for shard_key, kvs in shard_data.items():            pipe.hset(shard_key, mapping=dict(kvs))        pipe.execute()    # 4. 每批迁移后休眠，降低Redis压力    time.sleep(0.1)
```

关键点：

*   用`HSCAN`替代`HGETALL`，每次扫描少量数据（如 1000 条）。
    
*   迁移期间不删除老 Key 的 field，避免新写入的数据与迁移数据冲突。
    

**这里千万不要用 `HGETALL` ， 尼恩一个 大厂学员 用 `HGETALL` ， 结果把 redis 打挂了， 损失惨重。**

[太惨：用错一个 redis 命令，一波请求高峰把 redis 打挂了。 绩效被降级，如何避免再犯？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247505805&idx=1&sn=87f5ad87aa01b32dcc6b912e74bc3a3f&scene=21#wechat_redirect)

## 5.4：灰度切换：把 读请求 逐步切换 到 new  Key

待新 Key 数据完整后（通过校验新老 Key 的 field 数量确认），逐步切换读请求到新 Key：

**第一步：抽样验证**：

随机抽取 1% 的用户 ID，对比新 Key 和老 Key 的查询结果，确保一致性。

**第二步：流量切分：**

*   先将 10% 的读请求切换到新 Key（通过配置中心动态控制）。
    
*   监控 Redis 性能（如`INFO stats`中的`keyspace_hits`、`latency`）和业务错误率，无异常则逐步提升比例，一直到 至 100%。
    

**灰度切换 示例**：

```
public UserInfo getUserInfo(Long uid) {    // 从配置中心获取切换比例（如100%）    int switchRatio = config.getInteger("user.info.switch.ratio", 100);    if (ThreadLocalRandom.current().nextInt(100) < switchRatio) {        // 读新Key        int shardId = Math.abs(uid.hashCode() % 100);        String info = redis.hget("user:info:" + shardId, uid.toString());        if (info != null) return JSON.parseObject(info, UserInfo.class);    }    // 降级读老Key（兜底）    String info = redis.hget("user:info:all", uid.toString());    return info != null ? JSON.parseObject(info, UserInfo.class) : null;}
```

#### 防请求穿透到数据库的兜底设计

迁移过程中若出现新 Key 查询失败 ，需通过多级 兜底：

**第一级兜底**：新 Key 查询为空时，先查老 Key

**第二级兜底：** 允许查数据库，且数据库查询结果异步回写 Redis 新 Key。

**限流保护**：用 Redis 或网关对数据库查询接口限流（如每秒最多 100 次），避免大量穿透击垮数据库。

## 5. 5：删除老 Key（非阻塞）

确认所有请求已切换到新 Key 后，用`UNLINK`（异步删除）替代`DEL`删除老 Key，避免阻塞：

```
# 异步删除老Key，Redis后台线程处理，不阻塞主线程127.0.0.1:6379> UNLINK user:info:all(integer) 1
```

**这里千万不要用 `DEL` ， 可以会 损失惨重，还是这个案例。**

[太惨：用错一个 redis 命令，一波请求高峰把 redis 打挂了。 绩效被降级，如何避免再犯？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247505805&idx=1&sn=87f5ad87aa01b32dcc6b912e74bc3a3f&scene=21#wechat_redirect)

若老 Key 体积极大（如 1GB+），可先通过

```
HSCAN + HDEL
```

分批删除 field，再删除 Key：

```
# 分批删除老Key的fieldcursor = 0while True:    cursor, fields = r.hscan(old_key, cursor=cursor, count=1000)    if not fields:        break    r.hdel(old_key, *fields.keys())    time.sleep(0.1)r.unlink(old_key)  # 最后删除空Key
```

## 5.6  监控与回滚预案

实时监控：

*   Redis 指标监控：`latency`（延迟）、`used_memory`（内存变化）、`expired_keys`（过期 Key 数）。
    
*   业务指标监控：接口响应时间、错误率（如 5xx/4xx 状态码）、数据库查询量。
    

**回滚机制**：若发现异常，通过配置中心立即将读请求切回老 Key，暂停迁移任务。

优化大 Key 的核心是 “**大 key 拆分 +  双写双读 + 非阻塞迁移  + 灰度放量迁移 + 非阻塞删除 + 可以回滚**”，

通过后台迁移避免影响线上流量，同时用多层缓存和降级策略防止请求穿透到数据库。

整个过程需持续监控，确保每一步都可回滚，最终实现平滑过渡。

## 说在最后：有问题找老架构取经‍

只要按照上面的 尼恩团队梳理的 方案去作答， 你的答案不是 100 分，而是 120 分。  面试官一定是 心满意足， 五体投地。

按照尼恩的梳理，进行 深度回答，可以充分展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer 直提”。

在面试之前，建议大家系统化的刷一波 5000 页《[尼恩 Java 面试宝典 PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》，里边有大量的大厂真题、面试难题、架构难题。

很多小伙伴刷完后， 吊打面试官， 大厂横着走。

在刷题过程中，如果有啥问题，大家可以来 找 40 岁老架构师尼恩交流。

另外，如果没有面试机会， 可以找尼恩来改简历、做帮扶。前段时间，**[指导一个小伙 暴涨 200%（涨 2 倍），29 岁 / 7 年 / 双非一本 ， 从 13K 一次涨到 37K ，逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486433&idx=1&sn=0f630e8fc7620b3bf0e054827b48d91e&scene=21#wechat_redirect)。**

狠狠卷，实现 “offer 自由” 很容易的， 一个武汉的跟着尼恩卷了 2 年的小伙伴， 在极度严寒 / 痛苦被裁的环境下， offer 拿到手软， 实现真正的 “offer 自由” 。

## Java+AI  弯道超车，  转架构 捷径 

 [会 AI 的程序员，工资暴涨 50%！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486827&idx=1&sn=db3f268c6752ddb074a98c7b13a84011&scene=21#wechat_redirect)

  

[涨薪传奇： 18k->38K , 单月暴 20K，32 岁小伙伴 2 个月时间年薪 翻 1.5 倍 ，一步登天 + 逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486960&idx=1&sn=f57253a448694c32e834207381c42284&scene=21#wechat_redirect)

  

[低学历 传奇：29 岁 6 年专套本，受够了外包，狠卷 3 个月逆袭大厂 涨 1 倍， 逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486945&idx=1&sn=f6ff6231ccf3585624161c2ef1f50fdf&scene=21#wechat_redirect)

  

[极速上岸： 被裁 后， 8 天 拿下 京东，狠涨 一倍 年薪 48W， 小伙伴 就是 做对了一件事](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486913&idx=1&sn=edd6774e9d17c39327e8dcb95fdd68d9&scene=21#wechat_redirect)

  

[外包 + 二本 进 美团： 26 岁小 2 本 一步登天， 进了顶奢大厂（ 美团） ， 太爽了](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486904&idx=1&sn=e7c878f99ed101ebfbd1d80ec0e07401&scene=21#wechat_redirect)

  

[超牛的 Java+Al 双栖架构： 34 岁无路可走，一个月翻盘，拿 3 个架构 offer，靠  Java+Al   逆天改命！！！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486848&idx=1&sn=ff43d058271b0801f84aba3f3d957855&scene=21#wechat_redirect)

  

[Java+Al 逆袭 1： 34 岁无路可走，一个月翻盘，拿 3 个架构 offer，靠  Java+Al   逆天改命！！！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486848&idx=1&sn=ff43d058271b0801f84aba3f3d957855&scene=21#wechat_redirect)

  

java+AI 逆袭 2：[：3 年 程序媛 被裁， 25W-》40W 上岸， 逆涨 60%。 Java+AI 太神了， 架构小白 2 个月逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486858&idx=1&sn=9cd122c18d166433b43d0952d3a7a7a8&scene=21#wechat_redirect)

  

  

[Java+AI 逆袭 3 ： 36 岁 / 失业 7 个月 / 彻底绝望 。狠卷 3 个月 Java+AI ，终于逆风翻盘，顺利 上岸](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486870&idx=1&sn=990a540a155cdb8254ebce6c75816882&scene=21#wechat_redirect)

[Java+AI 逆袭 ： 闲了一年，41 岁 / 失业 12 个月 / 彻底绝望 。狠卷 2 个月 Java+AI ，终于逆风翻盘](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486880&idx=1&sn=a37033157c766ac5ae7c9195fe776693&scene=21#wechat_redirect)

[Java+AI 逆袭 5：1 个月大涨 2.5W，37 岁 脱坑外包， 入了正编，GO+AI 要逆天了](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486885&idx=1&sn=4e26fbb093f45d437dedf14ea9b9e6c5&scene=21#wechat_redirect)

**职业救助站**

实现职业转型，极速上岸

![](https://mmbiz.qpic.cn/mmbiz_jpg/9qY2Gt3PBs3icE6WcM4nBOdRRcczOW4wkNDgCMZhBfqeet9suwGHfgQduP33YYqqZLOppecMI72Pv5ibNkPeIk6g/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=1)

关注**职业救助站**公众号，获取每天职业干货  
助您实现**职业转型、职业升级、极速上岸**  
---------------------------------

**技术自由圈**

实现架构转型，再无中年危机

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/xlgvgPaib7WMtyFcJx45oIeWL1d6AENcdoAvuTPuq46E9n8pPe4SHXoSicuEVvhibHRSIzmEBPlC7POcDaqD6ID7w/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=2)

关注**技术自由圈**公众号，获取每天技术千货  
一起成为牛逼的**未来超级架构师**

**几十篇架构笔记、5000 页面试宝典、20 个技术圣经  
请加尼恩个人微信 免费拿走**

**暗号，请在 公众号后台 发送消息：领电子书**

如有收获，请点击底部的 " 在看 "和" 赞 "，谢谢