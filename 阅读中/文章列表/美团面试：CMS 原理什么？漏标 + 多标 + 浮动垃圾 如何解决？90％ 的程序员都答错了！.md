---
source: https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247505019&idx=1&sn=9905bb2caad9178a330783a9b90fbdef&scene=21#wechat_redirect
create: 2025-07-15 17:46
read: true
knowledge: true
knowledge-date: 2025-08-22
tags:
  - JVM
  - Java
summary: "[[CMS原理]]"
---
FSAC 未来超级架构师

架构师总动员  
实现架构转型，再无中年危机

# 尼恩说在前面

在 40 岁老架构师 尼恩的**读者交流群** (50+) 中，最近有小伙伴拿到了一线互联网企业如得物、阿里、滴滴、极兔、有赞、希音、百度、网易、美团的面试资格，遇到很多很重要的面试题：

听说你是高手，说说，你的 CMS 怎么调优？

说说，CMS 垃圾回收器的底层原理？

**说说，CMS  的浮动垃圾，是怎么处理的？**

说说，CMS 垃圾回收器的调优过程？

美团面试：CMS 原理什么？漏标 + 多标 + 浮动垃圾 如何解决？

最近有小伙伴在面试 美团，又遇到了相关的面试题。小伙伴懵了，因为没有遇到过，所以支支吾吾的说了几句，面试官不满意，面试挂了。

所以，尼恩给大家做一下系统化、体系化的梳理，使得大家内力猛增，可以充分展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer 直提”。

当然，这道面试题，以及参考答案，也会收入咱们的 《[尼恩 Java 面试宝典 PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》V171 版本，供后面的小伙伴参考，提升大家的 3 高 架构、设计、开发水平。

最新《尼恩 架构笔记》《尼恩高并发三部曲》《尼恩 Java 面试宝典》的 PDF，请关注本公众号【技术自由圈】获取，回复：领电子书

另外，此文的内容，作为 第 10 章，收入尼恩的《JVM 调优圣经》PDF。

文章目录：（重点在后面）

- 尼恩说在前面

- 第 10 章：cms 底层原理和调优实战

 - 面试背景

 - 回答核心要点

 - CMS 底层原理

 - CMS 调优策略

 - 回答策略

 - 避坑指南

- CMS 解决了什么问题？

- 什么是 CMS？

 - 9 款  垃圾回收器 的 JDK 版本

 - CMS‌的核心机制‌是什么？

 - CMS‌的特点：

- CMS 几个基本数据结构

 - 结构 1：卡表

 - 为什么需要卡表？

 - 什么是卡表？

 - 结构 2：写屏障

- CMS 的垃圾回收的触发时机‌

- CMS 的整体 工作流程

 - 按照 7 个阶段划分， cms 基本流程：

 - 按照 4 个阶段划分， cms 基本流程：

 - 为什么有的文章 cms 是 4 个阶段， 有的 文章 cms 是 6 个阶段 ?

 - 四个阶段的划分

 - 六个阶段的划分

 - 预清理和可中断的预清理的作用

 - 第一个‌阶段：初始标记（Initial Mark）

 - 为什么需要初始标记（Initial Mark）

 - 什么是初始标记（Initial Mark）

 - 初始标记（Initial Mark） 阶段定位与核心任务

 - 初始标记（Initial Mark） 执行流程‌

 - 初始标记（Initial Mark） 技术实现细节‌

 - 初始标记（Initial Mark）与其他阶段的对比‌

 - 初始标记（Initial Mark）设计意义与局限性

 - 第二个阶段：并发标记（Concurrent Mark）

 - 为什么需要 并发标记（Concurrent Mark）

 - 什么是 并发标记（Concurrent Mark）

 - 并发标记（Concurrent Mark）阶段定义与核心作用‌

 - 并发标记（Concurrent Mark）执行特点与机制‌

 - 并发标记（Concurrent Mark）性能影响与挑战‌

 - 并发标记（Concurrent Mark）与其他阶段的协同‌

 - 并发标记（Concurrent Mark）优化参数与配置‌

 - 第三个阶段： 预清理（Preclean）

 - 为什么需要预清理（Preclean）？

 - 什么是预清理（Preclean）？

 - 预清理（Preclean）‌ 阶段定义与核心作用‌

 - 预清理（Preclean）‌执行机制与关键操作‌

 - 预清理（Preclean）性能优化与参数配置‌

 - 预清理（Preclean）对后续阶段的影响‌

 - 第四个阶段： 可终止预清理（Abortable Preclean）

 - 为什么需要 可终止预清理（Abortable Preclean）？

 - 什么是 可终止预清理（Abortable Preclean）

 - ‌可终止预清理（Abortable Preclean）阶段定义与核心任务

 - ‌可终止预清理（Abortable Preclean）‌执行机制与关键操作‌

 - ‌可终止预清理（Abortable Preclean）‌‌ 性能优化与挑战‌

 - ‌可终止预清理（Abortable Preclean）‌参数配置与调优‌

 - 预清理和可终止预清理区别：

 - 第五个阶段： 重新标记（Remark）

 - 为什么需要 重新标记（Remark）？

 - 什么是 重新标记（Remark）？

 - 重新标记（Remark）阶段定义与核心作用‌

 - ‌重新标记（Remark）执行机制与关键操作‌

 - 重新标记（Remark） 性能影响与挑战‌

 - ‌重新标记（Remark） 与其他阶段的协同‌

 - ‌重新标记（Remark） 优化参数与配置‌

 - ‌第六个阶段： 并发清理（Concurrent Sweep）

 - 并发清理（Concurrent Sweep）阶段定义与核心作用‌

 - ‌并发清理（Concurrent Sweep）执行机制与关键操作‌

 - ‌并发清理（Concurrent Sweep）性能影响与挑战‌

 - ‌并发清理（Concurrent Sweep）参数配置与优化‌

 - ‌并发清理（Concurrent Sweep）与其他阶段的协同‌

 - 第七个阶段：  ‌重置（Reset）阶段

 - 为什么需要  重置（Reset）阶段？

 - ‌重置（Reset）阶段 阶段定义与核心作用‌

 - ‌重置（Reset）阶段 执行机制与关键操作‌

 - ‌重置（Reset）阶段 性能影响与挑战‌

 - ‌重置（Reset）阶段 与其他阶段的协同‌

- CMS 优缺点分析

 - ‌核心优点‌

 - ‌主要缺点‌

 - ‌适用场景与替代方案‌

- CMS 与 G1 对比分析

 - ‌1. 工作机制对比‌

 - ‌2. 内存管理与碎片处理‌

 - ‌3. 性能特点对比‌

 - ‌4. 适用场景与限制‌

 - ‌5. 关键缺陷对比‌

- CMS 的垃圾回收 调优

 - 调优工具与监控手段‌

 - 关键调优动作与参数调整‌

 - 场景化调优建议‌

 - 调优验证与风险控制‌

- Concurrent Mode Failure 是什么？

 - Concurrent Mode Failure 触发条件

 - 1 老年代预留空间不足

 - ‌2  晋升失败（Promotion Failed）‌

 - ‌3  大对象直接分配失败‌

 - ‌4 CMS 回收速度滞后于对象分配速度‌

 - Concurrent Mode Failure 参数与调优

 - 三色标记的漏标、多标问题

 - 漏标问题：

 - 多标问题：

 - 大厂面试：CMS 如何 解决 漏标 ？

 - CMS 通过以下方式处理 漏标问题：

 - 1. 并发标记阶段（Concurrent Marking）

 - 2 重新标记阶段（Remark Phase）

 - 大厂面试：CMS 如何 解决 多标 ？（浮动垃圾）

 - （1）增量更新（Incremental Update）

 - （2）重新标记阶段（Remark Phase）

 - （3）并发清理阶段（Concurrent Sweep）

 - （4）下一次 GC 回收

# 第 10 章：cms 底层原理和调优实战

《cms 底层原理和调优实战》内容正在写作中，本月底发布。

尼恩希望通过 JVM 调优圣经一个 PDF，帮助大家一举成为 JVM 调优 小王子。

实现通过 JVM 调优 的超级技能，去毒打面试官。

完整的 PDF 还在写作中，晚些时候进行发布。

说在最后：有问题找老架构取经

《JVM 调优圣经》PDF 第 11 章：

[美团面试：G1 垃圾回收 底层原理是什么？说说你的调优过程？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247503490&idx=1&sn=fe8dcd5a67b7bd7b1d5bebecd21a3086&scene=21#wechat_redirect)

接下来，咱们言归正传，开始讲 cms

## 面试背景

CMS（Concurrent Mark Sweep）作为老年代垃圾回收器，以‌**低延迟**‌为核心目标，适用于对响应时间敏感的系统。

尽管 G1、ZGC 等新回收器逐渐普及，但 CMS 仍广泛存在于传统企业级应用中。

面试官提问 CMS 底层原理及调优，主要考察以下能力：

(1) 对并发垃圾回收机制的理解（如如何减少 STW 时间）‌；

(2) 实际调优经验（参数调整、问题定位能力）‌；

(3) 对 JVM 内存模型与 GC **算法的综合掌握程度（如卡表机制、碎片处理）‌。**

## 回答核心要点

### CMS 底层原理

‌**四阶段工作流程**  

- ‌**初始标记（Initial Mark）**‌：STW 阶段，标记 GC Roots 直接关联的对象‌；

- ‌**并发标记（Concurrent Mark）**‌：与用户线程并发执行，遍历老年代对象引用链‌；

- ‌**重新标记（Remark）**‌：STW 阶段，修正并发标记期间变动的引用关系（通过增量更新或原始快照算法）‌；

- ‌**并发清除（Concurrent Sweep）**‌：删除无引用对象，回收内存空间‌。

‌**关键技术机制**‌

- ‌**卡表（Card Table）**‌：将老年代划分为 512 字节的卡片，记录跨代引用，避免 YGC 时扫描整个老年代‌；

- ‌**增量并发**‌：通过交替执行 GC 线程与用户线程减少 STW 时间，但可能引发并发失败（Concurrent Mode Failure）‌。

### CMS 调优策略

**核心参数调整**‌

*   `-XX:CMSInitiatingOccupancyFraction=70`：老年代内存占用达 70% 时触发 CMS 回收，避免 Full GC‌；
    
*   `-XX:+UseCMSCompactAtFullCollection`：Full GC 时压缩内存碎片（默认关闭，需权衡性能）‌；
    
*   `-XX:+CMSParallelRemarkEnabled`：启用并行重新标记，减少 Remark 阶段耗时‌。
    

‌**典型问题场景调优**  

‌- ‌**频繁 Concurrent Mode Failure**‌ 

表现：触发 Serial Old 回收器导致长 STW。 

解决：增大老年代空间或降低`CMSInitiatingOccupancyFraction`阈值‌。

- ‌**内存碎片严重**‌ 

表现：老年代剩余空间足够但无法分配大对象（晋升失败）。 

 解决：启用`UseCMSCompactAtFullCollection`或周期性 Full GC‌。

- ‌**Remark 阶段耗时高**‌ 

 解决：通过`CMSScavengeBeforeRemark`在 Remark 前触发 YGC，减少跨代引用跟踪负担‌。

‌**调优优先级策略**‌

- ‌**先监控后调优**‌：使用`jstat -gcutil`观察 GC 频率 / 耗时，`jmap`分析内存分布‌；

- ‌**扩容优先**‌：若硬件成本允许，优先增加内存而非复杂调优（符合生产环境常见做法）‌。

## 回答策略

‌**结构化表达**‌：

按 “原理→问题→解决方案” 逻辑展开，如先说明 CMS 阶段，再针对各阶段常见问题给出调优方法‌；

**结合场景举例**‌：

如描述某次线上服务因碎片问题导致 Full GC 耗时增加，通过调整`CMSFullGCsBeforeCompaction`解决‌；

‌**强调 权衡意识**‌：

如低延迟与吞吐量的取舍、碎片压缩与暂停时间的平衡‌；

**关联新技术**‌：

对比 CMS 与 G1/ZGC 的优劣，体现技术视野（如 CMS 适用于中小堆，ZGC 适合超大堆）‌。

## 避坑指南

避免过度调优：强调 JVM 默认参数在多数场景已足够可靠，调优需以监控数据为依据‌；

勿混淆概念：区分卡表（Card Table）与记忆集（Remembered Set），前者用于跨代引用，后者用于分代收集器的跨区域引用‌。

# CMS 解决了什么问题？

CMS 的核心价值在于 ‌**以并发回收机制实现毫秒级停顿**‌，尤其适合中小规模堆内存下对延迟敏感的场景（如实时服务）‌。

但其内存碎片和 CPU 资源占用问题需通过参数调优规避，且在超大堆（≥8GB）或长期运行场景中，G1 或 ZGC 可能是更优选择‌。

CMS（Concurrent Mark Sweep）垃圾回收器主要解决以下问题：

*   减少垃圾回收停顿时间
    
*   高吞吐量与低延迟的平衡
    
*   大堆内存管理
    
*   解决内存碎片问题
    
*   并发标记和清除
    
*   处理对象引用变化
    

# 什么是 CMS？

CMS 是英文 Concurrent Mark-Sweep 的简称，是以牺牲吞吐量为代价来获得最短回收停顿时间的垃圾回收器。

对于要求服务器响应速度的应用上，这种垃圾回收器非常适合。

在启动 JVM 的参数加上`-XX:+UseConcMarkSweepGC`来指定使用 CMS 垃圾回收器。

CMS 使用的是标记 - 清除的算法实现的，所以在 gc 的时候回产生大量的内存碎片，当剩余内存不能满足程序运行要求时，系统将会出现 Concurrent Mode Failure，临时 CMS 会采用 Serial Old 回收器进行垃圾清除，此时的性能将会被降低。

CMS（Concurrent Mark Sweep）是基于‌标记 - 清除算法‌的老年代垃圾收集器，设计目标为‌最小化停顿时间‌‌。其核心思想是通过‌并发标记与清理‌减少 STW（Stop The World）时长，适用于对延迟敏感的应用场景（如 Web 服务）‌。

## 9 款  垃圾回收器 的 JDK 版本

了解下 HotSpot 虚拟机中 9 款  垃圾回收器 的发布时间及其对应的 JDK 版本，如下图：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WOwKYibYDGPpJbjFMjhM8juVw671iansebOYdiayicXcHvwta4JybBlD6Fsr2XZI0HWsGHVxyloIn3l1w/640?from=appmsg)

<table><thead><tr><td><span><strong><span leaf="">版本</span></strong></span></td><td><span><strong><span leaf="">发布时间</span></strong></span></td><td><span><strong><span leaf="">默认收集器</span></strong></span></td><td><span><strong><span leaf="">事件</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">jdk1.3</span></span></section></td><td><section><span><span leaf="">2000-05-08</span></span></section></td><td><section><span><span leaf="">serial</span></span></section></td><td><section><span leaf=""><br></span></section></td></tr><tr><td><section><span><span leaf="">jdk1.4</span></span></section></td><td><section><span><span leaf="">2004-02-06</span></span></section></td><td><section><span><span leaf="">ParNew</span></span></section></td><td><section><span leaf=""><br></span></section></td></tr><tr><td><section><span><span leaf="">jdk1.5/5.0</span></span></section></td><td><section><span><span leaf="">2004-09-30</span></span></section></td><td><section><span><span leaf="">Parallel Scavenge/serial</span></span></section></td><td><section><span><span leaf="">CMS 登场</span></span></section></td></tr><tr><td><section><span><span leaf="">jdk1.6/6.0</span></span></section></td><td><section><span><span leaf="">2006-12-11</span></span></section></td><td><section><span><span leaf="">Parallel Scavenge/Parallel Old</span></span></section></td><td><section><span leaf=""><br></span></section></td></tr><tr><td><section><span><span leaf="">dk1.7/7.0</span></span></section></td><td><section><span><span leaf="">2011-07-28</span></span></section></td><td><section><span><span leaf="">Parallel Scavenge/Parallel Old</span></span></section></td><td><section><span><span leaf="">G1 登场</span></span></section></td></tr><tr><td><section><span><span leaf="">jdk1.8/8.0</span></span></section></td><td><section><span><span leaf="">2014-03-18</span></span></section></td><td><section><span><span leaf="">Parallel Scavenge/Parallel Old</span></span></section></td><td><section><span leaf=""><br></span></section></td></tr><tr><td><section><span><span leaf="">jdk1.9/9.0</span></span></section></td><td><section><span><span leaf="">2014-09-8</span></span></section></td><td><section><span><span leaf="">G1</span></span></section></td><td><section><span><span leaf="">CMS 废弃</span></span></section></td></tr><tr><td><section><span><span leaf="">jdk10</span></span></section></td><td><section><span><span leaf="">2018-03-21</span></span></section></td><td><section><span><span leaf="">G1</span></span></section></td><td><section><span leaf=""><br></span></section></td></tr><tr><td><section><span><span leaf="">jdk11</span></span></section></td><td><section><span><span leaf="">2018-09-25</span></span></section></td><td><section><span><span leaf="">G1</span></span></section></td><td><section><span><span leaf="">ZGC 登场</span></span></section></td></tr><tr><td><section><span><span leaf="">jdk12</span></span></section></td><td><section><span><span leaf="">2019-3</span></span></section></td><td><section><span><span leaf="">G1</span></span></section></td><td><section><span><span leaf="">Shenandoah</span></span></section></td></tr><tr><td><section><span><span leaf="">jdk13</span></span></section></td><td><section><span><span leaf="">2019-9</span></span></section></td><td><section><span><span leaf="">G1</span></span></section></td><td><section><span leaf=""><br></span></section></td></tr><tr><td><section><span><span leaf="">jdk14</span></span></section></td><td><section><span><span leaf="">2020-3</span></span></section></td><td><section><span><span leaf="">G1</span></span></section></td><td><section><span><span leaf="">CMS 移除</span></span></section></td></tr><tr><td><section><span><span leaf="">jdk15</span></span></section></td><td><section><span><span leaf="">2020-9-15</span></span></section></td><td><section><span><span leaf="">G1</span></span></section></td><td><section><span><span leaf="">ZGC、Shenandoah 转正</span></span></section></td></tr><tr><td><section><span><span leaf="">jdk16</span></span></section></td><td><section><span><span leaf="">2021-3-16</span></span></section></td><td><section><span><span leaf="">G1</span></span></section></td><td><section><span leaf=""><br></span></section></td></tr><tr><td><section><span><span leaf="">jdk17</span></span></section></td><td><section><span><span leaf="">2021-09-14</span></span></section></td><td><section><span><span leaf="">G1</span></span></section></td><td><section><span leaf=""><br></span></section></td></tr><tr><td><section><span><span leaf="">jdk21</span></span></section></td><td><section><span><span leaf="">2022-3-22</span></span></section></td><td><section><span><span leaf="">G1</span></span></section></td><td><section><span><span leaf="">ZGC 分代</span></span></section></td></tr><tr><td><section><span><span leaf="">jdk23</span></span></section></td><td><section><span><span leaf="">2022-9-22</span></span></section></td><td><section><span><span leaf="">ZGC</span></span></section></td><td><section><span leaf=""><br></span></section></td></tr></tbody></table>

JDK 1.4.1 时 ，CMS 垃圾收集器被引入，在 2020 年 3 月，JDK 14 版本，CMS 从 JDK 中移除。

G1 垃圾收集器 在 JDK 7 时引入，在 JDK 9 时 G1 取代 CMS 成为了默认的垃圾收集器。

就目前来说，JVM 的垃圾收集器主要分为两大类：分代收集器和分区收集器，分代收集器的代表是 CMS，分区收集器的代表是 G1 和 ZGC，下面我们来看看这两大类的垃圾收集器。

如下图：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WOwKYibYDGPpJbjFMjhM8juV38mzeLMyFIwrGpnWJYUyiad26lG4zhu5JZicCJVaKfRicMWnDX4iaCTGsw/640?from=appmsg)

