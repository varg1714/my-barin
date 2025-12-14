---
source: https://mp.weixin.qq.com/s/wCbg-CTcg_Xt5KB3dLYNpA
create: 2025-11-23 22:30
read: true
knowledge: true
knowledge-date: 2025-11-24
tags:
  - Redis
  - 系统运维
summary: "[[得物自建 Redis 运维经验]]"
---
![](http://mmbiz.qpic.cn/mmbiz_gif/AAQtmjCc74DZeqm2Rc4qc7ocVLZVd8FOASKicbMfKsaziasqIDXGPt8yR8anxPO3NCF4a4DkYCACam4oNAOBmSbA/640?wx_fmt=gif&wxfrom=5&wx_lazy=1#imgIndex=0)

**目录**

一、前言

二、规模现状

三、技术演进介绍

 1. 自建 Redis 系统架构

 2. 接入方式演进

 3. 同城双活就近读

 4. Redis-server 版本与能力

 5. 实例架构与规格

 6. proxy 限流

 7. 自动化运维

四、总结

**一**

**前 言**

自建 Redis 上线 3 年多以来，一直围绕着技术架构、性能提升、降低成本、自动化运维等方面持续进行技术演进迭代，力求为公司业务提供性能更高、成本更低的分布式缓存集群，通过自动化运维方式提升运维效率。

本文将从接入方式、同城双活就近读、Redis-server 版本与能力、实例架构与规格、自动化运维等多个方面分享一下自建 Redis 最新的技术演进。

**二**

**规模现状**

随着公司业务增长，自建 Redis 管理的 Redis 缓存规模也一直在持续增长，目前自建 Redis 总共管理 1000 + 集群，内存总规格 160T，10W + 数据节点，机器数量数千台，其中内存规格超过 1T 的大容量集群数十个，单个集群最大访问 QPS 接近千万。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74CHdibCD8w9EDpEk7SfqRcABoIUtfqwfBdfEnKB5laePZLVr0q6EexCZP1sAqfP3OofN8xCzPCFialw/640?wx_fmt=png&from=appmsg#imgIndex=1)

**三**

**技术演进介绍**

**自建 Redis 系统架构**

下图为自建 Redis 系统架构示意图：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74CHdibCD8w9EDpEk7SfqRcABId1pOYWBI8zXsBx3GyQGV4P2Gnib5OQPJ0blpGXALL9RrsAh0tkKicWA/640?wx_fmt=png&from=appmsg#imgIndex=2)

自建 Redis 架构示意图

自建 Redis 集群由 Redis-server、Redis-proxy、ConfigServer 等核心组件组成。

*   Redis-server 为数据存储组件，支持一主多从，主从多可用区部署，提供高可用、高性能的服务；
    
*   Redis-proxy 为代理组件，业务通过 proxy 可以像使用单点实例一样访问 Redis 集群，使用更简单，并且在 Redis-proxy 上提供同区优先就近读、key 维度或者命令维度限流等高级功能；
    
*   ConfigServer 为负责 Redis 集群高可用的组件。
    

自建 Redis 接入方式支持通过域名 + LB、service、SDK 直连（推荐）等多种方式访问 Redis 集群。

自建 Redis 系统还包含一个功能完善的自动化运维平台，其主要功能包括：

*   Redis 集群实例从创建、proxy 与 server 扩缩容、到实例下线等全生命周期自动化运维管理能力；
    
*   业务需求自助申请工单与工单自动化执行；
    
*   资源（包含 ECS、LB）精细化管理与自动智能分配能力、资源报表统计与展示；
    
*   ECS 资源定期巡检、自动均衡与节点智能调度；
    
*   集群大 key、热 key 等诊断与分析，集群数据自助查询。
    

下面将就一些重要的最新技术演进进行详细介绍。

**接入方式演进**

自建 Redis 提升稳定性的非常重要的一个技术演进就是自研 DRedis SDK，业务接入自建 Redis 方式从原有通过域名 + LB 的方式访问演进为通过 DRedis SDK 连接 proxy 访问。

**LB 接入问题**

在自建 Redis 初期，为了方便业务使用，使用方式保持与云 Redis 一致，通过 LB 对 proxy 做负载均衡，业务通过域名（域名绑定集群对应 LB）访问集群，业务接入简单，像使用一个单点 Redis 一样使用集群，并且与云 Redis 配置方式一致，接入成本低。

随着自建 Redis 规模增长，尤其是大流量业务日渐增多，通过 LB 接入方式的逐渐暴露出很多个问题，部分问题还非常棘手：

*   自建 Redis 使用的单个 LB 流量上限为 5Gb，阈值比较小，对于一些大流量业务单个 LB 难以承接其流量，需要绑定多个 LB，增加了运维复杂度，而且多个 LB 时可能会出现流量倾斜问题；
    
*   LB 组件作为访问入口，可能会受到网络异常流量攻击，导致集群访问受损；
    
*   由于 Redis 访问均是 TCP 连接，LB 摘流业务会有秒级报错。
    

**DRedis 接入**

自建 Redis 通过自研 DRedis SDK，通过 SDK 直连 proxy，不再强依赖 LB，彻底解决 LB 瓶颈和稳定性风险问题，同时，DRedis SDK 默认优先访问同可用区 proxy，天然支持同城双活就近读。

DRedis SDK 系统设计图如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74CHdibCD8w9EDpEk7SfqRcABX4Kw5CdHpXQEIQxhOqZKGdskv8sj9LGqc2SF9GsMdoC18ibTu00xfbA/640?wx_fmt=png&from=appmsg#imgIndex=3)

Redis-proxy 启动并且获取到集群拓扑信息后，自动注册到注册中心；可通过管控白屏化操作向配置中心配置集群使用的 proxy 分组与权重、就近读规则等信息；DRedis SDK 启动后，从配置中心获取到 proxy 分组与权重、就近读规则，从注册中心获取到 proxy 节点信息，然后与对应 proxy 节点建立连接；应用通过 DRedis SDK 访问数据时，DRedis SDK 通过加权轮询算法获取一个 proxy 节点（默认优先同可用区）及对应连接，进行数据访问。

DRedis SDK 并且对原生 RESP 协议进行了增强，添加了一部分自定义协议，支持业务灵活开启就近读能力，对于满足就近读规则的 key 访问、或者通过注解指定的就近读请求，DRedis SDK 通过自定义协议信息，通知 proxy 在执行对应请求时，优先访问同可用区 server 节点。

DRedis SDK 目前支持 Java、Golang、C++（即将上线）三种开发语言。

*   Java SDK 基于 Redisson 客户端二次开发，后续还会新增基于 Jedis 二次开发版本，供业务灵活选择，并且集成到 fusion 框架中
    
*   Golang SDK 基于 go-Redis v9 进行二次开
    
*   C++ SDK 基于 brpc 二次开发
    

**DRedis 接入优势**

业务通过 DRedis SDK 接入自建 Redis，在稳定性、性能等方面都能得到大幅提升，同时能降低使用成本。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74CHdibCD8w9EDpEk7SfqRcABic6OEMlU1QaBYpG1YGJExlW3wJ3EOeVia6SH0TUAERkbrSJn6jibMcc1g/640?wx_fmt=png&from=appmsg#imgIndex=4)

社区某应用升级后，业务 RT 下降明显，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74CHdibCD8w9EDpEk7SfqRcAB3fVq6ldydqFUFZnKzlCxnDL4cibB2UKEhsPEZsmdGkkGSHT0opiaqgNA/640?wx_fmt=png&from=appmsg#imgIndex=5)

**DRedis 接入现状**

DRedis SDK 目前在公司内部大部分业务域的应用完成升级。

Java 和 Golang 应用目前接入上线超过 300+。

**同城双活就近读**

自建 Redis 同城双活采用**中心写就近读**的方案实现，可以降低业务多区部署时访问 Redis RT。

同城双活就近读场景下，业务访问 Redis 时，需要 SDK 优先访问同可用区 proxy，proxy 优先访问同可用区 server 节点，其中 proxy 优先访问同区 server 节点由 proxy 实现，但是在自研 DRedis SDK 之前，LB 无法自动识别应用所在同区的 proxy 并自动路由，因此需要借助 service 的同区就近路由能力，同城双活就近读需要通过容器 proxy+service 接入。

自建 Redis 自研 DRedis SDK 设计之初便考虑了同城双活就近读需求，DRedis 访问 proxy 时，默认优先访问同区 proxy。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74CHdibCD8w9EDpEk7SfqRcABKqaGDJ8JW3eq73ibJWo5AsMpt33NPhjDNAqs45TyGtoa8PrVYwufKCQ/640?wx_fmt=png&from=appmsg#imgIndex=6)

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74CHdibCD8w9EDpEk7SfqRcABN9KibRZrWJFsGwNZedJnjs1111Sh9JUSrbCzEibe2FriblXGAVokRkZ6g/640?wx_fmt=png&from=appmsg#imgIndex=7)

**service 接入问题**

目前，自建 Redis server 和 proxy 节点基本都是部署在 ECS 上，并且由于 server 节点主要消耗内存，而 proxy 节点主要消耗 CPU，因此默认采用 proxy + server 节点混部的方式，充分利用机器的 CPU 和内存，降低成本。

而为了支持同城双活就近读，需要在容器环境部署 proxy，并创建 service，会带来如下问题：

*   运维割裂，运维复杂度增加，除了需要运维 ECS 环境部署节点，额外增加了容器环境部署方式。
    
*   成本增加，容器环境 proxy 需要独立机器部署，无法与 server 节点混部，造成成本增加。
    
*   RT 上升，节点 CPU 更高，从实际使用效果来看，容器环境 proxy 整体的 CPU 和响应 RT 都明显高于 ECS 环境部署的节点。
    
*   访问不均衡，service 接入时，会出现连接和访问不均衡现象。
    
*   无法定制化指定仅仅少量特定 key 或者 key 前缀、指定请求开启就近读。
    

**DRedis 接入**

自建 Redis 自研 DRedis SDK 设计之初便考虑了同城双活就近读需求，DRedis 访问 proxy 时，**默认优先访问同区 proxy**；当同可用区可用 proxy 数量小于等于 1 个时，启用调用保护，DRedis 会主动跨区访问其他可用区 proxy 节点。

通过 service 接入方式支持同城双活就近读，是需要在 proxy 上统一开启就近读配置，开启后，对全局读请求均生效，所有读请求都默认优先同区访问。

由于 Redis 主从复制为异步复制，主从复制可能存在延迟，**理论上在备可用区可能存在读取到的不是最新数据**。

某些特定业务场景下，业务可能在某些场景能够接受就近读，但是其他一些场景需要保证强一致性，无法接受就近读，通过 service 接入方式时无法灵活应对这种场景。

DRedis SDK 提供了两种方式供这种场景下业务使用：

*   支持指定 key 精确匹配或者 key 前缀匹配的方式，定向启用就近读。
    
*   Java 支持通过声明式注解（@NearRead）指定某次请求采用就近读；Golang 新增 80 个类似 xxxxNearby 读命令，支持就近读。
    

使用以上两种方式指定特定请求使用就近读时，无需 proxy 上统一配置同区优先就近读。默认情况下，所有读请求访问主节点，业务上对 RT 要求高、一致性要求低的请求可以通过以上两种方式指定优先同区就近读。

**Redis-server 版本与能力**

在自建 Redis 初期，由于业务在前期使用云 Redis 产品时均是使用 Redis4.0 版本，因此自建 Redis 初期也是选择 Redis4.0 版本作为主版本，随着 Redis 社区新版本发布，结合当前业界使用的主流版本，自建 Redis 也新增了 Redis6.2 版本，并且将 Redis6.2 版本作为新集群默认版本。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74CHdibCD8w9EDpEk7SfqRcABhueKd9Rw8QaFdlTg6JL8eYbUsIUemhYMtOyicqxLXabzLibjgicOv5TWA/640?wx_fmt=png&from=appmsg#imgIndex=8)

不管是 Redis4.0 还是 Redis6.2 版本，均支持了多线程特性、实时热 key 统计能力、水平扩容异步迁移 slot 能力，存量集群随着日常资源均衡迁移调度，集群节点版本会自动升级到同版本的最新安装包。

*   **多线程特性**
    

Redis6.2 版本支持 IO 多线程，在 Redis 处理读写业务请求数据时使用多线程处理，提高 IO 处理能力，自建 Redis 将多线程能力也移植到了 Redis4.0 版本，测试团队测试显示，开启多线程，读写性能提升明显。

![](https://mmbiz.qpic.cn/mmbiz_jpg/AAQtmjCc74CHdibCD8w9EDpEk7SfqRcAB3bRHOB0V5Spr9XQZy4vtOyVInsajRt7IdBzAwRicZAPvrGWCBajia2YA/640?wx_fmt=jpeg&from=appmsg#imgIndex=9)

多线程版本 VS 普通版本

![](https://mmbiz.qpic.cn/mmbiz_jpg/AAQtmjCc74CHdibCD8w9EDpEk7SfqRcABCCmToiaq5jyzWRQqZIqEnkaFeLz9oPThia4oGv9rOKX851PX70UcH2wQ/640?wx_fmt=jpeg&from=appmsg#imgIndex=10)

多线程版本 VS 云产品 5.0 版本

*   **实时热 key 统计**
    

自建 Redis4.0 和 Redis6.2 版本均支持 Redis 服务端实时热 key 统计能力，管控台白屏化展示，方便快速排查热 key 导致的集群性能问题。方案详细可阅读[《基于 Redis 内核的热 key 统计实现方案》](https://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247535466&idx=1&sn=af408a50f7045d2be82b801c9a2cd90a&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74CHdibCD8w9EDpEk7SfqRcABjuqv68b2ibes1IdrUKSIOtWeGMsHA24WJAcz9oqeOb6bnLgdVkeiaoYQ/640?wx_fmt=png&from=appmsg#imgIndex=11)

*   **水平扩容异步迁移**
    

自建 Redis 支持水平扩容异步数据迁移，解决大 key 无法迁移或者迁移失败的稳定性问题，支持多 key 并发迁移，几亿 key 数据在默认配置下水平扩容时间从**平均 4 小时缩短到 10 分钟**，**性能提升 20 倍**，对业务 **RT 影响下降 90% 以上**。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74CHdibCD8w9EDpEk7SfqRcAB2aYyk6xAu1iah3QPGfNX7beRqKESx8yTwW8ia0XjleMZibDyhQPSWPsLw/640?wx_fmt=png&from=appmsg#imgIndex=12)

算法某实例 2.5 亿 key 水平扩容花费时间和迁移过程对业务 RT 影响

**实例架构与规格**

**Redis 单点主备模式**

自建 Redis 实例默认均采用集群架构，但是通过 proxy 代理屏蔽集群架构细节，集群架构对业务透明，业务像使用一个单点 Redis 实例一样使用 Redis 集群。

但是集群架构下，由于底层涉及多个分片，不同 key 可能存在在不同分片，并且随着水平扩容，key 所在分片可能会发生变化，因此，集群架构下，对于一些多 key 命令（如 eval、evalsha、BLPOP 等）要求命令中所有 key 必须属于同一个 slot。因此集群架构下，部分命令访问与单点还是有点差异。

实际使用中，有少数业务由于依赖了一些开源的三方组件，其中可能由于存储非常少量的数据，所以使用到 Redis 单点主备模式实例，因此，考虑到这种场景，自建 Redis 在集群架构基础上，也支持了 Redis 单点主备模式可供选择。

**一主多从规格**

自建 Redis 支持一主多从规格用于跨区容灾，提供更快的 HA 效率，当前**支持一主一从（默认），一主两从、一主三从 3 种副本规格**，支持配置**读写分离**策略提升系统性能（一主多从规格下，开启读写分离，可以有多个分片承接读流量）

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74CHdibCD8w9EDpEk7SfqRcABQoZWea3MEuYBClQtTwHiaQaytic6O7xuWFhQlE2oYYy6bzYHMsXEs5qQ/640?wx_fmt=png&from=appmsg#imgIndex=13)

一主一从

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74CHdibCD8w9EDpEk7SfqRcABahscrZWib29JKUqkwoz0KxOwqep4xvf5syHHrtIBNicOsjkcKVJY0tUg/640?wx_fmt=png&from=appmsg#imgIndex=14)

一主两从

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74CHdibCD8w9EDpEk7SfqRcABoCVroicTfxcH0gJnlRr1nYibmz1LuE9ZOQRJw6NyCHTQGx8YicKibj1wlA/640?wx_fmt=png&from=appmsg#imgIndex=15)

一主三从

*   一主一从时默认主备可用区各部署一个副本（master 在主可用区）
    
*   一主两从时默认主可用区部署一主一从，备可用区部署一从副本
    
*   一主三从时默认主可用区部署一主一从，备可用区部署两从副本
    

**proxy 限流**

为了应对异常突发流量导致的业务访问性能下降，**自建 Redis-proxy 支持限流能力**。

有部分业务可能存在特殊的已知大 key，业务中正常逻辑也不会调用查询大 key 全量数据命令，如 hgetall、smembers 等，查询大 key 全量数据会导致节点性能下降，极端情况下会导致节点主从切换，因此，自建 Redis 也支持配置命令黑名单，**在特定的集群，禁用某些特定的命令**。

*   支持 key 维度限流，指定 key 访问 QPS 阈值
    
*   支持命令维度限流，指定命令访问 QPS 阈值
    
*   支持命令黑名单，添加黑名单后，该实例禁用此命令
    

**自动化运维**

自建 Redis 系统还包含一个功能完善的自动化运维平台，一直以来，自建 Redis 一直在完善系统自动化运维能力，通过丰富的自动化运维能力，实现集群全生命周期自动化管理，资源管理与智能调度，故障自动恢复等，提高资源利用率、降低成本，提高运维效率。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74CHdibCD8w9EDpEk7SfqRcABXNbHdSyEHKiaUEBEf5dKj0mO5lFIvBn4n933icqPbobuWSialcqfoKg5A/640?wx_fmt=png&from=appmsg#imgIndex=16)

*   **资源池自动化均衡调度**
    

自建 Redis 资源池支持**按内存使用率自动化均衡调度、按内存分配率自动化均衡调度、按 CPU 使用率均衡调度、支持指定机器凌晨迁移调度（隐患机器提前维护）等**功能，均衡资源池中所有资源的负载，提高资源利用率。

*   **集群自动部署与下线**
    

当业务提交集群申请工单审批通过后，判断是否支持自建，如符合自建则自动化进行集群部署和部署结果校验，校验集群可用性后自动给业务交付集群信息，整个过程高效快速。

业务提交集群下线工单后，自动检测是否满足下线条件，比如是否存在访问连接，如满足下线条件，则自动释放 proxy 资源，保留 7 天后自动回收 server 节点资源，在 7 天内，如果存在特殊业务仍在使用的情况，还支持快速恢复使用。

*   **资源管理**
    

对 ECS 机器资源和 LB 资源进行打标，根据特殊业务需要做不同资源池的隔离调度，支持在集群部署与扩容时，资源自动智能化分配。

*   **集群扩缩容**
    

自建 Redis 支持 server 自动垂直扩容，业务申请集群时，可以选择是否开启自动扩容，如果开启自动扩容，当集群内存使用率达到 80% 时，系统会自动进行垂直扩容，对业务完全无感，快速应对业务容量上涨场景。

ecs-proxy，docker-proxy 扩容，server 节点的扩缩容也支持工单自动化操作，业务提交工单后，系统自动执行。

*   工单自动化
    

当前 80% 以上的运维场景已完成工单自动化，如 Biz 申请、创建实例、密码申请、权限申请、删除 key、实例升降配，集群下线等均完成工单自动化。业务提单审批通过后自动校验执行，执行完成后自动发送工单执行结果通知。

*   **告警自动化处理**
    

系统会自动检测机器宕机事件，如发现机器宕机重启，会自动拉起机器上所有节点，快速恢复故障，提高运维效率。

关于自建 Redis 自动化运维能力提升详细设计细节，后续会专门分享，敬请期待。

**四**

**总结**

本文详细介绍了自建 Redis 最新技术演进，详细介绍了自研 DRedis SDK 优势与目前使用现状，以及 DRedis 在同城双活就近读场景下，可以更精细化的控制部分请求采用优先同区就近读。

介绍了自建 Redis 目前支持最新的 Redis6.2 版本，以及在 Redis4.0 和 Redis6.2 版本均支持多线程 IO 能力、实时热 key 统计能力、水平扩容异步迁移能力。自建 Redis 除了支持集群架构，也支持单点主备架构实例申请，同时支持一主多从副本规格，可以提供可靠性和读请求能力（读写分离场景下）。自建 Redis-proxy 也支持多种限流方式，包括 key 维度、命令维度等。

自建 Redis 自动化运维平台支持强大的自动化运维能力，提高资源利用率，降低成本，提高运维效率。

自建 Redis 经过长期的技术迭代演进，目前支持的命令和功能上完全对比云 Redis，同时，自建 Redis 拥有其他一些特色的能力与优势，比如不再依赖 LB、支持自动垂直扩容、支持同区优先就近读等。

**往期回顾**

1. [Golang HTTP 请求超时与重试：构建高可靠网络请求｜得物技术](https://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247541618&idx=1&sn=3d81d0aa7624f329ad9741007bf4a2b9&scene=21#wechat_redirect)

2. [RN 与 hawk 碰撞的火花之 C++ 异常捕获｜得物技术](https://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247541611&idx=1&sn=d997c79fb5746d7be6de462178794421&scene=21#wechat_redirect)

3. [得物 TiDB 升级实践](https://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247541552&idx=1&sn=ed3e6d033bdc80f369f008feafbb6691&scene=21#wechat_redirect)

4. [得物管理类目配置线上化：从业务痛点到技术实现](https://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247541530&idx=1&sn=a00c3e72e03feb1938d29cae6b4a21bb&scene=21#wechat_redirect)

5. [大模型如何革新搜索相关性？智能升级让搜索更 “懂你”｜得物技术](https://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247541503&idx=1&sn=6bb4217d88785544e4afa31c96de920a&scene=21#wechat_redirect)

文 / 竹径

关注得物技术，每周一、三更新技术干货

要是觉得文章对你有帮助的话，欢迎评论转发点赞～

未经得物技术许可严禁转载，否则依法追究法律责任。

“

**扫码添加小助手微信**

如有任何疑问，或想要了解更多技术资讯，请添加小助手微信：

![](https://mmbiz.qpic.cn/mmbiz_jpg/AAQtmjCc74CHdibCD8w9EDpEk7SfqRcABTCjz1Tgsnss0OUdIcibYEBsLFGrQONdTo7sGibzmw7icsJzA2gcia4icPjg/640?wx_fmt=jpeg&from=appmsg#imgIndex=17)