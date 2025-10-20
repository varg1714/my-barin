---
source:
  - "[[Spring 进阶 - Spring IOC 实现原理详解之 Bean 实例化 (生命周期_ 循环依赖等) _ Java 全栈知识体系]]"
  - "[[spring 三级缓存 - 搜索结果 - 知乎]]"
create: 2025-10-13
---

## 1. Spring IoC 深度解析：三级缓存、Bean 生命周期与容器架构

本文档是对 Spring IoC 核心原理的深度整合与剖析，旨在彻底厘清循环依赖的解决方案、Bean 的完整生命周期以及容器的核心架构。内容源于对 Spring 源码的分析和权威技术文章的提炼，结构化、细节化地呈现了 Spring 的内部工作机制。

### 1.1. Part 1: 核心难题 - 循环依赖的精妙解决方案

Spring IoC 容器设计的一个亮点便是其对循环依赖的处理能力。但需要明确的是，Spring 仅解决了特定场景下的循环依赖：

- **作用域**：仅限 `singleton` (单例) Bean。
- **注入方式**：仅限属性注入 (Field Injection, 如 `@Autowired`) 或 Setter 方法注入。

#### 1.1.1. 三级缓存：The Three Pillars

Spring 使用三个独立的缓存（本质是 Map）协同工作，构成了解决循环依赖的基石。这三个缓存均定义在 `DefaultSingletonBeanRegistry` 类中。

| 缓存级别 | 变量名 | 类型 | 存放内容 | 核心职责与设计意图 |
| :--- | :--- | :--- | :--- | :--- |
| **一级缓存** | `singletonObjects` | `Map<String, Object>` | **成品 Bean** | **最终归宿**。存放已完成完整生命周期（实例化、属性注入、初始化）的单例对象。这是最常用的缓存，提供稳定、可用的 Bean 实例。 |
| **二级缓存** | `earlySingletonObjects` | `Map<String, Object>` | **早期暴露的半成品 Bean** | **打破循环的关键**。存放已实例化但尚未完成属性注入和初始化的对象。它的存在是为了让依赖它的其他 Bean 能提前获取到一个引用，从而让创建流程得以继续。**它保证了在一次循环依赖中，所有依赖方获取到的早期引用是同一个对象实例**。 |
| **三级缓存** | `singletonFactories` | `Map<String, ObjectFactory<?>>` | **Bean 工厂** | **处理 AOP 代理的精髓**。它不直接存储对象，而是存储一个能创建该对象的工厂 (`ObjectFactory`)。这个工厂封装了“如果需要代理，则创建代理；否则返回原始对象”的逻辑。**实现了代理对象的按需、延迟创建**。 |

#### 1.1.2. 深度剖析：为什么必须是三级？

##### 1.1.2.1. 场景一：无 AOP 的循环依赖（二级缓存即可解决）

这是一个理想化的简单场景，用于理解早期暴露机制。

1. **创建 A**: `getBean("a")` -> `doCreateBean("a")`。
2. **实例化 A**: `createBeanInstance("a")`，通过反射 `new A()` 得到一个原始对象 `a`。此时 `a.b` 为 `null`。
3. **早期暴露 A**: 将原始对象 `a` 放入**二级缓存 `earlySingletonObjects`**。
4. **填充 A**: `populateBean("a")`，发现 `a` 依赖 `b`。
5. **创建 B**: `getBean("b")` -> `doCreateBean("b")`。
6. **实例化 B**: `new B()` 得到原始对象 `b`。
7. **填充 B**: `populateBean("b")`，发现 `b` 依赖 `a`。
8. **解决 B 的依赖**: `getBean("a")`。
    - 查一级缓存 `singletonObjects` -> 未找到。
    - 查**二级缓存 `earlySingletonObjects`** -> **成功找到**半成品 `a`。
9. **注入 A 到 B**: 将半成品 `a` 的引用注入到 `b.a`。
10. **B 创建完成**: `b` 完成后续初始化，成为成品，被放入**一级缓存 `singletonObjects`**，并从二级缓存中移除。
11. **A 创建完成**: `a` 的创建流程继续，获取到成品 `b` 并注入到 `a.b`。`a` 完成初始化，成为成品，放入**一级缓存**，并从二级缓存中移除。