说明：分代垃圾收集器中，新生代有 Serial、ParNew、Parallel Scavenge，老年代包括 CMS、MSC、Parallel old，收集器之间的连线说明两者可以搭配使用。

## CMS‌的核心机制‌是什么？

**(1) ‌并发标记与清理‌：**

通过多线程与用户线程并发执行标记和清理操作，仅初始标记和重新标记阶段需短暂 STW‌

**(2) ‌三色标记法‌：**

基于黑（已标记且存活）、灰（标记中）、白（未标记或垃圾）的对象状态跟踪，结合‌写屏障（Write Barrier）‌记录并发阶段的对象引用变化，防止漏标‌

**(3) ‌内存碎片处理‌：**

标记 - 清除算法不压缩内存 ，长期运行后可能产生碎片，依赖 Full GC（Serial Old）或参数触发碎片整理‌**

## CMS‌的特点：

*   并发收集：GC 线程与用户线程并发执行
    
*   低停顿：追求最短回收停顿时间
    
*   标记 - 清除算法：会产生内存碎片
    
*   分代收集：与 ParNew 配合使用（年轻代用 ParNew，老年代用 CMS）
    

# CMS 几个基本数据结构

## 结构 1：卡表

### 为什么需要卡表？

CMS 垃圾回收器需要卡表的核心原因在于‌**优化跨代引用扫描效率**‌，解决跨代引用扫描问题的效率问题：

**1 跨代引用带来的性能瓶颈**‌

在 YGC（年轻代垃圾回收）时，老年代对象可能引用年轻代对象。

若直接扫描整个老年代寻找  跨代引用，时间和资源消耗将无法接受‌。

**2 并发标记阶段，记录引用变化**

CMS 并发标记阶段中，‌**老年代引用变化会通过写屏障标记到卡表**‌，核心目的是在重新标记阶段高效修正并发期间遗漏的存活对象标记。

卡表在此场景下不仅用于跨代引用跟踪，还直接服务于老年代内部引用变更的记录与处理‌

**3 卡表的核心作用**‌

卡表通过‌**将老年代划分为固定大小的卡片（Card，通常 512 字节）**‌，并记录卡片是否被修改（即脏卡标记）。

YGC 时仅需扫描标记为脏的卡片，而非全量扫描老年代，大幅降低扫描范围‌。

卡表  在  YGC 的时候， 贡献很大。

### 什么是卡表？

对于分代垃圾回收器，势必存在一个跨代引用的问题，

而卡表就是最常用的一种跨代引用 记录结构  ，它是一个字节数组，用于记录堆内存的映射关系，下面是 HotSpot 虚拟机默认的卡表标记逻辑：

```
// >> 9 代表右移 9位，即 2^9 = 512 字节CARD_TABLE[this address >> 9] = 0;
```

每个元素都对应着其标识的内存区域中一块特定大小的内存块，这个内存块叫做 “卡页（Card Page）”。

因为卡页代表的是一个区域，所以可能存在很多对象，只要有一个对象存在跨代引用，就把数组的值设为 1，称该元素 “变脏（Dirty）”，该卡页叫 “脏页（Dirty Page）”，

如下： 1 2 当垃圾回收时，只要筛选卡表中有变脏的元素，即数组值为 1，就能判断出其对应的内存区域存在对象跨代引用，卡表和卡页的关系如下图：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WOwKYibYDGPpJbjFMjhM8juVtMrsGVibsODbUkFjWwj5PBHQEJxgZN5lm5JqCdct78Mrq07OmhUzBZw/640?from=appmsg)

在 CMS（Concurrent Mark-Sweep）垃圾收集器中，卡表（Card Table）的主要作用是记录老年代对象对年轻代对象的引用，以加速年轻代垃圾收集（YGC）的过程。在 YGC 时，卡表帮助快速定位哪些老年代对象可能引用了年轻代对象，从而避免扫描整个老年代，提升效率。

在 Full GC（全局垃圾收集）的场景中，卡表的贡献相对有限，因为 Full GC 会扫描整个堆内存，包括年轻代和老年代，确保所有不可达对象都被回收。具体来说：

**1 Full GC 的全面扫描**：

Full GC 会遍历整个堆内存，标记所有存活对象，并回收所有不可达对象。由于 Full GC 已经进行了全局扫描，卡表的作用被弱化，因为不再需要依赖卡表来定位跨代引用。

**2 卡表的维护**：

尽管在 Full GC 中卡表的作用不明显，但卡表仍然会被维护，以确保在后续的 YGC 中能够正常工作。Full GC 会清理卡表，确保其内容与堆内存的实际情况一致。

**3 性能影响**：

在 Full GC 期间，卡表的维护可能会带来一定的开销，但这种开销通常较小，因为 Full GC 的主要性能瓶颈在于全局扫描和对象回收。

总结来说，在 Full GC 的场景中，卡表的贡献较小，因为 Full GC 已经通过全局扫描处理了所有对象引用。卡表的主要作用仍然体现在 YGC 中，帮助加速跨代引用的处理。

