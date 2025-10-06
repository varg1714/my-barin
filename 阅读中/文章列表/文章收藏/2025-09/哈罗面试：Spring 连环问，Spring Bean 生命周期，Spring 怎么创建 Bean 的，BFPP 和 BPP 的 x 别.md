---
source: https://mp.weixin.qq.com/s/5GZ2FKFv0vDM0zHmUq7dog
create: 2025-09-15 11:30
read: true
knowledge: true
knowledge-date: 2025-10-03
tags:
  - Spring
summary: "[[Spring Bean 生命周期解析]]"
---
2025年09月15日 11:21@技术自由圈 2025年09月15日 11:21

FSAC未来超级架构师

架构师总动员  
实现架构转型，再无中年危机

  

# 尼恩说在前面：

最近大厂机会多了， 在45岁老架构师 尼恩的**读者交流群**(50+)中，最近有小伙伴拿到了一线互联网企业如得物、阿里、滴滴、极兔、有赞、shein 希音、shopee、百度、网易的面试资格，遇到很多很重要的面试题：

> Spring连环问，Spring Bean生命周期?
> 
> Spring怎么创建Bean的?
> 
> FactoryBeanPostProssor和BeanPostProssor的x别?

前几天 小伙伴面试 哈罗，遇到了这个问题。但是由于 没有回答好，导致面试挂了。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WOCwSJngBvtO9S4RSOg0ZcCFHbJthm4eKcJFicDicz32DZcIrN5MicWddst8bLRcpRGqhJSgh2j8Pib4g/640?from=appmsg&watermark=1#imgIndex=0)

小伙伴面试完了之后，来求助尼恩。

那么，遇到 这个问题，该如何才能回答得很漂亮，才能 让面试官刮目相看、口水直流。

所以，尼恩给大家做一下系统化、体系化的梳理，使得大家内力猛增，可以充分展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer直提”。

当然，这道面试题，以及参考答案，也会收入咱们的 《[尼恩Java面试宝典](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》V175版本PDF集群，供后面的小伙伴参考，提升大家的 3高 架构、设计、开发水平。

> 最新《尼恩 架构笔记》《尼恩高并发三部曲》《尼恩Java面试宝典》的PDF，请关注本公众号【技术自由圈】获取，后台回复：领电子书

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WOCwSJngBvtO9S4RSOg0ZcC9GR57JmdFT4XSTocLH2cHhEORVcr4vUpets78iaEYyPPckMbqiaHzWvA/640?from=appmsg&watermark=1#imgIndex=1)

## 一：Spring   Bean 生命周期 主要分 6个阶段

Spring 中生成 Bean 主要包括以下几个阶段：

### 1.1 Bean 生命周期阶段1: Bean 定义加载阶段

Spring 容器首先会加载 Bean 的定义信息。这些定义信息来自配置文件（如 XML 配置）或者注解配置（如使用 `@Component` 系列注解）。

例如，当使用 `@ComponentScan` 注解时，Spring 会扫描指定包及其子包下的类，查找带有 `@Component`、`@Service`、`@Repository`、`@Controller` 等注解的类，将这些类的相关信息（如类名、属性等）加载到容器中，形成 BeanDefinition 对象。

### 1.2 Bean 生命周期阶段2: Bean 实例化阶段

当需要创建 Bean 实例时，Spring 容器会根据 BeanDefinition 中的信息，通过反射的方式，调用类的构造方法（如 `Constructor.newInstance()`）或工厂方法创建 Bean 的原始对象，此时对象属性均为默认值（如 `null` 或 `0`）

例如，对于一个简单的带有 `@Service` 注解的类 `MyService`，Spring 会使用其无参构造方法创建一个 `MyService` 对象。

具体流程如下：

**第一步：实例化 Bean**：在 `AbstractAutowireCapableBeanFactory[#doCreateBean]` 方法中，首先通过构造函数创建 Bean 实例（尚未进行属性填充和初始化）。

**第二步：添加到三级缓存**：实例化后，Spring 会创建一个 `ObjectFactory`（通常是 `BeanWrapper` 的匿名实现），用于后续生成代理对象或直接返回原始对象，并将该 `ObjectFactory` 存入三级缓存 `singletonFactories` 中，键为 Bean 的名称。

这一步的代码逻辑大致如下：

```
 `// 实例化后添加到三级缓存  
 addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));`
```

