---
source: https://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247489429&idx=1&sn=0f7c0c2062c56abe6fb69944b6d48fa3&scene=21&poc_token=HJCqAWmjZKExMtpW-60h7KS1XPpWBW50Lkpxqvhh
create: 2025-10-29 13:48
read: true
knowledge: true
knowledge-date: 2025-10-29
tags:
  - 系统架构
summary: "[[得物客服 IM 消息通信 SDK 经验]]"
---
## **一、背景**

客服 IM 的核心业务就是在线沟通，客服与用户通过实时沟通的方式可以在最短的时间内帮助用户解决问题。初期为了快速支撑业务需求，便基于第三方 SDK 进行了二次开发，同时也埋下了问题定位困难，特殊功能实现成本高等隐患。随着公司业务的快速发展，客服对 IM 聊天的性能和体验都有了更高的要求，第三方 SDK 消息通信逐渐遇到了瓶颈，为解决第三方 SDK 接入带来的潜在隐患、提升 IM 的稳定性和高扩展性，自研一套可控、稳定、灵活的 IM 系统已是无法避开的一条道路了。以下主要是以客服端 (web) 为主。

## **二、思考**

客服与用户在聊天过程中，直观上是客服在**输入文案，然后通过****网络****发送给用户**，但是 SDK 该**如何****设计****才能使客服在发送消息过程中感知不到卡顿**，这一点是非常关键的，要**避免卡顿就要设计合理的发送策略以及避免大量** **JS** **脚本执行**，举个客服与用户聊天的例子：

*   客服发送了 “**客服小冰为您服务**” 这个文案，通过业务侧调用 SDK 的接口，传入到 SDK 里，SDK 会先创建消息体，即把这个字符串**封装成一个自定义的结构体 model**；
    
*   再将该数据存储到数据池中，**序列化后把这个数据对象 data 传递给 socket 接口**，通过网络通道发送到网关；
    
*   网关侧接收到消息后，**再反序列化，传递到数据池中进行处理，组装成业务可识别的 model**，推送到业务侧使用。
    

其聊天流程如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AyIMpnZEFomz0CL4CFSJphWNMbfQxIVKr5jOVWBMFrAw3SjeAedLva9TytgqE3ckpqiasA4pe6Bfw/640?wx_fmt=png#imgIndex=0)

如上图所示，可以清晰的看出消息发送和接收的流程链路。**如果** **SDK** **设计****不合理，发送消息和接收消息流程出现了卡顿，将直接影响用户的****体验****。**

## **三、自研框架架构图**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AyIMpnZEFomz0CL4CFSJphZoTvRXk8ictsLUmkqicdMibdrbSIiavb0UMhshE9FSjjONRC5pxb2ZC6UQ/640?wx_fmt=png#imgIndex=1)

整体的技术改造有两个方面：

*   **对消息链路的抽象改造：**主要是消息数据存储和消息排序的重构。
    
*   **业务接入侧的抽象改造：**主要是将业务逻辑和 SDK 源码进行解耦，做到代码分层更加的清晰。
    

## **四、消息链路发布订阅实现**

在 SDK 自研开发过程中，如何解耦框架代码和业务代码，做到灵活的消息监听，前期调研之后使用了 RxJS，这里简单介绍几个 RxJS 的核心概念：

*   **Observable(可观察对象)：**表示一个可调用的未来值或事件的集合。
    
*   **Observer(观察者)：**监听由 Observable 提供的值。
    
*   **Subscription (订阅)：**表示 Observable 的执行。Subscription 有一个重要的方法，即 unsubscribe，它不需要任何参数，只是用来清理由 Subscription 占用的资源主要用于取消 Observable 的执行。
    

SDK 底层在接收到数据后需要同步到业务侧，之前的做法是通过监听方式实现，这种方式不具备取消订阅的能力，维护成本相对较高。而使用 RxJS 可以清晰的梳理出数据流向，通过发布订阅的方式实现数据的通信。RxJS 在发布订阅的实现流程如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AyIMpnZEFomz0CL4CFSJphXIB8dBrEYRQwT06BdcsImxQ9nIibtasQE0PLagfCp4PCzeJBeOPc5Rw/640?wx_fmt=png#imgIndex=2)

从上图可以看到**消息处理的整个流向非常清晰，框架底层接收消息，订阅者消费消息。**