## 结构 2：写屏障

在 HotSpot 虚拟机中，写屏障本质上是引用字段被赋值这个事件的一个环绕切面（Around AOP），即一个引用字段被赋值的前后可以为程序提供额外的动作（比如更新卡表），

写屏障分为：前置写屏障（Pre-Write-Barrier）和后置写屏障（Post-Write-Barrier）2 种类型。

需要注意的是：这里的写屏障和多线程并发中的内存屏障不是一个概念。

CMS 的写屏障可以通俗地理解为一个 “引用关系 观察员”、“引用关系 监督员”、“引用关系  小卫士” 。

内存中有很多对象，这些对象之间会有引用关系。

写屏障的作用就是，在程序修改对象的引用关系时，比如把一个对象的引用从一个对象指向另一个对象，Write-Barrier 写屏障 记录下这些变化。

Write-Barrier  就像是一个 “小监督员”，时刻关注着对象引用的变动情况。

当垃圾回收器进行垃圾回收时，它会参考写屏障记录下的信息，确保不会漏掉任何存活对象，也不会错误地回收还在被使用的对象。

**“写屏障”  用于解决  三色标记法的 漏标、多标 等 问题。**

三色标记法的 漏标问题： 在三色标记过程中，有些本应该 回收的垃圾对象，没有被标记到，最后导致它们没有被回收，浪费了内存空间。

比如说，在标记过程中，一个白色对象被一个灰色对象引用，而理所当然应该被标记成灰色，但由于某些原因，这个引用关系被改变了，导致白色对象没有被标记上灰色，就好像这个 “宝藏” 被错误地遗漏了。

CMS 采用 “写屏障” 技术来解决漏标、多标问题。

简单来说，就是当有对象的引用关系发生变化时，就会通过 AOP 的方式，触发写屏障。

写屏障会检查这个变化是否会导致漏标，比如说，看看新的引用对象， 是不是 灰色、或者黑色。

如果是，就会把相关的白色对象直接标记为灰色，确保它们不会被遗漏。

这就像是有个 “小卫士” 时刻盯着对象引用的变化，一旦发现可能漏标的情况，就马上把对象标记好，防止 “宝藏” 被遗漏。

# CMS 的垃圾回收的触发时机‌

**老年代内存占用达到阈值‌：**

*   默认阈值‌：老年代空间使用率超过 ‌`-XX:CMSInitiatingOccupancyFraction`‌ 参数设定值（默认 92%）时触发初始标记‌。‌
    
*   参数调整‌：若需自定义阈值，需同时启用 ‌`-XX:+UseCMSInitiatingOccupancyOnly`‌ 参数，否则 JVM 可能动态调整触发条件‌。
    

**元数据区（Metaspace）空间不足‌**：当 ‌元数据区内存不足‌（如加载大量类或动态生成类时），可能直接触发 CMS 初始标记‌。

**年轻代晋升失败：**若 ‌年轻代对象晋升到老年代失败‌（老年代剩余空间不足以容纳晋升对象），将强制触发初始标记以回收老年代空间‌。

‌关联参数‌：‌`-XX:+CMSScavengeBeforeRemark`‌ 可配置在初始标记前先执行年轻代 GC，释放空间以降低晋升失败风险‌。

**显式调用垃圾回收‌：** 通过代码 ‌`System.gc()`‌ 显式触发垃圾回收时，若配置了 ‌`-XX:+ExplicitGCInvokesConcurrent`‌ 参数，会直接启动 CMS 初始标记‌。

# CMS 的整体 工作流程

### 按照 7 个阶段划分， cms 基本流程：

前 5 个阶段  是标记阶段

(1) Initial Mark（初始标记）：会 Stop The World

(2) Concurrent Marking（并发标记）

(3) Preclean**（**预清理**）**

(4) Abortable  Preclean（可终止与清理）

(5) Remark（重新标记）：会 Stop The World

后 2 个阶段  是  清  理  阶段

(6) Concurrent  Sweep（并发清除）

(7) Resetting（重置）

整个过程可以抽象成下图：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WOwKYibYDGPpJbjFMjhM8juVEmzrzTNXgePEdBp1nRwq5eOhQn8ejTI4xGUcm4CMmzY5ZhotnRw9Cw/640?from=appmsg)

另外，还有一个 碎片压缩阶段， CMS 退化至 Full GC 时，JVM 默认通过 ‌**Serial Old 收集器**‌ 使用 ‌**标记 - 压缩算法**‌ 处理全堆内存（含新生代与老年代），解决内存碎片问题并压缩空间‌

### 按照 4 个阶段划分， cms 基本流程：

CMS 的整个回收过程可以抽象成下图：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WOwKYibYDGPpJbjFMjhM8juV0rBoZkXwibRWoN4jPGREOfbCAXNPvVrASGf7vpWiaAWhPgu0loapSRaA/640?from=appmsg)

CMS 垃圾收集器通过三色标记算法，实现了垃圾回收线程与用户线程的并发执行，从而极大地降低了系统响应时间，提高了强交互应用程序的体验。

它的运行过程分为 4 个步骤，包括：

**1）初始标记 （CMS initial mark）**

指的是寻找所有被 GCRoots 引用的对象，该阶段需要 STW。仅仅只是标记一下 GCRoots 能直接关联到的对象，并不需要做整个引用的扫描，因此速度很快。

说明：在此阶段，标记所有根对象（GCRoots）为灰色，并将其直接引用的对象标记为灰色。

**2）并发标记 （CMS Concurrent mark）**

指的是对「初始标记阶段」标记的对象进行整个引用链的扫描，该阶段不需要 STW。

对整个引用链做扫描需要花费非常多的时间，因此通过垃圾回收线程与用户线程并发执行，可以降低垃圾回收的时间。这个阶段在整个过程中耗时最长。

这也是 CMS 能极大降低 GC 停顿时间的核心原因，但这也带来了一些问题，即：并发标记的时候，引用可能发生变化，因此可能发生漏标（本应该回收的垃圾没有被回收）和多标（本不应该回收的垃圾被回收）。

说明：这个阶段，灰色对象会被处理，检查它们引用的对象，并将这些对象标记为灰色。这一过程会一直进行，直到没有灰色对象为止。

**3）重新标记 （CMS remark）**

为了修正「并发标记阶段」期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段稍长一些，但远比并发标记的时间短。此阶段也需要 STW。

说明：这个阶段，垃圾收集器会遍历之前标记为灰色的对象，检查它们引用的新对象。这些新对象将被标记为灰色或黑色，表示它们是可达的。如果一个对象已经被标记为黑色，垃圾收集器会跳过对该对象的再次处理，避免多标的情况。

**4）并发清除 (CMS Concurrent Sweep)**

指的是将标记为垃圾的对象（白色的对象）进行清除，该阶段不需要 STW。 在这个阶段，垃圾回收线程与用户线程可以并发执行，因此并不影响用户的响应时间。

**优缺点**

*   优点：并发收集、低停顿。
    
*   缺点: 处理器资源敏感、内存碎片、CMS 无法处理浮动垃圾，
    

内存碎片

CMS 采用的是「标记 - 清除」算法，会产生大量的内存碎片，导致空间不连续，当出现大对象无法找到满足大小并且连续的内存空间时，就会触发下一次垃圾收集，这会导致系统的停顿时间变长。

无法处理浮动垃圾

当 CMS 在进行垃圾回收的时候，应用程序还在不断地产生垃圾，这些垃圾会在 CMS 垃圾回收结束之后产生，这些垃圾就是浮动垃圾，CMS 无法处理这些浮动垃圾，只能在下一次 GC 时清理掉。

### 为什么有的文章 cms 是 4 个阶段， 有的 文章 cms 是 6 个阶段 ?

CMS 垃圾回收器的阶段划分存在差异，主要是因为不同的资料对阶段的划分标准和详细程度有所不同。

有的资料将 CMS 的执行过程简化为四个主要阶段，而另一些资料则更详细地描述了六个阶段，包括预清理和可中断的预清理阶段。

以下是两种划分方式的解释：

### 四个阶段的划分

**1 初始标记（Initial Mark）**：

这个阶段会暂停所有其他线程（STW），标记从 GC Roots 直接可达的老年代对象，以及新生代直接引用的老年代对象。这个过程很快，因为只标记第一层对象。

**2 并发标记（Concurrent Mark）**：

与用户线程并发执行，从初始标记的对象开始，遍历所有其他对象引用，标记所有存活对象。这个阶段耗时较长，但不会暂停用户线程。

**3 重新标记（Remark）**：

再次暂停所有其他线程（STW），重新扫描并发标记阶段可能遗漏的对象，确保所有存活对象都被正确标记。这个阶段的时间相对较短。

**4 并发清除（Concurrent Sweep）**：

与用户线程并发执行，清理所有未被标记的死亡对象，回收它们占用的内存空间。这个阶段也会产生一些新的垃圾（浮动垃圾），需要在下一次垃圾回收时处理。

### 六个阶段的划分

**1 初始标记（Initial Mark）**：

同四个阶段中的初始标记。

**2 并发标记（Concurrent Mark）**：

同四个阶段中的并发标记。

**3 并发预清理（Concurrent Preclean）**：

处理在并发标记阶段由于对象引用变化而产生的脏页（dirty pages），减少重新标记阶段的工作量。

**4 可中断的并发预清理（Concurrent Abortable Preclean）**：

进一步处理新生代指向老年代的新引用，标记可能遗漏的存活对象。这个阶段可以被中断，以确保重新标记阶段的停顿时间尽可能短。

**5 重新标记（Remark）**：

同四个阶段中的重新标记。

**6 并发清除（Concurrent Sweep）**：

同四个阶段中的并发清除。

#### 预清理和可中断的预清理的作用

**1 预清理阶段**：在并发标记阶段，由于对象引用的动态变化，可能会导致一些对象的标记状态不准确。预清理阶段会处理这些变化，标记可能遗漏的存活对象，减少重新标记阶段的工作量。

**2 可中断的预清理阶段**：这个阶段会尽可能多地承担重新标记阶段的工作，通过扫描新生代和处理脏页来标记更多的存活对象。

这个阶段可以被中断，以确保重新标记阶段的停顿时间尽可能短。

总的来说，CMS 垃圾回收器的设计目标是减少停顿时间，提高应用程序的响应速度。

无论是四个阶段还是六个阶段的划分，其核心思想都是通过并发执行来减少对用户线程的影响，同时通过多次标记和清理来确保垃圾回收的准确性。

## 第一个‌阶段：初始标记（Initial Mark）

初始化标记阶段，是 CMS GC 的第一个阶段，也是标记阶段的开始。

### 为什么需要初始标记（Initial Mark）

CMS 的初始标记阶段通过 ‌**极短 STW 确定存活对象起点**‌、‌**处理跨代引用**‌ 和 ‌**初始化卡表**‌，解决了全堆扫描的长停顿问题，并为并发标记提供高效增量扫描的基础‌。该阶段是 CMS 实现低延迟与高准确性的关键设计，确保后续阶段在用户线程并行运行的同时，仍能可靠追踪存活对象‌

### 什么是初始标记（Initial Mark）

‌初始标记（Initial Mark）  主要工作是标记可直达的存活对象。

**‌初始标记（Initial Mark） 主要标记过程**

*   从 GC Roots 遍历可直达的老年代对象；
    
*   遍历被新生代存活对象所引用的老年代对象。
    

**程序执行情况**

*   支持单线程或并行标记。
    
*   发生 stop-the-world，暂停所有应用线程。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WOwKYibYDGPpJbjFMjhM8juVZBWb5k0gxhl0mpS1JVu5NLTwGBWlAyTuYzrUckHedH4rCRPhPPHeGw/640?from=appmsg)

### 初始标记（Initial Mark） 阶段定位与核心任务

‌**定位**‌：Initial Mark 是 CMS 垃圾收集流程的‌**第一个 STW（Stop-The-World）阶段**‌，负责快速标记 GC Roots 直接引用的存活对象，为后续并发标记阶段提供基准‌。

**‌核心任务‌：**

*   扫描并标记所有 ‌**GC Roots 直接可达的对象**‌（如线程栈中的局部变量、静态变量、JNI 引用等）‌。
    
*   记录老年代对象的‌**跨代引用**‌（例如年轻代对象指向老年代的引用），避免后续并发标记阶段因跨代引用导致漏标‌。
    

### 初始标记（Initial Mark） 执行流程‌

**(1) ‌暂停所有应用线程（STW）‌**