### 1.3 Bean 生命周期阶段3:属性填充阶段（依赖注入阶段）

创建好 Bean 实例后，Spring 容器会根据 BeanDefinition 中的属性信息，为 Bean 的属性进行赋值。

**属性赋值** ‌：通过 Setter 方法或字段直接赋值（如 `@Value("${config.value}")`）

‌**依赖注入** ‌：通过 `@Autowired`、`@Resource` 等注入其他 Bean（如 `UserDAO` 注入到 `UserService`）

这是一个 的关键阶段。

例如，如果 `MyService` 类中有一个类型为 `MyDao` 的属性，并且使用 `@Autowired` 注解标注，Spring 会在属性填充阶段找到容器中对应的 `MyDao` Bean，将其注入到 `MyService` 对象中。

在这个阶段，Spring 会处理各种依赖关系，包括构造方法注入、字段注入和 setter 方法注入等多种方式。

**循环依赖在此阶段解决** ‌：若遇到循环引用（如 A 依赖 B，B 又依赖 A），Spring 通过‌**二级缓存、三级缓存** ‌提前暴露半成品 Bean 完成注入

### 1.4 Bean 生命周期阶段4: 初始化 阶段

在属性填充完成之后，Spring 容器 会进行 Bean 的初始化 。

初始化方法可以通过在 Bean 类中使用 `@PostConstruct` 注解的方法，或者在 XML 配置中通过 `init-method` 属性指定的方法。

在 Spring 中，Bean 属性填充完成后的定制化逻辑执行顺序是这样的：

**初始化 阶段的操作之1： Aware 接口回调**

包括 `BeanNameAware`（设置 Bean 名称）、`BeanFactoryAware`（注入 `BeanFactory`）等接口的实现。

当 Bean 实现这些接口时，Spring 会在属性填充完成后，调用相应的回调方法。

例如，`BeanNameAware` 的 `setBeanName()` 方法会在 Bean 的属性填充后被调用，Spring 会将该 Bean 的名称作为参数传递给这个方法。

这样，Bean 就可以获取到自己在 Spring 容器中的名称。

**初始化 阶段的操作之2：BeanPostProcessor 前置处理**

`postProcessBeforeInitialization`：这是 `BeanPostProcessor` 接口中的一个方法。

这个方法会在 Bean 的初始化方法（包括 `@PostConstruct` 注解方法、`InitializingBean` 接口的 `afterPropertiesSet()` 方法、`init-method` 指定的方法）调用之前执行。

**初始化 阶段的操作之3：Bean 初始化方法**

*   `@PostConstruct` 注解方法：如果 Bean 中有使用 `@PostConstruct` 注解的方法，Spring 会在 `postProcessBeforeInitialization` 方法执行完成后，调用这个注解的方法。这个方法主要用于执行 Bean 创建完成后的初始化工作，比如资源的加载、对象的初始化等。
    
*   `InitializingBean` 接口的 `afterPropertiesSet()` 方法：如果 Bean 实现了 `InitializingBean` 接口，Spring 会在 `@PostConstruct` 注解方法执行之后，调用 `afterPropertiesSet()` 方法。这个方法也是一个初始化方法，用于进行一些初始化操作。它和 `@PostConstruct` 注解方法的功能类似，但 `@PostConstruct` 是 Java 自带的注解，而 `InitializingBean` 是 Spring 提供的接口。
    
*   XML 或 `@Bean` 中指定的 `init-method`：如果通过 XML 配置或者 `@Bean` 注解的 `initMethod` 属性指定了初始化方法，这个方法会在 `InitializingBean` 接口的 `afterPropertiesSet()` 方法执行之后被调用。这个方法也是用于执行初始化操作，它提供了另一种配置初始化方法的方式。
    

例如：

