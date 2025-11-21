---
source: https://mp.weixin.qq.com/s/qYkyzD79nQuNGe59M2wWgg
create: 2024-08-16 16:30
read: true
knowledge: true
knowledge-date: 2025-10-29
tags:
  - Mysql
  - 框架原理
summary: "[[MySQL 8.0：filesort 性能退化的问题分析]]"
---

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naKPzAv76YOSFTMmyUbbZnameYkhH76AGD3rHeqwT2cvELicmZXFWzXtCx1vaNvfa3Vo8GdLyoRKNEA/640?wx_fmt=jpeg&from=appmsg)

阿里妹导读

用户将 RDS MySQL 实例从 5.6 升级到 8.0 后，发现相同 SQL 的执行时间增长了十几倍。本文就该问题逐步展开排查，并最终定位根因。

一、背景

用户将 MySQL 实例从 5.6 升级到 8.0 后，发现相同 SQL 的执行时间增长了十几倍。用户的查询在简化后可以认为是一个对非索引字段做 order by 的全表扫描，理论上性能差异不应该如此巨大。带着这个疑问，本文将逐步展开问题排查的过程，定位性能退化的根因。

二、问题分析

**2.1 初分析**

我们首先基于问题实例进行分析，直接对比了问题 SQL 在不同版本之间的执行计划差异。

*   从 EXPALIN 的结果来看，两者相同，均为全表扫描 + filesort；

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naKUlu2HvmqF0xcwteNJvQ8xFwkcyrl3BLOpcXmrVRdQ9pODaoGj3J7BzSHcKf0ibZZLdjvKJn97iceQ/640?wx_fmt=other&from=appmsg)

线上实例的 EXPLAIN 对比

*   从 Optimizer Trace 结果来看，除了版本间 trace 信息输出格式的差异外，执行情况基本相同；

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naKUlu2HvmqF0xcwteNJvQ8xjKicDRPxicj9rjstVJiaDdh4Dz8zibzx4r79pW2ULn3Am0SozUMGpSTVZQ/640?wx_fmt=other&from=appmsg)

线上实例的 Optimizer Trace 对比

*   从 profiling 信息看，只能看出 8.0 版本在 executing 阶段的耗时大大增加了。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naKUlu2HvmqF0xcwteNJvQ8xZrvPHUYPUmvLTK7uib1rI3aLJ19Lb2f4HoxotbvcV5eG32FmRKCPK4Q/640?wx_fmt=other&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naKUlu2HvmqF0xcwteNJvQ8xZAgaoOoMl3bnuracf5DicficIQjwjRQ1h4Cv2Niaj4via5PciaPicD00jvWQ/640?wx_fmt=other&from=appmsg)

线上实例的 profiling 对比

至此，执行计划差异的嫌疑排除。同时，通过对比机型、CPU 代际、相关参数配置等发现，不同版本实例的前述信息基本都是对齐的，可以初步认为是数据库内核中存在的问题。

进一步地，笔者使用 perf 工具对用户实例 SQL 执行过程中函数堆栈信息进行采集，发现热点函数集中在 row_sel_store_mysql_rec 上，这个函数主要的作用是将数据从 InnoDB 层转换到 Server 层。不论是 5.6 版本还是 8.0 版本，这个函数内部的实现逻辑都是相似的，在数据读取的过程中，这个函数也是绕不开的函数。

**2.2 再分析**

*   问题复现

为了能更深入地对这个问题进行分析，笔者对用户查询涉及的表结构和 SQL 进行简化，在开发、测试环境下复现了用户的问题：**MySQL 8.0 执行耗时 1.14s，而在 MySQL 5.6 中执行耗时仅为 0.14s。**表结构和 SQL 如下所示：

考虑到 RDS MySQL 并没有对涉及到的部分做大的改造，因此测试使用的是 MySQL 社区版本（8.0.38 和 5.6.51）。

