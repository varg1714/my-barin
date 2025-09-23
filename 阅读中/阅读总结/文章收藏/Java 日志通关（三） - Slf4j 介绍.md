---
source: "[[阅读中/文章列表/文章收藏/Java 日志通关（三） - Slf4j 介绍]]"
create: 2025-09-22
---

## 1. Slf4j 核心概念与使用笔记

### 1.1. Slf4j 简介

Slf4j (Simple Logging Facade for Java) 是一个日志**门面**（或称**抽象层**），它本身不提供具体的日志实现，而是提供一套标准的 API 接口。开发者在代码中面向 Slf4j API 编程，在运行时可以灵活地搭配不同的日志实现框架（如 Logback, Log4j2, java.util.logging 等），实现了代码与具体日志框架的解耦。

### 1.2. 获取 Logger 实例

获取 `Logger` 实例是使用 Slf4j 的第一步，主要有两种方式：

1. **通过工厂函数 `LoggerFactory.getLogger()`**
    *  **传入 Class 对象**：这是最推荐的方式，它会自动使用该类的全限定名作为 Logger 的名字（LoggerName）。

        ```java
        private static final Logger log = LoggerFactory.getLogger(ExampleService.class);
        // 等价于: LoggerFactory.getLogger("com.example.service.ExampleService");
        ```

    *  **传入字符串**：可以自定义 Logger 的名字，方便对一组 Logger 进行统一配置。

        ```java
        private static final Logger log = LoggerFactory.getLogger("service");
        ```

2. **通过 Lombok 注解**
    Lombok 提供了便捷的注解来自动生成 `Logger` 实例，简化了样板代码。
    *  `@Slf4j`: 自动生成一个名为 `log` 的静态 `Logger` 实例，其 LoggerName 为当前类的全限定名。
    *  `@Slf4j(topic = "service")`: 可以指定自定义的 LoggerName。

### 1.3. 日志级别

Slf4j 定义了五个日志级别，按严重程度由低到高排列：

*  **TRACE**: 用于追踪详细的程序执行路径，如方法进入和退出。
*  **DEBUG**: 用于记录调试信息，如方法的输入输出参数。
*  **INFO**: 默认级别，用于记录应用运行过程中的关键信息、重要节点。
*  **WARN**: 记录预期之外的、但程序仍可继续运行的警告情况。
*  **ERROR**: 记录导致程序出错的异常信息，通常在 `catch` 块中使用。

**核心思想**：通过在日志实现层（如 Logback）配置一个输出阈值（如 `INFO`），只有高于或等于该阈值的日志才会被输出。这使得我们可以在开发时输出 `DEBUG` 日志，而在生产环境只保留 `INFO` 及以上级别的日志。

### 1.4. 日志打印接口

1. **占位符 `{}`**
    Slf4j 推荐使用占位符 `{}` 的方式来拼接日志，而不是手动用 `+` 连接字符串。
    *  **优点**：只有当该日志级别被启用时，才会真正执行字符串的拼接操作，避免了不必要的性能开销。
    *  **示例**：

        ```java
        // 推荐
        log.debug("querySomething request={}", request);

        // 不推荐 (无论日志级别是否开启，都会执行字符串拼接)
        log.debug("querySomething request=" + request);
        ```

2. **`isXxxEnabled()` 判断**
    对于日志内容本身需要复杂计算或远程调用才能获取的情况，建议先使用 `isXxxEnabled()` 方法进行判断，避免资源浪费。

    ```java
    if (log.isInfoEnabled()) {
        // toJSONString() 可能非常耗时
        log.info("result={}", JSON.toJSONString(bigObject));
    }
    ```

### 1.5. Marker 深度解析：为日志打上“业务标签”

`Marker` 是 Slf4j 中一个非常强大但容易被忽视的功能。如果说**日志级别 (Level) 是一维的，描述事件的严重程度**，那么 **Marker 就是多维的，用于描述事件的业务分类**。

**1. 核心作用**

`Marker` 可以理解为给日志打上的**“标签” (Tag)**。它与日志级别正交，让你能够基于**业务含义**而不是简单的严重级别来处理日志。

