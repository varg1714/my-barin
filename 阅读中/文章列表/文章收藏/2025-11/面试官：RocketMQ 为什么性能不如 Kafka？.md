---
source: https://mp.weixin.qq.com/s/3j1dkkYIo1FqKe4L84xCEA
create: 2025-11-18 21:02
read: true
knowledge: true
knowledge-date: 2025-11-18
tags:
  - 消息队列
summary: "[[Rocket 与 Kafka的区别]]"
---
![](https://mmbiz.qpic.cn/sz_mmbiz_png/J0g14CUwaZdAribHzlibOfhwicEh7luZaAgD5E4VQrJecJkBKfTvLXWYjGKzdZOWYK40eks5yL3NeUlmXrhePNV7g/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=0)

图解学习网站：[https://xiaolincoding.com](https://mp.weixin.qq.com/s?__biz=MzUxODAzNDg4NQ==&mid=2247507000&idx=1&sn=c045101b45dd70ec37f9b81361b09f14&scene=21#wechat_redirect)

在上篇文章[《rocketmq 是什么》](https://mp.weixin.qq.com/s?__biz=MzUxODAzNDg4NQ==&mid=2247535325&idx=2&sn=bcab1f79880ea654c753aebe2c0006a7&chksm=f98d0e77cefa876168f11f59af88ffdd13bacfbd719f805b36de7a7f8aabb458af278db1fda0&token=534946220&lang=zh_CN&scene=21#wechat_redirect)中，我们了解到 RocketMQ 的架构其实参考了 kafka 的设计思想，同时又在 kafka 的基础上做了一些调整。看起来，RocketMQ 好像各方面都比 kafka 更能打。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/icMUEqOiagpkgyybhSU27icv9O136ocrtoeOJn9jXRIicricwFPAKTrLEPtMhicOsTDbTpYY38XXXtgySClbOGuKuzYA/640?wx_fmt=jpeg&from=appmsg#imgIndex=1)

但 kafka 却一直没被淘汰，说明 RocketMQ 必然是有着不如 kafka 的地方。

是啥呢？ **性能**，严格来说是**吞吐量**。阿里中间件团队对它们做过压测，同样条件下，kafka 比 RocketMQ 快 50% 左右。但即使这样，RocketMQ 依然能每秒处理 10w 量级的数据，依旧非常能打。你不能说 RocketMQ 弱，只能说 Kafka 性能太强了。

不过这就很奇怪了，**为什么 RocketMQ 参考了 kafka 的架构，却不能跟 kafka 保持一样的性能呢**？在回答这个问题之前，我们来聊下什么是**零拷贝**。

## 零拷贝是什么

我们知道，消息队列的消息为了防止进程崩溃后丢失，一般不会放内存里，而是放磁盘上。那么问题就来了，消息从消息队列的磁盘，发送到消费者，过程是怎么样的呢？

### 消息的发送过程

操作系统分为**用户空间**和**内核空间**。程序处于用户空间，而磁盘属于硬件，操作系统本质上是程序和硬件设备的一个**中间层**。程序需要通过操作系统去调用硬件能力。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/icMUEqOiagpkgyybhSU27icv9O136ocrtoe17BIpXE8Izoib5Fiawa9Nf0hSozJRnS2zvAuictUlDUzg9BB27ic8k4dRg/640?wx_fmt=jpeg&from=appmsg#imgIndex=2)

如果用户想要将数据从磁盘发送到网络。那么就会发生下面这几件事：程序会发起**系统调用**`read()`，尝试读取磁盘数据，

*   • 磁盘数据从设备**拷贝**到内核空间的缓冲区。
    
*   • 再从内核空间的缓冲区**拷贝**到用户空间。
    

程序再发起**系统调用**`write()`，将读到的数据发到网络：

*   • 数据从用户空间**拷贝**到 socket 发送缓冲区
    
*   • 再从 socket 发送缓冲区**拷贝**到网卡。
    

最终数据就会经过网络到达消费者。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/icMUEqOiagpkgyybhSU27icv9O136ocrtoeNK8TERYkpiaicJ9GCvib62oic6UWGcjWUnXQKyfBoJPHpwJYQ8aGcepzibg/640?wx_fmt=jpeg&from=appmsg#imgIndex=3)

整个过程，本机内发生了 `2` 次**系统调用**，对应 `4` 次用户空间和内核空间的**切换**，以及 `4` 次数据**拷贝**。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/icMUEqOiagpkgyybhSU27icv9O136ocrtoeDhSQNXDbsm979fT1muicZSPEDslyyYAgibBibKs4MkWS26IDsdjdz1hDA/640?wx_fmt=jpeg&from=appmsg#imgIndex=4)

一顿操作猛如虎，结果就是同样一份数据来回拷贝。有没有办法优化呢？有，它就是零拷贝技术，常见的方案有两种，分别是 `mmap` 和 `sendfile`。我们来看下它们是什么。

### mmap 是什么

`mmap` 是操作系统内核提供的一个方法，可以将内核空间的缓冲区**映射**到用户空间。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/icMUEqOiagpkgyybhSU27icv9O136ocrtoex18hnbjID704qoTy4J6wgLIvENrjSBpIJTUx2ApfibC1xmsAp2SJPIA/640?wx_fmt=jpeg&from=appmsg#imgIndex=5)

用了它，整个发送流程就有了一些变化。程序发起**系统调用**`mmap()`，尝试读取磁盘数据，具体情况如下：

*   • 磁盘数据从设备**拷贝**到内核空间的缓冲区。
    
*   • 内核空间的缓冲区**映射**到用户空间，这里**不需要**拷贝。
    

程序再发起**系统调用**`write()`，将读到的数据发到网络：

*   • 数据从内核空间缓冲区**拷贝**到 socket 发送缓冲区。
    
*   • 再从 socket 发送缓冲区**拷贝**到网卡。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/icMUEqOiagpkgyybhSU27icv9O136ocrtoemq720A5Q0Ce46LKsGpPlYoj8Nt3PwiamZgDEqOrYuAEInt1TEXv9hiaQ/640?wx_fmt=jpeg&from=appmsg#imgIndex=6)

整个过程，发生了 `2` 次系统调用，对应 `4` 次用户空间和内核空间的切换，以及 `3` 次数据拷贝，对比之前，省下**一次**内核空间到用户空间的拷贝。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/icMUEqOiagpkgyybhSU27icv9O136ocrtoeicB7E1duaSxf1YhTiafyKpoibZxEHPeyJBMkJOdLhZibnDZSbdMV8EZFQQ/640?wx_fmt=jpeg&from=appmsg#imgIndex=7)

看到这里大家估计也蒙了，不是说零拷贝吗？怎么还有 3 次拷贝。mmap 作为一种零拷贝技术，指的是用户空间到内核空间这个过程不需要拷贝，而不是指数据从磁盘到发送到网卡这个过程零拷贝。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/icMUEqOiagpkgyybhSU27icv9O136ocrtoetekcSrZm6LyhQFAJN45icbvjuibc0OibyicQyaHw10owF837D6SheVGMAg/640?wx_fmt=jpeg&from=appmsg#imgIndex=8)

**确实省了一点，但不多**。有没有更彻底的零拷贝？有，用 `sendfile`.

### sendfile 是什么

`sendfile`，也是内核提供的一个方法，从名字可以看出，就是用来**发送文件数据**的。程序发起**系统调用**`sendfile()`，内核会尝试读取磁盘数据然后发送，具体情况如下：

*   • 磁盘数据从设备**拷贝**到内核空间的缓冲区。
    
*   • 内核空间缓冲区里的数据**可以**直接**拷贝**到网卡。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/icMUEqOiagpkgyybhSU27icv9O136ocrtoeEl1iaJjNgGI8qP9R6ReA8HlAlHKjowONDCoTUDgDLSiasrpcCVJTUcTg/640?wx_fmt=jpeg&from=appmsg#imgIndex=9)

整个过程，发生了 `1` 次系统调用，对应 `2` 次用户空间和内核空间的切换，以及 `2` 次数据拷贝。这时候问题很多的小明就有意见了，说好的**零**拷贝怎么还有 `2` 次拷贝？

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/icMUEqOiagpkgyybhSU27icv9O136ocrtoeuRcFxCs1F8iaic1NiaHSaR27zibXgTPHNSCwAuicrkMSGYtYibT4Aic2fNTyw/640?wx_fmt=jpeg&from=appmsg#imgIndex=10)

其实，这里的零拷贝指的是**零 CPU** 拷贝。也就是说 sendfile 场景下，需要的两次拷贝，都不是 CPU 直接参与的拷贝，而是其他硬件设备技术做的拷贝，不耽误我们 CPU 跑程序。

### kafka 为什么性能比 RocketMQ 好

聊完两种零拷贝技术，我们回过头来看下 kafka 为什么性能比 RocketMQ 好。这是因为 **RocketMQ 使用的是 mmap 零拷贝技术，而 kafka 使用的是 sendfile**。kafka 以更少的拷贝次数以及系统内核切换次数，获得了更高的性能。但问题又来了，为什么 RocketMQ 不使用 sendfile？参考 kafka 抄个作业也不难啊？我们来看下 `sendfile` 函数长啥样。

```
ssize_t sendfile(int out_fd, int in_fd, off_t* offset, size_t count);
// num = sendfile(xxx);

```

再来看下 `mmap` 函数长啥样。

```
void *mmap(void *addr, size_t length, int prot, int flags,
           int fd, off_t offset);
// buf = mmap(xxx)

```

注释里写的是两个函数的用法，`mmap` 返回的是数据的**具体内容**，应用层能获取到消息内容并进行一些逻辑处理。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/icMUEqOiagpkgyybhSU27icv9O136ocrtoeKuPbUBK43fTLmueqAP1QEeWWUrkGt16yIIvktCPXa2xqTfb5PjQacw/640?wx_fmt=jpeg&from=appmsg#imgIndex=11)

而 `sendfile` 返回的则是发送成功了几个**字节数**，**具体发了什么内容，应用层根本不知道**。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/icMUEqOiagpkgyybhSU27icv9O136ocrtoeZJIgfdsJ8305bhibLDRiaUApn68CXN9DBAIsyTrtBQr87iclXGKYhLsEg/640?wx_fmt=jpeg&from=appmsg#imgIndex=12)

而 RocketMQ 的一些功能，却需要了解具体这个消息内容，方便二次投递等，比如将消费失败的消息重新投递到死信队列中，如果 RocketMQ 使用 sendfile，那根本没机会获取到消息内容长什么样子，也就没办法实现一些好用的功能了。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/icMUEqOiagpkgyybhSU27icv9O136ocrtoeNXricYvMyvBdKibbSKibRDwJaaAsRN0fb3rRQkdR3OmJMzZz3wtibsfpRg/640?wx_fmt=jpeg&from=appmsg#imgIndex=13)

而 kafka 却没有这些功能特性，追求极致性能，正好可以使用 sendfile。

除了零拷贝以外，kafka 高性能的原因还有很多，比如什么批处理，数据压缩啥的，但那些优化手段 rocketMQ 也都能借鉴一波，唯独这个零拷贝，那是毫无办法。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/icMUEqOiagpkgyybhSU27icv9O136ocrtoe5XmicwWQ0CISV3S46S0y5xvfjZIFLXhjl2INrFIsQDRiauzcX7l1mA5Q/640?wx_fmt=jpeg&from=appmsg#imgIndex=14)

所以还是那句话，没有一种架构是完美的，一种架构往往用于适配某些场景，你很难做到既要又要还要。当场景不同，我们就需要做一些定制化改造，通过牺牲一部分能力去换取另一部分能力。做架构，做到最后都是在做折中。是不是感觉升华了。

### kafka 和 RocketMQ 怎么选？

这时候大家估计还是想知道 kafka 和 RocketMQ 到底该怎么选，用哪个。官方点的回答是 "这个要看场景的"。说了等于没说。这不是我的风格。我的标准只有一个，如果是大数据场景，比如你能频繁听到 spark，flink 这些关键词的时候，那就用 kafka。除此之外，如果公司组件支持，尽量用 RocketMQ。

现在大家通了吗？

## 总结

*   • RocketMQ 和 kafka 相比，在架构上做了减法，在功能上做了加法
    
*   • 跟 kafka 的架构相比，RocketMQ 简化了协调节点和分区以及备份模型。同时增强了消息过滤、消息回溯和事务能力，加入了延迟队列，死信队列等新特性。
    
*   • 凡事皆有代价，RocketMQ 牺牲了一部分性能，换取了比 kafka 更强大的功能特性。
    

**推荐阅读：**

[原来 8 张图，就可以搞懂「零拷贝」了](http://mp.weixin.qq.com/s?__biz=MzUxODAzNDg4NQ==&mid=2247485624&idx=1&sn=248eca4d8dd214126fb89d75acb5f34e&chksm=f98e4c12cef9c5048810530b04d6f58cd449649ffd1372d1eef8f3c3b1592598579ef15f45e0&scene=21#wechat_redirect)

[面试官：你的项目为什么要用消息队列？](http://mp.weixin.qq.com/s?__biz=MzUxODAzNDg4NQ==&mid=2247529456&idx=2&sn=b08e8fdd2102cd867129bce96fa8a7f7&chksm=f98d375acefabe4cabf18f98cbef24689e604ec19cdbfea8762b56495eb043dfb7dd0997b71f&scene=21#wechat_redirect)  

[面试官：你说说 Kafka 为什么是高性能的？](http://mp.weixin.qq.com/s?__biz=MzUxODAzNDg4NQ==&mid=2247534717&idx=2&sn=804859d964b7415325c7fe265692a04a&chksm=f98d0cd7cefa85c11b29ef8715e9e4d681929e01bc6d3bc12d8524dd073f8a544804327acc61&scene=21#wechat_redirect)