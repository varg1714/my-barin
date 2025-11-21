---
source: https://mp.weixin.qq.com/s/UGWoRO5-pFB0p01mc73wLA
create: 2024-07-29 13:51
read: true
knowledge: true
knowledge-date: 2025-11-06
tags:
  - Java
  - 框架原理
summary: "[[Stream 原理与执行流程]]"
---

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAwGtUDICpgiaUAMhW8yQHZ8oLlAmvhc3jw726HozKNxj1LDnw1GEtia7cQ/640?wx_fmt=jpeg&from=appmsg)

阿里妹导读

本文简单讲述了 Stream 原理，并以一段比较简单常见的 stream 操作代码为例进行讲解。

大家都知道可以将 Collection 类转化成流（Stream）进行操作，代码变得简约流畅，写起来一气呵成。为什么流能支持这种流水线式工作模式，用以替代 for 循环呢？接下来让我们来简单探究下。

下面是一段比较简单常见的 stream 操作代码，经过映射与过滤操作后，最后得到的 endList=["ab"]，下文讲解都会以此代码为例。

```java
List<String> startlist = Lists.newArrayList("a","b","c");
List<String> endList = startlist.stream().map(r->r+"b").filter(r->r.startsWith("a")).collect(Collectors.toList());
```

流的三大特点

当我们查阅资料时，得到了流有三大特点：

1. 流并**不存储元素**。这些元素可能**存储在底层的集合中**，或者是按需生成。
2. 流的操作不会修改其数据元素，而是生成一个新的流。
3. 流的操作是尽可能**惰性执行**的。这意味着直至需要其结果时，操作才会执行。

我们可从中捕捉到若干关键词，包括 “不存储元素”，“惰性执行” 等，这些含义希望读者看完流的运行流程后能找到答案。

流的运行流程

一段 Stream 代码的运行包括以下三部分：

1. 搭建流水线，定义各阶段功能。
2. 从终结点反向索引，生成操作实例 Sink。
3. 数据源送入流水线，经过各阶段处理后，生成结果。

**整体类图介绍**

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAwS0Ze49XLqwwaryJ4jgZKe9U9LPRyibnD1PhwzyykYhkq5J9FeLnXFAA/640?wx_fmt=png&from=appmsg)

图 1：Stream 类图

在对原理进行介绍前，先对 Stream 整体类图进行介绍，帮助后续代码理解。

Stream 是一个接口，它定义了对 Stream 的操作，主要可分为中间操作与终结操作，中间操作对流进行转化，比如对数据元素进行映射 / 过滤 / 排序等行为。终结操作启动流水线，获取结果数据。

AbstractPipline 是一个抽象类，定义了流水线节点的常用属性，sourceStage 指向流水线首节点，previousStage 指向本节点上层节点，nextStage 指向本节点下层节点，depth 代表本节点处于流水线第几层（从 0 开始计数），sourceSpliterator 指向数据源。

ReferencePipline 实现 Stream 接口，继承 AbstractPipline 类，它主要对 Stream 中的各个操作进行实现，此外，它还定义了三个内部类 Head/StatelessOp/StatefulOp。Head 为流水线首节点，在集合转为流后，生成 Head 节点。StatelessOp 为无状态操作，StatefulOp 为有状态操作。无状态操作只对当前元素进行作用，比如 filter 操作只需判断 “a” 元素符不符合 “startWith("a")” 这个要求，无需在对 “a” 进行判断时关注数据源其他元素（“b”，“c”）的状态。有状态操作需要关注数据源中其他元素的状态，比如 sorted 操作要保留数据源其他元素，然后进行排序，生成新流。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAwpWyZSDakn93RDpyTzl2KOCGNaxpmHA4AXVAmZib7ztL3T6RPMm3VNYA/640?wx_fmt=png&from=appmsg)

表 1:Stream 操作分类

**搭建流水线**