## **五、消息框架的分层实现**

在整个 IM 消息通信框架中，主要有三层结构：**网络****层、数据链路层和应用层**，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AyIMpnZEFomz0CL4CFSJphhzsdB7rxoktZYlZFYQsEsq2DjvrubyRBiaY1LQLbM2FXcjX0yoQUOIw/640?wx_fmt=png#imgIndex=3)

### **1、网络层**

网络层作为消息发送的最底层，**负责 TCP 的连接，消息发送 & 接收**，网络协议我们选择的是 TCP 协议，我们为什么没有选择 UDP 呢？因为 UDP 是无连接的，不够安全，无法提供可靠传输的服务，**通过 TCP 连接传送的数据可以无差别、不丢失、不重复且按序到达。**

整个 SDK 的通信方式我们采用的是 **Websocket + Json、grpc + protobuf**，第一步我们要做的就是建立 Websocket 连接，代码层面我们会先创建一个 Connection 的抽象类，主要处理网络连接相关配置、超时后重新连接的补偿实现，和一些继承类需要实现的抽象方法。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AyIMpnZEFomz0CL4CFSJphVFb0ZoJicrmicUpibibW6g6GUSm4YQGiblnl9hlxIXaBQicBpOaN5so17tdw/640?wx_fmt=png#imgIndex=4)

如上述代码所示，**核心在处理超时重连**，传统的重试策略是每隔一段时间重试一次，由于是固定的时间间隔重试，重试时又会有大量的请求在同一时刻涌入，会不断地造成限流。这里使用了指数退避的方式，**指数退避是一种通过反馈，成倍地降低某个过程的速率，以逐渐找到合适速率的算法**，可根据时隙和重试尝试次数来决定延迟重试，其实现算法大致如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AyIMpnZEFomz0CL4CFSJphxEKDettocKzh5ZBO2GacdFV1kRuzNPpuSO08l9ibAwLY4AKeXiaWCiczQ/640?wx_fmt=png#imgIndex=5)

Websocket 的连接我们是通过继承 Connect 类实现的，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AyIMpnZEFomz0CL4CFSJph15EBbyOFK3QDNicWSicGx7dN8S70O5aFffocuEM1vI2Jx0CrTBJ1gR9g/640?wx_fmt=png#imgIndex=6)

至此网络层连接就已完成了，相对比较简单，都是一些 socket api 的封装，核心的点在用指数退避算法实现消息发送失败重连接。

### **2、数据链路层**

数据链路层是 SDK 的核心层，**主要涉及到用户信息、消息、数据池等等**，我们来一步步对每个模块进行分析。首先梳理一下客服在登录到用户进线发送消息和接收消息的全过程，过程有如下几个阶段：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AyIMpnZEFomz0CL4CFSJphkQheyAx3MZeQaxkH71eXEiaibQOXG87t0EECqWicxrMHUOT2v5zUMZvdw/640?wx_fmt=png#imgIndex=7)

#### **2.1 协议类型**

消息协议类型非常重要，是消息发送的基石。初始化协议数据体，可以用于后续各种消息、事件的发送。在 IM 自研的 SDK 通信协议类型主要有如下几种：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AyIMpnZEFomz0CL4CFSJphj8Eu14ZEe8kuPHvV2PSiaE0XG6aaIVtuZ66LXt7NLptNiaic0G81owJcg/640?wx_fmt=png#imgIndex=8)

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AyIMpnZEFomz0CL4CFSJphzCmDlIdSRiarjVscfxeaAbylpYxPTPa2br6RTWdeictOo4llSibwDBhQQ/640?wx_fmt=png#imgIndex=9)

*   **Hi**：发送客户端基础信息，告诉 server 当前 client 的版本、设备类型、语言等信息
    
*   **Login:** 登录，token 验证，获取或创建当前用户 topic 信息
    
*   **Sub:** 订阅 topic 或更新 topic 数据
    
*   **Leave**: 取消订阅，解绑之前的订阅关系
    
*   **Pub:** 发送数据消息给指定 topic 的订阅者
    
*   **Get**: 获取 topic 的 metadata 信息，例如：获取订阅者列表、历史数据等
    
*   **Set:** 更新 topic 的 metadata 信息，例如：删除消息或删除 topic
    
*   **Del**: 用于删除操作，包括删除消息、删除订阅关系、删除 topic 等
    
