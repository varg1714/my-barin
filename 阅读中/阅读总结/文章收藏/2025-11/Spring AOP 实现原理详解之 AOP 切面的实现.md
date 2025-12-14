---
source:
  - "[[Spring进阶 - Spring AOP实现原理详解之AOP切面的实现]]"
create: 2025-11-26
---

## 1. Spring AOP 源码深度解析 (一)：启动流程与核心组件注册

### 1.1. 全局视角：AOP 介入的入口

Spring AOP 的核心原理是利用 Spring IOC 容器的扩展点 —— **`BeanPostProcessor`**（后置处理器）。

要让 AOP 生效，Spring 必须在容器启动初期，向容器内注册一个特殊的 `BeanPostProcessor`。这个处理器就像一个“海关安检员”，所有后续创建的 Bean 都要经过它的检查。如果发现某个 Bean 匹配了切面规则，就偷梁换柱，将其替换为代理对象。

这个“安检员”就是 **`AnnotationAwareAspectJAutoProxyCreator`**。

本章将详细剖析它是如何通过配置被解析并注册到容器中的。

### 1.2. 配置解析的幕后：从 XML 到 BeanDefinition

当我们配置 `<aop:aspectj-autoproxy/>` 时，Spring 的解析工作流如下：

#### 1.2.1. 命名空间处理器 `AopNamespaceHandler`

Spring 解析 XML 时，根据命名空间 URI 找到 `META-INF/spring.handlers` 中定义的 `AopNamespaceHandler`。

```java
public class AopNamespaceHandler extends NamespaceHandlerSupport {
    @Override
    public void init() {
        // 1. 处理 <aop:config> (基于 XML 的 AOP)
        registerBeanDefinitionParser("config", new ConfigBeanDefinitionParser());
        
        // 2. 处理 <aop:aspectj-autoproxy> (基于注解的 AOP) -> 重点关注
        registerBeanDefinitionParser("aspectj-autoproxy", new AspectJAutoProxyBeanDefinitionParser());
        
        // ... 其他标签处理
    }
}
```

#### 1.2.2. 解析器 `AspectJAutoProxyBeanDefinitionParser`

`AspectJAutoProxyBeanDefinitionParser` 实现了 `BeanDefinitionParser` 接口。它的 `parse` 方法是注册流程的起点。

```java
@Override
@Nullable
public BeanDefinition parse(Element element, ParserContext parserContext) {
    // 步骤 A: 注册 AnnotationAwareAspectJAutoProxyCreator
    AopNamespaceUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(parserContext, element);
    
    // 步骤 B: 处理子标签 (如 <aop:include>)，用于控制哪些 Bean 需要被代理
    extendBeanDefinition(element, parserContext);
    
    return null;
}
```

这里看似简单，实则暗藏玄机。核心逻辑都在 `AopNamespaceUtils` 工具类中。

### 1.3. 深度调用链：注册与优先级升级机制

Spring 的设计非常严谨，它考虑到了用户可能重复配置 AOP，或者同时配置了多种类型的 AOP 基础设施。因此，注册过程并不是简单的 `new`，而是一个 **"注册或升级" (Register or Escalate)** 的过程。

#### 1.3.1. 调用时序图

