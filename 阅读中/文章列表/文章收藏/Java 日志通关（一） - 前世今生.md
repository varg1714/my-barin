---
source: https://mp.weixin.qq.com/s/eIiu08fVk194E0BgGL5gow
create: 2024-05-07 12:27
read: false
---

# Java 日志通关（一） - 前世今生

## 1. Java 日志通关（一） - 前世今生

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLDxQhOUZSTPCpqZT5UXBPK7YjShC5XCBLeUcf8nD3R1xQ1tDkE1dPRb1aoE3fvyD72nfWtpo09cA/640?wx_fmt=jpeg&from=appmsg)

阿里妹导读

作者日常在与其他同学合作时，经常发现不合理的日志配置以及五花八门的日志记录方式，后续作者打算在团队内做一次 Java 日志的分享，本文是整理出的系列文章第一篇。

序

写这篇文章的初衷，是想在团队内做一次 Java 日志的分享，因为日常在与其他同学合作时，经常发现不合理的日志配置以及五花八门的日志记录方式。但在准备分享、补充细节的过程中，我又进一步发现目前日志相关的文章，都只是专注于某一个方面，或者讲历史和原理，或者解决包冲突，却都没有把整个 Java 日志知识串联起来。最终这篇文章超越了之前的定位，越写越丰富，为了让大家看得不累，我的文章将以系列的形式展示。  

一、前言

日志发展到今天，被抽象成了三层：接口层、实现层、适配层：

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naKAIwp3hOt1o65w1O31YKYJIXYsdUqXaZRWqjXicMnfNW7zprtyONibn7tP6N26WUegsvaX8ZD3t4Vg/640?wx_fmt=png&from=appmsg)

*   接口层：或者叫日志门面（facade），就是 interface，只定义接口，等着别人实现。
    
*   实现层：真正干活的、能够把日志内容记录下来的工具。但请注意它不是上边接口实现，因为它不感知也不直接实现接口，仅仅是独立的实现。
    
*   适配层：一般称为 Adapter，它才是上边接口的 implements。因为接口层和适配层并非都出自一家之手，它们之间无法直接匹配。而鲁迅曾经说过：「计算机科学领域的任何问题都可以通过增加一个中间层来解决」（All problems in computer science can be solved by another level of indirection. -- David Wheeler[1]），所以就有了适配层。

适配层又可以分为绑定 (Binding) 和桥接 (Bridging) 两种能力：

*   绑定 (Binding)：将接口层绑定到某个实现层（实现一个接口层，并调用实现层的方法）
    
*   桥接 (Bridging)：将接口层桥接到另一个接口层（实现一个接口层，并调用另一个接口层的接口），主要作用是方便用户低成本的在各接口层和适配层之间迁移

如果你觉得上面的描述比较抽象生硬，可以先跳过，等把本篇看完自然就明白了。  
接下来我们就以时间顺序，回顾一下 Java 日志的发展史，这有助于指导我们后续的实践，真正做到知其所以然。  

二、历史演进

**2.1 标准输出 (<1999)**

Java 最开始并没有专门记录日志的工具，大家都是用 System.out 和 System.err 输出日志。但它们只是简单的信息输出，无法区分错误级别、无法控制输出粒度，也没有什么管理、过滤能力。随着 Java 工程化的深入，它们的能力就有些捉襟见肘了。

虽然 System.out 和 System.err 默认输出到控制台，但它们是有能力将输出保存到文件的：

```
System.setOut(new PrintStream(new FileOutputStream("log.txt", true)));
System.out.println("这句将输出到 log.txt 文件中");

System.setErr(new PrintStream(new FileOutputStream("error.txt", true)));
System.err.println("这句将输出到 error.txt 文件中");
```

**2.2 Log4j (1999)**

在 1996 年，一家名为 SEMPER 的欧洲公司决定开发一款用于记录日志的工具。经过多次迭代，最终发展成为 Log4j。这款工具的主要作者是一位名叫 Ceki Gülcü[2] 的俄罗斯程序员，请记住他的名字：Ceki，后面还会多次提到他。

