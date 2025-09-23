---
source: "[[阅读中/文章列表/文章收藏/Java 日志通关（四） - Logback 介绍]]"
create: 2025-09-22
---

## 1. 文章核心内容分析

文章可以分为以下几个关键部分：

1.  **配置入口与方式**
    *   Logback 默认会查找 `logback-test.xml` 或 `logback.xml` 作为配置文件。
    *   当与 Spring Boot 集成时，推荐使用 `logback-spring.xml`。这个文件可以利用 Spring 的特性，例如：
        *   使用 `<springProfile>` 标签根据不同的环境（如开发、测试、生产）应用不同的日志配置。
        *   使用 `<springProperty>` 标签获取 Spring 应用的配置信息（如 `spring.application.name`）。

2.  **配置文件详解**
    *   **变量定义 (`<property>` 和 `<springProperty>`)**: 展示了如何定义和使用变量来简化配置，例如定义日志路径和日志格式。
    *   **输出器 (`<appender>`)**: 解释了 `appender` 的作用是负责日志的输出。文中重点介绍了两种常用的 `appender`：
        *   `RollingFileAppender`: 用于将日志写入文件，并根据大小和时间进行滚动归档，防止单个日志文件过大。
        *   `AsyncAppender`: 将日志写入操作异步化，避免因大量日志打印而阻塞业务线程，提升应用性能。
    *   **日志记录器 (`<logger>` 和 `<root>`)**:
        *   `<logger>` 用于为特定的包或类设置日志级别（如 `INFO`, `DEBUG`）。
        *   `<root>` 是所有 logger 的根配置，作为默认或兜底的日志级别设置。
        *   文章强调了 `additivity="false"` 的重要性，它可以防止日志被父级 logger 重复打印。

3.  **日志格式占位符**
    *   **Conversion Word**: 详细列举并解释了多种常用的日志格式占位符，如 `%logger`（记录器名）、`%message`（日志内容）、`%exception`（异常堆栈）、`%date`（时间）、`%thread`（线程名）等。
    *   **Format Modifiers**: 介绍了如何通过修饰符来控制日志输出的格式，例如 `%-5level` 可以让日志级别左对齐并占用固定的宽度，使日志更整齐易读。

4.  **高级应用与最佳实践**
    *   **Java API**: 提及了 Logback 提供了丰富的 Java API，可以实现更高级的定制功能，例如：
        *   自定义日志消息转换规则（如对象自动转 JSON）。
        *   对敏感信息（如手机号）进行脱敏处理。
        *   在程序运行时动态修改日志级别。
    *   **MDC 与 traceId**: 强调了一个重要的最佳实践：在分布式系统中，应该利用 OpenTelemetry 等工具自动将 `traceId` 放入 MDC (Mapped Diagnostic Context)，然后在日志格式中通过 `%X{trace_id}` 或 `%mdc{trace_id}` 来统一打印，而不是在每条日志代码中手动拼接 `traceId`。

## 2. 总结

这篇文章是一篇非常实用的 Logback 配置和使用指南。它不仅涵盖了基础的配置文件结构，还深入讲解了与 Spring Boot 集成的最佳实践、日志格式化的高级技巧以及通过 Java API 进行扩展的思路。对于希望规范和优化项目日志的 Java 开发者来说，这篇文章提供了清晰、全面且可操作的指导。