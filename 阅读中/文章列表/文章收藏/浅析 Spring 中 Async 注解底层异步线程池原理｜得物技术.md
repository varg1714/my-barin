---
source: https://mp.weixin.qq.com/s/FySv5L0bCdrlb5MoSfQtAA
create: 2024-06-20 15:30
read: true
knowledge: true
knowledge-date: 2025-09-19
tags:
  - Spring
  - 多线程
summary: "[[Spring 中 Async 注解底层异步线程池原理]]"
---

# 浅析 Spring 中 Async 注解底层异步线程池原理｜得物技术

## 1. 浅析 Spring 中 Async 注解底层异步线程池原理｜得物技术

### 1.1. 浅析 Spring 中 Async 注解底层异步线程池原理｜得物技术

![](https://mmbiz.qpic.cn/mmbiz_gif/AAQtmjCc74DZeqm2Rc4qc7ocVLZVd8FOASKicbMfKsaziasqIDXGPt8yR8anxPO3NCF4a4DkYCACam4oNAOBmSbA/640?wx_fmt=gif)

**目录**

一、前言

二、Async 注解简介

    1. Async 注解定义源码

    2. Async 注解异步调用实现原理概述

三、Async 注解底层异步线程池原理探究

    1. 获取 Async 注解线程池主流程解析

    2. Spring 是怎么为 Async 注解提供默认线程池的

四、总结

**一**

**前言**

开发中我们经常会用到异步方法调用，具体到代码层面，异步方法调用的实现方式有很多种，比如最原始的通过实现 Runnable 接口或者继承 Thread 类创建异步线程，然后启动异步线程；再如，可以直接用 java.util.concurrent 包提供的线程池相关 API 实现异步方法调用。

如果说可以用一行代码快速实现异步方法调用，那是不是比上面方法香很多。

Spring 提供了 Async 注解，就可以帮助我们一行代码搞定异步方法调用。Async 注解用起来是很爽，但是如果不对其底层实现做深入研究，难免有时候也会心生疑虑，甚至会因使用不当，遇见一些让人摸不着头脑的问题。

本文首先将对 Async 注解做简单介绍，然后和大家分享一个我们项目中因 Async 注解使用不当的线上问题，接着再深扒 Spring 源码，对 Async 注解底层异步线程池的实现原理一探究竟。

**二**

**Async 注解简介**

**Async 注解定义源码**  

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DJT8fRRo8KY2mVXWXewGo5ick699TI2icVNcRZsBRibTjicEBjKgoebic7H7Jvm22da1TPOicBIxH1EaPw/640?wx_fmt=png&from=appmsg)

从源码可以看出 @Async 注解定义很简单，只需要关注两点：

*   Target({ElementType.TYPE, ElementType.METHOD}) 标志 Async 注解可以作用在方法和类上，作用在类上时，类的所有方法可以实现异步调用。
    
*   String value( ) default "" 是唯一字段属性，用来指定异步线程池，且该字段有缺省值。

**Async 注解异步调用实现原理概述**

在 Spring 框架中，**Async 注解的实现是通过 AOP 来实现的**。具体来说，Async 注解是由 AsyncAnnotationAdvisor 这个切面类来实现的。

AsyncAnnotationAdvisor 类是 Spring 框架中用于处理 Async 注解的切面，**它会在被 Async 注解标识的方法被调用时，创建一个异步代理对象来执行方法**。这个异步代理对象会**在一个新的线程中调用被 @Async 注解标识的方法**，从而实现方法的异步执行。

在 AsyncAnnotationAdvisor 中，会使用 AsyncExecutionInterceptor 来处理 Async 注解。AsyncExecutionInterceptor 是实现了 MethodInterceptor 接口的类，用于拦截被 Async 注解标识的方法的调用，并在一个新的线程中执行这个方法。

通过 AOP 的方式实现 Async 注解的异步执行，Spring 框架可以在方法调用时动态地创建代理对象来实现异步执行，而不需要在业务代码中显式地创建新线程。

总的来说，Async 注解的实现是通过 AOP 机制来实现的，具体的切面类是 AsyncAnnotationAdvisor，它利用 AsyncExecutionInterceptor 来处理被 Async 注解标识的方法的调用，实现方法的异步执行。

**三**

**Async 注解底层异步线程池**

**原理探究**

**获取 Async 注解线程池主流程解析**

