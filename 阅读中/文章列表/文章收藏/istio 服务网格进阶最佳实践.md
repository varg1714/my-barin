---
source: https://mp.weixin.qq.com/s?__biz=MzI1NjEwMTM4OA==&mid=2651234684&idx=1&sn=947f2958af56323e317ee0dc360cb207&chksm=f1d9dc1fc6ae55098384440792be70ffc3b1dca1befba180df27ce1e438f309ba6b73d630e14&mpshare=1&scene=1&srcid=1031qigbakG1HsfyPvjNeeZr&sharer_shareinfo=b7776596c78782c47ea21f9dc77173e7&sharer_shareinfo_first=b7776596c78782c47ea21f9dc77173e7#rd
create: 2023-11-13 17:10
---

❝

作者：delphisfang，目前就职于腾讯音乐 / 社区产品部 / 公共框架团队，从事微服务、服务网格等云原生架构的开发与落地。

本文约 6000 字，预计阅读完毕需要 15 分钟。

1. 前言

本文从 Service Mesh 开发者的角度，阐述 istio 进阶的个人最佳实践，希望帮助正在入门 istio 的同学节省一点时间，少走一点弯路。

# 1.1 本文出发点

学会 istio[1] 或 envoy[2] 需要多久？是一周、两周，还是一个月？笔者无法给出准确的答案，那取决于每个人对 “学会” 这个标准的定义。网上有不少介绍 istio 的入门文章，如果只是想快速地完成一篇「Service Mesh 综述」的话，看看过往的文章应该已经足够。不过按照笔者的经验，看完之后大概会达到一种浅尝辄止的状态：能说上那么两句，但是仅此而已，不能立即投入线上开发，也无法调试定位具体的问题。如果想要更深入一步，则需要去啃那些「大部头书籍」，花费的时间会成倍增长。

因此，笔者撰写本文的出发点是，尝试找到入门文章与大部头书籍之间的一个平衡，以一篇文章的篇幅，帮助初学者达到「玩转 istio」的阶段。也可以说是完成对 istio 的祛魅，为 istio 的初学者提供一把打开 Service Mesh 进阶之路的钥匙。希望它既不是一篇「入门介绍」，也不是一本巨细靡遗、主次不分的「参考手册」，而是一篇只讲重点、只讲笔者自身经验的最佳实践。

# 1.2 何为「最佳实践」？

“最佳实践” 这个名词起源于管理领域，但是已经有一些被滥用了。我们可以在网上搜到各式各样的「某某最佳实践」。然而事实上，“最佳实践” 无疑是一个伪命题。这个世界上永远不可能有什么工程实践是最佳的，除非你是 Jeff Dean（冷笑话）。实事求是地说，本文介绍的只是一种还算不错的、可以快速上手的实践。

# 1.3 何为「进阶」？

我们经常看到有人在简历里写 “熟练掌握某某技术”、“精通某某技术”。那么，到底什么程度才能算精通？精通的那根标准线在哪里？「进阶开发」的英文是 Advanced Programming。Advanced 既有高级的意思，也有先进的含义。高级是指，掌握了这个事物的一些不为初学者所知的、精微的技术细节，能让你完成别人完成不了的工作，或者比别人更快地完成。先进则是指，这个事物本身带来了一些更新颖的、更现代化的想法或理念，比如：Google 在 2006 年发布的三驾马车论文、Docker 带来的容器思想、Kubernetes 引入的声明式 API。

# 2. 最佳实践的一点方法论

在厘清本文的定位以及标题关键词的含义之后，就可以开始正文部分了。

在面对一个庞大的系统时，我们该如何去掌握它？像 Kubernetes[3] 这种活跃度超级高的开源项目，其代码行数通常是以百万计的，没有人能够清楚地记得所有的细节，也没有这个必要。本文并不会 case by case 地去讲解 istio/envoy 的每一个功能或特性，而是尝试介绍一种通用的工程实践方法，让开发者在需要研究某一项特性时，知道如何快速切入、纵向深挖与横向发散，并快速得出结论、验证原型，最终应用于生产环境。

# 2.1 三重证据学习法

历史学领域有一个著名的研究方法：三重证据法，即：「传世文献 · 出土文字 · 出土文物」三方相互印证，构成严密的证据链，从而推断出某个历史的真相。在这里我们提出：可以使用类似的思路学习和掌握复杂的技术系统，即：「官方文档 · 源码 · 原型实验」 三位一体，构成一个完整的学习闭环。

# 2.2 去除不必要的流程或组件

