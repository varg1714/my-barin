---
source: https://mp.weixin.qq.com/s/azIR61meAJxMggB4dMgZ6g
create: 2023-11-13 17:13
---

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe94cQiccAo2zibZETiaOnMVLNQAO0Zne2x8KlehRMR8AsOTW90m1pAicBEw5wBJFkQiax8ricKGbKibEKV8gQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz_gif/VY8SELNGe96srmm5CxquJGSP4BbZA8IDLUj8l7F3tzrm8VuILsgUPDciaDLtvQx78DbkrhAqOJicxze5ZUO5ZLNg/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

👉导读

会控为整个会议最为核心的业务，由于海量请求的高性能要求，后台存储全部为 Redis。在业务飞速发展期，各模块边界不够清晰，大家对存储的使用处于失控状态，随着 PCU 的不断上涨，逐步暴露出存储和架构的诸多问题，同时也对系统容灾能力有了更高的要求。会控业务历史包袱重，存储改造伤筋动骨，要做到平滑迁移需要考虑的细节较多。有幸作为 owner 负责 (2022.12-2023.08) 了会控存储的优化改造，本文主要从业务、个人和企业数据分库、异地容灾和多活（下一步目标）层面总结了会控存储治理的成功实践，目的是形成一套方法论，沉淀下来一套可以复用的工具，以供大家后续工作中参考。

👉目录

1 背景

2 目标

3 存储收拢

4 业务改造

5 个人企业数据隔离和多活容灾

6 会议 ID 路由编码

7 平行扩展

8 分库

9 未来展望 & 后续计划

# 01

背景

特殊时期期间 PCU 猛涨，对会控存储带来了巨大的压力；近年来国内公司因可用区机房故障导致服务不可用的情况屡见不鲜，这也对系统容灾能力有了更高的要求；会控业务的逻辑层无状态可平行扩容，可跨城多 AZ 混部，但存储层 Redis 单实例存在诸多风险：

▶︎ 个人会议和企业会议数据共用存储实例，难以独立运营：

个人会议流量占绝对优势，个人会议流量的冲击可能会导致 Redis 异常，从而影响到重要的企业会议；

▶︎ 多 SET 存储层的隔离和互通：

为实现精细化运营，业务逻辑层已经做了多 SET 隔离：setA，setB，setC，setD，setE，流量调度规则复杂，多 SET 间共享存储层，存储未做隔离；

▶︎ 会议单实例 Redis 已达产品极限能力，难以垂直扩展：

写请求和高一致读的场景，主分片负载消耗高：Redis 集群的主分片数已经扩容到 128，Redis 运维不建议继续扩容分片，否则在运维难度、failover 等各方面都存在风险；

读写分离：Redis 集群的分片副本数已经扩容到 4，副本数增加将直接导致集群的 Proxy 压力增大，而会控 Redis 的 Proxy 数已经超过最大推荐值，且达到非标准运维的边界；

▶︎ 高容灾要求：

异地容灾 / 多活方面会议逻辑和存储均部署在广州，如果广州挂掉，将导致所有用户的服务不可用；

▶︎ 难以满足 10w QPS 的入会高要求：

流量如果继续上涨如何应对？

生产环境的各种异常可能导致大量用户断连，然后重新入会（可能多次重试），这种瞬时流量的冲击对存储性能提出了更高的要求；

▶︎ 大会议的冲击：

网络研讨会可能有数万参会者，每一次消息扩散带来的存储热 key 查询压力巨大；今年 4 月份一次 6W + 人数的会议，导致 Redis 单分片 CPU 被压高 30%；

▶︎ 容量和成本：

会控 Redis 均未设置过期时间，导致 Redis 单分片已是最高内存规格无法扩展；同时成本花费巨大。

# 02

目标

存储治理主要有五个目标：

▶︎ 异地容灾 / 多活：广州机房挂了，可切换至上海正常服务，提升 SLA 质量，达到可用性 5 个 9；

▶︎ 数据隔离：多 SET 数据按个人、企业隔离存储，优先保障企业服务的稳定性和 SLA 质量；

▶︎ 业务改造：推动非会控业务迁移，增加过期时间（内存压力），数据清理，降低成本；

▶︎ 平行扩展：对会控核心 Redis 实例水平拆分，通过平行扩展来支撑高 QPS，消除核心链路瓶颈；

▶︎ 存储收拢：提供存储代理，所有存储访问全部收拢；读写分离（主读写压力，充分利用集群性能）、鉴权、限流降级、监控告警等能力全部下沉至存储代理，统一管控。

随着特殊时期结束，水平扩展的优先级降低，读写分离从去年 12 月份开始已经基本完成，个人企业数据隔离上半年已经完成。核心存储改造势必会带来业务风险和架构调整，异地多活、个人企业隔离和 SET 隔离只是不同维度层面上的数据隔离，因此我们希望能统一处理，目标是个人和企业数据隔离工作能为下一步的异地容灾 / 多活做好铺垫。

会控存储目标架构如下：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe964d4J6z3T18icFOJdEWRIJ9kZzC0PtTjMs0C4PKDhbiaIcRhzVLtiadfCZETqfL9QhqL4wBfwjNPibNw/640?wx_fmt=png)

存储主要有两种部署模式：

▶︎ 多活模式：会议信息和成员列表实例，各自读写本地 Redis，通过跨地域复制互为灾备，最为理想的一种部署方式；

▶︎ 中心写镜像读模式：可以看作同城 Redis 集群的跨城扩展，主要用于难以多活的场景。读写广州中心，通过跨地域复制一份数据到上海镜像，镜像主要用于异地灾备和本地镜像读（可容忍短暂数据不一致，对一致性要求不高的场景，否则读主），比如成员信息。如果根据会议 ID 多活（绝大多数请求中带有会议 ID），广州和上海的实例最终都会存储全量的成员信息数据，这对业务的影响不好评估，且对少量无会议 ID 场景如历史会议查询、超时退会查询场景增加难度（需同时查询广州和上海）。当然这里需要在做异地多活的时候进一步验证。

**容灾说明：**

**容灾 5：**部分 AZ 挂了，Redis 实例同城多可用区容灾，主广四，备广三和广六，跨可用区 failover；

**容灾 4：**广州 Redis 集群挂了，手动切换接入层流量到上海，广州和上海双向同步（或成员信息广州单向同步至上海），因为逻辑层会多次操作 Redis ，导致跨城耗时放大，故直接在接入层切换流量，未同步完成的会议服务有损，如用户开麦可能失败，但是切换至上海后重试可成功。

为什么需要手动切换流量到上海？

由上图中七彩石配置控制，其优先级最高，因为某城市是否挂掉通过程序自动判断是比较危险的，此种极端情况人工判断更可靠。

**容灾 2 和 3：**同城多可用区部署，服务无状态可平行扩展，北极星容灾负载均衡；

广州逻辑层和（会控核心）数据代理层全挂了，手动切换接入层流量到上海，类容灾 4；

广州 SET 和上海 SET 交叉部署（即广州 SET 在广州和上海均部署机器，上海亦然），同样由多次访问存储导致的耗时无法接受；

**容灾 1：**广州所有服务包括接入层 RS 全挂了，CLB 会自动将流量切换到上海；

此场景同样需要人工介入，更改路由配置，所有请求不再转发广州，其调度优先级高于会议 ID 本身的路由策略；这是因为区域级别的灾难可能很多 RS 处于半死不活的状态，为保证质量流量不再调度广州；

不过这里可以考虑下优化：广州的会议被 CLB 调度到上海，上海的接入层 RS 发现是广州会议转发至广州接入层（人工配置还未生效阶段），如果（重试）超时，可调度至上海集群 SET 兜底，广州亦然。

## 1. 2.1 异地容灾和多活

基于延时考虑，一次信令可能会数十次查询 Redis 存储（包括依赖服务），多次跨城带来的访问延时无法接受，单纯地将存储异地灾备或者跨城混部无法解决问题。

经验证，部署在上海的会控服务访问广州的存储，预定会议耗时增加 600～800ms，会议作为即时通信工具对时延比较敏感，另外耗时增加对整个链路的连接数、内存等资源压力较大。因此我们需要将业务逻辑和存储在区域全套部署，这样在一个城市故障后，将流量切为另一个城市，流量同城闭环。根据 Redis 提供的能力，已支持北极星调度和跨城跨地域复制容灾，预计下半年支持多活，因此会控这边的计划也是先异地容灾，再异地多活。当然，对于难以多活的场景以及各项依赖需要进一步梳理验证。

容灾 / 多活必然会增加成本，业务逻辑层无状态，可依靠 HPA 扩缩容，但 MQ 和存储组件一般不支持异地，需要准备相同量级的集群，不过可以降级非核心功能。

## 2. 2.2 内部网关

