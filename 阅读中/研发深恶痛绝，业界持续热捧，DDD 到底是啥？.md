![](https://mmbiz.qpic.cn/mmbiz_gif/VY8SELNGe949kg74XCNJRZbo2J69kUDlHL9cFWgKWlaW2UH03In3QZGTlN4LMZSLuibxWflH4kOEkPA7nPLL7hw/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

# 关注并星标腾讯云开发者

# 每周 3 | 谈谈我在腾讯的架构设计经验

# 第 7 期 | 刘啸南：领域驱动设计四论 — 在江湖派与学院派之间架起桥梁

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97oUO8lib6RcQaathXVMMLkTC3UfYoaOjPl6y3qt0wtFNzF4SMlsGlcE20PmIh1zic01AOicnkpU75mw/640?wx_fmt=png)

冯友兰治哲学，提出 “照着讲” 和“接着讲”的方法论。近两年，我断断续续梳理出关于领域驱动设计的两个 PPT：《领域驱动设计》和《领域驱动设计四论》。前者的内容主要是关于 DDD 经典著作的读书笔记，可视为照着讲，以证明自己学有所本，讲的不是野狐禅；后者则是在继承的基础上所做的创新阐释，可视为接着讲，发前人之未发。本文重点围绕《四论》展开，从四个方面梳理出 DDD 的整个逻辑脉络。

曾见郭象注庄子, 却是庄子注郭象。一些领域驱动设计的拥趸者，如果看到本文的论述和自己的理解相左，丝毫不用奇怪，本文阐述的四论，不是我注六经，而是六经注我。

本文是 DDD 系列三篇文章的开篇，为你提纲挈领地梳理出 DDD 的核心脉络。在后两篇文章中，我们还将展开分享 DDD 的系统架构设计与核心概念、关键方法等内容。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbClsvcfOx2U099UBd026crjZoDnbL60hs44tw8bZiapRNEicKmib8urobAg/640?wx_fmt=png)

在系统开发这一行摸爬滚打多年，开发过众多业务系统，也接触过各种技术流派，结合众多书籍，从技术风格上分为两派：

*   江湖派：重实践，重个人领悟，以思想体系见长
    
*   学院派：重规范，重系统训练，以方法体系见长
    

在技术社区里，热闹程度可谓冰火两重天，江湖一派热热闹闹，追随者众，耳濡目染，无师自通；学院一派则是养在深闺人未识，门可罗雀，亟待挖掘。

有趣的是，两派似乎从不往来，从两派的著作来看，江湖派偶尔涉足学院派的领域，但学院派决口不提江湖派的东西。看来在技术圈子里，同样也是文人相轻、道不同不相为谋，各自混各自的圈子。

作为一个互联网工程师，长期奋战在开发第一线，对这种技术分裂深感无奈，总感觉有座巴别塔横亘其中。本人结合自身长期在一线工作的经验积累、对各种经典著作的熟稔以及对软件开发的持久兴趣，试图通过本文在两派之间架起一座桥梁，相互借鉴，熔于一炉。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCEyre5g6AqV9x4boUBbSibqWMczGhFm0ibOcKibKoCbOhB7droo6UPIczQ/640?wx_fmt=png)

从《领域驱动设计 · 软件核心复杂性应对之道》（英文版）在 2004 年第一次发表算起，到现在已有近 20 年，期间阐述 DDD 的书汗牛充栋，但 DDD 具体包含哪些内容，似乎从没有达成共识，有种包罗万象的感觉。本文围绕 DDD 的经典著作，参照学院派的风格，将 DDD 概括为四部分：

*   结构论：战术建模与战略建模。
    
*   过程论：系统重构与软件工程。
    
*   语言论：基于模型的统一语言。
    
*   建模论：模型驱动的领域建模。
    

这个划分，其实体现了本人长期的实践体悟和理论思考，并不是随意为之。王阳明晚年将其毕生所学，浓缩为四句话：

*   无善无恶心之体。
    
*   有善有恶意之动。
    
