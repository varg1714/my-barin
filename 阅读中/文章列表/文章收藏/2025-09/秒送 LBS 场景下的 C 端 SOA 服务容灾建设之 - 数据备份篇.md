---
source: https://mp.weixin.qq.com/s/D_qnPLOgglCLJGvS0OsLew
create: 2024-08-16 16:29
read: false
---

在面向服务的架构（SOA）系统中，容灾能力是保障系统稳定性的重要组成部分。通过引入多数据中心部署、自动化故障转移、数据备份等技术手段，可以有效提升系统在面对突发灾难事件时的恢复能力。例如，采用主从复制和异地多活架构，可以确保在某个数据中心发生故障时，其他数据中心能够迅速接管业务，避免服务中断。此外，定期进行灾难恢复演练和系统压力测试，也是提高容灾能力的关键措施。通过这些手段，SOA 系统能够在各种极端情况下保持高可用性和稳定性，从而为用户提供持续可靠的服务。今天重点聊一下 LBS 场景下的数据备份方案。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXkiaPytJYOXuXj0EJUGRIE6wRMJ7jo7OwxslJTSvDybB5ubFJ1Vsqnxp74KbqedibgAFzLDsb3qKZg/640?wx_fmt=png&from=appmsg&wxfrom=13)

 **一、问题背景**

随着秒送业务的上线，基于 LBS 的用户交易的流量逐渐提升，不同于 JD 主站的 B2C 场景，O2O 场景下的资源场景划分的非常精细，根据门店履约围栏划分 3 公里、2 公里、1 公里等不同的范围，资源与资源之间互相交错、覆盖，构建成一个具有 LBS 特性的资源网。作为入口级流量的后台 SOA 服务，我们需要根据业务特性区分哪些数据强实时 / 弱实时来制定不同的数据备份策略、还需要弄清楚底层数据生产的逻辑链路，才能保障数据备份的真实性、还需要考虑用户使用的这份备份数据体验等等因素。因此，LBS 场景下的容灾数据备份的难度瞬间升级。

## 1. 二、痛点 / 挑战

基于上述背景描述，我们在制定方案的过程中遇到了很多挑战，主要分为三大类

### 1.1. 如何缓存 poi 经纬度数据

如果要按照全国 poi 经纬度去缓存数据，为了降低数据差异，我们把不同 poi 点的距离精确到（米级），那么全国 poi 的数量是多少呢？根据主站网格地址统计，精确度在 75 米的 poi 点在 6.8 亿 +，如果按照 1 米算，估计千亿 + 级别的 poi。如此量级的 poi 数据，我们如果每一个 poi 都做数据备份的话（理想状态）；拿秒送首页、频道页举例，首页一次交互数据大小 600kb 左右，频道页核心 5 个，每个 400kb 左右，那么每个 poi 需要 2600KB 的存储，按照 75 米的 poi 距离，需要 6.8 亿 * 2600kb=1646TB，C 端数据最终存储在 redis 中，按照 5G=1 核 = 15 元 / 月，那么需要花费 500 万 + 人民币 / 月，老板听完方案汇报脸都黑了。

### 1.2. 如何降低容灾数据存储成本

每个月 500 多万的存储开销成本显然是不现实的，于是我们迎来了第二个挑战，那就是怎么才能降低这么大的存储成本，于是我们内部多次讨论，绞尽脑汁，得出结论：如果想让用户得到最好的体验那么似乎和成本之间是互斥的，我们怎么才能寻找一个中间点，来平衡户体验和成本。

### 1.3. 如何保障缓存数据资源的有效性

在进行数据备份时，数据一致性问题是不可避免的关键点。考虑到如此庞大的数据量，若采用实时请求备份的方式，SOA 层面的请求负载将可能对底层 RPC 造成极大的查询压力。相对而言，异步构建方法则要求我们复制主站搜索、推荐和 GIS 定位等异构逻辑（因为秒送首页、门店详情等资源来源于主站搜索、推荐和 GIS），这不仅意味着需要处理上千个中台的消息队列（MQ），还要确保逻辑与现有系统保持一致，这显然是个巨大挑战。

因此，我们的选择余地有限。有观点认为，数据备份并不需要完全真实，稍有偏差也是可接受的，甚至可以使用静态图片代替。我们承认，实现数据容灾备份与线上数据保持 100% 一致是不现实的。但我们的目标是尽可能保证 90% 的用户体验与线上相似，同时确保交易流程的连续性，并在系统异常时为用户提供无感知的体验。（此处价值观强调升华，用户为先）

