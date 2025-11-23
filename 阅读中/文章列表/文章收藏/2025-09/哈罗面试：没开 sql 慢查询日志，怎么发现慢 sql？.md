---
source: https://mp.weixin.qq.com/s/yecb_gpRU0i6vnENK2U0rA
create: 2025-09-11 21:11
read: true
knowledge: true
knowledge-date: 2025-11-23
tags:
  - Mysql
  - 系统运维
summary: "[[Mysql 慢日志追踪]]"
---
FSAC 未来超级架构师

架构师总动员  
实现架构转型，再无中年危机

# 尼恩说在前面：

在 40 岁老架构师 尼恩的**读者交流群** (50+) 中，最近有小伙伴拿到了一线互联网企业如得物、阿里、滴滴、极兔、有赞、shein 希音、shopee、百度、网易的面试资格，遇到很多很重要的面试题：

MySQL 的慢查询、如何监控、如何排查？

你知道如何排查慢 sql 嘛?

你知道如何对 mysql 进行优化嘛?

mysql 慢查询如何定位分析？哪些情况会导致慢查询？

没开 sql 慢查询日志，怎么发现慢 sql？

前几天 小伙伴面试 哈罗，遇到了这个问题。但是由于 没有回答好，导致面试挂了。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WNmfIJzMI8dicGNC2hlYhIhWhofesyACUnwB2LFYSWqlFfMrTuU4iaerovTOw6ibKtZ8H8fsmJE3LuoQ/640?from=appmsg&watermark=1#imgIndex=0)

小伙伴面试完了之后，来求助尼恩。

那么，遇到 这个问题，该如何才能回答得很漂亮，才能 让面试官刮目相看、口水直流。

所以，尼恩给大家做一下系统化、体系化的梳理，使得大家内力猛增，可以充分展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer 直提”。