暂停用户线程以确保标记过程中对象引用关系不被修改，保证标记结果的准确性‌。

暂停时间极短（通常毫秒级），仅依赖 GC Roots 数量，与堆大小无关‌。

**(2) ‌标记 GC Roots 直接关联对象‌**

通过 ‌**GC Roots Tracing**‌ 算法，遍历根节点直接引用的老年代对象‌。

‌**不遍历对象图**‌（如对象间的间接引用），仅处理直接可达的对象，大幅减少标记时间‌。

**(3) ‌处理跨代引用‌**

记录年轻代到老年代的引用（通过 ‌**Card Table**‌ 或类似机制），确保并发标记阶段能正确处理跨代存活对象‌。

### 初始标记（Initial Mark） 技术实现细节‌

‌**源码实现：**- 在 OpenJDK 中，该阶段入口为 `VM_CMS_Initial_Mark.doit()`，通过 `checkpointRootsInitialWork` 方法完成根对象扫描与标记‌。- 使用 `CLDToOopClosure` 等工具处理类加载器引用，确保静态变量等根对象被完整标记‌。

### 初始标记（Initial Mark）与其他阶段的对比‌

<table><thead><tr><td><span><strong><span leaf="">‌</span><strong><span leaf="">阶段</span></strong><span leaf="">‌</span></strong></span></td><td><span><strong><span leaf="">‌</span><strong><span leaf="">是否 STW</span></strong><span leaf="">‌</span></strong></span></td><td><span><strong><span leaf="">‌</span><strong><span leaf="">主要操作</span></strong><span leaf="">‌</span></strong></span></td><td><span><strong><span leaf="">‌</span><strong><span leaf="">耗时</span></strong><span leaf="">‌</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">‌</span><strong><span leaf="">Initial Mark</span></strong><span leaf="">‌</span></span></section></td><td><section><span><span leaf="">是（短暂）</span></span></section></td><td><section><span><span leaf="">标记 GC Roots 直接关联对象、处理跨代引用</span></span></section></td><td><section><span><span leaf="">极短（毫秒级）‌</span></span></section></td></tr><tr><td><section><span><span leaf="">‌</span><strong><span leaf="">Concurrent Mark</span></strong><span leaf="">‌</span></span></section></td><td><section><span><span leaf="">否</span></span></section></td><td><section><span><span leaf="">遍历对象图，标记所有可达对象</span></span></section></td><td><section><span><span leaf="">较长（依赖堆大小）‌</span></span></section></td></tr><tr><td><section><span><span leaf="">‌</span><strong><span leaf="">Remark</span></strong><span leaf="">‌</span></span></section></td><td><section><span><span leaf="">是</span></span></section></td><td><section><span><span leaf="">修正并发标记期间的引用变动</span></span></section></td><td><section><span><span leaf="">短（略长于初始标记）‌</span></span></section></td></tr></tbody></table>

### 初始标记（Initial Mark）设计意义与局限性

通过极短的 STW 快速完成基准标记，为后续并发阶段奠定基础，‌**最小化整体停顿时间**‌（CMS 核心目标）‌。

**局限性‌：**

*   无法避免 STW，但对标记准确性至关重要‌。
    
*   仅处理直接引用，间接引用需依赖并发标记阶段（可能导致浮动垃圾）‌。
    

‌**总结**‌：CMS 的 Initial Mark 阶段通过极短暂的 STW 完成根对象直接引用标记与跨代引用处理，是 CMS 低延迟设计的关键起点，但需与后续阶段协同确保垃圾回收的完整性‌

‌**‌初始标记（Initial Mark） 阶段是并发标记的起点**‌：

Initial Mark 阶段完成后，CMS 进入 ‌**Concurrent Mark（并发标记）阶段**‌，基于初始标记结果继续遍历对象图‌。

## 第二个阶段：并发标记（Concurrent Mark）

并发标记阶段，是 CMS GC 的第二个阶段。

### 为什么需要 并发标记（Concurrent Mark）

CMS 的并发标记阶段通过 ‌**允许用户线程与 GC 线程并行运行**‌，解决了传统算法全堆标记需长时间 STW 的痛点，同时 ‌**覆盖全堆引用链的可达性分析**‌，并通过卡表机制与后续重新标记阶段协同修正动态变更‌。

这一设计在保障低延迟的前提下，实现了吞吐量与标记准确性的平衡，是 CMS 作为 “Mostly Concurrent” 垃圾回收器的核心能力体现‌

### 什么是 并发标记（Concurrent Mark）

在该阶段，GC 线程和应用线程将并发执行。

也就是说，在第一个阶段（Initial Mark）被暂停的应用线程将恢复运行。

并发标记阶段的主要工作是，通过遍历第一个阶段（Initial Mark）标记出来的存活对象，继续递归遍历老年代，并标记可直接或间接到达的所有老年代存活对象。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WOwKYibYDGPpJbjFMjhM8juVXWzg2fribW26vpN3BH8ep0MX72YJF9nSCJQR4fj9jYKqibmz28Su5MIA/640?from=appmsg)

（Current obj：该对象的引用关系发生变化，对下一个对象的引用被删除。）

由于在并发标记阶段，应用线程和 GC 线程是并发执行的，因此可能产生新的对象或对象关系发生变化，例如：

*   新生代的对象晋升到老年代；
    
*   直接在老年代分配对象；
    
*   老年代对象的引用关系发生变更；
    
*   等等。
    

对于这些对象，需要重新标记以防止被遗漏。

为了提高重新标记的效率，本阶段会把这些发生变化的对象所在的 Card，通过写屏障标识为 Dirty，这样后续就只需要扫描这些 Dirty Card 的对象，从而避免扫描整个老年代。

### 并发标记（Concurrent Mark）阶段定义与核心作用‌

‌**定位**‌：

CMS 垃圾回收流程的‌**第二个阶段**‌，与用户线程‌**并发执行**‌，负责从初始标记的起点出发递归标记所有可达对象，形成完整存活对象图‌。

‌**核心任务**‌：- 遍历堆内存，标记所有从‌**GC Roots 直接关联对象**‌出发的可达对象（即所有存活对象）‌。- 通过‌**写屏障（Write Barrier）**‌ 记录并发期间对象的引用变更（如新增或删除的引用）‌。

### 并发标记（Concurrent Mark）执行特点与机制‌

**并发性**‌：**与应用线程并行运行**‌，不暂停用户线程，极大减少停顿时间‌。

**写屏障：**假设应用线程修改了对象 A 的引用指向对象 B，写屏障会记录这一变更，供后续重新标记阶段处理‌。

**增量标记**‌：采用‌**增量式标记算法**‌，允许标记过程分批次进行，避免长时间占用 CPU 资源‌。

‌**资源敏感**‌：默认启动 ‌**(CPU 数 + 3)/4**‌ 个垃圾回收线程，占用约 25% 的 CPU 资源（当 CPU≥4 时）‌。

### 并发标记（Concurrent Mark）性能影响与挑战‌

‌**浮动垃圾（Floating Garbage）**‌：并发期间用户线程可能产生新垃圾（如对象引用被置空），这些垃圾需等待下次 GC 回收‌。

‌**标记遗漏风险**‌：若用户线程在并发标记期间修改引用关系（如删除引用），可能导致存活对象被误标为垃圾，需通过写屏障和重新标记阶段修正‌。

‌**吞吐量下降**‌：并发占用 CPU 资源，可能降低应用整体吞吐量（如计算密集型任务）‌。

### 并发标记（Concurrent Mark）与其他阶段的协同‌

**依赖初始标记结果**‌：从初始标记阶段记录的 GC Roots 直接关联对象出发，进行全堆遍历‌。

**衔接重新标记阶段**‌：并发标记结束后，进入‌**重新标记（Remark）阶段**‌，通过 STW 暂停修正并发期间的引用变更，确保标记准确性‌。

### 并发标记（Concurrent Mark）优化参数与配置‌

‌**并行度调整**‌：`-XX:ConcGCThreads`：手动指定并发标记线程数，平衡 CPU 资源占用‌。

‌**增量标记控制**‌：`-XX:+CMSScheduleRemarkEdenSizeThreshold`：控制年轻代晋升到老年代的阈值，减少标记压力‌。

## 第三个阶段： 预清理（Preclean）

### 为什么需要预清理（Preclean）？

在并发标记阶段（Concurrent Mark），用户线程仍在运行，可能导致对象引用关系变更。

CMS 通过‌**写屏障（Write Barrier）**‌ 将修改引用的对象所在内存区域（即 512 字节的卡片）标记为 “脏卡”‌。

预清理阶段会扫描这些脏卡，重新标记受影响的存活对象，避免最终标记阶段（Final Remark）因处理大量脏卡而延长 STW 时间‌。

**预清理（Preclean）降低最终标记阶段的负担。**

### 什么是预清理（Preclean）？

在并发预清洗阶段，将会重新扫描前一个阶段标记的 Dirty 对象，并标记被 Dirty 对象直接或间接引用的对象，然后清除 Card 标识。

标记被 Dirty 对象直接或间接引用的对象：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WOwKYibYDGPpJbjFMjhM8juVnlibFib51lTH7LpbF3zS0DCX4SG0icaoqWgIa1NDvZK1h8icrZHaL0Yxdw/640?from=appmsg)

清除 Dirty 对象的 Card 标识：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WOwKYibYDGPpJbjFMjhM8juVSBnn4eL3FtibYzBJGTFJNQMkFJtX1cpTVKiclSLIDVoib930jUyJB6icaQ/640?from=appmsg)

### 预清理（Preclean）‌ 阶段定义与核心作用‌

‌**定位**‌：

CMS 垃圾回收流程的‌**第三个阶段**‌，在并发标记之后、重新标记之前执行，‌**与应用线程并发运行**‌，负责处理并发标记期间因用户线程操作产生的引用变更，减少重新标记阶段的 STW 时间‌。

‌**核心任务**‌：- **修正标记遗漏**‌：通过写屏障（Write Barrier）记录的引用变更（如对象引用新增或删除），更新标记状态‌。- ‌**处理跨代引用**‌：扫描‌**卡表（Card Table）**‌ 中记录的跨代引用（如年轻代对象晋升到老年代）‌。

### 预清理（Preclean）‌执行机制与关键操作‌

‌**增量修正标记**‌：遍历写屏障记录的‌**脏页（Dirty Card）**‌，重新标记并发标记期间被修改的引用关系，避免存活对象被误标为垃圾‌。_示例_：若用户线程在并发标记阶段将对象 A 的引用指向对象 B，预清理阶段会重新标记 B 为存活‌。

**跨代引用处理**‌：使用卡表快速定位老年代中被年轻代存活对象引用的区域，避免重新标记阶段全堆扫描‌。

‌**年轻代存活对象晋升**‌：若开启 ‌-XX:+CMSScavengeBeforeRemark，预清理阶段可能触发年轻代 GC，将存活对象晋升至老年代，减少重新标记阶段的跨代引用复杂度‌。

### 预清理（Preclean）性能优化与参数配置‌

‌**并发性**‌：与应用线程并行执行，无需 STW 暂停，但占用部分 CPU 资源（默认由 ‌-XX:ConcGCThreads 控制并发线程数）‌。

‌**预清理阈值控制**‌：‌-XX:CMSPrecleanThreshold：设置预清理阶段的最大执行时间或迭代次数，防止因引用变更过多导致预清理阶段过长‌。

‌**卡表优化**‌：‌-XX:+CMSCleanOnEnter：在进入预清理阶段前强制清理卡表，减少脏页数量‌。

### 预清理（Preclean）对后续阶段的影响‌

‌**降低重新标记压力**‌：

通过预清理阶段的增量修正，将重新标记阶段的 STW 时间缩短至 10~50ms（取决于引用变更频率）‌。

‌**减少浮动垃圾**‌：

及时处理并发标记期间的引用变更，降低因标记遗漏导致的 “浮动垃圾” 比例，避免触发 Concurrent Mode Failure‌。

## 第四个阶段： 可终止预清理（Abortable Preclean）

### 为什么需要 可终止预清理（Abortable Preclean）？

可终止预清理，就是边巡逻变扫描，达到阈值终止，优化重新标记阶段的效率

### 什么是 可终止预清理（Abortable Preclean）

本阶段尽可能承担更多的并发预处理工作，从而减轻在 Remark  Remark 阶段的 stop-the-world。

在该阶段，主要循环的做两件事：

*   处理 From 和 To 区的对象，标记可达的老年代对象；
    
