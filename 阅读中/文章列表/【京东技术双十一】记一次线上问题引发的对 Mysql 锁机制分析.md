---
source: https://mp.weixin.qq.com/s/ssXSwqlXjE0jv-yeq7GX9A
create: 2023-11-14 09:05
read: true
---

最近双十一开门红期间组内出现了一次因 Mysql 死锁导致的线上问题，当时从监控可以看到数据库活跃连接数飙升，导致应用层数据库连接池被打满，后续所有请求都因获取不到连接而失败。

整体业务代码精简逻辑如下：

```
@Transaction
public void service(Integer id) {
    delete(id);
    insert(id);
}
```

数据库实例监控：

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1WEuMK1kkKL7dBvicoSicrcZ98Gj8VLldX4ELsVaoJxGzO2HAQwAickm2NCVZEN5qnosIQ3UC7CR0IOA/640?wx_fmt=png)

图 1. 数据库实例监控示意

当时通过分析上游问题流量限流解决后，后续找时间又重新分析了下问题发生的根本原因，现将其总结如下：本篇文章会先对 Mysql 中的各种锁进行分析，包括互斥锁、间隙锁和插入意向锁，让大家对各种锁的使用场景有一个了解，然后在此基础上再对本问题进行分析，希望大家未来再碰到相似场景时，能够快速的定位问题。

**02** 

 **Mysql 锁机制**

目标页面展示到屏幕。

在 Mysql 中为了解决对同一行记录并发写的问题，引入了行锁机制，多个事务不能同时对一行数据进行修改操作，当需要对数据库中的一行数据进行修改时，会首先判断该行数据是否加锁，如果没加锁，那么当前事务加锁成功，可以进行后续的修改操作；但如果该行数据已经被其他事务加锁，则当前事务只有等待加锁的事务释放锁后才能加锁成功，继续执行修改操作。

本篇文章中所有实验用到的建表语句：

```sql
create table `test` (
    `id` int(11) NOT NULL,
    `num` int(11) NOT NULL,
    PRIMARY KEY (`id`),
    KEY `num` (`num`)
) ENGINE = InnoDB;

insert into
    test
values
(10, 10),
(20, 20),
(30, 30),
(40, 40),
(50, 50);
```

**2.1  Shared and Exclusive Locks**

shared(S) lock 表示共享锁，当一个事务持有某行上的 S 锁后可以对该行的数据进行读操作，通过语句 select ... from test lock in share mode 可以添加共享锁，**一般使用的较少**，不做过多阐述。

exclusive(X) lock 表示互斥锁，当一个事务对**某行数据进行 update 或 delete 操作**时都要先获取到该记录上的 X 锁，如果已经有其他事务获取到了该记录上的 X 锁，那么当前事务会阻塞等待直到上一事务释放了对应记录上的 X 锁。

S 锁之间不互斥，多个事务可以同时获取一条记录上的 S 锁 X 锁之间互斥，多个事务不能同时获取同一条记录上的 X 锁 S 锁和 X 锁之间互斥，多个事务不能同时获取同一条记录上的 S 锁和 X 锁

当多个事务同时去 update 索引上同一条记录时，都需要先获取到该记录上的 X 锁，**所谓的锁也就是会在内存中生成一个数据结构来记录当前的事务信息、锁类型和是否等待等信息**。下图中就是 T1 和 T2 同时去更新 id = 30 的这行记录，并且 T1 成功获取到了锁，其在内存中生成的锁结构信息字段 is_wating 为 false，可以继续执行事务的后续逻辑，而 T2 获取锁失败，则生成的锁结构信息字段 is_wating 为 true，阻塞等待 T1 上的锁释放。

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1WEuMK1kkKL7dBvicoSicrcZ9spVswn2EKHrdmIlV42zPoAt7PN3a7UqiaPdI5tfKmjlv2lAtV03mZUw/640?wx_fmt=png)

图 2. T1 和 T2 同时去更新 id = 30 的这行记录示意

互斥锁在 Mysql 日志中的锁信息为：**lock_mode X locks rec but not gap**。

