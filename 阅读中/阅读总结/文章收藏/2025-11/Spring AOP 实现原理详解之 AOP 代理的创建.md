---
source:
  - "[[Spring进阶 - Spring AOP实现原理详解之AOP代理的创建]]"
create: 2025-11-28
---

## 1. Spring AOP 核心原理：代理对象的创建过程

### 1.1. 概述与背景

**核心问题**：Spring 是如何在 Bean 的生命周期中“偷梁换柱”，将原始 Bean 替换为具有增强功能的代理对象（Proxy）的？

**前置知识**：
* **Advisor**：切面（Aspect）的运行时封装，包含切点（Pointcut）和通知（Advice）。
* **BeanPostProcessor**：AOP 的核心介入点是 `AbstractAutoProxyCreator`，它是一个 BPP。

### 1.2. AOP 的介入时机

AOP 代理的创建主要发生在 Bean 初始化之后（`postProcessAfterInitialization`），但在处理循环依赖时会提前介入。

#### 1.2.1. 生命周期全景图

![Mermaid Chart - Create complex, visual diagrams with text.-2025-11-28-130926.svg](https://r2.129870.xyz/img/2025/bf52b39c581cfef6404cc13d142b9ebb.svg)

#### 1.2.2. 核心入口：`postProcessAfterInitialization`

源码位置：`AbstractAutoProxyCreator`

```java
@Override
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
    if (bean != null) {
        Object cacheKey = getCacheKey(bean.getClass(), beanName);
        // 【难点解析】循环依赖的防重判断
        // earlyProxyReferences 存储了因循环依赖而被"提前代理"的 Bean
        if (this.earlyProxyReferences.remove(cacheKey) != bean) {
            return wrapIfNecessary(bean, beanName, cacheKey);
        }
    }
    return bean;
}
```

> [!TIP] `earlyProxyReferences` 的作用
> 当 A 依赖 B，B 依赖 A 时：
> 1. A 实例化 -> 暴露早期引用（此时若 A 需要代理，会调用 `getEarlyBeanReference` 创建代理并存入 `earlyProxyReferences`）。
> 2. A 注入 B -> B 注入 A（拿到的是 A 的代理）。
> 3. A 初始化完成 -> 进入 `postProcessAfterInitialization`。
> 4. **判断**：此时检查 `earlyProxyReferences`，发现 A 已经被代理过了，因此直接返回原始 Bean（容器中最终使用的是步骤 1 创建的代理），**避免重复创建代理**。
> 
> `this.earlyProxyReferences.remove(cacheKey) != bean` 这行代码包含了两层含义：
> 
> 1. **判空逻辑**：如果 Map 里没东西（返回 null），`null != bean` 成立，去创建代理。（对应普通 Bean）
> 2. **身份一致性校验**：如果 Map 里有东西，必须确保**提前暴露的那个原始对象**，和**现在传进来的这个对象**是**同一个内存地址**。如果不是同一个，说明中间发生了变故（被其他 BPP 替换了），必须重新处理。

### 1.3. 代理决策：`wrapIfNecessary`

此方法决定一个 Bean 是否有资格被代理。

#### 1.3.1. 决策流程

1. **排除自定义 TargetSource**：避免重复处理。
2. **检查缓存 (`advisedBeans`)**：如果之前判断过不需要代理，直接跳过。
3. **基础设施类检查**：
    * 如果 Bean 本身是 AOP 的基础类（如 `Advice`, `Pointcut`, `Advisor`），绝对不能代理，否则会死循环。
    * `shouldSkip`：通常用于跳过 `@Aspect` 定义的切面类。
4. **获取 Advisor (核心)**：调用 `getAdvicesAndAdvisorsForBean`。
5. **创建代理**：如果找到了匹配的 Advisor，则创建代理对象。

### 1.4. 筛选 Advisor (The Funnel)

Spring 容器中可能有成百上千个 Advisor，如何找到专门针对当前 Bean 的那几个？

#### 1.4.1. 筛选漏斗模型

![Mermaid Chart - Create complex, visual diagrams with text.-2025-11-28-130629.svg](https://r2.129870.xyz/img/2025/f5b54ed17e7c97e9d93f6b7f03c881be.svg)

#### 1.4.2. 核心匹配逻辑：`findAdvisorsThatCanApply`

这是 AOP 性能开销最大的步骤，依赖于 Pointcut 匹配。

* **初筛 (ClassFilter)**：检查切点表达式是否匹配当前**类**。如果不匹配，直接丢弃（性能优化）。
* **细筛 (MethodMatcher)**：如果类匹配，遍历该类所有**方法**。只要有**任意一个方法**匹配，该 Advisor 就被保留。
* **引介增强 (Introduction)**：只匹配类，不匹配方法。

### 1.5. 代理工厂与策略选择

当确定需要代理后，Spring 使用 `ProxyFactory` 来生产代理对象。

#### 1.5.1. 代理创建流程 (`createProxy`)

1. **初始化 `ProxyFactory`**。
2. **填充属性**：将筛选出的 Advisor 加入工厂，设置 TargetSource。
3. **冻结配置 (`frozen`)**：优化项，表示代理配置不再变更，允许进行激进的性能优化。
4. **获取代理**：调用 `proxyFactory.getProxy(classLoader)`。

#### 1.5.2. JDK vs CGLIB 决策树

决策逻辑由 `DefaultAopProxyFactory` 实现。

![Mermaid Chart - Create complex, visual diagrams with text.-2025-11-28-130406.svg](https://r2.129870.xyz/img/2025/6e2798a40d35bd30f4b1d3794f1568fc.svg)

#### 1.5.3. 两种代理方式对比

| 特性 | JDK 动态代理 | CGLIB 代理 |
| :--- | :--- | :--- |
| **核心原理** | 反射 (`java.lang.reflect.Proxy`) | 字节码生成 (ASM) + 继承 |
| **前提条件** | 目标类必须**实现接口** | 目标类不能是 `final` (无法继承) |
| **Spring 默认** | 有接口时的默认选择 | 无接口或强制配置时的选择 |
| **Spring Boot 2.x+** | 默认 `false` | **默认选择** (`proxy-target-class=true`) |

> [!NOTE] **Spring Boot 的默认行为变化**
> 在 Spring Boot 2.x 及以上版本中，`spring.aop.proxy-target-class` 默认为 `true`。这意味着除非你显式配置为 `false` 或者目标类是接口，否则 Spring Boot 倾向于使用 CGLIB。这是因为 CGLIB 性能已大幅提升，且代理类本身（而非接口）在注入时更加方便。

### 1.6. 总结

Spring AOP 代理创建过程体现了极高的设计水准：

1. **严谨的生命周期管理**：通过 `BeanPostProcessor` 在初始化后介入，同时利用 `earlyProxyReferences` 完美解决了循环依赖问题。
2. **高效的筛选机制**：利用 `ClassFilter` 进行快速初筛，避免了不必要的方法级匹配，并利用缓存 (`advisedBeans`) 提升性能。
3. **灵活的策略模式**：根据配置和目标类特征，动态选择 JDK 或 CGLIB 代理，既保证了兼容性（JDK），又提供了灵活性（CGLIB）。

## 2. Spring AOP 实现原理之 Cglib 代理

### 2.1. Cglib 原生实现机制

在理解 Spring 源码前，必须掌握 Cglib 的两个核心组件：

* **`Enhancer`**：字节码生成器（工厂），用于生成代理类（子类）。
* **`MethodInterceptor`**：方法拦截器（回调），所有代理方法的调用都会转发到这里。

#### 2.1.1. 标准 Demo 重构

```java
public class LogProxy implements MethodInterceptor {
    // 创建代理对象
    public Object getProxy(Class<?> targetClass) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(targetClass); // 1. 设置父类 (目标类)
        enhancer.setCallback(this);          // 2. 设置回调 (拦截器)
        return enhancer.create();            // 3. 生成字节码并实例化
    }

    // 拦截逻辑
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("[Before] ...");
        
        // 关键点：调用 invokeSuper
        // 意思是在代理对象(obj)上调用父类(目标类)的方法
        // 这种方式利用 FastClass 机制，比反射更快，且不需要持有目标对象引用
        Object result = proxy.invokeSuper(obj, args);
        
        System.out.println("[After] ...");
        return result;
    }
}
```

#### 2.1.2. 关键限制

**Final 方法无法代理**：因为 Cglib 基于继承和重写，Java 规定 `final` 方法不能被重写，因此无法被拦截。调用代理对象的 `final` 方法会直接执行父类原逻辑。

### 2.2. Spring 的代理工厂：`CglibAopProxy`

Spring 并没有直接使用 `Enhancer`，而是通过 `CglibAopProxy` 类进行了复杂的工程化封装。

#### 2.2.1. `getProxy` 构建流程

该方法负责配置 `Enhancer` 并生成代理对象。

![Mermaid Chart - Create complex, visual diagrams with text.-2025-11-28-134034.svg](https://r2.129870.xyz/img/2025/2f7603199c832bc6cdaf076f78ce9dc1.svg)

#### 2.2.2. 关键设计哲学

##### 2.2.2.1. 扁平化处理 (Flattening)

* **问题**：如果一个 Bean 已经被 Cglib 代理过（例如被其他框架代理），Spring AOP 再次代理时，如果直接继承这个代理类，会导致继承链过深（Proxy -> Proxy -> Target），逻辑复杂且易错。
* **解决**：Spring 检测到 `rootClass` 已经是 Cglib 代理类时，会**剥离**外壳，直接以其**父类（原始目标类）**作为新代理的父类。确保结构始终是 `Proxy -> Target`。

##### 2.2.2.2. 接口注入

Spring 会强制代理类实现 `SpringProxy` 和 `Advised` 接口。这使得我们可以将任何 Spring 代理对象强转为 `Advised` 类型，从而在运行时动态操作切面。

### 2.3. 运行时拦截逻辑：Callback 体系

这是 Spring AOP 的“心脏”。Spring 配置了一个包含 7 个拦截器的数组，并使用 `CallbackFilter` 来决定每个方法走哪个拦截器。

#### 2.3.1. Callback 数组结构

`getCallbacks()` 方法返回的数组通常包含：

1. **`DynamicAdvisedInterceptor` (索引 0)**：**核心**。负责执行 AOP 切面链。
2. **`StaticUnadvisedInterceptor`**：优化。直接调用目标方法（用于 `expose-proxy` 场景）。
3. **`SerializableNoOp`**：空操作。
4. **`EqualsInterceptor` / `HashCodeInterceptor`**：特殊处理 `equals` 和 `hashCode`，确保比较的是代理配置而非对象地址。

#### 2.3.2. 核心拦截器：`DynamicAdvisedInterceptor`

当业务方法被调用时，会进入此拦截器的 `intercept` 方法。

**源码逻辑精简：**

```java
public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) {
    // 1. 获取 Advisor 链 (带有缓存)
    // 根据方法名匹配 Pointcut，找出所有需要执行的切面
    List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);

    // 2. 处理 expose-proxy
    // 如果配置开启，将当前代理对象放入 ThreadLocal
    if (this.advised.isExposeProxy()) {
        AopContext.setCurrentProxy(proxy);
    }

    // 3. 执行链
    if (chain.isEmpty()) {
        // 无切面，直接调用目标方法 (利用 FastClass)
        return methodProxy.invoke(target, args);
    } else {
        // 有切面，启动递归调用链 (Before -> Target -> After)
        return new CglibMethodInvocation(..., chain, ...).proceed();
    }
}
```

#### 2.3.3. 运行时时序图

![Mermaid Chart - Create complex, visual diagrams with text.-2025-11-28-135517.svg](https://r2.129870.xyz/img/2025/6e9209e2451e0fcdfc7a022db8684e30.svg)

### 2.4. 常见面试题与疑难点

#### 2.4.1. Q1: `this` 调用导致 AOP 失效的问题

* **现象**：在方法 A 中调用 `this.methodB()`，即使 B 有 `@Transactional`，事务也不生效。
* **原因**：`this` 代表的是**目标对象**（Target），而不是代理对象。内部调用直接绕过了 Cglib 的拦截器。
* **解决**：
    1. 配置 `expose-proxy=true`。
    2. 使用 `((Service) AopContext.currentProxy()).methodB()` 进行调用。

#### 2.4.2. Q2: Cglib 比 JDK 代理快吗？

* **创建性能**：JDK 代理创建更快（直接生成字节码较少）。Cglib 创建较慢。
* **运行性能**：
    * 在 JDK 6/7 时代，Cglib 明显快于 JDK 代理（因为 JDK 使用反射，Cglib 使用 FastClass 索引）。
    * 在 JDK 8+ 之后，JDK 反射性能大幅优化，两者差距已非常小，甚至 JDK 代理在某些场景下更快。
* **结论**：现在选择哪种方式主要取决于**是否有接口**，而不是性能。

#### 2.4.3. Q3: 什么是 FastClass 机制？

Cglib 不使用 Java 反射 (`Method.invoke`) 来调用目标方法，而是为每个代理类生成一个索引类 (`FastClass`)。它根据方法签名计算出一个 `int index`，然后通过 `switch(index)` 直接调用对应的方法。这避免了反射的安全检查开销。

## 3. Spring AOP 中 JDK 代理的深度剖析

在 Spring AOP 中，`JdkDynamicAopProxy` 是一个至关重要的类。它身兼数职：既是**代理工厂**（生产代理对象），又是**调用处理器**（处理代理对象的逻辑执行）。

### 3.1. 核心设计理念与角色

`JdkDynamicAopProxy` 实现了两个核心接口：

1. `AopProxy`：定义了 `getProxy()` 方法，用于生成代理对象。
2. `InvocationHandler`：JDK 动态代理的标准接口，定义了 `invoke()` 方法，用于拦截方法调用。

**为什么这样设计（Rationality）？**这种设计体现了**高内聚**。将“如何创建代理”和“代理执行什么逻辑”封装在同一个类中，因为这两者在 Spring AOP 的上下文中是紧密绑定的（代理对象创建时必须绑定 `this` 作为 Handler）。

### 3.2. Spring AOP JDK 代理的创建

代理对象的创建过程非常简洁，核心在于 `getProxy` 方法。

#### 3.2.1. 源码解析

```java
@Override
public Object getProxy() {
    return getProxy(ClassUtils.getDefaultClassLoader());
}

@Override
public Object getProxy(@Nullable ClassLoader classLoader) {
    if (logger.isTraceEnabled()) {
        logger.trace("Creating JDK dynamic proxy: " + this.advised.getTargetSource());
    }
    // 核心逻辑：调用 JDK 原生 API 生成代理对象
    // 1. classLoader: 类加载器
    // 2. proxiedInterfaces: 被代理的接口集合（Spring 会自动收集目标对象实现的接口）
    // 3. this: InvocationHandler，即 JdkDynamicAopProxy 实例本身
    return Proxy.newProxyInstance(classLoader, this.proxiedInterfaces, this);
}
```

**必要性说明：**
这里传入 `this` 是关键。这意味着生成的代理对象在调用任何接口方法时，都会回调 `JdkDynamicAopProxy.invoke()` 方法，从而进入 Spring 的 AOP 拦截逻辑。

### 3.3. Spring AOP JDK 代理的执行 (`invoke` 方法)

这是 AOP 的心脏。当我们在代码中调用 `userService.findUserList()` 时，实际上执行的是 `JdkDynamicAopProxy.invoke()`。

#### 3.3.1. 执行流程 Mermaid 图解

![Mermaid Chart - Create complex, visual diagrams with text.-2025-11-28-141314.svg](https://r2.129870.xyz/img/2025/36c2a3d17e0af2813f744938a022623d.svg)

#### 3.3.2. 源码深度剖析与纠偏

原笔记中的源码注释较为简单，以下是对关键步骤的详细解读及“为什么要这么做”的分析。

```java
@Override
@Nullable
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    Object oldProxy = null;
    boolean setProxyContext = false;

    TargetSource targetSource = this.advised.targetSource;
    Object target = null;

    try {
        // 1. 基础方法处理
        // 为什么？equals 和 hashCode 通常不需要 AOP 增强，且需要保证代理对象自身的身份识别逻辑正确。
        if (!this.equalsDefined && AopUtils.isEqualsMethod(method)) {
            return equals(args[0]);
        }
        else if (!this.hashCodeDefined && AopUtils.isHashCodeMethod(method)) {
            return hashCode();
        }
        // ... 省略部分装饰类判断 ...

        Object retVal;

        // 2. 暴露代理对象到 ThreadLocal
        // 为什么？解决“自调用”问题。如果目标对象内部调用自己的另一个方法（this.methodB()），
        // 默认是不会触发 AOP 的。通过 AopContext.currentProxy() 可以获取当前代理对象来调用。
        if (this.advised.exposeProxy) {
            oldProxy = AopContext.setCurrentProxy(proxy);
            setProxyContext = true;
        }

        // 获取目标对象（Target）
        target = targetSource.getTarget();
        Class<?> targetClass = (target != null ? target.getClass() : null);

        // 3. 获取拦截器链（核心）
        // 解释：Spring 会根据配置（Pointcut）判断当前方法需要执行哪些 Advice（Before, After, Around等），
        // 并将它们封装成一个 List<Object> chain。
        List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);

        // 4. 策略判断：直接调用 vs 链式调用
        if (chain.isEmpty()) {
            // 优化：如果没有切面需要执行，直接反射调用目标方法，避免创建 MethodInvocation 对象的开销。
            Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
            retVal = AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse);
        }
        else {
            // 5. 创建方法调用器并执行
            // 为什么？ReflectiveMethodInvocation 维护了拦截器链的执行状态（当前执行到第几个拦截器了）。
            // 它实现了“责任链模式”，确保所有 Before 通知在目标方法前执行，After 通知在目标方法后执行。
            MethodInvocation invocation =
                    new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
            
            // 开始递归执行拦截器链
            retVal = invocation.proceed();
        }

        // 6. 处理返回值
        // 为什么？如果方法返回的是 this，需要将其替换为 proxy，防止原始对象泄露（除非返回类型不兼容）。
        Class<?> returnType = method.getReturnType();
        if (retVal != null && retVal == target &&
                returnType != Object.class && returnType.isInstance(proxy) &&
                !RawTargetAccess.class.isAssignableFrom(method.getDeclaringClass())) {
            retVal = proxy;
        }
        // 处理基本数据类型返回 null 的异常情况
        else if (retVal == null && returnType != Void.TYPE && returnType.isPrimitive()) {
            throw new AopInvocationException(
                    "Null return value from advice does not match primitive return type for: " + method);
        }
        return retVal;
    }
    finally {
        // 7. 资源释放与上下文恢复
        if (target != null && !targetSource.isStatic()) {
            targetSource.releaseTarget(target);
        }
        if (setProxyContext) {
            // 恢复旧的代理对象，防止 ThreadLocal 污染
            AopContext.setCurrentProxy(oldProxy);
        }
    }
}
```

### 3.4. 关键点总结与补充

#### 3.4.1. 为什么需要 `ReflectiveMethodInvocation`？

在源码中，`invocation.proceed()` 是触发切面逻辑的关键。

* **必要性**：拦截器链（Chain）是一个列表，包含多个 Advice（如事务、日志、安全）。我们需要一个协调者来按顺序执行它们，并在所有拦截器执行完毕后，最终调用目标对象的方法。
* **实现原理**：`ReflectiveMethodInvocation` 内部维护了一个索引 `currentInterceptorIndex`。每次调用 `proceed()`，索引加 1，取出下一个拦截器执行；当索引到达链尾时，通过反射调用 `target` 的实际方法。

#### 3.4.2. 性能考量

Spring 在 `invoke` 方法中做了明显的性能优化：

* **缓存拦截器链**：`getInterceptorsAndDynamicInterceptionAdvice` 方法内部有缓存机制，不会每次调用都重新解析切面配置。
* **快速路径**：如果 `chain.isEmpty()`，直接反射调用，跳过复杂的责任链创建过程。