首先需要区分一个概念，Stream(流) 并不是一个容器，不存储数据，它更像是一个个具有不同功能的流水线节点，可相互串联，容许数据源挨个通过，最后随着终结操作生成结果。Stream 流水线搭建包括三个阶段：

1. 创建一个流，如通过 stream() 产生 Head，Head 就是初始流，数据存储在 Spliterator。
2. 将初始流转换成其他流的中间操作，可能包含多个步骤，比如上面 map 与 filter 操作。
3. 终止操作，用于产生结果，终结操作后，流也就走到了终点。

## 1. 定义输入源 HEAD

只有实现了 Collection 接口的类才能创建流，所以 Map 并不能创建流，List 与 Set 这种单列集合才可创建流。上述代码使用 stream() 方法创建流，也可使用 Stream.of() 创建任何数量引元的流，或是 Array.stream(array,from,to) 从数组中 from 到 to 的位置创建输入源。

### 1.1. stream() 运行结果

示例代码中使用 stream() 方法生成流，让我们看看生成的流中有哪些内容：

```
Stream<String> headStream =startlist.stream();
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAwoXrD4Yb7mXLFukUezZkic57ibSgWJic1k1XslEVgxgErVvicB5LxVAr05Q/640?wx_fmt=other&from=appmsg)

从运行结果来看，stream(）方法生成了 ReferencPipeline$Head 类，ReferencPipeline 是 Stream 的实现类，Head 是 ReferencePipline 的内部类。其中 sourceStage 指向实例本身，depth=0 代表 Head 是流水线首层，sourceSpliterator 指向底层存储数据的集合，其中 list 即初始数据源。

### 1.2. stream() 源码分析

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAw3TY4zlwsukgvKJ3RlvlAJotK8Qx9PjAsmq2Hpw7lArhngxJjBC7FEg/640?wx_fmt=other&from=appmsg)

spliterator()将 “调用 stream() 方法的对象本身 startlist”传入构造函数，生成 Spliterator 类，传入 StreamSupport.stream()方法。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAwFSnsm72Jp3ZvTa5Eq7eJqJPtNScNicVaVibWTrGybxlEYKQYybIVB9Cg/640?wx_fmt=other&from=appmsg)

StreamSupport.stream() 返回了 ReferencPipeline$Head 类。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAw45yF17Ou6xZpia2hUNRic1P52OicNqicnZiaSKibVX3ib8ibDBERPmqQic62ToA/640?wx_fmt=other&from=appmsg)

一路追溯至 AbstractPipline 中，可看到使用 sourceSpliterator 指向数据源，sourceStage 为 Head 实例本身，深度 depth=0。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAwIDbNzrUbaHsv3uCHhdS4x2GQXdgpqItKlqmacBial3pZgicrRgBlwZRg/640?wx_fmt=other&from=appmsg)

## 2. 定义流水线中间节点

### 2.1. Map

### 2.2. map() 运行结果

对数据进行映射，对每个元素后接 "b"。

```java
Stream<String> mapStream =startlist.stream().map(r->r+"b");
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAwicoSPhe3cNBFVhaRXRncmBqHYudxlHJfgP2JEpKzAMImcXOabJOpicug/640?wx_fmt=other&from=appmsg)

此时 sourceStage 与 previousStage 皆指向 Head 节点，depth 变为 1，表示为流水线第二节点，由于代码后续没接其他操作，所以 nextStage 为 null。其中 mapper 代表函数式接口，指向 lambda 代码块，即 “r->r+"b"” 这个操作。

### 2.3. map() 源码分析

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAwwWGF3ZZiaFgHnPf6m8BA9f0A7QGZbEIricHAAWTibxKN8VTU7Qv56c5cQ/640?wx_fmt=other&from=appmsg)