*   知善知恶是良知。
    
*   为善去恶是格物。
    

本人将 DDD 概括为四论，不是要盖棺定论，只是希望去掉枝叶保留主干，避免 DDD 逐渐沦为一种坊间流传的野狐禅。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCic4dibnqP3YMKrtsjt5rIhNldRQb07WETFyVdZRpxF5YEBtNxb2acrXA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbChZjeaTFlvkEfbEwMcK05WqicAtql7JOpjRanibQ14I4ZrZWrzCLkjkow/640?wx_fmt=png)

*   ### 复杂度
    

《没有银弹：软件工程的本质性与附属性工作》（No Silver Bullet—Essence and Accidents of Software Engineering），1986 年发表一篇关于软件工程的经典论文，打开了复杂度这个潘多拉魔盒。论文中 brooks 把失控的、复杂的软件项目比作中世纪的狼人，只有银弹才能杀死它。但是由于软件开发的本质复杂性，使得真正的银弹并不存在，即没有任何技术或管理上的进展， 能够独立地许诺十年内使软件系统项目生产率、 可靠性或简洁性获得数量级上的进步。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCvkhnlvxwfKq608iaSxHUUJ2XUlaID8YddIWalISHKnPgpIgogGm740A/640?wx_fmt=png)

到现在几十年过去了，狼人依旧未被杀死，人月依旧是个神话，银弹在哪？  

复杂度也许永远不能消除，但我们可以分析复杂度，进而管理复杂度。在软件开发领域，大体上可以将复杂度分为三类：软件本身固有的复杂度，业务逻辑带来的复杂度，以及技术复杂度。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCRUw173CdJoosrpGtwEl4VhSl3k5UqCHIzR2OoScngMPUugcq4wzTPA/640?wx_fmt=png)

*   ### 应对之道
    

从我看过的几本经典书籍来看，软件核心复杂性的应对之道，不外乎就是：抓住总体，理清局部。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCKmZgtlHGmaHAzpc9uFZvycKumNSPkjuCRydZMl1QT3bdoppC7TwHOA/640?wx_fmt=png)

*   ### DDD 与复杂度
    

DDD 提供了一些应对复杂度的具体方法：

*   通过架构设计来分离业务复杂度和技术复杂度。
    
*   通过限界上下文将一个大系统切分为若干高内聚低耦合的子领域。
    
*   通过领域模型对业务领域的知识进行抽象。
    

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCjFFUs0yrW4EuSRzLTtr24Dzzk1h5OFHSHnshQR6aGqBbHUoEXh9UZA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCwo92tdESQiaS957POyLLaia0ZFckZP669l9WdjOMefb6bbE3TJGWSGEA/640?wx_fmt=png)

*   ### Domain-Driven Design 还是 Model-Driven Design ？
    

在《领域驱动设计 · 软件核心复杂性应对之道》这本书里，对什么是 “领域”，只有简单的一句话：“每个软件程序是为了执行用户的某项活动，或是满足用户的某种需求。这些用户应用软件的问题区域就是软件的领域。” 除此之外，讲得更多的是“模型” 或者“领域模型”，纵观全书，“Model-Driven Design” 是其核心概念之一。某种程度上，把 “领域驱动设计” 改为 “模型驱动设计”，一点问题都没有，甚至“模型驱动设计” 更能体现整本书的精髓。

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCwXsZGqhHmdmQRoWonmIf1NuicZCvrEcLcQW0NX70dCfG88yVfdL5OSg/640?wx_fmt=jpeg)

*   ### 领域划分与领域模型
    

总的来说，“领域”这个词可能承载了太多含义。领域既可以表示整个业务系统，也可以表示其中的某个核心域或者支撑子域。在本书中，我将尽可能地区分这些概念。当谈及到业务系统中的某个方面时，我会使用诸如 “核心域” 或者 “子域” 以示区别。 