```
RECORD LOCKS space id 58 page no 3 n bits 72 index `PRIMARY` of table `test`.`t`
trx id 10078 lock_mode X locks rec but not gap
Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 8000000a; asc     ;;
 1: len 6; hex 00000000274f; asc     'O;;
 2: len 7; hex b60000019d0110; asc        ;;
```

**2.2  Gap Locks**

上一小节中介绍了 Exclusive Locks，该锁可以避免多个事务同时对一行记录进行更新操作，**但不能解决幻读的问题**，所谓的幻读就是指一个事务在前后两次查询同一个范围时，后一次查询到了前一次没有的记录。

<table><colgroup><col width="auto"><col width="664.12"><col width="auto"></colgroup><tbody><tr data-slate-node="element" data-slate-inline="false"><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="30.333333333333332"><br></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="282.3333333333333"><span>session A</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="464"><span>session B</span></td></tr><tr data-slate-node="element" data-slate-inline="false"><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="51.33333333333333"><span>T1</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="282.3333333333333"><span>select num from test where num &gt; 10 and num &lt;15 for update; (0 rows)</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="464"><br></td></tr><tr data-slate-node="element" data-slate-inline="false"><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="51.33333333333333"><span>T2</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="282.3333333333333"><br></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="464"><span>insert into test values(12, 12);</span></td></tr><tr data-slate-node="element" data-slate-inline="false"><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="51.33333333333333"><span>T3</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="282.3333333333333"><span>select num from test where num &gt; 10 and num &lt;15 for update; (1 rows)</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="464"><br></td></tr></tbody></table>

在上面这个场景中，session A 分别在 T1、T3 时刻进行了两次范围查询，session B 在 T2 时刻插入了一条该范围内的数据，如果 session A 能在 T3 时刻查询出 session B 插入的数据，就说明发生了幻读。此时只使用互斥锁是无法解决幻读的，因为 num = 12 的记录在数据库中还不存在，不能给其加上互斥锁来防止 T2 时刻 session B 的插入。

因此为了解决幻读问题，只有引入新的锁机制，**也就是间隙锁 (Gap Locks)**。间隙锁和互斥锁不同，互斥锁是行锁，只会锁定一行特定的记录，而间隙锁则是锁定两行记录之间的空隙，防止其他事务在此间隙中插入新的记录。

引入了间隙锁之后，session A 在 T1 时刻会给 id = 20 记录生成一个 Gap Locks，之后 session B 在 T2 时刻想要插入记录时，需要先判断待插入位置的后一条记录上是否存在 Gap Locks，很明显此时 id = 20 的记录上已经存在了 Gap Locks，那么 session B 就需要在 id = 20 的记录上生成一个**插入意向锁**，并进入锁等待。

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1WEuMK1kkKL7dBvicoSicrcZ9mbiaBRODdpmIODZibKWC9iaphJs7hufIfiaLibKsRU95RdlICH0djJI6UXA/640?wx_fmt=png)

图 3. 引入了间隙锁后

间隙锁在 Mysql 中的锁日志信息如下：**lock_mode X locks gap before rec**。

```
RECORD LOCKS space id 133 page no 3 n bits 80 index PRIMARY of table `test`.`test` trx id 38849 lock_mode X locks gap before rec
Record lock, heap no 4 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 4; hex 8000001e; asc     30 ;;
 1: len 6; hex 00000000969c; asc       ;;
 2: len 7; hex a60000011a0128; asc       (;;
 3: len 4; hex 8000001e; asc     ;;
```

间隙锁虽然解决了幻读问题，但因每次都会锁住一段间隙，大大降低了数据库整体的并发度，且因间隙锁和间隙锁之间不互斥，不同事务可以同时对同一间隙加上 Gap Locks，这也往往是各种死锁产生的源头。

**2.3  Next-Key Locks**

‍‍

Next-Key Locks 是 (Shard/Exclusive Locks + Gap Locks) 的结合，当 session A 给某行记录 R 添加了互斥型的 Next-Key Locks 后， 相当于拥有了记录 R 的 X 锁和记录 R 的 Gap Locks。