##### 1.1.2.2. 场景二：有 AOP 的循环依赖（三级缓存的舞台）

这是真实且复杂的场景，揭示了三级缓存的必要性。

**核心矛盾**：AOP 代理的创建时机（`BeanPostProcessor` 的初始化后阶段）与循环依赖注入的时机（属性填充阶段）存在冲突。注入的必须是代理对象，但代理对象在正常流程中尚未创建。

**三级缓存的优雅解决流程：**

1. **创建 A**: `getBean("a")` -> `doCreateBean("a")`。
2. **实例化 A**: `new A()` 得到原始对象 `a`。
3. **暴露工厂**: Spring **不**将 `a` 放入二级缓存，而是创建一个 `ObjectFactory`，并将其放入**三级缓存 `singletonFactories`**。这个工厂的 `getObject()` 方法大致逻辑是：
    调用 `getEarlyBeanReference()`，检查当前 Bean 是否需要被 AOP 代理。如果需要，创建代理对象并返回；如果不需要，返回原始对象。

    是否创建代理对象由 `SmartInstantiationAwareBeanPostProcessor` 决定，在容器启动时，Spring 会扫描所有的配置，找到所有定义了 AOP 功能的组件。在现代 Spring 中，这通常是带有 `@Aspect` 注解的类。Spring 会解析这些类，把里面的切点（`@Pointcut`）和通知（`@Before`, `@Around` 等）打包成一个叫做 **`Advisor`** 的对象。

    `getEarlyBeanReference()` 内部就会去检查所有的 `Advisor` 规则，看看当前这个半成品的 Bean 是否符合任何一条 AOP 规则。

    - **如果符合**：就立刻为它创建代理对象，并把这个**代理对象**放入二级缓存，以供其他 Bean 依赖。
    - **如果不符合**：就直接返回原始对象。
    
    > [!warning]- 早期暴露 bean 与 BeanPostProcessor 的冲突
    > **一个关键问题：自定义代理与循环依赖**
    > 
    > 一个自然而深刻的疑问是：如果开发者自定义了一个 `BeanPostProcessor`，它也在初始化后期（`postProcessAfterInitialization`）返回了一个代理对象（我们称之为 `MyProxy`），这与 Spring AOP 的代理（`AopProxy`）以及三级缓存的早期暴露机制如何交互？
    > 
    > 如果 Bean A 和 Bean B 循环依赖，并且 A 同时需要被 AOP 代理和我们自定义的 `MyProxy` 代理，会发生什么？
    > 
    > 1. 在解决循环依赖时，B 会触发 A 的 `ObjectFactory`，`getEarlyBeanReference` 方法被调用，创建并返回了 A 的早期代理 `AopProxy`，并放入二级缓存。B 成功注入了 `AopProxy`。
    > 2. A 的生命周期继续，走到 `initializeBean` 阶段，执行所有 `BeanPostProcessor` 的 `postProcessAfterInitialization` 方法。
    > 3. 此时，我们自定义的处理器执行，它接收到 `AopProxy`，并将其再次包装，返回了一个全新的 `MyProxy` 对象。
    > 4. **矛盾产生**：最终要放入一级缓存的是 `MyProxy`，但之前为了打破循环而暴露给 B 的是 `AopProxy`。这意味着容器内外对 Bean A 的引用不一致，严重违反了单例原则。
    > 
    > **Spring 的解决方案：防御性检查与正确的扩展点**
    > 
    > Spring 的设计者预见了这种致命的矛盾，并设计了周密的机制来应对：
    > 
    > **1. 防御性机制：最终一致性检查**
    > 
    > 在 Bean 创建流程的最后阶段（位于 `AbstractAutowireCapableBeanFactory` 的 `doCreateBean` 方法末尾），Spring 会进行一次严格的最终一致性检查：
    > 
    > - **检查前提**：该 Bean 是否曾因循环依赖被提前暴露过（`earlySingletonExposure` 标志）。
    > - **检查逻辑**：比较“当初提前暴露的引用”（`earlySingletonReference`，即 `AopProxy`）和“经过所有后置处理器处理后的最终对象”（`exposedObject`，即 `MyProxy`）。
    > - **触发异常**：如果发现两者不一致，并且确实有其他 Bean 依赖了它，Spring 会立即抛出 `BeanCurrentlyInCreationException`，并附带非常明确的错误信息，指出“原始版本已被注入，但最终被包装，导致引用不一致”，从而阻止这种错误的发生。
    > 
    > **2. 扩展性设计：正确的扩展点 `SmartInstantiationAwareBeanPostProcessor`**
    > 
    > 为了让开发者能够安全地在循环依赖场景下自定义代理，Spring 提供了 `BeanPostProcessor` 的一个更为强大的子接口：`SmartInstantiationAwareBeanPostProcessor`。
    > 
    > |接口| `postProcessAfterInitialization` | `getEarlyBeanReference` |
    > |:--|:--|:--|
    > |**`BeanPostProcessor`**|在 Bean **初始化后**执行。适用于绝大多数情况，但处理循环依赖代理时机已晚。|(无此方法)|
    > |**`SmartInstantiationAwareBeanPostProcessor`**|同样拥有此方法。|在 Bean **实例化后**、**循环依赖发生时**执行，是专门用于解决提前暴露引用的扩展点。|
    > 
    > `ObjectFactory` 的核心职责，正是调用所有 `SmartInstantiationAwareBeanPostProcessor` 的 `getEarlyBeanReference` 方法。
    > 
    > 因此，正确的做法是让我们自定义的处理器实现 `SmartInstantiationAwareBeanPostProcessor` 接口，并将代理逻辑放入 `getEarlyBeanReference` 方法中。这样，当循环依赖发生时：
    > 
    > 1. `ObjectFactory` 被调用。
    > 2. 它会依次调用所有相关处理器（包括 Spring AOP 的和我们自定义的）的 `getEarlyBeanReference` 方法。
    > 3. 通过 `@Order` 控制顺序，可以形成一个代理链。例如，我们的处理器先将原始对象包装成 `MyProxy`，然后 AOP 处理器再将 `MyProxy` 包装成 `AopProxy`。
    > 4. 最终，**提前暴露出去的是这个经过层层包装的最终形态的代理对象**。
    > 5. 由于代理在最早的时机就已经创建完毕，后续的生命周期中引用保持一致，完美解决了矛盾。
    > 
    > **结论**：三级缓存中的 `ObjectFactory` 不仅仅是为了延迟 AOP 代理的创建，它更是 Spring 提供的一个精妙的扩展点，它确保了在复杂的循环依赖场景下，所有需要对 Bean 进行早期修改（尤其是代理）的操作都能以一种有序、一致的方式进行，从而保证了单例 Bean 引用的绝对唯一性。
