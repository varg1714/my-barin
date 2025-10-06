---
source: "[[哈罗面试：Spring 连环问，Spring Bean 生命周期，Spring 怎么创建 Bean 的，BFPP 和 BPP 的 x 别]]"
create: 2025-10-03
---

## 1. Spring Bean 生命周期终极版深度解析

### 1.1. 一、核心设计哲学：为何需要复杂的生命周期？

Spring Bean 的生命周期并非为了复杂而复杂，它的每一个阶段都服务于 Spring 的核心设计目标，理解这一点是掌握全局的关键。

* **实现控制反转 (IoC)**：Spring 的核心是容器，而非开发者去创建和管理对象。这个生命周期就是 Spring **全面接管**对象管理权的体现，从定义、创建、组装到销毁。
* **提供极致的可扩展性**：Spring 在生命周期的关键节点，通过 `BeanFactoryPostProcessor` 和 `BeanPostProcessor` 等接口，开放了强大的**钩子 (Hooks)**。这使得开发者和第三方框架（如 AOP、MyBatis、Dubbo）能够无缝地介入和增强 Bean 的行为，而无需修改 Spring 源码。
* **解耦与灵活性**：通过支持 JSR-250 标准（如 `@PostConstruct`）和提供多种配置方式（注解、接口、XML），Spring 允许开发者在**框架耦合度**和**开发便利性**之间做出灵活选择。

### 1.2. 二、Bean 生命周期的全景图：从定义到销毁的详细旅程

我们将整个生命周期划分为五个主要阶段，并深入每个阶段的内部细节。

#### 1.2.1. 阶段一：准备阶段 (容器启动)

此阶段在任何 Bean 实例被创建之前发生，主要工作是准备“蓝图”和“生产环境”。

1. **扫描与加载 `BeanDefinition`**
    * **动作**：Spring 容器启动，通过 `ConfigurationClassPostProcessor` 等工具扫描指定的包（如 `@ComponentScan`）或解析 XML 文件。
    * **产物**：为每个找到的 Bean（带有 `@Component`, `@Service` 等注解的类）创建一个 `BeanDefinition` 对象。
    * **`BeanDefinition` 是什么？**：它是 Bean 的**元数据描述**或**“设计蓝图”**。包含了创建 Bean 所需的一切信息：类名、作用域 (singleton/prototype)、构造函数参数、属性值、是否懒加载、`init-method` 等。

2. **执行 `BeanFactoryPostProcessor` (BFPP)**
    * **时机**：在所有 `BeanDefinition` 加载完成，但**任何 Bean 实例都还未创建**时。
    * **目的**：这是 Spring 提供的第一个大规模扩展点，允许我们对**所有 Bean 的“设计蓝图”**进行全局性的修改。
    * **核心方法**：`postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory)`。
    * **典型应用**：
        * `PropertySourcesPlaceholderConfigurer`：解析配置文件中的 `${...}` 占位符，并将其替换为实际值。
        * **自定义加密/解密**：如我们讨论的，在属性注入前，对配置文件中的加密值进行解密。这是最优雅的实现方式。
        * 动态修改 Bean 的作用域或属性。

#### 1.2.2. 阶段二：实例化与属性填充阶段 (创建与组装)

此阶段开始真正地创建 Bean 对象并组装其依赖关系。

3. **实例化 (Instantiation)**
    * **动作**：Spring 根据 `BeanDefinition`，通过反射调用类的构造函数（或工厂方法）来创建一个**原始的、空的 Bean 对象**。此时，对象的所有属性都还是默认值（`null`, `0`, `false`）。
    * **核心方法**：`AbstractAutowireCapableBeanFactory#createBeanInstance`。

4. **早期暴露与解决循环依赖**
    * **动作**：实例化之后，Spring 会立即将一个能获取到该原始对象的 `ObjectFactory` 放入**三级缓存 (`singletonFactories`)** 中。
    * **目的**：这是解决单例 Bean 循环依赖的关键。如果其他 Bean 在创建过程中需要注入当前 Bean，它可以从缓存中提前获取一个“早期引用”（可能是一个原始对象或一个早期代理），从而打破循环等待。

5. **属性填充 (Populate Properties)**
    * **动作**：Spring 遍历 `BeanDefinition` 中的属性信息，使用 `@Autowired`, `@Resource`, `@Value` 等注解，将依赖的其他 Bean 或配置文件中的值注入到原始对象中。
    * **核心方法**：`AbstractAutowireCapableBeanFactory#populateBean`。

#### 1.2.3. 阶段三：初始化阶段 (精加工与增强)

这是生命周期中最复杂也最关键的阶段，Bean 在此阶段从一个“半成品”变为一个完全可用的“成品”。

