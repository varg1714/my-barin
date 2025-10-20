---
source: https://mp.weixin.qq.com/s/jABbG4MKvEiWXwdYwUk8SA
create: 2025-09-22 11:29
read: true
knowledge: true
knowledge-date: 2025-09-22
tags:
  - Java
summary: "[[阅读中/阅读总结/文章收藏/2025-09/Java 日志通关（二） - Slf4j+Logback 整合及排包|Java 日志通关（二） - Slf4j+Logback 整合及排包]]"
---

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naIgx4meDqKQrQics0xRoaOmcHXQuicV1EkuOV5nT4AXeRUKfQPJye81Q9b0smfMXYt7BB9UzDS9NF6Q/640?wx_fmt=jpeg&from=appmsg#imgIndex=0)

阿里妹导读

作者日常在与其他同学合作时，经常发现不合理的日志配置以及五花八门的日志记录方式，后续作者打算在团队内做一次 Java 日志的分享，本文是整理出的系列文章第二篇。

一、为什么是 Slf4j+Logback

看完前一篇的「理论」知识，接下来就要动手了。这一篇就是我的最佳实践：Slf4j+Logback，选择它们的原因如下：

*   Slf4j 的 API 相比 JCL 更丰富，且得到 Intellij IDEA 编辑器的完整支持。这是核心优势，我们会在《Java 日志通关（三） - Slf4 介绍》中详细讲解；
    
*   Slf4j 支持日志内容惰性求值，相比 JCL 性能更好（性能其实也没差多少 [^1]，但码农总是追求极致）；
    
*   在前边选定 Slf4j 的前提下，同一厂牌且表现优异的 Logback 自然中标（并无暗箱操作，举贤不避亲）；
    
*   Slf4j+Logback 是目前大部分开发者的选择（2021 年 Slf4j 76%、Logback 48%[^2]），万一遇到问题参考文档会多一些；

接下来我们就根据前边了解到的理论知识，将这套环境配置起来。

我会先介绍思路，最后再给出整体依赖配置。为了节省空间，依赖会尽量使用冒号分隔的 GAV 风格表达，XML 太占地儿了。

二、基础依赖项

根据前边的知识，我们可以很容易的知道以下三个包是必须的：

*   Slf4j 是基本的日志门面，它的核心 API 在 `org.slf4j:slf4j-api` 中；
    
*   Logback 的核心实现层在 `ch.qos.logback:logback-core` 中；
    
*   Logback 针对 Slf4j 的适配层在 `ch.qos.logback:logback-classic` 中；

其中 logback-classic 会直接依赖另外两项，而且它依赖的一定是它能够支持的最合适版本，所以为了避免歧义，我们可以在项目中仅显式依赖 logback-classic 即可。当然你想提升版本权重，单拎出来也可以。

另外要注意，Slf4j 和 Logback 的版本并不完全向前兼容，它们之间也有对应关系，下边我们逐一介绍。

**2.1 Slf4j 版本兼容性**

Slf4j 2.0.x 有不小的改动 [^3]，不再主动查找 `org.slf4j.impl.StaticLoggerBinder`，而是改用 JDK ServiceLoader[^4]（也就是 SPI，Service Provider Interface） 的方式来加载实现。这是 JDK 8 中的特性，所以 Slf4j 对 JDK 的依赖显而易见：

<table width="732"><tbody><tr><td width="141"><section><span>Slf4j 版本</span></section></td><td width="177"><section><span>JDK 版本</span></section></td><td width="414"><section><span>备注</span></section></td></tr><tr><td width="141"><section><span>Slf4j 1.7.x</span></section></td><td width="177"><section><span>&gt;= JDK 1.5</span></section></td><td width="414"><br></td></tr><tr><td width="141"><section><span>Slf4j 2.0.x</span></section></td><td width="177"><section><span>&gt;= JDK 8</span></section></td><td width="414"><br></td></tr><tr><td width="141"><section><span>Slf4j 2.1.x</span></section></td><td width="177"><section><span>很可能 &gt;= JDK 11</span></section></td><td width="414"><section><span>Ceki 正在征求大家的意见 [^5]</span></section></td></tr></tbody></table>

其中，Slf4j 在发布了几个 1.8 alpha/beta 版后，直接跳到了 2.0，所以 1.8 不在我们的讨论范围内了。

**2.2 Logback 版本兼容性**

因为 Slf4j 技术方案变化，导致 logback-classic 也需要分别做适配，如果使用了不匹配的版本将会报异常（见【2.8 常见问题】）：

