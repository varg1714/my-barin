---
source: https://mp.weixin.qq.com/s/inFhwTpbRySRxbIo0jFVDQ
create: 2025-04-17 21:40
read: false
---
《架构师之路：架构设计中的 100 个知识点》

40.session 一致性

今天系统性和大家聊一聊 **session 一致性**。

**什么是 session？**  

服务器为每个用户创建一个会话，存储用户的相关信息，以便多次请求能够定位到同一个上下文。

Web 开发中，web-server 可以自动为同一个浏览器的访问用户自动创建 session，提供数据存储功能。最常见的，会把用户的登录信息、用户信息存储在 session 中，以保持登录状态。

**什么是 session 一致性问题？**

只要用户不重启浏览器，每次 http 短连接请求，理论上服务端都能定位到 session，保持会话。

![](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOyR8icObiaWH0Rqxy2Wxp1qy1OPUibuqUauMD8dicVggibNxLNTjkhBVD2zbLQSngaGdWDr3E2d7h3IBnA/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

当只有一台 web-server 提供服务时，每次 http 短连接请求，都能够正确路由到存储 session 的对应 web-server。

_画外音：废话，因为只有一台。_

此时的 web-server 是无法保证高可用的，采用 “冗余 + 故障转移” 的多台 web-server 来保证高可用时，每次 http 短连接请求就不一定能路由到正确的 session 了。

![](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOyR8icObiaWH0Rqxy2Wxp1qy167nlucicLdjvDQ0CV0JOSpLdic35wZSNiaiakKE7YYprI0Uq8MYTA76DKQ/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如上图，假设用户包含登录信息的 session 都记录在第一台 web-server 上，反向代理如果将请求路由到另一台 web-server 上，可能就找不到相关信息，而导致用户需要重新登录。

**在 web-server 高可用时，如何保证 session 路由的一致性呢？**

常见的，有这么 5 种方法。

**方案一：session 同步法。**

![](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOyR8icObiaWH0Rqxy2Wxp1qy1TLD5NicwPENdyiaJ8UE9brHEiauXol0V3Xe04G6jW64dibn74DiaeBmAQoA/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**思路**：多个 web-server 之间相互同步 session，这样每个 web-server 之间都包含全部的 session。

**优点**：web-server 支持的功能，应用程序不需要修改代码。

**不足**：

*   session 的同步需要数据传输，占**内网带宽**，有时延
    
*   所有 web-server 都包含所有 session 数据，数据量受内存限制，无法水平扩展
    
*   有更多 web-server 时要歇菜
    

**方案二：客户端存储法。**

![](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOyR8icObiaWH0Rqxy2Wxp1qy1VfRPiaRxFRD9WuTht6gd6TrvzzsHB7FIKvXHumaA4A1x4eIkPWe5wHA/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**思路**：服务端存储所有用户的 session，内存占用较大，可以将 session 存储到浏览器 cookie 中，每个端只要存储一个用户的数据了。

**优点**：服务端不需要存储。

**缺点**：

*   每次 http 请求都携带 session，占**外网带宽**
    
*   数据存储在端上，并在网络传输，存在泄漏、篡改、窃取等安全隐患
    
*   session 存储的数据大小受 cookie 限制
    

“端存储” 的方案虽然不常用，但确实是一种思路。

**方案三 + 方案四：反向代理 hash 一致性。**

**思路**：web-server 为了保证高可用，有多台冗余，反向代理层能不能做一些事情，让同一个用户的请求保证落在一台 web-server 上呢？

![](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOyR8icObiaWH0Rqxy2Wxp1qy1HyicJoxkDDsllhxxGMcQIMVUibK6B13LIBliarqUicx7Mvn2gp2AYYFKVQ/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**其一，四层代理 hash。**

反向代理层使用用户 ip 来做 hash，以保证同一个 ip 的请求落在同一个 web-server 上。

![](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOyR8icObiaWH0Rqxy2Wxp1qy1mVeqorZ7lhz7zWkNmAe4b1Ft41ZjFWc81sGvib3Qy8xiauqCa82t9tbQ/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**其二，七层代理 hash。**

反向代理使用 http 协议中的某些业务属性来做 hash，例如 sid，city_id，user_id 等，能够更加灵活的实施 hash 策略，以保证同一个浏览器用户的请求落在同一个 web-server 上。

**优点**：

*   只需要改 nginx 配置，不需要修改应用代码
    
*   负载均衡，只要 hash 属性是均匀的，多台 web-server 的负载是均衡的
    
*   可以支持 web-server 水平扩展
    

**不足**：

*   如果 web-server 重启，一部分 session 会丢失，产生业务影响，例如部分用户重新登录
    
*   如果 web-server 水平扩展，rehash 后 session 重新分布，也会有一部分用户路由不到正确的 session
    

session 一般是有有效期的，所有不足中的两点，可以认为等同于部分 session 失效，一般问题不大。

对于四层 hash 还是七层 hash，个人推荐前者：**让专业的软件做专业的事情**，反向代理就负责转发，尽量不要引入应用层业务属性。

**方案五：后端统一存储。**

![](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOyR8icObiaWH0Rqxy2Wxp1qy1JZuKJDhpeeTicTbDxJW9kG37fI3XLhOxyo31KuMnXhQrt5JRyGlQicsA/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**思路**：将 session 存储在 web-server 后端的存储层，数据库或者缓存。

**优点**：

*   没有安全隐患
    
*   可以水平扩展，数据库 / 缓存水平切分即可
    
*   web-server 重启或者扩容都不会有 session 丢失
    

**不足**：增加了一次网络调用，并且需要修改应用代码。

对于 db 存储还是 cache，个人推荐后者：session 读取的频率会很高，数据库压力会比较大。如果有 session 高可用需求，cache 可以做高可用，但大部分情况下 session 可以丢失，一般也不需要考虑高可用。

**总结**

**保证 session 一致性**的架构设计常见方法：

*   session 同步法：多台 web-server 相互同步数据
    
*   客户端存储法：一个用户只存储自己的数据
    
*   反向代理四层 hash 一致性：保证一个用户的请求落在一台 web-server 上
    
*   反向代理七层 hash 一致性：保证一个用户的请求落在一台 web-server 上
    
*   后端统一存储：web-server 重启和扩容，session 也不会丢失（用得最多）
    

知其然，知其所以然。

**思路比结论更重要。**

== 全文完 ==

附录：近 5 年系列内容

**1. 架构篇，已完结：**《[80 个经典架构问题！](https://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651975539&idx=1&sn=309b491524f10ddbab2fb9af0321ff7a&scene=21#wechat_redirect)》

**2. IM 篇，已完结：**《[关于即时通讯架构的一切！](https://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651975468&idx=1&sn=54ab265bee4998da9a0d32091699cb1d&scene=21#wechat_redirect)》

**3. 架构篇，进行中：**《架构设计中 100 个知识点》

**4. AI** **篇****，进行中：**《deepseek 原理应用与实践》

**5.** **知行合一篇****，规划中，**《[1743 天，299 万...](https://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651975938&idx=1&sn=369a3d4ce6480dc32cf685e263fd3028&scene=21#wechat_redirect)》

讲技术的宝藏号，日更，保护起来。

**点赞，转发，在看**，感激不尽！