```
# 建表
create table t1 (
  id int auto_increment primary key,
  col2 int,
  col3 char(255),
  col4 varchar(8192),
  paytime int unsigned DEFAULT NULL
) ENGINE=INNODB;
 # 导入数据
DELIMITER //
CREATE PROCEDURE insert_data()
BEGIN
    DECLARE i INT DEFAULT 0;

    WHILE i < 300000 DO
        INSERT INTO t1 VALUES (null, i, repeat('a', 255), repeat('a', 8192), FLOOR(RAND() * 300000));
        SET i = i + 1;
    END WHILE;
END //

CALL insert_data;
 # 查询
SELECT * FROM t1 ORDER BY paytime LIMIT 1;
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naKUlu2HvmqF0xcwteNJvQ8xOy1slNbPhZ6LZ4BWBJQPZQBxFYXeSbLreibIibIrgEa1qavqYdIbWUFw/640?wx_fmt=other&from=appmsg)

8.0 和 5.6 执行 SQL 的时间对比

表的特点是：单行记录大，在特定的 row_format 下，溢出的数据会保存到额外的 page 中。

查询的特点是：查询多个字段、order by 非索引字段、只取 1 行数据。

*   Perf

在简化 SQL 执行的过程中，通过对比 perf 中的信息，我们可以发现热点函数 rew_sel_store_mysql_rec 和线上的扁鹊堆栈吻合。不论是 5.6 版本还是 8.0 版本，数据读取（ha_rnd_next）都占了 80% 左右的 CPU，区别在于 80 对于 row_sel_store_mysql_rec 的处理效率很低，占比 63%，而 56 仅为 28%。

进一步展开后，可以看到 8.0 版本在此操作中做了大量的内存拷贝操作。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naKUlu2HvmqF0xcwteNJvQ8xzF8o7zlVmoQibXj8CNxvhxIcicsgCl0v2jZL4CvnbgrUOeORK8oc4Bnw/640?wx_fmt=other&from=appmsg)

5.6 perf 火焰图

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naKUlu2HvmqF0xcwteNJvQ8xIxibhTZWNEwmgrGjZ7e3NBZgkZ2GsTRavZmJ79w20bOyohJmkIoyRjQ/640?wx_fmt=other&from=appmsg)

8.0 perf 火焰图

*    bpftrace 

为什么 row_sel_store_mysql_rec 在 8.0 版本中执行得这么慢？带着这个问题，笔者使用 bpftrace 对相关的函数进行了分析。

```
# 监测8.0中row_sel_store_mysql_rec的执行情况
bpftrace -e '
  uprobe:/home/songhuaxiong.shx/huaxiong.shx/code/mysql-community//mysql-8.0.38-linux-glibc2.12-x86_64/bin/mysqld:_Z23row_sel_store_mysql_recPhP14row_prebuilt_tPKhPK8dtuple_tbPK12dict_index_tS9_PKmbPN3lob11undo_vers_tERP16mem_block_info_t {
    @start[tid] = nsecs;
  }

  uretprobe:/home/songhuaxiong.shx/huaxiong.shx/code/mysql-community//mysql-8.0.38-linux-glibc2.12-x86_64/bin/mysqld:_Z23row_sel_store_mysql_recPhP14row_prebuilt_tPKhPK8dtuple_tbPK12dict_index_tS9_PKmbPN3lob11undo_vers_tERP16mem_block_info_t /@start[tid]/ {
    @duration = hist(nsecs - @start[tid]);
    delete(@start[tid]);
    @count++;
  }

  END {
    printf("Function execution count: %d\n", @count);
    print(@duration);
  }
'
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naKUlu2HvmqF0xcwteNJvQ8xUAqekCpwZkiaEeOu5icQHLjZtGKibyjTHY3UYAwg3ibgicFX9RXEhNSgJgA/640?wx_fmt=other&from=appmsg)

5.6 row_sel_store_mysql_rec 执行情况

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naKUlu2HvmqF0xcwteNJvQ8xcHqK2x5cQiaHbeEwxHRxPmribRN1iceiaBaSURp1vXEPDlibibjNNLB9plTQ/640?wx_fmt=other&from=appmsg)

8.0 row_sel_store_mysql_rec 执行情况

可以发现，8.0 中函数 row_sel_store_mysql_rec 的执行时间区间直接比 5.6 翻了一倍！进一步地，结合 Perf 热点来看：

btr_copy_externally_stored_field 

这个函数非常可疑，这个函数是对溢出页做处理的函数，而用户的表和线下构造的表中确实页是包含了溢出页的。自然地，笔者对：

btr_copy_externally_stored_field 

这个函数执行情况也进行了监测，关键点终于浮出水面。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naKUlu2HvmqF0xcwteNJvQ8xKSiaeq3SpbOTnabw5iaH1Cl31vZibosEmpIlVBwc3LMshkLxAMpmuKYJw/640?wx_fmt=other&from=appmsg)

5.6 btr_copy_externally_stored_field 执行情况

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naKUlu2HvmqF0xcwteNJvQ8xrWI66RHefJNcbTspYnRX6fiarHFribeKPibHcUUMORvc1ZZ9fSBibVuF5g/640?wx_fmt=other&from=appmsg)

8.0 btr_copy_externally_stored_field 执行情况

8.0 中，btr_copy_externally_stored_field 执行了 300000 次，这和全表扫描的数量是匹配的；5.6 中 btr_copy_externally_stored_field 函数仅执行了 1 次，这和 limit 1 的操作是匹配的。

