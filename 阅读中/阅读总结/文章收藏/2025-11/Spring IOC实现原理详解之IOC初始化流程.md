---
source:
  - "[[Spring进阶- Spring IOC实现原理详解之IOC初始化流程]]"
create: 2025-11-26
---

## 1. Spring IoC 源码深度解析：初始化流程笔记

### 1.1. 核心目标与顶层思维

**核心目标**：探究 Spring 如何将静态的配置资源（如 XML 文件）转化为容器内部可管理的动态数据结构（`BeanDefinition`），并最终注册到容器中。

**数据流转全景**：

$$ \text{XML文件} \xrightarrow{\text{ResourceLoader}} \text{Resource资源} \xrightarrow{\text{DocumentLoader}} \text{Document(DOM)} \xrightarrow{\text{ParserDelegate}} \text{BeanDefinition} \xrightarrow{\text{Registry}} \text{Map<Name, Definition>} $$

#### 1.1.1. 核心设计模式

在阅读源码前，需理解以下设计意图，避免陷入细节：

* **模板方法模式 (Template Method)**：`refresh()` 方法定义了容器启动的标准骨架，子类只需实现特定的资源加载逻辑。
* **策略模式 (Strategy)**：`ResourceLoader` 屏蔽了底层资源（Classpath/File/URL）的差异。
* **单一职责与委派**：
    * `Reader`：负责读取协调。
    * `DocumentLoader`：负责 XML 解析（转 DOM）。
    * `Delegate`：负责 Spring 语义解析（转 BeanDefinition）。
    * `Registry`：负责存储。

### 1.2. 流程全景图 (Mermaid)

![beandefinition.svg](https://r2.129870.xyz/img/2025/0a782c7959bbeb08d0f816842ba3e97f.svg)

### 1.3. 源码级详细步骤梳理

#### 1.3.1. 第一阶段：入口与环境准备

* **入口**：`ClassPathXmlApplicationContext` 构造函数。
* **关键动作**：
    1. `super(parent)`：初始化父类，设置资源加载器（`PathMatchingResourcePatternResolver`）。
    2. `setConfigLocations`：解析配置路径（处理占位符）。
    3. **`refresh()`**：触发容器初始化的核心。

#### 1.3.2. 第二阶段：初始化主体 (refresh)

`refresh()` 是 Spring 容器启动的“总指挥”。

* **核心步骤**：`obtainFreshBeanFactory()`。
    * 这个方法负责**初始化 BeanFactory** 并 **载入 Bean 定义**。
    * 它调用了子类的 `refreshBeanFactory()`。

#### 1.3.3. 第三阶段：加载 Bean 定义 (loadBeanDefinitions)

这是本文分析的重点，位于 `AbstractXmlApplicationContext` 中。

1. **创建读取器**：实例化 `XmlBeanDefinitionReader`。
2. **配置环境**：将 `ResourceLoader`、`EntityResolver`（用于校验 XML）设置给读取器。
3. **执行加载**：调用 `reader.loadBeanDefinitions(configLocations)`。

#### 1.3.4. 第四阶段：从 Resource 到 Document

`XmlBeanDefinitionReader` 开始工作：

1. **资源定位**：利用 `ResourceLoader.getResource()` 将字符串路径转换为 `Resource` 对象。
2. **加载文档**：调用 `doLoadBeanDefinitions` -> `doLoadDocument`。
    * **组件**：`DocumentLoader` (默认实现 `DefaultDocumentLoader`)。
    * **动作**：使用 Java 标准库 (JAXP) 将 XML InputStream 解析为 `org.w3c.dom.Document` 对象。
    * *注：此时还只是纯粹的 XML 解析，与 Spring Bean 逻辑无关。*

#### 1.3.5. 第五阶段：解析 Document 为 BeanDefinition

拿到 `Document` 后，Spring 开始将其翻译为内部结构。

1. **解析入口**：`registerBeanDefinitions` -> `DefaultBeanDefinitionDocumentReader`。
2. **创建代理**：创建 `BeanDefinitionParserDelegate`。
    * *设计意图*：专门负责解析 XML 元素（如 `<import>`, `<alias>`, `<bean>`）的工具类。
    * 如果 XML 解析逻辑变了（比如换了 XML 解析库），只需要改 DocumentLoader；如果 Spring 的\<bean> 标签属性变了，只需要改 Delegate。各司其职，互不干扰。
3. **解析流程** (`parseBeanDefinitions`)：
    * 遍历 XML 根节点下的子节点。
    * 如果是默认命名空间（DefaultNamespace），调用 `parseDefaultElement`。
    * **处理 `<bean>` 标签**：调用 `processBeanDefinition`。
        * 提取 `id`, `name`, `class` 等属性。
        * 将这些信息封装进 **`BeanDefinitionHolder`** 对象中。

#### 1.3.6. 第六阶段：注册 (Registration)

解析完成后，需要将 `BeanDefinition` 存入容器。

1. **工具类调用**：`BeanDefinitionReaderUtils.registerBeanDefinition`。
2. **最终落脚点**：**`DefaultListableBeanFactory`**。
3. **核心代码**：

    ```java
    // DefaultListableBeanFactory.java
    this.beanDefinitionMap.put(beanName, beanDefinition);
    this.beanDefinitionNames.add(beanName);
    ```

    * `beanDefinitionMap`：一个 `ConcurrentHashMap`，是 IoC 容器存储 Bean 定义的“档案库”。

### 1.4. 关键组件术语表

| 组件名 | 作用 | 形象比喻 |
| :--- | :--- | :--- |
| **Resource** | 统一的资源抽象（文件/URL/Classpath） | 原材料 |
| **BeanDefinition** | Bean 的定义信息（类名、作用域、属性等） | 产品图纸 |
| **BeanDefinitionReader** | 读取资源并转换为 BeanDefinition 的接口 | 绘图员 |
| **DocumentLoader** | 将 XML 文件转换为 DOM 树 | 翻译官 (XML->DOM) |
| **BeanDefinitionParserDelegate** | 解析 DOM 元素具体的 Spring 标签 | 质检员 (DOM->Spring 配置) |
| **DefaultListableBeanFactory** | Spring 默认的 Bean 工厂实现 | 核心仓库 |
| **beanDefinitionMap** | 存放 BeanDefinition 的 Map | 档案柜 |

### 1.5. 总结与后续

通过上述流程，Spring 完成了**IoC 容器的初始化**。

* **当前状态**：容器中已经有了所有 Bean 的“图纸”（BeanDefinition），都在 `beanDefinitionMap` 里躺着。
* **注意**：此时，大部分 Bean **还没有被实例化**（没有 new 出来）。
* **下一步**：`refresh()` 方法的后续步骤（如 `finishBeanFactoryInitialization`）会遍历这个 Map，根据图纸将非懒加载的单例 Bean 实例化，并进行依赖注入。