```java
@Service  
public class MyService {  
 @PostConstruct  
 public void init() {  
 System.out.println("MyService 初始化方法被调用");  
 }  
}  
```

这个初始化方法可以用于执行一些 Bean 创建完成后的初始化操作，如资源的加载等。

#### 初始化 阶段的操作之4：BeanPostProcessor 后置处理

`postProcessAfterInitialization`：这是 `BeanPostProcessor` 接口中的另一个方法。

它会在 Bean 的初始化方法执行完成后被调用。

Spring 会遍历所有注册的 `BeanPostProcessor`，依次调用它们的 `postProcessAfterInitialization` 方法。这个方法可以用于对已经完成初始化的 Bean 进行进一步的处理或者检查，比如对某些特殊场景下的 Bean 进行验证或者修改。

这些后BeanPostProcessor 处理器可以用于实现 AOP（面向切面编程）等功能。

例如，AOP 代理的生成通常在这个阶段完成。Spring 会遍历所有注册的 `BeanPostProcessor`，依次调用它们的 `postProcessAfterInitialization` 方法，对 Bean 进行一些额外的处理，比如为需要进行 AOP 切面操作的 Bean 创建代理对象。

例如，在 AOP 中，`AspectJAutoProxyCreator` 这样的BeanPostProcessor 后处理器，会在合适的阶段为 Bean 创建 AOP 代理对象。

#### 初始化过程的优先级顺序：

*   ‌**Aware 接口优先于所有初始化逻辑** ‌：确保 Bean 先获取容器基础设施（如名称、工厂）
    
*   ‌**@PostConstruct > afterPropertiesSet() > init-method** ‌：注解优先于接口，接口优先于配置
    

这个顺序确保了 Bean 在创建过程中，从属性填充到初始化再到后续处理的各个阶段，都能按照合理的顺序执行相关的定制化逻辑。

### 1.5 Bean 生命周期阶段5 : 使用阶段

Bean 可以被应用程序使用

### 1.6 Bean 生命周期阶段6:销毁阶段

对于Singleton且定义了销毁方法的Bean，容器会将其注册起来，以便在容器关闭时执行销毁逻辑。

## 二：Spring Bean 的生命周期（微观11步）

Spring Bean的生命周期指的是一个Bean从被定义、创建、初始化到使用、到啊最终被销毁的完整过程， 尼恩给大家 精炼为以下**11个核心步骤**：

**(1) 实例化 (Instantiation)：**

```
Spring容器根据Bean的定义（如XML、注解、Java配置），通过反射（或`FactoryBean`）调用构造方法创建一个原始的Bean对象。此时对象内的属性都是默认值（如null, 0）。
```

**(2) 填充属性 (Populate Properties)：**

依赖注入，容器解析并注入Bean所依赖的其他Bean（`@Autowired`, `@Resource`等）和配置值。

**(3) BeanPostProcessor前置处理（BPP前置处理）：**

如果容器中存在实现了`BeanPostProcessor`接口的Bean，则会调用其`postProcessBeforeInitialization`方法。

**(4) 初始化方法回调 - @PostConstruct：**

```
如果Bean中定义了`@PostConstruct`注解的方法，容器会调用它。
```

**(5) 初始化方法回调 - InitializingBean：**

```
如果Bean实现了`InitializingBean`接口，容器会调用其`afterPropertiesSet()`方法。
```

**(6) 初始化方法回调 - 自定义init-method：**

如果Bean在定义时指定了自定义的初始化方法（如XML中的`init-method`或`@Bean(initMethod = "..."）`），容器会调用它。

**(7) BeanPostProcessor后置处理（BPP后置处理）：**

```
`BeanPostProcessor`的`postProcessAfterInitialization`方法被调用。
```

这是AOP动态代理创建的关键时机（`AbstractAutoProxyCreator`就是一个`BeanPostProcessor`）。

**(8) Bean就绪 (Ready)：**

此时，Bean已经完全初始化，属性被填充，所有初始化回调都已完成，并且已经被可能的代理包装。

Bean被存放在Spring容器中，可以被应用程序使用。