进入到 Spring 源码 Async 注解 AOP 切面实现部分，我们重点剖析异步调用实现中线程池是怎么处理的。下图是 org.springframework.aop.interceptor.AsyncExecutionInterceptor#invoke 方法的实现，可以看出是调用 determineAsyncExecutor 方法获取异步线程池。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DJT8fRRo8KY2mVXWXewGo5bQsRevUMAkU5p3thkOx2NCuOxWibER9B5cb0FqDYjH0Yp7yUPiaGQA3g/640?wx_fmt=png&from=appmsg)

AsyncExecutionInterceptor#invoke

下图是 determineAsyncExecutor 方法实现：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DJT8fRRo8KY2mVXWXewGo504z68XhWZ0A7mA0aMzARxxDBeBwc331K6p5HensGWxkac1ico9K9CjA/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DJT8fRRo8KY2mVXWXewGo5Kwicyu0QHSWs6yE5CcXSbO1WGQUREFWLAomYaN41y8DHKOWAiaqYLvaA/640?wx_fmt=png&from=appmsg)

左图为 AsyncExecutionInterceptor#determineAsyncExecutor，右图为 AsyncExecutionAspectSupport#getExecutorQualifier

从代码实现中可以看到 determineAsyncExecutor 获取线程池的大致流程：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DJT8fRRo8KY2mVXWXewGo58a2BhoxNDJc8oFRzPLSD3AH553j2PX3b0nIUCbcE38C8RibTa7UPLxw/640?wx_fmt=png&from=appmsg)

determineAsyncExecutor 获取线程池流程  

如果在使用 Async 注解时指定了自定义线程池比较好理解，如果使用 Async 注解时没有指定自定义线程池，Spring 是怎么处理默认线程池呢？继续深入源码看看 Spring 提供的默认线程池的实现。

**Spring 是怎么为 Async 注解提供**

**默认线程池的**

Async 注解默认线程池有下面两个方法实现：   

*   org.springframework.aop.interceptor.AsyncExecutionInterceptor#getDefaultExecutor
    
*   org.springframework.aop.interceptor.AsyncExecutionAspectSupport#getDefaultExecutor

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DJT8fRRo8KY2mVXWXewGo527OAeGeuy3rOIx8pQdickjUeAz37jGDVsictTYlt0IMnUia8Xic4yDG09A/640?wx_fmt=png&from=appmsg)

AsyncExecutionInterceptor#getDefaultExecutor

可以看出 AsyncExecutionInterceptor#getDefaultExecutor 方法比较简单：先尝试调用父类 AsyncExecutionAspectSupport#getDefaultExecutor 方法获取线程池，**如果父类方法获取不到线程池再用创建 SimpleAsyncTaskExecutor 对象作为 Async 的线程池返回**。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DJT8fRRo8KY2mVXWXewGo5ZwHARMiaRKFcRXVlCJYplicGgeHVfruFRfUDQJNW95eF3uYglHU30L8A/640?wx_fmt=png&from=appmsg)

AsyncExecutionAspectSupport#getDefaultExecutor

再来看父类 AsyncExecutionAspectSupport#getDefaultExecutor 方法的实现，可以看到 Spring 根据类型从 **S****pring 容器中获取 TaskExecutor 类的实例，先记住这个关键点**。

我们知道，Spring 根据类型获取实例时，如果 Spring 容器中有且只有一个指定类型的实例对象，会直接返回，否则的话，会抛出 NoUniqueBeanDefinitionException 异常或者 NoSuchBeanDefinitionException 异常。

但是，对于 Executor 类型，Spring 容器却 “网开一面”，有一个特殊处理：当从 Spring 容器中获取 Executor 实例对象时，**如果满足 @ConditionalOnMissingBean(Executor.class) 条件，Spring 容器会自动装载一个 ThreadPoolTaskExecutor 实例对象，而 ThreadPoolTaskExecutor 是 TaskExecutor 的实现类**。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DJT8fRRo8KY2mVXWXewGo5Idc1lk0iadY9ichYzWYyRK1P7aeDfATvcfeicicQibZyoiaxFEACicpBmfKPQ/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DJT8fRRo8KY2mVXWXewGo5V0rfe8ySLqZicrwyCwCXbBRFuibnY0I8nyiadCbE6gteakUGvdicAwYtRQ/640?wx_fmt=png&from=appmsg)

左图为 TaskExecutionAutoConfiguration，右图为 TaskExecutionProperties