在上面 Gap Locks 的例子中事务 1 加的就是 Next-Key Locks，即同时给 id = 20 的记录加了 X 锁和 Gap 锁。

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1WEuMK1kkKL7dBvicoSicrcZ9mbiaBRODdpmIODZibKWC9iaphJs7hufIfiaLibKsRU95RdlICH0djJI6UXA/640?wx_fmt=png)

图 4. 同时给 id = 20 的记录加了 X 锁和 Gap 锁

在可重复读隔离级别下，**update 和 delete 操作默认都会给记录添加 Next-Key Locks**，Mysql 中 Next-Key Locks 的锁日志信息为：**lock_mode X**。

```
RECORD LOCKS space id 58 page no 3 n bits 72 index `PRIMARY` of table `test`.`t`
trx id 10080 lock_mode X
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;

Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 8000000a; asc     ;;
 1: len 6; hex 00000000274f; asc     'O;;
 2: len 7; hex b60000019d0110; asc        ;
```

**2.4  Insert Intention Locks**

插入意向锁 (Insert Intention Locks) 也是一种间隙锁，由 INSERT 操作在行数据插入之前获取。

在插入一条记录前，需要先定位到该记录在 B+ 树中的存储位置，然后判断待插入位置的下一条记录上是否添加了 Gap Locks，如果下一条记录上存在 Gap Locks，那么插入操作就需要阻塞等待，直到拥有 Gap Locks 的那个事务提交，同时执行插入操作等待的事务也会在内存中生成一个锁结构，表明有事务想在某个间隙中插入新记录，但目前处于阻塞状态，**生成的锁结构就是插入意向锁**。

实验模拟如下：

<table><colgroup><col width="auto"><col width="auto"><col width="auto"><col width="auto"></colgroup><tbody><tr data-slate-node="element" data-slate-inline="false"><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="40.33333333333333"><br></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="121.33333333333333"><span>session 1</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="128.33333333333334"><span>session 2</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="150.33333333333334"><span>session 3</span></td></tr><tr data-slate-node="element" data-slate-inline="false"><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="41"><span>T1</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="141.33333333333331"><span>begin;</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="148.33333333333337"><br></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="150.33333333333334"><br></td></tr><tr data-slate-node="element" data-slate-inline="false"><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="41"><span>T2</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="141.33333333333331"><span>select * from test where id = 25 for update;</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="148.33333333333337"><br></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="150.33333333333334"><br></td></tr><tr data-slate-node="element" data-slate-inline="false"><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="41"><span>T3</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="141.33333333333331"><br></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="148.33333333333337"><span>insert into test values(26, 26); (blocked)</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="150.33333333333334"><br></td></tr><tr data-slate-node="element" data-slate-inline="false"><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="41"><span>T4</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="141.33333333333331"><br></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="148.33333333333337"><br></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="150.33333333333334"><span>insert into test values(26, 26); (blocked)</span></td></tr></tbody></table>

对于语句 select * from test where id = 25 for update 因当前表中不存在该记录，在可重复读隔离级别下，为了避免幻读，会给 (20, 30] 间隙加上 Gap Locks。

从锁日志可以看出 session 1 给记录 30 添加了间隙锁 (**lock_mode X locks gap before rec**)。

```
RECORD LOCKS space id 133 page no 3 n bits 80 index PRIMARY of table `test`.`test` trx id 38849 lock_mode X locks gap before rec
Record lock, heap no 4 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 4; hex 8000001e; asc     30 ;;
 1: len 6; hex 00000000969c; asc       ;;
 2: len 7; hex a60000011a0128; asc       (;;
 3: len 4; hex 8000001e; asc     ;;
```

当 session 2 插入记录 26 时，会在 B+ 树中先定位到待插入位置，再判断插入位置的间隙是否存在 Gap Locks，也就是判断待插入位置的后一记录 id = 30 是否存在 Gap Locks，如果存在需要在该记录上生成插入意向锁等待。

```
RECORD LOCKS space id 133 page no 3 n bits 80 index PRIMARY of table `test`.`test` trx id 38850 lock_mode X locks gap before rec insert intention waiting
Record lock, heap no 4 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 4; hex 8000001e; asc    30 ;;
 1: len 6; hex 00000000969c; asc       ;;
 2: len 7; hex a60000011a0128; asc       (;;
 3: len 4; hex 8000001e; asc     ;;
```