**2. 为什么需要 Marker？**

很多场景仅靠日志级别无法满足需求：

*  **安全审计**：需要将所有用户登录、权限变更的操作（可能是 `INFO` 或 `WARN` 级别）记录到独立的 `security.log`。
*  **业务追踪**：需要将所有“支付流程”相关的日志（跨越多个级别）筛选出来，用于问题排查。
*  **触发特殊动作**：当某个特定的 `ERROR`（如支付网关连续宕机）发生时，需要触发邮件告警，而其他 `ERROR` 则不需要。

**3. 主要应用场景**

*  **日志路由与过滤 (Log Routing & Filtering)**
    这是最核心的用途。通过为不同业务日志打上特定 `Marker`，可以在日志配置文件（如 `logback.xml`）中设置 `MarkerFilter`，将这些日志精准地输出到不同的文件或目的地。

    *  **代码示例**：

        ```java
        Marker securityMarker = MarkerFactory.getMarker("SECURITY");
        log.info(securityMarker, "User '{}' logged in successfully.", userId);
        ```

    *  **配置思想**：在 `logback.xml` 中配置一个 Appender，并为其添加 `<filter class="ch.qos.logback.classic.filter.MarkerFilter">`，设置 `<marker>SECURITY</marker>`，并指定 `<onMatch>ACCEPT</onMatch>` 和 `<onMismatch>DENY</onMismatch>`。
*  **触发特殊动作 (Triggering Special Actions)**
    可以定义一个特殊的 `Marker`（如 `NOTIFY_ADMIN`），并配置一个专门的 Appender（如 `SMTPAppender` 用于发邮件），使其只处理带有此 `Marker` 的日志。

**4. Marker 与 Level 对比总结**

| 特性 | 日志级别 (Level) | Marker |
| :--- | :--- | :--- |
| **目的** | 描述事件的**严重程度** | 描述事件的**业务分类**或**处理方式** |
| **维度** | 一维（TRACE < ... < ERROR） | 多维（可自定义任意多个，如 SECURITY, PAYMENT） |
| **结构** | 线性、有序 | 树状结构（Marker 可包含子 Marker），更灵活 |
| **主要用途** | 控制日志的**输出阈值** | 实现日志的**精准路由、过滤、触发特殊动作** |

### 1.6. MDC (Mapped Diagnostic Context)

MDC 提供了一个**线程安全**的、用于存放诊断上下文信息的容器，可以看作是一个 `ThreadLocal` 的 `Map<String, String>`。

*  **核心用途**：在日志中植入贯穿整个请求处理链路的上下文信息，最典型的应用就是**分布式链路追踪**中的 `traceId`。
*  **使用方法**：

    ```java
    // 在请求开始时（如 Filter 或 Interceptor 中）放入 traceId
    MDC.put("traceId", "some-unique-trace-id");

    // 在代码任意位置打印日志，日志实现层会自动从 MDC 中获取并打印 traceId
    log.info("Processing user request.");

    // 在请求结束时清理，防止内存泄漏
    MDC.remove("traceId"); // 或 MDC.clear()
    ```

---

### 1.7. Fluent API (链式调用)

Slf4j 从 2.0 版本开始引入了 Fluent API，提供了一种更灵活、更具可读性的日志记录方式。

*  **特点**：通过链式调用设置日志的各个属性（级别、Marker、异常、消息、参数等），最后调用 `.log()` 方法完成记录。
*  **示例对比**：

    ```java
    Marker marker = MarkerFactory.getMarker("foobar");
    Exception e = new RuntimeException();

    // 传统方式
    log.info(marker, "request a={}, b={}", 1, 2, e);

    // Fluent API 方式
    log.atInfo()
       .addMarker(marker)
       .setCause(e)
       .log("request a={}, b={}", 1, 2);
    ```

*  **优势**：
    *  **支持多个 Marker**：`addMarker()` 可以多次调用。
    *  **惰性求值**：`addArgument(() -> expensiveMethod())` 支持传入 `Supplier`，参数的计算会被推迟到日志确定要被记录时。
    *  **键值对记录**：`setKeyValue("key", value)` 提供了结构化日志的记录方式。