## 2. 三、业界方案调研

flag 立完了，但是确实没啥好思路，B2C 场景的不适用，只能是去跟隔壁同行取取经，通过多方渠道找到了一些靠谱的某团的技术大佬，并且做了 1 对 1 的沟通。大致梳理了下他们的架构和容灾方案。

某团的超市便利架构：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXkiaPytJYOXuXj0EJUGRIE6poO749IYmBwywMPrfpszBLt6ntDicRnYKD7bCtZ4MN8htmsDcv1sJrA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

通过沟通得知，某团在 SOA 层，没有做任何的数据备份容灾，完全依赖底层的数据提供，如果底层数据返回空，那么页面也会空白... 至于底层数据有没有做容灾数据备份，就不得而知，我相信是有的。于是到这里，感觉前路又被堵死了。

## 3. 四、方案构思

调研无果的情况下，我们回过头来重新整理下现有的逻辑链路，从中寻找一些突破点，为了整体容灾数据备份方案做预设计。

### 3.1. 首先根据业务筛选下，哪些入口是阻塞交易链路，不可降级的

从中识别出：首页、频道页、门详页是阻塞交易的环节，需要进行数据备份。

### 3.2. 底层数据依赖的链路逻辑分别是什么

#### 3.2.1. （1）首页、频道页

首页和频道页充当着网站的主入口，它们利用 CMS 系统和定向投放系统实现了基于地理位置服务 (LBS) 的楼层内容过滤。网站内容通过商品组合、广告组合以及推荐机制构建，核心资源主要依赖于推荐系统的数据池。我们分析后注意到，不同用户在同一地点或同一用户在不同地点可见的内容各不相同，这主要是由推荐系统的数据池结合特定策略实现的。推荐数据所依赖的因素来源于多种策略的分配，而这些策略因素又会反过来影响推荐结果。这导致我们无法仅通过备份当前数据来保持数据的一致性，因为策略的即时变化可能会使备份数据的结果发生变化。若要备份数据与线上数据完全一致，我们需要复制一整套推荐系统的数据流转过程，这是目前的一个难点。为了应对这种情况，我们采用了一个通用账号执行普通策略来进行数据备份，以尽可能排除变量因素，虽然这牺牲了一定的个性化推荐能力，但确保了数据正常展现。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXkiaPytJYOXuXj0EJUGRIE62IYthhWMHRCgkjkG6UQjrQEmajfbOnMhAYiaQax0DNC1E2PYnyZtJYg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 （2）门详页

门详页是秒送流量的关键交易通道，其核心门店分类和商品信息均通过搜索引擎和商家分类中台获得。尽管集团的主搜索引擎已经分离出专门的垂直搜索服务供秒送场景使用，但数据源仍然是共用的，区别仅在于网关层面的分离。与首页和频道页不同，门详页专注于门店层面的分类和商品数据，不需涉及广告投放和目标人群等因素。只要门店能在首页展示，用户点击进入后看到的分类和商品就会保持一致。因此，我们的主要关注点是确保当前搜索引擎的数据源是准确的。我们对线上 100 家不同类型的门店（包括连锁品牌和非连锁品牌）进行了一周的观察，发现门店分类变动较少，而分类下的商品状态和库存变化较频繁。基于此，我们可以对门店分类实施全量缓存（考虑到较低的时效性需求），而对于分类下的商品则根据情况实施缓存（需要较高的时效性）。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXkiaPytJYOXuXj0EJUGRIE6HrBzNdwSmMVgRSKciaJC9Ur1TIsLwa8ia1DclHK1k7ECmIelNaQubMOw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 3、削减数据备份量级思路

因为底层依赖的数据源不一致，所以首页、门详页分别制定不同的数据备份设计方案，首页的由于和 poi 强相关，所以更复杂一些。门详因为不涉及 poi 以及策略，所以相对首页来说容易一些。

#### 3.2.2. （1）首页、频道页（引入网格思想）

在整理推荐系统的数据链路时，我们注意到推荐结果中的门店商品资源高度依赖 GIS 系统。深入分析 GIS 逻辑后，了解到 GIS 维护着不同覆盖范围的网格围栏，如 3 公里、5 公里和 20 公里网格，并且每个网格下的门店商品资源是相同的。这启发了我们，如果我们也能构建一个类似 GIS 的网格系统并缓存门店商品，岂不是能显著提升数据的一致性？然而，现实告诉我们，推荐系统调用 GIS 接口是基于策略模型的选择，并非易事。若要完整复制推荐系统逻辑几乎不可能。