**(9) 使用期 (In Use)：**

应用程序从容器中获取Bean并使用它。

**(10) 销毁方法回调 - @PreDestroy：**

```
当容器关闭时，如果Bean中定义了`@PreDestroy`注解的方法，容器会调用它。
```

**(11) 销毁方法回调 - DisposableBean：**

```
 如果Bean实现了`DisposableBean`接口，容器会调用其`destroy()`方法。
```

**(12) 销毁方法回调 - 自定义destroy-method：**

```
 如果Bean在定义时指定了自定义的销毁方法，容器会调用它。
```

**记忆技巧**：

创建（实例化 -> 属性填充） -> 初始化（`BeanPostProcessor`前 -> `@PostConstruct`-> `afterPropertiesSet`-> `init-method`-> `BeanPostProcessor`后） -> 使用 -> 销毁。

## 三. BeanFactory 如何创建 Bean？ 主要是以下 4步

Spring主要通过`BeanFactory`（及其默认实现`DefaultListableBeanFactory`）来创建Bean。

这个过程是上述生命周期步骤的具体实现，其核心流程在`AbstractAutowireCapableBeanFactory`的`createBean()`和`doCreateBean()`方法中：

### BeanFactory 创建 Bean第一步：解析Bean定义：

这个过程就对应了生命周期中的**第1步**，

容器首先找到对应Bean的`BeanDefinition`，它包含了创建这个Bean所需的所有元信息（类名、作用域、属性值、构造函数参数等）。

### BeanFactory 创建 Bean第2步：实例化 (createBeanInstance)：

这个过程就对应了生命周期中的**第1步**，

*   如果Bean实现了`FactoryBean`接口，则会调用其`getObject()`方法来获取Bean实例。
    
*   否则，容器会根据`BeanDefinition`中的信息（如是否有指定的构造函数、是否存在构造器注入）来决定使用何种策略（无参构造、有参构造、工厂方法）通过反射来创建实例。
    

### BeanFactory 创建 Bean第3步：依赖注入 (populateBean)：

这个过程就对应了生命周期中的**第2步**，

容器检查Bean的依赖关系（通过注解或XML配置），然后从容器中查找或创建所依赖的Bean，并通过反射（或setter方法）将它们注入到目标Bean中。

### BeanFactory 创建 Bean第4步：初始化 (initializeBean)：

这个过程就对应了生命周期中的**第3到第7步**，

依次执行各种`BeanPostProcessor`和初始化回调方法。

## 四：说说 BeanFactoryPostProcessor （BFPP）和 BeanPostProcessor （BPP）的区别

BeanFactoryPostProcessor 和 BeanPostProcessor 是 Spring 中两个重要的扩展接口，它们在 Bean 的生命周期中发挥不同作用，核心区别如下：

### 第一个不同： 作用时机不同

**BFPP**：

在 **Bean 定义加载完成后，Bean 实例化之前** 执行。

此时 Spring 容器已经解析完所有 XML 或注解配置，生成了 BeanDefinition（Bean 的元数据定义），但还未创建任何 Bean 实例。

**BPP**：

在 **Bean 实例化之后，初始化前后** 执行。

此时 Bean 已经通过构造方法或工厂方法创建出实例，正在进行属性赋值和初始化操作。

### 第二个不同： 处理对象不同

**BFPP处理的是 BeanDefinition（Bean 的定义信息）**：

比如修改 Bean 的属性值、作用域等元数据。

例如：可以动态修改 `@Value` 注解的占位符，或给 Bean 添加额外的属性配置。

**BPP处理的是 Bean 实例本身**

比如对实例进行增强、代理包装、添加额外功能等。例如：AOP 就是通过 BPP在初始化后为 Bean 创建代理对象。

### 第三个不同：核心方法与功能

**BFPP**：

核心方法是 `postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory)`，通过该方法可以获取并修改所有 BeanDefinition。

典型应用：`PropertyPlaceholderConfigurer`（处理配置文件中的占位符 `${}`）。