到了 1999 年，Log4j 已经被广泛使用，随着用户规模的增长，用户诉求也开始多样化。于是 Ceki 在 2001 年选择将 Log4j 开源，希望借助社区的力量将 Log4j 发展壮大。不久之后 Apache 基金会向 Log4j 抛出了橄榄枝，自然 Ceki 也加入 Apache 继续从事 Log4j 的开发，从此 Log4j 改名 Apache Log4j[3] 并进入发展的快车道。  

Log4j 相比于 System.out 提供了更强大的能力，甚至很多思想到现在仍被广泛接受，比如：

*   日志可以输出到控制台、文件、数据库，甚至远程服务器和电子邮件（被称做 Appender）；
    
*   日志输出格式（被称做 Layout）允许定制，比如错误日志和普通日志使用不同的展现形式；
    
*   日志被分为 5 个级别（被称作 Level），从低到高依次是 debug, info, warn, error, fatal，输出前会校验配置的允许级别，小于此级别的日志将被忽略。除此之外还有 all, off 两个特殊级别，表示完全放开和完全关闭日志输出；
    
*   可以在工程中随时指定不同的记录器（被称做 Logger），可以为之配置独立的记录位置、日志级别；
    
*   支持通过 properties 或者 xml 文件进行配置；

随着 Log4j 的成功，Apache 又孵化了 Log4Net[4]、Log4cxx[5]、Log4php[6] 产品，开源社区也模仿推出了如 Log4c[7]、Log4cpp[8]、Log4perl[9] 等众多项目。从中也可以印证 Log4j 在日志处理领域的江湖影响力。

不过 Log4j 有比较明显的性能短板，在 Logback 和 Log4j 2 推出后逐渐式微，最终 Apache 在 2015 年宣布终止开发 Log4j 并全面迁移至 Log4j 2[10]（可参考【2.7 Log4j 2 (2012)】）。  

**2.3 JUL (2002.2)**

随着 Java 工程的发展，Sun 也意识到日志记录非常重要，认为这个能力应该由 JRE 原生支持。所以在 1999 年 Sun 提交了 JSR 047[11] 提案，标题就叫「Logging API Specification」。不过直到 2 年后的 2002 年，Java 官方的日志系统才随 Java 1.4 发布。这套系统称做 Java Logging API，包路径是 java.util.logging，简称 JUL。

在某些追溯历史的文章中提到，「Apache 曾希望将 Log4j 加入到 JRE 中作为默认日志实现，但傲慢的 Sun 没有答应，反而很快推出了自己的日志系统」。对于这个说法我并没有找到出处，无法确认其真实性。

不过从实际推出的产品来看，更晚面世的 JUL 无论是功能还是性能都落后于 Log4j，颇有因被寄予厚望而仓促发布的味道，也许那个八卦并非空穴来风，哈哈。虽然在 2004 年推出的 Java 5.0 (1.5) [12] 上 JUL 进步不小，但它在 Log4j 面前仍无太多亮点，广大开发者并没有迁移的动力，导致 JUL 始终未成气候。

我们在后文没有推荐 JUL 的计划，所以这里也不多介绍了（主要是我也不会）。  

**2.4 JCL (2002.8)**

在 Log4j 和 JUL 之外，当时市面上还有像 Apache Avalon[13]（一套服务端开发框架）、 Lumberjack[14]（一套跑在 JDK 1.2/1.3 上的开源日志工具）等日志工具。  
对于独立且轻量的项目来说，开发者可以根据喜好使用某个日志方案即可。但更多情况是一套业务系统依赖了大量的三方工具，而众多三方工具会各自使用不同的日志实现，当它们被集成在一起时，必然导致日志记录混乱。  

为此 Apache 在 2002 年推出了一套接口 Jakarta Commons Logging[15]，简称 JCL，它的主要作者仍然是 Ceki。这套接口主动支持了 Log4j、JUL、Apache Avalon、Lumberjack 等众多日志工具。开发者如果想打印日志，只需调用 JCL 的接口即可，至于最终使用的日志实现则由最上层的业务系统决定。我们可以看到，这其实就是典型的接口与实现分离设计。

