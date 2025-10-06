---
source: https://mp.weixin.qq.com/s?__biz=Mzg3Njc0NTgwMg==&mid=2247503399&idx=1&sn=486abeda98f6d2faf2150b006ff6b94b&scene=58&subscene=0&poc_token=HFTNwmijyNHW1IfH1ZDtEcHH0AhQJeOtDxEzJ6AA
create: 2025-09-11 21:23
read: true
knowledge: true
knowledge-date: 2025-09-12
tags:
  - 网络
  - 计算机原理
summary: "[[TLS-1.3 多了一个RTT的分析]]"
---
**一. 前言**

在正文开始之前，先简要介绍一下 TLS 1.3 与 TLS 1.2 有哪些主要差异：

 1. 更快的响应速度：

 a. TLS 完整握手时间从 2 RTT 减少为 1 RTT

![](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754TbaVc2TcGPuEN32FsdZDAPmiasWdcB79nj2Jr9RccnEHw0jPVRb2zD2Vr61GyCvOIV0QlsN5yhSGw/640?wx_fmt=png&from=appmsg#imgIndex=0)

 b. 增加 0 RTT 模式（以牺牲某些安全特性为代价）

 2. 更安全:

 a. 加密更多握手数据

 b. 更简洁更安全的加密套件：TLS 1.3 极大地简化了加密套件的设计，移除了不安全的加密算法。目前标准定义了 5 种加密套件，而非 TLS 1.2 中上百种复杂的可选组合，大幅降低了复杂性。

为了向大家提供更快、更安全的服务，笔者在前段时间为某个服务升级支持了 TLS 1.3，然而在升级过程中发现 RTT 并没有按预期减少，所以进行了排查与记录，并分享给大家。

**二. 问题排查**

笔者到测试 Server 的 RTT 约 36 ms，就当笔者升完服务，准备开开心心验收时，天塌了。打开浏览器一看 TLS 握手时长是 2 RTT，说好的 1 RTT 呢？

![](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754TbaVc2TcGPuEN32FsdZDAPtErjIZ7JP4oTkJTHgI1vytB8Lcwswa85R45UnqP0RVw09WicMCbftnw/640?wx_fmt=png&from=appmsg#imgIndex=1)

在反复确认了升级的软件版本没有异常，TLS1.3 相关配置没有异常之后，我们请出网络数据包分析利器 Wireshark。

![](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754TbaVc2TcGPuEN32FsdZDAPKIZ1wfKic2c9ibtCibHtQkqGibzHFVXNz7OA7Y6GclVvKGf5M2Vjaf2bOQ/640?wx_fmt=png&from=appmsg#imgIndex=2)

乍看之下没有什么异常，TLS1.3 握手正常，也到了 HTTP 请求阶段。那我们再对比其他站点的请求看看。

![](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754TbaVc2TcGPuEN32FsdZDAPynV5I2FF5iaUZYaoUia0Qxict0aott1tl52vltKtoghZJyyDmmib4ibHZ2Q/640?wx_fmt=png&from=appmsg#imgIndex=3)

仔细对比和分析后我们发现，多出来的 1 RTT 产生在第 12 个包，Server 端收到了 Client 的 Ack 后才发送了 Certificate Verify 和 Finished。所以我们判断可能和 TCP 层面的某些机制有关。

![](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754TbaVc2TcGPuEN32FsdZDAP2KTxSeDIsbtfVnoA0xhw84kWb4x6icPpIv9n8UPKu1MUhNy9RFbkmJg/640?wx_fmt=png&from=appmsg#imgIndex=4)

这里就引出一个问题，TCP 一次可以批量发送的数据受到哪些因素的影响：

1. 接收方窗口大小（RWND）： 看抓包显示 Win 足够大，不会阻塞 Server 端传输

2. 接收和发送方的 MSS：都是 1460，正常范围

3. 拥塞控制：

a. 慢启动：根据 rfc6928  标准，初始窗口为 10。经确认我们服务器上也确实为 10。所以最大可以发送的数据量为 10 * MSS（1460）= 14 KB，Server 发送的数据量尚未达到初始窗口限制

b. 拥塞避免：比如丢包、或者 RTT 变长，看数据包判断没有触发

4. tcp_wmem: 由最小、默认、最大三个值组成，最小 4KB,  默认 16KB。考虑到测试节点没有压力，不会触发此限制。其次笔者尝试调大最小值做验证，依然没有解决问题

5. 其他因素

既然没能直接从数据包中推测出原因，咱们就再上服务器找找原因，根据之前的推断笔者用 tcp 作为关键词搜了一下服务配置，发现了线索: 

```
tcp_nodelay off
```

这段代码开启了 Nagle 算法。翻阅 RFC896 我们可以发现 Nagle 算法是为了解决小数据包问题，比如下面这种情况：各种协议必要的头尾数据占 58 bytes，真正需要传输的数据只有 1 byte，有效载荷比不到 2%。

![](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754TbaVc2TcGPuEN32FsdZDAPHnj8J7Pa31xwMFku97ibN9heB0ibGeBBTiauxaTChjTp4CiaR3Sq7cMZgg/640?wx_fmt=png&from=appmsg#imgIndex=5)

于是 Nagle 算法通过一种自适应的方法来减少小数据包的数量，提升网络效率。其本质是通过增加时延来换取更高的有效载荷比

Nagle 算法的核心内容可以概括为：不一下子把所有小分组都发出去，而是等到前一个小分组的 ACK 收到或者攒够一个 MSS 大小再一起发

Nagle 算法的规则：

1.  满载的数据包，允许发送
    
2.  包含 FIN，允许发送
    
3.  TCP_NODELAY 被设置，允许发送
    
4.  所有送的小数据包（长度小于 MSS）都被确认了，允许发送
    
5.  上述条件都未满足，但发送了超时（一般为 200 ms），则立即发送
    

根据上述规则，我们可以看到 Server 发送的序号为 9 和 12 的都是未满载的数据包，所以 12 号包是等 Server 收到了 Client 对 9 号包的 Ack 后才发送的。这就增加了 1 RTT。

![](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754TbaVc2TcGPuEN32FsdZDAP8vFILdlLG8TSYGDs7desbATrCkfXLaN8DhuT5fZkhicEHO1FJ6nIwjw/640?wx_fmt=png&from=appmsg#imgIndex=6)

Nagle 算法提出于 1984 年，那时的带宽、数据包处理能力都远不如今天。而在当前环境下，对于时延敏感的应用，通常建议关闭 Nagle 算法。

经确认 Nagle 算法并不适合我们现在的场景，所以关闭 Nagle 算法后再做验证，TLS 握手时间果然只有 1 RTT。看数据包 Serever 端的 Certificate Verify 和 Finished 包也不需要等 Client 的 Ack 就直接发送了。

![](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754TbaVc2TcGPuEN32FsdZDAP5P5X8ds5oACMyw7EeeH6yYAhgY2nLXAffAibOq2aEM4PNG7nYl0vCvA/640?wx_fmt=png&from=appmsg#imgIndex=7)

![](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754TbaVc2TcGPuEN32FsdZDAPAib2J7icJWcHSUvXAuDGMKib8tibIHqs8NU9JHia1ZzfydOjzz1DHReOpbA/640?wx_fmt=png&from=appmsg#imgIndex=8)

至此问题已圆满解决。我们已将上述优化上线，可将国内用户首次访问时延减少 10 ～ 40ms，海外最高减少上百毫秒。

咱们再回过头来仔细看一下 TLS 1.3 的握手流程，会多一层理解：1 RTT 只是 TLS 交互逻辑上的，真正端到端的交互时间还受到底层协议比如 TCP 的影响。

![](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754TbaVc2TcGPuEN32FsdZDAP1ic7Mf7jsa6m2SkJ01GzDlrawCtnvEOfgc0fsJEwlEQWuAZAe4S5ziaA/640?wx_fmt=png&from=appmsg#imgIndex=9)

**三. 结语**

这次问题的排查过程让笔者对网络协议的实际行为有了更深入的理解。同时笔者也从基础的网络知识中受益颇多，所以将整个过程整理分享出来，希望也会对你有所帮助。

**推荐阅读：**

*   对 RFC 8446（即 TLS 1.3）的详细解读： https://blog.cloudflare.com/rfc-8446-aka-tls-1-3/
    
*   TLS 1.3 愿望清单：https://www.ietf.org/proceedings/87/slides/slides-87-tls-5.pdf
    
*   RFC 8446（TLS 1.3 规范）： https://datatracker.ietf.org/doc/html/rfc8446
    
*   Nagle 算法以及所谓 TCP 粘包 ：https://www.cnblogs.com/jojop/p/14376423.html
    
*   RFC896（IP/TCP 互联网中的拥塞控制）： https://datatracker.ietf.org/doc/html/rfc896
    

-End-

作者丨 House

**开发者问答**

**大家还有过哪些有趣的性能优化或者 Debug 经历？**

欢迎在留言区分享你的见解~

转发本文至朋友圈并留言，即可参与**下方抽奖****⬇️**

小编将抽取 1 位幸运的小伙伴获取 **JOJO 的奇妙冒险 石之海 冷水杯**

**抽奖截止时间：9 月 2 日 12:00**

如果喜欢本期内容的话，欢迎点个 “在看” 吧！

**往期精彩指路**

*   [全链路压测改造之全链自动化测试实践](https://mp.weixin.qq.com/s?__biz=Mzg3Njc0NTgwMg==&mid=2247487748&idx=1&sn=c9cbcacf3bba25b478abf2a0f5c0e75f&scene=21#wechat_redirect)
    
*   [哔哩哔哩⼤数据建设之路—实时 DQC 篇](https://mp.weixin.qq.com/s?__biz=Mzg3Njc0NTgwMg==&mid=2247493174&idx=1&sn=648bf0ffb5e31b1c211d22e636e2c3df&scene=21#wechat_redirect)
    
*   [Apache Kyuubi 在 B 站大数据场景下的应用实践](https://mp.weixin.qq.com/s?__biz=Mzg3Njc0NTgwMg==&mid=2247491092&idx=1&sn=b09492563c760775ed61f75db1bd5822&scene=21#wechat_redirect)
    

[通用工程](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3Njc0NTgwMg==&action=getalbum&album_id=3289447926347317252#wechat_redirect)丨[大前端](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3Njc0NTgwMg==&action=getalbum&album_id=2390333109742534656#wechat_redirect)丨[业务线](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3Njc0NTgwMg==&action=getalbum&album_id=3297757408550699008#wechat_redirect)

[大数据](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3Njc0NTgwMg==&action=getalbum&album_id=2329861166598127619#wechat_redirect)丨 [AI](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3Njc0NTgwMg==&action=getalbum&album_id=2782124818895699969#wechat_redirect) 丨[多媒体](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3Njc0NTgwMg==&action=getalbum&album_id=2532608330440081409#wechat_redirect)