*   **Note**: client 发送通知给 topic 的订阅者，例如消息已收到，消息已读，当前正在输入等
    
*   **Action**: 触发的事件，例如：切换客服状态、获取机器人问题等
    
*   **Datares**: ack 机制，告诉网关已收到该消息
    

#### **2.2 创建连接**

对网络层消息链接实例化，实现消息的正常发送和接收，其实现如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AyIMpnZEFomz0CL4CFSJph19oCd1Lfy9wdrctubxVSwicbvfJlhck1tdcJ6kfBCmCOKc3qNL9zCJg/640?wx_fmt=png#imgIndex=10)

#### **2.3 消息定义**

客服要发送一条消息，肯定有对应的消息结构体 model，即需要对消息体进行设计，这里会设计一下 message 类，每次创建新的消息体都会 new 一个实例，通过对实例的操作可以更新消息状态等，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AyIMpnZEFomz0CL4CFSJphkCMjZiaYemnO0EujQXUOeue2rPgdoRn7Zk8TdxVBicIa4oOBw5H33GAw/640?wx_fmt=png#imgIndex=11)

针对单个消息，我们也要定义好消息状态，用于聊天过程中消息状态的更新，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AyIMpnZEFomz0CL4CFSJph3UDI5kykFpCu9SDe34OluEBDTrV0cuKNye4GFvZnkuA2C1GJWQNFRA/640?wx_fmt=png#imgIndex=12)

#### **2.4 数据池**

消息类创建好之后，就需要有消息数据池来存储，消息池结构定义如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AyIMpnZEFomz0CL4CFSJphR7wSWlofdUGWiccX7x9NqeyuKs8Af2wB2DPJ0oAQZS7Y8WT3kD6GVLA/640?wx_fmt=png#imgIndex=13)

这里还涉及到消息体的一些基本操作方法对数据池中的数据进行操作，就不做过多的阐述。

#### **2.5 用户维度**

上面都是在分析公共模块，但是客服和用户是一对多的关系存在，还需要设计一个用户维度模块，后续在业务侧的操作基本都是以用户维度来操作，需要从单个用户维度设计对应的订阅关系、消息发送、删除等等。其实现大致如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AyIMpnZEFomz0CL4CFSJphkmHUsXSTvP85xkfp12OBLWcdMiaBeSlWu1WFonXIZZyrTaaIxSfPayQ/640?wx_fmt=png#imgIndex=14)

##### **2.5.1 发送消息链路分析**

针对客服发送消息，我们**首先要站在客服角度考虑消息是否已发出去，优先展示的聊天页面**，而不是等网关给了回复后在展示到聊天页面，根据已往经验来看，**只要回车消息就要立即展示到聊天页面，否则客服会认为出现了卡顿，****体验****效果不佳**，鉴于这种场景的需求，在设计发送消息链路的时候就要充分考虑到这一点，所以设计流程如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AyIMpnZEFomz0CL4CFSJphog0YuRyMmV0HoAWLkUcojEUgurtF2xuBKvv6zqCa8kTwqZCecLJPrg/640?wx_fmt=png#imgIndex=15)

如上图所示，先在 SDK 内进行处理对应的消息，**处理完成后返回到业务侧完成渲染后再进行消息发送****到****网关**，正常情况下都在一帧之内，客服是感知不到有延迟的，这里要**关注消息体序列化、反序列化的时机，避免无谓的性能浪费**。上述图中有个**虚拟 seq，主要是为了在未收到** **IM 网关****响应之前进行排序用的**，图片、视频、断网发送消息、消息发送失败，或收到 IM 网关回复缺少 seq(场景：敏感词) 等情况都需要通过虚拟 seq 进行准确排序。

##### **2.5.2 接收消息链路分析**

接收消息过程相对比较简单，**收到消息进行反序列化后更新相关数据**，然后在数据池中完成去重 (重试机制)、排序后更新到业务侧渲染即可。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AyIMpnZEFomz0CL4CFSJphlcLGdGiban38ricR7EeFZaPdicSkK8niaGKO8ZFRLzn7jt7PBcj6rotewQ/640?wx_fmt=png#imgIndex=16)

##### **2.5.3 消息的可靠传递**

IM 消息的可靠投递主要是指：**消息在发送接收过程中，能够做到不丢消息、消息不重复、消息顺序不错乱。**我们先来分析以下 2 种情况：