```java
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {  
 @Override  
 public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {  
 // 修改Bean的定义信息，例如修改属性值  
 BeanDefinition beanDef = beanFactory.getBeanDefinition("userService");  
 beanDef.getPropertyValues().add("username", "modifiedName");  
 }  
}  
```

**BPP核心方法是两个**：

*   方法一：`postProcessBeforeInitialization(Object bean, String beanName)`：在 Bean 初始化方法（如 `@PostConstruct`、`init-method`）执行前调用。
    
*   方法二：`postProcessAfterInitialization(Object bean, String beanName)`：在 Bean 初始化方法执行后调用。
    

典型应用：

AOP 的 `AnnotationAwareAspectJAutoProxyCreator`（为 Bean 创建代理）。

```java
public class MyBeanPostProcessor implements BeanPostProcessor {  
 @Override  
 public Object postProcessBeforeInitialization(Object bean, String beanName) {  
 // 初始化前增强（如添加日志）  
 if (bean instanceof UserService) {  
 System.out.println("UserService初始化前处理");  
 }  
 return bean;  
 }  
 @Override  
 public Object postProcessAfterInitialization(Object bean, String beanName) {  
 // 初始化后增强（如创建代理）  
 return bean;  
 }  
}  
```

### 第四个不同：执行范围不同

**BFPP**：

作用于整个 BeanFactory（容器），会扫描所有 BeanDefinition 并处理。

**BPP**：

作用于容器中所有实例化后的 Bean（可通过条件判断过滤特定 Bean）。

### BFPP 和 BPP的区别 总结

<table style="line-height: 1.75em;padding: 6px 13px;border-color: rgb(198, 203, 209) rgb(223, 226, 229) rgb(223, 226, 229);background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;"><thead><tr><td style="line-height: 1.75em;padding: 6px 13px;border-color: rgb(198, 203, 209) rgb(223, 226, 229) rgb(223, 226, 229);background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;"><span style="font-size: 15px;"><strong><span leaf="">维度</span></strong></span></td><td style="line-height: 1.75em;padding: 6px 13px;border-color: rgb(198, 203, 209) rgb(223, 226, 229) rgb(223, 226, 229);background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;"><span style="font-size: 15px;"><strong><span leaf="">BFPP</span></strong></span></td><td style="line-height: 1.75em;padding: 6px 13px;border-color: rgb(198, 203, 209) rgb(223, 226, 229) rgb(223, 226, 229);background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;"><span style="font-size: 15px;"><strong><span leaf="">BeanPostProcessor</span></strong></span></td></tr></thead><tbody><tr style="background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">作用阶段</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">BeanDefinition 加载后，实例化前</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">Bean 实例化后，</span><strong><span leaf="">初始化前后</span></strong><span leaf="">、</span><strong><span leaf="">初始化前+后</span></strong></span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">处理对象</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">BeanDefinition（元数据）</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">Bean 实例本身</span></span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">核心功能</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">修改 Bean 定义（如属性、作用域）</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">增强 Bean 实例（如代理、功能扩展）</span></span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">典型应用</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">处理配置占位符、动态修改配置</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">AOP 代理、依赖注入后的额外处理</span></span></section></td></tr></tbody></table>

简单说：BFPP 是 “修改图纸”，BPP是 “加工产品”。

## 说在最后：有问题找老架构取经‍

只要按照上面的 尼恩团队梳理的 方案去作答， 你的答案不是 100分，而是 120分。  面试官一定是 心满意足， 五体投地。

按照尼恩的梳理，进行 深度回答，可以充分展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer直提”。