*   和上一个阶段一样，扫描处理 Dirty Card 中的对象。
    

具体执行多久，取决于许多因素，满足其中一个条件将会中止运行：

*   执行循环次数达到了阈值；
    
*   执行时间达到了阈值；
    
*   新生代 Eden 区的内存使用率达到了阈值。
    

### ‌可终止预清理（Abortable Preclean）阶段定义与核心任务

‌**定位**‌：

CMS 垃圾回收流程的‌**第四个阶段**‌，在预清理阶段之后、重新标记之前执行，‌**与用户线程并发运行**‌，目标是进一步减少重新标记阶段的 STW 时间，并处理年轻代晋升对老年代的影响‌。

‌**核心任务**‌：- **处理年轻代存活对象的引用变更**‌：扫描年轻代（From/To Survivor 区）中存活对象对老年代的引用，标记相关卡表（Card Table）为脏页（Dirty Card）‌。- ‌**等待一次 Minor GC 触发**‌：通过触发年轻代 GC，减少存活对象晋升到老年代的概率，从而降低重新标记阶段的跨代引用复杂度‌。

### ‌可终止预清理（Abortable Preclean）‌执行机制与关键操作‌

‌**增量处理跨代引用**‌：

遍历年轻代存活对象，若其引用老年代对象，则将该老年代区域对应的卡表标记为脏页，后续重新标记阶段仅需扫描脏页区域‌。

_示例_：若年轻代对象 A 存活且引用老年代对象 B，则 B 所在卡页被标记为脏页‌。

**‌动态终止机制‌**：

*   ‌**时间阈值**‌：默认最长执行 5 秒（通过 ‌-XX:CMSMaxAbortablePrecleanTime 调整），超时后强制终止并进入重新标记阶段‌。
    
*   ‌**空间阈值**‌：当 Eden 区使用率超过 ‌-XX:CMSScheduleRemarkEdenPenetration 设定值（默认 50%）时，提前终止以触发重新标记‌。
    

### ‌可终止预清理（Abortable Preclean）‌‌ 性能优化与挑战‌

‌**降低重新标记阶段的 STW 时间**‌：

通过提前处理跨代引用和脏页，将重新标记阶段的 STW 时间缩短至 10~50ms（取决于脏页数量）‌。

‌**资源消耗与延迟风险**‌：

若脏页过多或年轻代晋升频繁，可能导致可终止预清理阶段耗时过长（如超过 5 秒），进而触发 ‌**Concurrent Mode Failure**‌（强制切换为 Serial Old GC）‌。

### ‌可终止预清理（Abortable Preclean）‌参数配置与调优‌

*   ‌-XX:CMSMaxAbortablePrecleanTime：控制可终止预清理阶段的最大执行时间（默认 5 秒），避免无限等待用户线程的 Minor GC‌。
    
*   ‌-XX:CMSScheduleRemarkEdenPenetration：设置 Eden 区使用率阈值（默认 50%），当 Eden 区占用超过该阈值时强制终止预清理，确保及时进入重新标记阶段‌。
    

### 预清理和可终止预清理区别：

预清理‌：必须干完活，专门清理已知的脏区域。

可终止预清理‌：干到一半可能跑路，主要对付新增的脏区域

类比‌：- 预清理像 “扫固定区域的地”，必须扫完；- 可终止预清理像 “边巡逻边扫新掉落的垃圾”，但随时可能下班。

## 第五个阶段： 重新标记（Remark）

### 为什么需要 重新标记（Remark）？

预清理阶段也是并发执行的，并不一定是所有存活对象都会被标记，因为在并发标记的过程中对象及其引用关系还在不断变化中。

因此，需要有一个 stop-the-world 的阶段来完成最后的标记工作，这就是重新标记阶段（CMS 标记阶段的最后一个阶段）。

重新标记（Remark）主要目的： 是重新扫描之前并发处理阶段的所有残留更新对象。

### 什么是 重新标记（Remark）？

CMS 的重新标记阶段通过 ‌**短暂 STW 修正并发标记的引用变更**‌，解决了三色标记法的并发缺陷，避免了漏标和错标问题‌。

重新标记（Remark） 核心价值在于 ‌**以极短停顿换取标记准确性**‌，确保后续并发清除阶段仅回收已确认的垃圾对象，是 CMS 实现低延迟与高可靠性的关键设计‌。

主要工作：

*   遍历新生代对象，重新标记；（新生代会被分块，多线程扫描）
    
*   根据 GC Roots，重新标记；
    
*   遍历老年代的 Dirty Card，重新标记。这里的 Dirty Card，大部分已经在 Preclean 阶段被处理过了。
    

### 重新标记（Remark）阶段定义与核心作用‌

‌**定位**‌：CMS 垃圾回收流程的‌**第五个阶段**‌，在可终止预清理阶段后执行，‌**完全 STW（Stop-The-World）暂停**‌，负责修正并发标记期间因用户线程操作导致的标记遗漏，确保存活对象标记的准确性‌。

‌**核心任务**‌：- **修正并发标记阶段的引用变更**‌：通过‌**增量更新（Incremental Update）**‌ 或‌**SATB（Snapshot-At-The-Beginning）**‌ 机制，处理并发期间新增或删除的引用关系‌。- ‌**处理跨代引用**‌：扫描卡表（Card Table）记录的脏页，标记年轻代存活对象对老年代的引用‌。

### ‌重新标记（Remark）执行机制与关键操作‌

**STW 暂停**‌：完全暂停用户线程，避免标记过程中引用关系再次变更，确保标记结果的确定性‌。

**增量更新与 SATB 机制**‌：

*   ‌**增量更新**‌：追踪并发标记期间新增的引用（如用户线程创建的新对象），重新标记相关对象为存活‌。
    
*   ‌**SATB**‌：基于初始标记的快照，修正被删除的引用（如用户线程断开引用导致的对象不可达）‌。
    

‌**卡表脏页处理**‌：扫描卡表中记录的脏页区域（即年轻代对象对老年代的引用），标记相关老年代对象为存活‌。

### 重新标记（Remark） 性能影响与挑战‌

‌**STW 时间可控性**‌：- 重新标记阶段的停顿时间通常为‌**10~50ms**‌，取决于脏页数量和引用变更频率‌。- _优化示例_：若启用 ‌-XX:+CMSScavengeBeforeRemark，在重新标记前触发一次年轻代 GC，减少跨代引用数量，缩短 STW 时间‌。

‌**资源敏感性与并发失败风险**‌：若脏页过多或引用变更频繁，可能导致 STW 时间过长，触发 ‌**Concurrent Mode Failure**‌（强制切换为 Serial Old GC）‌。

### ‌重新标记（Remark） 与其他阶段的协同‌

**依赖可终止预清理结果**‌：基于可终止预清理阶段处理的脏页和年轻代晋升结果，减少重新标记阶段的扫描范围‌。

‌**衔接并发清除阶段**‌：重新标记完成后，进入‌**并发清除（Concurrent Sweep）阶段**‌，回收未被标记的垃圾对象‌。

### ‌重新标记（Remark） 优化参数与配置‌

*   ‌`-XX:+CMSParallelRemarkEnabled`‌：启用多线程并行重新标记，提升标记效率（默认开启）‌。
    
*   ‌`-XX:CMSScheduleRemarkEdenPenetration`‌：控制 Eden 区使用率阈值（默认 50%），若超过该值则提前终止预清理阶段，避免重新标记阶段处理过多年轻代引用‌。
    

## ‌第六个阶段： 并发清理（Concurrent Sweep）

并发清理阶段，主要工作是清理所有未被标记的死亡对象，回收被占用的空间。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WOwKYibYDGPpJbjFMjhM8juV2fjj86dy0oQq7lVjNLnKvibASxIupCCmOoHR9SFGx89L2z7Ksqsm4Sg/640?from=appmsg)

### 并发清理（Concurrent Sweep）阶段定义与核心作用‌

‌**定位**‌：CMS 垃圾回收流程的‌**第六个阶段**‌，在重新标记阶段后执行，‌**与用户线程并发运行**‌，负责回收未被标记的垃圾对象，释放老年代空间，同时允许用户线程继续分配新对象‌。

‌**核心任务**‌：- ‌**清理垃圾对象**‌：遍历老年代，回收未被标记（即不可达）的对象内存‌。- ‌**处理浮动垃圾**‌：允许用户线程在清理阶段继续修改引用关系，新产生的垃圾（浮动垃圾）将在下一次 GC 中处理‌。

### ‌并发清理（Concurrent Sweep）执行机制与关键操作‌

‌**并发清理流程**‌：- ‌**并行线程**‌：默认由 ‌-XX:ParallelGCThreads 控制清理线程数，与应用线程并发执行‌。- ‌**内存回收**‌：采用‌**标记 - 清除（Mark-Sweep）**‌ 算法，直接释放未被标记的内存块，不进行内存整理（导致内存碎片）‌。

‌**新对象处理**‌：在清理阶段新创建的对象会被直接标记为存活（黑色），避免被误清理‌。

### ‌并发清理（Concurrent Sweep）性能影响与挑战‌

**无 STW 停顿**‌：与应用线程并发执行，不强制暂停用户线程，但占用部分 CPU 资源‌。

‌**内存碎片问题**‌：- 标记 - 清除算法不整理内存，可能导致老年代空间碎片化，触发 Full GC（Serial Old GC）进行压缩整理‌。- _优化示例_：启用 ‌-XX:+UseCMSCompactAtFullCollection 在 Full GC 时压缩内存，或设置 ‌-XX:CMSFullGCsBeforeCompaction 控制压缩频率‌。

**资源竞争风险**‌：若用户线程频繁分配大对象或老年代碎片严重，可能导致并发清理阶段无法及时完成，触发 ‌**Concurrent Mode Failure**‌‌。

### ‌并发清理（Concurrent Sweep）参数配置与优化‌

*   ‌`-XX:+CMSParallelSweepEnabled`‌：启用多线程并行清理（默认开启），提升清理效率‌。
    
*   ‌`-XX:CMSInitiatingOccupancyFraction`‌：设置老年代空间占用阈值（默认 68%），触发 CMS 回收以避免 Full GC‌。
    

### ‌并发清理（Concurrent Sweep）与其他阶段的协同‌

**依赖重新标记结果**‌：基于重新标记阶段的精确标记结果，确定需要清理的垃圾对象‌。

**衔接并发重置阶段**‌：清理完成后进入‌**并发重置（Concurrent Reset）阶段**‌，重置 CMS 内部数据结构，准备下一次 GC‌。

## 第七个阶段：  ‌重置（Reset）阶段

并发重置阶段，将清理并恢复在 CMS GC 过程中的各种状态，重新初始化 CMS 相关数据结构，为下一个垃圾收集周期做好准备。

### 为什么需要  重置（Reset）阶段？

CMS（Concurrent Mark Sweep）垃圾回收器的重置（Reset）阶段是其回收过程中的一个重要步骤，主要目的是为下一轮垃圾回收做好准备。

以下是重置阶段的主要作用和原因：

**1 清理标记信息**

在 CMS 垃圾回收器的标记阶段（包括初始标记、并发标记和重新标记），会对对象进行标记以确定哪些对象是存活的，哪些是垃圾对象。这些标记信息存储在对象的标记字（mark word）中。在重置阶段，需要清理这些标记信息，将对象的状态恢复到未标记的状态，以便在下一轮垃圾回收中能够正确地进行标记。

**2 重置卡表**

CMS 垃圾回收器使用卡表（Card Table）来记录老年代中对象的引用变化情况。在并发标记和重新标记阶段，卡表会被更新以记录对象引用的变化。在重置阶段，需要重置卡表，将所有卡标记为干净（clean）状态，表示没有发生过引用变化。这样在下一轮垃圾回收中，卡表能够准确地记录新的引用变化。

**3 重置其他辅助数据结构**

除了卡表和对象的标记信息，CMS 垃圾回收器可能还会使用其他辅助数据结构来支持垃圾回收过程，如增量更新队列等。在重置阶段，这些辅助数据结构也需要被清理和重置，以确保下一轮垃圾回收的正确性和效率。

**4  统计和分析**

在重置阶段，CMS 垃圾回收器还可以进行一些统计和分析工作，例如计算本次垃圾回收的效率、记录回收过程中的一些关键指标等。这些信息可以用于优化后续的垃圾回收策略，提高整体的垃圾回收性能。

**5  准备下一轮垃圾回收**

重置阶段是垃圾回收周期的结束，同时也是下一轮垃圾回收的开始准备阶段。