4. **填充 A**: `populateBean("a")`，发现依赖 `B`。
5. **创建 B**: `getBean("b")` -> ... -> 发现依赖 `A`。
6. **解决 B 的依赖**: `getBean("a")`。
    - 查一级缓存 -> 未找到。
    - 查二级缓存 -> 未找到。
    - 查**三级缓存 `singletonFactories`** -> **成功找到** `a` 对应的 `ObjectFactory`。
7. **调用工厂，创建代理**: 调用该工厂的 `getObject()` 方法。该方法内部通过 `SmartInstantiationAwareBeanPostProcessor` (如 AOP 的后置处理器) 创建了 `A` 的**代理对象**并返回。
8. **缓存升级**: 这个新创建的**代理对象 `A`** 被放入**二级缓存 `earlySingletonObjects`**，同时从三级缓存中移除对应的工厂。这一步至关重要，确保了后续任何对 `A` 的早期依赖都将从二级缓存获取到同一个代理实例。
9. **注入 A 到 B**: `B` 成功获取到 `A` 的**代理对象**，完成创建，放入一级缓存。
10. **A 创建完成**: `A` 的流程继续，获取到成品 `B`。在 `A` 的生命周期末尾，Spring 会检查并确保最终放入一级缓存的是在第 7 步创建的那个**代理对象**，而不是原始对象，保证了容器内外引用的唯一性和一致性。