后台接入设备、开放 API 以及定时任务、MQ 消费者直接通过后台 RPC 的自研协议调用会控服务，异地部署后需要走内部网关收拢。现在会议查询服务对外提供会控核心数据读能力，需要继续收拢信令接入，异地多活后主调无需关心路由、鉴权、限流等策略。个人 / 企业数据分库对这部分逻辑无影响。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjnhAuI7szNGsUTzhxP2XOict0wjUG41zOfdp0klZia1jCkLQxkicx97E8kg/640?wx_fmt=png)

# 03

存储收拢

历史原因，会议后台 50 + 服务直连 Redis 实例，导致存储改造难度极大；同时由于业务使用不规范，缺乏统一管控，导致 Redis 面临诸多潜在风险。

因此存储改造首要的前置工作就是存储访问收拢，会控提供存储访问代理，后台服务经代理访问存储。考虑到会控本身的业务情况，以及访问时延、QPS、工作量和成本，最终决定存储访问收拢于如下几个服务：

▶︎ 会控系列：会控以及拆分服务；SPP 框架开发，访问量大，涉及会议最为核心的功能，C++ SDK 方式集成；

▶︎ cache 读写：会控内部服务读 / 写，Go Module 方式；

▶︎ 会议查询服务 / 内部网关：外部团队服务通过会议查询服务 / 内部网关读写会控数据，Go Module 方式。

## 1. 3.1 存储代理

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjnzT4NawbV4icd1UCDrC4NIwFdgn3Hcbj6rn9eiabvvs5CPCQhMbQ20Uzg/640?wx_fmt=png)

会议后台 TCP 和 UDP 都用的比较多，大会议批量查询很容易超过 UDP 包大小 64K（SPP 的网络库中被修改为 32K），故批量接口采用 TCP 查询；

由于 trpc 协议采用的 PB3，而 SPP 服务则采用的 PB2，为方便 SPP 服务调用提供了 HTTP 接口；

采用配置化方式支持不同类型的 key 查询；RESP 协议编码会将参数转换为 string，据此提供了通用查询接口，存储代理只关心路由鉴权等策略。

## 2. 3.2 策略下沉

存储访问收拢后，多种访问控制策略即可下沉至代理。这带来的好处有三个：

▶︎ 规范新增流量，新接入的流量鉴权，读主还是读备，QPS 评估，key 大小，业务使用是否合理，是否可以使用带缓存的接口等；

▶︎ 整顿存量接入，存量流量收拢的过程中评估其使用合理性；

▶︎ 统一管控，提供更细粒度的监控和存储保护策略。

具体策略如下：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjnfXjMSL1v9cr65ibQDMc2CwpdW7bJpWgKThquibNeSAQM85OLicTSw3mGQ/640?wx_fmt=png)

其中路由主要屏蔽异地多活、Redis 分库（个人企业数据隔离或水平扩展）逻辑，对主调透明。

## 3. 3.3 动静分离

当遇到数万人的大会议时，消息扩散和各业务功能对会议信息的请求量骤增，会造成严重的热 Key 问题，曾经一个 6W + 人的大会议将会议信息 Redis 的单分片 CPU 压高了 30%。

会议信息存储的是一个 KV 结构，其中 Key 为会议 ID，64Bit 的无符号整数，Val 为一个大 PB 结构序列化后的二进制 Bytes 串，PB 结构有 200 + 个字段。我们的优化策略是对大表垂直拆分，将 Val 中不需要频繁变更的字段拆分出来作为 StableMeetingInfo 单独存储，为方便不同场景的查询，Val 仍然保留全量数据。经过拆分，部分场景可以直接查询 StableMeetingInfo，从而减小会议信息 Redis 的查询压力。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjnN6PQicqxdauLsX8uvrrbAY9tqxevq8qicibRZ3KFryZlKQlDUlYJAOoaw/640?wx_fmt=png)

## 4. 3.4 多级缓存

动静分离后，会议信息 Redis 的请求压力有所下降，但 QPS 仍处于高水位，此时存储代理采用 fastcache 本地缓存来应对热 key 流量，对于实时性要求不高的场景走本地缓存。根据数据的变更频率和实时性的不同要求，存储代理对本地缓存进行分级，提供 1 秒和 5min 过期的本地缓存接口。另外，静态数据 StableMeetingInfo 可以设置更长的本地过期时间。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjn9MBMIhWjmicGxLqKU50B0YOlZw52FAHlBeYCUOWjeJNxFeAcicnj64ibw/640?wx_fmt=png)

虽然会议后台没有走一致性哈希调度，本地缓存也可以拦截大量的 Redis 请求。假设某 Key QPS 为 20W，存储代理 100 台 POD 结点，那么每秒最少有（20W - 100）次请求 Cache Hit 本地缓存。

## 5. 3.5 过载保护

如果请求量仍然超过了系统的处理能力，则启动过载保护机制。为尽量减小对业务的影响，存储代理提供了多维度的降级策略：

▶︎ 按主调降级：手动配置触发，支持按比例随机丢弃，防止主调业务不合理的使用场景；

▶︎ 按热 Key 降级：手动配置触发，支持按比例随机丢弃，降低热 Key 造成的 Redis 单分片压力；

两种策略均依赖手动配置触发，因为难以给出降级触发的临界值做到程序自动化。采用了比较简单的随机数方式，对每个请求降级机会均等。

同时存储代理本身也有过载保护策略：

▶︎ trpc-go：依赖协程池，若请求到来工作协程耗尽，则丢弃；

▶︎ spp：proxy 和 worker 间的共享内存队列，超时则丢弃，防雪崩。

## 6. 3.6 监控感知

前面提到了手动配置触发，那如何感知生产异常？

▶︎ 存储实例定时巡检播报；

▶︎ 存储热 key、大 key、高负载告警；

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjn6FjMvIQiar2d7zxNKtKrZrCibY53RFIicqE6dydFAPMsuk8yr8AIWG4UQ/640?wx_fmt=png)

▶︎ 模调；

▶︎ CLS 日志抽样热 Key / 大 Key 跟踪；

▶︎ 存储实例 Key 统计日报；

▶︎ 核心功能的 P0 用例拨测。

为方便回溯问题根源，存储代理也对主调的调用情况做了报表统计：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjnQzHoiaN4lSwNCpVtaqPsR7uOmSXLBmeAibbmUQVpulTvz1owNQ4qhskg/640?wx_fmt=png)

## 7. 3.7 性能优化

所有的存储访问请求都会走存储代理，代理服务的性能至关重要。

trpc-go：

▶︎ bytes 和 string 转换，减少拷贝：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjniaHRaWPsX9CY4sf8I9XxGmSeP03jSPySIrMbdkiaIk8qoicnbq0gt7uwg/640?wx_fmt=png)

▶︎ golang 随机数涉及到全局锁，参考 jaeger 用对象池优化：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjnuL0tcbHLfVenhSVicM55cS8KIs6xHibn2deJHmRMiblOnEW2BPII0ClBA/640?wx_fmt=png)

benchmark 发现效果提升明显。

spp-c++：

▶︎ 全局票据 sig 本地缓存；

▶︎ 无锁化，七彩石路由配置加载线程和 SPP 业务线程不是同一个线程，我根据 SPP 的线程模型参考 Linux kfifo 实现了一个单线程写单线程读的无锁队列，SPP 协程定时消费，将配置赋值给全局对象，这样 SPP 请求直接无锁读取：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjnsbzUqwPgN15srMK6VAgPhcic5nFsXZxHtlJJ4E9lpOowNKrwZPxkNtw/640?wx_fmt=png)

为了方便阅读，没有利用 kfifo 的无符号数溢出特性，哈哈，当然如果你了解补码和无符号数的转换关系也比较简单。

C++ 中定时更新的场景很多都是用的双缓冲，因为执行间隔一般较长，在交换索引再次更新时业务逻辑已经处理完了，所以不存在读写并发；不过对于七彩石外部 SDK 不太可控，也没有详细了解其长轮询的间隔，故自己实现了个一读一写的无锁队列。

## 8. 3.8 可用容灾

存储代理成为了关键依赖，其本身的可用容灾由跨城部署、同城多 AZ 部署、HPA 和异地容灾来保证。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjnwicuAsJM6nYG7BYo6Qpia34oesm7fk0Eicibwdz3kRwAuDeiaH32Nrg63Lg/640?wx_fmt=png)

## 9. 3.9 业务推动

存储收拢需要推动各业务：

▶︎ 发现：建立后台所有服务的代码仓库（git 公共账号），按关键字（VIP 和 key）grep 搜索；

▶︎ 整改：难点是如何确认大家已经全部完成收拢，没有一个确切的指标来度量。我采用的方法是代码扫描 + 模调 + Redis 抓包综合排查，因为很多情况是一个 Redis 实例中太多业务 key，无法一次性直接去掉 VIP 配置；

▶︎ 预防：给大家宣讲存储使用规范后，难免还是担心由同学不按规范来操作。因此在大家整改完成后的时间点 (0521)，grep 扫描代码得到各服务包含关键字的信息快照，以此快照为参照，每天定时扫描对账。