此时 session 2 和 session 3 都在 id = 30 的记录上添加了插入意向锁等待 session 1 上的 Gap Locks 释放，生成的锁记录如下：

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1WEuMK1kkKL7dBvicoSicrcZ9JqPLPsRHxZTe5fclA3BvIQpSXibzr1WWNStMJtNOqpLolJh1PCJ1KRw/640?wx_fmt=png)

图 5. 生成的锁记录示意

**03** 

## 2. 线上问题分析

目标页面展示到屏幕。

在对 Mysql 中的各种锁结构有了一个清晰的了解之后，回过头来再看看前面的线上问题：

```
@Transaction
public void service(Integer id) {
    delete(id);
    insert(id);
}
```

对于上面的业务代码可能存在下面两种情况：

*   传入的参数 id 在原数据库中不存在
    
*   传入的参数 id 在原数据库中存在

本次主要会针对 id 记录在原数据库中不存在进行分析

<table><colgroup><col width="auto"><col width="auto"><col width="auto"><col width="auto"></colgroup><tbody><tr data-slate-node="element" data-slate-inline="false"><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="28.333333333333332"><br></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="105.33333333333333"><span>session 1</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="81.33333333333333"><span>session 2</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="73.33333333333333"><span>session 3</span></td></tr><tr data-slate-node="element" data-slate-inline="false"><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="48"><span>T1</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="105.33333333333333"><span>delete from test where id = 15;</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="101.33333333333333"><br></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="93.33333333333333"><br></td></tr><tr data-slate-node="element" data-slate-inline="false"><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="48"><span>T2</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="105.33333333333333"><br></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="101.33333333333333"><span>delete from test where id = 15;</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="93.33333333333333"><span>delete from test where id = 15;</span></td></tr><tr data-slate-node="element" data-slate-inline="false"><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="48"><span>T3</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="105.33333333333333"><span>insert into test values(15, 15);</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="101.33333333333333"><br></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="93.33333333333333"><br></td></tr><tr data-slate-node="element" data-slate-inline="false"><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="48"><span>T4</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="105.33333333333333"><br></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="101.33333333333333"><span>insert into test values(15, 15);</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="93.33333333333333"><br></td></tr><tr data-slate-node="element" data-slate-inline="false"><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="48"><span>T5</span></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="105.33333333333333"><br></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="101.33333333333333"><br></td><td colspan="1" rowspan="1" data-slate-node="element" data-slate-inline="false" width="93.33333333333333"><span>insert into test values(15, 15);</span></td></tr></tbody></table>

因 id = 15 在数据库中不存在，在 T1 时刻 session 1 会给其所在间隙的下一条记录添加上 Gap Locks，又因 Gap Locks 不互斥， 在 T2 时刻 session 2 和 session 3 都会同时获取到 id = 20 的 Gap 锁。

下图中 tx: T1、T2、T3 分别代表 session 1、session 2 和 session 3。

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1WEuMK1kkKL7dBvicoSicrcZ9iab9vBLqwMdsRJ5wPmwY9ibN29BpHDqWqtlV0hGk0tqxOuWicGGXSP8Gw/640?wx_fmt=png)

图 6.

当在 T3 时刻 session 1 插入 id = 15 的记录时，会判断其插入位置的后一条记录是否存在 Gap Locks，如果存在，则需要在该记录上生成 Insert Intention Locks 并等待持有 Gap Locks 的事务释放锁。

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1WEuMK1kkKL7dBvicoSicrcZ9fWicwgXzXuqiaH1FJianrImoNqCzS9gyZDv60KM89K0bh7feaib81MCZUQ/640?wx_fmt=png)

图 7.

在 T4 时刻 session 2 执行插入语句，同样会因插入位置的后一条记录中存在 Gap Locks 而需要生成 Insert Intention Locks 等待。此时很明显就形成了死锁，session 1 生成插入意向锁等待 session 2 和 session 3 上的 Gap 锁释放，而 session 2 同样生成插入意向锁等待 session 1 和 session 3 上的 Gap 锁释放。

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1WEuMK1kkKL7dBvicoSicrcZ9cVOdfUejxxYE7ic0bD1hnDRpOjbcdSPyzDjEryZITibe5KWXGE8RbWqA/640?wx_fmt=png)