该说法出自 Elon Musk 的视频采访 Five Step Improvement Process[4] 中的第 2 步：Remove unnecessary process，它更广为人知的名字是：奥卡姆剃刀原则。早在 2012 年，微信之父张小龙在《微信背后的产品观》的讲座中，就不断提到类似的理念：保持简单。我们也认为，在学习乃至设计一个东西的过程中，需要不断地追问：

*   哪些东西是必不可少的，哪些只是锦上添花？
    
*   哪些东西是新颖的，哪些是业界已有的套路？

这个步骤可以大幅降低系统的复杂度和理解成本，为我们节省出大量的时间。拨开表面的那些枝叶，我们才能看清一个东西的骨架，即 “原型”。

# 2.3 关键词定位

Talk is cheap, Dive into the code. 与启发式的 AI 神经网络算法相比，大多数工程类的项目都有一个优点：它们的一切奥秘都在源码之中，其行为以及背后驱动的代码都是确定无疑、可以被解释的。这意味着：你一定能在源码中定位到它、读懂它，乃至修改它。如果某个项目的源码在可读性方面做得还还不赖的话，你就可以用**关键词定位法**快速地锁定代码段，大大节省你的时间。

以 istio 和 envoy 为例，两者均已发展成为数十万行代码的项目。但是得益于高度模块化、可扩展的设计，两者的代码在 可读性 与 内聚度 两方面都达到了很高的水平，这使得关键词检索法能够发挥最大的威力。你甚至不需要一个解析代码的 IDE，仅靠 grep 大法也可以锁定某个特性背后的核心代码。

综合上述几点方法论，可以总结出一套**进阶最佳实践**：

**1. 预设问题：**带着问题出发，逐个寻找答案，在有限时间内收敛闭环，避免无限发散。

**2. 文档调研：**了解目标对象是什么、不是什么、边界在哪里、竞品有哪些，它在更大的 Landscape 中的位置。

**3. 原型搭建：**搭建目标系统的架构原型，设计实验，验证其行为是否符合自己预期。

**4. 深入源码：**修改它、重编它、运行它、调试它，让它按照你所指挥的那样去工作。

为了避免本文演变成虚无的、形而上的坐而论道，方法论部分点到为止。

# 3. istio 进阶最佳实践案例

下面以「istio 就近地域路由」这个特性为例，展示这套最佳实践的具体操作过程。「就近地域路由」是一个粒度适中的特性，既不大也不小，很适合用于展示 istio 进阶 的最佳实践。

# 3.1 预设问题

下面的问题是逐层递进的。有些只需要做文档调研就能回答；有些则需要深入源码，甚至运行实验才能得出确切的结论。尽管不同人提出的问题可能会非常不同，但是笔者认为问题列表应该存在一个最低标准，即：将这些问题紧密串联起来，应该要能回答整个架构的每一环是如何工作的，即：每一个环节的输入是什么、输出是什么。

Q1：何为 Locality？

Q2：希望就近地域路由如何表现？

Q3：如何驱动 envoy 的就近地域路由？需要如何配置？  

Q4：如何驱动 istio 的就近地域路由？需要如何配置？  

Q5：istio 如何获知某个 envoy 的 locality？  

Q6：就近地域路由的核心算法是怎么样的？在 istio 层还是 envoy 层实现？

# 3.2 文档调研

3.2.1 何为 Locality？  

在云计算领域，Locality 通常指二元组 <Region, Availability Zone>，即：< 地域, 可用区 >。此概念起源于云计算大厂 Amazon[5]。【地域】的划分，是为了尽量靠近当地的用户，使用户获得较低的接入时延；不同地域之间只能通过公网互通。【可用区】的划分，是为了冗余和容灾；可用区之间的网络是互通的，距离在 100KM 以内，且采用高速光纤相连。这意味着，用户连到 A 区 或 连到 B 区，在时延上的差别几乎可以忽略。通常一个可用区就是一个数据中心（即一个机房），规模较大的可用区也可以由多个距离靠近的数据中心组成，形成庞大的服务器集群。

Kubernetes 沿袭了云厂商的提法，支持 Region、AZ 两个级别 [6]。而在 istio 中，Locality 则是一个三元组：<Region, Zone, Sub-zone>，即：< 地域，可用区，子区 >。按照 Istio 官方 [6] 的说法，Sub-zone 是为了做更精细的划分，比如：划分到 “机架” 的粒度。

## 1. istio 就近地域路由策略：Locality failover