由于 “领域模型” 包含了 “领域” 这个词，我们可能会认为应该为整个业务系统创建一个单一的、内聚的、全功能式的模型。然而，这并不是我们使用 DDD 的目标。正好相反，在 DDD 中，一个领域被分为若干子域，领域模型在限界上下文中完成开发。事实上，在开发一个领域模型时，我们关注的通常只是这个业务系统的某个方面。试图创建一个全功能的领域模型是非常困难的，并且很容易导致失败。

*   ### 关于 DDD 的一个野狐禅
    

本人接触领域驱动大概已有 10 年时间，直到近期才听同事说起过 “贫血模型”，然后在网上一查，原来是 martin fowler 在 2003 年发的一个 blog：AnemicDomainModel。其本意只是指出，很多人其实误用了领域模型，把领域模型的 Model 直接等同于 MVC 的 Model，导致 Model 里只有数据没有逻辑，fowler 给这种情形取了一个名字叫 “Anemic Domain Model”。

Fowler 把这种误用概括为一种反模式 AnemicDomainModel，在流传过程中，有人把 “Anemic Domain Model” 翻译为“贫血模型”，后面又衍生出“充血模型”“失血模型”“胀血模型”，这些就都是穿凿附会之说，跟 Martin Fowler 无关，跟领域驱动更无关系。很多 blog 都是把 DDD 跟 MVC/DAO/ORM/TDD 放在一个层次来理解，这说明完全不理解 DDD，实际上 DDD 是跟 MBSE/SysML/UML 是一个层次的概念。

在领域驱动的经典著作里，完全没有 AnemicDomainModel 的相关提法，把 martin fowler 的说法穿凿附会乱加引申，将领域模型简单分类为 “失血，贫血，充血，胀血”，这个大概就属于野狐禅了，完全背离了 DDD 的本质精神。

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCFeRRvCH7nxJFQOiaico8yxTr8nicnd2rhz3fbeYn59R4lRTMv6fgPd7qw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCB7Eh9KjpbQdgFlrYsTDx3zI6U6AEQNogCZpTc3OePGniaia3SaqjHCuQ/640?wx_fmt=png)

*   ### 战略建模与战术建模的划分
    

DDD 相关经典著作，一般都将 DDD 划分为战略设计和战术设计两部分。《领域驱动设计 · 软件核心复杂性应对之道》和《实现领域驱动设计》都明确有战略设计的提法，其他两本书则明确提出了战略设计和战术设计。

顾名思义，战略设计就是宏观设计，即对系统整体进行建模，称之为 “战略建模”；战术设计则是微观设计，即对系统的局部进行细粒度的建模，称之为 “战术建模”。

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCAbplNvMnUEulXM81bZvPCNibBkXzIuFQicJePdCgckW0Kg1CPHDiam7Eg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCR1ADicnt62FeyGQQALNmr4ADlqBOzUYURAqKzKsKvSKoCBpMcDnz5Tw/640?wx_fmt=png)

分享的一张图，很清晰：  

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCZS7b56yEW6DFZ9XaKUyxy5sIIXGeNZ3gVFCBS0PsytnvLS9fIAezTg/640?wx_fmt=png)

*   ### 战略建模  
    

战略设计原则必须指导设计决策，以便减少各个部分之间的互相依赖，在使设计意图更为清晰的同时而又不失去关键的互操作性和协同性。战略设计原则必须把模型的重点放在捕获系统的概念核心，也就是系统的 “远景” 上。而且在完成这些目标的同时又不能为项目带来麻烦。为了帮助实现这些目标，我们提出了战略设计的 3 大原则：上下文、精炼和大型结构。

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCWS5y7O79a6Z1Ad2CUrickicicZiat1DNW4d1zjV4l0bQJjkVzreW72omcw/640?wx_fmt=jpeg)

理解大型系统的常用方法：  

*   使用隐喻，比如电动汽车其实就是一台电脑装了四个轮子。
    
*   把大型系统从逻辑上切分成若干层，分而治之。
    