但因为是先有的实现（Log4j、JUL）后有的接口（JCL），所以 JCL 配套提供了接口与实现的适配层（没有使用它的最新版，原因会在【1.2.7 Log4j2 (2012)】提到）：

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naKAIwp3hOt1o65w1O31YKYJqCFEH75mouFdF4iayGDQgZ65RzmHn4UTS5vIqaYo8icNgpMic4pRJmYIw/640?wx_fmt=png&from=appmsg)

简单介绍一下 JCL 自带的几个适配层 / 实现层：

*   AvalonLogger/LogKitLogger：用于绑定 Apache Avalon 的适配层，因为 Avalon 不同时期的日志包名不同，适配层也对应有两个
    
*   Jdk13LumberjackLogger：用于绑定 Lumberjack 的适配层
    
*   Jdk14Logger：用于绑定 JUL（因为 JUL 从 JDK 1.4 开始提供）的适配层
    
*   Log4JLogger：用于绑定 Log4j 的适配层
    
*   NoOpLog：JCL 自带的日志实现，但它是空实现，不做任何事情
    
*   SimpleLog：JCL 自带的日志实现 ，让用户哪怕不依赖其他工具也能打印出日志来，只是功能非常简单

当时项目前缀取名 Jakarta，是因为它属于 Apache 与 Sun 共同推出的 Jakarta Project[16] 项目（邮件 [17]）。现在 JCL 作为 Apache Commons[18] 的子项目，叫 Apache Commons Logging，与我们常用的 Commons Lang[19]、Commons Collections [20] 等是师兄弟。但 JCL 的简写命名被保留了下来，并没有改为 ACL。

**2.5 Slf4j (2005)**

Log4j 的作者 Ceki 看到了很多 Log4j 和 JCL 的不足，但又无力推动项目快速迭代，加上对 Apache 的管理不满，认为自己失去了对 Log4j 项目的控制权（博客 [21]、邮件 [22]），于是在 2005 年选择自立门户，并很快推出了一款新作品 Simple Logging Facade for Java[23]，简称 Slf4j。

Slf4j 也是一个接口层，接口设计与 JCL 非常接近（毕竟有师承关系）。相比 JCL 有一个重要的区别是日志实现层的绑定方式：JCL 是动态绑定，即在运行时执行日志记录时判定合适的日志实现；而 Slf4j 选择的是静态绑定，应用编译时已经确定日志实现，性能自然更好。这就是常被提到的 classloader 问题，更详细地讨论可以参考 What is the issue with the runtime discovery algorithm of Apache Commons Logging[24] 以及 Ceki 自己写的文章 Taxonomy of class loader problems encountered when using Jakarta Commons Logging[25]。  

在推出 Slf4j 的时候，市面上已经有了另一套接口层 JCL，为了将选择权交给用户（我猜也为了挖 JCL 的墙角），Slf4j 推出了两个桥接层：

*   jcl-over-slf4j：作用是让已经在使用 JCL 的用户方便的迁移到 Slf4j 上来，你以为调的是 JCL 接口，背后却又转到了 Slf4j 接口。我说这是在挖 JCL 的墙角不过分吧？
    
*   slf4j-jcl：让在使用 Slf4j 的用户方便的迁移到 JCL 上，自己的墙角也挖，主打的就是一个公平公正公开。

Slf4j 通过推出各种适配层，基本满足了用户的所有场景，我们来看一下它的全家桶：

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naKAIwp3hOt1o65w1O31YKYJcZ05uMB9PvkiaR0QE5vkcAClX4QoO80wVpjM9JzAib7zgQ35XfjvwOiag/640?wx_fmt=png&from=appmsg)

网上介绍 Slf4j 的文章，经常会引用它官网上的两张图：

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naKAIwp3hOt1o65w1O31YKYJdTz15JjRialdoqRsydUF8prBXcBtUyK84avdzb5v8sga5eUTLuLQ1bw/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naKAIwp3hOt1o65w1O31YKYJx6Y7O8lhsertvsUz3APMqTdCwwXwxA0SOvh2CM9aB0NJ1wCD4EeIpA/640?wx_fmt=png&from=appmsg)