在面试之前，建议大家系统化的刷一波 5000页《[尼恩Java面试宝典PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》，里边有大量的大厂真题、面试难题、架构难题。

很多小伙伴刷完后， 吊打面试官， 大厂横着走。

在刷题过程中，如果有啥问题，大家可以来 找 40岁老架构师尼恩交流。

另外，如果没有面试机会， 可以找尼恩来改简历、做帮扶。前段时间，**[指导一个小伙 暴涨200%（涨2倍），29岁/7年/双非一本 ， 从13K一次涨到 37K ，逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486433&idx=1&sn=0f630e8fc7620b3bf0e054827b48d91e&scene=21#wechat_redirect)。**

狠狠卷，实现 “offer自由” 很容易的， 一个武汉的跟着尼恩卷了2年的小伙伴， 在极度严寒/痛苦被裁的环境下， offer拿到手软， 实现真正的 “offer自由” 。

## Java+AI  弯道超车，  转架构 捷径 

 [会 AI的程序员，工资暴涨50%！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486827&idx=1&sn=db3f268c6752ddb074a98c7b13a84011&scene=21#wechat_redirect)

  

[涨薪传奇： 18k->38K , 单月暴20K，32岁小伙伴 2个月时间年薪 翻1.5倍 ，一步登天+逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486960&idx=1&sn=f57253a448694c32e834207381c42284&scene=21#wechat_redirect)

  

[低学历 传奇：29岁6年专套本，受够了外包，狠卷3个月逆袭大厂 涨 1倍， 逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486945&idx=1&sn=f6ff6231ccf3585624161c2ef1f50fdf&scene=21#wechat_redirect)

  

[极速上岸： 被裁 后， 8天 拿下 京东，狠涨 一倍 年薪48W， 小伙伴 就是 做对了一件事](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486913&idx=1&sn=edd6774e9d17c39327e8dcb95fdd68d9&scene=21#wechat_redirect)

  

[外包+二本 进 美团： 26岁小2本 一步登天， 进了顶奢大厂（ 美团） ， 太爽了](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486904&idx=1&sn=e7c878f99ed101ebfbd1d80ec0e07401&scene=21#wechat_redirect)

  

[超牛的Java+Al 双栖架构： 34岁无路可走，一个月翻盘，拿 3个架构offer，靠  Java+Al   逆天改命！！！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486848&idx=1&sn=ff43d058271b0801f84aba3f3d957855&scene=21#wechat_redirect)

java+AI 逆袭2：[：3年 程序媛 被裁， 25W-》40W 上岸， 逆涨60%。 Java+AI 太神了， 架构小白 2个月逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486858&idx=1&sn=9cd122c18d166433b43d0952d3a7a7a8&scene=21#wechat_redirect)

  

  

[Java+AI逆袭3 ： 36岁/失业7个月/彻底绝望 。狠卷 3个月 Java+AI ，终于逆风翻盘，顺利 上岸](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486870&idx=1&sn=990a540a155cdb8254ebce6c75816882&scene=21#wechat_redirect)

[Java+AI逆袭 ： 闲了一年，41岁/失业12个月/彻底绝望 。狠卷 2个月 Java+AI ，终于逆风翻盘](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486880&idx=1&sn=a37033157c766ac5ae7c9195fe776693&scene=21#wechat_redirect)

  

[Java+AI逆袭5：1个月大涨2.5W，37岁 脱坑外包， 入了正编，GO+AI 要逆天了](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486885&idx=1&sn=4e26fbb093f45d437dedf14ea9b9e6c5&scene=21#wechat_redirect)

  

**职业救助站**

实现职业转型，极速上岸

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/9qY2Gt3PBs3icE6WcM4nBOdRRcczOW4wkNDgCMZhBfqeet9suwGHfgQduP33YYqqZLOppecMI72Pv5ibNkPeIk6g/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=2)

关注**职业救助站**公众号，获取每天职业干货  
助您实现**职业转型、职业升级、极速上岸**  
---------------------------------

**技术自由圈**

实现架构转型，再无中年危机

  

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/xlgvgPaib7WMtyFcJx45oIeWL1d6AENcdoAvuTPuq46E9n8pPe4SHXoSicuEVvhibHRSIzmEBPlC7POcDaqD6ID7w/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=3)

关注**技术自由圈**公众号，获取每天技术千货  
一起成为牛逼的**未来超级架构师**

**几十篇架构笔记、5000页面试宝典、20个技术圣经  
请加尼恩个人微信 免费拿走**

**暗号，请在 公众号后台 发送消息：领电子书**

如有收获，请点击底部的"在看"和"赞"，谢谢