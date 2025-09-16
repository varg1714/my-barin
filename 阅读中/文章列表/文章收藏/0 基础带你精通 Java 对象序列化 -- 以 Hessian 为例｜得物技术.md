---
source: https://mp.weixin.qq.com/s/gPJgrzyJCdok5x7F0Ls12A
create: 2025-09-11 21:10
read: true
knowledge: true
knowledge-date: 2025-09-11
tags:
  - Java
  - 数据结构
  - 分布式
summary: "[[Hessian 序列化技术分析]]"
---

![](http://mmbiz.qpic.cn/mmbiz_gif/AAQtmjCc74DZeqm2Rc4qc7ocVLZVd8FOASKicbMfKsaziasqIDXGPt8yR8anxPO3NCF4a4DkYCACam4oNAOBmSbA/640?wx_fmt=gif&wxfrom=5&wx_lazy=1#imgIndex=0)

**目录**

一、概述

二、基础编码原理

 1. 对象图遍历
 2. 编码格式

三、Hessian 编码格式

 1. 数据块
 2. 数据块标签 (tag)
 3. POJO 编码

四、Hessian 编码细节

 1. 重复对象复用
 2. 小整数内联 (direct)
 3. 字符串编码
 4. 整数压缩

五、总结

**一**

**概述**

在高级编程语言的世界中，开发者始终与**【object/struct】**这类高度抽象的数据结构打交道。然而在分布式架构下，任何服务进程都不是数据孤岛——跨进程数据交换是必然需求。

以 Java 为例，业务逻辑的输入输出都是**【object】**。但在 RPC 场景中，这些对象必须经由网络传输。这里出现了一个根本性矛盾：网络介质 (网线 / 光纤) 对面向对象编程 (OOP) 一无所知，它们只会用光和电忠实地传输扁平化的字节流 (**byte[]**)。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQrQQREKpf7LUHGLrKibYeMXIcHmtWNchzia17z9lkXZsMVkeqqJ3SlWug/640?wx_fmt=png&from=appmsg#imgIndex=1)

软件工程经典的分层理论驱使我们去添加一个转换层。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQgnjoUoTLnwawplv5UoSk167p06ibMKJwibNrd7ia3c1PoD1Roa8ic8L9Mw/640?wx_fmt=png&from=appmsg#imgIndex=2)

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQUaaGzp6kOBFtjrcbSD8vVVgYUZ4rRZeUZa0kypw3dFBvnF6mNxNPwA/640?wx_fmt=png&from=appmsg#imgIndex=3)

我们需要有个工具或者组件来协助进行**【object】**和**【byte[]】**之间的双向转换。这个过程包含两个对称的流程：

1.  【**object**】->【**byte[]**】：业界一般称为序列化 /serialize，但是那个单词念起来很拗口，本文我们都叫它【编码 /encode】好了。
    
2.  【**byte[]**】->【**object**】：业界一般称为反序列化 /deserialize，但是那个单词念起来很拗口，本文我们都叫它【解码 /decode】好了。

Hessian 作为 Java 生态中久经考验的对象编解码器，相较于同类产品具有以下两大核心优势：

1.  深度 Java 生态适配：与 JSON、Protobuf 等语言中立的通用协议不同，Hessian 专为 Java 深度优化，对泛型、多态等 **Java 特有语言特性提供原生支持**。
    
2.  高效二进制协议：相较 JSON 等文本协议，Hessian 采用精心设计的二进制编码方案，在编解码效率和数据压缩率方面表现更优。

需要强调的是，软件工程没有银弹——业务场景的差异决定了编解码器的选择必然需要权衡取舍。但就 Java RPC 而言，Hessian 应该是经过广泛实践验证的稳健选择。

本文将系统解析 Hessian 的编码流程，重点揭示其实现【**object**】->【**byte[]**】转换的核心机制。

**二**

**基础编码原理**

对象编码过程主要包含如下两大核心：

*   **对象图遍历**：遍历高级数据结构
    *   通过反射或元编程技术遍历对象图（Object Graph）。
        
    *   是同类产品的通用逻辑，不管 jackson、fastjson、hessian 都需要用不同的方式做类似的事情。
*   **编码格式**：将高级数据结构按协议拍平放到 **byte[]**
    *   同类产品百家争鸣，各有各的思路。
        
    *   是同类产品的竞技场，各个产品在这里体现差异化的竞争力。
    
*   设计权衡包括：
    *   二进制效率 vs 可读性（如 Hessian 二进制 vs JSON 文本）
        
    *   编码紧凑性 vs 扩展灵活性
        
    *   跨语言支持 vs 语言特性深度优化

对象图遍历决定了编码能力的下限 (能否正确处理对象结构)，而编码格式决定了编码能力的上限 (传输效率、兼容性等)。 

**对象图遍历**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQglSLwdLPouEib52mTH0s8DMTNn9XM5rRZL5xYmVcGPZpMVfjsxSspEQ/640?wx_fmt=png&from=appmsg#imgIndex=4)

对象图遍历的本质是按深度优先进行对象属性导航。

举个例子：

![](https://mmbiz.qpic.cn/mmbiz_jpg/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQtwNHHPTyChfqPEguArbYQPUL0ibH1ibVsWiaWaBwcxKJ6OlYlDEckyJMQ/640?wx_fmt=jpeg&from=appmsg#imgIndex=5)

宏观来看，A 类型的对象其实是一棵树 (或图)，如果脑补不出来的话，我给你画个图：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQLg1P2xol6KtuTJdNRrZrjaaJqC6KWEicw58e8cPeiakOwfqWr0TO8WYQ/640?wx_fmt=png&from=appmsg#imgIndex=6)

可以看到这棵树的叶子结点都一定是 Java 内置的基本数据类型。换句话说，Java 的 8 种基础数据类型和他们的数组变体，支撑了 Java 丰富的预定义 / 自定义数据结构。

_八股文：Java 的 8 种基础数据类型是哪些？String 算不算基础数据类型？_

编码的本质就是深度优先的遍历这棵树，拍平它，然后放到 **byte[]** 里。

我举个例子吧。

**伪代码**

为降低伪代码复杂度，我们假设 Java 只有 1 种基础数据类型 int，也就是说 Java 里只有 int 和只包含 int 字段的自定义 POJO。

_我们定义 POJO 指的是用于传输、存储使用的简单 Java Bean 或者常说的 DTO。_

_从某种意义上来说，Integer 也是基于 int 封装的自定义 POJO。_

**字节流抽象**

我们使用标准库里的 java.io.DataOutput 来进行伪代码说理，这个类提供了一些语义化的编码 function。

![](https://mmbiz.qpic.cn/mmbiz_jpg/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQqTc2Hj0V1Ujr2jicLIj4wSvOfYPHwd1Hhkd4icmrhlEoIMiaVkrIFiaERQ/640?wx_fmt=jpeg&from=appmsg#imgIndex=7)

_java.io.DataOutput_

**对象图遍历**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQgg4oPOQvrLtb5RMnMd0VTnSo86b0pdGuibxR10P55Ud6UUWg5aJVevw/640?wx_fmt=png&from=appmsg#imgIndex=8)

**字节流布局**

最终呈现出来的字节流层面的数据布局会是这样：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQX4IEibToIm2s3vNbEAib1SISeGjricttO1dulDH6lM2zrbFlxhZzn0iaOg/640?wx_fmt=png&from=appmsg#imgIndex=9)

看起来没毛病，唯一的问题就是不好解码。

当解码端收到一个 16 字节的字节流以后，它分不清哪块数据是 A 对象的，哪块数据是 B 对象的。甚至都分不清这到底是 4 个 int32 还是 2 个 int64。

这个问题需要编码格式来解决。

**编码格式**

上面遗留的问题，聪明的你肯定想到了答案。

就是因为编码产物太太太简陋了，整个过程中只是一股脑的把树拍平，把叶子节点的值写入字节流，缺少结构元数据。

最最最重要的结构元数据就是数据块的边界，上述 4 个数据块，最起码应该添加 3 个边界标识。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQU7guw2jFPEMlniassIddS59Lia7cQrZE1wINLphRKPIpTEBJZN8jYMtw/640?wx_fmt=png&from=appmsg#imgIndex=10)

我们先用我们耳熟能详的 JSON 格式来理解下编码格式这个事情。

**伪代码**

JSON 是这样解决这个问题的：

![](https://mmbiz.qpic.cn/mmbiz_jpg/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQtyPpbWCGpPcavlq0hrA61rhaPgRjoD1rfdNZFargoIibC4bTUTDlJ4w/640?wx_fmt=jpeg&from=appmsg#imgIndex=11)

JSON 协议在嵌套的 POJO 上用 {} 来作为边界，POJO 内部的字段键值用 , 来做边界，: 拆分字段键值。

**字节流布局**

结果就变成这样：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQRPic2iarpwmia6jt5nRuSn6AxjzrEtvz9I6DV3DqansOU7dyybWIibo9Ag/640?wx_fmt=png&from=appmsg#imgIndex=12)

这样在解码的时候，可以通过 {、}、,、: 等 token 来切割 JSON 字符串，判定数据块边界并恢复出对象图。

**三**

**Hessian 编码格式**

接下来我们可以开始介绍 Hessian 的编码魔法了。

需要强调的是：Hessian 跟 JSON 不同，Hessian 是二进制格式。如果一个字节流直接按字符集解码不能得到一个完整的、有意义的字符串，那它就是二进制编码数据。

Hessian 在编码时，按数据块类型为每一个数据块添加一个前缀字节 (byte) 作为结构元数据，这些元数据和数据块一起，交给解码端使用。

**数据块**

对象图里的每一个节点，都是一个数据块。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQtYlX569IYEOyheFb4L7QQv5ictDNouH23JibZicuR8iciacxZUuMm6VfiawA/640?wx_fmt=png&from=appmsg#imgIndex=13)

如上图所示，以 A 对象为根的对象图，一共有 6 个数据块。

**数据块标签（tag）**

Hessain 在编码每一个数据块时，都会根据数据块的类型在字节流中写入一个前缀字节 (**0-255**)，这个字节说明了数据块的语义和结构。

以 **int32** 为例，其最基础的编码格式如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQWdIGy7OibFhWVLUSh82tNt6MGJr5Xicmicc4c2EIVFiaib3aJFNnvof12ibA/640?wx_fmt=png&from=appmsg#imgIndex=14)

_除该基础编码格式外，int32 的编码还有其他变体。_

上述 I 就是整数类型的 tag。解码端读取 tag 后，按 tag 值来解码数据。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQXP0H9iczBD2lXlQRZicuOO3SNgDdLIq5upOsKecO51jqoBefbCibVhqVw/640?wx_fmt=png&from=appmsg#imgIndex=15)

`com.alibaba.com.caucho.hessian.io.Hessian2Input[#readObject](javascript:;)(java.util.List<java.lang.Class<?>>)`

由此延伸、拓展，其他的数据类型都是类似的模式。常见数据类型及其对应的 tag 值如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQ2N0dwA2iahic8t6pyZriaNrhqicurWrxe7nXvFL5SXvN8mticS32NV7NWCg/640?wx_fmt=png&from=appmsg#imgIndex=16)

值得注意的是，N、F、T 三个 tag 是自解释的，和固定值映射、绑定。

**POJO 编码**

POJO 是一种特殊的数据块，Hessian 将 POJO 的结构和值拆开，分别编码。

**POJO 结构编码**

POJO 结构的 tag 为 C，对照 int32 的编码格式，POJO 结构的编码格式如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQE64tbAmsRQEqTAyhF0vmD2Y0INy7YP8IBEYCXDTPPzzXlqVJXD26Bw/640?wx_fmt=png&from=appmsg#imgIndex=17)

举个例子：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQo4XeUPiaEwn9gskKy4M2CEeJviaOKMF0m81vXribL0nFVQZiavhLvricxQg/640?wx_fmt=png&from=appmsg#imgIndex=18)

编码 POJO 时，Hessain 会将 POJO 的类名、字段名列表写入字节流，供解码端使用。后续编码 POJO 字段值时，需要按照字段名列表 (如上述 bb、cc) 的顺序来编码字段值。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQk2385mPeEudWYkiauob0sV7F0BMXlClteDTIrJrW0ibF3gzicGu0BAYow/640?wx_fmt=png&from=appmsg#imgIndex=19)

**POJO 字段值编码**

POJO 字段值的 tag 为 O，对照 int32 的编码格式，POJO 字段值的编码格式如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQ3ib4KibV2Luc7aMjic7jtgtURpgdWem9dpniacJ5JTb9t3LKkBSvQUTlxw/640?wx_fmt=png&from=appmsg#imgIndex=20)

举个例子：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQSKRtugdm36SIcM60K6QXf22xiawMAa3eCia6tYGHzrPmpZFTcE8ysFTQ/640?wx_fmt=png&from=appmsg#imgIndex=21)

可以看到，编码 POJO 字段值的时候，在 tag 后面有一个 POJO 结构序号。

这是 Hessian 的一个数据复用的小技巧。

**POJO 结构复用**

JSON 协议有一个缺点，那就是重复数据带来的存储 / 传输开销。举个例子：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQndhtuZOAhvAzHvOeYDnJxUlNV33AkIJWXDft6E3kEV2ICG4YOgAw0w/640?wx_fmt=png&from=appmsg#imgIndex=22)

如上图，B 类型的字段名 (dd、ee) 在编码产物中重复出现！

Hessian 希望解决这个问题，同一类型的多个 POJO 对象在序列化时，只需要在第一次的时候编码类名、字段名等元数据，后续可以被重复的引用、使用，无需重复编码。

如果用 Hessian 来编码，结果会是这样：

**数据布局**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQbu459UIOicBylofvm5L2myntXmC3HoWm4SnKiar4nwicoic2ACX189wyHA/640?wx_fmt=png&from=appmsg#imgIndex=23)

**数据布局详解**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQK4rsQBqOhbaCCd0Z5K8Bibc746yKdKoRSibOyUlxDGvZgBNAj91tkE8g/640?wx_fmt=png&from=appmsg#imgIndex=24)

如上图，APojo、BPojo 的字段名只会编码一次。多个 BPojo 对象在编码时会通过结构引用序号 (1) 来引用它。相对 JSON，Hessian 避免了多次编码 BPojo 字段名的开销。

为什么 APojo 的序号是 1、BPojo 的序号是 2？

Hessain 在编码过程中，每次遇到一个新的、没有处理过的新 POJO 类型时，会给它分配一个从 0 开始、单调递增的序号。

遥相呼应的，解码侧每次解码一个 tag 为 C 的 POJO 结构数据块时，也会按解码顺序维护好其索引序号。

**四**

**Hessian 编码细节**

到现在，我们已经对 Hessian 编码有了一个的概括性的认识，接下来我们来看看一些值得注意的细节。

**重复对象复用**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQibqYGT9G9KguyDE6kS6HZcqvF25HicIApfnLps5NU0rH2QdJl99zsjCw/640?wx_fmt=png&from=appmsg#imgIndex=25)

A 对象里有两个字段 (d、e) 指向同一个对象 B。如果不做处理，会因为重复编码而带来不必要的开销。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQSdQYfPrW05VPGPqPozcyqpgookibsmnBdmQc0ykcNUia4u8rLORLbPnQ/640?wx_fmt=png&from=appmsg#imgIndex=26)

相同的一个 B 对象，因为被两个字段重复引用，导致 2 次编码、产生 2 份数据空间占用！

如果只是有额外的开销，没有可用性问题那都还好。关键是在循环引用场景下，会因为引用成环导致递归进行对象图遍历时触发方法栈溢出！

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQeclEiaZNg80ibJcMtARwYQKCMbzksK77ibibVY1iaejJdicUr2UvuhclCickw/640?wx_fmt=png&from=appmsg#imgIndex=27)

循环引用是重复引用的特例，只要将重复引用处理掉，循环引用也就没问题了。

Hessian 通过对象引用来解决这个问题。在对象图遍历过程中，遇到一个之前没有遇到过、处理过的 POJO 对象时，会给它分配一个从 0 开始、单调递增的序号。

后续再次需要序列化相同的对象时，直接跳过编码流程，将这个对象的序号写入字节流。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQzueSWADaa1KkNjnaVSmzSwkia0Xct64IFQZfjYL1O04iccqm7eSPZsWQ/640?wx_fmt=png&from=appmsg#imgIndex=28)

解码时，解码侧按相同的顺序来恢复出引用序号表，解码后续的对象引用。

**小整数内联 (direct)**

很多编码类型，都需要在 tag 后再维护一个整数类型的字段。比如：

*   POJO 的编码 tag O 需要一个整数来引用 POJO 结构引用序号。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQibhsxVAtoGOpTSanAaTJVZRJb8uibvUDiasEC5l9lvrNZpxLYqibJ2W1nQ/640?wx_fmt=png&from=appmsg#imgIndex=29)

*   类似 String 的变长类型需要一个整数来标识变长数据的长度。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQqlXsSZaC0WVuBmb21Pricm4ej1xhkTXgicwp8psTmsI0At1iboG18jznQ/640?wx_fmt=png&from=appmsg#imgIndex=30)

当字符串很短，就比如 **"hi"** 吧，短字符串编码格式的长度字段可能比实际字符数据还大（用 4 字节存储长度 2），效率低下。

**tag 分段**

**Hessian 将一些 tag 值的语义富化，让它既体现数据类型，也体现小数值。**

因为 tag 是一个 **byte(int8)**，取值范围是 **0-255**，每个 tag 标识一种特定的数据类型 (int、boolean 等)，但是这些数据类型最多几十种，取值范围内还有很大的数值区间没有被使用，其实比较浪费。那我们就可以把这些空闲的 tag 值，挪作他用，提升 tag 数值空间利用率。

我举个例子，注意这个是参考 Hessian 思路的一个简单示意，**具体的 Tag 值和 Hessian 无关**。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQ8iapnj0U1gJSf1W20H4pJFbs5oBFJ3Z1O9Wf0ib3ZoYiafjQjN88ywgHQ/640?wx_fmt=png&from=appmsg#imgIndex=31)

**长度内联**

对于长度≤31 的字符串，Hessian 用 tag 同时编码类型和长度。

1.  当 0 <= tag <= 31 时，标识后续的数据块为字符串。
    
2.  tag 的数值即为后续数据块的长度。

示例如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQic5iaIKjMLZqCicWILMS6eicLr3lWKtbZiaGzvz4ChicYHicuWhbrgIu6P0dQ/640?wx_fmt=png&from=appmsg#imgIndex=32)

**序号内联**

当结构引用**序号 <=16** 时，Hessain 用 tag 同时编码类型和序号。

1. 当 0x60 <= tag <= 0x70 时，标识后续的数据块为 POJO 字段值。
2. tag - 0x60 的值，即为 POJO 结构 (类名 + 字段名) 引用序号。

示例如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQQPoAkIyHM3xApRffZnPsv1LicNSdrTDOVTM6KhVeWAk4VFTuewxtRXA/640?wx_fmt=png&from=appmsg#imgIndex=33)

相关源码如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQe6sUUBlGyUlLb4VRR5ibsvaib3jCEeyicO2HxbAbAGDLiaLAibgVNrCY4ibg/640?wx_fmt=png&from=appmsg#imgIndex=34)

`_com.alibaba.com.caucho.hessian.io.Hessian2Output[#writeObjectBegin](javascript:;)_`

**字符串编码**

Hessian 编码字符串的关键流程是：**字符串分段 + 不同长度的子串使用不同的 tag。**

*   分段原则

字符串会被分割为若干块，每块最大长度为 32768（0x8000）。前 N-1 块均为完整长度的子串（32768 字节），使用固定 tag **R** 标识；最后一块为剩余部分，长度范围为 0-32768 字节，根据实际长度选择动态 tag。

*   尾段 tag 的选择基于尾块的长度决定
*   长度≤31（0x1F）：使用单字节 tag 0x00-0x1F 直接内联长度值。
    
*   32≤长度≤1023（0x3FF）：使用 tag 0 后跟 1 字节长度 (大端序)，10bit 的计数空间由 tag 字节和长度字节共同提供。这个地方有点绕，看下代码吧。
    
*   长度≥1024：使用 tag S 后跟 2 字节长度（大端序）。
*   相关源码

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQzKVUbM9LDYsXicvQNKk9I6RduNKjco59Kl9jRW1V7fibfZGTOncbIia0A/640?wx_fmt=png&from=appmsg#imgIndex=35)

`_com.alibaba.com.caucho.hessian.io.Hessian2Output[#writeString](javascript:;)(java.lang.String)_`

这种设计通过减少长字符串的冗余长度标记，在保持兼容性的同时显著提升了编码效率。

**整数压缩**

**基础编码**

整数 (int32) 的的取值范围很大(-23^31 - 2^31)，保守的编码格式会用 4 个 byte 来编码整数。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQXrlkUicTaSHlTsOPv5cT6QZokpMAUcF7mI5ZwiaL47nG3jt2cfY2VRFQ/640?wx_fmt=png&from=appmsg#imgIndex=36)

但是日常使用中，我们会大量使用小整数，比如 1、31。这时候如果还用 4 字节编码就很不划算啦~

**变长编码**

Hessian 根据整数的值范围，动态的选择不同的编码方式，且不同的编码方式有不同的 tag：

*   **单字节整数编码**：类似【长度压缩】，tag 中直接内联数值

适用范围：-16 到 47（共 64 个值）

编码方式：使用单字节，值为 value + 0x90（144）

例如：0 编码为 0x90，-1 编码为 0x8f，47 编码为 0xbf

*   **双字节整数编码**

适用范围：-2048 到 2047

编码方式：首字节为 0xc8 + (value>> 8)，后跟一个字节存储 value 剩下的 bit。

这种编码可以表示 12bit 有符号整数

*   **三字节整数编码**

适用范围：-262144 到 262143

编码方式：首字节为 0xd4 + (value>> 16)，后跟两个字节存储 value 的高 8 位和低 8 位。

这种编码可以表示 19bit 有符号整数。

*   **五字节整数编码**

适用范围：超出上述范围的所有 32 位整数

编码方式：以 'I'（0x49）开头，后跟 4 个字节表示完整的 32 位整数值。

*   相关源码：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQB0OLjQPmIfEke4s3yEKgnrH1YM8m7xudrEH2dQGZiaHibUSxInndNoKg/640?wx_fmt=png&from=appmsg#imgIndex=37)

`_com.alibaba.com.caucho.hessian.io.Hessian2Output[#writeInt](javascript:;)_`

**收益**

*   小整数（如 0、-1）仅需 1 字节，而传统 int32 固定 4 字节。
*   大整数动态扩展，避免固定长度浪费（如 1000 仅需 2 字节）。

其他的数值类型比如 int64 也有类似的机制。

**五**

**总结**

Hessian 专为 Java 优化，采用高效二进制协议，通过对象图遍历和编码协议实现对象与字节流的转换，利用数据块标签、重复对象复用、数据压缩等机制，提升编解码效率和数据压缩率。

本文没有去展开 Hessian 的代码细节，而是尽可能深入浅出的介绍了 Hessain 的核心编码原理，以帮助读者建立对 Hessian 的宏观认知，从而可以更好的去理解和使用它。

尽管不同语言 / 生态的序列化框架选型让人眼花缭乱，但是各自需要解决的问题和解决问题的思路都大同小异；我们对 Hessain 原理的认识可以迁移到其他序列化框架，甚至自己写一个领域特定的序列化框架。

相关内容均为笔者走读源码整理而来，如有疏漏，欢迎指正。

**参考：**

*   **Hessian 2.0 Serialization Protocol**（_ http://hessian.caucho.com/doc/hessian-serialization.html_ ）
    
*   **Hessian 2.0 序列化协议（中文版）**_（__ https://www.diguage.com/post/hessian-serialization-protocol/ ）_
    
*   **Hessian 协议解释与实战（一）：布尔、日期、浮点数与整数**（_ https://www.diguage.com/post/hessian-protocol-interpretation-and-practice-1/_ ）
    
*   **Hessian 协议解释与实战（二）：长整型、二进制数据与 Null**（_ https://www.diguage.com/post/hessian-protocol-interpretation-and-practice-2/_ ）
    
*   **Hessian 协议解释与实战（三）：字符串**（_ https://www.diguage.com/post/hessian-protocol-interpretation-and-practice-3/_ ）
    
*   **Hessian 协议解释与实战（四）：数组与集合**（_ https://www.diguage.com/post/hessian-protocol-interpretation-and-practice-4/_ ）
    
*   **Hessian 协议解释与实战（五）：对象与映射**（_ https://www.diguage.com/post/hessian-protocol-interpretation-and-practice-5/_ ）
    
*   **Hessian 源码分析（Java）**（_ https://www.diguage.com/post/hessian-source-analysis-for-java/_ ）

**往期回顾**

1. [前端日志回捞系统的性能优化实践｜得物技术](https://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247541105&idx=1&sn=ab35ebae0a2d52f2c5f511ac384753f6&scene=21#wechat_redirect)

2. [得物灵犀搜索推荐词分发平台演进 3.0](https://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247541086&idx=1&sn=cca9a43c9627db1e6e7cfc1663b7ea03&scene=21#wechat_redirect)

3. [R8 疑难杂症分析实战：外联优化设计缺陷引起的崩溃｜得物技术](https://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247541053&idx=1&sn=ab0b9cdca2a8dc2eb12be0ab5b324d0e&scene=21#wechat_redirect)

4. [可扩展系统设计的黄金法则与 Go 语言实践｜得物技术](https://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247540985&idx=1&sn=b1b9b4ebff16cded2025c53cf0706f8b&scene=21#wechat_redirect)

5. [营销会场预览直通车实践｜得物技术](https://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247540958&idx=1&sn=776aafdca08eccdec59fc8241160419a&scene=21#wechat_redirect)

文 / 羊羽

关注得物技术，每周一、三更新技术干货

要是觉得文章对你有帮助的话，欢迎评论转发点赞～

未经得物技术许可严禁转载，否则依法追究法律责任。

“

**扫码添加小助手微信**

如有任何疑问，或想要了解更多技术资讯，请添加小助手微信：

![](https://mmbiz.qpic.cn/mmbiz_jpg/AAQtmjCc74DfrQfPwzfAVBicC2hN5FibKQ6iaicVxzgaLzVL1DGibhLIxwicJ0kzqhpPhK1wbSLwUWryOgZPMW3Po44g/640?wx_fmt=jpeg&from=appmsg#imgIndex=38)