map() 方法是在 ReferencePipline 中被实现的，返回一个无状态操作 StatelessOp，定义 opWrapSink 方法，运行时会将 lambda 代码块的内容替换 apply 方法，对数据元素 u 进行操作。opWrapSink 方法将返回 Sink 对象，其用处将在下文讲解。downstream 为 opWrapSink 的入参 sink。

### 2.4. Filter

### 2.5. filter() 运行结果

filter 对元素进行过滤，只留存以 “a” 开头的数据元素。

```java
Stream<String> filterStream =startlist.stream().map(r->r+"b").filter(r->r.startsWith("a"));
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAwkiauy9eJdOyblNdw0s0xW8ZlrCv1O9khvX8KtNUibedTIzqMD8ABKKlg/640?wx_fmt=other&from=appmsg)

Filter 阶段的 depth 再次 + 1，sourceStage 指向 Head，predict 指向代码块：

“r->r.startsWith("a")”，previousStage 指向前序 Map 节点，同时可见到 Map 节点中的 nextStage 开始指向 Filter，形成双向链表。

### 2.6. filter() 源码分析

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAwBlCbvPfV6suB6y50TqScQODiacWbTlDkbfLzYqEoELibGLRb1tt1Qmpg/640?wx_fmt=other&from=appmsg)

filter() 也是在 ReferencePipline 中被实现，返回一个无状态操作 StatelessOp，实现 opWrapSink 方法，也是返回一个 Sink，其中 accept 方法中的 predicate.test="r->r.startsWith("a")"，用以过滤符合要求的元素。downstream 等于 opWrapSink 入参 Sink。

StatelessOp 的基类 AbstractPipline 中有个构造方法帮助构造了双向链表。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAwcO6ykx2cAqeY32lYwQY0vhpA6QibW5kPwIPpGqNozeCVE2dhBORywYg/640?wx_fmt=other&from=appmsg)

## 3. 定义终结操作

### 3.1. collect() 运行结果

```java
List<String> endList = startlist.stream().map(r->r+"b").filter(r->r.startsWith("a")).collect(Collectors.toList());
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAwXL0h46UY3K0ibmdzrNpia8d208bAEuFibSMhe4OLl29w5PQEP12Kicjtwg/640?wx_fmt=other&from=appmsg)

经过终结操作后，生成最终结果 [“ab”]。

### 3.2. collect() 源码分析

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAwWQ7SyNbCpcDTia6wUZNmlCuqjdibloae6OZ2FDfOCZA5mibdM6QeFasLA/640?wx_fmt=other&from=appmsg)

同样的，collect 终结操作也在 ReferencePipline 中被实现。由于不是并行操作，只要关注 evaluate() 方法即可。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAwzfy1XLp7dpqwDdc6pV35hqolYvxMy5hc4W0ENBTvicLaYTaLnAk4ToA/640?wx_fmt=other&from=appmsg)

makeRef() 方法中也有个类似 opWrapSink 一样返回 Sink 的方法，不过没有以其他 Sink 为输入，而是直接 new 一个 ReducingSInk 对象。

至此，我们可以根据源码绘出下图，使用双向链表连接各个流水线节点，并将每个阶段的 lambda 代码块存入 Sink 类中。数据源使用 sourceSpliterator 引用。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAw6ia4JSwapUTicEh62j6u2blaVdlvYqzXvgSRtTsfrEhWl9xPzf1JKIkQ/640?wx_fmt=other&from=appmsg)

图 2: 流水线搭建

灰色表示还未生成 Sink 实例。

**反向回溯生成操作实例**

还记得前面说的 “惰性执行” 么，在一层一层搭建中间节点时，并未有任何结果产生，而在终结操作 collect 之后，生成最终结果 endList，终结节点到底有什么魔力，让我们探究一下 collect()方法中的 evaluate 方法。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAwJBsQyZ6swLDBuhIPGxhcDOUkG91lL8kqSmLppWqtWqmCAKoMCcTEPA/640?wx_fmt=other&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAwuQlrk9dVgE8wBM1BHEjZk9xM5Y9HMrgwVUiaxngukyEFOvMbIrvOzsw/640?wx_fmt=other&from=appmsg)