*   把大型系统提炼为一个抽象结构，例如，冯诺依曼计算机 = IO+CPU+Memory。
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbC6ic5KPyFtBG3eLQAZyAdeZfnjS0EBTe3wZxr74AQ4QvSHNEicLlBFudg/640?wx_fmt=jpeg)

DDD 中的上下文（Context）是个让人迷惑的词，从一种比较宽泛的视角来看的话，Context 可以对应于 UML 的 class 或者 SysML 的 block，即 Context 可理解为是一个类或模块。ContextMap 则对应 UML/SysML 的 Relationship。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCgzjrmUic3Vq9Qe9OVoXFa2vRS1e5iaibdGj4vuJQ7bPJf7Jv82GEXqTKg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCQmib3ukUeWOUYR7r8xV7hhX4Nj3Vc6ncGTdKYRjjUx73vu0XN1GpmfA/640?wx_fmt=jpeg)

战略精炼：对核心域进一步萃取，过滤掉不必要的杂质，使得其方向更清晰，内容更准确、内核更精干。  

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCicMVq9HbbLOZic3bLsiaTzCKJ8xssfZKb5zH00AM3wOXXmIBhoHyF7aibg/640?wx_fmt=jpeg)

战略精炼：相关方法可以分为三大类，即提炼出内核、进一步精炼内核、增强沟通。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCR9XzC1WznjKnrMlico3RodpAnpP9MicibIWj8ApRgvcJvzrmOaEaEs0Iw/640?wx_fmt=png)

*   ### 战术建模
    

战术建模侧重于从微观层面对系统进行建模。DDD 提到的战术建模方法主要是构造块与柔性设计。

构造块：在类、对象、组合、继承等层次上对系统进行设计。按照 DDD 的术语，我们可以把服务、事件、实体、值对象归类为原子块，把资源库、聚合、工厂归类为组合块。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCFgZiaYSnj9uAEO9tG9ia84H1jRAiaVs4p9VfM7niblIUMksRrHKwfxuwiag/640?wx_fmt=png)

柔性设计：列举了一些设计原则，类似于常见的软件设计原则，只是换了一种说法。 例如通过明确概念、避免概念过载、处理副作用等做法，得到的是一个高内聚低耦合的设计。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCjsuTcIyoYYUgjND1vwgVMmxrFuEmTLSD0LvFxESNmKbPBmRlicDH1DQ/640?wx_fmt=png)

软件设计原则，说到底不外乎，模块内高内聚，模块间去耦合。下面是学院派总结的一些原则，可以对照来看。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbC3es1DaU7rQDia6lbH6AY5vlonsVV6UAsDqQUCyun9c3uMl9l8NIjvpA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCfVDoVUp1rdTricKoVHicQk5USW2gUlDxhKU5UNR4qGg2S4F5Qj9QNcog/640?wx_fmt=png)

*   ### DDD 关于开发过程的论述
    

关于开发过程，《领域驱动设计 · 软件核心复杂性应对之道》这本书第三部分用 6 章的内容重点论述了重构，但对于软件工程，只是简单提及 “敏捷开发过程”。整体而言，作者更倾向于用重构等手段不断迭代，开发人员和业务专家紧密配合，让协作贯穿整个项目的生命周期。本小节从下面三个方面对 DDD 的开发过程作一扩展性论述：

*   通过重构加深理解
    
*   敏捷开发过程与持续交付
    
*   引入软件工程的规范方法
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCTxaBl7ngVicib4uPmytrQoHnKKKwo1zPkZXzQLib2Jrf300cP74H4MKZQ/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCALBW4msZ7sibpX4gQXmOFTq0cUZIdeywqnYWXczRyj4KdAVu0hwkWvw/640?wx_fmt=jpeg)

*   ### 通过重构加深理解
    

领域驱动设计的重点在系统设计阶段，但领域驱动设计同样将重构作为重要内容。《领域驱动设计 · 软件核心复杂性应对之道》这本书第三部分共 6 章的篇幅在介绍重构。看其内容，名为重构，实则仍然是设计，例如概念建模、柔性设计、分析模式、设计模式等等，基本都是一些偏宏观的设计内容。内容上，跟 Martin Fowler 的《重构：改善既有代码的设计》还是有很大区别。

