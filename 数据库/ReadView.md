#数据库 

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/u014274324/article/details/107881756)

## 1: 数据库的隔离级别
----------

数据库的隔离级分类分为四类 READ_UNCOMMITTED，READ_COMMITTED，REPEATABLE_READ，SERIALIZABLE。

1.1：READ_UNCOMMITTED

读未提交时，读事务直接读取主记录，无论更新事务是否完成

1.2：READ_COMMITTED

读提交时，读事务每次都读取 undo log 中最近的版本，因此两次对同一字段的读可能读到不同的数据（幻读），但能保证每次

都读到最新的数据。

1.3：REPEATABLE_READ

每次都读取指定的版本，这样保证不会产生幻读，但可能读不到最新的数据

1.4：SERIALIZABLE

锁表，读写相互阻塞，使用较少

## 2：MVCC 多版本机制
------------

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131400.png)

为了实现上面的数据库的隔离级别，mvcc 应运而生，mysql 怎么实现 mvcc，这依赖于 mysql 的隐藏列，trx_id 事务 id，

roll_pointer 回滚指针，row_id 主键，这个主键是表不存在主键和唯一索引的时候自动添加的。

### 2.1：insert

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131404.png)

当插入的是一条新数据时，roll_pointer 指向 insert 的 undo 日志，事务提交之后就没有意义了。

###  2.2：update

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131408.png)

每次对记录进行改动，都会记录一条 undo 日志，每条 undo 日志也都有一个 roll_pointer 属性 (INSERT 操作对应的 undo 日志没有该属性，因为该记录并没有更早的版本)，可以将这些 undo 日志都连起来，串成一个链表

### 2.3：delete

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131413.png)

 delete 语句和 update 基本上没有区别，只是把记录的 delete_mask 置成删除，在后面 purge 的时候，加入到删除的数据页当中去。

### 2.4：readview

有了多版本之后，不知道怎么用也不行啊，mvcc 真正使用的地方是 readview，readview 包含四个内容。

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131417.png)

m_ids: 表示在生成 ReadView 时当前系统中活跃的读写事务的事务 id 列表。

min_trx_id: 表示在生成 ReadView 时当前系统中活跃的读写事务中最小的事务 id，也就是 m_ids 中的最小值。

max_trx_id: 表示生成 ReadView 时系统中应该分配给下一个事务的 id 值。

creator_trx_id: 表示生成该 ReadView 的事务的事务 id。

*   如果被访问版本的 trx_id 属性值与 ReadView 中的 creator_trx_id 值相同，意味着当前事务在访问它自己修改过的记录，所以该版本可以被当前事务访问。
*   如果被访问版本的 trx_id 属性值小于 ReadView 中的 min_trx_id 值，表明生成该版本的事务在当前事务生成 ReadView 前已经提交，所以该版本可以被当前事务访问。
*   如果被访问版本的 trx_id 属性值大于 ReadView 中的 max_trx_id 值，表明生成该版本的事务在当前事务生成 ReadView 后才开启，所以该版本不可以被当前事务访问。
*   如果被访问版本的 trx_id 属性值在 ReadView 的 min_trx_id 和 max_trx_id 之间，那就需要判断一下 trx_id 属性值是不是在 m_ids 列表中，如果在，说明创 建 ReadView 时生成该版本的事务还是活跃的，该版本不可以被访问; 如果不在，说明创建 ReadView 时生成该版本的事务已经被提交，该版本可以被访问。这个的可以访问，在不同的隔离级别下的表现并不相同，READ_COMMITTED 和 REPEATABLE_READ 的表现并不相同，rc 下，事务提交了就可见，rr 下事务提交了，并不可见，保证每次查询的记录相同。但是存在例外，比如 a 提交了事务，b 去查询 a 事务是查不到提交的内容，但是如果去更新 a 提交的事务，是能够更新的，对于这个的讨论，有很多意见，这里我就不发表意见了，下面看一下具体的例子。

### 2.5：READ COMMITTED —— 每次读取数据前都生成一个 ReadView

```
# Transaction 100
    BEGIN;
UPDATE hero SET name = '关羽' WHERE number = 1;
UPDATE hero SET name = '张飞' WHERE number = 1;
 
# Transaction 200 BEGIN;
# 更新了一些别的表的记录 ...
```

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131422.png)