通过重置各种数据结构和状态，CMS 垃圾回收器能够以一个干净的状态开始下一轮的垃圾回收过程，确保每次垃圾回收都能够正确、高效地执行。

总结来说，CMS 垃圾回收器的重置阶段是为了清理和重置标记信息、卡表以及其他辅助数据结构，为下一轮垃圾回收做好准备，确保垃圾回收过程的正确性和高效性。

### ‌重置（Reset）阶段 阶段定义与核心作用‌

**定位**‌：CMS 垃圾回收流程的‌**第七个阶段**‌，在并发清理阶段后执行，‌**与用户线程并发运行**‌，负责重置 CMS 内部数据结构，为下一次垃圾回收周期做准备‌。

**核心任务**‌：

*   **重置标记状态**‌：清除所有对象的标记位（如三色标记法中的颜色标记），恢复内存区域的初始状态‌。
    
*   ‌**清理内部数据结构**‌：重置卡表（Card Table）、标记位图（BitMap）等辅助数据结构，释放本次 GC 周期占用的资源‌。
    

### ‌重置（Reset）阶段 执行机制与关键操作‌

**并发执行**‌：重置阶段‌**无需暂停用户线程**‌，与应用线程并行运行，仅占用少量 CPU 资源‌。

**内部操作**‌：

*   ‌**标记位图清零**‌：将对象的三色标记（黑 / 灰 / 白）状态全部重置为初始值（白色）‌。
    
*   ‌**卡表清理**‌：清理由并发标记阶段记录的脏页（Dirty Card）信息，确保下一次 GC 周期从干净状态开始‌。
    

### ‌重置（Reset）阶段 性能影响与挑战‌

**资源消耗低**‌：重置操作仅涉及元数据清理，不涉及对象内存操作，执行时间极短（通常毫秒级）‌。

‌**内存一致性保障**‌：若用户线程在重置阶段频繁修改引用关系，可能导致下次 GC 周期的初始标记阶段需处理更多脏页，但 CMS 通过卡表机制动态记录变更，避免数据遗漏‌。

### ‌重置（Reset）阶段 与其他阶段的协同‌

**衔接下一次 GC 周期**‌：重置完成后，CMS 进入‌**初始标记（Initial Mark）阶段**‌，开始新的回收周期‌。

**依赖并发清理结果**‌：仅当并发清理阶段完成老年代垃圾回收后，重置阶段才能安全重置数据结构，避免残留标记干扰后续流程‌。

# CMS 优缺点分析

CMS 作为早期低延迟垃圾收集器的代表，通过并发标记 / 清除机制降低了 STW 时间，但内存碎片、CPU 资源占用等问题限制了其长期适用性‌35。在 ‌**JDK 14+ 环境**‌中，‌**G1**‌ 和 ‌**ZGC**‌ 凭借更优的压缩算法、内存管理及低延迟特性，已成为主流选择‌67。若需沿用 CMS，需严格监控碎片和并发失败率，并结合 `-XX:+CMSScavengeBeforeRemark` 等参数优化性能‌。

## ‌核心优点‌

**低延迟设计**‌

*   ‌**并发标记与清除**‌：CMS 的 ‌**Concurrent Mark**‌（并发标记）和 ‌**Concurrent Sweep**‌（并发清除）阶段允许垃圾回收线程与用户线程并行工作，显著减少 ‌**STW（Stop-The-World）停顿时间**‌，适合对延迟敏感的应用（如实时系统）‌。
    
*   ‌**可控停顿**‌：通过参数 `-XX:MaxGCPauseMillis` 可设定最大停顿时间阈值，优先保证用户体验‌。
    

‌**分代收集优化**‌- 针对老年代设计，结合年轻代的 ‌**ParNew**‌ 收集器，实现分代垃圾回收，降低整体回收压力‌。

## ‌主要缺点‌

**内存碎片问题**‌

*   ‌**标记 - 清除算法缺陷**‌：CMS 采用标记 - 清除算法，不压缩内存，导致老年代产生大量内存碎片，可能触发频繁 ‌**Full GC**‌（需启用 `-XX:+UseCMSCompactAtFullCollection` 参数手动整理碎片）‌。
    

**CPU 资源敏感**‌

*   ‌**并发阶段占用资源**‌：并发标记和清除阶段需占用约 ‌**25%~30% CPU 资源**‌（默认线程数公式：`(CPU核数 +3)/4`），在高并发或低配环境下可能导致应用吞吐量下降‌。
    

**浮动垃圾处理不足**‌

*   ‌**并发期间新垃圾产生**‌：在并发标记阶段，用户线程可能继续创建新对象（称为 ‌**浮动垃圾**‌），CMS 无法及时回收，需依赖下次 GC 处理‌。
    
*   ‌**Concurrent Mode Failure 风险**‌：若老年代预留空间不足（默认阈值 `-XX:CMSInitiatingOccupancyFraction=92%`），可能触发 ‌**Full GC**‌（退化为 ‌**Serial Old**‌ 收集器），导致长时间 STW‌。
    

**维护与兼容性**‌

*   ‌**已淘汰技术**‌：CMS 自 ‌**JDK 9 起被标记为废弃**‌，‌**JDK 14 中彻底移除**‌，不再受官方支持，仅适用于老旧系统维护‌。
    

## ‌适用场景与替代方案‌

<table><thead><tr><td><span><strong><span leaf="">‌</span><strong><span leaf="">场景</span></strong><span leaf="">‌</span></strong></span></td><td><span><strong><span leaf="">‌</span><strong><span leaf="">CMS 适用性</span></strong><span leaf="">‌</span></strong></span></td><td><span><strong><span leaf="">‌</span><strong><span leaf="">现代替代方案</span></strong><span leaf="">‌</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">低延迟需求（如实时交易系统）</span></span></section></td><td><section><span><span leaf="">有限适用（需容忍碎片和 CPU 开销）</span></span></section></td><td><section><span><span leaf="">‌</span><strong><span leaf="">ZGC</span></strong><span leaf="">‌（亚毫秒级延迟）</span></span></section></td></tr><tr><td><section><span><span leaf="">大堆内存应用</span></span></section></td><td><section><span><span leaf="">不推荐（碎片问题加剧）</span></span></section></td><td><section><span><span leaf="">‌</span><strong><span leaf="">G1</span></strong><span leaf="">‌（分区压缩，可控延迟）</span></span></section></td></tr><tr><td><section><span><span leaf="">老旧系统维护</span></span></section></td><td><section><span><span leaf="">可能适用（需兼容性验证）</span></span></section></td><td><section><span><span leaf="">升级至 JDK 17+ 并迁移至 G1/ZGC</span></span></section></td></tr></tbody></table>

# CMS 与 G1 对比分析

CMS 以低延迟为核心优势但存在内存碎片和淘汰风险，G1 通过分区模型和可控延迟成为现代应用主流选择，尤其在 JDK 9+ 环境中‌。

### ‌1. 工作机制对比‌

<table><thead><tr><td><span><strong><span leaf="">‌</span><strong><span leaf="">特性</span></strong><span leaf="">‌</span></strong></span></td><td><span><strong><span leaf="">‌</span><strong><span leaf="">CMS</span></strong><span leaf="">‌</span></strong></span></td><td><span><strong><span leaf="">‌</span><strong><span leaf="">G1</span></strong><span leaf="">‌</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">核心算法</span></span></section></td><td><section><span><span leaf="">标记 - 清除（产生内存碎片）‌</span></span></section></td><td><section><span><span leaf="">标记 - 整理（复制算法，减少碎片）‌</span></span></section></td></tr><tr><td><section><span><span leaf="">‌分代管理</span></span></section></td><td><section><span><span leaf="">分代收集（老年代专用，需搭配 ParNew 收集年轻代）‌</span></span></section></td><td><section><span><span leaf="">分区模型（将堆划分为多个 Region，独立管理新生代 / 老年代）‌</span></span></section></td></tr><tr><td><section><span><span leaf="">‌阶段划分</span></span></section></td><td><section><span><span leaf="">5 阶段：初始标记（STW）→ 并发标记 → 重新标记（STW）→ 并发清理 → 重置‌</span></span></section></td><td><section><span><span leaf="">4 阶段：初始标记（STW）→ 并发标记 → 最终标记（STW）→ 筛选回收（STW）‌</span></span></section></td></tr><tr><td><section><span><span leaf="">‌内存整理</span></span></section></td><td><section><span><span leaf="">不整理内存，依赖&nbsp;</span><code><span leaf="">UseCMSCompactAtFullCollection</span></code><span leaf="">&nbsp;参数触发 Full GC 压缩‌</span></span></section></td><td><section><span><span leaf="">每次回收后自动整理 Region，避免碎片‌</span></span></section></td></tr></tbody></table>

### ‌2. 内存管理与碎片处理‌

CMS‌：- 采用标记 - 清除算法，回收后不压缩内存，导致老年代内存碎片累积，可能频繁触发 Full GC（使用 Serial Old 收集器整理，长时间 STW）‌。- 需手动设置 `CMSInitiatingOccupancyFraction` 预留空间（默认 92%），避免并发模式失败‌。

G1‌：- 将堆划分为多个大小固定的 Region（默认 2048 个），优先回收垃圾比例高的 Region（Garbage-First 策略）‌。- 通过复制算法整理内存，避免碎片问题，适合大堆内存场景‌。

### ‌3. 性能特点对比‌

<table><thead><tr><td><span><strong><span leaf="">‌</span><strong><span leaf="">指标</span></strong><span leaf="">‌</span></strong></span></td><td><span><strong><span leaf="">‌</span><strong><span leaf="">CMS</span></strong><span leaf="">‌</span></strong></span></td><td><span><strong><span leaf="">‌</span><strong><span leaf="">G1</span></strong><span leaf="">‌</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">‌</span><strong><span leaf="">延迟</span></strong><span leaf="">‌</span></span></section></td><td><section><span><span leaf="">低（STW 仅发生在初始标记和重新标记阶段，毫秒级）‌</span></span></section></td><td><section><span><span leaf="">可控延迟（通过&nbsp;</span><code><span leaf="">MaxGCPauseMillis</span></code><span leaf="">&nbsp;参数设定目标停顿时间，通常 10-200ms）‌</span></span></section></td></tr><tr><td><section><span><span leaf="">‌</span><strong><span leaf="">吞吐量</span></strong><span leaf="">‌</span></span></section></td><td><section><span><span leaf="">较低（并发标记 / 清理占用 CPU 资源，默认线程数&nbsp;</span><code><span leaf="">(CPU核数+3)/4</span></code><span leaf="">）‌</span></span></section></td><td><section><span><span leaf="">较高（通过 Region 分区和并行回收优化吞吐量）‌</span></span></section></td></tr><tr><td><section><span><span leaf="">‌</span><strong><span leaf="">内存占用</span></strong><span leaf="">‌</span></span></section></td><td><section><span><span leaf="">较低（仅需少量元数据）‌</span></span></section></td><td><section><span><span leaf="">较高（需维护 Region 元数据，占用约 10%~20% 额外内存）‌</span></span></section></td></tr></tbody></table>

### ‌4. 适用场景与限制‌

CMS 适用场景‌：- 对延迟敏感的老年代回收（如实时交易系统），但需容忍内存碎片和 CPU 资源竞争‌。- ‌**已淘汰**‌：JDK 9 标记废弃，JDK 14 移除，仅限老旧系统维护‌。

G1 适用场景‌：- 大堆内存（4GB 以上）且需平衡吞吐量与延迟（如大数据处理、服务端应用）‌。- JDK 9+ 默认垃圾收集器，适合长期运行项目‌。

### ‌5. 关键缺陷对比‌

<table><thead><tr><td><span><strong><span leaf="">‌</span><strong><span leaf="">问题</span></strong><span leaf="">‌</span></strong></span></td><td><span><strong><span leaf="">‌</span><strong><span leaf="">CMS</span></strong><span leaf="">‌</span></strong></span></td><td><span><strong><span leaf="">‌</span><strong><span leaf="">G1</span></strong><span leaf="">‌</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">‌</span><strong><span leaf="">内存碎片</span></strong><span leaf="">‌</span></span></section></td><td><section><span><span leaf="">严重（需 Full GC 手动整理，引发长停顿）‌</span></span></section></td><td><section><span><span leaf="">无（自动整理 Region）‌</span></span></section></td></tr><tr><td><section><span><span leaf="">‌</span><strong><span leaf="">浮动垃圾</span></strong><span leaf="">‌</span></span></section></td><td><section><span><span leaf="">并发阶段可能产生新垃圾，需预留空间避免 Concurrent Mode Failure‌16</span></span></section></td><td><section><span><span leaf="">通过 SATB 算法解决漏标问题，浮动垃圾影响较小‌</span></span></section></td></tr><tr><td><section><span><span leaf="">‌</span><strong><span leaf="">CPU 资源占用</span></strong><span leaf="">‌</span></span></section></td><td><section><span><span leaf="">高（并发阶段与用户线程竞争）‌</span></span></section></td><td><section><span><span leaf="">较低（并行回收优化资源分配）‌</span></span></section></td></tr></tbody></table>