为了避免对关键字改动感知过于敏感，我采用关键字在每个服务中出现的次数来对账，最终发现效果很好。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe964d4J6z3T18icFOJdEWRIJ9QO36hFLDjPA8SbicgqOsaPSmt0OGJQGLmFRa2nMibG3xR9RoHxWZLcKw/640?wx_fmt=png)

# 04

业务改造

业务层面的优化主要涉及到非会控业务 Key 迁移，和调用是否合理评估。

▶︎ 边界划分：

和各团队达成共识：会控仅负责会前和会中的数据，会后的数据由数据组负责；

▶︎ 业务迁移：

划分边界后，所有使用了会控 Redis 的非会控业务，推动迁移；

▶︎ 过期时间：

由于会控不再负责会后数据，对于将会控数据作为 DB 使用的业务推动改造，同时对所有会控数据逐步灰度增加过期时间，减小成本压力；

▶︎ 合理性评估：

对一致性要求不高的业务读写分离（主备同步存在延迟 5ms 以内），充分利用 Redis 集群水平扩展的能力，以及调用带本地缓存的接口，这在存储收拢和增量接入阶段完成；同时对于日常巡检群的热 Key 告警，分析业务使用是否合理；

▶︎ 数据清理：

非会控业务迁移完成后，脏数据以及会控的存量数据均需要清理，为防止操作失误我们对所有的修改都进行热备，以便灾难恢复。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjnkl65uODc5T1YpE9TksrRFUJJrTFChWiaW4y75pVXYOWHmJ6vMnxI3ag/640?wx_fmt=png)

# 05

个人企业数据隔离和多活容灾

个人企业数据隔离、SET 隔离和多活本质上是从不同维度对存储进行拆分，然后按策略路由调度，我们希望多种维度的拆分规则能够统一处理，这样方案更加优雅，实现起来也可以按部就班地迭代。

## 1. 5.1 业界调研

对于异地多活，业界也有许多优秀实践，本文主要以 QQ 的状态系统为参照。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjnNVvrMgf1QAqIOdzVv5XwSsgnKibL2dFnPs4ibUWmUr40FuwYZPVFSic1A/640?wx_fmt=png)

请求经 TGW 调度哪个地域则读写在此地域闭环处理，地域间两两相互同步，每个区域都有好友状态的全量数据，状态变更直接批量 PUSH 给异地好友（在本区域的存储读取异地好友状态列表）的接入点，状态系统可以容忍数据跨城同步的延迟（几十 ms）。

针对会控业务无法使用这种方式，会控需要高一致读：不同地区的人可参加同一个会议，会议业务的特点是会议开始时刻参会者集中入会，在这瞬间如果读区域本地的存储，异地数据（开麦 / 开视频等）大概率还没有同步过来，导致看到的状态不正确。另一个问题是异地写同一份数据导致的冲突合并数据不一致问题。

## 2. 5.2 请求路由

存储拆分后，如何将请求路由至正确的处理单元？

业界比较知名的单元化实践案例采用用户号段的方式，其处理方式类似于 Redis cluster 集群的 slot 机制，客户端初次请求（路由模块间转发）后，会将 uid 和其对应的处理单元编号存于 cookie。会议作为业务的核心，绝大多数的存储 key 和会议 ID 有关，很自然想到根据会议 ID 请求路由：相同会议的写请求只会路由到相同区域，业务层面数据区域自治，不同区域相互同步互为灾备；考虑到客户端改造成本高（新老版本兼容，各种客户端类型），且涉及到多域名改造，故从后台接入层开始着手。

那么根据会议 ID 路由能否满足异地多活延时要求？我们的目标是一次请求的跨城调用次数可控，整体延时在可接受范围内。梳理了下会控现有的 Key 情况：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjnrAialpic5zzYfib2OUG40eFnD2uEJbFqR3NjnppCUGYE9TGib2hX6hQxpw/640?wx_fmt=png)

对于无会议 ID 的场景，需要具体分析测试验证对业务的影响，不重要的场景，则可直接降级掉。小写请求量且一致性要求高的场景可考虑并行多写。

在接入层做流量转发（现在上海用户本身就会跨城访问），各 SET 流量最大程度区域自治。

## 3. 5.3 拆分策略

▶︎ 身份维度：个人和企业数据隔离存储；

▶︎ 会议信息：根据会议创建者账号类型隔离，如创建者是企业用户，则存储于企业实例；如创建者是个人用户，则存储于个人实例；

▶︎ 成员列表：根据会议 ID 隔离，每个会议都对应一个成员列表；

▶︎ 成员信息：根据成员自身账号类型隔离，企业账号存储于企业实例，个人用户存储于个人实例；存储 key 和会议 ID 无关；

对于成员信息的个人企业隔离，不会影响到下一步的异地多活；若成员信息中心写的方式，则中心实例为广州个人和广州企业；由于成员进退会、会中操作请求都会带会议 ID，亦可采用会议 ID 路由方式，则最终广州、上海的个人实例都拥有全量个人数据，企业实例都拥有全量企业数据；多份数据并存，若使用不当则为脏数据，有一定的风险。  

▶︎ 地域维度：广州上海异地多活，互为灾备，按地域划分 SET，流量区域自治，数据互备；前面提到，我们的目标是两种拆分维度能够统一处理。

## 4. 5.4 方案选型

会议的路由规则比较复杂，不能简单地按会议 ID 号段分配映射。接入层流量调度策略不仅有按环境变量 env 规则路由，也有根据会议本身的业务信息调度（如网络研讨会）；业务逻辑层各 SET 仅是机器运维层面的隔离，请求层面不做隔离，如企业 SET 可以调度个人会议，大盘 SET 也可以调度企业会议，这些路由规则都会用到会议业务信息，无法单纯根据会议 ID 决策，现状是接入层并行双（多）查。

个人和企业会议是通过会议创建者是否为企业账号区分，而创建者账号本身又存储于会议信息 VAL 结构中；加之异地多活后，会议信息进一步拆分为广州上海两地存储，接入层现在的处理方式显然不具备扩展性，因此要有一个中心化的第三方数据库专门存储路由信息，当然可以通过回源读，本地缓存的方式优化。我们这里需要将会议 ID 对应的所有路由信息：区域（gz，sh，tj，sg 等），身份（个人或企业），后续可能还会有某个 SET 单独一套存储系统等路由信息都保存起来。

经过讨论，最终有如下两种方案：路由表和会议 ID 编码路由信息，并且最终选择了会议 ID 编码路由信息的方案。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe964d4J6z3T18icFOJdEWRIJ9vZyg33xaHHEzO6uLA18zzLXk1QBSGSDZMWUYmBL5Y0mkfetSxZrXnw/640?wx_fmt=png)

## 5. 5.5 路由表

▶︎ 方案流程：预定会议时，更新会议信息和成员列表路由表，路由表格式 <会议 ID, val>，考虑到接入层需要会议信息路由调度，VAL 包括个人 / 企业、存储实例索引，以及会议类型、创建者企业 appid 等路由元数据，减少一次再查询会议信息的 RPC；

查询会议信息和成员列表数据时，先寻址路由中心服务，获取到个人 / 企业实例的索引；

▶︎ 路由信息链路透传：为减少下游查询路由的次数，接入层可以将路由信息透传，需要先打通后台的链路跟踪；

▶︎ 性能：存储代理查询到路由后，可在本地建立 LRU 缓存，加快下次访问速度（可一致性哈希调度提升缓存命中率）；

go：fastcache；

cpp：unordered_map + 双链表；

下游如没有路由信息，则兜底查路由中心，本地缓存；

异地多活后，路由存储可类 CDN 架构优化，优先查本区域存储，查不到则回源至主，集群间保持同步；

▶︎ 高可用：路由存储层的高可用由 Redis 集群来保证，逻辑层可类似存储代理来保证高可用。

▶︎ 路由表 Redis 过期：会控 Redis 仅做会前会中查询，预定（修改）会议时，设置过期时间为（结束时间 - 预定时间）+ 1 年；

快速会议，默认 1 小时结束，设置过期时间为 1 小时 + 1 个月；

▶︎ 存量会议：需要补充存量会议的路由信息，否则需要双（多）查不同区域的路由服务。

路由表扩展性强，无需迁移存量业务数据，但**面临诸多问题****：**

▶︎ 引入了新的关键依赖路由中心，并且路由中心没有经过生产大流量的验证，风险较大；

▶︎ 可用性问题，是否要考虑路由中心挂了的情况，如果挂了是否降级兜底直连路由存储？

▶︎ 路由中心本身又引入了存储，这涉及到后续的扩展性和异地多活各种问题，方案很重，运维复杂，增加成本；

▶︎ 因为拆分有按地域和个人企业维度，为性能考虑，依赖会议后台全链路打通透传，这也是一项大工作；

▶︎ 过期时间问题，就会控业务来说只关注会前和会中数据，但是接入层也会有会后的查询请求，这里的路由过期时间设置多久？

▶︎ 无法兼顾成员信息数据；

▶︎ 各种本地缓存排查问题困难。

综合考虑，最终放弃了此方案，而选择了在会议 ID 编码路由信息的方式。

