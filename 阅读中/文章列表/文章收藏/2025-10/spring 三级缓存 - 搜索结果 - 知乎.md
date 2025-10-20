---
source: https://www.zhihu.com/search?type=content&q=spring%20%E4%B8%89%E7%BA%A7%E7%BC%93%E5%AD%98
create: 2025-10-11 22:49
read: true
knowledge: true
knowledge-date: 2025-10-13
tags:
  - Spring
summary: "[[Spring 的三级缓存]]"
---
## 
[[Spring 进阶 - Spring IOC 实现原理详解之 Bean 实例化 (生命周期_ 循环依赖等) _ Java 全栈知识体系#BeanFactory 中 getBean 的主体思路]]
[_spring_ 为什么使用_三级缓存_而不是两级？](/question/445446018/answer/2443911807)[直达问题](https://www.zhihu.com/question/445446018)

[

![](https://pic1.zhimg.com/50/1f75a291a7f65d756aaf9bfacb5a8121_l.jpg?source=4e949a73)

](//www.zhihu.com/people/ding-meng-yang-69)

[今天你学 java 了吗](//www.zhihu.com/people/ding-meng-yang-69)

转行程序员一枚，用最通俗易懂的语言，带大家学习 java

[发布于 2022-04-17 16:14 ，编辑于 2023-02-09 20:31](/question/445446018/answer/2443911807)

说实话，大部分的回答就是在鸡同鸭讲，我问你为什么，你告诉我是什么...... 都在那里解释三级缓存的作用，而为什么要使用三级缓存而不是两级，甚至是一级，并没有自己的思考，没有改源码去 debug 测试。

首先先回答题主的几个疑问：

1. 实际上代理类是相当于持有一个原对象（spring 用的两种代理，Proxy 和 cglib 都是一样）：先创建对象，再创建代理类，再初始化原对象，和初始化之后再创建代理类，是一样的。

![](https://pic2.zhimg.com/v2-e9b32799bbed6a138bad80ca972dcbf7_1440w.jpg)

![](https://pic2.zhimg.com/v2-e9b32799bbed6a138bad80ca972dcbf7_r.jpg)

![](https://pic4.zhimg.com/v2-c8663e5f5c1cb46fc0b71b4b388dfc35_1440w.jpg)

![](https://pic4.zhimg.com/v2-c8663e5f5c1cb46fc0b71b4b388dfc35_r.jpg)

基于上述类写个 main 方法测试：

![](https://pic4.zhimg.com/v2-cdbc9cfc96757e8c67fdb2354275927d_1440w.jpg)

![](https://pic4.zhimg.com/v2-cdbc9cfc96757e8c67fdb2354275927d_r.jpg)

先根据空对象创建代理类，再初始化空对象，执行代理类方法，没问题！（忽略 get/set 方法）

2. 尾部对象依赖前面对象，所以在尾部对象初始化时，就调用三级缓存中对象工厂的接口方法，即 AbstractAutowireCapableBeanFactory#getEarlyBeanReference 方法根据前面的空对象创建代理类，并设值给尾部对象。

前面的对象在装配和初始化完成之后，spring 通过这段代码，将二级缓存中的代理类取出返回，最后会设置到一级缓存中，从而保证尾部对象依赖的，和容器中的前面对象，是一个对象。

![](https://pic1.zhimg.com/v2-84f4c39e29a8c48d2c28e85fecf5f7ba_1440w.jpg)

![](https://pic1.zhimg.com/v2-84f4c39e29a8c48d2c28e85fecf5f7ba_r.jpg)

* * *

下面说下我的结论：只用两级缓存可以解决循环依赖，甚至一级缓存就行（AOP 也同样适用）。

要理解下面的内容需要阅读过一定的 spring 源码，基础较弱的读者建议去看本回答的出处：[【超级干货】为什么 spring 一定要弄个三级缓存？](https://zhuanlan.zhihu.com/p/496273636)。以免错过此次刷新你认知的机会（不是吹牛）。

不信你可以改下添加三级缓存的源码，直接加入第二层缓存或者第一层缓存里。

![](https://pic2.zhimg.com/v2-6438b1808bd3c3317dd0a84671377e21_1440w.jpg)

![](https://pic2.zhimg.com/v2-6438b1808bd3c3317dd0a84671377e21_r.jpg)

启动正常，所以验证了上述结论

为什么呢？很简单，解决循环依赖只需要保证创建完成的 bean 和创建中与设置到其他相互引用的 bean 里的 bean 是同一个就行。

**没有代理的情况下（getEarlyBeanReference 返回原对象）去创建 AServiceImpl（简称 AS）：**

1.  反射创建 AS 的实例，并放入第一层缓存
2.  初始化 AS 实例时发现需要依赖注入 BS，则获取 BS 的实例
3.  反射创建 BS 的实例，并放入第一层缓存
4.  初始化 BS 实例时发现需要依赖注入 AS，则获取 AS 的实例，直接从第一层里获取
5.  BS 实例初始化完成，放入第一层缓存（此时 BS 里的 AS 只是刚创建完，未初始化）
6.  回到第 2 步，AS 实例初始化完成，放入第一层缓存（由于是同一个对象引用，所以 BS 里的 AS 也初始化完成）

**有代理的情况下（getEarlyBeanReference 返回代理对象）去创建 AServiceImpl（给 AServiceImpl 和 BServiceImpl 都加上事务注解）：**

前面 5 步都没啥问题，只是第一层缓存里的是 AS 代理类（BS 里的也是），第 6 步中，原 AS 对象初始化完成，则 AS 代理类其实也初始化完成，所以进行引用覆盖，返回缓存中的代理类即可。

**有问题吗？没问题！**

* * *

从技术的角度看一层缓存就能解决循环依赖，为什么 spring 要整的这么复杂呢？

请大家换个角度，不要再站在 spring 使用者的角度去思考了，现在要假设自己是 spring 的开发者！

不要在别人已经既定设计好的方案里去猜人家当时为什么这么设计，这就是从结果反推过程，说实话很容易潜意识认为就应该这样设计，从而变成想方设法圆他人所说。

**先抛却掉这几级缓存，重新审视下创建和初始化 bean 实例的代码。**

一个 bean 在创建过程中可能会产生两个对象：

*   一个是循环依赖时需要设值给与此 bean 相互引用的其他 bean 的对象（getEarlyBeanReference）
*   一个是初始化后的对象（initializeBean）

如果现在要对 bean 做增强，比如实现切面，则需要生成代理类，所以 spring 在上述两个方法中通过 BeanPostProcessor 类提供了拓展点。

![](https://pic2.zhimg.com/v2-8e298e3fa5be76227b452de93627962d_1440w.jpg)

![](https://pic2.zhimg.com/v2-8e298e3fa5be76227b452de93627962d_r.jpg)

![](https://pic2.zhimg.com/v2-0f1d0ca04842dd2ffc5fea60b87eed83_1440w.jpg)

![](https://pic2.zhimg.com/v2-0f1d0ca04842dd2ffc5fea60b87eed83_r.jpg)

假如我是 spring 这块代码的开发者，如果我这么设计（只用一层缓存）：

![](https://picx.zhimg.com/v2-28c0232c470d91117d923eb03b98dfd7_1440w.jpg)

![](https://picx.zhimg.com/v2-28c0232c470d91117d923eb03b98dfd7_r.jpg)

假设 spring 没有提供 AOP，需要使用 spring 的人自己去实现，那我就需要写个说明文档告诉他们：

1.  请实现 BeanPostProcessor 接口的两个方法：getEarlyBeanReference 和 postProcessAfterInitialization（假如我是设计者，按我的设计那我肯定都整合在一个接口里了，不会再整个 SmartInstantiationAwareBeanPostProcessor）
2.  这两个方法在 bean 刚创建完成但还未初始化时，和已装配并执行初始化方法之后会被调用，方法的入参 bean 分别是空对象和已初始化后的对象
3.  这两个方法的返回最好是同一个对象，如果不一样，由于最后引用会重新赋值，以 getEarlyBeanReference 方法返回为最终值

相信这样的文档会很让使用者很困惑：

1.  这两个方法都会执行，而且第一个方法的返回值为优先，所以我实现第二个方法干嘛呢？
2.  第一个方法的入参还是个空对象，没有什么有用的信息啊？
3.  第二个方法的入参里有有用的信息，但是返回的对象还是会被第一个方法的覆盖啊？

而且还暴露了很多内部设计细节，一个优秀的框架就是要让使用者对内部细节知道的越少越好，这样才便于迭代升级。

所以大家有没有发现，这种设计，**让 getEarlyBeanReference 成为了创建 bean 时必会被调用的核心方法**，而原本此处只是为了循环依赖时先给其他 bean 赋值！

倘若我们压根都没有循环依赖，bean 本身的创建流程就应该是先 new（一般是通过反射创建）一个，再装配初始化，最后放入实例缓存，此种设计就成了本末倒置！

那怎么调整呢？**让 getEarlyBeanReference 延迟触发，只在有循环依赖时被引用的 bean 需要赋值当前 bean 时才触发！**

所以就有了对象工厂 ObjectFactory，也就有了第三级缓存。

而**最理想的情况**下，可以不用第二级缓存。

举个例子，A 依赖 B，B 依赖 A 和 C 和 D，C 和 D 又依赖 A，创建 A 的时候初始化需要 B，创建 B 的时候初始化需要 A，拿到 A 的 ObjectFactory 后调用接口方法获取对象，B 还需要 C 和 D，它们又需要去调用 A 的 ObjectFactory，所以就重复调用了 getObject 方法，其实只要 getEarlyBeanReference 方法实现保证同一个 beanName 返回同一个对象，就不需要第二级缓存。

但是，这又暴露了内部实现细节，假如我弄个 BeanPostProcessor 实现类，spring 也没有提示我要遵守上述约定，而我在 getEarlyBeanReference 方法里只是创建新对象返回，这就会导致 B 里面的是 A1，C 里面是 A2，D 里面是 A3，那完了，芭比 Q 了！

所以让实现者去做重复性判断是不可控的，很容易出现问题，于是乎引入了第二级缓存，当调用三级缓存里的对象工厂的 getObject 方法之后，spring 就会把返回值放入二级缓存，删除三级缓存，这样 C 和 D 取的就是二级缓存里的 A 对象，和 B 里的是同一个。

**所以二级缓存和三级缓存其实是一套组合拳，不要拆成两个独立的东西去理解，出发点就不对。**

基于这种设计，没有发生循环依赖的 bean 就是正常的创建流程，有相互引用的 bean（除链尾的那个，比如之前的 BS）会触发 getEarlyBeanReference。

spring 其实也不想你用框架前还要先了解循环引用，所以把 getEarlyBeanReference 方法设计在了 SmartInstantiationAwareBeanPostProcessor 接口中。从设计者这里的注释也能看出：此接口是一个专用接口，主要用于框架内的内部使用！

![](https://pic3.zhimg.com/v2-c025e2198e7a8cd0117112222fd521cc_1440w.jpg)

![](https://pic3.zhimg.com/v2-c025e2198e7a8cd0117112222fd521cc_r.jpg)

* * *

AOP 是 spring 内部集成的，它的开发者知道这些逻辑，所以 AbstractAutoProxyCreator 实现了 SmartInstantiationAwareBeanPostProcessor 接口，可以看看其对这两个方法的实现。

![](https://pic3.zhimg.com/v2-7a398c83e309af537f61b017ad4b6e58_1440w.jpg)

![](https://pic3.zhimg.com/v2-7a398c83e309af537f61b017ad4b6e58_r.jpg)

循环引用的 bean 的创建过程会触发这两个方法，所以在 getEarlyBeanReference 方法中会打标记再去判断是否需要创建代理类（wrapIfNecessary），而 postProcessAfterInitialization 方法则需要先判断标记，以免重复执行 wrapIfNecessary。

也得亏根据空对象去创建代理类后再去初始化原对象，和根据已初始化后的对象创建代理类效果一样。  
否则这里 getEarlyBeanReference 方法就没啥用了，只能返回原对象，在 postProcessAfterInitialization 方法返回前加上把相互引用的其他 bean 的引用指向方法返回值的操作了，那就没现在这么简单了。

最后再对这三级缓存做个简单的总结：

*   第一层缓存：最基础的缓存，创建完并初始化（createBean）后的 bean 实例会放入，项目启动完成后获取 bean 实例时从此获取
*   第三层缓存：创建 bean 过程中用于处理循环依赖的临时缓存，由于只有在初始化时才知道有没有循环依赖，所以通过 ObjectFactory 临时 “存储” 刚创建完的 bean，并延迟触发循环依赖时被引用的 bean 需要赋值当前 bean 时去获取当前 bean 的逻辑，且获取对象会作为当前 bean 的最终对象
*   第二级缓存：创建 bean 过程中用于处理循环依赖的临时缓存，搭配第三层缓存，用于其 ObjectFactory 返回对象的缓存，保证多个关联对象对当前 bean 的引用为同一个

* * *

上面的内容大部分提炼自我的文章，语言比较精炼直接。如果读者本身对 spring 这块源码理解不深，建议去看文章，会从基础说起。

[![](https://pic1.zhimg.com/v2-ca73adb2c711435e29138f6cd87230ae.jpg?source=7e7ef6e2&needBackground=1)

今天你学 java 了吗：【超级干货】为什么 spring 一定要弄个三级缓存？180 赞同 · 59 评论](https://zhuanlan.zhihu.com/p/496273636) 文章

* * *

再推下我写过的比较受好评的一些高赞文章和回答：

[![](https://pic1.zhimg.com/v2-ec0fd35d792d1e962187d3ab0cf6f603.jpg?source=7e7ef6e2&needBackground=1)

今天你学 java 了吗：【spring 源码深度解析】：spring 是如何利用 @Transactional 注解实现数据库事务的？把握住事务的基本用法你就懂了 70 赞同 · 14 评论](https://zhuanlan.zhihu.com/p/358657396) 文章

[![](https://picx.zhimg.com/v2-7d0c5195ef43a8309b8bb2e231644967.jpg?source=7e7ef6e2&needBackground=1)

消息队列（mq）是什么？66 赞同 · 2 评论](https://www.zhihu.com/question/54152397/answer/2149218100) 回答

[![](https://picx.zhimg.com/v2-4780b674de31d19d3cf31e692b4dff3d.jpg?source=7e7ef6e2&needBackground=1)

Java 后端工作 3 年，尝试阅读 spring 源码，但发现根本看不下去。有什么方法能让阅读源码更得心应手?43 赞同 · 0 评论](https://www.zhihu.com/question/445632945/answer/1987266176) 回答

* * *

有一些朋友留言觉得内容很赞很硬核，其实得益于我阅读源码有一套自己的方法论，能抓住主线，下面给大家分享下。

[![](https://picx.zhimg.com/v2-68a5aed62735e147f21538c3714d721e.jpeg?source=7e7ef6e2&needBackground=1)

今天你学 java 了吗：【干货】高级 java 工程师必备技能之如何高效阅读源码（上篇：准备工作）5 赞同 · 0 评论](https://zhuanlan.zhihu.com/p/546042344) 文章

[![](https://picx.zhimg.com/v2-c4f03c99658a0ea6abfd549d9e1b833c.png?source=7e7ef6e2&needBackground=1)

今天你学 java 了吗：【干货】高级 java 工程师必备技能之如何高效阅读源码（下篇：springMVC 处理请求流程案例实操）8 赞同 · 0 评论](https://zhuanlan.zhihu.com/p/546799257) 文章

[发布于 2022-04-17 16:14 ，编辑于 2023-02-09 20:31](/question/445446018/answer/2443911807)

​赞同 1122​​收起评论

​分享

​收藏​喜欢

​

收起​

![](https://picx.zhimg.com/v2-55e9d81d17185afa48efdfef58d70735_l.jpg?source=32738c0c&needBackground=1)

理性发言，友善互动

  

89 条评论

默认

最新

[

![](https://picx.zhimg.com/v2-abed1a8c04700ba7d72b45195223e0ff_l.jpg?source=06d4cd63)

](https://www.zhihu.com/people/b44855593bb16691d44c23fa4589abb6)

[kirito](https://www.zhihu.com/people/b44855593bb16691d44c23fa4589abb6)

写的很好 当时看源码的时候就感到疑惑 网上查了查也几乎是不能看的  
也分享一下自己的想法  
二级缓存 主要考虑单例 / 多例的情况 + 尽可能保证 bean 执行流程只有在需要时 (出现循环依赖时) 才提前创建  
三级缓存 保证在该缓存中的是完成整个 bean 创建流程的, 不会是创建到一半的

2022-04-21

​回复​11

[

![](https://picx.zhimg.com/v2-8cca2909393e6c45dc54b5eec41a7e41_l.jpg?source=06d4cd63)

](https://www.zhihu.com/people/7187f1b56a030ab4f69b4aea96c35713)

[JOE](https://www.zhihu.com/people/7187f1b56a030ab4f69b4aea96c35713)

自己尝试写一个 aop 和 ioc 功能的小框架就能明白这个东西 自己写不考虑拓展等问题 一级缓存确实可以了 但是代理的情况下。增强类里面存储了被增强类得对象（因为要保证被增强类是属性注入过后的成品）这个也算做一个缓存吧

![](https://pic4.zhimg.com/v2-bffb2bf11422c5ef7d8949788114c2ab.png)

2022-05-19

​回复​7

[

![](https://picx.zhimg.com/1f75a291a7f65d756aaf9bfacb5a8121_l.jpg?source=06d4cd63)

](https://www.zhihu.com/people/2f632890efc38c2d8b29255195a65bed)

[今天你学 java 了吗](https://www.zhihu.com/people/2f632890efc38c2d8b29255195a65bed)

作者

我认为增强类只是在实现上是持有原对象，严格意义上不算一种缓存，毕竟缓存需要方便的 put 和 get，代理类不适用

2022-05-20

​回复​2

[

![](https://pica.zhimg.com/v2-8cca2909393e6c45dc54b5eec41a7e41_l.jpg?source=06d4cd63)

](https://www.zhihu.com/people/7187f1b56a030ab4f69b4aea96c35713)

[JOE](https://www.zhihu.com/people/7187f1b56a030ab4f69b4aea96c35713)

[今天你学 java 了吗](https://www.zhihu.com/people/2f632890efc38c2d8b29255195a65bed)

这倒也是 比较增强类缓存的对象只有增强类可用 并没有缓存的功能在里面

2022-05-20

​回复​1

[

![](https://pic1.zhimg.com/v2-927b4c2fe63d476115ff78bf051c5f1f_l.jpg?source=06d4cd63)

](https://www.zhihu.com/people/acd53c157e09314fe77393e661703a34)

[JokerZeroGxxx](https://www.zhihu.com/people/acd53c157e09314fe77393e661703a34)

感觉还是没看懂，不要三级缓存，二级缓存存放代理后的 bean 可以么？

2023-09-20

​回复​3

[

![](https://pica.zhimg.com/v2-abed1a8c04700ba7d72b45195223e0ff_l.jpg?source=06d4cd63)

](https://www.zhihu.com/people/5f6f98bff3aa7f87b868b74b559f173c)

[Tinyguns](https://www.zhihu.com/people/5f6f98bff3aa7f87b868b74b559f173c)

实际上为了应对提前 aop 用二级缓存也可以实现, 但是当我一个 bean 在需要被其他多个 bean 注入的时候, 你会发现设计三层缓存比二级缓存来实现更加优雅和合理

02-06

​回复​1

[

![](https://pic1.zhimg.com/v2-abed1a8c04700ba7d72b45195223e0ff_l.jpg?source=06d4cd63)

](https://www.zhihu.com/people/ede3514802c68535c53fda68b848abdd)

[张小明](https://www.zhihu.com/people/ede3514802c68535c53fda68b848abdd)

完全可以

2024-08-15

​回复​1

[

![](https://picx.zhimg.com/v2-d7f79622efec2d0eebe1d44deb22dd03_l.jpg?source=06d4cd63)

](https://www.zhihu.com/people/90eed364c9cc5160d6aebf5cac3583d1)

[云烟](https://www.zhihu.com/people/90eed364c9cc5160d6aebf5cac3583d1)

看得我有点头晕，system.err.print(今天不想学佳娃)

2023-02-18

​回复​3

[

![](https://picx.zhimg.com/v2-9c77024037eb3b58012252586d55648f_l.jpg?source=06d4cd63)

](https://www.zhihu.com/people/7d76b875edb291401024685c169f1379)

[浪得虚名](https://www.zhihu.com/people/7d76b875edb291401024685c169f1379)

平安老师有两把刷子

![](https://pic1.zhimg.com/v2-3ac403672728e5e91f5b2d3c095e415a.png)

2022-06-29

​回复​3

[

![](https://picx.zhimg.com/1f75a291a7f65d756aaf9bfacb5a8121_l.jpg?source=06d4cd63)

](https://www.zhihu.com/people/2f632890efc38c2d8b29255195a65bed)

[今天你学 java 了吗](https://www.zhihu.com/people/2f632890efc38c2d8b29255195a65bed)

作者

马上不在平安了

![](https://pic1.zhimg.com/v2-b62e608e405aeb33cd52830218f561ea.png)

2022-07-04

​回复​4

[

![](https://pic1.zhimg.com/d34c2c5d47d14790899a7db1d4ccbce6_l.jpg?source=06d4cd63)

](https://www.zhihu.com/people/c977d41274c990d52c0004cb70bbf894)

[文小名](https://www.zhihu.com/people/c977d41274c990d52c0004cb70bbf894)

[今天你学 java 了吗](https://www.zhihu.com/people/2f632890efc38c2d8b29255195a65bed)

原来是科技的老师呀

2024-06-25

​回复​喜欢

[

![](https://pica.zhimg.com/v2-abed1a8c04700ba7d72b45195223e0ff_l.jpg?source=06d4cd63)

](https://www.zhihu.com/people/1e545cefd3da8f9ad50e7c2720368ad7)

[CcIoJ](https://www.zhihu.com/people/1e545cefd3da8f9ad50e7c2720368ad7)

怎么就感觉我看的云里雾里的

2023-08-10

​回复​2

[

![](https://picx.zhimg.com/2264585baef9fb6bbdf6b5329c1e3980_l.jpg?source=06d4cd63)

](https://www.zhihu.com/people/2d1a4ce99c6bae617ccc5c633bd9d7f0)

[Asche](https://www.zhihu.com/people/2d1a4ce99c6bae617ccc5c633bd9d7f0)

太硬核了

![](https://pic1.zhimg.com/v2-0942128ebfe78f000e84339fbb745611.png)

2022-09-22

​回复​2

[

![](https://pic1.zhimg.com/1f75a291a7f65d756aaf9bfacb5a8121_l.jpg?source=06d4cd63)

](https://www.zhihu.com/people/2f632890efc38c2d8b29255195a65bed)

[今天你学 java 了吗](https://www.zhihu.com/people/2f632890efc38c2d8b29255195a65bed)

作者

哈哈，全网独此一家

![](https://pic4.zhimg.com/v2-c96dd18b15beb196b2daba95d26d9b1c.png)

2022-09-22

​回复​1

[

![](https://picx.zhimg.com/v2-e5df140ad12eec3f079367f89f530919_l.jpg?source=06d4cd63)

](https://www.zhihu.com/people/31294caae528ee3529b4467698d21017)

[趣谈编程](https://www.zhihu.com/people/31294caae528ee3529b4467698d21017)

作者你好，感谢分享，你的观点我大多数认同，只是有一个小疑问，就是我既然重新设计了，二层缓存也可以，getEarlyBeanReference 这个方法我都不要了，我在实例化后就进行代理类的生成，就不会有你说的 getEarlyBeanReference 重复调用还返回多个对象，那我也可以规定生成一个，或者不要这个方法，直接实例化后进行代理类的生成，所以我认为二级缓存是区分完整的 bean 和半成品的 bean 的一个缓存，你觉得尼

03-06

​回复​1

[

![](https://picx.zhimg.com/v2-be8e41938561c095713191a0d52c4e13_l.jpg?source=06d4cd63)

](https://www.zhihu.com/people/2782cf9ab4326b02f3ff4cd2d22a6148)

[杭二](https://www.zhihu.com/people/2782cf9ab4326b02f3ff4cd2d22a6148)

简单来讲，就是通过第三级缓存（不是三级共同作用）的延迟初始化来达到循环依赖 ** 一级缓存：** 存放初始化完全的 bean 实例缓存（用于查找，没有循环依赖一级缓存足够使用）  
** 三级缓存：** bean 在实例化之后，会放入的未初始化的 bean 工厂方法来延迟初始化  
本质是调用 `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#getEarlyBeanReference` 方法的 bean 工厂匿名类  
** 二级缓存：** 延迟初始化后（本质调用 getEarlyBeanReference 包装了 原先的 bean 实例）  
earlySingletonObjects.put(getEarlyBeanReference 返回的实例)  
singletonFactories.remove(bean 工厂实例)  
本质是防止 `getEarlyBeanReference`bean 方法由于每次延迟初始化返回不同实例而导致注入的 bean 不是同一个  
例如 `() -> getEarlyBeanReference(beanName, mbd, bean)` 会调用到构建代理的组件  
`org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator#getEarlyBeanReference`  
如果没有二级缓存，会重复创建代理的实例的问题

2023-03-10

​回复​1

[

![](https://pica.zhimg.com/v2-cbcf7a3947fa8b3bd181b97ae75a1359_l.jpg?source=06d4cd63)

](https://www.zhihu.com/people/e4d649159e3284eb08b2137ea0c0e324)

[深海鱼肝油](https://www.zhihu.com/people/e4d649159e3284eb08b2137ea0c0e324)

这里的延迟初始化是指创建代理类而不进行初始化吗

2023-07-11

​回复​喜欢

[

![](https://picx.zhimg.com/v2-d4b437abdbe4f23ccf61c40436b4373c_l.jpg?source=06d4cd63)

](https://www.zhihu.com/people/3e7ab73e94da66b7817839b6afcdf219)

[cheng](https://www.zhihu.com/people/3e7ab73e94da66b7817839b6afcdf219)

我觉得也是，作为使用者来说，没有必要纠结为什么要用二级 三级缓存。 从 spring 产品开发角度来说，实现的方案千千万，刚好选择了这一种。

2022-08-18

​回复​1

[

![](https://picx.zhimg.com/1f75a291a7f65d756aaf9bfacb5a8121_l.jpg?source=06d4cd63)

](https://www.zhihu.com/people/2f632890efc38c2d8b29255195a65bed)

[今天你学 java 了吗](https://www.zhihu.com/people/2f632890efc38c2d8b29255195a65bed)

作者

对的，解决循环依赖其实挺简单的，但是网上大部分观点都硬是强调三层缓存的重要性，搞的好像只有这种解决方法，读者的思维也会受限。

2022-08-18

​回复​2

[

![](https://picx.zhimg.com/af43f5937_l.jpg?source=06d4cd63)

](https://www.zhihu.com/people/0b81818990a067106a7fdf3d3486f881)

[骨灰级](https://www.zhihu.com/people/0b81818990a067106a7fdf3d3486f881)

[今天你学 java 了吗](https://www.zhihu.com/people/2f632890efc38c2d8b29255195a65bed)

循环依赖是个很 der 的东西，就应该直接禁止掉这玩意，纯粹是代码架构问题

04-03

​回复​喜欢

点击查看全部评论

收起评论​

![](https://picx.zhimg.com/v2-55e9d81d17185afa48efdfef58d70735_l.jpg?source=32738c0c&needBackground=1)

理性发言，友善互动