![调用时序图.svg](https://r2.129870.xyz/img/2025/681baa1c44151b980dc68c8c00b5e9e9.svg)

#### 1.3.2. 源码核心：`AopConfigUtils`

让我们深入 `AopConfigUtils.registerOrEscalateApcAsRequired` 方法，这是源码中容易被忽略但极具设计感的地方：

```java
@Nullable
private static BeanDefinition registerOrEscalateApcAsRequired(
        Class<?> cls, BeanDefinitionRegistry registry, @Nullable Object source) {

    // 1. 检查容器中是否已经存在了自动代理创建器 (AutoProxyCreator)
    // Spring AOP 所有的 Creator 都共用同一个 Bean Name: "org.springframework.aop.config.internalAutoProxyCreator"
    if (registry.containsBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME)) {
        
        BeanDefinition apcDefinition = registry.getBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME);
        
        // 2. 优先级比较 (Escalation)
        // 如果容器里已经存在的 Creator 类名与当前要注册的不同
        if (!cls.getName().equals(apcDefinition.getBeanClassName())) {
            int currentPriority = findPriorityForClass(apcDefinition.getBeanClassName());
            int requiredPriority = findPriorityForClass(cls);
            
            // 3. 如果当前要注册的类优先级更高，则替换掉旧的
            if (currentPriority < requiredPriority) {
                apcDefinition.setBeanClassName(cls.getName());
            }
        }
        // 如果已经存在且优先级够高，则什么都不做，直接返回
        return null;
    }

    // 4. 如果容器中不存在，则创建新的 BeanDefinition 并注册
    RootBeanDefinition beanDefinition = new RootBeanDefinition(cls);
    beanDefinition.setSource(source);
    beanDefinition.getPropertyValues().add("order", Ordered.HIGHEST_PRECEDENCE); // 设置最高优先级
    beanDefinition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE); // 标记为基础设施 Bean
    registry.registerBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME, beanDefinition);
    return beanDefinition;
}
```

**为什么需要优先级比较？**
Spring 中有多种 AutoProxyCreator，例如：
1. `InfrastructureAdvisorAutoProxyCreator` (优先级最低)
2. `AspectJAwareAdvisorAutoProxyCreator`
3. `AnnotationAwareAspectJAutoProxyCreator` (优先级最高，支持 @Aspect 注解)

如果你的配置中既开启了事务（可能注册低优先级的 Creator），又开启了 `@Aspect` AOP。Spring 必须保证最终留在容器里的是功能最全、优先级最高的那个（即 `AnnotationAware...`），否则 `@Aspect` 注解将无法被解析。

### 1.4. 核心类剖析：AnnotationAwareAspectJAutoProxyCreator

注册成功的这个类，到底长什么样？它的继承体系决定了它拥有哪些能力。

#### 1.4.1. 类继承体系图解

这个类处于继承树的底端，集成了所有祖先的能力。

![](https://pdai.tech/images/spring/springframework/spring-springframework-aop-3.png)

#### 1.4.2. 关键层级职责解析

1. **`AbstractAutoProxyCreator`**:
    *  **职责**：这是 AOP 的骨架。它实现了 `BeanPostProcessor`，规定了在 Bean 初始化后 (`postProcessAfterInitialization`) 进行代理创建的逻辑。它不关心切面从哪来，只关心“如果有切面，我就创建代理”。
    
2. **`AbstractAdvisorAutoProxyCreator`**:
    *  **职责**：它解决了“切面从哪来”的问题。它会扫描容器中所有实现了 `Advisor` 接口的 Bean。

3. **`AnnotationAwareAspectJAutoProxyCreator` (主角)**:
    *  **职责**：它进一步增强了查找能力。它不仅能找到普通的 `Advisor` Bean，还能扫描带有 **`@Aspect`** 注解的类，并利用 `AspectJAdvisorFactory` 将这些注解解析成 Spring 能看懂的 `Advisor`。

### 1.5. 第一部分总结

至此，AOP 的准备工作已经完成。

1. **配置解析**：Spring 识别了 `<aop:aspectj-autoproxy/>`。
2. **组件注册**：通过 `AopConfigUtils` 的优先级机制，确保容器中注册了功能最强大的 **`AnnotationAwareAspectJAutoProxyCreator`**。
3. **能力就绪**：这个 Creator 作为一个 `BeanPostProcessor`，已经蓄势待发。它继承了祖先的能力，既能拦截 Bean 的创建，又能解析 `@Aspect` 注解。

**接下来的问题是：**
当 Spring 容器开始实例化一个普通的业务 Bean（如 `UserService`）时，这个 Creator 是如何在 Bean **实例化之前** 就开始工作的？它是如何从成千上万个 Bean 中精准地找出那些 `@Aspect` 切面类的？

## 2. Spring AOP 源码深度解析 (二)：核心拦截机制 —— 寻找与筛选切面

### 2.1. 拦截的时机：Bean 实例化之前

在 Spring IOC 容器中，Bean 的创建是一个漫长的过程。`AnnotationAwareAspectJAutoProxyCreator` 实现了 `InstantiationAwareBeanPostProcessor` 接口，这给了它一个极早的介入机会：**`postProcessBeforeInstantiation`**。

这个方法在 Bean **实例化（Instantiation）之前** 就会被调用。也就是说，在 `new UserService()` 还没执行时，AOP 就已经开始工作了。

#### 2.1.1. 为什么要在这么早的阶段介入？

代理不是在 Bean 初始化之后才创建吗？为什么要在实例化之前就拦截？

主要有两个原因：

1. **处理自定义 TargetSource**：这是一个高级特性（通常用得少），允许用户自己管理目标对象的生命周期（如对象池）。
2. **提前扫描切面（关键）**：Spring 需要在创建代理之前，先知道容器里到底有哪些切面规则。虽然扫描动作可以延迟，但尽早识别出哪些是“切面基础设施类”，可以避免后续不必要的处理。

### 2.2. 核心方法剖析：postProcessBeforeInstantiation

让我们走进 `AnnotationAwareAspectJAutoProxyCreator` 的父类 `AbstractAutoProxyCreator` 中的这个核心方法。

```java
@Override
public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) {
    Object cacheKey = getCacheKey(beanClass, beanName);

    // 1. 检查缓存：如果这个 Bean 已经被处理过（确认需要/不需要代理），直接跳过
    if (!StringUtils.hasLength(beanName) || !this.targetSourcedBeans.contains(beanName)) {
        if (this.advisedBeans.containsKey(cacheKey)) {
            return null;
        }
        
        // 2. 核心判断：这个类是 AOP 基础设施类吗？或者应该跳过吗？
        // 这里的 shouldSkip 方法暗藏玄机，它触发了切面的扫描！
        if (isInfrastructureClass(beanClass) || shouldSkip(beanClass, beanName)) {
            // 标记为不需要代理 (FALSE)
            this.advisedBeans.put(cacheKey, Boolean.FALSE);
            return null;
        }
    }

    // ... 处理自定义 TargetSource 逻辑 (略)

    return null;
}
```

#### 2.2.1. 决策流程图

![决策流程.svg](https://r2.129870.xyz/img/2025/60cb237b261cade82b067bfa97e7cbea.svg)

### 2.3. 深度解析：shouldSkip 与切面扫描

`shouldSkip` 方法的名字很有迷惑性。它表面上是在问“这个 Bean 是否应该跳过自动代理？”，但实际上，为了回答这个问题，它必须**先找出容器中所有的 Advisor**。

对于 `AnnotationAwareAspectJAutoProxyCreator` 来说，`shouldSkip` 的逻辑如下：

```java
@Override
protected boolean shouldSkip(Class<?> beanClass, String beanName) {
    // 1. 寻找所有的候选 Advisor (核心动作！)
    // 这里会扫描所有的 @Aspect 注解类，并解析成 Advisor
    List<Advisor> candidateAdvisors = findCandidateAdvisors();
    
    // 2. 遍历找到的 Advisor
    for (Advisor advisor : candidateAdvisors) {
        // 如果当前 Bean 本身就是一个 AspectJ 切面类，那它当然应该被“跳过”
        // (切面自己不能被代理，否则会死循环)
        if (advisor instanceof AspectJPointcutAdvisor &&
                ((AspectJPointcutAdvisor) advisor).getAspectName().equals(beanName)) {
            return true;
        }
    }
    
    // 调用父类逻辑 (通常返回 false)
    return super.shouldSkip(beanClass, beanName);
}
```

#### 2.3.1. 触发扫描：findCandidateAdvisors

这是 AOP 准备阶段最“重”的一步。它负责把容器翻个底朝天，找出所有的切面。

```java
@Override
protected List<Advisor> findCandidateAdvisors() {
    // 1. 找 XML 配置的 Advisor (调用父类方法)
    List<Advisor> advisors = super.findCandidateAdvisors();
    
    // 2. 找 @Aspect 注解的切面 (核心扩展)
    if (this.aspectJAdvisorsBuilder != null) {
        advisors.addAll(this.aspectJAdvisorsBuilder.buildAspectJAdvisors());
    }
    return advisors;
}
```

#### 2.3.2. 扫描器：BeanFactoryAspectJAdvisorsBuilder

`buildAspectJAdvisors` 方法使用了 **双重检查锁 (DCL)** 模式来实现单例缓存。这意味着，虽然 `shouldSkip` 会被每个 Bean 调用，但**全量扫描只会在第一个 Bean 经过时发生一次**。

**扫描逻辑详解：**

1. **获取所有 Bean Name**：调用 `BeanFactory.getBeanNamesForType(Object.class)`，拿到容器里所有 Bean 的名字。
2. **遍历与检查**：
    *  遍历每个 Bean Name。
    *  检查 Bean 的 Class 上是否有 **`@Aspect`** 注解。
    *  如果没有，跳过。
3. **解析与缓存**：
    *  如果找到了 `@Aspect` 类，使用 `AdvisorFactory` 将其内部的 `@Before`、`@After` 等方法解析成一个个 `Advisor` 对象。
    *  将解析结果存入 `advisorsCache` 缓存。
4. **后续调用**：直接从 `advisorsCache` 返回结果，速度极快。

### 2.4. 关键细节：如何判断“基础类” (isInfrastructureClass)

为了防止 AOP 系统乱套，Spring 必须保证 AOP 自己的基础设施类不会被代理。

`AnnotationAwareAspectJAutoProxyCreator` 重写了 `isInfrastructureClass` 方法：

```java
@Override
protected boolean isInfrastructureClass(Class<?> beanClass) {
    // 1. 调用父类判断：是否实现了 Advice, Pointcut, Advisor, AopInfrastructureBean 接口
    boolean retVal = super.isInfrastructureClass(beanClass);
    
    // 2. 扩展判断：是否带有 @Aspect 注解
    // 如果一个类有 @Aspect 注解，它就是切面定义本身，属于基础设施，不应该被代理
    if (this.aspectJAdvisorFactory != null && this.aspectJAdvisorFactory.isAspect(beanClass)) {
        retVal = true;
    }
    
    return retVal;
}
```

**结论**：只要你给一个类加上了 `@Aspect`，或者它实现了 `Advice` 接口，Spring 就会给它发一张“免死金牌”（`advisedBeans.put(cacheKey, Boolean.FALSE)`），它永远不会被代理。

### 2.5. 第二部分总结

在这一阶段，Spring AOP 完成了至关重要的**信息收集**工作：

1. **拦截**：通过 `postProcessBeforeInstantiation` 拦截了每个 Bean 的创建。
2. **过滤**：利用 `isInfrastructureClass` 识别并放行了所有的 AOP 基础设施类（包括 `@Aspect` 类）。
3. **扫描**：在处理第一个 Bean 时，通过 `shouldSkip` -> `findCandidateAdvisors` 触发了全量扫描。
4. **缓存**：所有的 `@Aspect` 切面都被解析成了 `Advisor` 对象列表，并被缓存在内存中，等待后续使用。

**接下来的问题是：**
扫描到的 `@Aspect` 类中，那些 `@Before("execution(...)")` 注解具体是如何变成 Spring 内部的 `Advisor` 对象的？当一个普通的业务 Bean 初始化完成后，Spring 又是如何利用这些缓存的 `Advisor` 来创建代理的？

## 3. Spring AOP 源码深度解析 (三)：从注解到对象 —— Advisor 的构建细节与总结

这一部分将深入最微观的层面：**Spring 是如何把一个写着 `@Before` 的 Java 方法，变成一个能够拦截方法调用的 `Advisor` 对象的？** 同时，我们将串联起最后一步——代理的创建，完成整个 AOP 准备阶段的闭环。

### 3.1. 微观视角：Advisor 的诞生

在第二部分提到的 `buildAspectJAdvisors` 方法中，当 Spring 发现一个 `@Aspect` 类后，会调用 `advisorFactory.getAdvisors(factory)` 来进行解析。

这个过程就像是“翻译”：把 AspectJ 的注解语言，翻译成 Spring AOP 的对象语言。

#### 3.1.1. 核心工厂：ReflectiveAspectJAdvisorFactory

这个工厂类负责所有的解析工作。它的 `getAdvisors` 方法逻辑如下：

1. **获取所有方法**：反射获取切面类中所有不包含 `@Pointcut` 注解的方法。
2. **遍历解析**：对每个方法调用 `getAdvisor`。
3. **排序**：对生成的 Advisor 进行排序（根据 `@Order` 或方法声明顺序）。

#### 3.1.2. 单个 Advisor 的构建流程

`getAdvisor` 方法是构建的最小单元。

```java
public Advisor getAdvisor(Method candidateAdviceMethod,MetadataAwareAspectInstanceFactory aspectInstanceFactory, int declarationOrderInAspect, String aspectName) {

    // 1. 验证切面类 (略)

    // 2. 获取切点表达式 (Pointcut)
    // 解析方法上的注解 (@Before, @After...)，提取表达式信息
    AspectJExpressionPointcut expressionPointcut = getPointcut(
            candidateAdviceMethod, aspectInstanceFactory.getAspectMetadata().getAspectClass());
    
    if (expressionPointcut == null) {
        return null;
    }

    // 3. 封装成 Advisor 对象
    // 将 切点(Pointcut) + 通知(Advice) + 其他元数据 包装在一起
    return new InstantiationModelAwarePointcutAdvisorImpl(
            expressionPointcut, candidateAdviceMethod,
            this, aspectInstanceFactory, declarationOrderInAspect, aspectName);
}
```

#### 3.1.3. 关键对象映射图

Spring 将不同的注解映射为不同的 `Advice` 实现类。

![advisor.svg](https://r2.129870.xyz/img/2025/bcb07c4601c593aed18d1c7a702c9de3.svg)

### 3.2. 实例化 Advice：将方法变成拦截器

在 `InstantiationModelAwarePointcutAdvisorImpl` 的构造函数中，会进一步调用 `instantiateAdvice`。这一步是将切面类中的那个 `public void beforeMethod(...)` 方法，包装成一个标准的 AOP `Advice` 对象。

```java
public Advice getAdvice(...) {
    // ... 前置检查
    
    AbstractAspectJAdvice springAdvice;

    // 根据注解类型，创建对应的 Advice 对象
    switch (aspectJAnnotation.getAnnotationType()) {
        case AtBefore:
            springAdvice = new AspectJMethodBeforeAdvice(...);
            break;
        case AtAfter:
            springAdvice = new AspectJAfterAdvice(...);
            break;
        case AtAround:
            springAdvice = new AspectJAroundAdvice(...);
            break;
        // ... 其他类型 (AfterReturning, AfterThrowing)
    }

    // 配置 Advice (设置参数绑定、切面名称等)
    springAdvice.setAspectName(aspectName);
    // ...
    
    return springAdvice;
}
```

**至此，所有的准备工作全部完成。** 容器中所有的切面规则，都已经变成了内存中整整齐齐的 `Advisor` 对象列表。

### 3.3. 闭环：代理的创建 (postProcessAfterInitialization)

让我们回到 Bean 的生命周期。

当一个普通的业务 Bean（比如 `UserService`）完成了实例化（`new`）、属性填充（`populateBean`）和初始化（`initializeBean`）之后，它会来到生命周期的最后一步：**`postProcessAfterInitialization`**。

这也是 `AnnotationAwareAspectJAutoProxyCreator` 发挥作用的第二个战场。

```java
@Override
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
    if (bean != null) {
        Object cacheKey = getCacheKey(bean.getClass(), beanName);
        // 如果之前没有创建过代理（避免循环依赖场景下的重复代理）
        if (this.earlyProxyReferences.remove(cacheKey) != bean) {
            // 尝试进行包装 (Wrap)
            return wrapIfNecessary(bean, beanName, cacheKey);
        }
    }
    return bean;
}
```

#### 3.3.1. wrapIfNecessary 的逻辑

1. **获取 Advisor**：从之前缓存的 Advisor 列表中，找出所有**适用于当前 Bean** 的 Advisor（通过 Pointcut 匹配）。
2. **如果没有匹配**：直接返回原始 Bean。
3. **如果匹配成功**：
    *  创建一个代理工厂 (`ProxyFactory`)。
    *  将匹配的 Advisor 加入工厂。
    *  根据配置（是否强制 CGLib、是否实现了接口）决定使用 JDK 动态代理还是 CGLib。
    *  **返回代理对象**。

### 3.4. 全文总结：Spring AOP 实现原理全景图

回顾这三部分的内容，我们可以将 Spring AOP 的切面实现过程概括为以下一张完整的流程图：

![aop扫描整体流程.svg](https://r2.129870.xyz/img/2025/b0884f97f3ee4dcde7cf97c7288ca142.svg)

#### 3.4.1. 核心知识点回顾

1. **入口**：`AopNamespaceHandler` 和 `AspectJAutoProxyBeanDefinitionParser` 负责解析配置并注册核心组件。
2. **核心组件**：`AnnotationAwareAspectJAutoProxyCreator` 是 AOP 的心脏，它是一个 `BeanPostProcessor`。
3. **扫描时机**：在第一个 Bean 实例化前的 `postProcessBeforeInstantiation` 阶段，触发全量扫描。
4. **对象转换**：`ReflectiveAspectJAdvisorFactory` 负责将 `@Aspect`、`@Before` 等注解转化为 Spring 内部的 `Advisor` 和 `Advice` 对象。
5. **代理创建**：在 Bean 初始化后的 `postProcessAfterInitialization` 阶段，根据匹配结果决定是否创建代理。

通过这套机制，Spring 巧妙地将 AOP 的复杂逻辑隐藏在了 IOC 容器的生命周期之中，实现了对业务代码的无侵入增强。