<table width="730"><tbody><tr><td width="187"><section><span>Logback 版本</span></section></td><td width="148"><section><span>Slf4j 版本</span></section></td><td width="126"><section><span>JDK 版本</span></section></td><td width="269"><section><span>备注</span></section></td></tr><tr><td width="187"><section><span>Logback 1.2.x</span></section></td><td width="148"><section><span>Slf4j 1.7.x</span></section></td><td width="126"><section><span>&gt;= JDK 1.5</span></section></td><td width="269"><br></td></tr><tr><td width="187"><section><span>Logback 1.3.x</span></section></td><td width="148"><section><span>Slf4j 2.0.x</span></section></td><td width="126"><section><span>&gt;= JDK 8</span></section></td><td width="269"><br></td></tr><tr><td width="187"><section><span>Logback 1.4.x</span></section></td><td width="148"><section><span>Slf4j 2.0.x</span></section></td><td width="126"><section><span>&gt;= JDK 11</span></section></td><td width="269"><br></td></tr><tr><td width="187"><section><span>Logback 1.5.x</span></section></td><td width="148"><section><span>&gt;=Slf4j 2.0.12</span></section></td><td width="126"><section><span>&gt;= JDK 11</span></section></td><td width="269"><section><span>1.5.x 用于替代 1.4.x</span></section></td></tr></tbody></table>

其中 logback 1.3.x 和 1.4.x 是并行维护版本，每次更新都会同时发布 `1.3.n` 和 `1.4.n`，用户需要根据项目的 JDK 版本进行选择。不过目前 Logback 已经全面升级 1.5.x，且 1.3.x 和 1.4.x 不再维护 [^6]，更详细的信息可以参考官网的更新文档 [^7]。

**2.3 总结**

从前边的版本兼容性我们可以知道：

*   如果使用 JDK 8，建议选择 Slf4j 2.0 + Logback 1.3；
    
*   如果使用 JDK 11 及以上，建议选择 Slf4j 2.0 + Logback 1.5；

但还没完，Spring Boot 的日志系统 [^8] 对 Slf4j 和 Logback 又有额外的版本要求。我们放在下一节讨论这个问题。

三、适配 Spring Boot

Spring Boot 通过 spring-boot-starter-logging[^9] 包直接依赖了 Logback（然后再间接依赖了 Slf4j），它通过 `org.springframework.boot.logging.LoggingSystem[^10]` 查找日志接口并自动适配，所以我们使用 Spring Boot 时一般并不需要关心日志依赖，只管使用即可。但因为 Slf4j 2.0.x 与 Slf4j 1.7.x 实现不一致，导致 Spring Boot 也会挑版本：

<table width="613"><tbody><tr><td width="169"><section><span>Spring Boot 版本</span></section></td><td width="130"><section><span>Slf4j 版本</span></section></td><td width="157"><section><span>Logback 版本</span></section></td><td width="157"><section><span>JDK 版本</span></section></td></tr><tr><td width="169"><section><span>Spring Boot 1.5</span></section></td><td width="130"><section><span>Slf4j 1.7.x</span></section></td><td width="157"><section><span>Logback 1.1.x</span></section></td><td width="157"><section><span>&gt;= JDK 7</span></section></td></tr><tr><td width="169"><section><span>Spring Boot 2.x</span></section></td><td width="130"><section><span>Slf4j 1.7.x</span></section></td><td width="157"><section><span>Logback 1.2.x</span></section></td><td width="157"><section><span>&gt;= JDK 8</span></section></td></tr><tr><td width="169"><section><span>Spring Boot 3.x</span></section></td><td width="130"><section><span>Slf4j 2.0.x</span></section></td><td width="157"><section><span>Logback 1.4.x</span></section></td><td width="157"><section><span>&gt;= JDK 17</span></section></td></tr></tbody></table>

根据这个表格，以及前一节总结的版本兼容关系，最终可以得到以下结论：

*   如果使用 Spring Boot 2 及以下，建议选择 Slf4j 1.7.x + Logback 1.2.x；
    
*   如果使用 Spring Boot 3，建议选择 Slf4j 2.0.x + Logback 1.4.x（本篇发表时 Spring 官方还没做好 Logback 1.5.x 的适配）；

如果你使用 Spring Boot 的早期版本又想用上最新的 Slf4j/Logback，可以参考这个讨论 [^11]，其中有不少道友给出了适配方案，比如这个 [^12]，不过我自己没有验证，祝你好运吧。

四、桥接其他实现层

我们还要保证项目中依赖的二方、三方包能够正常打印出日志，而它们可能依赖的是 JCL/Log4j/Log4j2/JUL，我们可以统一引入适配层做好桥接：