# CMS 的垃圾回收 调优

CMS 调优需围绕‌降低 STW 时间‌、‌控制内存碎片‌及‌预防并发失败‌三大核心目标展开，结合场景特性选择参数组合。关键工具链（如 jstat、VisualVM）与监控体系（Prometheus）是调优基础，而参数动态调整（如阈值控制、线程数优化）与场景适配（高并发 / 大内存）则是成败关键‌。

## 调优工具与监控手段‌

‌JVM 内置工具‌- ‌`jstat`‌：实时监控 GC 频率、各代内存使用率及耗时，支持动态观测 CMS 各阶段执行情况（如`jstat -gcutil <pid> 1000`）‌。- ‌`jmap`‌与‌`jstack`‌：生成堆转储与线程快照，分析内存泄漏或线程阻塞问题‌。

‌可视化工具‌- ‌VisualVM‌：图形化展示堆内存、GC 活动及线程状态，支持插件扩展（如 GC 日志分析）‌。- ‌Arthas‌：实时诊断工具，提供内存热更新、GC 监控及线程级性能分析‌。

‌第三方监控系统‌- ‌Prometheus + Grafana‌：通过 JMX Exporter 采集 JVM 指标，构建 CMS 关键参数（如 STW 时间、老年代碎片率）的实时监控面板‌。

## 关键调优动作与参数调整‌

<table><thead><tr><td><span><strong><span leaf="">‌调优目标‌</span></strong></span></td><td><span><strong><span leaf="">‌关键参数与操作‌</span></strong></span></td><td><span><strong><span leaf="">‌作用与场景‌</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">‌减少 STW 时间‌</span></span></section></td><td><section><span><code><span leaf="">-XX:+CMSParallelRemarkEnabled</span></code><span leaf=""><br></span><span leaf="">（启用并行重新标记）‌</span></span></section></td><td><section><span><span leaf="">通过多线程缩短重新标记阶段停顿（默认开启）</span></span></section></td></tr><tr><td><section><span><code><span leaf="">-XX:+CMSScavengeBeforeRemark</span></code><span leaf=""><br></span><span leaf="">（重新标记前触发 Young GC）‌</span></span></section></td><td><section><span><span leaf="">减少跨代引用，降低重新标记阶段扫描范围（适用于频繁晋升场景）</span></span></section></td><td><section><span leaf=""><br></span></section></td></tr><tr><td><section><span><span leaf="">‌控制并发失败风险‌</span></span></section></td><td><section><span><code><span leaf="">-XX:CMSInitiatingOccupancyFraction=70</span></code><span leaf=""><br></span><span leaf="">（设置老年代触发阈值）‌</span></span></section></td><td><section><span><span leaf="">预留足够空间避免并发清理阶段内存耗尽（推荐值 70%-80%）</span></span></section></td></tr><tr><td><section><span><code><span leaf="">-XX:+UseCMSInitiatingOccupancyOnly</span></code><span leaf=""><br></span><span leaf="">（强制按阈值触发 CMS）‌</span></span></section></td><td><section><span><span leaf="">防止 JVM 动态调整阈值导致意外 Full GC</span></span></section></td><td><section><span leaf=""><br></span></section></td></tr><tr><td><section><span><span leaf="">‌缓解内存碎片‌</span></span></section></td><td><section><span><code><span leaf="">-XX:+UseCMSCompactAtFullCollection</span></code><span leaf=""><br></span><span leaf="">（Full GC 时压缩内存）‌</span></span></section></td><td><section><span><span leaf="">减少内存碎片，但会延长 Full GC 停顿时间</span></span></section></td></tr><tr><td><section><span><code><span leaf="">-XX:CMSFullGCsBeforeCompaction=4</span></code><span leaf=""><br></span><span leaf="">（每 4 次 Full GC 压缩一次）‌</span></span></section></td><td><section><span><span leaf="">平衡压缩频率与性能损耗</span></span></section></td><td><section><span leaf=""><br></span></section></td></tr><tr><td><section><span><span leaf="">‌优化并发标记效率‌</span></span></section></td><td><section><span><code><span leaf="">-XX:ConcGCThreads=4</span></code><span leaf=""><br></span><span leaf="">（设置并发标记线程数）‌</span></span></section></td><td><section><span><span leaf="">根据 CPU 核心数调整（建议为物理核心数的 25%-50%）</span></span></section></td></tr><tr><td><section><span><code><span leaf="">-XX:+CMSParallelInitialMarkEnabled</span></code><span leaf=""><br></span><span leaf="">（启用初始标记并行化）‌</span></span></section></td><td><section><span><span leaf="">加速初始标记阶段（JDK8 + 默认开启）</span></span></section></td><td><section><span leaf=""><br></span></section></td></tr></tbody></table>

## 场景化调优建议‌

**‌高并发低延迟场景‌**

‌核心矛盾‌：降低重新标记阶段 STW 时间，避免因跨代引用过多导致停顿激增。

优化方案‌：

*   启用`-XX:+CMSScavengeBeforeRemark`，降低重新标记阶段扫描范围‌。
    
*   增大 Survivor 区（`-XX:SurvivorRatio=6`），减少对象过早晋升至老年代‌。
    
*   限制大对象分配（`-XX:PretenureSizeThreshold=1M`），避免直接进入老年代‌。
    

**‌大内存应用场景（堆 > 16GB）‌**

‌核心矛盾‌：内存碎片导致 Full GC 频繁触发，并发清理阶段易超时。

优化方案‌：- 启用`-XX:+UseCMSCompactAtFullCollection`并降低压缩频率（`-XX:CMSFullGCsBeforeCompaction=5`）‌。- 使用`-XX:+UseCMSInitiatingOccupancyOnly`固定触发阈值，避免动态调整失效‌。- 监控碎片率（`jstat -gccapacity`），定期触发 Full GC 维护内存连续性‌。

**‌实时性敏感系统（如金融交易）‌**

核心矛盾‌：STW 停顿时间波动需严格控制在阈值内。

‌优化方案‌：- 启用增量模式（`-XX:+CMSIncrementalMode`），分片执行 GC 任务（牺牲吞吐换延迟）‌。- 设置最大停顿时间（`-XX:MaxGCPauseMillis=50`），JVM 自动优化 GC 行为‌。

**‌混合负载系统（兼顾吞吐与延迟）‌**

‌核心矛盾‌：平衡 CMS 并发资源占用与业务线程性能。

优化方案‌：- 限制并发线程数（`-XX:ConcGCThreads=2`），减少 CPU 争抢‌。- 启用自适应策略（`-XX:+CMSAdaptiveSizePolicy`），JVM 动态调整各代比例‌。

## 调优验证与风险控制‌

基准测试验证：使用‌`gcviewer`‌分析 GC 日志，验证 STW 时间、吞吐量及内存碎片率是否达标‌

‌灰度发布策略‌：分批部署参数变更，通过 APM 工具（如 SkyWalking）观测业务指标（如 TP99 延迟）‌

‌熔断机制‌：设置`-XX:+UseGCOverheadLimit`，在 GC 耗时超过 98% 时主动抛出 OOM，防止系统僵死‌

# Concurrent Mode Failure 是什么？

Concurrent Mode Failure 是 CMS 垃圾回收器在并发执行阶段因内存不足而触发的关键问题。

Concurrent Mode Failure 的核心触发条件是‌**老年代在并发回收阶段预留空间不足**‌，可能由内存碎片、对象晋升过快或分配速率过高导致。

需结合阈值调整、碎片压缩及内存监控综合优化‌。

### Concurrent Mode Failure 触发条件

Concurrent Mode Failure 是 CMS 垃圾回收器 关键问题，其触发条件主要包括以下场景：

#### 1 老年代预留空间不足

CMS 启动并发回收时，需‌**预留部分内存供用户线程分配新对象**‌。若预留空间被占满，无法继续分配对象，则会触发该失败‌。

**默认阈值**‌：老年代占用达 ‌**92%**‌ 时触发 CMS 回收（通过 `-XX:CMSInitiatingOccupancyFraction` 可调整阈值）‌。

**典型表现**‌：用户线程在并发标记 / 清理阶段持续分配对象，导致预留空间耗尽。

#### ‌2  晋升失败（Promotion Failed）‌

‌**场景**‌：新生代对象在 Young GC 后需晋升到老年代，但老年代‌**剩余连续空间不足**‌（即使总空闲空间足够）。

**原因‌：**

*   老年代内存碎片严重，无法容纳晋升的大对象‌；
    
*   晋升速率超过 CMS 回收速度（如短时间大量对象晋升）‌。
    

#### ‌3  大对象直接分配失败‌

若应用程序直接请求分配‌**大对象**‌至老年代（如通过 `-XX:PretenureSizeThreshold` 设置），而老年代无足够连续空间时，会直接触发该失败‌ 。

#### ‌4 CMS 回收速度滞后于对象分配速度‌

‌**并发阶段耗时过长**‌（如并发标记时间长），导致用户线程持续分配对象，最终预留空间被占满‌。

**优化方向**‌：通过 `-XX:+CMSScavengeBeforeRemark` 在重新标记前触发 Young GC，减少需跟踪的跨代引用‌。

## Concurrent Mode Failure 参数与调优

**调整触发阈值‌：**

```
-XX:CMSInitiatingOccupancyFraction=70  # 降低阈值，预留更多空间
```

**减少碎片影响‌：**

```
bashCopy Code-XX:+UseCMSCompactAtFullCollection     # Full GC 时压缩碎片-XX:CMSFullGCsBeforeCompaction=5       # 每5次Full GC压缩一次
```

**监控与扩容‌：**

通过 `jstat -gcutil` 监控老年代占用率及 GC 频率‌；

若频繁触发，优先考虑‌**扩大堆内存或老年代比例**‌‌。

## 三色标记的漏标、多标问题

在垃圾回收过程中，三色标记算法把对象分为三种颜色：

*   白色（未访问）。白色代表还没被标记到的对象，就像是还没被发现的 “宝藏”；
    
*   灰色（已访问但引用对象未全部访问）。灰色表示已经被标记了，但是它引用的对象还没被全部处理完，就像正在被挖掘的 “宝藏堆”；
    
*   黑色（已访问且引用对象已全部访问）。黑色则是已经处理完了，它引用的所有对象也都处理好了，是 “已经挖完的宝藏区”。
    

由于标记阶段是从 GC Roots 开始标记可达对象，那么在并发标记阶段可能产生两种变动：

1）本来可达的对象，变得不可达了    （多标 了活对象）

2）本来不可达的对象，变得可达了   （  漏标 了活对象）

具体来说，当垃圾回收线程与用户线程同时运行时，会出现以下问题：

### 漏标问题：

**本来不可达的对象，变得可达了**

原本应该被标记为存活的对象 (灰色、黑色)，被遗漏标记为白色，从而导致该对象被错误回收。

**漏标问题， 可以理解为  遗漏了存活对象。**

例如，假设对象 E 在被标记为灰色后，用户线程执行了`objE.fieldG = null`，切断了 E 到 G 的引用，但此时另一个对象 D 又引用了 G。由于 D 已经是黑色，不会再被扫描，导致 G 被漏标。

漏标问题，会导致 对象被错误回收， 导致程序发生错误。

### 多标问题：

**本来可达的对象，变得不可达了**

原本应该回收的对象，被多余地标记为黑色对象（存活对象），从而导致该垃圾对象没有被回收。

**多标问题， 可以理解为 ， 多标记了存活对象。  这些 对象 应该是垃圾对象， 叫做 浮动垃圾。**

例如，在并发标记阶段，之前已经被标记为存活的对象，突然变成了不可达对象，原因是其引用被删除了。但因为该对象已经是灰色，仍会被当作存活对象继续遍历，最终被标记为黑色存活状态。

## 大厂面试：CMS 如何 解决 漏标 ？