很明显，对于【就近路由】这个特性，最直接的诉求就是：istio 应该总是尝试将流量转发给距离主调方最近的目标服务实例。这是一个不断做降级 (Failover) 的过程：

1.  如果有与主调方处在同一个 region/zone/sub-zone 的被调实例，则 istio 优先路由给这些实例；否则转到第 2 步。
    
2.  如果有与主调方处在同一个 region/zone 的被调实例，则 istio 优先路由给这些实例；否则转到第 3 步。
    
3.  如果有与主调方处在同一个 region 的被调实例，则 istio 优先路由给这些实例；否则转到第 4 步。
    
4.  将流量路由给其他实例。

在 istio 中，该策略被称为 Locality failover，具体的例子可以参考文献 [8]。

![](https://mmbiz.qpic.cn/mmbiz_png/FYK8QeJX2bOPjGHp53Dh7t2ZJicIMvqciaicYnf0FWjHrpiag4jxnfIrQAUrHcVhiaNdD3agNDJpp6nnsBjYemGr7Uw/640?wx_fmt=png)

# 3.3 istio 原型搭建

## 1. 理解 istio 架构原型

本节将向读者展示 istio 架构 [9] 真正必不可少的部件（即原型）是什么，以及在自助学习进阶 istio 的阶段，我们可以暂时不去关注哪些内容，只聚焦在最核心的部件上。本质上，只有 envoy 与 pilot 两个组件是必不可少的。当然，如果您有充足的时间，建议您还是部署一套 Kubernetes 集群，并在其上部署一套完整的 istio 系统。

![](https://mmbiz.qpic.cn/mmbiz_png/FYK8QeJX2bOPjGHp53Dh7t2ZJicIMvqciaFNvhCaJMYSCQhAbn5u1iasa7JANh6uIlvLWodfLaxKyKYjOEhVWXtZQ/640?wx_fmt=png)

一个完整的 istio service mesh 架构 [10] 如上图所示，包含几个要点：

*   整套 istio 必须部署运行于 Kubernetes 之上。
    
*   主调方与被调方均需部署 istio proxy，分别接管 outbound 流量 和 inbound 流量。
    
*   istio 的控制面 istiod 包含 Pilot、Citadel、Galley 三个主要的子模块，分别用于下发 xDS 配置给 envoy、管理证书、为其他控制面组件生成配置。

**简化理解 istio 架构：**

*   如今 minikube、kind 等工具已经使得部署一个 Kubernetes 集群变得非常简单，然而要想在 K8s 上调试一个 istio 集群，您仍然需要具备操纵 K8s Deployment、Pod、ConfigMap 等资源的前置能力。当您需要验证某个 istio 特性时，K8s 这一层的存在，将使你所花费的时间成倍地增长。
    
*   istio 默认要求在主、被调双方都部署 proxy (即 envoy)，以接管一切出、入流量。然而如果仔细分析，除了全局限流 (global ratelimit)、服务间调用鉴权 (service authentication) 等几个特性之外，大多数流量治理能力都可以只由 主调方 proxy 承担。即是说，架构可以简化为只部署主调方的 proxy。
    
*   在 istio 控制面的三个组件中，pilot 是必不可少的，我们需要利用它给数据面的 envoy 下发 xDS 配置，驱动 envoy 按照我们指挥的那样去工作。
    
*   综上三点，我们可以得出一套 简化的 istio service mesh 架构，便于学习 - 调试 - 验证阶段的快速切入，如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/FYK8QeJX2bOPjGHp53Dh7t2ZJicIMvqciayFoFkdNtBxNeHEIia45OFxalQibxYshIGllicCsWnub1fM7qwz6F2mWXA/640?wx_fmt=png)

**3.3.2 理解 envoy 层的地域配置：locality endpoint**

由第 2 节的调研结果可知：envoy 并不会直接去计算每个目标实例与自己的距离远近，而是由 istio 告知 envoy 每个实例的优先级 (priority)；envoy 只需要将流量优先转发给 priority 值最小的那些实例。

在 envoy 层面，使用 locality endpoint 表示一批拥有相同 locality 的实例；而 locality 和 priority 都是 locality endpoint 的属性，分别用于指明这批实例的 地理位置 和 优先级。

Envoy locality endpoint 配置示例如下：

```
- name: helloworld  connect_timeout: 0.1s  type: static  lb_policy: ROUND_ROBIN  load_assignment:    cluster_name: helloworld    endpoints:    - priority: 0 // 第1个实例，位于华南-深圳-01，优先级为0 (最高)      locality:        region: south-china        zone: shenzhen        sub_zone: 01      lb_endpoints:      - endpoint:          address:            socket_address:              address: 1.1.1.1              port_value: 12345    - priority: 1 // 第2个实例，位于华南-深圳-02，优先级为1      locality:        region: south-china        zone: shenzhen        sub_zone: 02      lb_endpoints:      - endpoint:          address:            socket_address:              address: 2.2.2.2              port_value: 12345    - priority: 2 // 第3个实例，位于华南-广州-01，优先级为2      locality:        region: south-china        zone: guangzhou        sub_zone: 01      lb_endpoints:      - endpoint:          address:            socket_address:              address: 3.3.3.3              port_value: 12345    - priority: 3 // 第4个实例，位于华东-上海-01，优先级为3 (最低)      locality:        region: east-china        zone: shanghai        sub_zone: 01      lb_endpoints:      - endpoint:          address:            socket_address:              address: 4.4.4.4              port_value: 12345
```

**3.3.3 理解 istio 层的地域配置：DestinationRule**  

istio 层面的【地域路由】称作 Locality failover，需要使用 DestinationRule [19] 进行配置才能启用。如下面的 yaml 配置所示，该 DestinationRule 描述了 helloworld 服务的 3 个核心策略，分别是：connectionPool (连接参数)、loadBalancer (负载均衡策略)、outlierDetection(熔断踢除策略)。其中与【就近地域路由】相关的配置就是第 2 项：localityLbSetting。需要注意的是：必须同时有 outlierDetection 配置项，locality failover 才能生效。

DestinationRule locality 配置示例如下：

```
apiVersion: networking.istio.io/v1beta1kind: DestinationRulemetadata:  name: helloworld  namespace: demo-namespacespec:  host: helloworld.svc.cluster.local  trafficPolicy:    connectionPool:      http:        maxRequestsPerConnection: 100    loadBalancer:      simple: ROUND_ROUBIN      localityLbSetting: // 启用 locality failover 策略        enabled: true        failover: []    outlierDetection: // 必须有该配置项，locality策略才能生效      consecutive5xxErrors: 10      interval: 5s      baseEjectionTime: 1m
```

# 3.4 深入 istio 源码

# 3.4.1 读懂 istio 地域路由的核心算法  

**源码路径：https://github.com/istio/istio/blob/master/pilot/pkg/networking/core/v1alpha3/loadbalancer/loadbalancer.go#L158**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/6GYDCe0a26iblSvUOjddL4u1XY86RIJnRibRPrfgcwDZqhKJB1TIAlexgjhA3ibymRbvib67lsBs2kiaU88ic77SyzlQ/640?wx_fmt=png)

在 istio 源码中，尝试检索关键词 Locality 和 Failover，可以看到：确实有一个名为 applyLocalityFailover 的函数（如上述源码所示）。阅读该函数的注释，不难发现：该函数正是 Locality Failover 的核心算法实现。该函数通过计算 endpoint locality 与 envoy locality 的匹配程度，得出 endpoint 的 priority。当然，这个 priority 值仅仅只是针对该 envoy 来说的。换成另一个 envoy，计算出来的 endpoint priority 就可能完全不同。

核心算法如下：

1.  若 region/zone/sub-zone 三层均匹配，则将实例的 priority 设置为 0。
    
2.  若仅 region/zone 两层匹配，则将实例的 priority 设置为 1。
    
3.  若仅 region 匹配，则将实例的 priority 设置为 2。
    
4.  若均不匹配，则将实例的 priority 设置为 3。

## 1. istio 如何获知每个服务实例的 Locality？

我们知道，每一个 istio Service Entry [17] 都用于唯一表示一个服务，包括隶属于该服务的所有实例 (endpoint)。而 Locality 又是 endpoint 的一个属性。因此，实例的 Locality 一定存在于 ServiceEntry 对象中，如下面的 yaml 配置所示。istio 通过解析 ServiceEntry 对象，即可获知每个实例的 Locality。

Service Entry locality 配置示例如下：

```
apiVersion: networking.istio.io/v1beta1kind: ServiceEntrymetadata:  name: helloworld  namespace: demo-namespacespec:  hosts:   - helloworld.svc.cluster.local  ports:  - number: 12345    name: helloworld    protocol: HTTP  resolution: static  endpoints:  - address: 1.1.1.1    locality: south-china/shenzhen/01  - address: 2.2.2.2    locality: south-china/shenzhen/02  - address: 3.3.3.3    locality: south-china/guangzhou/01  - address: 4.4.4.4    locality: east-china/shanghai/01
```

**3.4.3 istio 如何获知每个 envoy 的 Locality？**

**源码路径：****https://github.com/istio/istio/blob/master/pilot/pkg/model/context.go**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/6GYDCe0a26iblSvUOjddL4u1XY86RIJnRibKiccWJiawzMgS1VkgKwhu3RibkV0Oic21Me7K9MdMC5pXibKZZb5p3asFQ/640?wx_fmt=png)

如上面的 istio 源码所示：在 istio 中，每一个连接到它的 envoy 实例都由一个 Proxy 对象唯一表示，其中就包含 Locality 属性。而 Locality 的取值就存放在该 envoy 实例的 bootstrap 启动配置文件中。

# 4. istio 进阶知识点

1. envoy 并不依赖 容器 与 Kubernetes，可以部署在虚拟机、物理机乃至裸金属环境上。
2. envoy 也不依赖 istio，而是反过来，istio 选择了 envoy 作为数据面组件。
3. 在 envoy 层面，服务、实例、路由配置、监听器 等所有资源都被定义为 xDS API [14]，可以动态获取。当然，这些资源也可以静态配置在 envoy bootstrap 配置文件 [15] 中。在调试验证阶段，采用静态配置的方式显然会方便很多。
4. 在 istio 层面，沿袭了 K8s 的设计，使用 CRD (Custom Resource Definition [16]) 表示各种资源。其中，最核心的 CRD 有 3 个：

*   istio Service Entry [17] 用于定义一个服务及其下的实例列表。
    
*   istio Virtual Service [18] 用于定义一个服务的流量路由策略 (traffic routing)。  
    
*   istio Destination Rule [19] 用于定义一个服务的关键参数，包括：连接参数、负载均衡策略、故障踢除策略。

5. 虽然 istio 控制面包含有若干个子模块：pilot、citadel、galley，但在大多数应用场景下，只有 pilot 是必不可少的。一套最简单的 Istio Service Mesh 可以只部署 pilot 和 envoy，让 envoy 连接到 pilot 动态拉取资源。更极致地，你可以只部署 envoy，将实验所需的资源都写在 envoy bootstrap config 中。
6. 虽然 envoy 的代码量已经超过了 70w 行，istio 的代码量超过了 30w 行，但是不必担心，也不要企图搞清楚每一处细节。你甚至不需要一款全功能的 IDE，有需要时可以直接搜索关键词。
7. 虽然本文的标题是「istio 进阶最佳实践」，不过本实践路径并不局限于学习 istio，也适用于其他开源技术 (尤其是云原生技术) 的学习与进阶。

# 5. istio 进阶技术文档 

**[1] The istio service mesh.**

https://istio.io/latest/about/service-mesh/

**[2] Envoy proxy.**  

https://www.envoyproxy.io/

**[3] Kubernetes Github.**

https://github.com/kubernetes/kubernetes

**[4] Elon Musk Five Step Improvement Process.**

https://www.youtube.com/watch?v=Jgw-_hlFQk4  

**[5] AWS Regions & Zones.** 

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html

**[6] K8s Topology Aware Routing.**

https://kubernetes.io/docs/concepts/services-networking/topology-aware-routing/

**[7] Istio Locality Load Balancing.**

https://istio.io/latest/docs/tasks/traffic-management/locality-load-balancing/

**[8] Istio Locality failover.**

https://istio.io/latest/docs/tasks/traffic-management/locality-load-balancing/failover/

**[9] Istio architecture.**

https://istio.io/latest/zh/docs/ops/deployment/architecture/

**[10] Istio Locality weighted distribution.**

https://istio.io/latest/docs/tasks/traffic-management/locality-load-balancing/distribute/

**[11] Envoy Locality weighted load balancing.**

https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/upstream/load_balancing/locality_weight

**[12] Envoy Priority Levels.**

https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/upstream/load_balancing/priority#arch-overview-load-balancing-priority-levels

**[13] Envoy Advanced.**

https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/advanced/advanced

**[14] Envoy xDS API.**

https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/operations/dynamic_configuration

**[15] Envoy bootstrap config example.**

https://www.envoyproxy.io/docs/envoy/latest/configuration/overview/examples

**[16] Kubernetes CRD.**

https://kubernetes.io/zh-cn/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/  

**[17] Istio Service Entry.**

https://istio.io/latest/docs/reference/config/networking/service-entry/

**[18] Istio Virtual Service.**

https://istio.io/latest/docs/reference/config/networking/virtual-service/

**[19] Istio Destination Rule.**

https://istio.io/latest/docs/reference/config/networking/destination-rule/