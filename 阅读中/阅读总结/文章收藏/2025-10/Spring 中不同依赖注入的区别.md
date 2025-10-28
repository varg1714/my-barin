---
source:
  - "[[又写 bug 了？我的 bean 配置居然没生效？]]"
create: 2025-10-27
---

Spring 中存在几种核心的依赖注入（DI）方式：`@Autowired`、`@Resource`、`@Bean` 方法参数注入，以及传统的 XML 配置，它们的主要差异在于 **来源**、**默认的注入策略（按类型 vs 按名称）**，以及 **处理歧义的方式**。

## 1. `@Autowired` (Spring 特有)

这是 Spring 框架中最常用、最核心的注解之一。

* **来源**：Spring 框架自身提供。
* **核心注入策略**：**按类型 (By Type)**。
    * Spring 会在容器中寻找与需要注入的字段或参数**类型完全匹配**的 Bean。
    * 如果只找到一个匹配的 Bean，注入成功。
* **处理歧义（当找到多个同类型 Bean 时）**：
    1.  **`@Qualifier` 优先**：如果同时使用了 `@Qualifier("beanName")`，Spring 会忽略类型，直接按名称 (`beanName`) 查找并注入，这是最精确的控制方式。
    2.  **匹配字段/参数名**：如果没有 `@Qualifier`，Spring 会尝试查找一个 Bean，其名称（ID）与**字段名**或**参数名**相同。
    3.  **`@Primary` 注解**：如果以上两种方式都无法确定，Spring 会寻找被 `@Primary` 注解标记的 Bean。`@Primary` 表示“首选”，在有多个候选者时会被优先选择。
    4.  **抛出异常**：如果以上所有规则都无法解决歧义（例如，多个同类型 Bean，没有 `@Qualifier`，没有 `@Primary`，且没有一个 Bean 的名称与字段名匹配），Spring 将会抛出 `NoUniqueBeanDefinitionException` 异常。
* **适用位置**：构造器、字段、Setter 方法。
    * **构造器注入**是 Spring 官方推荐的方式，因为它可以保证依赖在对象实例化时就已准备好，且对象是不可变的。
* **总结**：`@Autowired` 非常强大和灵活，但它的“按类型”优先策略是其优点也是其潜在的坑点。当容器中存在多个同类型 Bean 时，必须有额外的机制（如 `@Qualifier`）来保证注入的确定性。

## 2. `@Resource` (Java 标准)

这是 Java 的标准注解（JSR-250），Spring 提供了对它的支持。

* **来源**：Java EE / Jakarta EE 标准 (在 `javax.annotation` 或 `jakarta.annotation` 包中)。
* **核心注入策略**：**按名称 (By Name)**。
    * 它的注入顺序非常明确，不易产生混淆。
* **注入顺序**：
    1.  **按 `name` 属性查找**：如果使用了 `@Resource(name = "beanName")`，它会直接按指定的名称 `beanName` 查找 Bean。
    2.  **按字段/方法名查找**：如果没有指定 `name` 属性，它会使用**字段名**或 Setter 方法名（去掉 `set` 并将首字母小写后的名字）作为 Bean 的名称进行查找。
    3.  **备用策略：按类型查找**：**只有在按名称完全找不到任何 Bean 的情况下**，`@Resource` 才会回退到**按类型**查找。
    4.  **处理类型歧义**：如果在按类型查找时发现多个候选 Bean，它也会参考 `@Primary` 注解。如果仍无法确定，则会抛出异常。
* **适用位置**：字段、Setter 方法（不推荐用于构造器）。
* **总结**：`@Resource` 的行为更加确定，因为它优先“按名称”注入。这使得它在很多情况下比 `@Autowired` 更不容易出错，因为开发者通常对 Bean 的名称有明确的预期。它的主要缺点是不如 `@Autowired` 在构造器注入方面那么方便和流行。

## 3. `@Bean` 方法参数注入 (在 `@Configuration` 类中)