# 06

会议 ID 路由编码

参考 QQ 看点和浏览器内容中心的 rowkey 设计，其将区域和时间戳等信息预埋至 rowkey，那么能否在会议 ID 这里采用类似的思路？

会议 ID 是一个 uint64 位的整数，较之字符串编码有一定的难度。前面提到会议业务路由规则复杂，涉及拆分维度多，我们的思路是将各种隔离维度策略下的存储进行编排，编排后分配一个唯一的编号，然后将按编号信息编码到会议 ID。收到请求的时候根据会议 ID 解码出存储位置。

## 1. 6.1 存储编排

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe964d4J6z3T18icFOJdEWRIJ94cxCWu6fVxCWCpAz558AAnxA22BF8T5LewbU2mCf44w9B2G2QjzAzg/640?wx_fmt=png)

编排方案支持 SET 粒度的存储隔离，不过业务暂无此需求。

## 2. 6.2 会议 ID 编码

会议后台已经运行了 3 年，已经有一套自己的 ID 生成规则；加之复杂的业务应用场景也对会议 ID 强加了各种隐形规则，梳理了现在会议 ID 的各种限制条件： 

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe964d4J6z3T18icFOJdEWRIJ9iabLNbajHDSgg6RQibPjCemakVa8ZMK8Zl2jksyWyPoYAEGoSiatj2b7g/640?wx_fmt=png)

业务整改难度大，我们还是希望尽量兼容；我们的目的就是在不影响业务逻辑的情况下，重塑会议 ID 生成规则，平滑迁移。

旧编码规则确认：幸运的是，没有业务利用旧会议 ID 的编码规则做逻辑，都是当作一个随机数处理；

存储编码位数确认：X 位二进制，0～(2^X - 1) 共 2^X 套存储；

分组会议：分组会议将会议 ID 作为 ZSet 的 score，score 是一个浮点数，浮点数有什么问题了？（网传 Go 语言之父曾吐槽：不懂浮点数不配当码农，哈哈哈，我赶快恶补下）。

浮点数表示的范围很大，但精度有限，如按规范化表示，双精度尾数为 53 位，对应十进制 9007199254740992，16 位十进制数有效数字，因此分组会议 ID 的值不能超过此值。

考虑到分组会议的量级较小，号段分配范围过大浪费空间，最终决定 0～48 共 49 位二进制用于分组会议 ID，如不够用后续可考虑回收复用（分组会议的业务场景如此）。普通会议范围 (0x0001ffffffffffff, 0xffffffffffffffff] 和分组会议的号段范围 [0, 0x0001ffffffffffff] 不交叉。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjngAqKNZuaialIeLXkdREW7k5t1mUiaxYcQhLZNoI8PFkdsSQZwXXXibDfA/640?wx_fmt=png)

成本最小化，增量会议 ID 需要全量长期存储用于去重，成本较高。我们的做法是减少增量会议的存储容量：

在 64 位整数中选取 T 位用来表示相对时间（20230302 项目启动时间），T 位二进制可表示 2^T 天，超过后回绕；因为每个 Redis 实例对应一个唯一的编号，故对于增量会议来说，各实例间必不会重复；

对于同一个实例，每天生成的会议 ID 必不相同（由相对时间保证）；对于同一个实例同一天新生成的会议 ID，由于会议 ID 有一定的过期时间（大于 3 天），而且会控每次写 Redis 都采用 lua 原子方式，实现类似于 SETNX 的能力，这就保证了同实例同天会议 ID 的唯一性，以及时钟回拨导致的问题；

**相对时间位置：**由于分组会议的号段范围限制，为统一处理，存储编码和相对时间位置被限定在 0~48 位；

存量会议 ID 也需要用于去重，全量存储的话需要 300+G 的 Redis 内存，扫描存量会议 ID，发现用第 e 和 f 位的话，相对时间为 50 年内的所有数据只占 3G 空间。最终形成了上图的编码格式。

## 3. 6.2.1 编码规则

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjn84eMLYibfyEYJojmXGzG8eZxNdGUsKMlAqY2BMV2NHStJOArla2URPg/640?wx_fmt=png)

编码规则主要用于生成会议 ID 的服务，部署区域由 TKEX_REGION 环境变量获取，新增部署区域需要强制开发同学感知存储，即需要配置区域对应的编码规则；现考虑广州上海多活，现在需要在北京部署服务逻辑层，那么北京是使用广州的存储还是上海的存储？由开发同学指定。

配置兜底编码规则 *;*;*，如果 TKEX_REGION 出问题则兜底至 26 号（企业）实例，仍然可以正常路由，只不过流量不均衡；

SET 暂时全部配置为通配 *，后续若要支持某个 SET 单独一套存储，可以简单扩展；

最后提一下，上层业务可以自己定制策略决定会议的存储，如将付费用户路由至企业编号（不建议），以解燃眉之急。

## 4. 6.3 容量分析

现假设存储编号占用 X=4Bits，相对时间占用 T=14Bits，X+T 一共占用 18Bits，则每天的会议 ID 容量：

分组会议 31Bits，最大值 2^31，可支持每天平均 24800/s 的消耗；普通会议 46Bits，最大值 2^46 - 2^31，可支持每天平均 8 亿 / s 的消耗；

结论：容量足够。

## 5. 6.3.1 冲突碰撞

会议 ID 的随机数我采用 C++11 的梅森旋转 19937_64 和均匀分布生成；为什么不采用全局唯一 ID 生成服务？

冲突的根本原因是每次生成的会议 ID 是完全随机的独立事件，对于分布式 ID 生成，业界也有比较成熟的方案，如百度基于雪花算法的 UidGenerator，美团基于 DB / 雪花算法的 Leaf。

会议没有现成可用的全局唯一 ID 生成服务，做为引入的关键依赖，分布式 ID 生成服务的降级、性能、容灾、异地和高可用等需要较多的开发工作量和生产流量验证；基于质量和每天的期望冲突次数考虑，我们采用冲突重试的方案，尽量减少依赖。

## 6. 6.3.2 冲突预估

前面提到存量会议在相对时间 50 年内的量较少，先计算增量会议在一天内的冲突概率和期望值。

普通会议按 1kw / 天计，则一天中请求冲突的概率为 0，1/2^46，2/2^46，...，9999999/2^46，冲突期望次数为（0+1+2+...+9999999)/2^46 = 0.71 次 / 天；

分组会议按 6w / 天计（现在约为 5.8w / 天），则一天中请求冲突的概率为 0，1/2^31, 2/2^31,...,59999/2^31，冲突期望次数为（0+1+2+...+59999)/2^31 = 0.84 次 / 天；

从数学理论层面证明了每天的最大期望重试次数是完全可以接受的！！！

写一个小 Demo 执行 5 次， 冲突次数如下：

<table selecttype="cells"><colgroup><col width="155"><col width="203"><col width="219"><col width="162"><col width="101"><col width="101"></colgroup><tbody><tr height="24"><td width="166.33333333333334"><br></td><td width="5.333333333333314"><section><span>第一次</span></section></td><td width="65.33333333333333"><section><span>第二次</span></section></td><td width="55.33333333333333"><section><span>第三次</span></section></td><td width="78.33333333333333"><section><span>第四次</span></section></td><td width="124"><section><span>第五次</span></section></td></tr><tr height="24"><td width="243"><section><span>普通会议 1kw</span></section></td><td width="5.333333333333314"><section><span>0</span></section></td><td width="65.33333333333333"><section><span>0</span></section></td><td width="55.33333333333333"><section><span>1</span></section></td><td width="78.33333333333333"><section><span>0</span></section></td><td width="124"><section><span>1</span></section></td></tr><tr height="24"><td width="243"><section><span>分组会议 10w</span></section></td><td width="5.333333333333314"><section><span>3</span></section></td><td width="65.33333333333333"><section><span>1</span></section></td><td width="55.33333333333333"><section><span>1</span></section></td><td width="78.33333333333333"><section><span>3</span></section></td><td width="124"><section><span>3</span></section></td></tr><tr height="24"><td width="243"><section><span>分组会议 20w</span></section></td><td width="5.333333333333314"><section><span>9</span></section></td><td width="65.33333333333333"><section><span>12</span></section></td><td width="55.33333333333333"><section><span>7</span></section></td><td width="78.33333333333333"><section><span>11</span></section></td><td width="124"><section><span>10</span></section></td></tr></tbody></table>

生产视图符合预期：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjnXPUmibEkslsaCFjIlusQR2PGMmHHJpbicWvic6Hqibp14MoibrczbmqNhsA/640?wx_fmt=png)

## 7. 6.3.3 冲突优化

期望冲突次数越大，因为要重试，故请求耗时越高；由于存量数据，每天的期望冲突次数应该会大于上述计算次数，可以进一步做如下优化：

每次请求生成两个随机数，全部冲突则冲突，降低冲突概率：

▶︎ 普通会议：  0，0，(2*2)/(2^46 * 2^46)，(3*3)/(2^46 * 2^46)，...，（9999999*9999999)/(2^46 * 2^46)；