这里调用了 Collect 中定义的 makeSink() 方法，输入终结节点生成的 sink 与数据源 spliterator。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAwUtqxcCY0t5ZK4ZFaQial7xykgwc3KevRGdlKw5kl1LVeibwEhuSlMXPA/640?wx_fmt=other&from=appmsg)

先来看 wrapSink 方法，在这个方法里，中间节点的 opWrapSink 方法将发挥大作用，它利用 previousStage 反向索引，后一个节点的 sink 送入前序节点的 opWrapSink 方法中做入参，也就是 downstream，生成当前 sink，再索引向前，生成套娃 Sink。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAwz41hWDQWIyhYU5rmt9Dp3jj1xu7CibiaXWx5bULUsgCyhIOKp4VTAEDg/640?wx_fmt=other&from=appmsg)

最后索引到 depth=1 的 Map 节点，生成的结果 Sink 包含了 depth2 节点 Filter 与终结节点 Collect 的 Sink。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAwNvdv4ODBia4ibxIINyn6F8tmL8DDeKeIk6Fy4ibpmaNHxu7MrP0JRsofw/640?wx_fmt=other&from=appmsg)

红色框图表示 Map 节点的 Sink，包含当前 Stream 与 downstream（Filter 节点 Sink），黄色代表 Filter 节点 Sink，downstream 指向 Collect 节点。

Sink 被反向套娃实例化，一步步索引到 Map 节点，可以对图 2 进行完善。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAwGXFiaNKdaTolDuTUj4cxL6F3Jibicu8BnlGpTcr8gyricWyaZ2hosDMgmQ/640?wx_fmt=other&from=appmsg)

图 3: 反向索引生成 Sink

**启动流水线**

一切准备就绪，就差把数据源冲入流水线。卷起来！在 wrapSink 方法套娃生成 Sink 之后，copyInto 方法将数据源送入了流水线。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAwrN1KNVJTpjho4BGeUAmYpUwduz6HSPEaWjHXXzuicuQK1GORkx42BTA/640?wx_fmt=other&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAwmoSr68kyTricfnsNaavm7yj3RPnHL830lgnolD4uUicZYo59hPSIib9uQ/640?wx_fmt=other&from=appmsg)

先是调用 Sink 中已定义好的 begin 方法，做些前序处理，Sink 中的 begin 方法会不断调用下一个 Sink 的 begin 方法。

随后对数据源中各个元素进行遍历，调用 Sink 中定义好的 accept 方法处理数据元素。accept 执行的就是咱在每一节点定义的 lambda 代码块。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAwlTCmJ9fE23yM5zdOrhjiakJI1DgVaYeHJibqbffVGRSVMiaDKNy20rzfQ/640?wx_fmt=other&from=appmsg)

随后调用 end 方法做后序扫尾工作。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLTicsQR7ga0W0khWaxjSoAw5Ricmb22383PtToMbddTOPoAJOeicneVzdEw444emkdEHspdGTyxNy2A/640?wx_fmt=other&from=appmsg)

图 4: 数据源冲入操作实例，生成最终结果

一个简单 Stream 整体关联图如上所示，最后调用 get() 方法生成结果。 

总结

本文简单讲述了 Stream 原理，可以看到 Stream 流水线就是使用双向链表将各个节点串联而成，当最终节点不为终结操作，则不会产生任何数据结果，只有遇到终结操作，才会产生数据，这就是 “惰性执行”。每跟随一个节点，则会产生一个新的 Stream 对象，这就是 “流的操作不会改变原容器中的数据元素，而是产生新流”。 

**参考链接：**

1. 书籍：《Java 核心技术卷 II》
2. 原来你是这样的 Stream﻿：https://zhuanlan.zhihu.com/p/47478339