感兴趣的同学也可以参考。

这里解释一下 slf4j-log4j12 这个名字，它表示 Slf4j + Log4j 1.2（Log4j 的最后一个版本） 的适配层。类似的，slf4j-jdk14 表示 Slf4j + JDK 1.4（就是 JUL）的适配层。

**2.6 Logback (2006)**

然而 Ceki 的目标并不止于 Slf4j，面对自己一手创造的 Log4j，作为原作者自然是知道它存在哪些问题的。于是在 2006 年 Ceki 又推出了一款日志记录实现方案：Logback[26]。无论是易用度、功能、还是性能，Logback 都要优于 Log4j，再加上天然支持 Slf4j 而不需要额外的适配层，自然拥趸者众。目前 Logback 已经成为 Java 社区最被广泛接受的日志实现层（Logback 自己在 2021 年的统计是 48% 的市占率 [27]）。  

相比于 Log4j，Logback 提供了很多我们现在看起来理所当然的新特性：

*   支持日志文件切割滚动记录、支持异步写入
    
*   针对历史日志，既支持按时间或按硬盘占用自动清理，也支持自动压缩以节省硬盘空间
    
*   支持分支语法，通过 <if>, <then>, <else> 可以按条件配置不同的日志输出逻辑，比如判断仅在开发环境输出更详细的日志信息
    
*   大量的日志过滤器，甚至可以做到通过登录用户 Session 识别每一位用户并输出独立的日志文件
    
*   异常堆栈支持打印 jar 包信息，让我们不但知道调用出自哪个文件哪一行，还可以知道这个文件来自哪个 jar 包

Logback 主要由三部分组成（网上各种文章在介绍 classic 和 access 时都描述的语焉不详，我不得不直接翻官网文档找更明确的解释）：

*   logback-core：记录 / 输出日志的核心实现
    
*   logback-classic：适配层，完整实现了 Slf4j 接口
    
*   logback-access[28]：用于将 Logback 集成到 Servlet 容器（Tomcat、Jetty）中，让这些容器的 HTTP 访问日志也可以经由强大的 Logback 输出

**2.7 Log4j 2 (2012)**

看着 Slf4j + Logback 搞的风生水起，Apache 自然不会坐视不理，终于在 2012 年憋出一记大招：Apache Log4j 2[29]，它自然也有不少亮点：

*   插件化结构 [30]，用户可以自己开发插件，实现 Appender、Logger、Filter 完成扩展
    
*   基于 LMAX Disruptor 的异步化输出 [31]，在多线程场景下相比 Logback 有 10 倍左右的性能提升，Apache 官方也把这部分作为主要卖点加以宣传，详细可以看 Log4j 2 Performance[32]。

Log4j 2 主要由两部分组成：

*   log4j-core：核心实现，功能类似于 logback-core
    
*   log4j-api：接口层，功能类似于 Slf4j，里面只包含 Log4j 2 的接口定义

你会发现 Log4j 2 的设计别具一格，提供 JCL 和 Slf4j 之外的第三个接口层（log4j-api，虽然只是自己的接口），它在官网 API Separation[33] 一节中解释说，这样设计可以允许用户在一个项目中同时使用不同的接口层与实现层。

不过目前大家一般把 Log4j 2 作为实现层看待，并引入 JCL 或 Slf4j 作为接口层。特别是 JCL，在时隔近十年后，于 2023 年底推出了 1.3.0 版 [34]，增加了针对 Log4j 2 的适配。还记得我们在【1.2.4 JCL (2002.8)】中没有用最新版的 JCL 做介绍吗，就是因为这个十年之后的版本把那些已经「作古」的日志适配层 @Deprecated 掉了。

多说一句，其实 Logback 和 Slf4j 就像 log4j-core 和 log4j-api 的关系一下，目前如果你想用 Logback 也只能借助 Slf4j。但谁让它们生逢其时呢，大家就会分别讨论认为是两个产品。  

虽然 Log4j 2 发布至今已有十年（本文写于 2024 年），但它仍然无法撼动 Logback 的江湖地位，我个人总结下来主要有两点：