▶︎ 分组会议：  0，0，(2*2)/(2^31 * 2^31)，(3*3)/(2^31 * 2^31)，...，（9999999*9999999)/(2^31 * 2^31)；

每天的冲突期望次数：

▶︎ 普通会议：（2*2 + 3*3 + ... + 9999999*9999999)/(2^46 * 2^46) = [n*(n+1)*(2n+1)/6 - 1] / (2^46 * 2^46) = 0.0000004 次；

▶︎ 分组会议：（2*2 + 3*3 + ... + 99999*99999)/(2^31 * 2^31) = [n*(n+1)*(2n+1)/6 - 1] / (2^31 * 2^31) = 0.00001 次；

现考虑存量数据 37 亿，由随机事件的乘法原理，固定为某一编号和相对时间值的期望数量为 37 亿 * 1/2^X * 1/2^T=1.3W，冲突概率很小了（相当于每次冲突的次数增加 1.3W，再除以样本空间）。

## 8. 6.4 会议 ID 去重

会议 ID 的全局唯一性是整个会议数据的基础，会议 ID 重复可能会看到其他人的会议，导致严重的安全越权。存储拆分后如何保证会议 ID 的全局唯一性？加之在会议 ID 中进行了编码，进一步导致重复的概率增大。由容量分析可知冲突的概率较小，但对业务来说却是无法容忍的。前面讲到增量会议的冲突很容易解决，但存量会议无任何规则，新生成的会议 ID 肯定有概率与其冲突，如何处理？

首先看我们的方案：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe964d4J6z3T18icFOJdEWRIJ9ajM6BA3A1kJ2050FqPjhuTPDhE3yjULSDbRkvWBiaTprDWTuPshqicdw/640?wx_fmt=png)

新增了去重 Redis，去重 Redis 仅导入全部存量数据的会议 ID，那么问题来了去重 Redis 网络调用失败如何处理？从安全层面来讲这应该是一个关键依赖，如果重试最终调用失败则请求返回失败，但如果去重 Redis 异常会导致整个会议 app 不可创建会议，是存在一定风险的，不过 Redis 的 SLA 是有保证的。

对于异地集群需要跨城调用，当然上海或者北京调度到广州其实距离不是问题。

## 9. 6.4.1 shadow key 方案优化

作为项目 owner，说实话对上述方案并不是很满意，因为新增了存储依赖（还有逻辑层面的风险），并且有跨城调用，很不优雅，我给出了一种新方案：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe964d4J6z3T18icFOJdEWRIJ9Nw3FCCiat4LtvUcfpoicaDe8PxNOYNmDSccTY4VFvMNqsmRrWCuopA5A/640?wx_fmt=png)

因为会议信息是必定要存储的，会议 ID 编号划分后，每个集群只可能和（按新规则解析）相同编号的存量会议 ID 冲突，为不引入依赖组件，我们就利用会议信息 Redis 本身的存储，将存量会议按编号进行迁移。

基于成本大小考虑，我们只迁移会议 ID 用于去重（会议 ID 为 hash tag），VAL 为 empty。预订会议时，在 lua 中先查下会议 ID 是否和当前实例 shadow key 冲突。

那要迁移多少数据了？很容易估算，XBits 一共 2^X 套存储，存量会议 ID 完全随机，则每个实例需要迁移的期望数据量为：300G（总存量 ID 大小）* （1/2^X），约等于 10G；如果再考虑相对时间为 50 年内的数据，则占用空间更少。

如此一来，就去掉了跨城调用以及每次请求去重的 RPC。

## 10. 6.5 存量会议

一个比较棘手的问题：按 X 个 Bits 划分号段后，存量会议如何处理？

因为不同号段的 ID 会存储于不同的 Redis 实例，我的做法是按编号规则解析，将存量数据按编号进行迁移（现在看起来可能比较容易，不过当时想到这的时候确实兴奋不已）。

如此一来就将问题转化为 DB 分库了，只不过分库策略是会议 ID 的存储编号，具体分库方案将在第 8 章仔细讨论。那么号码不在 [0～2^X) 区间内的 ID 如何处理？用旧实例兜底承接即可（也可用新实例兜底，需要数据迁移）。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjn0nOhfJH7xMUlWDKXBddviceMxc9vLF4kRhjvsl2aGnqiakCTrc3pkX1Q/640?wx_fmt=png)

按照新的解析规则，存量的企业会议可能会被判定存储于个人实例，个人会议也可能被判定为企业实例，增量会议无影响。新实例也可用于企业数据的灾备。

## 11. 6.6 异地多活

▶︎ 搭建上海 SET，北极星权重初为 0；

▶︎ 将广州个人企业实例数据分别跨地域复制到上海：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe964d4J6z3T18icFOJdEWRIJ9VAibib78tBlPuCu0asicQmknfibkgHaaOZAETWJ82H2XDXbASJyqSYhnSQ/640?wx_fmt=png)

考虑新增上海，将流量切换到上海的情况，此时手动更改配置；

上海 SET 放量过程中，由于存量会议（编号新规则之前的存量，新规则之后且放量之前全部为广州）和接入层机器无法做到转发配置同时生效，可能会在灰度生效过程中出现广州和上海并发更改同一个会议的情况，不过这种场景出现概率较小，要同时满足如下条件：

▶︎ 配置生效过程中，修改存量会议（会议 ID 编号新规则之前的数据）；

▶︎ 并发修改

▶︎ 请求恰好落到配置生效和未生效的机器；

本身概率极小，另外可以在晚上低峰操作。

再考虑广州挂掉，流量切换到上海的情况，同样手动更改配置，直接将接入层所有流量路由到上海，此时配置的路由规则强制覆盖会议 ID 路由规则，因为广州和上海的 Redis 实例通过跨地域复制双向同步，切换后仍可以正常服务，由于跨城同步延迟，在切换瞬间服务是有损的，不过对于会控业务是可以接受的：比如用户开麦，流量切换到上海后，开麦信息可能又丢失了，用户重试操作下即可。

上海多活放量时，需要预先把广州生成的编号为上海的存量数据迁移到上海，比如存量预订的会议；如果广州挂掉所有的流量都将切至上海，因此灾难切换前需要把全部有效的存量数据迁移至上海。

## 12. 6.7 会议 ID 路由

## 13. 6.7.1 路由规则

由上述分析，加上存量数据，会议 ID 总的解析规则如下：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe964d4J6z3T18icFOJdEWRIJ9DFt8ic4hibzyrtk8PI0WHnkj90BPqXHUpCbJ9B0iavsp1SKicLicLvsqhcA/640?wx_fmt=png)

cowork 上已经提供会议 ID 解析小工具。

## 14. 6.7.2 路由配置

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjn4hsvEWCtArPoZBjKN3oN84wdHDD8mGDrcYm9qaX2unMrHriadctSFWQ/640?wx_fmt=png)

deployed_db_code 为已经部署的存储编号集合，对于存量会议按规则解析的编号如果在此集合中，则直接路由至编号存储（见 6.5 数据会迁移）；否则采用兜底 bottom 存储；

对于 m2u 兜底的是个人新实例，存量数据仍然需要按分库流程迁移至新实例，此方式对企业旧实例更加安全。

## 15. 6.7.3 路由优先级

区域发生故障后需要将流量调度到其他区域，这需要人工介入发布配置，因为区域判死是比较高危的行为。接入层和内部网关此时应优先选择配置路由策略，即配置路由的优先级高于会议 ID 路由。

## 16. 6.8 业务梳理

会议后台生成会议 ID 的服务有会议管理、会控、PMI 会议和分组会议。其他的服务都只会根据会议 ID 路由。

## 17. 6.9 容灾演练

对会议核心存储做了这么大的改造，其价值是什么？效果如何？

模拟个人流量冲击，不影响企业服务；模拟广州挂掉，流量切换到上海仍可以正常继续提供服务；制定计划，定期进行容灾演练。

## 18. 6.10 风险

▶︎ 会议 ID 设计规则复杂，增加了冲突概率；

▶︎ 无会议 ID 请求无法支持，需要单独处理；

▶︎ 支持的存储套数有限，最多 2^X 套；

▶︎ 会议 ID 编码后就固定了路由规则，后续只有通过配置来控制路由优先级（配置路由 > 会议 ID 路由）。

## 19. 6.11 小插曲之成员列表

成员列表在 Redis 中存储的为一个 Hash 结构，和会议信息、成员信息不同的是，memberlist 的 value 并无 SEQ 序列号（val 前 8 个字节的一个无符号数，通过 lua 做 CAS 并发控制）。

因为担心通用的分库方式（第 8 章）在数据迁移过程中，由于成员列表的并发请求导致的不一致难以收敛，也讨论了一个**备用**方案：即在预订会议时，在会议信息中存储成员列表的 Redis 实例。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjncUx96oXNAvXIRQlu5njRSObFklwbWWTkBoO8QhXMSW03B7zee0yIOg/640?wx_fmt=png)