论重构，当首推软件开发一代宗师 Martin Fowler 的《重构：改善既有代码的设计》，无人能出其右。

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCtLDDDZ3Cg0XRMrBSamauicJJ7ichVYqibKjWreibWvSPcB1VztBOaDw58g/640?wx_fmt=jpeg)

《重构：改善既有代码的设计》这本书的主要内容，可以用一个思维导图展示如下。

从这本书开始，” 代码坏味道” 成为开发领域的一个标准术语。在重构的过程中，有一些基于经验的原则可供参考。

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCwhFgpmHtWuvSdd6cZ2prWIyZFz3z6iauicE5alfT0vWuXyGoOI8u4aJg/640?wx_fmt=jpeg)

就重构的具体做法上，可以从函数、数据、业务逻辑三个方面来展开。

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCFsWtghwvc1R2KDyQPqhO61xyu2147hVL2egF86pCqGmOqYicaRPicFHg/640?wx_fmt=jpeg)

某种程度而言，软件工程师仍然是手工业者，软件开发仍然没有银弹，重构仍然是软件在生长过程中不可或缺的调校手段。

因此，我们也不用迷信什么银弹，也不必忌讳什么过度设计与设计不足，通过多次重构迭代，让正确的设计逐步显现。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbC3wuYEB0Ac2werETFFibkY130gQ2jdnotezTiapXVOzDpkMSn7HCzCDXg/640?wx_fmt=png)

*   ### 敏捷开发与持续交付
    

在《领域驱动设计》出版的年代（2004），正值敏捷开发和极限编程大行其道的年代，《领域驱动设计》尽管不局限于某种固定的开发过程，但主要还是面向 “敏捷开发过程” 这一新体系。

二十年后的今天，敏捷开发和极限编程早已式微，但乔梁老师的著作《持续交付》给我们提供了新的视角。乔梁老师在其著作《持续交付》里梳理了软件工程的进化史。最近两三年，作为腾讯外聘高级管理顾问，其 “价值探索 - 快速验证” 的持续交付 2.0 双环模型，令人印象深刻。

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCuFQxicXc7OIHyEMus26aWuMK0AQ8vThTcfYXoTVOuqpbB1dAKibkUT9w/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCVp5sCmcvvmQmvOVliafZLe9VbmLTSkQibQGmPgq0qwpTgx7IMNE5NZrg/640?wx_fmt=png)

*   ### 引入软件工程的规范方法
    

国内最新的一本著作《解构领域驱动设计》，作者试图参考 RUP，给 DDD 建立一种类似的规范过程。

《解构领域驱动设计》出版于 2021，据了解作者张逸是业内知名 DDD 专家，也是《实现领域驱动设计》一书的审校。在《解构领域驱动设计》这本书里，作者试图将 DDD 与 RUP 融合起来，提出了 DDDUP（领域驱动设计统一过程）模型，能否成功，我们拭目以待。

《解构领域驱动设计》这本书很厚，500 多页，值得一读，读完能体会到作者深厚的行业经验。

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCyzCnoCnHjGJZfWYicfssknaeulTY2ianHqC4hN7dxFAjfz8Nf3H2lZag/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCA7S43oFyEpRU0NLt3YVqNU9MPp6lVZnxCFxic9G8vkSmtNIJT26agFQ/640?wx_fmt=jpeg)

领域驱动设计统一过程（DDDUP）参考了统一过程（rationalunified process，RUP）的二维开发模型。整个过程的二维模型图所示，横轴代表推动领域驱动设计在构建过程中的时间，体现了过程的动态结构，构成元素主要为 3 个阶段（phase），每个阶段可以由多个迭代构成；纵轴表现了领域驱动设计在各个阶段中执行的活动，体现了过程的静态结构，构成元素包括工作流（workflow）和元模型（meta model）。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbC320ibF7C21KjTqZ3tcaRtrOpZOnohbg1aMTicNXR8IqpqSfDmsLsecCw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCZISbCGamwDtFf1AkBIeA5HzWWSxgLiaqAPkHxO7Q30bCEDeiaISiaZhcw/640?wx_fmt=png)