*   Log4j 2 虽然顶着 Log4j 的名号，但却是一套完全重写的日志系统，无法只通过修改 Log4j 版本号完成升级，历史用户升级意愿低
    
*   Log4j 2 比 Logback 晚面世 6 年，却没有提供足够亮眼及差异化的能力（前边介绍的两个亮点对普通用户并没有足够吸引力），而 Slf4j+Logback 这套组合已经非常优秀，先发优势明显

比如，曾有人建议 Spring Boot 将日志系统从 Logback 切换到 Log4j2[35]，但被 Phil Webb[36]（Spring Boot 核心贡献者）否决。他在回复中给出的原因包括：Spring Boot 需要保证向前兼容以方便用户升级，而切换 Log4j 2 是破坏性的；目前绝大部分用户并未面临日志性能问题，Log4j 2 所推崇的性能优势并非框架与用户的核心关切；以及如果用户想在 Spring Boot 中切换到 Log4j 2 也很方便（如需切换可参考 官方文档 [37]）。  

**2.8 spring-jcl (2017)**

因为目前大部分应用都基于 Spring/Spring Boot 搭建，所以我额外介绍一下 spring-jcl [38] 这个包，目前 Spring Boot 用的就是 spring-jcl + Logback 这套方案。

Spring 曾在它的官方 Blog《Logging Dependencies in Spring》[39] 中提到，如果可以重来，Spring 会选择李白 Slf4j 而不是 JCL 作为默认日志接口。

现在 Spring 又想支持 Slf4j，又要保证向前兼容以支持 JCL，于是从 5.0（Spring Boot 2.0）开始提供了 spring-jcl 这个包。它顶着 Spring 的名号，代码中包名却与 JCL 一致（org.apache.commons.logging），作用自然也与 JCL 一致，但它额外适配了 Slf4j，并将 Slf4j 放在查找的第一顺位，从而做到了「既要又要」（你可以回到【1.2.4 JCL (2002.8)】节做一下对比）。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naKAIwp3hOt1o65w1O31YKYJF7qQ4BrRRpHLygvLWwjjibqwfYudFUqkV7tfMxgHQ9xBVPL6APpeqKw/640?wx_fmt=jpeg&from=appmsg)

如果你是基于 Spring Initialize [40] 新创建的应用，可以不必管这个包，它已经在背后默默工作了；如果你在项目开发过程中遇到包冲突，或者需要自己选择日志接口和实现，则可以把 spring-jcl 当作 JCL 对待，大胆排除即可。

**2.9 其他**

除了我们上边提到的日志解决方案，还有一些不那么常见的，比如：

*   Flogger[41]：由 Google 在 2018 年推出的日志接口层。首字母 F 的含义是 Fluent，这也正是它的最大特点：链式调用（或者叫流式 API，Slf4j 2.0 也支持 Fluent API 了，我们会在后续系列文章中介绍）
    
*   JBoss Logging[42]：由 RedHat 在约 2010 年推出，包含完整的接口层、实现层、适配层
    
*   slf4j-reload4j[43]：Ceki 基于 Log4j 1.2.7 fork 出的版本，旨在解决 Log4j 的安全问题，如果你的项目还在使用 Log4j 且不想迁移，建议平替为此版本。（但也不是所有安全问题都能解决，具体可以参考上边的链接）

因为这些日志框架我们在实际开发中用的很少，此文也不再赘述了（主要是我也不会）。  

三、总结

历史介绍完了，但故事并没有结束。两个接口（JCL、Slf4j）四个实现（Log4j、JUL、Logback、Log4j2），再加上无数的适配层，它们之间串联成了一个网，我专门画了一张图：

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naKAIwp3hOt1o65w1O31YKYJpYicay4ougJR5G3FVibW5GNCUMXYbwiaicbJ3eiat6X0ITibUcVoUoDuDgKQ/640?wx_fmt=png&from=appmsg)

解释 / 补充一下这张图：

1.  相同颜色的模块拥有相同的 groupId，可以参考图例中给出的具体值。
    