显然此方案对业务耦合较重，业务还需要关注路由逻辑；另外，在做存储访问收拢时发现调用成员列表的场景较多，此方案需要先查询会议信息，获取到成员列表路由信息，然后将其层级透传至成员列表查询接口。

另外，这将导致成员列表的路由规则和会议信息的路由规则不一致。

# 07

平行扩展

特殊时间期间的请求量已达 Redis 单实例的处理极限，很自然的想法就是水平扩展为多个实例，拆分策略为按会议 ID 取模，如此一来问题又被转化为 DB 分库的问题了，哈哈哈，神奇吧，具体我们在下一章节讨论。

特殊时间之后 PCU 下降了不少，评估 ROI 不高，结论是暂不用支持多实例。

## 1. 7.1 成倍扩容

按会议 ID 取模实现起来很简单，那么后续如何扩容了？答案是采用成倍扩容的方式：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjn8PibEyzg7AfPtnDXvkQfWia3mCIbqEl3micjavmvuicWKYkKLpzJiaMlsicA/640?wx_fmt=png)

假设已经做了多实例扩展，%2=0 的路由至实例 11，%2=1 的路由至实例 21；现在简单分析下扩容步骤。

▶︎ 新增实例 12 和实例 22；

▶︎ %2=0 的流量拆分为 %4=0 和 %4=2 的两部分，分别由实例 11 和实例 12 承接；

▶︎ %2=1 的流量拆分为 %4=1 和 %4=3 的两部分，分别由实例 21 和实例 22 承接；

▶︎ 存量数据迁移。

给大家分析一下：

一个正数 a%2=0，若 a < 4，则 a 为 2，则 a%4=2；

若 a >= 4，则 a 可以写为 2m，m>=2，因此 a=2m=2(2*n+r)=4n + 2r => a%4=0 或者 2，其中 r 为 0 或者 1；

一个正数 a%2=1，若 a < 4，则 a 为 1 或 3，则 a%4=1 或 3；

若 a >= 4，则 a 可以写为 2m+1（奇数），m>=2，因此 a=2m+1=2(2*n+r)+1=4n+2r+1，其中 r 为 0 或者 1。

# 08

分库

前面提到，无论是个人企业隔离，异地多活，还是水平扩展，本质上都是一个分库操作，只不过采用的是不同的拆分策略。内部多个团队都做过或者正在做 DB 拆分，大家使用的方案都不太相同，并且体量也没有会控业务大，不能完全参考。这里总结了会控业务的存储分库成功实践，目的是形成一套方法论，沉淀下来一套可以复用的工具，以供后续业务参考。

## 1. 8.1 拆分要求

分库流程满足可回退、可观测、可灰度；

数据迁移方向：为保证企业服务的稳定，将个人会议数据迁出到新实例；

切换过程：平滑迁移；

业务感知：业务透明。

## 2. 8.2 方案调研

经过调研和讨论权衡，一共如下四种方案：

▶︎ 路由表：前面已经讲过；

▶︎ 双写 + 双读 + 回写：增量数据双写，存量数据采用类似懒加载的方式处理：新实例读不到则读旧实例，然后更新到新实例；数据迁移周期较长；

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjnXppRIPaG96oNtDOyVXapAPdakJueOCzibXsxW2PBZVZYibZghg8rFWag/640?wx_fmt=png)

▶︎ 双写 + 存量数据迁移：增量数据双写，存量数据采用工具迁移；

▶︎ Redis 自身支持分库能力：正常来讲，分库分表应该是 DB 的基本能力；如果 Redis 本身支持分库则能业务透明，代价和风险都做到最小；支持分库也非常有价值和通用：如 QQ 业务的 256 分片的大实例，会议基础数据组的头像 Redis 业务分库，不过需要 Redis 侧支持，时间不可控。

## 3. 8.3 会控分库方案

我们最终选择的双写 + 存量数据迁移方案。会议到点入会的特性决定了分库过程中需要尽量满足高一致读，用户 a 入会写主存储，用户 b 入会需要读主存储，否则由于同步延迟，读备可能读不到数据 a，造成会中不可见的问题。当然，如果你的业务场景只需要保证最终一致，或者读写体量比较小，或者流程上无需灰度、回退，我觉得可以视情况简单处理。

**为什么迁移存量数据，不选择双读？**会控业务是典型的写少读多，读的服务 50+（虽然有收拢，但实际开发过程中收拢和分库工作并行），基于工作量和完成时间考虑，双读耗时较长。

**为什么会议信息和成员信息需要迁移存量数据？**历史原因，会议业务功能多，涉及团队多，在飞速发展过程中，为了功能快速上线，部分业务将会控存储做为持久存储，加之之前缺乏规范管控，会议信息和成员信息本身也没有设置过期时间；

如果不迁移存量数据，则需要保证存量数据（如会议结束后的数据）删除后是否对业务无影响，没人能给出明确的结论，全部梳理的话工作量比较大，大部分业务经历过多次架构调整和重构变更，新老服务并存，很难证明梳理整改是完备的，整个事情将变得不可控。因此**基于质量优先的原则**，决定对存量数据进行迁移。当然，我们本身也在收拢存储，且加了监控（读存量数据的请求）上报，以及推动其他业务团队梳理的事情并行，不过这个耗时可能比较长，比如云录制场景，可能会查一年前的会议信息，个人中心页面会查询两年前的会议，但这种查询是相当低频的，如果是企业用户没有查询到会议信息导致回放录制失败报障，对会议业务的口碑也有影响。

**成员列表是否需要迁移存量数据？**理论上可以不迁移，主要基于现有逻辑事实：

▶︎ 会议结束则会删除成员列表；

▶︎ 会议媒体房间一个月过期，会议失效。

不过项目进度比较紧，1 个月时间太久，因此也一并迁移。

会控信令的一般处理流程是先查询存储（会议信息、成员信息等），执行业务逻辑，再更新回存储，具体分库操作有如下 5 个步骤：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe964d4J6z3T18icFOJdEWRIJ9WkTOHhhTmicqTSrIxHkOkaBVAlDiaib1x3lQhHK7sqRuybMtqgG5PicMAA/640?wx_fmt=png)

会控数据格式说明：会议信息和成员信息为 KV 结构，VAL 前 8 个字节为 SEQ 无符号整数，用于 lua CAS（严格判断 SEQ 相等）更新。

成员列表和 u2m 为 hash 结构，进退会更新，VAL 无 SEQ；m2u 为 zset 结构，VAL 无 SEQ；其中会议信息的 key 为会议 ID，成员信息的 key 格式为 member_tinyID_ 企业 appid，成员列表的 key 格式为 member_list_{会议 ID}。

## 4. 8.3.1 双写

目的是保证增量数据的一致性。

串行双写，先写旧实例，再写新实例；为保证分库过程中遇到异常随时可回退，我们始终以旧实例为基准，新旧实例双写，确保整个灰度过程中旧实例始终拥有全量的可靠的数据。

会控很多场景要求高一致读，因此必须写新实例完成后返回，这样客户端或者上层逻辑才会根据返回结果执行下一条信令。如果写旧实例完成后返回成功，异步写新实例，客户端认为信令执行成功发起下一条信令，若用户命中灰度，则会读取新实例，MQ 异步消费写入新实例存在时延，导致信令可能查询不到或者读到旧数据，进而出现各种诡异问题。

▶︎ 写旧实例失败，则向上返回失败；

▶︎ 写旧实例成功，写新实例成功，向上返回成功；

▶︎ 写旧实例成功，写新实例失败，通过写 Kafka **向前补偿。**

此时返回成功还是失败了？大部分场景返回成功即可，事实上会控数据迁移过程中也是如此处理的。此时就退化成了异步写的场景，不过考虑到失败场景很少，加之我们做了重试和 MQ 补偿，可以认为一定会成功，当然如果你的业务需要严格保证一致，则直接返回失败。数据修复可以选择向前补偿或向后补偿，向前补偿更加简单，直接以旧实例为基准覆盖新实例即可。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjnMzBMfD7kx8FtbgTDurlz7NtFicict9MkgZ3DZuxzq0gp5uHWPNmUVXbA/640?wx_fmt=png)

## 5. 8.3.1.1 重试

写新实例失败，则重试两次；仍然失败，则写 kafka 重试。失败继续重试两次；还失败，则置于延迟队列定时重试，最后人工介入。总之，可以保证写新实例一定可以补偿成功，且采用向前补偿的方式。

需要注意的是，写新实例失败包括网络失败和 SEQ 冲突失败全部需要重试，对于 SEQ 冲突这里重点说明下：

▶︎ 双写阶段 SEQ 不一致无需直接重试（请求原封不动再调用一次 RPC），灰度读阶段写新实例 SEQ 不一致需要重试，因为可能中间某个 SEQ 还未更新完成。

请求 1 读旧实例 seq1，请求 1 用 seq1 更新旧实例，旧实例 SEQ 变为 seq2；请求 2 读旧实例 seq2，请求 2 用 seq2 更新旧实例，请求 2 用 seq2 更新新实例 SEQ 不一致；请求 1 用 seq1 更新新实例，新实例 SEQ 变为 seq2，请求 2 重试成功；

