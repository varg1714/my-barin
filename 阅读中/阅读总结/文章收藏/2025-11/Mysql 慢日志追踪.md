---
source:
  - "[[哈罗面试：没开 sql 慢查询日志，怎么发现慢 sql？]]"
create: 2025-11-23
---

## 1. 基础概念：什么是慢 SQL？

* **定义**：执行时长超过 `long_query_time` 阈值的 SQL 语句。
* **阈值标准**：
    * **MySQL 默认**：10 秒（`long_query_time = 10`）。
    * **通用业务建议**：1-2 秒（Web 页面通常要求 < 2 秒）。
    * **核心高频业务**：建议 < 500ms（如支付、下单）。
    * **后台低频任务**：可放宽至 1-20 秒。

## 2. 常规手段：慢查询日志配置（复习）

虽然题目假设未开启，但需了解相关参数：

* `slow_query_log`：开关（1=开，0=关）。
* `long_query_time`：时间阈值。
* `log_queries_not_using_indexes`：记录未使用索引的查询。
* `log_output`：日志存储方式。建议设为 `FILE`（性能好），设为 `TABLE` 会占用数据库资源。

## 3. 核心问题：没开慢查询日志，如何发现慢 SQL？

这是文章的重点，提供了 4 种替代方案，分为**实时定位**和**历史追溯**两个维度。

### 3.1. 方法一：抓现行——实时监控面板

利用外部监控工具的可视化能力。

* **开源方案**：Prometheus + Grafana（配置 MySQL Exporter 模板，查看 Top SQL 或慢查询计数）。
* **云厂商工具**：阿里云 RDS、腾讯云 CDB 等自带的“性能洞察”或“实时性能监控”。

### 3.2. 方法二：抓现行——数据库实时连接 (`processlist`)

当业务反馈卡顿或 CPU 飙升时，直接查询数据库当前正在执行的线程。

* **命令**：
    * 简单查看：`show processlist;`
    * **筛选查看（推荐）**：

        ```sql
        select * from information_schema.processlist where command != 'Sleep';
        ```

* **关键字段分析**：
    * `Time`：执行时间。若 > 3s 需警惕。
    * `State`：当前状态。重点关注 `Sending data`（数据传输/扫描）、`Locked`（锁等待）。
    * `Info`：具体的 SQL 语句。

### 3.3. 方法三：应用侧——日志与链路追踪

从业务侧反向定位。

* **链路追踪（APM）**：使用 SkyWalking、Pinpoint 等工具，查看调用链中 Database 节点的耗时。
* **应用日志**：检查 Java/Python 等应用日志中记录的接口响应时间，结合时间点推断 SQL。

### 3.4. 方法四：查历史——利用 `performance_schema`

即使没开慢日志，MySQL 的性能模式（Performance Schema）通常是开启的，它会记录内存中的执行统计。

* **场景 A：查找执行时间长的历史 SQL**
    查询 `events_statements_history` 表（注意：`timer_wait` 单位通常是皮秒，需换算）：

    ```sql
    SELECT sql_text, timer_wait/1000000000 AS exec_time_sec
    FROM performance_schema.events_statements_history
    WHERE timer_wait/1000000000 > 3
    ORDER BY exec_time_sec DESC;
    ```

* **场景 B：查找全表扫描的 SQL（潜在慢 SQL）**
    通过扫描行数 `rows_examined` 远大于返回行数的情况来定位：

    ```sql
    SELECT sql_text, rows_examined
    FROM performance_schema.events_statements_history
    WHERE rows_examined > 10000
      AND (sql_text LIKE 'select%' OR sql_text LIKE 'update%' OR sql_text LIKE 'delete%')
    ORDER BY rows_examined DESC;
    ```

## 4. 慢 SQL 分析工具（定位到 SQL 后的步骤）

一旦找到了嫌疑 SQL，需要使用以下工具进行分析：

1. **EXPLAIN**：查看执行计划（索引命中情况、扫描类型）。
2. **OPTIMIZER_TRACE**：查看优化器如何选择执行计划。
3. **PROFILE**：查看 SQL 执行过程中的资源消耗详情。

### 4.1. 💡 总结与评价

* **文章价值**：该文章针对“生产环境未开启慢日志”这一高频痛点提供了切实可行的 SQL 层面解决方案，特别是利用 `information_schema.processlist` 和 `performance_schema` 进行排查的方法，是 DBA 和高级开发人员必须掌握的技能。
* **不足之处**：文章篇幅中有约 40% 为营销推广内容，阅读时需注意甄别，直接关注技术实现部分即可。