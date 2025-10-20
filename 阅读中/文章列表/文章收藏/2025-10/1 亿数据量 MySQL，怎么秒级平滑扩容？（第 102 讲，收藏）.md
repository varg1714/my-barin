---
source: https://mp.weixin.qq.com/s/Zuadr3v_G2W_eD8_vncwSg
create: 2025-10-10 13:04
read: true
knowledge: true
knowledge-date: 2025-10-10
tags:
  - 系统架构
  - 数据库
summary: "[[数据库扩容方案]]"
---
《架构师之路：架构设计中的 100 个知识点》

102.MySQL 秒级扩容

数据库秒级平滑扩容，这个问题之前写过，上周有个童鞋在评论区问我，说找不到原文了。这个方案实操性很强，曾经在 58 我们就是这么玩的。

另外，思路比结论更重要。

**一般来说，并发量大，吞吐量大的互联网分层架构是怎么样的？**

数据库上层都有一个微服务，服务层记录 “业务库” 与“数据库实例配置”的映射关系，通过数据库连接池向数据库路由 sql 语句。

![](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOzOGwoVobJtZTvibPmgp6DGTDicxhdHS3wrArZfuzkc7OUOLAu8CzY2SQU6lFwNXOicibC3sSAqFSr4pA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

如上图所示，服务层配置用户库 user 对应的数据库实例 ip。

_画外音：其实是一个内网域名。_

**该分层架构，如何应对数据库的****高可用****？**

数据库高可用，很常见的一种方式，使用双主同步 + keepalived + 虚 ip 的方式进行。

![](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOzOGwoVobJtZTvibPmgp6DGTWicwsXpucQ1QMZ1WLkyyKEzJzNtRRHLHr5Lse93ScRHMics7rKDJnVhg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)

如上图所示，两个相互同步的主库使用相同的虚 ip。

![](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOzOGwoVobJtZTvibPmgp6DGTDpNcQjHJ6lhzR3vbmRnhx32QEISagDblQibfP8ER7UEdWBG2vkrysmA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2)

当主库挂掉的时候，虚 ip 自动漂移到另一个主库，整个过程对调用方透明，通过这种方式保证数据库的高可用。

_画外音：关于高可用，__之前介绍过，本文不再展开。_

**该分层架构，如何应对****数据量的暴增****？**

随着数据量的增大，数据库要进行水平切分，分库后将数据分布到不同的数据库实例（甚至物理机器）上，以达到降低数据量，增强性能的扩容目的。

![](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOzOGwoVobJtZTvibPmgp6DGT2zfBUuaB18yMn41ETMv9YhLcGCDyEEOib3s00yVnWuTSib1uP0CgwaKg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3)

如上图所示，用户库 user 分布在两个实例上，ip0 和 ip1，服务层通过用户标识 uid 取模的方式进行寻库路由，模 2 余 0 的访问 ip0 上的 user 库，模 2 余 1 的访问 ip1 上的 user 库。

_画外音：此时，水平切分集群的读写实例加倍，单个实例的数据量减半，性能增长可不止一倍。_

综上三点所述，大数据量，高可用的互联网微服务分层的架构如下：

![](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOzOGwoVobJtZTvibPmgp6DGTdclkCtxvlGCicTicjaA4MRhcYhsrb9zRNycIWMJpNn1gSSq1PMPicsArQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4)

既有水平切分，又保证高可用。

**如果数据量持续增大，2 个库性能扛不住了，该怎么办呢？**

此时，需要继续水平拆分，拆成更多的库，降低单库数据量，增加库主库实例（机器）数量，提高性能。

**新的问题来了，分成 n 个库后，随着数据量的增加，要增加到 2*n 个库，数据库如何扩容，数据能否平滑迁移，能够持续对外提供服务，保证服务的可用性？**

_画外音：你遇到过类似的问题么？_

**停服扩容，是最容易想到的方案？**

在讨论秒级平滑扩容方案之前，先简要说明下停服扩容的方案的步骤：

1. 站点挂一个公告 “为了为广大用户提供更好的服务，本站点 / 游戏将在今晚 00:00-2:00 之间升级，届时将不能登录，用户周知”；

_画外音：见过这样的公告么，实际上在迁移数据。_

2. 微服务停止服务，数据库不再有流量写入；

3. 新建 2*n 个新库，并做好高可用；

4. 写一个小脚本进行数据迁移，把数据从 n 个库里 select 出来，insert 到 2*n 个库里；

5. 修改微服务的数据库路由配置，模 n 变为模 2*n；  

6. 微服务重启，连接新库重新对外提供服务；

整个过程中，最耗时的是第四步数据迁移。

**如果出现问题，如何进行回滚？**

如果数据迁移失败，或者迁移后测试失败，则将配置改回旧库，恢复服务即可。

**停服方案有什么优劣？**

优点：简单。

缺点：

1. 需要停止服务，方案不高可用；

2. 技术同学压力大，所有工作要在规定时间内完成，根据经验，压力越大越容易出错；

_画外音：这一点很致命。_

3. 如果有问题第一时间没检查出来，启动了服务，运行一段时间后再发现有问题，则难以回滚，如果回档会丢失一部分数据；

**有没有秒级实施、更平滑、更帅气的方案呢？**

![](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOzOGwoVobJtZTvibPmgp6DGTdclkCtxvlGCicTicjaA4MRhcYhsrb9zRNycIWMJpNn1gSSq1PMPicsArQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=5)

再次看一眼扩容前的架构，分两个库，假设每个库 1 亿数据量，**如何**平滑扩容，增加实例数，降低单库数据量呢？三个简单步骤搞定。

**步骤一：修改配置。**

![](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOzOGwoVobJtZTvibPmgp6DGTtQJjJa3MmwzsXwUNBwIdEDEIpa4F4p4zxWT9qIv1yNRWP265kMx6rw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=6)

主要修改两处：

其一，数据库实例所在的机器做**双虚 ip**：

1. 原 %2=0 的库是虚 ip0，现增加一个虚 ip00；

2. 原 %2=1 的库是虚 ip1，现增加一个虚 ip11；

其二，修改服务的配置，将 2 个库的**数据库配置**，改为 4 个库的数据库配置，修改的时候要注意旧库与新库的映射关系：

1. %2=0 的库，会变为 %4=0 与 %4=2；

2. %2=1 的部分，会变为 %4=1 与 %4=3；

_画外音：这样能够保证，依然路由到正确的数据。_

**步骤二：reload 配置，实例扩容。**

![](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOzOGwoVobJtZTvibPmgp6DGTYdtyLG7P9D4XsqP2EXJ2FaV0Qo4KXTicjDicrZs70B6TEdKA0ic6fNLpA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=7)

服务层 reload 配置，reload 可能是这么几种方式：

1. 比较原始的，重启服务，读新的配置文件；

2. 高级一点的，配置中心给服务发信号，重读配置文件，重新初始化数据库连接池；

不管哪种方式，reload 之后，数据库的实例扩容就完成了，原来是 2 个数据库实例提供服务，现在变为 4 个数据库实例提供服务，这个过程一般可以在秒级完成。

![](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOzOGwoVobJtZTvibPmgp6DGTYdtyLG7P9D4XsqP2EXJ2FaV0Qo4KXTicjDicrZs70B6TEdKA0ic6fNLpA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=8)

整个过程可以逐步重启，对服务的正确性和可用性完全没有影响：

1. 即使 %2 寻库和 %4 寻库同时存在，也不影响数据的正确性，因为此时仍然是双主数据同步的；

2. 即使 %4=0 与 %4=2 的寻库落到同一个数据库实例上，也不影响数据的正确性，因为此时仍然是双主数据同步的；

完成了实例的扩展，会发现每个数据库的数据量依然没有下降，所以第三个步骤还要做一些收尾工作。

_画外音：这一步，数据库实例个数加倍了。_

**步骤三：收尾工作，数据收缩。**

![](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOzOGwoVobJtZTvibPmgp6DGTIahl7W9Tf8Jib5aeIWExUDyvgXXcDaNOpj4YmHoLlqVethQB5F7U6ibg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=9)

有这些一些收尾工作：

1. 把双虚 ip 修改回单虚 ip；

2. 解除旧的双主同步，让成对库的数据不再同步增加；

3. 增加新的双主同步，保证高可用；

4. 删除掉冗余数据，例如：ip0 里 %4=2 的数据全部删除，只为 %4=0 的数据提供服务；

_画外音：这一步，数据库单实例数据量减半了。_

**总结**

![](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOzOGwoVobJtZTvibPmgp6DGTmBTw0nicWUrg8mrrkNHqHattRQgNYjwV3dj52yQ3RJ36wWhot4gZ8vg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=10)

互联网大数据量，高吞吐量，高可用微服务分层架构，数据库实现秒级平滑扩容的三个步骤为：  

1. 修改配置（双虚 ip，微服务数据库路由）；

2. reload 配置，实例增倍完成；

3. 删除冗余数据等收尾工作，数据量减半完成；

知其然，知其所以然。

**思路比结论更重要。**

== 全文完 ==

做了个 70 天短视频行动营，10 月下周开营。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/YrezxckhYOyLvK4ZlibqmDiasbgoWeLtic8DoBnTLJky5NRNAFDiaSnnUsWryzH47Y9FpdlvbEpj2XQ8XhOicTQdbrQ/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=3)

面向零基础，只需每天投入 25 分钟，70 天起号自己的短视频，详见：

《[70 天短视频行动营，欢迎！](https://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651978738&idx=1&sn=0d1a7a567d16c9daef2e76bc877159cb&scene=21#wechat_redirect)》

10 月社群直播主题：技术人的第二曲线！

欢迎预约！