然而，网格的概念是值得借鉴的，因为它可以大幅减少我们的 POI 缓存数据规模。如果我们反其道而行之，直接缓存当前 POI 下的推荐结果，就无需处理推荐逻辑的复杂性。接下来的挑战是构建一套尽可能与 GIS 网格重叠的网格系统。

重点来了！

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXkiaPytJYOXuXj0EJUGRIE6ic3iaicd1qB30557jh8bl8JJ4MRIQ9UDYWLKrNY1cZ7CqWBUD5rsSHlxQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 如何选择构建网格类型？

开源的网格 grid 构建空间索引又很多种，墨卡托、geohash、H3、S2 等等，那么我们如何去选择呢？GIS 提供的网格接口使用的空间索引是 geohash 四边形网格，但是为了以防万一，我们通过主站网格中台封装的 JMF 组件，异构了包含了 geohash 在内的多种空间索引，目的是全方位覆盖，为了后续的 diff 提供多种选择，得到最优结果。这里我就挑一种 H3 作为 demo 去讲述如何构建的网格。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXkiaPytJYOXuXj0EJUGRIE6SMiah1vOBoQibzOQbiccaOia6rNkhgZIZj5cqWU57Kw4lKXYulAl3OkWzQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 如何去构建网格？

另外一个问题，构建网格的分辨率应该如何选择，也就是我们常说的网格精度。我们去门店中台查了秒送的门店范围，发现 90% 以上的门店都是基于 3 公里圈定的不规则履约范围，而这个也是作为 GIS 判断网格是否覆盖该门店的核心因子。所以在构建网格的时候，每个蜂窝格子的面积限定 3 公里显然不能得到最优结果，于是我们通过 H3 的空间索引构建了 Hexagon 六边形网格，尽可能的模拟线上真实不规则的履约圈选范围。同时 Hexagon 的边长精度划定为 7 也就是 1.4km，得到最小六边形面积 3.12m²，最大六边形面积 6.22m²。同时针对精度 8 的也异构出一套数据作为 diff 参照。经过计算后得出（刨去偏远地区），精度 7 的热点网格数量在几十万 +，精度 8 个网格数量在百万 +，和原本的百亿级别相比，降低了 99%~~，最后经过成本计算：①精度 7 的 =2973 元 / 月 ；②精度 8 的 =14133 元 / 月 （相比于最初的 500 万 +/ 月，打折到骨折！）

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXkiaPytJYOXuXj0EJUGRIE6Z1cnCERep6HkTPquk13XUIKtiaWo7je8dPTYybyBFHy3lEwkxT662Bg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 Hexagon 官方文档：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXkiaPytJYOXuXj0EJUGRIE6ZgGy1CA0Dz1icUSp69jDib6DC0CaiaUHkCjKiaCCfqoxDgp6BXgmtKWZmw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 如何识别网格热点 poi？

（选择网格中的哪个 poi 作为当前网格的缓存代表？）

构建好网格后，问题又来了，因为网格是一组 poi 的集合，我们只有拿到一个网格内部数据备份最好的 poi 经纬度，才能去模拟线上用户请求进行缓存，作为这个网格的容灾数据备份，从而达到最优的数据备份效果。经过和网格中台沟通后得出，网格集成了格子内的用户请求分布情况，于是我们可以根据用户分布最集中的经纬度 poi 点进行数据备份，达到最佳效果。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXkiaPytJYOXuXj0EJUGRIE65818PW0ticvE7D6bD3OVjHI8kmYuge8K533xUcAsfC7OPoATDbyzhXQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 数据备份时间多久？

最后就是数据备份的时间，也就是数据一致性的问题。上面说过了，推荐包含了很多策略，同一个用户 白天和晚上看到的首页结果都不一样，这个就是涉及到体验和成本的问题，如果为了保持好的体验，我们可以缓存 2 套数据，白天一套、晚上一套。但是考虑到晚上用户占比过少，一期还是优先缓存白天的数据。

#### 3.2.3. （2）门详页

门详页的容灾备份上面提到了，我们只需要考虑如何缓存门店分类数据和商品数据，分类数据一个门店最多几百个，但是商品可能往往上万，掏出小本本，我们再算一笔账：我们现在百万 + 门店，每个门店的商品乘以每条商品数据平均按 1kb 算，那么成本预估 15W 元 +/ 月，领导听完汇报脸又黑了。

 策略选择