row_sel_store_mysql_rec 执行次数相同而 btr_copy_externally_stored_field 执行次数不同，这说明在 8.0 中，所有的字段都会参与到 InnoDB 到 Server 的转换中，其中就包括了 col4 这个超长的、会溢出的字段。而在 5.6 中，col4 字段的转换只发生了 1 次。

至此，笔者得出一个猜测：对于本案例中的 limit 操作，80 版本在排序过程会完成所有字段的转换，最后再去取 limit 条数据；56 在排序的过程并不会对不相关的 field 做转换（构造的 SQL 中 order by 涉及的字段仅有 paytime），只在筛选完成后对最后的 limit 条的数据去做完整 field 转换（构造的 SQL 中最终需要返回所有字段）。

**2.3 源码分析**

实际上通过阅读 5.6 和 8.0 的源码，我们可以发现，row_sel_store_mysql_rec 的处理逻辑几乎是相同的，聚焦问题相关的逻辑，可以简单理解为：row_sel_store_mysql_rec 会根据上层传入的标记（read_set，一个用于标记字段使用情况的 bitmap），决定对那些字段做数据从 InnoDB 到 Server 的转换。对于溢出字段，会单独调用 btr_copy_externally_stored_field 进行处理。

不论是 filesort 过程还是最后 limit 数据的读取过程，都会走到 row_sel_store_mysql_rec 的逻辑中。

发生问题的根源在于：

*   5.6 版本的 filesort 实现中，在正式读取数据并进行排序前，重置了 read_set，只把排序涉及的 filed 注册到了 read_set 中；
    
*   在 8.0 中，并没有对 read_set 做特殊的处理。这直接影响了后续 row_sel_store_mysql_rec 的行为。

在这个问题中，我们将这一过程简化为 3 个阶段：排序前、排序过程和排序完成。各阶段主要工作如下：

1. 排序前：read_set 中保存了本次查询涉及到的所有字段，对于本例，read_set 对应的是所有字段（SELECT *）；
2. 排序过程：MySQL 5.6 会做 read_set 的修改，在进入 row_sel_store_mysql_rec 前会将 read_set 设置为 tmp_set（全 0），随后在 read_set 重新标记上本次需要访问的字段，对于本例，重新标记的 read_set 对应主键字段和 paytime 字段（order by 字段）。因此在 filesort -> row_sel_store_mysql_rec 的执行过程中，就不会再对未标记的溢出字段（本例中为 col4）做数据的转换；

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naKUlu2HvmqF0xcwteNJvQ8x1obHJa1sxkMj90FnZiblWxAUb5ZdCSQfdN66driaZm9rGxASOQKQiaibdA/640?wx_fmt=other&from=appmsg)

5.6 排序前设置 read_set

3. 排序完成：MySQL 5.6 重新将 read_set 设置为排序前的值，开始数据读取。此时由于 read_set 是全部字段，因此这里发生了唯一的一次对溢出字段的转换处理。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naKUlu2HvmqF0xcwteNJvQ8xwCg8HHicdHFZk9HcGGvFE6CN6o3vywdy38IL8bPDhO1hndhuM1vbK7Q/640?wx_fmt=other&from=appmsg)

5.6 排序完成后重置 read_set

对于 80 来说，filesort 逻辑和前后的流程做了大的改动，设置 read_set 和重置 read_set 在 80 版本的 filesort 中已经不存在了。因此，不论是在 filesort 过程还是在最后的 read limit 数据过程，都会触发所有字段的数据转换，造成查询低效。

三、小结

8.0 排序过程中发生了对无关列的数据转换，导致了性能退化。用户的实例中包含了溢出列，由于溢出列的数据转换格外耗时，最终将这个性能退化的问题放大。

除了上述提到的问题，笔者使用 SEELCT * FROM t1 ORDER BY col4 LIMIT 1; 这条 SQL 进行查询（涉及溢出列），发现了 filesort 还存在其他的热点问题。在这个查询下，5.6 的有效操作（read） 的占比有 62%，而 8.0 中有效操作占比仅有 16%，非核心的操作占用了大量的 CPU 时间。

实际上，笔者也测试了 5.7 版本的 MySQL 并阅读了相关源码，该版本也不存在上述的性能问题。在和社区后续的沟通中，社区验证了该问题在 8.0 、8.4 和 9.0 中均存在，且该问题在 8.4 中的影响更为剧烈。

在实际的使用中，我们建议用户在查询中合理地去使用索引以提高执行效率，但同时需要明确的是，该问题在 MySQL 8.0 及后续版本中是客观存在的，有待进一步的优化。

**ECS 数据备份与保护**

随着企业核心业务规模不断扩大，需要根据业务需求对生产环境中的关键数据进行定期备份，在发生误操作、病毒感染、或攻击等情况时，能够快速从已有的快照恢复到某个历史状态，从而最大程度减少数据丢失带来的损失。

点击阅读原文查看详情。