*   通过 `org.slf4j:jcl-over-slf4j` 将 JCL 桥接到 Slf4j 上；
    
*   通过 `org.slf4j:log4j-over-slf4j` 将 Log4j 桥接到 Slf4j 上；
    
*   通过 `org.slf4j:jul-to-slf4j` 将 JUL 桥接到 Slf4j 上；
    
*   通过 `org.apache.logging.log4j:log4j-to-slf4j` 将 Log4j 2 桥接到 Slf4j 上；

注意，所有 `org.slf4j` 的包版本要完全一致，所以如果引入这些桥接包，要保证它们的版本与前边选择的 slf4j-api 版本对应。为此 Slf4j 从 2.0.8 开始提供了 bom 包，省去了维护每个包版本的烦恼（至于低版本就只能人肉保证版本一致性了）：

```
<dependencyManagement>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-bom</artifactId>
        <version>2.0.9</version>
        <type>pom</type>
    </dependency>
</dependencyManagement>
```

让我比较意外的是 `log4j-to-slf4j` 这个包，它很「健壮」，对 Slf4j 1 和 Slf4j 2 都能够支持，棒棒的。

五、去除无用依赖

其实桥接层就是个「李鬼」，使用与被桥接包一样的包结构，再暗渡陈仓将调用转到另一个接口上。所以如果同时引入桥接层以及被桥接的包，大概率会引起包冲突。

由于很多工具会在不经意间引入日志接口层 / 实现层，所以我们有必要从整个应用级别着眼，把那些无用的接口层 / 实现层排除掉，包括 JCL、Log4j 和 Log4j 2：

*   排掉 JCL：`commons-logging:commons-logging`
    
*   排掉 Log4j：`log4j:log4j`
    
*   排掉 Log4j 2：`org.apache.logging.log4j:log4j-core`