我们经过讨论后，决定先对每个末级分类的前两页商品进行缓存，并且剔除掉闭店的门店。经过策略估算后，缓存量级可以压缩到 900 元 / 月

（3）引入 GIZP 压缩

经过实践后发现，gizp 压缩对于纯文本的压缩预估能节省 60% 的空间，但是需要注意的是 QUERY 数据解压缩的性能耗时。我们在进行设计的时候，把所有的数据全部转化成 String 字符串，并且通过 gizp 压缩。最终得到以下备份数据量级：

<table width="677"><colgroup><col width="auto"><col width="auto"><col width="auto"><col width="auto"></colgroup><tbody><tr data-slate-node="element" data-slate-inline="false"><th colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false"><span data-slate-node="text"><span data-slate-leaf="true"><span data-slate-zero-width="n" data-slate-length="0">﻿<br></span></span></span></th><th colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false"><span data-slate-node="text"><span data-slate-leaf="true"><span data-slate-string="true">首页</span></span></span></th><th colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false"><span data-slate-node="text"><span data-slate-leaf="true"><span data-slate-string="true">频道页</span></span></span></th><th colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false"><span data-slate-node="text"><span data-slate-leaf="true"><span data-slate-string="true">门详页</span></span></span></th></tr><tr data-slate-node="element" data-slate-inline="false"><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false"><span data-slate-node="text"><span data-slate-leaf="true"><span data-slate-string="true">压缩前</span></span></span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false"><span data-slate-node="text"><span data-slate-leaf="true"><span data-slate-string="true">1087GB</span></span></span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false"><span data-slate-node="text"><span data-slate-leaf="true"><span data-slate-string="true">3624GB</span></span></span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false"><span data-slate-node="text"><span data-slate-leaf="true"><span data-slate-string="true">300GB</span></span></span></td></tr><tr data-slate-node="element" data-slate-inline="false"><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false"><span data-slate-node="text"><span data-slate-leaf="true"><span data-slate-string="true">压缩后</span></span></span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false"><span data-slate-node="text"><span data-slate-leaf="true"><span data-slate-string="true">652GB</span></span></span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false"><span data-slate-node="text"><span data-slate-leaf="true"><span data-slate-string="true">2174GB</span></span></span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false"><span data-slate-node="text"><span data-slate-leaf="true"><span data-slate-string="true">180GB</span></span></span></td></tr><tr data-slate-node="element" data-slate-inline="false"><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false"><span data-slate-node="text"><span data-slate-leaf="true"><span data-slate-string="true">最终成本</span></span></span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false"><span data-slate-node="text"><span data-slate-leaf="true"><strong><span data-slate-string="true">1956 元 / 月</span></strong></span></span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false"><span data-slate-node="text"><span data-slate-leaf="true"><strong><span data-slate-string="true">6522 元 / 月</span></strong></span></span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false"><span data-slate-node="text"><span data-slate-leaf="true"><strong><span data-slate-string="true">540 元 / 月</span></strong></span></span></td></tr></tbody></table>

至此，整个容灾数据备份量级压缩方案可行，相比于最初的 500W+/ 月，降低到了 9018 元 / 月。（还要啥自行车？领导当即拍板，赶紧做完上线）

#### 3.2.4. DIFF 思路 - 如何去和线上做 DIFF？

##### 3.2.4.1. （1）首页、频道页

到了验证环节了，因为我们无法完全和 GIS 的网格重合，那么我们需要一套规则来做 DIFF 验证网格的热点 poi 数据备份是否和线上 poi 数据返回一致（不可能 100% 一致，但是需要知道是否达到 90% 以上的效果），我们选择六边形网格的六个定点 poi 以及中心 poi，共计 7 个 poi 去请求线上，并且用返回的数据和热点 poi 请求返回的数据做 DIFF，核心 39 城每个城市抽查 10 组，最终得到一个比例。然后通过不同的空间索引精度，去挨个验证出一个最佳的索引精度。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXkiaPytJYOXuXj0EJUGRIE6ticoiaHB9AiccDV3tIXUHQ4Q3n0QTQWnEyz6sH0l6JCcCYw2XcXaRoRCA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 （2）门详页