当然，这道面试题，以及参考答案，也会收入咱们的 《[尼恩 Java 面试宝典](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》V145 版本 PDF 集群，供后面的小伙伴参考，提升大家的 3 高 架构、设计、开发水平。

最新《尼恩 架构笔记》《尼恩高并发三部曲》《尼恩 Java 面试宝典》的 PDF，请关注本公众号【技术自由圈】获取，后台回复：领电子书

  

## 一：什么是 慢 SQL（慢查询） 日志？

MySQL 的慢查询日志是 MySQL 提供的一种日志记录。

MySQL 的慢查询日志 ，主要 用来记录在 MySQL 中响应的时间超过 **执行时长阈值**的语句， **执行时长阈值** 通过 一个 参数 long_query_time（默认是 10 秒）配置。

一个 SQL 的执行，只要 超过 这个 long_query_time 时长， 就会被判为慢查询， 会被记录到慢查询日志中。

## 二： SQL 到底 执行 多长时间，才为慢 SQL ？

慢 SQL 时间 长度是   “执行耗时超出业务或数据库的合理阈值”，但具体定义需结合**数据库类型、业务场景、硬件配置**综合判断，无绝对统一标准。

慢 SQL 时间 长度 通常分为 通用阈值 和 业务定制阈值 两类：

第一个维度：通用参考阈值（以 MySQL 为例）

数据库层面默认有基础标准，最典型的是 MySQL 的`long_query_time`参数（慢查询日志的核心触发条件）：

**默认值：****10 秒 （即执行时间≥10 秒的 SQL 会被标记为慢 SQL）；**

**实际场景：**多数业务会将阈值下调至 **1-2 秒**（因 10 秒远超用户可接受的响应时间，比如 Web 页面加载通常要求 < 2 秒，接口响应 < 500ms）。

第二个维度：业务定制阈值（更关键）

通用阈值仅作参考，真正的慢 SQL 定义需贴合业务场景，例如：

*   高频核心业务（如电商下单、支付接口）：阈值可能设为 **500ms**（若 SQL 执行 1 秒，会直接导致用户卡顿）；
    
*   低频非核心任务（如每日数据统计）：阈值可放宽至 **1 秒 - 20s**（后台执行，不影响用户体验）；
    

## 三：如何设置 慢 SQL 相关参数？

下面是一些关键参数设置：

*   slow_query_log：控制慢查询日志的开关。
    

设置成 1，日志就开启；设置成 0，日志就关闭。

*   log-slow-queries（旧版本 5.6 以下使用）和 slow-query-log-file（5.6 及以上版本使用）：
    

这两个参数用来指定慢查询日志的保存位置。如果不设置，系统会自动生成一个默认文件，名字是 “主机名 - slow.log”。

*   long_query_time：慢查询阈值：
    

它决定了什么样的查询算慢查询，只要 SQL 语句执行时间超过这里设置的时长，就会被记录到日志里。

*   log_queries_not_using_indexes：
    

这是个可选参数，打开后，**没命中索引的查询语句**也会被记录下来。

*   log_output：用来选择日志保存方式。
    

有 FILE（快） , TABLE（满） 两种方式保持日志。设置成 FILE，日志就存到文件里，这是默认方式；

设置成  TABLE，日志会存到数据库的 mysql.slow_log 表里。

也可以同时用两种方式，写成 FILE,TABLE。不过，存到数据库表比存到文件更费资源，如果既想开慢查询日志，又想数据库跑得快，建议优先把日志存到文件里。

slow_query_log 的 参数查看：

```
mysql> show VARIABLES like '%slow_query_log%';+---------------------+-----------------------------------------------------------+| Variable_name       | Value                                                     |+---------------------+-----------------------------------------------------------+| slow_query_log      | OFF                                                       || slow_query_log_file | d:\java\mysql-5.7.28-winx64\data\DEEP-2020AEBKQR-slow.log |+---------------------+-----------------------------------------------------------+2 rows in set, 1 warning (0.00 sec)
```

long_query_time 的 参数查看：

```
mysql> show VARIABLES like 'long_query_time%';+-----------------+-----------+| Variable_name   | Value     |+-----------------+-----------+| long_query_time | 10.000000 |+-----------------+-----------+1 row in set, 1 warning (0.00 sec)
```

设置启用慢查询日志：

```
SET GLOBAL slow_query_log = 'ON';SET GLOBAL long_query_time = 1;
```

注意：修改全局参数后，当前连接会话仍使用旧值，需新建会话才会生效，需要重新登录

查看修改后参数：

```
mysql> show VARIABLES like '%slow_query_log%';+---------------------+-----------------------------------------------------------+| Variable_name       | Value                                                     |+---------------------+-----------------------------------------------------------+| slow_query_log      | ON                                                        || slow_query_log_file | d:\java\mysql-5.7.28-winx64\data\DEEP-2020AEBKQR-slow.log |+---------------------+-----------------------------------------------------------+2 rows in set, 1 warning (0.00 sec)mysql> show VARIABLES like 'long_query_time%';+-----------------+----------+| Variable_name   | Value    |+-----------------+----------+| long_query_time | 1.000000 |+-----------------+----------+1 row in set, 1 warning (0.00 sec)
```

## 四：3 大 慢 SQL 分析工具

慢查询分析一般分为三个步骤

1、 EXPLAIN2、 OPTIMIZER_TRACE3、 PROFILE

上面三大工具的 具体使用方法，请参见尼恩之前的博客文章：

[凌晨 2 点，报警群炸了：一条 sql 执行 200 秒！搞定之后，我总结了一个慢 SQL 查询、定位分析解决的完整套路](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247505363&idx=1&sn=ada0c821646c7d41eb4059e14fbaa92f&scene=21#wechat_redirect)

## 五、没开慢查询日志，怎么发现慢 SQL？ （4 个方法）

慢查询日志是 “事后追溯” 的核心工具，但未开启时，可通过「实时监控」「资源排查」「业务反馈」三类手段主动发现慢 SQL，覆盖**实时问题定位**和**历史问题追溯**场景。

## 5.1 抓现行： 实时监控面板

首先借助 直观的实时监控， 借助可视化工具 ，监控慢 sql：

*   **开源工具**：Prometheus + Grafana（配置数据库监控模板，可实时查看 “慢查询数”“SQL 执行耗时 TOP10”） ；
    
*   **云厂商工具**：阿里云 RDS 的「实时性能监控」、腾讯云 CDB 的「性能洞察」，可直接查看当前执行的慢 SQL 列表及资源占用。
    

## 5.2  抓现行： 实时 DB 定位

适用于 “业务反馈卡顿”“数据库 CPU/IO 飙升” 等紧急场景，直接查看数据库当前运行的 SQL，定位耗时久的语句。

用到命令：

*   `show processlist`
    
*   `` `information_schema.processlist``
    

这是最常用的实时排查命令，能查看当前所有数据库连接的执行状态、耗时、SQL 内容。

*   **基础命令**：`show processlist;`（仅显示前 100 条，适合快速查看）；
    
*   **完整命令**：`select * from information_schema.processlist where command != 'Sleep';`（显示所有非空闲连接，便于筛选）。
    

**关键字段解读**（重点关注 3 个字段）：

<table><thead><tr><td><span><strong><span leaf="">字段名</span></strong></span></td><td><span><strong><span leaf="">含义</span></strong></span></td></tr></thead><tbody><tr><td><section><span><code><span leaf="">Time</span></code></span></section></td><td><section><span><span leaf="">SQL 已执行的时间（单位：秒），若远超阈值（如 &gt; 3 秒），大概率是慢 SQL</span></span></section></td></tr><tr><td><section><span><code><span leaf="">State</span></code></span></section></td><td><section><span><span leaf="">SQL 当前的执行状态，如</span><code><span leaf="">Sending data</span></code><span leaf="">（数据传输，耗时可能长）、</span><code><span leaf="">Locked</span></code><span leaf="">（锁等待）</span></span></section></td></tr><tr><td><section><span><code><span leaf="">Info</span></code></span></section></td><td><section><span><span leaf="">完整的 SQL 语句（若过长，需用</span><code><span leaf="">show full processlist;</span></code><span leaf="">查看完整内容）</span></span></section></td></tr></tbody></table>

**示例**：

若发现`Time=15`（执行了 15 秒）、`State=Sending data`、`Info=select * from order where create_time < '2024-01-01'`，则该 SQL 是典型的慢 SQL。

## 5.3.  日志与链路追踪：从 “业务卡顿” 定位 SQL

业务层的异常反馈是慢 SQL 的重要线索，通过业务日志和链路追踪工具可反向找到对应的 SQL：

**步骤 1**：收集业务反馈（如 “用户提交订单时卡顿了 5 秒”），确定异常时间（如 2024-05-20 14:30）和业务接口（如`/order/submit`）；

**步骤 2**：查看应用日志（如 Java 的 Logback、Python 的 logging），找到该接口对应的 SQL 执行记录（若应用打印了 SQL 耗时，直接定位）；

**步骤 3**：若应用未打印 SQL，用链路追踪工具（如 SkyWalking ）查看接口调用链，找到 “数据库调用” 节点，查看耗时（如 “MySQL 调用耗时 4.5 秒”），并提取对应的 SQL 语句。

## 5.4 历史追溯： 查历史执行记录 发现 慢 SQL

若慢 SQL 已执行完毕（如夜间批量任务），可以 通过 “资源使用记录”“业务日志” 间接追溯。

### 5.4 .1.  分析 历史执行记录表中 的 “执行统计信息”

部分数据库会留存近期的 SQL 执行统计信息，可通过系统表查询：

```
performance_schema.events_statements_history
```

（默认保留 1000 条历史 SQL，可通过 setup_consumers 调整保留数）：

```
-- 筛选执行时间>3秒的历史SQL（单位：微秒，需转换为秒）select sql_text, timer_wait/1000000000 as exec_time from performance_schema.events_statements_history where timer_wait/1000000000 > 3 order by exec_time desc;
```

### 5.4 .2. 分析  历史执行记录  “全表扫描 SQL”

还可主动扫描数据库中 “可能成为慢 SQL” 的语句（如无索引、全表扫描的 SQL）。

比如，全表扫描（`type=ALL`）是慢 SQL 的主要诱因，可通过`explain`分析 SQL 执行计划，或直接查询系统表筛选：

```
-- 筛选出全表扫描且行数>1万的SQL（潜在慢SQL）select sql_text, rows_examined from performance_schema.events_statements_history where rows_examined > 10000   and (sql_text like 'select%' or sql_text like 'update%' or sql_text like 'delete%')order by rows_examined desc;
```

其中：

*   `rows_examined`：SQL 扫描的行数，若远大于实际返回行数（`rows_sent`），说明存在低效扫描。
    

## 说在最后：有问题找老架构取经‍

只要按照上面的 尼恩团队梳理的 方案去作答， 你的答案不是 100 分，而是 120 分。  面试官一定是 心满意足， 五体投地。

按照尼恩的梳理，进行 深度回答，可以充分展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer 直提”。

在面试之前，建议大家系统化的刷一波 5000 页《[尼恩 Java 面试宝典 PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》，里边有大量的大厂真题、面试难题、架构难题。

很多小伙伴刷完后， 吊打面试官， 大厂横着走。

在刷题过程中，如果有啥问题，大家可以来 找 40 岁老架构师尼恩交流。

另外，如果没有面试机会， 可以找尼恩来改简历、做帮扶。前段时间，**[指导一个小伙 暴涨 200%（涨 2 倍），29 岁 / 7 年 / 双非一本 ， 从 13K 一次涨到 37K ，逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486433&idx=1&sn=0f630e8fc7620b3bf0e054827b48d91e&scene=21#wechat_redirect)。**

狠狠卷，实现 “offer 自由” 很容易的， 一个武汉的跟着尼恩卷了 2 年的小伙伴， 在极度严寒 / 痛苦被裁的环境下， offer 拿到手软， 实现真正的 “offer 自由” 。

## Java+AI  弯道超车，  转架构 捷径 

 [会 AI 的程序员，工资暴涨 50%！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486827&idx=1&sn=db3f268c6752ddb074a98c7b13a84011&scene=21#wechat_redirect)

  

[涨薪传奇： 18k->38K , 单月暴 20K，32 岁小伙伴 2 个月时间年薪 翻 1.5 倍 ，一步登天 + 逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486960&idx=1&sn=f57253a448694c32e834207381c42284&scene=21#wechat_redirect)

  

[低学历 传奇：29 岁 6 年专套本，受够了外包，狠卷 3 个月逆袭大厂 涨 1 倍， 逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486945&idx=1&sn=f6ff6231ccf3585624161c2ef1f50fdf&scene=21#wechat_redirect)

  

[极速上岸： 被裁 后， 8 天 拿下 京东，狠涨 一倍 年薪 48W， 小伙伴 就是 做对了一件事](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486913&idx=1&sn=edd6774e9d17c39327e8dcb95fdd68d9&scene=21#wechat_redirect)

  

[外包 + 二本 进 美团： 26 岁小 2 本 一步登天， 进了顶奢大厂（ 美团） ， 太爽了](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486904&idx=1&sn=e7c878f99ed101ebfbd1d80ec0e07401&scene=21#wechat_redirect)

  

[超牛的 Java+Al 双栖架构： 34 岁无路可走，一个月翻盘，拿 3 个架构 offer，靠  Java+Al   逆天改命！！！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486848&idx=1&sn=ff43d058271b0801f84aba3f3d957855&scene=21#wechat_redirect)

  

[Java+Al 逆袭 1： 34 岁无路可走，一个月翻盘，拿 3 个架构 offer，靠  Java+Al   逆天改命！！！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486848&idx=1&sn=ff43d058271b0801f84aba3f3d957855&scene=21#wechat_redirect)

  

java+AI 逆袭 2：[：3 年 程序媛 被裁， 25W-》40W 上岸， 逆涨 60%。 Java+AI 太神了， 架构小白 2 个月逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486858&idx=1&sn=9cd122c18d166433b43d0952d3a7a7a8&scene=21#wechat_redirect)

  

  

[Java+AI 逆袭 3 ： 36 岁 / 失业 7 个月 / 彻底绝望 。狠卷 3 个月 Java+AI ，终于逆风翻盘，顺利 上岸](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486870&idx=1&sn=990a540a155cdb8254ebce6c75816882&scene=21#wechat_redirect)

[Java+AI 逆袭 ： 闲了一年，41 岁 / 失业 12 个月 / 彻底绝望 。狠卷 2 个月 Java+AI ，终于逆风翻盘](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486880&idx=1&sn=a37033157c766ac5ae7c9195fe776693&scene=21#wechat_redirect)

[Java+AI 逆袭 5：1 个月大涨 2.5W，37 岁 脱坑外包， 入了正编，GO+AI 要逆天了](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486885&idx=1&sn=4e26fbb093f45d437dedf14ea9b9e6c5&scene=21#wechat_redirect)

**职业救助站**

实现职业转型，极速上岸

![](https://mmbiz.qpic.cn/mmbiz_jpg/9qY2Gt3PBs3icE6WcM4nBOdRRcczOW4wkNDgCMZhBfqeet9suwGHfgQduP33YYqqZLOppecMI72Pv5ibNkPeIk6g/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=1)

关注**职业救助站**公众号，获取每天职业干货  
助您实现**职业转型、职业升级、极速上岸**  
---------------------------------

**技术自由圈**

实现架构转型，再无中年危机

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/xlgvgPaib7WMtyFcJx45oIeWL1d6AENcdoAvuTPuq46E9n8pPe4SHXoSicuEVvhibHRSIzmEBPlC7POcDaqD6ID7w/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=2)

关注**技术自由圈**公众号，获取每天技术千货  
一起成为牛逼的**未来超级架构师**

**几十篇架构笔记、5000 页面试宝典、20 个技术圣经  
请加尼恩个人微信 免费拿走**

**暗号，请在 公众号后台 发送消息：领电子书**

如有收获，请点击底部的 " 在看 "和" 赞 "，谢谢