以及，如果项目间接引入了其他的桥接包，也可能会引起冲突，需要排掉。或者你也可以对照【[1.3 总结](http://mp.weixin.qq.com/s?__biz=MzIzOTU0NTQ0MA==&mid=2247538416&idx=1&sn=363ea68e7c3fe2c4c842b53631a4cb02&chksm=e92a6bffde5de2e96870de37e15eb76450bb16576e18759c8f69cb40980caaae3aaddf87b5b6&scene=21#wechat_redirect)】中所列的包关系，自行判定是否真的需要它。真实项目环境复杂，我们就不在这里一个个枚举了。

**5.1 Gradle 统一排包方案**

如果你使用的是 Gradle，可以使用 `all*.exclude`[^13] 全局排除对应的包。

**5.2 Maven 统一排包方案**

如果你使用是 Maven，很可惜目前还没有全局排包的能力（2006 年就已有人提交了 issue[^14] 但目前仍未支持）。而如果每引入一个包就要考虑排一次又不太现实。综合过往经验以及参考这个 StackOverflow 讨论 [^15]，我们有以下方案可选：

*   方案一：将要排掉的包通过引入一个占位的空包（版本号一般比较特殊，比如 `999-not-exist`），从而达到排包的目的。但这种特殊版本的空包一般在 Mvnrepository Central 仓库是没有的（各厂的私有仓库一般会有这种包），你可以自己搭建私有仓库并上传这个版本，或者使用 Version 99 Does Not Exist [^16] 也行。这是最完美的方案，无论本地运行还是远程编译都不会有问题。
    
*   方案二：将需要排掉的包使用 `<scope>provided</scope>` 标识，这样这个包在编译时会被跳过，从而达到排包的目的，但此包在本地运行时仍会被引入，导致本地运行与远程机器环境差异，不利于调试。
    
*   方案三：使用 maven-enforcer-plugin[^17] 插件标识哪些包是要被排掉的，它只是一个校验，实际上你仍然需要在每个引入了错误包的依赖中进行排除。

六、最终依赖

行文至此，最终的依赖项就有了。其中版本号是截至此文完成（2024 年 4 月）时满足要求的最新版。

**6.1 JDK 8/11 + Spring Boot 1.5/2**

*   基础

*   `org.slf4j:slf4j-api:1.7.36`
    
*   `ch.qos.logback:logback-core:1.2.13`
    
*   `ch.qos.logback:logback-classic:1.2.13`

*   桥接包

*   `org.slf4j:jcl-over-slf4j:1.7.36`
    
*   `org.slf4j:log4j-over-slf4j:1.7.36`
    
*   `org.slf4j:jul-to-slf4j:1.7.36`
    
*   `org.apache.logging.log4j:log4j-to-slf4j:2.23.1`

*   排包

*   `commons-logging:commons-logging:99.0-does-not-exist`
    
*   `log4j:log4j:99.0-does-not-exist`
    
*   `org.apache.logging.log4j:log4j-core:99.0-does-not-exist`

**6.2 JDK 17/21 + Spring Boot 3**

*   基础

*   `org.slf4j:slf4j-bom:2.0.12` 通过 BOM 包统一管理依赖
    
*   `ch.qos.logback:logback-core:1.4.14`
    
*   `ch.qos.logback:logback-classic:1.4.14`

*   桥接包

*   `org.slf4j:jcl-over-slf4j` 参考【七、注意事项】
    
*   `org.slf4j:log4j-over-slf4j` 参考【七、注意事项】
    
*   `org.slf4j:jul-to-slf4j` 参考【七、注意事项】
    
*   `org.apache.logging.log4j:log4j-to-slf4j:2.23.1`

*   排包

*   `commons-logging:commons-logging:99.0-does-not-exist`
    
*   `log4j:log4j:99.0-does-not-exist`
    
*   `org.apache.logging.log4j:log4j-core:99.0-does-not-exist`

七、注意事项

根据前边的介绍，我们在实际项目中将主要做两类事情：

*   引入期望的包，并指定版本；
    
*   排除一些包（指定空版本）；

上边这两个动作，一般我们都是在父 POM 的 `<dependencyManagement>` 中完成的，但这只是管理包版本，在项目没有实际引用之前，并不会真的加载。

在实际项目中，我们一般会按照这个思路来处理：

*   有一个模块 A，依赖 Log4j 打印日志，所以它依赖了 `log4j:log4j` 包；
    
*   我们在父 POM 中把 `log4j:log4j` 排掉了，此时模块 A 调用 Log4j 时会报错；
    
*   我们在父 POM 中引入 `log4j-over-slf4j`，目标是把 Log4j 切到 Slf4j，让模块 A 不报错；

看起来很完美，项目也能正常启动。但当模块 A 需要打印日志时，我们却还是得到了一个错误 `log4j:WARN No appenders could be found for logger (xxx.xxx.xxx)`。这是因为 `log4j-over-slf4j` 并没有真的被引入我们的项目中（很少有哪个二方包会引这种东西，会被骂的）。

解决方案也很简单，将 `log4j-over-slf4j` 通过 `<dependencies>` 引入即可，在父 POM 做这个事也行，在实际有依赖的子 POM 也行。

八、常见问题

其实按照上边我们介绍的接入方案操作，你已经不太会遇到下边这些问题了。不过大多数同学是在维护项目中突然踩了坑，灭火要紧，所以我还是收集了一些日志相关的常见报错，并且尝试给出原因及修复方案（我猜测这可能会是本系列阅读量最高的一篇）。

在编写这部分内容时，我从 Slf4j 官网和 Logback 官网获得了大量有用信息，主要有（推荐你也看看）：

*   SLF4J warning or error messages and their meanings[^18]
    
*   Frequently Asked Questions about SLF4J[^19]
    
*   Bridging legacy APIs[^20]
    
*   Logback error messages and their meanings[^21]
    
*   Frequently Asked Questions (Logback)[^22]

## Q1: LoggerFactory is not a Logback LoggerContext but Logback is on the classpath. Either remove Logback or the competing implementation

包冲突，排掉不需要的 Slf4j 适配层即可，一般是 `logback-classic` 和 `slf4j-log4j12` 冲突，根据你使用的是 Logback 还是 Log4j 2，把另一个排掉。

深究的话，是因为 Spring Boot 在启动时会通过 `LoggingSystem` 获取当前有效的日志系统（参考【三、适配 Spring Boot】），默认支持 Slf4j、Logback、Log4j 2、JUL：

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naKawNNrlpC807icWYZxAWe0WPe5y8nFLRL3WlibGkLgW30iczWcVm6ic7Mxn9ExgSnmX0UnOBzicFaVqWA/640?wx_fmt=jpeg&from=appmsg#imgIndex=1)

LoggingSystem 实现

在获取 Logback 时，因为它和其他 Slf4j 适配层（如 `slf4j-log4j12`）都有名为 `StaticLoggerBinder` 的实现，如果命中的不是 Logback 实现，就会报这个错。

## Q2: java.lang.NoClassDefFoundError: org/slf4j/impl/StaticLoggerBinder

版本不匹配，因为 Slf4j 2.0.x 改用 SPI 方式加载实现（参考【2.2.1 Slf4j 版本兼容性】），当你引入的 Slf4j 和 Logback（或 Log4j 2）版本不匹配时，就会导致这个报错。

## Q3:java.lang.ClassNotFoundException: org.slf4j.impl.StaticLoggerBinder

与 Q2 原因相同。

## Q4:java.lang.ClassCastException:org.apache.logging.slf4j.SLF4JLoggerContext cannot be cast to org.apache.logging.log4j.core.LoggerContext

包冲突，大概率同时引入了 Log4j 2 和针对它的桥接层。原因可以参考【五、去除无用依赖】，但具体排包方案要看你希望使用哪个日志系统了，这里无法明确给出答案，但指导思想就是只保留一个。

## Q5:java.lang.ClassCastException:org.slf4j.impl.Log4jLoggerFactory cannot be cast to ch.qos.logback.classic.LoggerContext

包冲突，大概率同时引入了多个 Slf4j 的适配层，很可能是 `logback-classic` 和 `slf4j-log4j12`。原因和解法都与 Q4 类似。

## Q6: SLF4J: No SLF4J providers were found.

只有 `slf4j-api` 接口，没有适配层。一般添加 `logback-classic` 或者 `slf4j-log4j12` 即可解决。

## Q7: SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".

只有 `slf4j-api` 接口，没有适配层。一般添加 `logback-classic` 或者 `slf4j-log4j12` 即可解决。

不过我在自测时，发现明明有 `logback-classic`，明明项目启动正常，明明日志输出正常，但还可能会报这个错，我还没查到原因，期待高手解惑。

## Q8: SLF4J: Class path contains multiple SLF4J bindings.

Slf4j 发现了多个适配层，一般在这条错误日志后，会列出所有的适配层包路径，把不需要的包排掉即可。

当然 Slf4j 会自动选择第一项，所以如果只出现这一个错误，系统很可能正常启动、正常打日志，从表现来看一切正常。

## Q9: log4j:WARN No appenders could be found for logger (xxx.xxx.xxx)

一般将 `log4j-over-slf4j` 引到项目中可以解决。具体原因请参考【2.7 注意事项】节。

## Q10:java.lang.UnsupportedClassVersionError:ch/qos/logback/classic/spi/LogbackServiceProvider has been compiled by a more recent version of the Java Runtime (class file version 55.0), this version of the Java Runtime only recognizes class file versions up to 52.0

你在用 JDK 8（52.0），但引入的 Logback 版本 >=1.3（它只支持 JDK 11+，即 55.0）。具体兼容性请参考【2.2 Logback 版本兼容性】

## Q11: Failed to load class org.slf4j.impl.StaticLoggerBinder

我在某工程中遇到了这个报错，但项目一切正常，还没找到原因。

**参考链接：**  

[^1]: https://juejin.cn/post/6915015034565951501  

[^2]: https://logging.apache.org/log4j/2.x/manual/extending.html  

[^3]: https://www.slf4j.org/faq.html#changesInVersion200

[^4]: https://docs.oracle.com/javase/8/docs/api/java/util/ServiceLoader.html

[^5]: https://github.com/qos-ch/slf4j/discussions/379

[^6]: https://logback.qos.ch/download.html

[^7]: https://logback.qos.ch/news.html

[^8]: https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.logging

[^9]: https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-logging

[^10]: https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/logging/LoggingSystem.html

[^11]: https://github.com/spring-projects/spring-boot/issues/12649

[^12]: https://github.com/spring-projects/spring-boot/issues/12649#issuecomment -1569448932

[^13]: https://stackoverflow.com/questions/55441430/what-does-this-all-exclude-means-in-gradle-transitive-dependency

[^14]: https://issues.apache.org/jira/browse/MNG-1977

[^15]: https://stackoverflow.com/questions/4716310/is-there-a-way-to-exclude-a-maven-dependency-globally

[^16]: https://github.com/erikvanoosten/version99

[^17]: https://maven.apache.org/enforcer/maven-enforcer-plugin/

[^18]: https://www.slf4j.org/codes.html

[^19]: https://www.slf4j.org/faq.html

[^20]: https://www.slf4j.org/legacy.html

[^21]: https://logback.qos.ch/codes.html

[^22]: https://logback.qos.ch/faq.html

## [点击查看《Java 日志通关（一） - 前世今生](http://mp.weixin.qq.com/s?__biz=MzIzOTU0NTQ0MA==&mid=2247538416&idx=1&sn=363ea68e7c3fe2c4c842b53631a4cb02&chksm=e92a6bffde5de2e96870de37e15eb76450bb16576e18759c8f69cb40980caaae3aaddf87b5b6&scene=21#wechat_redirect)》