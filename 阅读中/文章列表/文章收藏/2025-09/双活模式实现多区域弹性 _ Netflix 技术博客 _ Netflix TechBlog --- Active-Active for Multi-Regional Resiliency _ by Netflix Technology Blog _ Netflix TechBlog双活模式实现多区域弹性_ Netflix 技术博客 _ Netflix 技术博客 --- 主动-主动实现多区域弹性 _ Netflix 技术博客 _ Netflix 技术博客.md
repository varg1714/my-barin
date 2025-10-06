---
source: https://netflixtechblog.com/active-active-for-multi-regional-resiliency-c47719f6685b
create: 2025-09-15 12:02
read: true
knowledge: true
knowledge-date: 2025-10-02
tags:
  - 系统架构
summary: "[[Netflix 的双活模式实现]]"
---
[

![](https://miro.medium.com/v2/resize:fill:64:64/1*BJWRqfSMf9Da9vsXG9EBRQ.jpeg)





](https://netflixtechblog.medium.com/?source=post_page---byline--c47719f6685b---------------------------------------)

_by Ruslan Meshenberg, Naresh Gopalani, and Luke Kosewski  
作者：Ruslan Meshenberg、Naresh Gopalani 和 Luke Kosewski_

In June, [we talked about Isthmus](https://medium.com/@Netflix_Techblog/isthmus-resiliency-against-elb-outages-d9e0623484f3) — our approach to achieve resiliency against region-wide ELB outage. After completing the Isthmus project we embarked on the next step in our quest for higher resiliency and availability — a full multi-regional Active-Active solution. This project is now complete, and Netflix is running Active-Active across the USA, so this post aims to highlight some of the interesting challenges and learnings we found along the way.  
6 月份， [我们讨论了 Isthmus——](https://medium.com/@Netflix_Techblog/isthmus-resiliency-against-elb-outages-d9e0623484f3) 一种实现区域级 ELB 故障弹性的方法。在完成 Isthmus 项目后，我们着手迈向更高弹性和可用性的下一步——打造一个完整的多区域双活解决方案。该项目现已完成，Netflix 已在美国各地运行双活方案。因此，本文旨在重点介绍我们在此过程中遇到的一些有趣挑战和经验教训。

## Failure — function of scale and speed.  
失败——规模和速度的函数。

In general, failure rates an organization is dealing with depend largely on 2 factors: scale of operational deployment and velocity of change. If both scale and speed are small, then most of the time things just work. Once scale starts to grow, even with slow velocity, the chance of hardware failure will increase. Conversely, even at small scale, if velocity is fast enough, chance of software failure will increase. Ultimately, if you’re running at scale and still pursuing high velocity — things will break all the time.  
一般来说，一个组织所面临的故障率主要取决于两个因素：运营部署的规模和变化速度。如果规模和速度都很小，那么大多数情况下一切都会顺利进行。一旦规模开始增长，即使速度很慢，硬件故障的几率也会增加。相反，即使规模很小，如果速度足够快，软件故障的几率也会增加。最终，如果你在大规模运营的同时仍然追求高速度——事情总是会出问题的。

Of course, not all failures are created equal. The types of failures that we’ll focus on in this post are the most important and difficult to deal with — complete and prolonged service outages with unhappy customers flooding customer service lines, going to twitter to express their frustration, articles popping up across multiple publications announcing “service X is down!”.  
当然，并非所有故障都是一样的。本文将重点讨论那些最重要且最难处理的故障类型——服务全面且长时间中断，不满意的客户涌向客服热线，在推特上发泄不满，各种出版物上都出现“X 服务宕机了！”的文章。

At Netflix, our internal availability goals are 99.99% — which does not leave much time for our services to be down. So in addition to deploying our services across multiple instances and Availability Zones, we decided to deploy them across multiple AWS Regions as well. Complete regional infrastructure outage is extremely unlikely, but our pace of change sometimes breaks critical services in a region, and we wanted to make Netflix resilient to any of the underlying dependencies. In doing so, we’re leveraging the principles of Isolation and Redundancy: a failure of any kind in one Region should not affect services running in another, a networking partitioning event should not affect quality of service in either Region.  
在 Netflix，我们的内部可用性目标是 99.99%——这意味着我们的服务不会长时间处于宕机状态。因此，除了跨多个实例和可用区部署服务外，我们还决定跨多个 AWS 区域部署它们。区域基础设施完全中断的可能性极小，但我们快速的变更速度有时会中断某个区域的关键服务，因此我们希望 Netflix 能够灵活应对所有底层依赖关系。为此，我们运用了隔离和冗余原则：一个区域中的任何故障都不应影响另一个区域运行的服务，网络分区事件也不应影响任何一个区域的服务质量。

## Active-Active Overview 双活概述

In a nutshell, Active-Active solution gets all the services on the user call path deployed across multiple AWS Regions — in this case US-East-1 in Virginia and US-West-2 in Oregon. In order to do so, several requirements must be satisfied  
简而言之，双活解决方案将用户调用路径上的所有服务部署在多个 AWS 区域——在本例中，部署在弗吉尼亚州的 US-East-1 和俄勒冈州的 US-West-2。为此，必须满足几个要求

*   Services must be stateless — all data / state replication needs to handled in data tier.  
    服务必须是无状态的——所有数据/状态复制都需要在数据层处理。
*   They must access any resource locally in-Region. This includes resources like S3, SQS, etc. This means several applications that are publishing data into an S3 bucket, now have to publish the same data into multiple regional S3 buckets.  
    它们必须在区域内本地访问任何资源，包括 S3、SQS 等资源。这意味着，多个应用程序原本将数据发布到一个 S3 存储桶，现在必须将相同的数据发布到多个区域 S3 存储桶。
*   there should not be any cross-regional calls on user’s call path. Data replication should be asynchronous.  
    用户的调用路径上不应该存在跨地域调用。数据复制应该是异步的。

In a normal state of operation, users would be geo-DNS routed to the closest AWS Region, with a rough split of 50/50%. In the event of any significant region-wide outage, we have tools to override geo-DNS and direct all of users traffic to a healthy Region.  
在正常运行状态下，用户将通过地理 DNS 路由到最近的 AWS 区域，大致比例为 50/50%。如果发生任何严重的区域级中断，我们会提供工具来覆盖地理 DNS，并将所有用户流量引导至正常运行的区域。

![](https://miro.medium.com/v2/resize:fit:960/0*r4lMdnHelpceHsiq.)

![](https://miro.medium.com/v2/resize:fit:960/0*UtIOoUIw36dyrIZm.)

There are several technical challenges in order to achieve such a setup:  
为了实现这样的设置，存在几个技术挑战：

*   Effective tooling for directing traffic to the correct Region  
    将流量引导至正确区域的有效工具
*   Traffic shaping and load shedding, to handle thundering herd events  
    流量整形和负载削减，以应对惊群事件
*   State / Data asynchronous cross-regional replication  
    状态/数据异步跨区域复制

## DNS — Controlling user traffic with Denominator  
DNS — 使用 Denominator 控制用户流量

We direct a user’s traffic to our services via a combination of UltraDNS and Route53 entries, our Denominator project provides a single client library and command line that controls multiple DNS providers. There are several reasons why we ended up using a combination of two:  
我们通过 UltraDNS 和 Route53 的组合将用户流量引导至我们的服务，我们的 Denominator 项目提供了一个客户端库和命令行来控制多个 DNS 提供商。我们最终采用两者组合的原因如下：

*   UltraDNS provides us ability to directionally route customers from different parts of North America to different regional endpoints. This feature is supported by other vendors including Dyn, but is not supported in Route53. We didn’t want to use a latency based routing mechanism because it could cause unpredictable traffic migration effects.  
    UltraDNS 使我们能够将来自北美不同地区的客户定向路由到不同的区域端点。包括 Dyn 在内的其他供应商也支持此功能，但 Route53 不支持。我们不想使用基于延迟的路由机制，因为它可能会导致不可预测的流量迁移效应。
*   By using Route53 layer between UltraDNS and ELBs, we have an additional ability to switch user traffic, and the Route53 API provides reliable and fast configuration changes that are not a strong point for other DNS vendor APIs.  
    通过在 UltraDNS 和 ELB 之间使用 Route53 层，我们拥有了切换用户流量的额外能力，并且 Route53 API 提供了可靠且快速的配置更改，而这不是其他 DNS 供应商 API 的强项。
*   Switching traffic using a separate Route53 layer makes such change much more straightforward. Instead of moving territories with directional groups, we just move Route53 CNAMEs.  
    使用单独的 Route53 层切换流量，使此类更改更加直接。我们无需移动定向群组的区域，只需移动 Route53 CNAME 即可。

![](https://miro.medium.com/v2/resize:fit:960/0*TNZHtNYH5VrBg88K.)

## Zuul — traffic shaping Zuul——流量整形

We recently talked about [Zuul in great detail](http://techblog.netflix.com/2013/06/announcing-zuul-edge-service-in-cloud.html), as we opened this component to the community in June 2013. All of Netflix Edge services are fronted with the Zuul layer. Zuul provides us with resilient and robust ways to direct traffic to appropriate service clusters, ability to change such routing at runtime, and an ability to decide whether we should shed some of the load to protect downstream services from being over-run.  
我们最近[详细讨论了 Zuul](http://techblog.netflix.com/2013/06/announcing-zuul-edge-service-in-cloud.html) ，因为我们在 2013 年 6 月向社区开放了这个组件。所有 Netflix Edge 服务都以 Zuul 层为前端。Zuul 为我们提供了弹性且强大的方式，将流量引导到合适的服务集群，能够在运行时更改路由，并能够决定是否应该减轻部分负载以保护下游服务免于超负荷。

We had to enhance Zuul beyond it’s original set of capabilities in order to enable Active-Active use cases and operational needs. The enhancements were in several areas:  
为了支持双活用例和运维需求，我们必须在 Zuul 原有功能的基础上进行增强。增强功能主要体现在以下几个方面：

*   Ability to identify and handle mis-routed requests. User request is defined as mis-routed if it does not conform to our geo directional records. This ensures a single user device session does not span multiple regions. We also have controls for whether to use “isthmus” mode to send mis-routed requests to the correct AWS Region, or to return a redirect response that will direct clients to the correct Region.  
    能够识别和处理错误路由的请求。如果用户请求不符合我们的地理定向记录，则被定义为错误路由。这可确保单个用户设备会话不会跨越多个区域。我们还可以控制是否使用“isthmus”模式将错误路由的请求发送到正确的 AWS 区域，或返回重定向响应以将客户端定向到正确的区域。
*   Ability to declare a region in a “failover” mode — this means it will no longer attempt to route any mis-routed requests to another region, and instead will handle them locally  
    能够将某个区域声明为“故障转移”模式——这意味着它将不再尝试将任何错误路由的请求路由到另一个区域，而是在本地处理它们
*   Ability to define a maximum traffic level at any point in time, so that any additional requests will be automatically shed (response == error), in order to protect downstream services against a thundering herd of requests. Such ability is absolutely necessary in order to protect services that are still scaling up in order to meet growing demands, or when regional caches are cold, so that the underlying persistence layer does not get overloaded with requests.  
    能够定义任意时间点的最大流量级别，以便自动舍弃任何额外请求（响应 == 错误），从而保护下游服务免受海量请求的冲击。这种能力对于保护仍在扩展以满足不断增长的需求的服务，或者当区域缓存处于冷状态时，保护底层持久层不被请求过载至关重要。

All of these capabilities provide us with a powerful and flexible toolset to manage how we handle user’s traffic in both stable state as well as failover situations.  
所有这些功能为我们提供了强大而灵活的工具集，以管理我们如何在稳定状态和故障转移情况下处理用户流量。

## Replicating the data — Cassandra and EvCache  
复制数据 — Cassandra 和 EvCache

One of the more interesting challenges in implementing Active-Active, was the replication of users’ data/state. Netflix has embraced Apache Cassandra as our scalable and resilient NoSQL persistence solution. One of inherent capabilities of Apache Cassandra is the product’s multi-directional and multi-datacenter (multi-region) asynchronous replication. For this reason all data read and written to fulfil users’ requests is stored in Apache Cassandra.  
实施双活模式最有趣的挑战之一是用户数据/状态的复制。Netflix 已采用 Apache Cassandra 作为我们可扩展且具有弹性的 NoSQL 持久化解决方案。Apache Cassandra 的一项固有功能是其多向、多数据中心（多区域）异步复制功能。因此，所有为满足用户请求而读写的数据都存储在 Apache Cassandra 中。

Netflix has operated multi-region Apache Cassandra clusters in US-EAST-1 and EU-WEST-1 before Active-Active. However, most of the data stored in those clusters, although eventually replicated to the other region, was mostly consumed in the region it was written in using consistency levels of CL_LOCAL_QUORUM and CL_ONE. Latency was not such a big issue in that use case. Active-Active changes that paradigm. With requests possibly coming in from either US region, we need to make sure that the replication of data happens within an acceptable time threshold. This lead us to perform an experiment where we wrote 1 million records in one region of a multi-region cluster. We then initiated a read, 500ms later, in the other region, of the records that were just written in the initial region, while keeping a production level of load on the cluster. All records were successfully read. Although the test wasn’t exhaustive, with negative tests and comprehensive corner case failure scenarios, it gave us enough confidence in the consistency/timeliness level of Apache Cassandra, for our use cases.  
在双活模式推出之前，Netflix 已在 US-EAST-1 和 EU-WEST-1 运营多区域 Apache Cassandra 集群。然而，这些集群中存储的大部分数据虽然最终会被复制到另一个区域，但主要还是在写入该区域的使用 CL_LOCAL_QUORUM 和 CL_ONE 一致性级别的区域内被消费。在这种用例中，延迟并非大问题。双活模式改变了这一模式。由于请求可能来自美国任一区域，我们需要确保数据复制在可接受的时间阈值内完成。为此，我们进行了一项实验：在多区域集群的一个区域写入 100 万条记录。然后，我们在另一个区域启动读取操作，读取 500 毫秒后写入初始区域的记录，同时保持集群负载处于生产级别。所有记录均成功读取。尽管测试并不详尽，但通过负面测试和全面的极端故障场景，它让我们对 Apache Cassandra 的一致性 / 及时性水平有足够的信心，适合我们的用例。

![](https://miro.medium.com/v2/resize:fit:960/0*pW_25RrvfARQvPVD.)

Since many of our applications that serve user requests need to do so in a timely manner, we need to guarantee that data tier reads are fast — generally in a single-millisecond range. In some cases, we also front our Cassandra clusters with a Memcached layer, in other cases we have ephemeral calculated data that only exists in Memcached. Managing memcached in a multi-regional Active-Active setup leads to a challenge of keeping the caches consistent with the source of truth. Rather than re-implementing multi-master replication for Memcached, we added remote cache invalidation capability to our [EvCache client](https://medium.com/@Netflix_Techblog/announcing-evcache-distributed-in-memory-datastore-for-cloud-c26a698c27f7) — a memcached client library that we open sourced earlier in 2013. Whenever there is a write in one region, EvCache client will send a message to another region (via SQS) to invalidate the corresponding entry. Thus a subsequent read in another region will recalculate or fall through to Cassandra and update the local cache accordingly.  
由于我们许多处理用户请求的应用程序都需要及时处理，因此我们需要保证数据层读取速度极快——通常在毫秒级。在某些情况下，我们还会在 Cassandra 集群前面放置一个 Memcached 层；在其他情况下，我们拥有仅存在于 Memcached 中的临时计算数据。在多区域双活设置下管理 Memcached 带来了保持缓存与真实数据源一致的挑战。我们没有重新实现 Memcached 的多主复制，而是在 [EvCache 客户端](https://medium.com/@Netflix_Techblog/announcing-evcache-distributed-in-memory-datastore-for-cloud-c26a698c27f7) （一个我们在 2013 年初开源的 Memcached 客户端库）中添加了远程缓存失效功能。每当一个区域发生写入操作时，EvCache 客户端都会向另一个区域（通过 SQS）发送消息，使相应的条目失效。因此，另一个区域的后续读取操作将重新计算或直接发送到 Cassandra，并相应地更新本地缓存。

## Automating deployment across multiple environments and regions  
跨多个环境和区域自动部署

When we launched our services in EU in 2012, we doubled our deployment environments from two to four: Test and Production, US-East and EU-West. Similarly, our decision to deploy our North American services in Active-Active mode increased deployment environments to six — adding Test and Production in the US-West region. While for any individual deployment we utilize [Asgard](https://medium.com/@Netflix_Techblog/asgard-web-based-cloud-management-and-deployment-2c9fc4e4d3a1) — an extremely powerful and flexible deployment and configuration console, we quickly realized that our developers should not have to go through a sequence of at least 6 (some applications have more “flavors” that they support) manual deployment steps in order to keep their services consistent through all the regions.  
2012 年，当我们在欧盟推出服务时，部署环境数量从两个翻了一番，增至四个：测试和生产环境、美国东部和欧盟西部。同样，我们决定以双活模式部署北美服务，部署环境也随之增至六个，其中在美国西部地区增加了测试和生产环境。虽然对于任何单独的部署，我们都使用了 [Asgard——](https://medium.com/@Netflix_Techblog/asgard-web-based-cloud-management-and-deployment-2c9fc4e4d3a1) 一个功能强大且灵活的部署和配置控制台，但我们很快意识到，我们的开发人员不应该为了在所有地区保持一致的服务而经历至少六个（某些应用程序支持的“版本”更多）手动部署步骤。

To make the multi-regional deployment process more automated, our Tools team developed a workflow tool called Mimir, based on our open source Glisten workflow language, that allows developers to define multi-regional deployment targets and specifies rules of how and when to deploy. This, combined with automated canary analysis and automated rollback procedures allows applications to be automatically deployed in several places as a staged sequence of operations. Typically we wait for many hours between regional updates, so we can catch any problems before we deploy them world-wide.  
为了使多区域部署流程更加自动化，我们的工具团队基于开源 Glisten 工作流语言开发了一款名为 Mimir 的工作流工具，允许开发者定义多区域部署目标，并指定部署方式和时间的规则。结合自动化金丝雀测试和自动化回滚程序，该工具能够以分阶段操作的方式自动部署到多个位置。通常，我们会在区域更新之间等待数小时，以便在进行全球部署之前发现任何问题。

## Monkeys — Gorilla, Kong, Split-brain  
猴子——大猩猩、金刚、裂脑

We’ve talked a lot about our Simian Army — a collection of various monkeys we utilize to break our system — so that we can validate that our many services are resilient to different types of failures, and learn how we can make our system anti-fragile. Chaos Monkey — probably the most well known member of Simian Army runs in both Test and Production environments, and most recently it now includes Cassandra clusters in its hit list.  
我们之前就 Simian Army（一个由各种猴子组成的团队，我们用它来攻破系统）进行过多次讨论，目的是验证我们的众多服务能否抵御不同类型的故障，并学习如何提升系统的抗脆弱性。Chaos Monkey 可能是 Simian Army 中最知名的成员，它在测试环境和生产环境中均有运行，最近，它又将 Cassandra 集群纳入了攻击范围。

To validate our architecture was resilient to larger types of outages, we unleashed bigger Simians:  
为了验证我们的架构能够应对更大规模的中断，我们释放了更大的 Simians：

*   **Chaos Gorilla**. It takes out a whole Availability Zone, in order to verify the services in remaining Zones are able to continue serving user requests without the loss of quality of service. While we were running Gorilla before Active-Active, continued regular exercises proved that our changed architecture was still resilient to Zone-wide outages.  
    **Chaos Gorilla** 。它会占用整个可用区，以验证剩余可用区中的服务是否能够继续处理用户请求，且服务质量不会降低。虽然我们在双活架构之前运行的是 Gorilla，但持续的定期演练证明了我们变更后的架构仍然能够抵御区域范围内的中断。
*   **Split-brain**. This is a new type of outage simulation where we severed the connectivity between Regions. We were looking to demonstrate that services in each Region continued to function normally, even though some of the data replication was getting queued up. Over the course of the Active-Active project we ran Split-brain exercise many times, and found and fixed many issues.  
    **脑裂** 。这是一种新型的中断模拟，我们切断了区域之间的连接。我们希望证明，即使部分数据复制排队，每个区域的服务仍能继续正常运行。在双活项目实施过程中，我们多次进行脑裂练习，发现并修复了许多问题。
*   **Chaos Kong**. This is the biggest outage simulation we’ve done so far. In a real world scenario this would have been triggered by a severe outage that would prompt us to rapidly shift user traffic to another region, but would inevitably result in some users experiencing total loss or lower quality of service. For the outage simulation we did not want to degrade user experience. We augmented what normally would have been an emergency traffic shifting exercise with extra steps so that users that were still routed to a “failed” region would still be redirected to the healthy region. Instead of getting errors, such users would still get appropriate responses. Also, we shifted traffic a bit more gradually than we would normally do under emergency circumstances in order to allow services to scale up appropriately and for caches to warm up gradually. We didn’t switch every single traffic source, but it was a majority and enough to prove we could take the full load of Netflix in US-West-2. We kept traffic in the west region for over 24 hours, and then gradually shifted it back to stable 50/50 state. Below you can see what this exercise looks like. Most traffic shifted from US-East to US-West, while EU-West remains unaffected:  
    **Chaos Kong** 。这是我们迄今为止进行过的最大规模的宕机模拟。在真实场景中，这可能是由严重的宕机触发的，这会促使我们迅速将用户流量转移到其他区域，但这不可避免地会导致部分用户完全丢失数据或服务质量下降。在这次宕机模拟中，我们不想降低用户体验。我们在通常的紧急流量转移演习中增加了额外的步骤，以便那些仍然被路由到“故障”区域的用户能够被重定向到正常的区域。这些用户不会收到错误信息，而是能够获得适当的响应。此外，我们转移流量的方式比通常的紧急情况下更加缓慢，以便服务能够适当扩展，缓存也能逐渐预热。我们并没有切换每一个流量来源，但切换了大多数流量来源，足以证明我们能够承受 Netflix 在 US-West-2 区域的全部负载。我们将流量保持在西部区域超过 24 小时，然后逐渐将其转移回稳定的 50/50 状态。您可以在下面看到这次演习的场景。大部分流量从美国东部转移到美国西部，而欧盟西部不受影响：

Press enter or click to view image in full size  
按 Enter 键或单击即可查看完整尺寸的图像

![](https://miro.medium.com/v2/resize:fit:1400/0*HATzPmB59JIchs6W.)

## Real-life failover 实际故障转移

Even before we fully validated multi-regional isolation and redundancy via Chaos Kong, we got a chance to exercise regional failover in real life. One of our middle tier systems in one of the regions experienced a severe degradation that eventually lead to the majority of the cluster becoming unresponsive. Under normal circumstances, this would have resulted in a severe outage with many users affected for some time. This time we had additional tool at our disposal — we decided to exercise the failover and route user requests to the healthy region. Within a short time, quality of service was restored to all the users. We could then spend time triaging the root cause of the problem, deploying the fix, and subsequently routing traffic back to the now healthy region. Here is the timeline of the failover, the black line is a guide from a week before:  
甚至在我们通过 Chaos Kong 全面验证多区域隔离和冗余之前，我们就有机会在实际环境中进行区域故障转移。我们位于某个区域的一个中间层系统经历了严重的性能下降，最终导致集群的大部分系统无响应。在正常情况下，这会导致严重的宕机，许多用户会在一段时间内受到影响。这次我们掌握了额外的工具——我们决定进行故障转移，并将用户请求路由到健康的区域。在很短的时间内，所有用户的服务质量都恢复了。然后，我们可以花时间对问题的根本原因进行分类，部署修复程序，然后将流量路由回现在健康的区域。以下是故障转移的时间表，黑线是一周前的指示：

Press enter or click to view image in full size  
按 Enter 键或单击即可查看完整尺寸的图像

![](https://miro.medium.com/v2/resize:fit:1400/0*zjuCxXJlkL9SrlS1.)

## Next steps in availability  
可用性的后续步骤

All the work described above for Active-Active project is just a beginning. The project itself still has an upcoming Phase 2 — where we’ll focus on operational aspects of all of our multi-regional tooling, and automate as many of current manual steps as we can. We’ll focus on minimizing the time that we need to make a decision to execute a failover, and the time that it takes to fail over all of the traffic.  
上述所有关于 Active-Active 项目的工作都仅仅是一个开始。该项目本身即将进入第二阶段——我们将专注于所有多区域工具的运维方面，并尽可能地实现当前手动步骤的自动化。我们将致力于最大限度地缩短执行故障转移决策所需的时间，以及故障转移所有流量所需的时间。

We’re also continuing to tackle some of the more difficult problems. For example, how do you deal with some of your dependencies responding slowly, or returning errors, but only some of the time? Arguably, this is harder than dealing with Chaos type of scenarios — when something is not there, or consistently failing, it’s much easier to decide what to do. To help us learn how our systems deal with such scenarios, we have a Latency Monkey — it can inject latencies and errors (both client and server-side) at a given frequency / distribution.  
我们还在继续解决一些更棘手的问题。例如，如何处理某些依赖项响应缓慢或返回错误，但只是偶尔发生的情况？可以说，这比处理混乱的情况更难——当某些东西不存在或持续失败时，决定该怎么做要容易得多。为了帮助我们了解系统如何处理此类情况，我们开发了 Latency Monkey——它可以以给定的频率/分布注入延迟和错误（客户端和服务器端）。

## Summary 概括

With large scale and velocity there is increased chance of failure. By leveraging the principles of Isolation and Redundancy we’ve made our architecture more resilient to widespread outages. The basic building blocks that we use to compose our services and make them resilient are available from our [OSS Github site](http://netflix.github.io/) — with most of the changes for Active-Active already available, and some going through code reviews before we update the source code.  
随着规模和速度的提升，发生故障的可能性也会增加。通过利用隔离和冗余原则，我们提高了架构对大范围中断的弹性。我们用于构建服务并使其具有弹性的基本构建块可从我们的 [OSS Github 网站](http://netflix.github.io/)获取——其中大多数针对 Active-Active 的更改已经可用，部分更改在我们更新源代码之前正在进行代码审查。

This post described the technical challenges and solutions for Active-Active. The non-technical level of complexity is even more difficult , especially given that many other important projects were being worked on at the same time. The secret sauce for success were the amazing teams and incredible engineers that work at Netflix, and the ability of AWS to rapidly provision additional capacity in US-West-2. It’s all of the teams working together that made this happen in only a few months from beginning to end.  
这篇文章介绍了双活方案的技术挑战和解决方案。非技术层面的复杂性更是难上加难，尤其是在当时还有许多其他重要项目在同时进行的情况下。成功的秘诀在于 Netflix 优秀的团队和工程师，以及 AWS 在 US-West-2 区域快速配置额外容量的能力。正是所有团队的通力合作，才使得这一切从始至终在短短几个月内就得以实现。

If similar challenges and chance of working with amazing teams excite you, check out our [jobs site](http://jobs.netflix.com/). We’re looking for great engineers to join our teams and help us make Netflix services even more resilient and available!  
如果您对类似的挑战以及与优秀团队合作的机会感到兴奋，请访问我们的[招聘网站](http://jobs.netflix.com/) 。我们正在寻找优秀的工程师加入我们的团队，帮助我们提升 Netflix 服务的弹性和可用性！

## See Also: 参见：