从 TaskExecutionProperties 和 TaskExecutionAutoConfiguration 两个配置类我们看到 Spring 自动装载的 **ThreadPoolTaskExecutor 线程池对象的参数：核心线程数 = 8；最大线程数 = Integer.MAX_VALUE；队列大小 = Integer.MAX_VALUE。**

**四**

**总结**

现在 Async 注解线程池源码已经看的差不多了，下面这张图是 Spring 处理 Async 异步线程池的流程：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DJT8fRRo8KY2mVXWXewGo5cTmFXXUbalSWHNw0WT72E4TVeT4rqOicLe5RFIVhQTSPJPttQwwMlicQ/640?wx_fmt=png&from=appmsg)

Async 异步线程池获取流程

归纳一下：如果在使用 Async 注解时没有指定自定义的线程池会出现以下几种情况：

*   当 Spring 容器中有且仅有一个 TaskExecutor 实例时，Spring 会用这个线程池来处理 Async 注解的异步任务，这可能会踩坑，如果这个 TaskExecutor 实例是第三方 jar 引入的，可能会出现很诡异的问题。
    
*   Spring 创建一个**核心线程数 = 8、最大线程数 = Integer.MAX_VALUE、队列大小 = Integer.MAX_VALUE 的线程池**来处理 Async 注解的异步任务，**这时候也可能会踩坑，由于线程池参数设置不合理，核心线程数 = 8，队列大小过大，如果有大批量并发任务，可能会出现 OOM**。
    
*   Spring 创建 **SimpleAsyncTaskExecutor 实例**来处理 Async 注解的异步任务，**SimpleAsyncTaskExecutor 不是一个好的线程池实现类，SimpleAsyncTaskExecutor 根据需要在当前线程或者新线程中执行异步任务。如果当前线程已经有空闲线程可用，任务将在当前线程中执行，否则将创建一个新线程来执行任务。由于这个线程池没有线程管理的能力，每次提交任务都实时创建新线程，所以如果任务量大，会导致性能下降**。

**往期回顾**

1. [客服测试流水线编排设计思路和准入准出应用｜得物技术](http://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247525842&idx=1&sn=8ef60b41b54efe9db929266934a7905d&chksm=c161308df616b99ba004dadd99239b94f163d7f51d5082fae34ca429ea68da56b35bc60260bb&scene=21#wechat_redirect)  
2. [深入剖析时序 Prophet 模型：工作原理与源码解析｜得物技术](http://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247525652&idx=1&sn=9ad2289419f427b572cef7623dc11ab3&chksm=c161304bf616b95d181e3c40e29a05ee7491f6f8f92fbc3365eb137d838a1d9d2c4eda836860&scene=21#wechat_redirect)  
3. [深入理解 Babel - 项目管理工具 lerna 解析｜得物技术](http://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247525627&idx=1&sn=0df103e2e7cfa5dabdacadfaeb503ddb&chksm=c16131a4f616b8b2609532df41fd6010ffdf05f47e25b9d3544149c9556410081887f1e91282&scene=21#wechat_redirect)  
4. [星创编辑器在投放业务中的落地｜得物技术](http://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247525615&idx=1&sn=6b23091e2937e1c7e3009a0884772b57&chksm=c16131b0f616b8a6720c24f1cf803fe10d88198572c856505f2ef4e54c4920aa26ff97325b38&scene=21#wechat_redirect)  
5. [Java 程序陷入时间裂缝：探索代码深处的神秘停顿｜得物技术](http://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247525463&idx=1&sn=78d614e3e45c0b461110affb38ca8714&chksm=c1613108f616b81e44dcd159a1e1910c46f656f70efcb13b0a011823bb3d4bc50491a6857459&scene=21#wechat_redirect)  

文 / Simon

关注得物技术，每周一、三、五更新技术干货

要是觉得文章对你有帮助的话，欢迎评论转发点赞～

未经得物技术许可严禁转载，否则依法追究法律责任。

“

**扫码添加小助手微信**

如有任何疑问，或想要了解更多技术资讯，请添加小助手微信：

![](https://mmbiz.qpic.cn/mmbiz_jpg/AAQtmjCc74DJT8fRRo8KY2mVXWXewGo567AfxGqWjiaAFW6lhRBZ3D4oyMpgaFBdVABwI5w8aezMfQ222LP64GA/640?wx_fmt=jpeg&from=appmsg)