*   ### 统一语言
    

UBIQUITOUS LANGUAGE，有的书翻译为通用语言，有的书翻译为统一语言，其核心要点为：

*   一个团队一种语言，语言统一才能沟通
    
*   将领域模型作为统一语言的核心，基于模型进行沟通
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbC3GpQxgS5Z7BmMz8nPLYMfZibtdX5GL0OuwlWEoOz6sGHvnSNjgRPy2w/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCpvOBCIxicGwsCb2ePwRPDIVHwXmWYHibV4ic4bKictMAvRUj9mrDDqMkFQ/640?wx_fmt=jpeg)

*   ### 业务领域的例子
    

业务领域有业务领域的语言，不同业务有不同语言。下面是腾讯动漫的一个例子。

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbChkHKWNAZHEChPBxPw4zF7wtT6v4JKc1rqWpNt0JQEpPPfsNbicZkAJg/640?wx_fmt=jpeg)

注：图片来源于内部同事的文章  

*   ### 技术领域的例子
    

以下是几个技术领域的例子，不同层次有不同层次的语言。

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCARvrEv9o8Sa39v8hCqibaDBHsF5Cx0uFuAloVd0cxg1uniamLwbQU8SA/640?wx_fmt=jpeg)

在《领域驱动设计 · 软件核心复杂性应对之道》这本书里，明确提到了设计模式，这可以看做比编程语言更高一个层级的语言，提高了思维的抽象层次。

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCduupuZaXZJO1icQQUwj9Phsswap25XNWwrAkK397yPgULPiahiar67sZw/640?wx_fmt=jpeg)

《微服务架构设计模式》和《面向模式的软件架构》在架构设计层面建立起了一套完整的模式语言，比设计模式再高一个层级。

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCgbwTmgCSmdck7cXMQubLlQltxs2Nh3GlqC7wQcyjp0Be4gdOFDAB3Q/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCQibgLvRcD5eLB3uMMyHhX12ldEZELTq9xXqlFDukgBnLdctevyLuvXA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCtYENWV4VhnruHC6xSXpT3o9meCMyWTqX1ffkK3wqskUadyXSRSAHlQ/640?wx_fmt=png)

*   ### 江湖派 vs 学院派
    

尽管在各自的著作中，两派是互不感冒，但在平时的工作中，通常不分派别，哪派管用用哪派。在诊断方法上，中医只能讲出望闻问切，西医能讲的东西就太多太多。在系统建模这个事情上，学院派能讲的东西，无论深度广度还是精度，都远超江湖派。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCPiblY5V48JEVjYDUWr4hjgpIqGRK4aUQeuqHRZ65TnUU68T4dyGtxicA/640?wx_fmt=png)

*   ### Model 是结合点
    

DDD、UML 和 Sysml 的底层逻辑全都指向 Model，Model 是三种技术的结合点。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCVJup0oqFtZDSacmYrctInGeicB9aJs8VtXZTuhSGqaaNE0Uz4Yj52rQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCZicpz1f4BmdFpKnEM0NXhxcE7EjfVntyQXzged5jkAUwN1vdMf5AM3w/640?wx_fmt=png)

*   ### 限界上下文是理解 DDD 的钥匙
    

限界上下文（Bounded Context）是 DDD 关键概念之一，同时可能是 DDD 里面最令人迷惑的一个概念。可以说，理解了限界上下文也就理解了 DDD。技术上，我们可以把限界上下文建模为 UML 的 Object/Class，或者是 SysML 的 Block，理解了这一点：Bounded Context = Object/Class = Block ，所有秘密迎刃而解。