*   第一种：如果客服 A 在把消息发送到 IM 网关的过程中，由于网络不通等原因失败了；或者 IM 网关接收到消息进行存储时失败了；或者 IM 网关一直没有返回结果，导致超时，这些情况客服 A 都会被提示消息发送失败。
    

*   第二种：消息在 IM 网关存储完后，客服 A 被告知消息发送成功了，然后 IM 网关把消息推送给用户 A 的在线设备。在推送的准备阶段或者把消息写入到内存后，如果服务端出现掉电，也会导致消息不能成功推送给用户 A。如果用户 A 的设备在接收到消息，在后续处理过程中出现问题，也会导致消息丢失。比如：用户 A 的设备在把消息写入本地 DB 时，出现异常导致落库失败，这种情况下，由于网络层面实际上已经成功传输，但用户 A 却看不到消息。我们客服 IM 对于消息丢失的处理方案如下：**参考 TCP 协议的 ACK 机制，实现一套基于业务层的 ACK 协议。**
    

添加 ACK 之前消息发送的时序图如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AyIMpnZEFomz0CL4CFSJph7xINXQDwHWD7Bbf9TDxWH4kYl8Gu2rDmfibmGH8icd6bI6q85InaXMoA/640?wx_fmt=png#imgIndex=17)

- ACK 机制 -

在 TCP 协议中，默认提供了 ACK 机制，通过一个协议自带的标准的 ACK 数据包，来对通信方接收的数据进行确认，告知通信发送方已确认成功接收了数据。ACK 机制也是类似，**需要解决的是：****IM 网关****推送后如何确认消息是否成功送达接收方并明确被接收方所接收。**具体实现的时序图如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AyIMpnZEFomz0CL4CFSJph0aILdt2TiaWicopaZ3xfoLQd0pfavMEBFEJk8ZickiaklCibIYL3D74OPoA/640?wx_fmt=png#imgIndex=18)

客服或用户在发送消息的过程中都会携带一个 msgid(32 位的 uuid，类似 TCP 的 sequenceId)，IM 网关在接收到消息后，会根据 msgid 到数据库中查询是否存在该条消息，如果存在就不落库，如果不存在就落库，然后再推送到接收方，接收方在收到消息后会回复 ACK，ACK 包中会携带上当前最新的 seqid，IM 网关收到 ACK 回复后会对最大的 seqid 进行更新。**这里为什么要更新最大 seqid 呢？有什么用呢？**这么设计肯定有一定道理的，IM 网关在收到发送方发送的消息后除了到数据库中检测该消息是否存在外，还会对比当前接收到消息的 seq 和最大 seqid 两者之间的差值，会把 [seq, seqid) 之间的数据全部推到接收方，正常情况下都是 [n, n-1)，如果 IM 网关没有收到接收方 ACK，n-1 就不会更新，推送的消息个数就大于 1 了。如果 seq 和 seqid 相等那就是发送方重复推送的消息，这个时候就不会向接收方推送。这里就涉及到了消息重试，继续向下分析吧。

- ACK 机制中的消息重试 -

消息推给 A 的过程中丢失了怎么办？比如:

*   A 网络实际已经不可达，但 IM 网关还没有感知到 (ping 出现问题)；
    
*   消息在中间网络途中被某些中间设备丢掉了。
    

解决这个问题也是**参考了 TCP 协议的重传机制**。我们会在客服端、IM 网关、用户端都维护一个超时计时器，一定时间内如果没有收到对方回的 ACK 包，会重新取出该消息进行重推。在重试一定次数后，如果还是没有收到 ACK，视为放弃。前端代码结构和效果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AyIMpnZEFomz0CL4CFSJphCgyicRKdv1DqwNuMqMTmD7oOW4e7Zy2EPLmJ2yVfMhDUg3NCYcblA5A/640?wx_fmt=png#imgIndex=19)

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AyIMpnZEFomz0CL4CFSJphZSpMpBs3v89iamTknlqqnyfJZ2GSwsIhx8Hsu56T9zdXx3nu7YsNPqQ/640?wx_fmt=png#imgIndex=20)

上述图片中的数据只是模拟消息重试，真实场景中执行频次肯定要比这个时间更久一些。

- 消息重复推送的问题 -