▶︎ 会控服务在上层逻辑会再次查询会议信息，更新字段，设置 Redis（0 这条路径），重试一次。

最后，写旧实例失败也重试（网络失败 + SEQ 冲突），对于灰度读的用户是查新实例先写旧实例在写新实例，写旧实例失败也重试，降低失败率；不过和写新实例失败的重试策略有些许不同：

▶︎ 网络失败和新实例一样，直接重试即可；

▶︎ SEQ 冲突，由于流程上先写旧实例再写新实例，因此 SEQ(旧实例) >= SEQ(新实例)，冲突则可推出此时 SEQ(旧实例) 大于 SEQ(新实例)，需要上层逻辑重新查询，更新字段，重试；直接重试必冲突；

对于未灰度读放量阶段，SEQ 冲突重试需要上层业务重新查询会议信息，更新字段，更新存储，否则直接重试更新 Redis 必冲突；对于灰度放量阶段，SEQ 冲突重试可依据写新失败还是写旧失败选择直接重试和上层业务重试相结合。

对于无 SEQ 的数据，只会存在网络失败的情况。当然，由于失败毕竟是小场景，全部重试也无大的问题。

## 6. 8.3.1.2 并发问题

双写面临的一个比较头疼的问题就是并发。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe964d4J6z3T18icFOJdEWRIJ9qToVhc0cp8px3giczaJ5Cb2JsXOBwAxgKU9v0xibhG73pN7KxIl96j0g/640?wx_fmt=png)

对于没有 SEQ 的数据，因为读写无法保证原子性，各请求执行时序导致数据不一致，解决方案可见 8.3.3 对账。

## 7. 8.3.1.3 并行双写

并行双写几乎不会增加请求时延，但可能出现写旧实例失败，写新实例成功的异常情况。灰度放量后多个请求并发，无法保证新旧实例 SEQ 的全局单调递增：命中灰度的用户，读新实例，并行写新旧实例；未命中灰度的用户，读旧实例，并行写新旧实例；对账修复；

并发组合起来情况比较复杂，较之串行多了回退修复：旧实例写失败，新实例写成功，小 SEQ 覆盖大 SEQ 修复。

加之代码实现层面串行更加简单，耗时也可以接受，我们实际操作过程中都是采用串行双写。

串行先写旧实例保证了新旧实例的 SEQ 必定全局单调递增，相当于从业务层面对 SEQ 加了一把全局锁，实时 / 人工修复工具逻辑就变得简单：大 SEQ 覆盖小 SEQ 一定可以保证业务逻辑正确。总之，越简单越不容易出错。

## 8. 8.3.2 存量数据迁移

目的是保证存量数据的一致性。

存量数据迁移可以全量 SCAN 旧 Redis，将满足条件的 Key 迁移至新 Redis。因为此时已经有双写，故数据迁移和双写可能存在并发时序问题。不容质疑地，双写优先级更高，故数据迁移应该遵循 set If Not Exist 语义：set key val nx/hsetnx/zadd key nx score member 等，如不支持 nx 语义，直接插入即可，一般来说系统不会有那么大的并发，或者选择流量低峰操作。

## 9. 8.3.3 对账

目的是发现新旧实例数据的差异，决策是否可以切换流量到新实例；增量数据和存量数据都一致了，何时可以将流量切换至新实例？有没有一个量化指标来表明我们可以把流量切至新实例无问题？

新旧实例数据 100% 相同则可以切换，如何判别两个存储实例的数据差异了？答案就是对账。

## 10. 8.3.3.1 实时对账

前面已经讲过，双写是先写旧实例再写新实例。

实时对账的目的是尽量减小对业务的影响，因为双写写新实例失败，会直接返回成功，我们应该在用户发起下一次操作前修复好数据。

触发条件：

▶︎ 1）写旧实例失败，存在很多场景写旧实例 SDK 返回失败，但是 Redis 实际成功；

▶︎ 2）写旧实例成功，写新实例失败；

▶︎ 3）写操作全量对账；如果对一致性要求很高，则可以采用此种方式，一般的业务 1）和 2）足矣；如成员列表的 VAL 无 SEQ，并发就会有时序问题，但实际操作过程中发现少有由于并发时序导致的数据不一致。

对账步骤：

▶︎ 新旧实例双查，对比数据是否一致；

▶︎ 一致，则返回；

▶︎ 不一致，则以旧实例为基准，修复新实例的数据。

如果希望对生产影响尽量小，修复新实例的数据可以采用 Lua CAS 的方式；SET 之前先和双查出来的数据比较，若相同则修复，否则说明有其他服务更新放弃修复；如果数据有 SEQ，用 SEQ 做比较值；

如果数据没有 SEQ，可以直接将 VAL 作为比较值；此时极限并发下可能存在 ABA（了解 C++ 无锁的同学应该知道）的问题：假设某时刻旧实例数据为 a，新实例为 b，a 为正确的数据，现在实时对账需要 CAS 修复新实例 b 为 a（还未修复），但此时业务并发请求恰好双写将双方的数据都修改为 b，b 是最终正确的数据，由于 CAS 修复和业务请求不是在同一个事务中，导致 CAS 又将新实例的数据修复为 a（时序发生在业务请求后修复），a 在此时已经是错误的数据。当然这种情况出现的概率较小，即便是出现我们通过离线对账来发现。

▶︎ 修复失败，则重试 1～3 步骤；

▶︎ 重试失败，则置于延迟队列，定时执行 1～3 步骤；

▶︎ 延迟修复失败，则告警，人工介入，工具修复。

## 11. 8.3.3.2 定时对账

定时对账的目的是发现新旧实例数据的不一致，可以认为是兜底。根据经验对账很容易发现业务本身的设计缺陷，这也是对账的价值所在，会控的对账过程中发现了很多模块设计缺陷。

由于 Redis SCAN 的有限保证，SCAN 过程中只有新增和删除的 key 是 undefined，这部分是增量数据由双写保证一致性。

**▶︎ 双向对账：**定时对账需要正向对账和反向对账结合使用：

正向对账：遍历旧实例，和新实例对比；一般来说这个是全量对账，数据量较大，遍历一次耗时较长，可以发现包括双写（写切换至新实例）是否遗漏、业务缺陷等所有问题；执行频率可以稍长，比如 2 小时；

反向对账：遍历新实例，和旧实例对比；一般来说新实例数据量较小，执行时间短，可以快速发现问题；执行频率可以稍快，比如 30 分钟；

▶︎ **结果播报：**对账的结果需要及时播报知会。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjnHowz9T9YfXo5z0AC2ODibeZYDHCXUE3OLVcaibJ4oyOh8ntd1nibgTpAw/640?wx_fmt=png)

▶︎ **延迟对账：**业务在双写，对账在遍历读取，由于不是同一个事务，很可能对账读取的时刻，业务双写才写完旧实例，新实例还没有写完，这也会导致不一致造成干扰。我们的解决方案是，对于对账不一致的数据延迟 30 秒再对账，甚至可以根据具体情况多延迟对账几次，完美解决。

▶︎ **数据修复：**对账出来不一致的数据需要立即修复，可以在对账完成后调用修复工具自动化完成；自动修复的前提是对账已经没有未知问题了，所有问题基本上都已经很清楚了，否则程序自动化修复有一定风险；自动化之前还是要靠人工排查和修复。

由于需要修复多种不同 KEY 对应的 VAL 和 TTL，经常用配置来操作比较容易出错，我们的解决方案是每一种修复类型的工具预先代码写死编译好，使用的时候直接执行就可以了，数据治理我一共准备了 20 + 个工具，哈哈哈。  

实时对账和修复工具采用的是大 SEQ 覆盖小 SEQ；对于无 SEQ 的数据，采用 VAL 作为 CAS 更新，这和实时对账面临同样的问题；不过不用担心，有定时对账兜底，其实很快就收敛了（用实践说话）。

▶︎ **并行对账：**我们需要遍历的数据有 20 亿 +，需要 128 个分片并行对账，否则时延不可接受。

## 12. 8.3.3.3 数据过滤

在实际对账的过程中，比如 u2m 和 m2u，由于功能设计缺陷导致报名和解散会议信令并发导致数据不一致，经分析此不一致对用户影响很小或无影响，则可先过滤掉这部分数据，同时业务并行修复；

## 13. 8.3.4 灰度读

两边数据对账一致后，则可以开始灰度切读流量到新实例了，同时由于双写，存储收拢工作可以并行不阻塞。灰度读放量后，写新实例失败的业务表现和双写补偿策略密切相关。

## 14. 8.3.4.1 写顺序

切换读的时候，写是否也要切换为先写新实例再写旧实例？

写不需要切换，仍然是先写旧实例再写新实例。主要基于如下两点考虑：

▶︎ 对整个切换过程持悲观态度，随时准备切换至旧实例，旧实例一定要有全量且可靠的数据；