有趣的是，《解构领域驱动设计》的作者，对限界上下文的理解也经历了一个禅宗式的参悟的过程：参禅之初，看山是山，看水是水；禅有悟时，看山不是山，看水不是水；禅中彻悟，看山仍然山，看水仍然是水。

大概所有研究 DDD 的人，都会经历类似的过程吧。从这个角度看，把一种技术方法讲成玄学，需要学习的人去慢慢参悟，这很难说不是一种退步。学院一派虽然繁琐但至少规范，而 DDD 在术语上的模棱两可进而导致很大的解释空间，让学习的人经常误入歧途，这是 DDD 的缺点之一。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCuccFLd1NY36EVn84z0ZicT06EhsAn1Ziaic1lhLG3Agqhye5hKzHzo3OA/640?wx_fmt=png)

*   ### UML/SysML 是重装备
    

学院一派，仍在不断精进，从多年前发展起来的 UML，到新近在其基础上发展起来的 SysML，建模技术正在从软件行业逐步拓展到一般技术行业。UML 强绑定于面向对象，SysML 去除了这种绑定，因而可用于更广泛的领域。在软件开发领域，一般学会 UML 就可以了，如果所属业务不采用面向对象开发模式的话，SysML 则是更适合的建模工具。

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCZrMzEAiaZicyLWHPZUrxvCrxGdkTIf8jWRUFwSdOA7lIpsNw4k5iaVf4g/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCNluu4Rd8VHp2s1286vSmibxfBHz3Ynx1a54Lucg0pACRL86AmOh8ibeA/640?wx_fmt=jpeg)

下面是一些用 UML 做的图，总体视觉效果看着还不错。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbCuibITpXe4noBXSgIiaHoSXGU27KicCcbIhmo1QicCVMrCMT3VI9HM1F2Wg/640?wx_fmt=png)

*   ### 架起桥梁
    

有一段时间，天天在琢磨 DDD/UML/SysML，脑子里各派的技术体系在翻江倒海地打架，偶然有一次把几幅图放在一起，突然间眼前灵光乍现，刹那间顿悟了，那一刻终于找到了解开密码的钥匙。从此困扰自己很久的 DDD 的几个核心概念，终于理清了头绪，完成了整个 DDD 大厦的最后一块拼图，感觉自己终于可以从路的这头走到路的那头，再从路的那头走到路的这头。注意下面这张图中左下角的表格，将 DDD/UML/SysML 之间的核心概念对应起来，三者之间的桥梁得以建立，从此一通百通。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbC1ou06abqxIRva9VAZOSp6NG9nlRgRwqibQR0nFLBA8bWXo1Hc0rfFOg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97UhKPPSWdtV7eIicicPCNLbC1jzQBcqq8Z9YUrkyjUB8VsMKVCUvytAQAasHiaGWkHy6VI5bAzZPXZA/640?wx_fmt=png)

DDD 自创立以来，到现在近 20 年，这期间 DDD 野蛮生长，逐渐变为一个杂乱无章的的技术丛林，一百本书有一百种讲法，包罗万象。而同样源远流长的 UML/SysML/RUP 等专业方法，却逐渐式微。

DDD 为什么这么香？其中的奥秘在哪？就 DDD 本身而言，顶多算一种思想体系，远未发展为一种规范方法，其解释空间很大导致人们的发挥空间也很大，这就非常适合技术咨询界拿来做包装讲故事。相应的，UML/SysML/RUP 等专业方法，因为严谨所以可发挥的空间就少，最关键的是，UML/SysML/RUP 等都有版权保护，各自也都推出了自己的专业认证体系，这就阻止了众多技术咨询公司的进入。

在中文社区里，主要是众多咨询公司在热情推动 DDD，沉寂多年的 DDD 进而又重出江湖。最近新出的一些书，各种 DDD 培训课，就其内容而言，要么是引经据典照着书本讲，要么就是顶着 DDD 的帽子讲的却是 UML 的内容。总的来说，DDD 没多少新东西，只是把同样的东西，以区别于学院派的语言，重新归纳了一遍，从而看起来像是新的东西。例如，把概要设计讲成战略建模，把详细设计讲成战术建模，没有本质区别，听起来高大上而已。