图 8.

在 T4 时刻检测到死锁后，Mysql 会选择其中一个事务进行回滚，假设此时 session 2 被回滚，释放了其持有的所有锁资源，session 1 可以继续执行吗？很明显不可以，session 1 还同时在等待 session 3 上的 Gap 锁释放，继续阻塞等待。

在 T5 时刻 session 3 开始执行插入语句，此时同 T4 时刻，死锁形成，session 1 生成的插入意向锁正在等待 session 3 上的 Gap Locks 释放，session 3 上生成的插入意向锁正在等待 session 1 上的 Gap Locks 释放，此时 session 3 回滚释放所有锁资源后，session 1 才可以最终执行成功。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/RQv8vncPm1WEuMK1kkKL7dBvicoSicrcZ91K9ic8kovIzYSR0EKWLQVcbCiaxInoo235mL6W6TcKaXzicbQ0nuIuLeg/640?wx_fmt=png)

图 9.

在完成了三个并发线程的死锁分析后，可能有人会想虽然有死锁，但通过死锁检测可以很快的检测出，程序也可以正常的执行，这有什么问题呢？其实上面没有问题主要是因为并发量较小，死锁检测可以很快检测出，如果此时将并发量扩大 100 倍甚至 1000 倍后，还会没有问题吗？

看看当时出现线上问题时，接口的调用量情况：

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1WEuMK1kkKL7dBvicoSicrcZ9ibM9MbYXICejiatW7ThBJ07Gtficf66SICxbbTe8yaNBvn68P5CMSOVicA/640?wx_fmt=png)

图 10. 出现线上问题时接口的调用量

进一步在本地模拟 300 个线程并发执行，因**人脑并发分析**所有事务的执行情况的话会非常复杂，本次只以事务 1 为一个点来进行分析。

从图中可以看到当 T1 在执行插入语句时，需要等待 T2- T101 上持有的 Gap Locks 释放，之后 T2 - T6 可能同时执行插入语句，然后进行死锁检测，事务回滚，看着似乎只要后续有事务执行了插入语句就会执行死锁回滚，正常运行，但在死锁检测的过程中还会有新事务 (T101 - T 200) 获取到 Gap Locks，造成锁等待队列中的事务越来越多，**而 Mysql 的整体死锁检测时间复杂度为 O(n^2)**，锁等待队列中的事务较多时，每一次有新事务进行锁等待，死锁检测都需要遍历锁等待队列中在其之前等待的事务，判断是否会因自己的加入形成环，此时检测会非常消耗 CPU 资源，造成数据库整体性能下降，死锁检测耗时增加，Mysql 活跃连接数大幅增加，并且因锁等待而连接无法释放，最终造成应用层连接池被打满。

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1WEuMK1kkKL7dBvicoSicrcZ9qWUic56LFOXBQ14ibw03WAeia1Vaq2DznTNDTNEnP5wfkpg7nsGI1gvJA/640?wx_fmt=png)

图 11.

综上分析，本次出现问题的最主要原因是在短时间内存在大并发的请求对**同一行数据**进行先删除再插入操作（先更新再插入同理），造成了死锁等待，应用层连接池被打满，大量上游请求超时重试，进一步导致锁等待，最终影响了所有依赖该数据库的业务。

因此对于未来在业务代码中存在相似逻辑的地方，一定要做好防重校验，避免短时间内存在对同一行数据的先更新再插入的并发操作。同时在可重复读隔离别下，更新和删除操作默认都会添加 Next-Key Locks，间隙锁的引入使得死锁问题在并发情况下很容易出现，这也是在业务逻辑实现上需要考虑的问题。

**04** 

 **总结** 

目

本文以一个线上问题为背景，对 Mysql 中的各种锁机制进行了详细的总结，分析了各个锁的加锁时机和具体使用场景，其中特别要注意间隙锁的使用，因间隙锁和间隙锁之间不互斥，当多个事务之间并发执行时很容易形成死锁。