假设现在有一个使用 READ COMMITTED 隔离级别的事务开始执行:

```
# 使用READ COMMITTED隔离级别的事务
BEGIN;
# SELECT1:Transaction 100、200未提交
SELECT * FROM hero WHERE number = 1; # 得到的列name的值为'刘备'
```

这个 SELECT1 的执行过程如下:

在执行 SELECT 语句时会先生成一个 ReadView，ReadView 的 m_ids 列表的内容就是 [100, 200]，min_trx_id 为 100，max_trx_id 为 201，creator_trx_id 为 0。

然后从版本链中挑选可见的记录，从图中可以看出，最新版本的列 name 的内容是'张飞'，该版本的 trx_id 值为 100，在 m_ids 列表内，所以不符合可见性要求，根据 roll_pointer 跳到下一个版本。

下一个版本的列 name 的内容是'关羽'，该版本的 trx_id 值也为 100，也在 m_ids 列表内，所以也不符合要求，继续跳到下一个版本。

下一个版本的列 name 的内容是'刘备'，该版本的 trx_id 值为 80，小于 ReadView 中的 min_trx_id 值 100，所以这个版本是符合要求的，最后返回给用户的版本就是这条 列 name 为'刘备'的记录。

之后，我们把事务 id 为 100 的事务提交一下，就像这样:

```
# Transaction 100
BEGIN;
UPDATE hero SET name = '关羽' WHERE number = 1; 
UPDATE hero SET name = '张飞' WHERE number = 1;    
COMMIT;
```

 然后再到事务 id 为 200 的事务中更新一下表 hero 中 number 为 1 的记录:

```
# Transaction 200 
BEGIN;
# 更新了一些别的表的记录 ...
UPDATE hero SET name = '赵云' WHERE number = 1;
UPDATE hero SET name = '诸葛亮' WHERE number = 1;
```

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131428.png)

然后再到刚才使用 READ COMMITTED 隔离级别的事务中继续查找这个 number 为 1 的记录，如下:

```
# 使用READ COMMITTED隔离级别的事务
BEGIN;
# SELECT1:Transaction 100、200均未提交
SELECT * FROM hero WHERE number = 1; # 得到的列name的值为'刘备' 
# SELECT2:Transaction 100提交，Transaction 200未提交
SELECT * FROM hero WHERE number = 1; # 得到的列name的值为'张飞'
```

 这个 SELECT2 的执行过程如下:

在执行 SELECT 语句时会又会单独生成一个 ReadView，该 ReadView 的 m_ids 列表的内容就是 [200](事务 id 为 100 的那个事务已经提交了，所以再次生成快照时就没有它 了)，min_trx_id 为 200，max_trx_id 为 201，creator_trx_id 为 0。

然后从版本链中挑选可见的记录，从图中可以看出，最新版本的列 name 的内容是'诸葛亮'，该版本的 trx_id 值为 200，在 m_ids 列表内，所以不符合可见性要求，根 据 roll_pointer 跳到下一个版本。

下一个版本的列 name 的内容是'赵云'，该版本的 trx_id 值为 200，也在 m_ids 列表内，所以也不符合要求，继续跳到下一个版本。 下一个版本的列 name 的内容是'张飞'，该版本的 trx_id 值为 100，小于 ReadView 中的 min_trx_id 值 200，所以这个版本是符合要求的，最后返回给用户的版本就是这条列 name 为'张飞'的记录。

### 2.6：REPEATABLEREAD——在第一次读取数据时生成一个 ReadView

```
比方说现在系统里有两个事务id分别为100、200的事务在执行:
# Transaction 100 BEGIN;
UPDATE hero SET name = '关羽' WHERE number = 1;
UPDATE hero SET name = '张飞' WHERE number = 1;
# Transaction 200 BEGIN;
# 更新了一些别的表的记录 ...
```

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131433.png) 

 假设现在有一个使用 REPEATABLE READ 隔离级别的事务开始执行:

```
# 使用REPEATABLE READ隔离级别的事务
BEGIN;
# SELECT1:Transaction 100、200未提交
SELECT * FROM hero WHERE number = 1; # 得到的列name的值为'刘备'
```

