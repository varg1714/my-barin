---
source: https://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247504872&idx=1&sn=1341f5a3ba9edad9d6a0c04bdbc01549&chksm=c16186b7f6160fa195eac95cfa9a96eb1753c74d7bf95933823f13314108970d86967adcb18f&scene=21#wechat_redirect
create: 2025-10-29 14:07
read: true
knowledge: true
knowledge-date: 2025-10-29
tags:
  - Java
  - JVM
summary: "[[JVM#1.2.5.2. ZGC 收集器]]"
---
![](https://mmbiz.qpic.cn/mmbiz_gif/AAQtmjCc74BOm5nOu7NGsmRFlmXicIibhicL1CSQAEzBib1Cfs64ovTRmL7DoXP3Ip11la9BXOickrxkZN5unyTibZYw/640?wx_fmt=gif#imgIndex=0)

**目录**

一、引言

二、ZGC 术语

三、ZGC 性能数据

四、ZGC 关键特性

    1. 着色指针 (Colored Pointer)  

    2. 读屏障 (Load Barrier)  

五、ZGC 执行周期

    1. 初始标记 (STW1)  

    2. 并发标记 (M/R)  

    3. 再标记阶段 (STW2)  

    4. 并发转移准备 (EC)  

    5. 初始转移 (STW3)  

    6. 并发转移（RE）

六、ZGC 算法演示

七、总结

# 一引言

垃圾回收对于 Javaer 来说是一个绕不开的话题，工作中涉及到的调优工作也经常围绕垃圾回收器展开。面对不同的业务场景没有一个统一的垃圾回收器能保证可 GC 性能。因此对程序员来说不仅要会编写业务代码，同时也要卷一下 JVM 底层原理和调优知识。这种局面可能因为 ZGC 的出现而发生改变，新一代回收器 ZGC 几乎不需要调优的情况下 GC 停顿时间可以降低到亚秒级。

Oracle 从 JDK11 开始正式引入 ZGC，ZGC 设计三大目标：

*   支持 TB 级内存 (8M~4TB) 。
    
*   停顿时间控制在 10ms 之内 (生产环境实际观测在微秒级) ，停顿不会随着堆的大小，或者活跃对象的大小而增加。
    
*   对程序吞吐量影响小于 15%。
    

ZGC 是如何设计怎么达到这个目标的呢？本文将从 ZGC 算法的关键特性入手，通过分析 ZGC 周期处理过程来理解这些特性，探索 ZGC 设计思想。

# 二 ZGC 术语

非分代：将对内存划分为新生代和老年代 (G1 已经逻辑分代) ，ZGC 取消分代设计，每个 GC 周期都将标记整个堆中的所有活动对象。  

页面：ZGC 将堆空间分解成一块块区域，这些区域叫做页面，ZGC 通过页面来回收内存。

并发性：GC 和线程和业务线程同时运行。ZGC 的高度并发设计，几乎所有 GC 工作、标记和堆碎片整理都是和业务线程 (mutators) 同时运行的，只包含了短暂的 STW 同步暂停。

并行：多个线程进行 GC 线程同时工作，加快回收速度。

标记 - 复制算法：标记 - 复制算法主要包括以下 3 个过程。

*   标记阶段，即从 GC Roots 集合开始，分析对象可达性，标记出活跃对象。
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/AAQtmjCc74Ae6OlorLlu9mkRJ6iabGkDicicRh2yONhcSIu8V0dICsfDcrV5iaPr8qmAQib9EbmvfDUqsOpickpbYJqA/640?wx_fmt=jpeg#imgIndex=1)

**图 1：可达性分析后对象的引用状态**

*   对象转移阶段，即把活跃对象复制到新的内存地址上。
    
*   重定位阶段，因为转移导致对象的地址发生了变化，在重定位阶段，所有指向对象旧地址的指针都要调整到对象新的地址上。
    

标记 - 复制算法的最大优势就是防止堆内存碎片化的出现，复制的过程就可以对堆内存进行整理。ZGC、CMS 和 G1 都是采用了标记 - 复制算法，但是不同的实现导致了很大的性能差异。

# 三 ZGC 性能数据

ZGC 设计致力于提供几毫秒的最大暂停时间，同时保证吞吐量不受影响。下面是 SPECjbb2015 针对 OpenJDK 中的不同收集器运行的性能测试数据。在 128G 堆内存下，无论是延迟还是吞吐量上面 ZGC 的性能表现都高于其他收集器。  

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74Ae6OlorLlu9mkRJ6iabGkDiciaicrz6Yciar7hYZJgMnYhKO7RAToGaGl3WUeQWjG2sWfPU24GMkbHic1Q/640?wx_fmt=png#imgIndex=2)

**图 2：SPECjbb2015GC 性能评分**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74Ae6OlorLlu9mkRJ6iabGkDic7ibqQs8hUlZWOsU1IOVSwgmuX2xMbgeDZcRLUqia1Oddk2VZPIwlDLWg/640?wx_fmt=png#imgIndex=3)

**图 3: SPECjbb2015GC 延迟比较**

# 四 ZGC 关键特性

ZGC 的周期是高度并发的，并发性越高意味着 GC 工作时对业务线程的影响越小，SPECjbb2015 的性能报告可以看出 ZGC 在延迟上比 G1 低 10 倍以上，ZGC 的工作周期只有三个阶段是 STW 的，其他阶段完全并发。这得益于 ZGC 在堆视图并发一致性设计上的改进。我们都清楚在并发的场景下需要协调各个线程对共享资源达成一致性，常用的手段就是对资源加锁，而在垃圾回收器下的思路也是类似，如果 GC 线程工作是需要锁定对象资源进行处理，业务线程则需要全部暂停，这就产生了 STW (Stop The Word) 。以往的垃圾回收器都是让 GC 线程和业务线程就堆中对象地址达成一致，对象在发生转移时业务线程是不能访问的 (因为对象的地址发生了变化) ，无论 G1 还是 CMS 对象在进行复制时都是需要 STW。ZGC 使用到的着色指针（Colored Pointer）和读屏障（Load Barrier）技术，可以让所有线程在并发的条件下就指针的颜色 (状态) 达成一致，而不是对象地址。因此，ZGC 可以并发的复制对象，这大大的降低了 GC 的停顿时间。我们先对着色指针和读屏障有个初步的理解，然后在通过 ZGC 回收周期来看这 2 项技术的具体运用。  

## 着色指针 (Colored Pointer)

在指针中嵌入元数据（使用地址中的高阶位来实现），这种通过在指针存储元数据的技术就叫做着色指针 (Colored Pointer) 。ZGC 中指针始终是 64 位结构，由元位（指针的颜色）和地址位组成。地址位数决定了理论上支持的最大堆大小，ZGC 使用 42 位存储地址也就意味着 ZGC 最大支持 4TB 堆内存。如图所示，低 42 位是地址位，中间 4 位是元位，高 18 位未使用。四个元位是 Finalized (F)、Remapped ( R )、Marked1 ( M1 ) 和 Marked0 ( M0 )。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74Ae6OlorLlu9mkRJ6iabGkDic0P5tFWNuqQ5y6VpBibhe09UBfxtKDGDwPHFLNIdPdAF8Qjj2B6fyeGw/640?wx_fmt=png#imgIndex=4)

**图 4: 64 位地址使用示意图**

ZGC 中将指定上的标记通过颜色来表示，颜色可以是 “good” (地址有效) 或“bad” (地址可能无效) 。指针的颜色由其元位的状态决定：F、R、M1 和 M0。“good” 是 R、M1、M0 元位中的一个被设置，另外三个未设置，比如 0100、0010 和 0001 属于 “good” 颜色。通过在指针上的颜色就能区分出对象状态，不用额外做内存访问，这使得 ZGC 在标记和转移阶段会更快。

通过设置地址元位的状态，可以形成不同地址视图，ZGC 同一物理堆内存被映射到虚拟地址空间三次，从而产生同一物理内存的三个 “视图”，GC 活动的不同时期会只存在一个活跃视图，根据垃圾回收的周期 ZGC 通过切换不同视图标来记出对象的颜色。

**下图是虚拟地址的空间划分：**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74Ae6OlorLlu9mkRJ6iabGkDicIpI15nVZ8UnExh6Y0VshUJyVF77xvseTUqyvDL166PWCSYm8iaianu0g/640?wx_fmt=png#imgIndex=5)

**图 5：虚拟地址空间划分和多视图映射**

\[0~4TB) 对应 Java 堆；

\[4TB ~ 8TB) 称为 M0 地址空间；

\[8TB ~ 12TB) 称为 M1 地址空间；

\[12TB ~ 16TB) 预留未使用；

\[16TB ~ 20TB) 称为 Remapped 空间。

ZGC 是不分代的，这意味着垃圾回收是需要扫描整个堆空间，地址视图将整个 Java 堆分成多个部分，并为每个部分分配一个虚拟内存段。在垃圾回收时，ZGC 只需要扫描其中一个虚拟内存段，并将其作为当前视图映射到实际的内存位置。同时，ZGC 会将其他虚拟内存段映射到虚拟地址上，这些内存段不会被收集器扫描。

## 读屏障 (Load Barrier)

ZGC 通过利用读屏障而不是写入屏障，与 HotSpot JVM 中以前的 GC (CMS，G1 等) 算法显著不同。读屏障解决了并发转移时对象指针更新问题：在转移期间，如果移动对象而不用更新引用对象的传入指针（移动的对象可能被堆中的任何其他对象所引用），就会产生悬空指针 (已经被释放的内存空间或者无效的内存地址，访问悬空指针会出现问题) 。通过读屏障技术能够捕获此类悬空指针对象，并触发代码，更新对象的新位置，从而 “修复” 悬空指针。为了跟踪对象如何移动，以便在加载时固定悬空指针，ZGC 中使用转发表 (forwarding tables) 来将重定位前（旧）地址映射到重定位后（新）地址。无论是业务线程作为使用者访问对象，还是 GC 线程遍历堆中的所有活动对象（在标记期间）都有可能会触发读屏障。  

ZGC 读屏障如何实现呢？举个例子，代码 `var x = obj.field`。x 是一个位于堆栈上的局部变量，field 是一个位于堆上的指针。业务线程在操作堆对象时触发读屏障。读屏障的执行路径有快 (fast path) 和慢 (slow path) 两种，如果正在加载的指针有效状态 (good color) ，则采用加载屏障的快速路径，否则，采用慢速路径。快速路径实际上是空的，而慢速路径包含计算有效状态指针的逻辑：检查对象是否已经（或即将）重新定位，如果是，则查找或生成新的地址。读屏障除了能让触发读屏障的线程读取到最新地址，同时还具有**自我修复指针**（self-healed）的功能，这意味着读屏障会修改指针的状态，以便后续其他线程访问时能执行快速路径。无论采用哪条路径，都会返回正确状态的地址。下面用伪代码表示 ZGC 在执行读屏障时的大体逻辑：

ZGC 的读屏障可能被 GC 线程和业务线程触发，并且只会在访问堆内对象时触发，访问的对象位于 GC Roots 时不会触发，这也是扫描 GC Roots 时需要 STW 的原因。

下面是一个简化的示例代码，展示了读屏障的触发时机。

# 五 ZGC 执行周期

如下图 所示，ZGC 周期由三个 STW 暂停和四个并发阶段组成：标记 / 重新映射 (M/R)、并发引用处理 ( RP )、并发转移准备 ( EC ) 和并发转移 ( RE )。为了读者能快速理解，下面对 ZGC 执行过程进行了大量简化。  

![](https://mmbiz.qpic.cn/mmbiz_png/cM52G8pP5mLUuAQyZUUhMxib50UUupoOmzmL2CwvGJk2hNhFgykGSuGf7eYEfPaUtEdblw38byTPEibLcH6Bia40g/640?wx_fmt=png#imgIndex=6)

**图 6：ZGC 周期表示**

## **初始标记 (STW1)**

ZGC 初始标记执行包含三个主要任务。  

*   地址视图被设置成 M0 (或 M1) ，M0 还是 M1 根据前一周期交替设置的。
    
*   重新分配新的页面给业务线程创建对象，ZGC 只会处理当前周期之前分配的页面。
    
*   初始标记只会存活的根对象被标记为 M0 (M1) ，并被加入标记栈进行并发标记。
    

**GC** **周期中地址视图窗口**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74Ae6OlorLlu9mkRJ6iabGkDicIZ9676muiccpDfUU9u9E6MMG2GaEHSrqBadWqxBwenZx1ptxR3OB56g/640?wx_fmt=png#imgIndex=7)

**图 7：ZGC 周期中状态窗口划分**

## **并发标记 (M/R)**

**并发标记的任务有 2 个：**  

第一，并发标记线程从待标记的对象列表出发，根据对象引用关系图遍历对象的成员变量，递归进行标记。

第二，计算，并更新关联页面的活跃度信息。活动信息是页面上的活动字节数，用于选择将要回收的页面，这些对象将作为堆碎片整理的一部分进行重新定位。

下面伪代码是并发标记的主要过程：

## **再标记阶段 (STW2)**

再标记阶段的主要任务有 3 个：  

*   执行修复任务，指线程运行 C2 编译的代码，在进入再标记阶段时可能发生漏标。
    
*   结束标记，并发标记后业务线程本地标记栈可能存在待标记的对象，执行本步骤的目的就是对这些待标记对象进行标记。
    
*   执行部分非强根并行标记。
    

## **并发转移准备 (EC)**

并发转移准备任务：  

*   筛选所有可以被回收的页面
    
*   选择垃圾比较多的页面作为页面转移集
    

## **初始转移 (STW3)**

初始转移主要以下过程：  

*   调整地址视图：将地址视图从 M0 或者 M1 调整为 Remapped，说明进入真正的转移，此后所有分配的对象视图都是 Remapped。
    
*   重定位 TLAB：因为地址视图调整，所以要调整 TLAB 中地址的视图。
    
*   开始转移：从根集合出发，遍历根对象的直接引用的对象，对这些对象进行转移。
    

初始转移是 STW 的，其处理时间和 GC Roots 的数量成正比，一般情况耗时非常短。

## **并发转移（RE）**

初始转移完成了 GC Roots 对象重定位，在并发转移阶段将对前面步骤确定的转移集 (EC) ，对转移集的每一页执行转移。  

并发转移的过程可以抽象成如下伪代码过程：

**转发表**的作用是存储对转移后旧地址到新地址的映射，转发表的数据存储在页面中，转移完成的页面即可被回收掉。

并发转移完成之后整个 ZGC 周期完成。

# 六 ZGC 算法演示

为了说明 ZGC 算法，下图演示了示例中的所有阶段。

![](https://mmbiz.qpic.cn/mmbiz_jpg/AAQtmjCc74Ae6OlorLlu9mkRJ6iabGkDiczwKsDmDkwO17dtV4PCWSar3VCBTNiaaOCv5BA2R6XTp6jlg17k0H8zg/640?wx_fmt=jpeg#imgIndex=8)

**图 8：ZGC 算法演示**

图 8(1) 显示了堆的初始状态，应用启动后 ZGC 完成了初始化。

在图 8(2) 中，选择 M0 作为全局标记，并且所有根指针都被标记成 M0。然后，所有根都被推送到标记堆栈，该标记堆栈在并发标记 (M/R) 期间由 GC 线程消耗。

如图 8(3) 所示，图中用合适的颜色绘制对象本身，以表明它们已被标记，即使指针有状态。

在图 8(4) 中，选择存活对象最少的页面（中间的页面）作为转移候选集 (EC) 。

随后，在图 8(5) 中，全局标记被设置为 Remmaped，并且所有根指针都已更新 Remmaped。如果根指向 EC，则相应的对象将被重新定位，并且根指针更新为新地址。

在图 8(6) 中，EC 中的对象被转移，并且地址记录被逐出页面中转发表上，用于新旧地址转换。当并发转移阶段结束时，当前 GC 周期也会结束。当前周期内整个 EC 都会被回收。这里可能有个疑问，对象的旧地址还没有更新，页面如果被回收了如何还能访问对象呢？原因是回收的是页面中对象存储空间，转发表不会被回收，如果此时业务线程访问这些对象，会触发读屏障的慢路径位，失效指针会被修复。对于没有访问到的失效指针，直到下一个 GC 并发标记 (M/R) 阶段才会被修复。

在图 8(7) 中，下一个 GC 循环开始，M1 被选择为全局状态（M0 和 M1 之间交替使用）。

在图 8(8) 中，并发标记阶段 (M/R) 通过查询转发表失效的指标被映射到新位置。

最后，在图 8(9) 中，上一周期 EC 页面的转发表被回收，为即将到来的并发转移 (RE) 阶段做准备。

# 七 总结

ZGC 是一个十分复杂的 JVM 子系统，没办法通过一篇文章把所有的细节描述清楚。本文详细探讨了 ZGC 的着色指针和读屏障关键技术，他们也是 ZGC 中创新点，最后通过一个示例对 ZGC 算法过程做了一个简化版的演示。通过对 ZGC 这种复杂系统的学习，让我也体会到分析复杂系统时没必要一开始就过多的纠结实现细节，可以先从关键流程入手再层层深入。

ZGC 的高并发设计造就了它的高性能，背后要归功于着色指针和读屏障运用，当然除了这 2 项还有其他精妙的设计比如：内存模型，并发模型，预测算法等这里不展开，读者可以参考其他文章。了解 ZGC 的基本原理可以帮助优化应用程序的性能，为应用调优做些知识储备。最后，ZGC 有卓越的性能和稳定性表现，我们在选择 GC 选型时可以优先考虑使用 ZGC。

**参考内容：**

[1] 彭成寒:《新一代垃圾回收器 ZGC 设计与实现》. 机械工业出版社, 2019.

[2]https://tech.meituan.com/2020/08/06/new-zgc-practice-in-meituan.html

[3]https://www.baeldung.com/jvm-zgc-garbage-collector

[4]https://openjdk.org/projects/zgc/

[5]https://www.jfokus.se/jfokus18/preso/ZGC--Low-Latency-GC-for-OpenJDK.pdf

**往期回顾**

[1. 包体积：Layout 二进制文件裁剪优化｜得物技术](http://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247504706&idx=1&sn=3b012ea0bb71873ea12b7802e2366302&chksm=c161861df6160f0bb4ade47688d6b1fdd4d40a605702be75a47fb050ebda24571b8e749ee66f&scene=21#wechat_redirect)  
[2. Go 语言进化之路：泛型的崛起与复用的新篇章｜得物技术](http://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247503875&idx=1&sn=1dd5bacc6beb038579082cb1600eec5f&chksm=c161855cf6160c4aa96f133ecd3f3fc923671429db4b612b9583a538ed9eacb745c2be0bbab5&scene=21#wechat_redirect)  
[3. 得物 SRE 视角下的蓝绿发布](http://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247503830&idx=1&sn=1e90fe4c04d5812a3a40846620338c2d&chksm=c1619a89f616139ffc2427fcb0ce2cd75e5edce8674f639cb5f8b6bd870d9d1384080aaf7cf5&scene=21#wechat_redirect)

[4.Enhancer - 轻量化的字节码增强组件包｜得物技术](http://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247501739&idx=1&sn=25e2f172e576d4348cb04f0525d8fe82&chksm=c16192f4f6161be21610deaaf8899f88a9ac4f4f354dbe85382a12a4791ad8b9db421c693566&scene=21#wechat_redirect)  

[5. 算法 AB 实验平台进化历程和挑战｜得物技术](http://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247500944&idx=1&sn=07851e09864ae37357223c3e50001803&chksm=c16191cff61618d9fa37740eb272b84cd0743cc7cba7a73fb83d1eea2d49f3f7498c384afa02&scene=21#wechat_redirect)

[6. 实时数仓混沌演练实践｜得物技术](http://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247497670&idx=1&sn=32538a79d4969f5257cda34ba16697f4&chksm=c161a299f6162b8f6124e29b21ec807a8572747ceb295d05b4e1d1fc95a515684c6c8396969b&scene=21#wechat_redirect)

* 文 / byteyangyang

关注得物技术，每周一、三、五更新技术干货  
要是觉得文章对你有帮助的话，欢迎评论转发点赞~  
未经得物技术许可严禁转载，否则依法追究法律责任。

“

**扫码添加小助手微信**

如有任何疑问，或想要了解更多技术资讯，请添加小助手微信：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/AAQtmjCc74CAGS6PldJufoMwZe4UZ1IwmaXQ5n9mkpElaPtrunYoYgbIB7sib5m1qD2jfErd5MZ449jicmLWqTZg/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=9)