这并非一个独立的注解，而是 `@Bean` 注解所在方法的一种特定注入场景。

* **来源**：Spring 框架。
* **核心行为**：其行为与 **`@Autowired` 完全一致**。
    * 当你定义一个 `@Bean` 方法，并为其添加参数时，Spring 会自动为这些参数寻找并注入依赖。
    * `public TairAccessor tmgXxxTairAccessor(TairManager tmgXxxTairManager)`
    * 在这个例子中，Spring 会在容器中**按类型**查找一个 `TairManager` 类型的 Bean 来作为参数传入。
* **处理歧义**：
    * 这正是原文中问题的根源。因为默认按类型注入，应用 B 中另一个被标记为 `@Primary` 的 `TairManager` Bean 被错误地注入了。
    * 解决方法和 `@Autowired` 一样：使用 `@Qualifier("tmgXxxTairManager")` 来精确指定要注入的 Bean 的名称。
* **总结**：可以简单地理解为，`@Bean` 方法的参数列表上隐式地执行了 `@Autowired` 操作。因此，所有 `@Autowired` 的规则和潜在问题在这里都同样适用。

## 4. XML 配置 (`<bean>`)

这是 Spring 最原始、最经典的配置方式。

* **来源**：Spring 框架。
* **核心注入策略**：**完全显式和确定性**。
    * 在 XML 中，你必须明确地指定所有依赖关系，不存在“自动”或“猜测”的过程。
* **注入方式**：
    * **Setter 注入**：使用 `<property>` 标签。

        ```xml
        <property name="tairManager" ref="tmgXxxTairManager"/>
        ```

        这里的 `ref` 属性直接引用另一个 Bean 的 `id` 或 `name`。这是**严格的按名称注入**，绝不会有歧义。

    * **构造器注入**：使用 `<constructor-arg>` 标签。

        ```xml
        <constructor-arg ref="tmgXxxTairManager"/>
        ```

        同样通过 `ref` 按名称引用。

* **优点**：
    * **确定性**：配置与代码分离，依赖关系一目了然，绝无歧义。
    * **解耦**：修改依赖关系无需重新编译 Java 代码。
* **缺点**：
    * **繁琐**：对于大型项目，XML 文件会变得非常庞大和难以维护。
    * **类型不安全**：IDE 的重构支持较弱，如果类名或属性名改变，XML 配置需要手动同步。
* **总结**：XML 配置提供了最强的控制力和确定性，但牺牲了开发的便捷性。在现代 Spring Boot 应用中已不常用，但在一些遗留系统或需要将配置与业务逻辑严格分离的场景下仍有价值。

## 5. 总结对比表

| 特性 | @Autowired | @Resource | @Bean 方法参数 | XML 配置 (`<bean>`) |
| :--- | :--- | :--- | :--- | :--- |
| **来源** | Spring 框架 | Java 标准 (JSR-250) | Spring 框架 | Spring 框架 (经典方式) |
| **主要策略** | **按类型 (By Type)** | **按名称 (By Name)** | **按类型 (By Type)** | **显式按名称/ID (Explicit By Name)** |
| **备用策略** | 按字段/参数名匹配 | 按类型匹配 | 按参数名匹配 | 无 (必须显式指定) |
| **解决歧义** | `@Qualifier`, `@Primary`, 字段/参数名 | `name` 属性, (类型匹配时可用 `@Primary`) | `@Qualifier`, `@Primary`, 参数名 | 无歧义，通过 `ref` 精确引用 |
| **适用位置** | 构造器、字段、方法 | 字段、方法 | `@Bean` 方法的参数 | 构造器 (`<constructor-arg>`), Setter (`<property>`) |
| **推荐用法** | 构造器注入，结合 `@Qualifier` 处理歧义 | 字段注入，当名称比类型更重要时 | 在 `@Configuration` 类中定义 Bean 间依赖 | 遗留项目或需要将配置与代码完全分离的场景 |