本人学院派出身，对学院一派的技术更熟悉也更擅长，奈何江湖一派大行其道，从耳濡目染到深入研究，历时数载，若有所得，在方法论上认识到一点：以学院一派的知识体系为根基，用江湖一派的方法去讲故事，两派融合方可修得上乘武功。有点类似中西医结合，以西医打底，以中医行走江湖，虽不中亦不远。伟人曾说，自己对词的欣赏趣味是 “偏于豪放，不废婉约”，我对不同技术流派的态度是 “偏于学院，不废江湖”。

当人们都在做多的时候，我选择做少，正本清源，理清主脉。试图在江湖派和学院派之间架起一座沟通的桥梁，是我一直想做的事情，本文只是一个起点。

以上就是关于 DDD 的核心脉络，如果文章对你有帮助，欢迎转发分享。

-End-

原创作者｜刘啸南

📢 你是怎么理解领域驱动设计的？有什么好的学习方法推荐吗？欢迎留言。我们将挑选一则最有意义的评论，为其留言者送出腾讯定制 - 咖啡杯 1 个（见下图）。10 月 25 日中午 12 点开奖。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95ic1qSEhRg8icIyISlSP81Iia5sOQq450Bj56K8fKqbV57xibHWIwFHuuF8D0epcJm0gYcS5F4MxziaPw/640?wx_fmt=png)

📢📢欢迎加入腾讯云开发者社群，社群专享券、大咖交流圈、第一手活动通知、限量鹅厂周边等你来~

  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Ap0CNBADIU4jiacH4CONuDN4M5kdp8KapCTJxNv9ZM3Qv0JXLD7IIcRib2ZeQWnPpol4WFImUE7JrwOLbs85HJww/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

（长按图片立即扫码）

  

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97J6SMQXfKJyNSyiaXODfAr6iaEunyQTZpmxY5ryuBSryD0BFgvCHl6ypzicw9bjRwyDjVBlvcHmELBg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

[![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95ic1qSEhRg8icIyISlSP81IiaXlUvNtngrr1nicwjeG3htcLlgetAEETHNOUruuFPlheibFe0M0nNTC1g/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=MzI2NDU4OTExOQ==&mid=2247657683&idx=1&sn=24ed88d436815c12d59eca01aed91310&chksm=eaa67283ddd1fb95c6d75ab293a3bcf782d213716ae5bf45fb96d10714ec2140f03f91a98900&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95ic1qSEhRg8icIyISlSP81Iiaf6fxo09Ztic184qyslnAGmHgv7cH7xUR4L0G69LPEArrb8JVZIWvVDg/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=MzI2NDU4OTExOQ==&mid=2247661708&idx=1&sn=b87cb7307ad2cf7c43536c956c6f4769&chksm=eaa662dcddd1ebca914b8691be68ebb9fe9fbdd03f2b3ed5c2d8e51fc9aa9ed77ea387bc2263&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe94leZfmPPdKJxKNd2nfPelNzoZRTESxDaRqg1NdrbN8HCibQ6TiaBQNMYCgfibLyVMARR8pL8sjQSLkA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)](http://mp.weixin.qq.com/s?__biz=MzI2NDU4OTExOQ==&mid=2247655743&idx=1&sn=902429c7eece1fe41bf0710f1aaa0ce2&chksm=eaa67b6fddd1f279a19cb4b67deca31bd2ae77cd2cd15ad8dda6b88b1a5ec226b6aad94532ba&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97J6SMQXfKJyNSyiaXODfAr6L7eeKJg8DF6FOZ7nKichx9xuZd0GfujDYZYA5prHP6S09WRx3XXfMTw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI2NDU4OTExOQ==&action=getalbum&album_id=3042498743402725378#wechat_redirect)

关注并星标腾讯云开发者

第一时间看鹅厂架构设计经验

（PS：关注公众号回复 1018 可获取本期封面表情包~）