**漏标问题是  遗漏了存活对象，一个非常严重的功能问题，问题的本质变了。**

CMS 垃圾收集器采用 写屏障的机制来处理漏标问题。

写屏障是一种在对象引用发生变更时进行拦截的机制。

CMS 利用写屏障来记录并发标记阶段对象引用的变化情况，以便在重新标记阶段进行修正。

**CMS 采用 “写屏障” 技术来解决漏标问题。**

简单来说，就是当有对象的引用关系发生变化时，就会触发写屏障。

写屏障会检查这个变化是否会导致漏标，如果有可能，就会把相关的白色对象直接标记为灰色，确保它们不会被遗漏。

这就像是有个 “小卫士” 时刻盯着对象引用的变化，一旦发现可能漏标的情况，就马上把对象标记好，防止 “宝藏” 被遗漏。

**也有文章 总结 CMS 如何处理漏标 问题 的方法是：增量更新。**

其实，原理也是一样的。

增量更新  ，就是 当 黑色对象增加了对白色对象的引用之后,  将它的这个引用关系 记录下来,  在最后标记的时候, 再以这个黑色对象为根, 对它的引用进行重新扫描.

可以简单理解为, 当一个黑色对象增加了对白色对象的引用， 那么这个黑色对象就被变灰。 这个 增量更新  的 过程，还是用了   “写屏障” 技术。

增量更新  ，  有一个缺点, 就是会重新扫描这个黑色对象的所有引用, 比较浪费时间。

如何解决 多标答案是：**增加一个「重新标记」阶段。**

**无论是在 CMS 回收器还是 G1 回收器，它们都在并发标记阶段之后，新增了一个「重新标记」阶段来校正「并发标记」阶段出现的问题。**

解决漏标， 真正的执行，也是在 重新标记阶段处。

CMS 的执行过程简化为四个主要阶段，分别是初始标记阶段、并发标记阶段、重新标记阶段、并发清除阶段。

CMS 在重新标记阶段会处理漏标问题。

在这个阶段，它会再次检查那些被标记为垃圾的对象（白色对象），看看它们是否真的没有被其他地方引用了。

如果发现有被错误漏记的对象，让它们恢复 “正常状态”。

这就像是在把 “废品” 扔掉之前，再仔细检查一遍，看看有没有把还能用的 “工具” 误当成废品了，如果有，就把它们拿回来继续使用。

#### CMS 通过以下方式处理 漏标问题：

#### 1. 并发标记阶段（Concurrent Marking）

在并发标记阶段，CMS 会与应用程序线程并发运行，标记所有存活对象。并发标记阶段（Concurrent Marking） 的 增量更新是 CMS 处理漏标问题的核心策略之一。

**增量更新（Incremental Update）**： 在并发标记阶段，如果黑色对象（已标记对象）增加了对白色对象（未标记对象）的引用，CMS 会通过写屏障记录这一变化，并将黑色对象重新标记为灰色。

*   **写屏障的辅助**：在并发标记阶段，写屏障会实时记录对象引用的变化，确保漏标问题被及时发现和处理。
    
*   **灰色对象队列**：CMS 会维护一个灰色对象队列，记录需要进一步扫描的对象，确保所有引用关系被正确追踪。
    

#### 2 重新标记阶段（Remark Phase）

重新标记阶段是 CMS 解决漏标问题的关键步骤。

*   **STW（Stop-The-World）**：在重新标记阶段，CMS 会暂停所有应用程序线程，确保标记过程的准确性。
    
*   **处理漏标**：CMS 会重新检查所有可能被漏标的对象，特别是那些在并发标记阶段被写屏障记录的引用变化。通过重新扫描，确保所有存活对象被正确标记。
    
*   **校正标记结果**：重新标记阶段会校正并发标记阶段的错误，确保没有存活对象被错误地标记为垃圾。
    

## 大厂面试：CMS 如何 解决 多标 ？（浮动垃圾）

**多标问题是 多标记了存活对象， 不是非常严重的功能问题，问题的本质没有那么严重， 算是一个性能问题。**

多标问题会出现 是因为：在并发标记阶段，有可能之前已经被标记为存活的对象，其引用被删除，从而变成了不可达对象。

例如下图中，假设我们现在遍历到了节点 E，此时应用执行了 `objD.fieldE = null;`。

那么此刻之后，对象 E、F、G 应该是被回收的。

但因为节点 E 已经是灰色的，那么 E、F、G 节点都会被标记为存活的黑色状态，并不会被回收。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/xlgvgPaib7WOwKYibYDGPpJbjFMjhM8juVJluphtVjq2anMecX3hSOPnwAibLicExCk5kC4GiblO7fDzPDvgHBmx92A/640?from=appmsg)

多标问题会导致内存产生浮动垃圾，但好在其可以再下次 GC 的时候被回收，因此问题还不算很严重。

浮动垃圾是由于在并发标记阶段，某些对象被标记为存活，但在标记完成后，这些对象实际上已经不可达。由于 CMS 的并发特性，这些对象无法在本次 GC 中被回收，只能等待下一次 GC。

CMS 通过以下方式处理浮动垃圾：

#### （1）增量更新（Incremental Update）

CMS 使用增量更新机制来减少浮动垃圾的产生。

在并发标记阶段，如果应用程序修改了对象的引用关系（例如将某个字段设置为`null`），CMS 会通过写屏障（Write Barrier）记录这些修改，并在重新标记阶段重新扫描这些对象，确保它们被正确标记。

#### （2）重新标记阶段（Remark Phase）

重新标记阶段是 CMS 解决多标问题的关键。

在这个阶段，CMS 会暂停应用程序线程（STW），重新扫描所有可能被修改的对象引用，确保所有存活对象被正确标记。

如果是浮动垃圾，会进行重新标记，标记为 垃圾对象。

虽然重新标记阶段无法完全消除浮动垃圾，但它可以显著减少其数量。

#### （3）并发清理阶段（Concurrent Sweep）

在并发标记和重新标记阶段结束后，CMS 会进入并发清理阶段。

在这个阶段，CMS 会 回收  所有  不存活的对象。

虽然浮动垃圾无法在本次 GC 中被回收，但它们会在下一次 GC 中被清理。

#### （4）下一次 GC 回收

浮动垃圾虽然无法在本次 GC 中被回收，但它们会在下一次 GC 中被清理。

CMS 的设计允许浮动垃圾的存在，因为它的主要目标是减少停顿时间，而不是完全避免浮动垃圾。

## 遇到问题，找老架构师取经

借助此文，尼恩给解密了一个高薪的 秘诀，大家可以 放手一试。**保证  屡试不爽，涨薪  100%-200%。**

后面，尼恩 java 面试宝典回录成视频， 给大家打造一套进大厂的塔尖视频。

通过这个问题的深度回答，可以充分展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer 直提”。

在面试之前，建议大家系统化的刷一波 5000 页《[尼恩 Java 面试宝典 PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》，里边有大量的大厂真题、面试难题、架构难题。

很多小伙伴刷完后， 吊打面试官， 大厂横着走。

在刷题过程中，如果有啥问题，大家可以来 找 40 岁老架构师尼恩交流。

另外，如果没有面试机会，**可以找尼恩来改简历、做帮扶。**

遇到职业难题，找老架构取经， 可以省去太多的折腾，省去太多的弯路。

尼恩指导了大量的小伙伴上岸，前段时间，**[刚指导一个 40 岁 + 被裁小伙伴，拿到了一个年薪 100W 的 offer。](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486587&idx=2&sn=da85d7cacbdbc57403a9b901663df178&scene=21#wechat_redirect)**

狠狠卷，实现 “offer 自由” 很容易的， 前段时间一个武汉的跟着尼恩卷了 2 年的小伙伴， 在极度严寒 / 痛苦被裁的环境下， offer 拿到手软， 实现真正的 “offer 自由” 。

 

  

## 空窗 1 年 - 空窗 2 年，彻底绝望投递，走投无路，如何  起死回生  ？ 

**[失业 1 年多，负债 20W 多万，彻底绝望，抑郁了。7 年经验小伙，找尼恩帮助后，跳槽 3 次  入国企](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486733&idx=1&sn=866c605c5f32715cb2c7c84c52cb6611&scene=21#wechat_redirect) 年薪 **[40W offer ，逆天改命了](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486733&idx=1&sn=866c605c5f32715cb2c7c84c52cb6611&scene=21#wechat_redirect)****

**[被裁 2 年，天快塌了，家都要散了，42 岁急救 1 个月上岸，成开发经理 offer，起死回生](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486028&idx=1&sn=cf10ecda7a986b26e0359eba7a9cfeff&scene=21#wechat_redirect)**

**[空窗 8 月：中厂大龄 34 岁，被裁 8 月收一大厂 offer， 年薪 65W，转架构后逆天改命!](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486405&idx=1&sn=02336456cf4a09268c142c0b13327682&chksm=97b57a4da0c2f35be50007aee12b8f9e9aa7a74864f720846ae6dc58a5fa4b092e22d1805e1e&scene=21#wechat_redirect)**  

**[空窗 2 年：42 岁被裁 2 年，天快塌了，急救 1 个月，拿到开发经理 offer，起死回生](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486028&idx=1&sn=cf10ecda7a986b26e0359eba7a9cfeff&chksm=97b57bc4a0c2f2d2be01f2656ac65e9bd8854b91c64b01eb55f56e645f3c733525418808b06c&scene=21#wechat_redirect)**

**[空窗半年：35 岁被裁 6 个月， 职业绝望，转架构急救上岸，DDD 和 3 高项目太重要了](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486098&idx=1&sn=bbc5732b071477573bfab8a259d208d3&chksm=97b57b1aa0c2f20c27dd74b490b6062c9eec262a3ac25548534ff70290b172dcc51c6eafe532&scene=21#wechat_redirect)**

**[空窗 1.5 年：失业 15 个月，学习 40 天拿 offer， 绝境翻盘，如何实现？](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486345&idx=1&sn=8978c2c98378e85efe9a089fa08fc9e8&chksm=97b57a01a0c2f317c29c7bedab1fa8a9f6101ab35cf005df447d662daa9c17d56f4e4c3a354a&scene=21#wechat_redirect)**

##  100W-200W  P8 级 的天价年薪  大逆袭,  如何实现  ？ 

# **[100W 案例，100W 年薪的底层逻辑是什么？](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486315&idx=1&sn=52cc0a640c1499115f5a45b82097d857&chksm=97b57ae3a0c2f3f502293e84cde4bc892c45d06dee3579471a522ab497fb53e32640fb00c339&scene=21#wechat_redirect)** [如何实现年薪百万？ 如何远离  中年危机？](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486315&idx=1&sn=52cc0a640c1499115f5a45b82097d857&chksm=97b57ae3a0c2f3f502293e84cde4bc892c45d06dee3579471a522ab497fb53e32640fb00c339&scene=21#wechat_redirect)

**[100W 案例 2](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)****[：](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)**[40](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)[岁小伙被裁 6 个月，猛卷 3 月拿 100W 年薪 ，秘诀：首席架构 / 总架构](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)

**[环境太糟，如何升 P8 级，年入 100W？](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486369&idx=1&sn=0b32e8907dbcf26eaf33ea3f37ddd099&chksm=97b57a29a0c2f33f468d29dafe739204543873efb459f324871c3c68a3ecc178b1a9ab08a2fa&scene=21#wechat_redirect)**

##   

**职业救助站**

实现职业转型，极速上岸

![](https://mmbiz.qpic.cn/mmbiz_jpg/9qY2Gt3PBs3icE6WcM4nBOdRRcczOW4wkNDgCMZhBfqeet9suwGHfgQduP33YYqqZLOppecMI72Pv5ibNkPeIk6g/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

关注**职业救助站**公众号，获取每天职业干货  
助您实现**职业转型、职业升级、极速上岸**  
---------------------------------

**技术自由圈**

实现架构转型，再无中年危机

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/xlgvgPaib7WMtyFcJx45oIeWL1d6AENcdoAvuTPuq46E9n8pPe4SHXoSicuEVvhibHRSIzmEBPlC7POcDaqD6ID7w/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

关注**技术自由圈**公众号，获取每天技术千货  
一起成为牛逼的**未来超级架构师**

**几十篇架构笔记、5000 页面试宝典、20 个技术圣经  
请加尼恩个人微信 免费拿走**

**暗号，请在 公众号后台 发送消息：领电子书**

如有收获，请点击底部的 " 在看 "和" 赞 "，谢谢