如果在一定时间内没有收到 ACK 包，就会触发重试机制。收不到 ACK 的情况有两种，除了推送的消息真正丢失导致 A 不回 ACK 外，还可能是 A 回的 ACK 包本身丢了。

**解决方案是：**发送方在发送消息时携带一个 msgid，msgid 是全局唯一的，针对同一条重推的消息 msgid 不变，接收方根据这个唯一的 msgid 进行去重，这样经过去重后，对于 A 来说，在聊天界面是不会看到重复的消息，不影响使用体验。

- 保证消息不会乱序 -

消息的一致性是非常重要的，在聊天过程中消息顺序不能错乱。

*   以发送方的本地时间戳为序号，但是这样有比较大的问题，发送方的时间戳是可以被改动的，这种方式不可取；
    
*   IM 网关服务是集群部署，会通过 topic 和 seqid 做为唯一索引，在接收到消息落库之前会生成 seqid，客服端和用户端接收到发送消息的回执时需要根据返回的 seqid(IM 网关自增) 进行消息排序，这种方式可取。
    

通过以上的分析，**客服 IM** **消息的可靠性就是**通过 **ACK 机制，重试机制，去重机制，排序机制来确保每一条消息的完整触达和准确排序**。

#### **3、应用层**

业务侧使用的时候**直接实例化** **SDK** **即可**，在消息链路发布订阅中已经提到了 RxJS，此时在业务侧订阅使用即可。需要注意的是在实例化 SDK 的时候传递了一个 filterMsgItem 方法，主要是为特殊业务场景提供使用的，就拿我们客服业务来说，有些特定消息是不需要展示到聊天页面的，比如：用户发送消息被篡改等，当然我们在业务侧重新对数据过滤或者渲染的时候也是可以做过滤的，这样操作是没什么问题，但是没有必要，如果不从源头过滤数据，后续参与二分、倒序查找的源数据也会增加。会有一些不必要的浪费。当然也可以不添加这个参数，SDK 都是全兼容的。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AyIMpnZEFomz0CL4CFSJphIHPtULZ4nEul1ocdOkv0xhkmdYMYgMrNH3uTEefDJOdYh9O0ibbs77A/640?wx_fmt=png#imgIndex=21)

至此我们就完成了整个 SDK 的实现以及在业务侧的使用，消息发送和接收也都正常，效果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AyIMpnZEFomz0CL4CFSJpheP3Q8RX222SXvw8WmQI0Ikw2GYdmgRgMF9MUfHJSicsfR0U4DjlhmDw/640?wx_fmt=png#imgIndex=22)

### **六、总结**

自研 SDK 还是蛮有挑战的一件事情，从单纯的基于第三方 SDK 二次开发到自研 SDK 并与我们的实际业务场景相对完美的结合。**在 SDK 的整体****设计****以及和业务侧如何更完美的结合并不是一蹴而就的，都是在实际业务场景中不断积累经验**，不断尝试才找到相对完美的解决方案。这里列举一个简单的案例吧，例如消息发送：**需要考虑到断网场景下该如何进行消息****显示****、排序、重新发送？**发送失败的场景下重新发送再次失败后又该如何显示、排序？弱网场景下发送消息触发重试机制该如何以最优的方式去重、排序？发送消息触发敏感词该如何处理？断网重连后对于发送失败和触发敏感词的消息又该如何处理？如果在涉及到文件又该如何处理？... 在自研过程中除了关注业务场景外，还**调研****了行业内比较好的一些 web 应用在某些特殊场景的处理方式****。**很多优秀的方案也都**只能是借鉴一些核心思想，还是要以业务为核心，真正通过技术手段解决业务痛点才是最重要的**。

自研 SDK 收益还是非常大的，也积累了很多 IM 方面的经验，完成自研 SDK 也只是一个开始，后续我们将会在耗时任务、数据安全等方面持续深耕细作。

### 参考文档：

*   RxJS
    

*   TCP/UDP 协议
    

*   指数补偿（Exponential backoff）在网络请求中的应用
    

* 文 / 王卫强

 关注得物技术，每周一三五晚 18:30 更新技术干货  
要是觉得文章对你有帮助的话，欢迎评论转发点赞～

**活动推荐**

主题：得物技术沙龙 - 算法专场  

时间：7 月 30 日 13:50-18:00

报名方式：