▶︎ 从开发同学压力角度考虑。

写顺序切换是通过配置控制的，配置发布期间是无法保证所有服务所有 POD 同时生效。这样就可能存在并发请求部分落到配置生效的机器，部分落到配置未生效的机器，导致 SEQ 相同且数据内容不一致，这种情况修复数据必定是有损的，当然可以选择晚上低峰期间操作，因为灰度放量需要多次操作，这对开发同学压力较大，另外本人不太喜欢深夜发布。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjnFbqKn7npK3mybibepynp6uDywmNREicuUEVAvV4zC69lPPQTdlM3zKiaQ/640?wx_fmt=png)

那很多同学就不爽了，我读流量都已经切换到新实例了，旧实例只是一个灾备，但写的时候还是要先写旧实例，不太合理。其实这没啥大问题，你就把旧实例写当作一次 RPC 就可以，并且云 Redis 承诺的 SLA 不低于 99.99%，其可靠性是有保障的。配置发布生效时间不一致导致的数据不一致并非危言耸听，事实上我们在灰度读之后由于特殊原因需要等一个月才能停双写，我也对先写旧实例不爽，于是在一个月黑风高 (2023.05.30 00:30) 的夜晚切换了写顺序，切换过程中并发信令导致了 SEQ 相同但 VAL 不同的记录，这些通过工具修复是有损的，不过庆幸的是，这个并发信令是 user_status 的 LEAVE 和 C2S 的 LEAVE 信令，用户已经退出了，无影响。这相当于做了一个折衷，灰度读 100% 后统一切换顺序，只操作一次。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjny6YFzzqnsyiaJXhDvxa7BpFJ1T1Dh0EyyZ0t2BomDTeaxiatzcVV8hUw/640?wx_fmt=png)

## 15. 8.3.4.2 正确性分析

串行双写，灰度放量后先读新实例再依次写旧实例新实例，这对业务逻辑是否有影响？下面可以分析下入会场景：

用户入会添加成员列表，业务逻辑处理后，再查询成员列表 PUSH 扩散；

假设 a 和 b 两个用户入会且命中灰度读，C 表示旧实例 (企业)，P 表示新实例 (个人)，则 a 和 b 入会流程：

a（写 C），a（写 P）， a（读 P）

b（写 C），b（写 P）， b（读 P）

考虑并发，按会议 ID 灰度，时序有如下排列：

1 ) a（写 C）， a（写 P），b（写 C），b（写 P）=> b（读 P）push a，b 读 P 的时候能读到 a，PUSH 后 a 能看到 b，无问题。

2) b（写 C），b（写 P），a（写 C），a（写 P）=> a（读 P）push b，类似 1），无问题。

3) a（写 C），b（写 C），b（写 P），a（写 P）=> a  (读 P)  push b，a 读 P 的时候能读到 b，PUSH 后 b 能看到 a，无问题。

4) a（写 C），b（写 C），a（写 P)，b（写 P）=> b（读 P）push a，类似 3），无问题。

5) b（写 C），a（写 C），a（写 P），b（写 P）=> b  (读 P)  push a，类似 3），无问题。

6) b（写 C），a（写 C），b（写 P)，a（写 P）=> a（读 P）push b，类似 3），无问题。

## 16. 8.3.4.3 监控和灰度

灰度读流量时如何确保业务无问题？可以思考如果新实例数据异常的表现，分两种情况：  
▶︎ 读到旧数据，如有 SEQ，则写请求 SEQ 不一致监控；如无 SEQ，放量观察日志，采样对账新旧实例；

▶︎ 数据不存在，在存储代理提前对查询不存在做好监控，放量的时候对比同比环比，尽量做到万无一失，这里注意的是对于 hash 和 zset 不存在返回的是 empty，kv 返回的是 nil。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97HDGkQ3Ijev7GON32HJrjnNsgzHDtpazjAVqib3U6QKuRJObXSZYzh7b7t6RHCbC79eVDMn0FSg4Q/640?wx_fmt=png)

放量的时候 golang 服务和 spp 服务分开按比例放量，golang 通常是读取，spp 一般是先读取然后再写。

## 17. 8.3.5 收尾

停双写：前面提到读请求都已经切换到新实例了，但是写请求还是先写旧实例，如鲠在喉；在这一步骤终于可以停写旧实例了，不过别急，也需要灰度逐步停止；

停双写的前提是读请求已经 100% 的切换至新实例，这里的难点在于很难找到一个指标来保证流量全部收拢。

代码清理：大量的灰度逻辑需要清理掉，以免影响后续的代码整洁和维护；

脏数据清理：流量已经全权由新实例接管，旧实例的数据需要清理；

## 18. 8.3.6 回退演练

先在预发 SET 停双写，对账工具过滤掉预发 SET 的数据，保证全量拨测用例通过；为防止停双写灰度过程中出现生产问题，比如某个服务收拢遗漏了，造成了严重的事故（概率较小，预发 SET 核心功能已验证通过），我们做了回退演练预案：  

▶︎ 恢复双写；

▶︎ 对账工具按停双写的比例对账，目的是尽快找出异常数据；

▶︎ 根据对账结果工具修复数据，修复方向新实例到旧实例；

在恢复双写这一步骤：

▶︎ 如果数据无 SEQ，无问题；

▶︎ 如果数据有 SEQ，且判断 SEQ 大于等于则更新，无问题；

▶︎ 如果数据有 SEQ，且严格判断 SEQ 相等才更新，由于停双写导致旧实例的数据 SEQ 落后，因此写旧实例必失败；解决方案可以改造 SEQ 判断代码，或者在灰度读 100% 观察一段时间后新增一个切换写顺序步骤，如前所述在晚上低峰期全量操作一次，如遇异常则可回滚至此阶段。

# 09

未来展望 & 后续计划

最好的方式是将分库流程做成平台化的组件，可供后续业务复用。不过事情太多了，实在卷不动了。

异地容灾和多活，基础工作已经基本完成，后续计划更多的是梳理业务依赖，部署和验证。

-End-

原创作者｜印俊

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe964d4J6z3T18icFOJdEWRIJ9NhNCWVdk1wV36gZgutOEyiaT6J0uM4wdfKicK0BkR8cneVJfQH0Aic9gQ/640?wx_fmt=png)

看完这篇文章，你对异地容灾和多活有什么看法？欢迎分享。我们将选取 1 则最有意义的评论，送出腾讯云开发者 - 手提袋 1 个（见下图）。9 月 27 日中午 12 点开奖。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/8BctpkuniaENbmgicbFggKbzZqS0uUu7CibhBTl96jB6skvYLbqxblFKQHGFdCbibv6EQSoXkFiaJLaHPCFmZCicMXBQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

📢📢欢迎加入腾讯云开发者社群，社群专享券、大咖交流圈、第一手活动通知、限量鹅厂周边等你来~

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Ap0CNBADIU4jiacH4CONuDN4M5kdp8KapCTJxNv9ZM3Qv0JXLD7IIcRib2ZeQWnPpol4WFImUE7JrwOLbs85HJww/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

（长按图片立即扫码）

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe979Bb4KNoEWxibDp8V9LPhyjmg15G7AJUBPjic4zgPw1IDPaOHDQqDNbBsWOSBqtgpeC2dvoO9EdZBQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

[

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe964d4J6z3T18icFOJdEWRIJ9fPfQ1tFXjGvFzoznChibCUj0cAicWU4WC3jfKCYkW2skZU7sxd4zhUfw/640?wx_fmt=png)

](http://mp.weixin.qq.com/s?__biz=MzI2NDU4OTExOQ==&mid=2247657137&idx=1&sn=aaf08c399ee70dd5deb33b083da9f038&chksm=eaa67ce1ddd1f5f7c722a5c7fb66abc690629387dd57825a54405ccd9a9ed8501ede3ddc2aa2&scene=21#wechat_redirect)

[

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe964d4J6z3T18icFOJdEWRIJ9kOkaGhaIPYp9RhFGKC6lhklkDG1sQibNdBIpNp4ibXLjGGaSbiaF7rRWA/640?wx_fmt=png)

](http://mp.weixin.qq.com/s?__biz=MzI2NDU4OTExOQ==&mid=2247658622&idx=1&sn=27295d0d9090ac6952d3112edd597cda&chksm=eaa6762eddd1ff38e4a8012b0717b5e747f9e1749146ef3e295d9f493919c5fd12d4e704a6f4&scene=21#wechat_redirect)  

[

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe964d4J6z3T18icFOJdEWRIJ9mwTqos77TrHctBVq5EEQVPRcwvEjqWINVJibibcZH5a4lv5oW65W0UhA/640?wx_fmt=png)

](http://mp.weixin.qq.com/s?__biz=MzI2NDU4OTExOQ==&mid=2247659012&idx=1&sn=26e77753efdf3291cc89319ecb14a01c&chksm=eaa67454ddd1fd429cbde9be40df276828aeacb14fba6a06f0eb9c5a9e67c5953c03d1449125&scene=21#wechat_redirect)