门详 DIFF 验证，抽选出重点 KA 门店，定时任务 check 门店下的分类和商品情况。如果抽查整体 DIFF 异常率超过 30%，我们则会触发新一轮的异构任务。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXkiaPytJYOXuXj0EJUGRIE6spq1HsQibBn3TLMNUCIctafNVjgFZWoMgt8UNmTUgtQlUqIGbZ9qGeA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 **五、落地过程**

上面的方案构思经过无数次的推演以及前期验证确保了方案的可行性。那么接下来就是到了落地环节，在这个环节里，我们发现其实客户端本身也有缓存，但是缓存的大小非常有限。客户端采用的 localStorage 本地缓存存储，是基于用户的手机内存做的，5M 左右的大小空间。所以适合做一些兜底灾备的缓存，比如服务端彻底 down 的情况下，应急去读客户手机内存缓存。用户每次成功请求服务端后，都会重写更新缓存，于是，一个可行（邪恶😈）的想法冒了出来，我们为啥不优先利用起来客户端的缓存呢？所以经过我们内部的反复算计下，决定也把客户端纳入我们容灾数据备份方案的一个环节。所以在落地的过程中，分拆了五个大的模块：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXkiaPytJYOXuXj0EJUGRIE6p4PFTM2rVYWI7mlYuEibu7ITAAIffzeic63wFicibXpQsv46OMCHYD9YHg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 1、客户端交互模块

客户端缓存每次请求服务端的时候，都会传给服务端标识：用户手机是否有上次缓存、缓存时间，服务端拿到标识后，如果内部数据发生异常，就会根据标识来判断是否让客户端去读取用户手机的内存缓存。客户端缓存的优点就是可以满足当前用户的特殊推荐策略，但是这种场景针对第一次请求的用户是无法生效的，另外如果缓存时间超过 1-2 天后，这份缓存的有效性也是一个问题。为此我们做了一个比较，并且最终决定双管齐下，相互弥补缺点，起到 1+1>2 的效果。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXkiaPytJYOXuXj0EJUGRIE6LodSZw2G0LMILAFXqzaRPwicrHJlP07fn1RgUeITicYiaSTQb3ic7ibb9sg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

对于秒送首页、频道页，全部请求数据大小也就是 2600kb 左右，localStorage 完全可以覆盖。但是门详页就不同了，在一个热点 poi 下，首页推出的门店数量可能上百个，每个门店如果全部缓存商品的话，远远超过了 5M，所以现阶段客户端缓存商品不太现实，只能由服务端来承接。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXkiaPytJYOXuXj0EJUGRIE6w5kXbokyCKphMR1H3wywIUH5L1g2e0lntOrphtYZqYK80C38iazMfZQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 3.2.5. 网格模块

通过网格中台提供的 JMF 组件，通过 H3 空间索引算法，异构出精度 7、精度 8 的两套六边形网格数据，并且调用网格热点接口来标识出当前网格的热点 poi 经纬度，录入至 mysql 中。以供 worker 异构的时候去查询基础网格经纬度数据库表。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXkiaPytJYOXuXj0EJUGRIE6xtKZmicBicGDpW70QqRNrk6WRNJyic5FRCt0KSN4POGOXUQHGGdTheBmQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 3.2.6. 任务模块

任务模块核心职责是为了异构数据，因此逻辑上不会特别复杂。需要注意点的点就是在异构数据的过程中，我们要考虑以下几个点：

• 异构数据环境以及并发对底层 RPC 中台的压力

• 做好异常重试、后门工具以防个别数据异构失败

• 异构的时间和任务执行是否成功的报警

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXkiaPytJYOXuXj0EJUGRIE6PZibe1yz0CHdJMq5p58UH4GhF01toaYxVqeV9xrDMU14W7DTPajsf0w/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 4、灰度模块

灰度方案的设计里，主要包含了两部分，一个是上线灰度前的压力测试，一个是上线后的灰度方式。

• 压力测试

数据备份最终的存储均在 redis 中，所以 redis 的集群是否能抗住线上的流量是我们上线前需要确定的。

• 灰度方案

除了机器灰度之外，我们增加了按照 pin、按照门店、按照城市灰度的策略，保障我们循序渐进的推进上线。

#### 3.2.7. 容灾备份数据切换方案

• 首页、频道页 SOA 层容灾备份数据切换场景

（1）在备份数据的切换流程上，灵活增加了功能开关，在新逻辑的场景下，服务端根据前端传过来的标识和缓存时间，和当前时间进行对比，如果用户的手机在 N 天内，我们视为有效缓存，则会直接使用客户端缓存，不会使用备份缓存