##### 1.1.2.3. 可视化流程

![三级缓存.svg](https://r2.129870.xyz/img/2025/b2fc14a1672d7fe8ffdc4bf5ed7cadae.svg)

#### 1.1.3. 边界情况与替代方案思考

- **为什么不能解决构造器循环依赖？**
    在执行构造函数时，对象本身还未创建完成，无法生成一个有效的引用放入三级缓存中提前暴露。这是一个“先有鸡还是先有蛋”的问题。
- **为什么不能解决 `prototype` 循环依赖？**
    Spring 对 `prototype` Bean 的设计理念是“用后即焚”，容器不持有其引用，因此不会对其进行缓存。没有缓存，就无法打破依赖环。
- **只用二级缓存可以吗？**
    **可以，但不优雅**。如果只用二级缓存，就必须在 Bean 实例化后，立即判断是否需要 AOP 并创建代理，然后将代理对象放入二级缓存。这**破坏了 Spring 将 AOP 逻辑后置到初始化阶段的设计原则**，并且会造成**性能浪费**（为很多并未卷入循环依赖的 Bean 过早地创建了代理）。

### 1.2. Part 2: 宏观蓝图 - Bean 的完整生命周期

Bean 的生命周期是理解 Spring 所有高级功能（AOP、事务、事件等）的基础。它定义了一个对象如何从一个简单的类定义，演变为一个功能完备、被容器管理的组件。

![Spring Bean 生命周期流程](https://pdai.tech/images/spring/springframework/spring-framework-ioc-source-102.png)

#### 1.2.1. Bean 创建的详细阶段与扩展点

1. **准备阶段：BeanDefinition 加载**
    Spring 容器启动，通过 `BeanFactoryPostProcessor` (如 `ConfigurationClassPostProcessor`) 扫描配置（XML, 注解），解析成 `BeanDefinition` 对象，并注册到 `beanDefinitionMap` 中。

2. **实例化 (Instantiation)**
    - **`InstantiationAwareBeanPostProcessor.postProcessBeforeInstantiation()`**: 这是第一个能完全中断 Bean 创建流程的扩展点。如果此方法返回一个非 `null` 的对象，该对象将直接作为最终的 Bean 返回，后续的实例化、填充、初始化流程将全部跳过。
    - **确定构造函数并实例化**: Spring 根据 `BeanDefinition` 和参数选择合适的构造函数，通过反射调用 `new` 关键字创建 Bean 的原始实例。

3. **属性填充 (Population)**
    - **`InstantiationAwareBeanPostProcessor.postProcessAfterInstantiation()`**: 在实例化之后、属性注入之前调用。如果返回 `false`，将跳过后续的属性注入。
    - **`InstantiationAwareBeanPostProcessor.postProcessProperties()`**: 对属性值进行处理，例如 `@Autowired` 注解的解析和依赖查找就是在此阶段前后完成的。Spring 在这里完成大部分依赖注入。

4. **初始化 (Initialization)**
    - **`Aware` 接口注入**: Spring 会检查 Bean 是否实现了特定的 `Aware` 接口，并将相应的容器资源注入。这是一个让 Bean 感知到其在容器中“身份”和“环境”的过程。
        - `BeanNameAware.setBeanName()`
        - `BeanClassLoaderAware.setBeanClassLoader()`
        - `BeanFactoryAware.setBeanFactory()`
        - `ApplicationContextAware.setApplicationContext()` (最常用)
    - **`BeanPostProcessor.postProcessBeforeInitialization()`**: 在任何初始化回调（如 `afterPropertiesSet` 或 `init-method`）之前调用。
    - **执行初始化回调**:
        - `@PostConstruct` 注解的方法。
        - `InitializingBean.afterPropertiesSet()`。
        - 自定义的 `init-method`。
    - **`BeanPostProcessor.postProcessAfterInitialization()`**: 在所有初始化回调之后调用。**这是 Spring AOP 实现动态代理的核心位置**。Spring 在这里检查 Bean 是否匹配某个切面，如果匹配，则会用代理对象替换原始对象。

5. **可用与销毁 (In-Use & Destruction)**
    - **Bean 可用**: 完成所有步骤后，Bean 成为“成品”，被放入一级缓存 `singletonObjects`，可供应用程序使用。
    - **销毁**: 当容器关闭时，触发销毁流程。
        - `@PreDestroy` 注解的方法。
        - `DisposableBean.destroy()`。
        - 自定义的 `destroy-method`。

#### 1.2.2. 生命周期与循环依赖的关联

- 循环依赖的解决机制发生在**实例化之后、初始化之前**的阶段。
- `getEarlyBeanReference()` 方法（由 `ObjectFactory` 调用）属于 `InstantiationAwareBeanPostProcessor`，它在生命周期的早期提供了一个提前创建代理的机会。
- 而常规的 AOP 代理创建发生在 `postProcessAfterInitialization()`，这是一个非常靠后的阶段。
- 正是这种时间差，凸显了三级缓存机制的必要性：它为处于生命周期早期的 Bean 提供了一个通往生命周期后期的“捷径”，以满足循环依赖的需求。

### 1.3. Part 3: 容器架构 - `ApplicationContext` vs. `BeanFactory`

这是理解 Spring 容器工作模式的顶层视角。

#### 1.3.1. 核心关系：`ApplicationContext` 是 `BeanFactory` 的超集

- **`BeanFactory`**: 是 Spring IoC 容器的**根接口**和**核心引擎**。它定义了 Bean 管理的最基本规范，是所有 Bean 创建、依赖注入、生命周期管理（包括三级缓存）的**执行者**。它像汽车的**发动机**，提供核心动力。
- **`ApplicationContext`**: 是 `BeanFactory` 的**子接口**，是功能更全面的**高级容器**。它内部**必然包含一个 `BeanFactory` 实例**（通常是 `DefaultListableBeanFactory`），并在其基础上增加了大量企业级功能。它像一辆**完整的汽车**，是整个应用的**协调者和指挥官**。

#### 1.3.2. 详细区别对比

| 特性 | BeanFactory | ApplicationContext |
| :--- | :--- | :--- |
| **继承关系** | 根接口，定义基础规范 | `BeanFactory` 的子接口，功能超集 |
| **Bean 实例化** | **懒加载 (Lazy Loading)**：默认在 `getBean()` 时才创建。 | **预加载 (Eager Loading)**：默认在容器启动时就创建所有单例 Bean。 |
| **优点/缺点** | **优点**：启动快，内存占用少。<br>**缺点**：运行时才暴露配置错误，首次访问慢。 | **优点**：启动时 fail-fast，运行时响应快。<br>**缺点**：启动慢，消耗更多资源。 |
| **功能支持** | 仅基础 DI 和生命周期管理。 | **企业级功能**：国际化(i18n)、事件发布、AOP 集成、Web 环境支持等。 |
| **后置处理器** | 需要**手动编码注册** `BeanPostProcessor` 等。 | **自动发现并注册**，使得 `@Autowired`、AOP 等功能开箱即用。 |
| **开发者体验** | 原始、繁琐，需要深入理解底层。 | 友好、自动化，是现代 Spring 开发的标准。 |
| **典型实现** | `XmlBeanFactory` (已废弃) | `ClassPathXmlApplicationContext`, `AnnotationConfigApplicationContext` 等。 |

#### 1.3.3. 揭秘：为什么在 Spring Boot 中感觉不到“选择”？

在 Spring Boot 应用中，开发者几乎只与 `@Bean`, `@Component` 等注解打交道，从未手动选择使用哪个容器。这是因为 Spring Boot 的自动化配置为我们处理了一切。

**启动流程揭秘：**

![beanfactory.svg](https://r2.129870.xyz/img/2025/220aa3721458d8d3a4d8f61b48784dc8.svg)

**结论**：我们面向 `ApplicationContext` 编程，但实际的 Bean 创建工作由其内部的 `BeanFactory` 引擎完成。Spring Boot 将这个“创建并指挥”的过程完全自动化，提供了无缝的开发体验。理解这一层委托关系，是打通 Spring IoC 知识体系的“最后一公里”。