这个 SELECT1 的执行过程如下:

在执行 SELECT 语句时会先生成一个 ReadView，ReadView 的 m_ids 列表的内容就是 [100, 200]，min_trx_id 为 100，max_trx_id 为 201，creator_trx_id 为 0。 然后从版本链中挑选可见的记录，从图中可以看出，最新版本的列 name 的内容是'张飞'，该版本的 trx_id 值为 100，在 m_ids 列表内，所以不符合可见性要求，根据 roll_pointer 跳到下一个版本。

下一个版本的列 name 的内容是'关羽'，该版本的 trx_id 值也为 100，也在 m_ids 列表内，所以也不符合要求，继续跳到下一个版本。

下一个版本的列 name 的内容是'刘备'，该版本的 trx_id 值为 80，小于 ReadView 中的 min_trx_id 值 100，所以这个版本是符合要求的，最后返回给用户的版本就是这条 列 name 为'刘备'的记录。

之后，我们把事务 id 为 100 的事务提交一下，就像这样:

```
# Transaction 100
BEGIN;
UPDATE hero SET name = '关羽' WHERE number = 1;
UPDATE hero SET name = '张飞' WHERE number = 1;
COMMIT;
然后再到事务id为200的事务中更新一下表hero中number为1的记录:
# Transaction 200 
BEGIN;
# 更新了一些别的表的记录 ...
UPDATE hero SET name = '赵云' WHERE number = 1; UPDATE hero SET name = '诸葛亮' WHERE number = 1;
```

 此刻，表 hero 中 number 为 1 的记录的版本链就长这样:

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131438.png)

然后再到刚才使用 REPEATABLE READ 隔离级别的事务中继续查找这个 number 为 1 的记录，如下:

```
# 使用REPEATABLE READ隔离级别的事务
BEGIN;
# SELECT1:Transaction 100、200均未提交
SELECT * FROM hero WHERE number = 1; # 得到的列name的值为'刘备'
# SELECT2:Transaction 100提交，Transaction 200未提交
SELECT * FROM hero WHERE number = 1; # 得到的列name的值仍为'刘备'
```

这个 SELECT2 的执行过程如下:

因为当前事务的隔离级别为 REPEATABLE READ，而之前在执行 SELECT1 时已经生成过 ReadView 了，所以此时直接复用之前的 ReadView，之前的 ReadView 的 m_ids 列表 的内容就是 [100, 200]，min_trx_id 为 100，max_trx_id 为 201，creator_trx_id 为 0。

然后从版本链中挑选可见的记录，从图中可以看出，最新版本的列 name 的内容是'诸葛亮'，该版本的 trx_id 值为 200，在 m_ids 列表内，所以不符合可见性要求，根据 roll_pointer 跳到下一个版本。

下一个版本的列 name 的内容是'赵云'，该版本的 trx_id 值为 200，也在 m_ids 列表内，所以也不符合要求，继续跳到下一个版本。 下一个版本的列 name 的内容是'张飞'，该版本的 trx_id 值为 100，而 m_ids 列表中是包含值为 100 的事务 id 的，所以该版本也不符合要求，同理下一个列 name 的内容是'关羽'的版本也不符合要求。

继续跳到下一个版本。 下一个版本的列 name 的内容是'刘备'，该版本的 trx_id 值为 80，小于 ReadView 中的 min_trx_id 值 100，所以这个版本是符合要求的，最后返回给用户的版本就是这条列 c 为'刘备'的记录。

也就是说两次 SELECT 查询得到的结果是重复的，记录的列 c 值都是'刘备'，这就是可重复读的含义。如果我们之后再把事务 id 为 200 的记录提交了，然后再到刚才使用 REPEATABLE READ 隔离级别的事务中继续查找这个 number 为 1 的记录，得到的结果还是'刘备'。


## 3：explain 简单使用
--------------

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131451.png)![](https://img-blog.csdnimg.cn/20200808200404203.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQyNzQzMjQ=,size_16,color_FFFFFF,t_70)

```
EXPLAIN SELECT age,name from test where age > 11 and age < 15
```

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131455.png)

