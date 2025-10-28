---
source: https://mp.weixin.qq.com/s/nq0mVMsIa2kOyiybwSruNg
create: 2025-10-22 21:30
read: true
knowledge: true
knowledge-date: 2025-10-23
tags:
  - Spring
  - 框架原理
summary: "[[Spring 的三级缓存]]"
---
![](http://mmbiz.qpic.cn/mmbiz_gif/AAQtmjCc74DZeqm2Rc4qc7ocVLZVd8FOASKicbMfKsaziasqIDXGPt8yR8anxPO3NCF4a4DkYCACam4oNAOBmSbA/640?wx_fmt=gif&wxfrom=5&wx_lazy=1#imgIndex=0)

**目录**

一、背景

二、相关知识点简介

 1. 循环依赖

 2.Spring 创建 Bean 主要流程

三、案例分析

 1. 代码分析

 2. 问题分析

 3. 解决方案

四、总结

**一**

**背 景**

预发环境一个后台服务 admin 突然启动失败，异常如下：

```java
org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'timeoutNotifyController': Injection of resource dependencies failed; nested exception is org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'spuCheckDomainServiceImpl': Bean with name 'spuCheckDomainServiceImpl' has been injected into other beans [...] in its raw version as part of a circular reference, but has eventually been wrapped. This means that said other beans do not use the final version of the bean. This is often the result of over-eager type matching - consider using 'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.
        at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:598)
        at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:90)
        at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessProperties(AutowiredAnnotationBeanPostProcessor.java:376)
        at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1404)
        at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:592)
        at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:515)
        at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:320)
        at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:222)
        at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:318)
        at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199)
        at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:847)
        at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:877)
        at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:549)
        at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:141)
        at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:744)
        at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:391)
        at org.springframework.boot.SpringApplication.run(SpringApplication.java:312)
        at org.springframework.boot.SpringApplication.run(SpringApplication.java:1215)
        at org.springframework.boot.SpringApplication.run(SpringApplication.java:1204)
        at com.shizhuang.duapp.commodity.interfaces.admin.CommodityAdminApplication.main(CommodityAdminApplication.java:100)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.springframework.boot.loader.MainMethodRunner.run(MainMethodRunner.java:48)
        at org.springframework.boot.loader.Launcher.launch(Launcher.java:87)
        at org.springframework.boot.loader.Launcher.launch(Launcher.java:51)
        at org.springframework.boot.loader.PropertiesLauncher.main(PropertiesLauncher.java:578)
```

错误日志中明确写道：“**Bean has been injected into other beans ... in its raw version as part of a circular reference, but has eventually been wrapped.**” 这不仅仅是一个简单的循环依赖错误。它揭示了一个更深层次的问题：当循环依赖遇上 Spring 的 AOP 代理（如 @Transactional 事务、自定义切面等），Spring 在解决依赖的时，不得已将一个 “半成品”（原始 Bean）注入给了其他 30 多个 Bean。而当这个“半成品” 最终被 “包装”（代理）成“成品” 时，先前那些持有 “半成品” 引用的 Bean 们，使用的却是一个错误的版本。

这就像在组装一个精密机器时，你把一个未经质检的零件提前装了进去，等质检完成后，机器里混用着新旧版本的零件，最终的崩溃也就不可避免。

本篇文章将带你一起：

*   熟悉 spring 容器的循环依赖以及 Spring 容器如何解决循环依赖，创建 bean 相关的流程。
    
*   深入解读这条复杂错误日志背后的每一个关键线索；
    
*   提供紧急止血方案；
    
*   分享如何从架构设计上避免此类问题的实践心得。
    

**二**

**相关知识点简介**

**循环依赖**

**什么是 Bean 循环依赖？**

循环依赖：说白是一个或多个对象实例之间存在直接或间接的依赖关系，这种依赖关系构成了构成一个环形调用，主要有如下几种情况。

第一种情况：自己依赖自己的直接依赖

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74CHLEiajvx2Ch4h8qXxj7jydeLbK0GQ2ZfnfQ67n3JupFbgPibtV2V0fExNtmXicZeGSDTnnaUsmo7aA/640?wx_fmt=png&from=appmsg#imgIndex=1)

第二种情况：两个对象之间的直接依赖

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74CHLEiajvx2Ch4h8qXxj7jyd5x3NAjINE9n3YicibbvOiblUvZsicnFpk1sM5bSyBHqVMITQoOdGwC7ibUg/640?wx_fmt=png&from=appmsg#imgIndex=2)

前面两种情况的直接循环依赖比较直观，非常好识别，但是第三种间接循环依赖的情况有时候因为业务代码调用层级很深，不容易识别出来。

**循环依赖场景**

**构造器注入循环依赖：**

```
@Service
public class A {public A(B b) {}}
@Service
public class B {public B(A a) {}}
```

结果：项目启动失败抛出异常 BeanCurrentlyInCreationException

```java
Caused by: org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'a': Requested bean is currently in creation: Is there an unresolvable circular reference?
        at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.beforeSingletonCreation(DefaultSingletonBeanRegistry.java:339)
        at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:215)
        at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:318)
        at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199)
```

构造器注入构成的循环依赖，此种循环依赖方式无论是 Singleton 模式还是 prototype 模式都是无法解决的，只能抛出 BeanCurrentlyInCreationException 异常表示循环依赖。原因是 Spring 解决循环依赖依靠的是 Bean 的 “中间态” 这个概念，而中间态指的是已经实例化，但还没初始化的状态。而完成实例化需要调用构造器，所以构造器的循环依赖无法解决。

**Singleton 模式 field 属性注入（setter 方法注入）循环依赖：**

**这种方式是我们最为常用的依赖注入方式：**

```java
@Service
public class A {
    @Autowired
    private B b;
    }
@Service
public class B {
    @Autowired
    private A a;
    }
```

结果：项目启动成功，正常运行

**prototype field 属性注入循环依赖：**

**prototype 在平时使用情况较少，但是也并不是不会使用到，因此此种方式也需要引起重视。**

```java
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
@Service
public class A {
    @Autowired
    private B b;
    }
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
@Service
public class B {
    @Autowired
    private A a;
    }
```

结果：**需要注意的是本例中启动时是不会报错的**（因为非单例 Bean 默认不会初始化，而是使用时才会初始化），所以很简单咱们只需要手动 getBean() 或者在一个单例 Bean 内 @Autowired 一下它即可。

```java
// 在单例Bean内注入
    @Autowired
    private A a;
```

这样子启动就报错：

```java
org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'mytest.TestSpringBean': Unsatisfied dependency expressed through field 'a'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'a': Unsatisfied dependency expressed through field 'b'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'b': Unsatisfied dependency expressed through field 'a'; nested exception is org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'a': Requested bean is currently in creation: Is there an unresolvable circular reference?
        at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:596)
        at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:90)
        at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessProperties(AutowiredAnnotationBeanPostProcessor.java:374)
```

如何解决？可能有的小伙伴看到网上有说使用 @Lazy 注解解决：

```java
    @Lazy
    @Autowired
    private A a;
```

此处负责任的告诉你这样是解决不了问题的 (**可能会掩盖问题**)，@Lazy 只是延迟初始化而已，当你真正使用到它（初始化）的时候，依旧会报如上异常。

对于 Spring 循环依赖的情况总结如下：

*   不能解决的情况：构造器注入循环依赖，prototype field 属性注入循环依赖
    
*   能解决的情况：field 属性注入（setter 方法注入）循环依赖
    

**Spring 如何解决循环依赖**

Spring 是通过三级缓存和提前曝光的机制来解决循环依赖的问题。

**三级缓存**

三级缓存其实就是用三个 Map 来存储不同阶段 Bean 对象。

```java
一级缓存
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
二级缓存
private final Map<String, ObjectearlySingletonObjects = new HashMap<>(16);
//三级缓存
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16)
```

*   singletonObjects：用于存放完全初始化好的 bean，**从该缓存中取出的 bean 可以直接使用**。
    
*   earlySingletonObjects：提前曝光的单例对象的 cache，存放原始的 bean 对象（尚未填充属性），用于解决循环依赖。
    
*   singletonFactories：单例对象工厂的 cache，存放 bean 工厂对象，用于解决循环依赖。
    

**三级缓存解决循环依赖过程**

假设现在我们有 ServiceA 和 ServiceB 两个类，这两个类相互依赖，代码如下：

```java
@Service
public class ServiceA {
    @Autowired
    private ServiceB serviceB;
    }

@Service
public class ServiceB {
    @Autowired
    private ServiceA serviceA ;
    }
```

下面的时序图说明了 spring 用三级缓存解决循环依赖的主要流程：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74CHLEiajvx2Ch4h8qXxj7jyd2uKciaWr3ZUNyK02z3nlQpYs6LxhJAnQQicDhU3BhBa05ibUOiaDotGe8g/640?wx_fmt=png&from=appmsg#imgIndex=3)

**为什么需要三级缓存？**

这是一个理解 Spring 容器如何解决循环依赖的核心概念。**三级缓存**是 Spring 为了解决**循环依赖**的同时，又能保证 **AOP 代理**的正确性而设计的精妙机制。

为了理解为什么需要三级缓存，我们一步步来看。

**如果没有缓存（Level 0）**

假设有两个 Bean：ServiceA  和 ServiceB，它们相互依赖。

_Java_

```java
@Component
public class ServiceA {
    @Autowired
    private ServiceB serviceB;
}
@Component
public class ServiceB {
    @Autowired
    private ServiceA serviceA;
}
```

**创建过程（无缓存）**：

*   开始创建 ServiceA -> 发现 ServiceA 需要 ServiceB -> 开始创建 ServiceB
    
*   开始创建 ServiceB -> 发现 ServiceB 需要 ServiceA -> 开始创建 ServiceA
    
*   开始创建 ServiceA -> 发现 ServiceA 需要 ServiceB -> ... 无限循环，StackOverflowError
    

**结论**：无法解决循环依赖，直接死循环。

**如果只有一级缓存（Singleton Objects）**

一级缓存存放的是**已经完全创建好、初始化完毕**的 Bean。

**问题**：在 Bean 的创建过程中（比如在填充属性 populateBean 时），ServiceA 还没创建完，它本身不应该被放入 "已完成" 的一级缓存。但如果 ServiceB 需要 ServiceA，而一级缓存里又没有 ServiceA 的半成品，ServiceB 就无法完成创建。这就回到了上面的死循环问题。

**结论**：一级缓存无法解决循环依赖。

**如果使用二级缓存**

二级缓存的核心思路是：**将尚未完全初始化好的 “早期引用” 暴露出来。**

现在我们有：

*   **一级缓存（成品库）**：存放完全准备好的 Bean。
    
*   **二级缓存（半成品库）**：存放刚刚实例化（调用了构造方法），但还未填充属性和初始化的 Bean 的早期引用。
    

**创建过程（二级缓存）：**

**开始创建 ServiceA**：

*   实例化 ServiceA（调用 ServiceA 的构造方法），得到一个 ServiceA 的原始对象。
    
*   将 ServiceA 的原始对象放入二级缓存（半成品库）。
    
*   开始为 ServiceA 填充属性 -> 发现需要 ServiceB。
    

**开始创建 ServiceB**：

*   实例化 ServiceB（调用 B 的构造方法），得到一个 ServiceB 的原始对象。
    
*   将 ServiceB 的原始对象放入二级缓存。
    
*   开始为 ServiceB 填充属性 -> 发现需要 ServiceA。
    

**ServiceB 从二级缓存中获取 A**：

*   ServiceB 成功从二级缓存中拿到了 ServiceA 的早期引用（原始对象）。
    
*   ServiceB 顺利完成了属性填充、初始化等后续步骤，成为一个完整的 Bean。
    
*   将完整的 ServiceB 放入一级缓存（成品库），并从二级缓存移除 ServiceB。
    

**ServiceA 继续创建：**

*   ServiceA 拿到了创建好的 ServiceB，完成了自己的属性填充和初始化。
    
*   将完整的 ServiceA 放入一级缓存（成品库），并从二级缓存移除 ServiceA。
    

**问题来了：如果 ServiceA 需要被 AOP 代理怎么办？**

如果 A 类上加了 @Transactional 等需要创建代理的注解，那么最终需要暴露给其他 Bean 的**应该是 ServiceA 的代理对象，而不是 ServiceA 的原始对象。**

在二级缓存方案中，ServiceB 拿到的是 A 的**原始对象**。但最终 **ServiceA** 完成后，放入一级缓存的是 **ServiceA 的代理对象**。这就导致了：

*   **ServiceB 里面持有的 ServiceA 是原始对象。**
    
*   **而其他地方注入的 ServiceA 是代理对象。**
    
*   这就造成了**不一致**！如果通过 ServiceB 的 ServiceA 去调用事务方法，事务会失效，因为那是一个没有被代理的原始对象。
    

**结论**：二级缓存可以解决循环依赖问题，但无法正确处理需要 AOP 代理的 Bean。

**三级缓存的登场（Spring 的终极方案）**

为了解决代理问题，Spring 引入了第三级缓存。它的核心不是一个直接存放对象（Object）的缓存，而是一个存放 **ObjectFactory（对象工厂）**的缓存。

三级缓存的结构是：Map<String, ObjectFactory\<?>> singletonFactories

**创建过程（三级缓存，以 ServiceA 需要代理为例）：**

*   **开始创建 ServiceA**：
    

1.  实例化 ServiceA，得到 ServiceA 的原始对象。
    
2.  向**三级缓存**添加一个 ObjectFactory。这个工厂的 getObject() 方法**有能力判断 ServiceA 是否需要代理，并返回相应的对象（原始对象或代理对象）**。
    
3.  开始为 ServiceA 填充属性 -> 发现需要 ServiceB。
    

*   **开始创建 B**：
    

1.  实例化 ServiceB。
    
2.  同样向三级缓存添加一个 ServiceB 的 ObjectFactory。
    
3.  开始为 ServiceB 填充属性 -> 发现需要 ServiceA。
    

*   **ServiceB 从缓存中获取 ServiceA**：
    

1.  ServiceB 发现一级缓存没有 ServiceA，二级缓存也没有 ServiceA。
    
2.  ServiceB 发现三级缓存有 A 的 ObjectFactory。
    
3.  B 调用这个工厂的 getObject() 方法。**此时，Spring 会执行一个关键逻辑：**
    

1.  如果 ServiceA 需要被代理，工厂会**提前生成 ServiceA 的代理对象**并返回。
    
2.  如果 ServiceA 不需要代理，工厂则返回 A 的原始对象。
    

5.  将这个**早期引用（可能是原始对象，也可能是代理对象）**放入**二级缓存**，同时从三级缓存移除 A 的工厂。
    
6.  ServiceB 拿到了 ServiceA 的**正确版本**的早期引用。
    

**后续步骤：**

*   ServiceB 完成创建，放入一级缓存。
    
*   ServiceA 继续用 ServiceB 完成创建。在 ServiceA 初始化的最后，Spring 会再次检查：如果 ServiceA 已经被提前代理了（即在第 3 步中），那么就直接返回这个代理对象；如果没有，则可能在此处创建代理（对于不需要解决循环依赖的 Bean）。
    
*   最终，将完整的 ServiceA（代理对象）放入一级缓存，并清理二级缓存。
    

**总结：为什么需要三级缓存？**

需要三级缓存，是因为 Spring 要解决一个复杂问题：**在存在循环依赖的情况下，如何确保所有 Bean 都能拿到最终形态（可能被 AOP 代理）的依赖对象，而不是原始的、未代理的对象。** 三级缓存通过一个 ObjectFactory 将代理的时机提前，完美地解决了这个问题。二级缓存主要是为了性能优化而存在的。

**spring 三级缓存为什么不能解决**

**@Async 注解的循环依赖问题**

这触及了 Spring 代理机制的一个深层次区别。@Async 注解的循环依赖问题确实比 @Transactional 更复杂，**三级缓存无法完全解决**。让我们深入分析原因。

**Spring 创建 Bean 主要流程**

为了容易理解 Spring 解决循环依赖过程，我们先简单温习下 Spring 容器创建 Bean 的主要流程。

从代码看 Spring 对于 Bean 的生成过程，步骤还是很多的，我把一些扩展业务代码省略掉：

```java
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
          throws BeanCreationException {
    if (mbd.isSingleton()) {
      instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
    }
    // Bean初始化第一步：默认调用无参构造实例化Bean
    // 如果是只有带参数的构造方法，构造方法里的参数依赖注入，就是发生在这一步
    if (instanceWrapper == null) {
      instanceWrapper = createBeanInstance(beanName, mbd, args);
    }

    //判断Bean是否需要提前暴露对象用来解决循环依赖，需要则启动spring三级缓存
    boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
       isSingletonCurrentlyInCreation(beanName));
   if (earlySingletonExposure) {
     if (logger.isTraceEnabled()) {
       logger.trace("Eagerly caching bean '" + beanName +
             "' to allow for resolving potential circular references");
      }
    addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
}

    // Initialize the bean instance.
    Object exposedObject = bean;
    try {
      // bean创建第二步：填充属性（DI依赖注入发生在此步骤）
      populateBean(beanName, mbd, instanceWrapper);
      // bean创建第三步：调用初始化方法，完成bean的初始化操作（AOP的第三个入口）
      // AOP是通过自动代理创建器AbstractAutoProxyCreator的postProcessAfterInitialization()
//方法的执行进行代理对象的创建的,AbstractAutoProxyCreator是BeanPostProcessor接口的实现
      exposedObject = initializeBean(beanName, exposedObject, mbd);


   if (earlySingletonExposure) {
    Object earlySingletonReference = getSingleton(beanName, false);
     if (earlySingletonReference != null) {
        if (exposedObject == bean) {
          exposedObject = earlySingletonReference;
        }
        else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
          String[] dependentBeans = getDependentBeans(beanName);
          Set<String> actualDependentBeans = new LinkedHashSet<>(dependentBeans.length);
          for (String dependentBean : dependentBeans) {
             if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
                actualDependentBeans.add(dependentBean);
             }
          }
          if (!actualDependentBeans.isEmpty()) {
             throw new BeanCurrentlyInCreationException(beanName,
                   "Bean with name '" + beanName + "' has been injected into other beans [" +
                   StringUtils.collectionToCommaDelimitedString(actualDependentBeans) +
                   "] in its raw version as part of a circular reference, but has eventually been " +
                   "wrapped. This means that said other beans do not use the final version of the " +
                   "bean. This is often the result of over-eager type matching - consider using " +
                   "'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.");
          }
       }
    }
}

    } catch (Throwable ex) {
      // ...
    }
    // ...
    return exposedObject;
    }
```

从上述代码看出，整体脉络可以归纳成 3 个核心步骤：

*   实例化 Bean：主要是通过反射调用默认构造函数创建 Bean 实例，此时 Bean 的属性都还是默认值 null。被注解 @Bean 标记的方法就是此阶段被调用的。
    
*   填充 Bean 属性：这一步主要是对 Bean 的依赖属性进行填充，对 @Value、@Autowired、@Resource 注解标注的属性注入对象引用。
    
*   调用 Bean 初始化方法：调用配置指定中的 init 方法，如 xml 文件指定 Bean 的 init-method 方法或注解 @Bean(initMethod = "initMethod") 指定的方法。
    

**三**

**案例分析**

**代码分析**

以下是我简化后的类之间大体的依赖关系，工程内实际的依赖情况会比这个简化版本复杂一些。

```java
@RestController
public class OldCenterSpuController {
    @Resource
    private NewSpuApplyCheckServiceImpl newSpuApplyCheckServiceImpl;
}
@RestController
public class TimeoutNotifyController {
    @Resource
    private SpuCheckDomainServiceImpl spuCheckDomainServiceImpl;
}
@Component
public class NewSpuApplyCheckServiceImpl {
    @Resource
    private SpuCheckDomainServiceImpl spuCheckDomainServiceImpl;
}
@Component
@Slf4j
@Validated
public class SpuCheckDomainServiceImpl {
    @Resource
    private NewSpuApplyCheckServiceImpl newSpuApplyCheckServiceImpl;
}
```

从代码看，主要是 SpuCheckDomainServiceImpl 和 NewSpuApplyCheckServiceImpl 构成了一个依赖环。而我们从正常启动的 bean 加载顺序发现首先是从 OldCenterSpuController 开始加载的，具体情况如下所示：

```java
OldCenterSpuController 
    ↓ (依赖)
NewSpuApplyCheckServiceImpl 
    ↓ (依赖)  
SpuCheckDomainServiceImpl 
    ↓ (依赖)
NewSpuApplyCheckServiceImpl 
```

异常启动的情况 bean 加载是从 TimeoutNotifyController 开始加载的，具体情况如下所示：

```java
TimeoutNotifyController 
    ↓ (依赖)
SpuCheckDomainServiceImpl 
    ↓ (依赖)  
NewSpuApplyCheckServiceImpl 
    ↓ (依赖)
SpuCheckDomainServiceImpl 
```

同一个依赖环，为什么从 OldCenterSpuController 开始加载就可以正常启动，而从 TimeoutNotifyController 启动就会启动异常呢？下面我们会从现场 debug 的角度来分析解释这个问题。

**问题分析**

在相关知识点简介里面知悉到 spring 用三级缓存解决了循环依赖问题。为什么后台服务 admin 启动还会报循环依赖的问题呢？

要得到问题的答案，还是需要回到源码本身，前面我们分析了 spring 的创建 Bean 的主要流程，这里为了更好的分析问题，补充下通过容器获取 Bean 的。

在通过 spring 容器获取 bean 时，底层统一会调用 doGetBean 方法，大体如下：

```java
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
       @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
    
    final String beanName = transformedBeanName(name);
    Object bean;
    
    // 从三级缓存获取bean
    Object sharedInstance = getSingleton(beanName);
    if (sharedInstance != null && args == null) {
       bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
    }else {
     if (mbd.isSingleton()) {
       sharedInstance = getSingleton(beanName, () -> {
       try {
         //如果是单例Bean，从三级缓存没有获取到bean，则执行创建bean逻辑
          return createBean(beanName, mbd, args);
       }
       catch (BeansException ex) {
          destroySingleton(beanName);
          throw ex;
       }
    });
    bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
  }   
 }
```

从 doGetBean 方法逻辑看，在 spring 从一二三级缓存获取 bean 返回空时，会调用 createBean 方法去场景 bean，createBean 方法底层主要是调用前面我们提到的创建 Bean 流程的 doCreateBean 方法。

_注意：doGetBean 方法里面 getSingleton 方法的逻辑是先从一级缓存拿，拿到为空并且 bean 在创建中则又从二级缓存拿，二级缓存拿到为空 并且当前容器允许有循环依赖则从三级缓存拿。并且将对象工厂移到二级缓存，删除三级缓存_

doCreateBean 方法如下：

```java
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
          throws BeanCreationException {
    if (mbd.isSingleton()) {
      instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
    }
    // Bean初始化第一步：默认调用无参构造实例化Bean
    // 如果是只有带参数的构造方法，构造方法里的参数依赖注入，就是发生在这一步
    if (instanceWrapper == null) {
      instanceWrapper = createBeanInstance(beanName, mbd, args);
    }

    //判断Bean是否需要提前暴露对象用来解决循环依赖，需要则启动spring三级缓存
    boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
       isSingletonCurrentlyInCreation(beanName));
   if (earlySingletonExposure) {
     if (logger.isTraceEnabled()) {
       logger.trace("Eagerly caching bean '" + beanName +
             "' to allow for resolving potential circular references");
      }
    addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
}

    // Initialize the bean instance.
    Object exposedObject = bean;
    try {
      // bean创建第二步：填充属性（DI依赖注入发生在此步骤）
      populateBean(beanName, mbd, instanceWrapper);
      // bean创建第三步：调用初始化方法，完成bean的初始化操作（AOP的第三个入口）
      // AOP是通过自动代理创建器AbstractAutoProxyCreator的postProcessAfterInitialization()
//方法的执行进行代理对象的创建的,AbstractAutoProxyCreator是BeanPostProcessor接口的实现
      exposedObject = initializeBean(beanName, exposedObject, mbd);


   if (earlySingletonExposure) {
    Object earlySingletonReference = getSingleton(beanName, false);
     if (earlySingletonReference != null) {
        if (exposedObject == bean) {
          exposedObject = earlySingletonReference;
        }
        else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
          String[] dependentBeans = getDependentBeans(beanName);
          Set<String> actualDependentBeans = new LinkedHashSet<>(dependentBeans.length);
          for (String dependentBean : dependentBeans) {
             if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
                actualDependentBeans.add(dependentBean);
             }
          }
          if (!actualDependentBeans.isEmpty()) {
             throw new BeanCurrentlyInCreationException(beanName,
                   "Bean with name '" + beanName + "' has been injected into other beans [" +
                   StringUtils.collectionToCommaDelimitedString(actualDependentBeans) +
                   "] in its raw version as part of a circular reference, but has eventually been " +
                   "wrapped. This means that said other beans do not use the final version of the " +
                   "bean. This is often the result of over-eager type matching - consider using " +
                   "'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.");
          }
       }
    }
}

    } catch (Throwable ex) {
      // ...
    }
    // ...
    return exposedObject;
    }
```

将 doGetBean 和 doCreateBean 的逻辑转换成流程图如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74CHLEiajvx2Ch4h8qXxj7jydO8YpR6FaFKdgzic0gOeKOL7ib9T3YjuKNlV8eZSrtpDwFibMXpGufrZiag/640?wx_fmt=png&from=appmsg#imgIndex=4)

从流程图可以看出，后台服务 admin 启动失败抛出 UnsatisfiedDependencyException 异常的必要条件是存在循环依赖，因为不存在循环依赖的情况 bean 只会存在单次加载，单次加载的情况 bean 只会被放进 spring 的第三级缓存。

而触发 UnsatisfiedDependencyException 异常的先决条件是需要 spring 的第一二级缓存有当前的 bean。所以可以知道当前 bean 肯定存在循环依赖。在存在循环依赖的情况下，当前 bean 被第一次获取（即调用 doGetBean 方法）会缓存进 spring 的第三级缓存，然后会注入当前 bean 的依赖（即调用 populateBean 方法），在当前 bean 所在**依赖环内其他 bean 都不在一二级缓存的情况下**，会触发当前 bean 的第二次获取（即调用 doGetBean 方法），由于第一次获取已经将 Bean 放进了第三级缓存，spring 会将 Bean 从第三级缓存移到二级缓存并删除第三级缓存。

最终会回到第一次获取的流程，调用初始化方法做初始化。最终在初始化有对当前 bean 做代理增强的并且提前暴露到二级缓存的对象有被其他依赖引用到，而且 allowRawInjectionDespiteWrapping=false 的情况下，会导致抛出 UnsatisfiedDependencyException，进而导致启动异常。

_注意：在注入当前 bean 的依赖时，这里 spring 将 Bean 从第三级缓存移到二级缓存并删除第三级缓存后，当前 bean 的依赖的其他 bean 会从二级缓存拿到当前 bean 做依赖。这也是后续抛异常的先决条件_

结合 admin 有时候启动正常，有时候启动异常的情况，这里猜测启动正常和启动异常时 bean 加载顺序不一致，进而导致启动正常时当前 Bean 只会被获取一次，启动异常时当前 bean 会被获取两次。为了验证猜想，我们分别针对启动异常和启动正常的 bean 获取做了 debug。

**debug 分析**

首先我们从启动异常提取到以下关键信息，从这些信息可以知道是 spuCheckDomainServiceImpl 的加载触发的启动异常。所以我们这里以 spuCheckDomainServiceImpl 作为前面流程分析的当前 bean。

```java
org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'timeoutNotifyController': Injection of resource dependencies failed; nested exception is org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'spuCheckDomainServiceImpl': Bean with name 'spuCheckDomainServiceImpl' has been injected into other beans [...] in its raw version as part of a circular reference, but has eventually been wrapped. This means that said other beans do not use the final version of the bean. This is often the result of over-eager type matching - consider using 'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.
```

然后提前我们在 doCreateBean 方法设置好 spuCheckDomainServiceImpl 加载时的条件断点。我们先 debug 启动异常的情况。最终断点信息如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74CHLEiajvx2Ch4h8qXxj7jydicTLCjPxJtyS2Fr8Q9oqNJ6iaRGlWdoJaM8ick1lJxF4RibY1L8bxO8dcg/640?wx_fmt=png&from=appmsg#imgIndex=5)

从红框 1 里面的两个引用看，很明显调 initializeBean 方法时 spring 有对 spuCheckDomainServiceImpl 做代理增强。导致 initializeBean 后返回的引用和提前暴露到二级缓存的引用是不一致的。这里 spuCheckDomainServiceImpl 有二级缓存是跟我们前面分析的吻合，是因为 spuCheckDomainServiceImpl 被获取了两次，即调了两次 doGetBean。

从红框 2 里面的 actualDependentBeans 的 set 集合知道提前暴露到二级缓存的引用有被其他 33 个 bean 引用到，也是跟异常提示的 bean 列表保持一致的。

**这里 spuCheckDomainServiceImpl 的加载为什么会调用两次 doGetBean 方法呢？**

从调用栈分析到该加载链如下：

```java
TimeoutNotifyController  ->spuCheckDomainServiceImpl-> newSpuApplyCheckServiceImpl-> ... ->spuCheckDomainServiceImpl
```

TimeoutNotifyController 注入依赖时第一次调用 doGetBean 获取 spuCheckDomainServiceImpl 时，从一二三级缓存获取不到，会调用 doCreateBean 方法创建 spuCheckDomainServiceImpl。

首先会将 spuDomainServiceImpl 放进 spring 的第三级缓存，然后开始调 populateBean 方法注入依赖，由于在循环中间的 newSpuApplyCheckServiceImpl 是第一次获取，一二三级缓存都获取不到，会调用 doCreateBean 去创建对应的 bean，然后会第二次调用 doGetBean 获取 spuCheckDomainServiceImpl，这时 spuCheckDomainServiceImpl 在第一次获取已经将 bean 加载到第三级缓存，所以这次 spring 会将 bean 从第三级缓存直接移到第二级缓存，并将第三级缓存里面的 spuCheckDomainServiceImpl 对应的 bean 删除，并直接返回二级缓存里面的 bean，不会再调 doCreateBean 去创建 spuCheckDomainServiceImpl。最终完成了循环中间的 bean 的初始化后（这里循环中间的 bean 初始化时依赖到的 bean 如果有引用到 spuCheckDomainServiceImpl 会调用 doGetBean 方法从二级缓存拿到 spuCheckDomainServiceImpl 提前暴露的引用），会回到第一次调用 doGetBean 获取 spuCheckDomainServiceImpl 时调用的 doCreateBean 方法的流程。继续调 initializeBean 方法完成初始化，然后将初始化完成的 bean 返回。最终拿初始化返回的 bean 引用跟二级缓存拿到的 bean 引用做对比，发现不一致，导致抛出 UnsatisfiedDependencyException 异常。

**那么这里为什么 spuCheckDomainServiceImpl 调用 initializeBean 方法完成初始化后与提前暴露到二级缓存的 bean 会不一致呢？**

看 spuCheckDomainServiceImpl 的代码如下：

```java
@Component
@Slf4j
@Validated
public class SpuCheckDomainServiceImpl {
    @Resource
    private NewSpuApplyCheckServiceImpl newSpuApplyCheckServiceImpl;
}
```

发现 SpuCheckDomainServiceImpl 类有使用到 **@Validated** 注解。查阅资料发现 **@Validated** 的实现是通过在 initializeBean 方法里面执行一个 org.springframework.validation.beanvalidation.MethodValidationPostProcessor 后置处理器实现的，MethodValidationPostProcessor 会对 SpuCheckDomainServiceImpl 做一层代理。导致 initializeBean 方法返回的 spuCheckDomainServiceImpl 是一个新的代理对象，从而最终导致跟二级缓存的不一致。

debug 视图如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74CHLEiajvx2Ch4h8qXxj7jyd8esBEs9DwgFwsib3nic9Ewpz31ElURjqx7r2RrtYYJwdYVDJ5n8j5ATA/640?wx_fmt=png&from=appmsg#imgIndex=6)

**那为什么有时候能启动成功呢？什么情况下能启动成功？**

我们继续 debug 启动成功的情况。最终观察到 spuCheckDomainServiceImpl 只会调用一次 **doGetBean**，而且从一二级缓存拿到的 spuCheckDomainServiceImpl 提前暴露的引用为 null，如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74CHLEiajvx2Ch4h8qXxj7jydNXHbicRwQsia76ibeYoQBb8lAXBbMZT2kl6Skkywia9NpazIRLyO1hIpeg/640?wx_fmt=png&from=appmsg#imgIndex=7)

这里为什么 spuCheckDomainServiceImpl 只会调用一次 **doGetBean** 呢？

首先我们根据调用栈整理到当前加载的引用栈：

```java
oldCenterSpuController-> newSpuApplyCheckServiceImpl-> ... ->spuCheckDomainServiceImpl -> newSpuApplyCheckServiceImpl
```

根据前面启动失败的信息我们可以知道，**spuCheckDomainServiceImpl** 处理依赖的环是：

```java
spuCheckDomainServiceImpl ->newSpuApplyCommandServiceImpl-> ... ->spuCheckDomainServiceImpl
```

失败的情况我们发现是从 spuCheckDomainServiceImpl 开始创建的，现在启动正常的情况是从 newSpuApplyCheckServiceImpl 开始创建的。

创建 newSpuApplyCheckServiceImpl 时，发现它依赖环中间这些 bean 会依次调用 doCreateBean 方法去创建对应的 bean。

调用到 spuCheckDomainServiceImpl 时，由于是第一次获取 bean，也会调用 doCreateBean 方法创建 bean，然后回到创建 spuCheckDomainServiceImpl 的 doCreateBean 流程，这里由于没有将 spuCheckDomainServiceImpl 的三级缓存移到二级缓存，所以不会导致抛出 UnsatisfiedDependencyException 异常，最终回到 newSpuApplyCheckServiceImpl 的 doCreateBean 流程，由于 newSpuApplyCheckServiceImpl 在调用 initializeBean 方法没有做代理增强，所以也不会导致抛出 UnsatisfiedDependencyException 异常。因此最后可以正常启动。

**这里我们会有疑问？类的创建顺序由什么决定的呢？**

通常不同环境下，代码打包后的 jar/war 结构、@ComponentScan 的 basePackages 配置细微差别，都可能导致 Spring 扫描和注册 Bean 定义的顺序不同。Java ClassLoader 加载类的顺序本身也有一定不确定性。如果 Bean 定义是通过不同的配置类引入的，配置类的加载顺序会影响其中所定义 Bean 的注册顺序。

**那是不是所有的类增强在有循环依赖时都会触发 UnsatisfiedDependencyException 异常呢？**

并不是，比如 @Transactional 就不会导致触发 UnsatisfiedDependencyException 异常。让我们深入分析原因。

核心区别在于代理创建时机不同。

@Transactional 的代理时机如下：

```java
// Spring 为 @Transactional 创建代理的流程
1. 实例化原始 Bean
2. 放入三级缓存（ObjectFactory）
3. 当发生循环依赖时，调用 ObjectFactory.getObject()
4. 此时判断是否需要事务代理，如果需要则提前创建代理
5. 将代理对象放入二级缓存，供其他 Bean 使用
```

@Validated 的代理时机：

```java
// @Validated 的代理创建在生命周期更晚的阶段
1. 实例化原始 Bean
2. 放入三级缓存（ObjectFactory）
3. 当发生循环依赖时，调用 ObjectFactory.getObject()
4. ❌ 问题：此时 @Validated 的代理还未创建！
5. 其他 Bean 拿到的是原始对象，而不是异步代理对象
```

问题根源：@Transactional 的代理增强是在三层缓存生成时触发的， @Validated 的增强是在初始化 bean 后通过后置处理器做的代理增强。

**解决方案**

**短期方案**

*   移除 SpuCheckDomainServiceImpl 类上的 Validated 注解
    
*   @lazy 解耦
    

*   原理是发现有 @lazy 注解的依赖为其生成代理类，依赖代理类，只有在真正需要用到对象时，再通过 getBean 的逻辑去获取对象，从而实现了解耦。
    

**长期方案**

**严格执行 DDD 代码规范**

这里是违反 DDD 分层规范导致的循环依赖。

**梳理解决历史依赖环**

通过梳理修改代码解决历史存在的依赖环。我们内部实现了一个能检测依赖环的工具，这里简单介绍一下实现思路，详情如下。

**日常循环依赖环：实战检测工具类解析**

在实际项目中，即使遵循了 DDD 分层规范和注入最佳实践，仍有可能因业务复杂或团队协作不充分而引入循环依赖。为了在开发阶段尽早发现这类问题，我们可以借助自定义的循环依赖检测工具类，在 Spring 容器启动后自动分析并报告依赖环。

**功能概述：**

*   条件启用：通过配置 circular.dependecy.analysis.enabled=true 开启检测；
    
*   依赖图构建：扫描所有单例 Bean，分析其构造函数、字段、方法注入及 depends-on 声明的依赖；
    
*   循环检测算法：使用 DFS 遍历依赖图，识别所有循环依赖路径；
    
*   通知上报：检测结果通过飞书机器人发送至指定接收人（targetId）。
    

简洁代码结构如下：

```java
@Component
@ConditionalOnProperty(value = "circular.dependency.analysis.enabled", havingValue = "true")
public class TimingCircularDependencyHandler extends AbstractNotifyHandler<NotifyData>
    implements ApplicationContextAware, BeanFactoryAware {
    
    @Override
    public Boolean handler(NotifyData data) {
        dependencyGraph = new HashMap<>();
        handleContextRefresh(); // 触发依赖图构建与检测
        return Boolean.TRUE;
    }
    
    private void buildDependencyGraph() {
        // 遍历所有Bean，解析其依赖关系
        // 支持：构造器、字段、方法、depends-on
    }
    
    private void detectCircularDependencies() {
        // 使用DFS检测环，记录所有循环路径
        // 输出示例：循环依赖1: A -> B -> C -> A
    }
}
```

**四**

**总结**

循环依赖暴露了代码结构的设计缺陷。理论上应通过分层和抽象来避免，但在复杂的业务交互中仍难以杜绝。虽然 Spring 利用三级缓存等机制默默解决了这一问题，使程序得以运行，但这绝不应是懈怠设计的借口。我们更应恪守设计原则，从源头规避循环依赖，构建清晰、健康的架构。

**往期回顾**

1. [Apex AI 辅助编码助手的设计和实践｜得物技术](https://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247541339&idx=1&sn=6d13fe7855ad4f350584f78e923a4a60&scene=21#wechat_redirect)

2. [从 JSON 字符串到 Java 对象：Fastjson 1.2.83 全程解析｜得物技术](https://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247541322&idx=1&sn=4fc8048deb13dccb6b6e85daaf93f701&scene=21#wechat_redirect)

3. [用好 TTL Agent 不踩雷：避开内存泄露与 CPU 100% 两大核心坑｜得物技术](https://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247541282&idx=1&sn=af1edf3514b35d807083282e0640b0bf&scene=21#wechat_redirect)

4. [线程池 ThreadPoolExecutor 源码深度解析｜得物技术](https://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247541243&idx=1&sn=e469e0729dccc5a27cc25291a03741f0&scene=21#wechat_redirect)

5. [基于浏览器扩展 API Mock 工具开发探索｜得物技术](https://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247541213&idx=1&sn=defd1bbb595f9b38af2390be29503dca&scene=21#wechat_redirect)

文 / 鲁班

关注得物技术，每周一、三更新技术干货

要是觉得文章对你有帮助的话，欢迎评论转发点赞～

未经得物技术许可严禁转载，否则依法追究法律责任。

“

**扫码添加小助手微信**

如有任何疑问，或想要了解更多技术资讯，请添加小助手微信：

![](https://mmbiz.qpic.cn/mmbiz_jpg/AAQtmjCc74CHLEiajvx2Ch4h8qXxj7jydx5plET2QH7DapScP3R3KKj7fFqzPjXrm9KrfVvl8xiao7Z2GdOHzTog/640?wx_fmt=jpeg&from=appmsg#imgIndex=8)