（2）用户手机有缓存的场景下，如果用户手机的缓存时间和当前时间超过 N 天，则判断用户手机的缓存无效，服务端会请求备份 redis 并且返回给客户端，缓存到用户手机上

（3）用户手机无缓存的的场景下，判断底层数据是否缺失，如果缺失，使用容灾备份 redis 数据

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXkiaPytJYOXuXj0EJUGRIE6ia8QdGdHzdqalN8aWHMokmvDEcwjibcJyR852OTYFnhfjCRvgGsYfJZg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

• 门详页 SOA 层容灾备份数据切换场景

门详的分类和商品加载是二次渲染。在第一阶段调用场景下，客户端会请求类目树，服务端组装好所有的类目树并且返回给客户端。在第二阶段调用场景下，前端会把类目树 id 传递给服务端，服务端根据分类 id 去聚合分类下的商品。

（1）一阶段下门详的分类来自于多数据源的聚合，所以不能根据单一场景判断进行是否切换备份数据。我们在整个类目树的组装阶段，判断是否类目树整体为空，如果类目树整体为空，则降级到备份类目 redis 缓存数据。

（2）二阶段下商品核心来自搜索，我们会在这个阶段对搜索返回的商品结果进行校验，如果商品信息为空，会执行 1 次默认重试，如果重试后依然为空，则直接降级到商品容灾 redis 数据上。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXkiaPytJYOXuXj0EJUGRIE6ibzsJNRWanjWofyvKmTU3E7Nw5TWMqmxDH8VMmaPvSlabIrMX7s0qlA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 六、效果展示

#### 3.2.8. 首页、频道页

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXkiaPytJYOXuXj0EJUGRIE6TOsok7icibicRy8MTiciaia1fMWvnco7h1yJvdKA6CuZdOb6Ap80tcECrwYA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 2、门详页

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXkiaPytJYOXuXj0EJUGRIE6B0UQNmC0fSicTxiao0XvGGPXKicPamgMnGOgNFZVnoFcPRUIp7Bllx6LA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 七、思考总结

以上是我们针对秒送 LBS 场景下的 C 端 SOA 服务容灾建设 - 数据备份篇的整个过程，那么思考下，对于流量后端服务的容灾，除了数据备份，我们还有哪些手段呢？以下是一些思考总结，我们下一个阶段也准备去做故障切换机制的打造，做到异常转移。如果大家感兴趣的话，可以评论区点赞留言，如果反馈热烈的话，下一期可以介绍下，我们 LBS 场景下的复杂问题定位能力建设是如何做到分钟级定位问题~ 也是独家~（卖个关子）。

1、容灾数据备份：

• 定期备份：定期对服务的容灾数据进行备份，确保在发生灾难时备用。

2、服务冗余：

• 主备模式：设置主服务和备份服务，当主服务不可用时，备份服务可以接管。

• 多活模式：多个服务实例同时运行，可以均衡负载，当某个实例出现故障时，其他实例可以继续提供服务。

3、故障切换机制：

• 自动故障切换：通过监控和自动化工具，能够在检测到故障时自动切换到备份服务或实例。

• 手动故障切换：在需要时由管理员手动切换到备用服务。

4、负载均衡：

• 硬件负载均衡器：使用硬件设备进行流量分发，确保服务的高可用性。

• 软件负载均衡器：使用软件（如 Nginx、HAProxy）进行流量管理和故障切换。

5、服务监控和告警：

• 实时监控：对服务的运行状态进行实时监控，及时发现和处理故障。

• 告警系统：设置告警机制，当出现异常情况时，能够及时通知相关人员进行处理。

6、业务连续性计划（BCP）和灾难恢复计划（DRP）：

•BCP：制定业务连续性计划，确保在灾难发生时业务能够继续运行。

•DRP：制定详细的灾难恢复计划，明确灾难发生后的恢复步骤和流程。

7、定期演练：

• 容灾演练：定期进行容灾演练，模拟各种灾难场景，验证和完善容灾方案。

![](https://mmbiz.qpic.cn/mmbiz_jpg/g9uU4FY8hdfyDOgjViaDP8bZ57HngFicyoPjYwf6qRIHsDqDSIyQiaRR2fDb1ZmGBKWbhQnkdUAicqRHoXmTdibCPOg/640?wx_fmt=jpeg&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=wxpic)

扫一扫，加入技术交流群