id: 选择标识符   select_type: 表示查询的类型   table: 输出结果集的表   partitions: 匹配的分区   type: 表示表的连接类型  
possible_keys: 表示查询时，可能使用的索引  key: 表示实际使用的索引   key_len: 索引字段的长度   ref: 列与索引的比较  
rows: 扫描出的行数 (估算的行数)   filtered: 按表条件过滤的行百分比   Extra: 执行情况的描述和说明 

在这些元素里面，可能大家经常使用的 type 和 extra，其他的也重要，有兴趣的可以自己去搜索一下，像物化表这些，在目前的环境中，公司是不允许出现太复杂 sql 的，太复杂的 sql，如果做架构升级，那也是噩梦，所以一般的 sql 都是单表操作的。

### 3.1：type 属性

**type:system** 当表中只有一条记录并且该表使用的存储引擎的统计数据是精确的，比如 MyISAM、Memory，那么对该表的访问方法就是 system。比方说我们新建一个 MyISAM 表，并为其插入一条记录:

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131459.png)

**type:const** 当我们根据主键或者唯一二级索引列与常数进行等值匹配时，对单表的访问方法就是 const

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131503.png)

**type:eq_ref** 在连接查询时，如果被驱动表是通过主键或者唯一二级索引列等值匹配的方式进行访问的 (如果该主键或者唯一二级索引是联合索引的话，所有的索引列都必须进行等值比较)，则对该被驱动表的 访问方法就是 eq_ref

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131506.png)

**type:ref** 当通过普通的二级索引列与常量进行等值匹配时来查询某个表，那么对该表的访问方法就可能是 ref

**type:ref_or_null** 当对普通二级索引进行等值匹配查询，该索引列的值也可以是 NULL 值时，那么对该表的访问方法就可能是 ref_or_null

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131510.png)

**type:range** 如果使用索引获取某些范围区间的记录，那么就可能使用到 range 访问方法

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131514.png)

**type:index** 当我们可以使用索引覆盖，但需要扫描全部的索引记录时，该表的访问方法就是 index

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131517.png)**type:all** 最熟悉的全表扫描

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131521.png)

### 3.2：extra 属性

**extra:Using index** 当我们的查询列表以及搜索条件中只包含属于某个索引的列，也就是在可以使用索引覆盖的情况下，在 Extra 列将会提示该额外信息

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131527.png)

 **extra:Using where** 当我们使用全表扫描来执行对某个表的查询，并且该语句的 WHERE 子句中有针对该表的搜索条件时，在 Extra 列中会提示上述额外信息

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131531.png)

**extra:Using index condition**

有些搜索条件中虽然出现了索引列，但却不能使用到索引，比如下边这个查询:

SELECT * FROM s1 WHERE key1 > 'z' AND key1 LIKE '%a';  
其中的 key1 > 'z'可以使用到索引，但是 key1 LIKE '%a'却无法使用到索引，在以前版本的 MySQL 中，是按照下边步骤来执行这个查询的:

先根据 key1 > 'z'这个条件，从二级索引 idx_key1 中获取到对应的二级索引记录。

根据上一步骤得到的二级索引记录中的主键值进行回表，找到完整的用户记录再检测该记录是否符合 key1 LIKE '%a'这个条件，将符合条件的记录加入到最后的结果集。 但是虽然 key1 LIKE '%a'不能组成范围区间参与 range 访问方法的执行，但这个条件毕竟只涉及到了 key1 列，所以设计 MySQL 的大叔把上边的步骤改进了一下:

先根据 key1 > 'z'这个条件，定位到二级索引 idx_key1 中对应的二级索引记录。  
对于指定的二级索引记录，先不着急回表，而是先检测一下该记录是否满足 key1 LIKE '%a'这个条件，如果这个条件不满足，则该二级索引记录压根儿就没必要回表。 对于满足 key1 LIKE '%a'这个条件的二级索引记录执行回表操作。

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131536.png)

**extra:using index & using where** 查找使用了索引，但是需要的数据都在索引列中能找到，所以不需要回表查询数据，这个和上面的相对，select 后面的列在索引上都有，不需要回表，而上面的有部分列没有，需要回表。