---
source: "[[阅读中/阅读总结/文章收藏/Java 日志通关（五） - 最佳实践|Java 日志通关（五） - 最佳实践]]"
create: 2025-09-22
---

## 1. 核心要点分析

文章从接口使用、日志内容、异常处理、性能开销等多个维度，提出了具体的建议和代码示例。

1.  **面向接口编程**：
    *   **建议**：总是使用 SLF4J 等日志接口层，而不是具体的实现（如 Logback）。
    *   **理由**：在提供三方库时，将日志实现标记为 `optional` 和 `runtime`，可以避免依赖冲突，让使用者自由选择日志框架。
        ```
        <dependency>
          <groupId>ch.qos.logback</groupId>
          <artifactId>logback-core</artifactId>
          <version>${logback.version}</version>
          <scope>runtime</scope>
          <optional>true</optional>
        </dependency>
        ```
2.  **避免无效的分隔线**：
    *   **建议**：不要打印如 `start` 这样的分隔线。
    *   **理由**：在并发或分布式环境中，日志会交错打印，分隔线无法起到有效分割作用。应使用关键字（如 `FooBarProcessor start`）来标记日志，便于 `grep` 或日志系统（如 SLS）过滤。

3.  **保证日志代码的健壮性**：
    *   **建议**：在记录日志前，对可能为 `null` 的对象进行空指针检查。
    *   **理由**：避免因为打印日志而抛出 `NullPointerException`，导致主流程中断。

4.  **优化 JSON 序列化**：
    *   **建议**：使用 Fastjson 的两个参数：
        *   `SerializerFeature.IgnoreErrorGetter`：忽略序列化时因调用 getter 方法抛出的异常。
        *   `SerializerFeature.IgnoreNonFieldGetter`：忽略没有实体字段对应的 getter 方法（如 `isError()`），避免在日志中产生无关的关键字。

5.  **正确打印异常堆栈**：
    *   **错误做法**：`log.error("exception={}", e);`，这只会打印异常的 `toString()` 信息，丢失堆栈。
    *   **正确做法**：`log.error("exception={}", e.getMessage(), e);`，将异常对象作为最后一个独立参数传入，确保堆栈信息被完整记录。

6.  **限制日志长度**：
    *   **建议**：通过 Logback 配置限制单条日志消息的最大长度（如 `%.2000message`）和异常堆栈的层级（如 `%exception{50}`）。
    *   **理由**：避免因打印过大的对象导致性能问题和日志文件臃肿。

7.  **将堆栈合并为一行**：
    *   **建议**：使用 Logback 的 `%replace` 功能将异常堆栈中的换行符替换为空格，并配合 `%nopex` 避免重复打印。
    *   **理由**：便于使用管道符（`|`）进行多层 `grep` 筛选。

8.  **避免使用高消耗的日志模式**：
    *   **建议**：不建议在 Logback 配置中使用 `%method` (方法名) 和 `%line` (行号)。
    *   **理由**：获取这些信息需要遍历当前堆栈，性能开销较大。建议将方法名等上下文信息硬编码到日志内容中。

9.  **不要输出到控制台**：
    *   **建议**：线上环境不要使用 `ConsoleAppender`。
    *   **理由**：生产环境无人看守控制台，输出日志是无效行为且浪费资源。本地调试应使用断点或日志文件。

10. **废弃自定义的 `LogUtil`**：
    *   **建议**：直接使用 SLF4J 的 API，而不是封装一个 `LogUtil` 工具类。
    *   **理由**：大部分 `LogUtil` 实现的功能（如拼接、转 JSON、追加 traceId 等）都可以通过正确配置 Logback（如 MDC、Layout）来实现，无需重复造轮子。

11. **遵循开发规约**：
    *   **建议**：熟读《阿里巴巴 Java 开发手册》中的《日志规约》。

12. **注意日志格式细节**：
    *   **建议**：为了提高可读性，日志前缀的符号应与日志内容的格式区分开。
        *   如果内容是 `key=value` 格式 (如 Lombok 的 `@ToString`)，前缀用冒号：`log.info("foo:{}", foo);`
        *   如果内容是 JSON 格式 (`"key":"value"`)，前缀用等号：`log.info("foo={}", JSON.toJSONString(foo));`

## 2. 总结

这篇笔记是一份非常实用的 Java 日志记录指南，涵盖了从基础规范到高级配置的方方面面。核心思想是：**让日志清晰、健壮、高效且易于管理**。通过遵循这些实践，可以显著提升应用的线上可维护性和问题排查效率。