2.  JCL 的适配层是直接在它自己的包中提供的，详情我们在前边已经介绍过，可以回【1.2.4 JCL (2002.8)】查看。
    
3.  要想使用 Logback，就一定绕不开 Slf4j（引用它的适配层也算）；同样的，要想使用 Log4j 2，那它的 log4j-api 也绕不开。

如果你之前在看「1.1 前言」时觉得过于抽象，那么此时建议你再回头看一下，相信会有更多体会。  

从这段历史，我也发现了几个有趣的细节：

*   在 Log4j 2 面世前后的很长一段时间，Slf4j 及 Logback 因为没有竞争对手而更新缓慢。英雄没有对手只能慢慢垂暮，只有棋逢对手才能笑傲江湖。
    
*   技术人的善良与倔强：面世晚的产品都针对前辈产品提供支持；面世早的产品都不搭理它的「后辈」。
    
*   计算机科学领域的任何问题都可以通过增加一个中间层来解决，如果不行就两个（桥接层干的事儿）。
    
*   Ceki 一人肩挑 Java 日志半壁江山 25 年（还在增长 ing），真神人也。（当然在代码界有很多这样的神人，比如 Linus Torvalds[44] 维护 Linux 至今已有 33 年，虽然后期主要作为产品经理参与，再比如已故的 Bram Moolenaar[45] 老爷子持续维护 Vim 32 年之久）。

**参考链接：**

[1]https://codedocs.org/what-is/david-wheeler-computer-scientist

[2]https://github.com/ceki  

[3]https://logging.apache.org/log4j/1.2/

[4]https://logging.apache.org/log4net/

[5]https://logging.apache.org/log4cxx/  

[6]https://logging.apache.org/log4php/

[7]https://log4c.sourceforge.net/  

[8]https://log4cpp.sourceforge.net/

[9]https://mschilli.github.io/log4perl/

[10]https://news.apache.org/foundation/entry/apache_logging_services_project_announces

[11]https://jcp.org/en/jsr/detail

[12]https://www.java.com/releases/  

[13]https://avalon.apache.org/

[14]https://javalogging.sourceforge.net/

[15]https://commons.apache.org/proper/commons-logging/  

[16]https://jakarta.apache.org/

[17]https://lists.apache.org/thread/53otcqljjfnvjs3hv8m4ldzlgz59yk6k  

[18]https://commons.apache.org/

[19]https://commons.apache.org/proper/commons-lang/

[20]https://commons.apache.org/proper/commons-collections/

[21]http://ceki.blogspot.com/2010/05/forces-and-vulnerabilites-of-apache.html

[22]https://lists.apache.org/thread/dyzmtholjdlf3h32vvl85so8sbj3v0qz  

[23]https://www.slf4j.org/

[24]https://stackoverflow.com/questions/3222895/what-is-the-issue-with-the-runtime-discovery-algorithm-of-apache-commons-logging

[25]https://articles.qos.ch/classloader.html  

[26]https://logback.qos.ch/

[27]https://qos.ch/  

[28]https://logback.qos.ch/access.html

[29]https://logging.apache.org/log4j/2.x/

[30]https://logging.apache.org/log4j/2.x/manual/extending.html

[31]https://logging.apache.org/log4j/2.x/manual/async.html

[32]https://logging.apache.org/log4j/2.x/performance.html  

[33]https://logging.apache.org/log4j/2.x/manual/api-separation.html

[34]https://commons.apache.org/proper/commons-logging/changes-report.html

[35]https://github.com/spring-projects/spring-boot/issues/16864

[36]https://spring.io/team/philwebb

[37]https://docs.spring.io/spring-boot/docs/3.2.x/reference/html/howto.html  

[38]https://docs.spring.io/spring-framework/reference/core/spring-jcl.html

[39]https://spring.io/blog/2009/12/04/logging-dependencies-in-spring

[40]https://start.spring.io/

[41]https://google.github.io/flogger/

[42]https://github.com/jboss-logging

[43]https://reload4j.qos.ch/

[44]https://github.com/torvalds

[45]https://moolenaar.net/

‍