6. **调用 `Aware` 系列接口**
    * **目的**：让 Bean “感知”到它所处的容器环境，并获取容器的资源。这是 Bean 与框架交互的桥梁。
    * **顺序与示例**：
        * `BeanNameAware`：注入 Bean 在容器中的 ID。
        * `BeanClassLoaderAware`：注入加载该 Bean 的 `ClassLoader`。
        * `BeanFactoryAware`：注入创建该 Bean 的 `BeanFactory`。

7. **执行 `BeanPostProcessor` 前置处理**
    * **核心方法**：`postProcessBeforeInitialization(Object bean, String beanName)`。
    * **时机**：在所有 `Aware` 接口回调之后，在任何自定义初始化方法（如 `@PostConstruct`）之前。
    * **目的**：提供一个在 Bean 自身初始化逻辑执行前，进行外部干预的机会。例如，对某些属性进行预处理或校验。

8. **执行 Bean 自身的初始化方法**
    * **目的**：为开发者提供一个时机，在所有依赖都已注入后，执行自定义的启动逻辑（如加载资源、建立连接、预热缓存等）。
    * **执行顺序是固定的**：
        1. **`@PostConstruct` 注解的方法**：源自 JSR-250 标准，**官方首选**，因为它与 Spring 框架解耦。
        2. **`InitializingBean` 接口**：如果 Bean 实现了此接口，则调用其 `afterPropertiesSet()` 方法。这是 Spring 早期的实现，与框架强耦合，不推荐在新代码中使用。
        3. **自定义 `init-method`**：在 XML 或 `@Bean` 注解中指定的初始化方法。它最大的优势是**非侵入性**，可以为无法修改源码的第三方库类指定初始化逻辑。
9. **执行 `BeanPostProcessor` 后置处理**
    * **核心方法**：`postProcessAfterInitialization(Object bean, String beanName)`。
    * **时机**：在 Bean 所有自身的初始化方法都执行完毕之后。
    * **目的**：这是对一个**完全初始化**的 Bean 进行最终增强或替换的最后机会。
    * **最重要的应用：Spring AOP**。`AnnotationAwareAspectJAutoProxyCreator` 就是一个 `BPP`，它在此阶段检查 Bean 是否需要被代理。如果需要，它会创建一个代理对象并返回。因此，**Spring 容器最终持有的可能是这个代理对象，而不是原始对象**。

#### 1.2.4. 阶段四：使用阶段 (服务期)

10. **Bean 就绪**
    * **状态**：此时的 Bean 已经是完全配置好、初始化完毕，并且可能已经被代理的最终形态。
    * **动作**：Spring 将这个最终的 Bean 放入**一级缓存 (`singletonObjects`)** 中，并移除二、三级缓存中的临时数据。现在，这个 Bean 可以被应用程序安全地获取和使用了。

#### 1.2.5. 阶段五：销毁阶段 (生命终点)

11. **执行销毁回调**
    * **时机**：当 Spring 容器关闭时（如调用 `context.close()`），对于所有单例 Bean。
    * **目的**：执行清理工作，优雅地释放资源（如关闭数据库连接池、停止线程池等）。
    * **执行顺序与初始化阶段对称**：
        1. **`@PreDestroy` 注解的方法**：JSR-250 标准，**首选**。
        2. **`DisposableBean` 接口**：调用其 `destroy()` 方法。
        3. **自定义 `destroy-method`**。

### 1.3. 六、核心扩展点 `BFPP` vs `BPP` 的终极对比

| 维度 | `BeanFactoryPostProcessor` (BFPP) | `BeanPostProcessor` (BPP) |
| :--- | :--- | :--- |
| **比喻** | **图纸设计师** (Blueprint Modifier) | **生产线质检/改装工** (Product Processor) |
| **作用时机** | 容器启动后，所有 Bean **实例化之前** | 每个 Bean 实例化后，**初始化前后** |
| **处理对象** | `BeanDefinition` (Bean 的元数据) | Bean 实例本身 (活生生的对象) |
| **作用范围** | **全局性**，作用于整个 Bean 工厂 | **个体性**，作用于每个被创建的 Bean 实例 |
| **核心方法** | `postProcessBeanFactory(...)` | `postProcessBeforeInitialization(...)`<br>`postProcessAfterInitialization(...)` |
| **核心目的** | 在 Bean 创建前，修改其**定义和配置** | 在 Bean 创建过程中，**增强或替换** Bean 实例 |
| **经典案例** | `PropertySourcesPlaceholderConfigurer` (处理 `${...}`)<br>Jasypt (处理加密配置) | `AnnotationAwareAspectJAutoProxyCreator` (创建 AOP 代理)<br>`AutowiredAnnotationBeanPostProcessor` (处理 `@Autowired`) |

