#数据库 #Mysql 

# 1. 基于成本的优化

`MySQL` 执行一个查询会选择其中成本最低的方案去执行，成本主要定义为：

- `I/O` 成本
    把数据或者索引从磁盘到内存这个加载的过程损耗的时间称之为 `I/O` 成本。
- `CPU` 成本
    读取以及检测记录是否满足对应的搜索条件、对结果集进行排序等这些操作损耗的时间称之为 `CPU` 成本。

对于 `InnoDB` 存储引擎来说，页是磁盘和内存之间交互的基本单位，读取一个页面花费的成本默认是 `1.0`，读取以及检测一条记录是否符合搜索条件的成本默认是 `0.2`，这些数字称之为 `成本常数`。

## 1.1. 单表查询成本

在一条单表查询语句真正执行之前，`MySQL` 的查询优化器会找出执行该语句所有可能使用的方案，对比之后找出成本最低的方案，这个成本最低的方案就是所谓的 `执行计划`，之后才会调用存储引擎提供的接口真正的执行查询：

1. 根据搜索条件，找出所有可能使用的索引。
2. 计算全表扫描的代价。
3. 计算使用不同索引执行查询的代价。
1. 对比各种执行方案的代价，找出成本最低的那一个。

### 1.1.1. 全表查询成本

全表扫描意味着把聚簇索引中的记录都依次和给定的搜索条件做一下比较，把符合搜索条件的记录加入到结果集，所以需要将聚簇索引对应的页面加载到内存中，然后再检测记录是否符合搜索条件。由于查询成本= `I/O` 成本+ `CPU` 成本，所以计算全表扫描的代价需要两个信息：

- 聚簇索引占用的页面数
- 该表中的记录数

`MySQL` 提供了 `SHOW TABLE STATUS` 语句来查看表的统计信息：

```yaml
mysql> USE xiaohaizi;
Database changed

mysql> SHOW TABLE STATUS LIKE 'single_table'\G
*************************** 1. row ***************************
           Name: single_table
         Engine: InnoDB
        Version: 10
     Row_format: Dynamic
           Rows: 9693
 Avg_row_length: 163
    Data_length: 1589248
Max_data_length: 0
   Index_length: 2752512
      Data_free: 4194304
 Auto_increment: 10001
    Create_time: 2018-12-10 13:37:23
    Update_time: 2018-12-10 13:38:03
     Check_time: NULL
      Collation: utf8_general_ci
       Checksum: NULL
 Create_options:
        Comment:
1 row in set (0.01 sec)
```

核心信息有以下两个：

- `Rows`
	表示表中的记录条数。对于使用 `MyISAM` 存储引擎的表来说，该值是准确的，对于使用 `InnoDB` 存储引擎的表来说，该值是一个估计值。
- `Data_length`
	本选项表示表占用的存储空间字节数。使用 `MyISAM` 存储引擎的表来说，该值就是数据文件的大小，对于使用 `InnoDB` 存储引擎的表来说，该值就相当于聚簇索引占用的存储空间大小，也就是说可以这样计算该值的大小：

	$$
	
	Data\_length = 聚簇索引的页面数量 \times 每个页面的大小
	
	$$

	页面大小默认为 `16KB`，而上边查询结果显示 `Data_length` 的值是 `1589248`，所以我们可以反向来推导出 `聚簇索引的页面数量` ：

$$

聚簇索引的页面数量 = 1589248 \div 16 \div 1024 = 97

$$

我们现在已经得到了聚簇索引占用的页面数量以及该表记录数的估计值，所以就可以计算全表扫描成本了， `MySQL` 的大叔在真实计算成本时会进行一些 `微调`：

* `I/O` 成本

    $$
    97 \times 1.0 + 1.1 = 98.1
    $$

    `97` 指的是聚簇索引占用的页面数，`1.0` 指的是加载一个页面的成本常数， `1.1` 是一个微调值。

* `CPU` 成本：

	$$
	9693 \times 0.2 + 1.0 = 1939.6
	$$

    `9693` 指的是统计数据中表的记录数，对于 `InnoDB` 存储引擎来说是一个估计值，`0.2` 指的是访问一条记录所需的成本常数， `1.0` 是一个微调值。

* 总成本：

	$$
	98.1 + 1939.6 = 2037.7
	$$

综上所述，对于 `single_table` 的全表扫描所需的总成本就是 `2037.7`。

### 1.1.2. 使用索引的成本

#### 1.1.2.1. 基于索引页估算记录条数

假设 `idx_key2` 对应的搜索条件是：`key2 > 10 AND key2 < 1000`，也就是说对应的范围区间就是：`(10, 1000)`。

对于使用 `二级索引 + 回表` 方式的查询查询的成本依赖两个方面的数据：

* 范围区间数量
    不论某个范围区间的二级索引到底占用了多少页面，查询优化器粗暴的认为读取索引的一个范围区间的 `I/O` 成本和读取一个页面是相同的：

    $$
    
    1 \times 1.0 = 1.0
    
    $$

* 需要回表的记录数
    优化器需要计算二级索引的某个范围区间到底包含多少条记录：
    1. 寻找区间最左记录
        先根据 `key2 > 10` 这个条件访问一下 `idx_key2` 对应的 `B+` 树索引，找到满足 `key2 > 10` 这个条件的第一条记录，我们把这条记录称之为 `区间最左记录`。在 `B+` 数树中定位一条记录的过程是常数级别的，所以这个过程的性能消耗是可以忽略不计的。
    2. 寻找区间最右记录
        然后再根据 `key2 < 1000` 这个条件继续从 `idx_key2` 对应的 `B+` 树索引中找出最后一条满足这个条件的记录，我们把这条记录称之为 `区间最右记录`。
    3. 计算区间内的记录数
        如果 `区间最左记录` 和 `区间最右记录` 相隔不太远（在 `MySQL 5.7.21` 这个版本里，只要相隔不大于 10 个页面即可），那就可以精确统计出满足 `key2 > 10 AND key2 < 1000` 条件的二级索引记录条数。

        否则**只沿着 `区间最左记录` 向右读 10 个页面，计算平均每个页面中包含多少记录，然后用这个平均值乘以 `区间最左记录` 和 `区间最右记录` 之间的页面数量**就可以了。

        那么如何估计 `区间最左记录` 和 `区间最右记录` 之间有多少个页面呢？解决这个问题还得回到 `B+` 树索引的结构中来：

         ![](https://r2.129870.xyz/img/202209082349305.png)

        如图，我们假设 `区间最左记录` 在 `页b` 中，`区间最右记录` 在 `页c` 中，那么我们想计算 `区间最左记录` 和 `区间最右记录` 之间的页面数量就相当于计算 `页b` 和 `页c` 之间有多少页面，而每一条 `目录项记录` 都对应一个数据页，所以**计算 `页b` 和 `页c` 之间有多少页面就相当于计算它们父节点（也就是页 a）中对应的目录项记录之间隔着几条记录**。

        如果 `页b` 和 `页c` 之间的页面实在太多，以至于 `页b` 和 `页c` 对应的目录项记录都不在一个页面中？则可以继续递归，也就是再统计 `页b` 和 `页c` 对应的目录项记录所在页之间有多少个页面。通常一个 `B+` 树有 4 层高左右，所以这个统计过程也不是很耗费性能。

    知道了如何统计二级索引某个范围区间的记录数之后，就需要回到现实问题中来，根据上述算法测得 `idx_key2` 在区间 `(10, 1000)` 之间大约有 `95` 条记录。读取这 `95` 条二级索引记录需要付出的 `CPU` 成本就是：

    $$
    
    95 \times 0.2 + 0.01 = 19.01
    
    $$

    其中 `95` 是需要读取的二级索引记录条数，`0.2` 是读取一条记录成本常数，`0.01` 是微调。

    在通过二级索引获取到记录之后，还需要干两件事儿：

    * 根据这些记录里的主键值到聚簇索引中做回表操作
        ** `MySQL` 评估回表操作的 `I/O` 成本是回表操作都相当于访问一个页面**：

        $$
        
        95 \times 1.0 = 95.0
        
        $$

        其中 `95` 是预计的二级索引记录数，`1.0` 是一个页面的 `I/O` 成本常数。

    * 回表操作后得到的完整用户记录，然后再检测其他搜索条件是否成立
        读取并检测这些完整的用户记录是否符合其余的搜索条件的 `CPU` 成本如下：

        $$
        
        95 \times 0.2 = 19.0
        
        $$

        其中 `95` 是待检测记录的条数，`0.2` 是检测一条记录是否符合给定的搜索条件的成本常数。

所以本例中使用 `idx_key2` 执行查询的成本就如下所示：

*   `I/O` 成本：

    $$
    
    1.0 + 95 \times 1.0 = 96.0 (范围区间的数量 + 预估的二级索引记录条数)
    
    $$

*   `CPU` 成本：
    读取二级索引记录的成本 + 读取并检测回表后聚簇索引记录的成本：
    
    $$
    
    95 \times 0.2 + 0.01 + 95 \times 0.2 = 38.01 
    
    $$

综上所述，使用 `idx_key2` 执行查询的总成本就是：

$$

96.0 + 38.01 = 134.01

$$

#### 1.1.2.2. 基于索引统计数据估算记录条数

使用索引执行查询时可能会有许多单点区间，例如 `IN` 语句：

```sql
SELECT * FROM single_table WHERE key1 IN ('aa1', 'aa2', 'aa3', ... , 'zzz');
```

这个查询可能使用到的索引就是 `idx_key1`，由于这个索引并不是唯一二级索引，所以并不能确定一个单点区间对应的二级索引记录的条数有多少。

计算方式就是先获取索引对应的 `B+` 树的 `区间最左记录` 和 `区间最右记录`，然后再计算这两条记录之间有多少记录。这种**通过直接访问索引对应的 `B+` 树来计算某个范围区间对应的索引记录条数的方式称之为 `index dive`**。

如果单点区间的数量非常多，`MySQL` 的查询优化器为了计算这些单点区间对应的索引记录条数要进行大量的 `index dive` 操作，成本膨胀的非常厉害。

考虑到这种情况，提供了一个系统变量 `eq_range_index_dive_limit`：

```txt
mysql> SHOW VARIABLES LIKE '%dive%';
+---------------------------+-------+
| Variable_name             | Value |
+---------------------------+-------+
| eq_range_index_dive_limit | 200   |
+---------------------------+-------+
1 row in set (0.08 sec)
```

如果 `IN` 语句中的参数个数小于 200 个的话，将使用 `index dive` 的方式计算各个单点区间对应的记录条数，如果大于或等于 200 个的话，则需要索引统计数据来进行估算。

类似于为每个表维护一份统计数据，`MySQL` 也会为表中的每一个索引维护一份统计数据：

```sql
mysql> SHOW INDEX FROM single_table;
+--------------+------------+--------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table        | Non_unique | Key_name     | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+--------------+------------+--------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| single_table |          0 | PRIMARY      |            1 | id          | A         |       9693  |     NULL | NULL   |      | BTREE      |         |               |
| single_table |          0 | idx_key2     |            1 | key2        | A         |       9693  |     NULL | NULL   | YES  | BTREE      |         |               |
| single_table |          1 | idx_key1     |            1 | key1        | A         |        968 |     NULL | NULL   | YES  | BTREE      |         |               |
| single_table |          1 | idx_key3     |            1 | key3        | A         |        799 |     NULL | NULL   | YES  | BTREE      |         |               |
| single_table |          1 | idx_key_part |            1 | key_part1   | A         |        9673 |     NULL | NULL   | YES  | BTREE      |         |               |
| single_table |          1 | idx_key_part |            2 | key_part2   | A         |        9999 |     NULL | NULL   | YES  | BTREE      |         |               |
| single_table |          1 | idx_key_part |            3 | key_part3   | A         |       10000 |     NULL | NULL   | YES  | BTREE      |         |               |
+--------------+------------+--------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
7 rows in set (0.01 sec)
```

<table> <thead> <tr> <th> 属性名 </th> <th> 描述 </th> </tr> </thead> <tbody> <tr> <td> <code> Table </code> </td> <td> 索引所属表的名称。</td> </tr> <tr> <td> <code> Non_unique </code> </td> <td> 索引列的值是否是唯一的，聚簇索引和唯一二级索引的该列值为 <code> 0 </code>，普通二级索引该列值为 <code> 1 </code>。</td> </tr> <tr> <td> <code> Key_name </code> </td> <td> 索引的名称。</td> </tr> <tr> <td> <code> Seq_in_index </code> </td> <td> 索引列在索引中的位置，从 1 开始计数。比如对于联合索引 <code> idx_key_part </code>，来说，<code> key_part1 </code>、<code> key_part2 </code> 和 <code> key_part3 </code> 对应的位置分别是 1、2、3。</td> </tr> <tr> <td> <code> Column_name </code> </td> <td> 索引列的名称。</td> </tr> <tr> <td> <code> Collation </code> </td> <td> 索引列中的值是按照何种排序方式存放的，值为 <code> A </code> 时代表升序存放，为 <code> NULL </code> 时代表降序存放。</td> </tr> <tr> <td> <code> Cardinality </code> </td> <td> 索引列中不重复值的数量。后边我们会重点看这个属性的。</td> </tr> <tr> <td> <code> Sub_part </code> </td> <td> 对于存储字符串或者字节串的列来说，有时候我们只想对这些串的前 <code> n </code> 个字符或字节建立索引，这个属性表示的就是那个 <code> n </code> 值。如果对完整的列建立索引的话，该属性的值就是 <code> NULL </code>。</td> </tr> <tr> <td> <code> Packed </code> </td> <td> 索引列如何被压缩，<code> NULL </code> 值表示未被压缩。这个属性我们暂时不了解，可以先忽略掉。</td> </tr> <tr> <td> <code> Null </code> </td> <td> 该索引列是否允许存储 <code> NULL </code> 值。</td> </tr> <tr> <td> <code> Index_type </code> </td> <td> 使用索引的类型，我们最常见的就是 <code> BTREE </code>，其实也就是 <code> B+ </code> 树索引。</td> </tr> <tr> <td> <code> Comment </code> </td> <td> 索引列注释信息。</td> </tr> <tr> <td> <code> Index_comment </code> </td> <td> 索引注释信息。</td> </tr> </tbody> </table>

其中 `Cardinality` 表示索引列中不重复值的个数，对于 InnoDB 存储引擎，这个 Cardinality 属性是一个估计值，并不是精确的。

使用索引统计数据进行成本评估主要包含以下成本：

* 使用 `SHOW TABLE STATUS` 展示出的 `Rows` 值，也就是一个表中有多少条记录。
* 使用 `SHOW INDEX` 语句展示出的 `Cardinality` 属性。

借此可以计算出平均一个值重复多少次。

$$

一个值的重复次数 \approx Rows \div Cardinality

$$

以 `single_table` 表的 `idx_key1` 索引为例，它的 `Rows` 值是 `9693`，它对应索引列 `key1` 的 `Cardinality` 值是 `968`，所以我们可以计算 `key1` 列平均单个值的重复次数就是：

$$

9693 \div 968 \approx 10（条）

$$

此时再看上边那条查询语句：

```sql

SELECT * FROM single_table WHERE key1 IN ('aa1', 'aa2', 'aa3', ... , 'zzz');

```

假设 `IN` 语句中有 20000 个参数的话，就直接使用统计数据来估算这些参数需要单点区间对应的记录条数，每个参数大约对应 `10` 条记录，所以总共需要回表的记录数就是：

$$

20000 \times 10 = 200000

$$

使用统计数据来相比 `index dive` 的方式简单，但是实际成本可能并不准确。

## 1.2. 连接查询的成本

### 1.2.1. 两表联合查询的成本

`MySQL` 中连接查询采用的是嵌套循环连接算法，驱动表会被访问一次，被驱动表可能会被访问多次，所以对于两表连接查询来说，它的查询成本由下边两个部分构成：

- 单次查询驱动表的成本
- 多次查询被驱动表的成本（具体查询多少次取决于对驱动表查询的结果集中有多少条记录）

对驱动表进行查询后得到的记录条数称之为驱动表的 `扇出`，对被驱动表的查询次数越少，连接查询的总成本也就越低。

**驱动表扇出值需要靠猜**：
- 如果使用的是全表扫描的方式执行的单表查询，那么计算驱动表扇出时需要猜满足搜索条件的记录到底有多少条。
- 如果使用的是索引执行的单表扫描，那么计算驱动表扇出的时候需要猜满足除使用到对应索引的搜索条件外的其他搜索条件的记录有多少条。

这个 **猜** 的过程称之为 [[Mysql的查询优化#4 1 10 filtered|condition filtering]]。

连接查询的成本计算公式：

$$
连接查询总成本 = 单次访问驱动表的成本 + 驱动表扇出数 \times 单次访问被驱动表的成本$$

对于左（外）连接和右（外）连接查询来说，它们的驱动表是固定的，所以想要得到最优的查询方案只需要分别为驱动表和被驱动表选择成本最低的访问方法。可是对于内连接来说，驱动表和被驱动表的位置是可以互换的，所以需要考虑两个方面的问题：

- 不同的表作为驱动表最终的查询成本可能是不同的，需要考虑最优的表连接顺序。
- 分别为驱动表和被驱动表选择成本最低的访问方法。

### 1.2.2. 多表联合的成本

对于多表连接查询，连接情况有 `n!` 种连接顺序。为了减少计算，可以采用多种剪枝算法：

- 提前结束某种顺序的成本评估
    维护当前最小的连接查询成本。如果在分析某个连接顺序的成本时，该成本已经超过当前最小的连接查询成本，那就不对该连接顺序继续往下分析了。
- 系统变量 `optimizer_search_depth`
     `optimizer_search_depth` 控制连接分析的表数量上限，不会分析超过这个数量的表连接顺序。
- 某些规则不考虑某些连接顺序
    `MySQL` 提出了一些 `启发式规则`，对于不满足这些规则的连接顺序不分析，系统变量 `optimizer_prune_level` 来控制是否启用这些启发式规则。

## 1.3. 调节成本常数

上面已经介绍了两个 `成本常数` ：
- 读取一个页面花费的成本默认是 `1.0`
- 检测一条记录是否符合搜索条件的成本默认是 `0.2`

除了这两个成本常数，还有其它的成本评估常数：

```sql
mysql> SHOW TABLES FROM mysql LIKE '%cost%';
+--------------------------+
| Tables_in_mysql (%cost%) |
+--------------------------+
| engine_cost              |
| server_cost              |
+--------------------------+
2 rows in set (0.00 sec)
```

### 1.3.1. mysql.server_cost 表

`server_cost` 表中在 `server` 层进行的一些操作对应的 `成本常数`，具体内容如下：

```sql
mysql> SELECT * FROM mysql.server_cost;
+------------------------------+------------+---------------------+---------+
| cost_name                    | cost_value | last_update         | comment |
+------------------------------+------------+---------------------+---------+
| disk_temptable_create_cost   |       NULL | 2018-01-20 12:03:21 | NULL    |
| disk_temptable_row_cost      |       NULL | 2018-01-20 12:03:21 | NULL    |
| key_compare_cost             |       NULL | 2018-01-20 12:03:21 | NULL    |
| memory_temptable_create_cost |       NULL | 2018-01-20 12:03:21 | NULL    |
| memory_temptable_row_cost    |       NULL | 2018-01-20 12:03:21 | NULL    |
| row_evaluate_cost            |       NULL | 2018-01-20 12:03:21 | NULL    |
+------------------------------+------------+---------------------+---------+
6 rows in set (0.05 sec)
```
- `cost_name`
    表示成本常数的名称。
- `cost_value`
    表示成本常数对应的值。如果该列的值为 `NULL` 的话，意味着对应的成本常数会采用默认值。
- `last_update`
    表示最后更新记录的时间。
- `comment`
    注释。

从 `server_cost` 中的内容可以看出来，目前在 `server` 层的一些操作对应的 `成本常数` 有以下几种：

<table> <thead> <tr> <th> 成本常数名称 </th> <th> 默认值 </th> <th> 描述 </th> </tr> </thead> <tbody> <tr> <td> <code> disk_temptable_create_cost </code> </td> <td> <code> 40.0 </code> </td> <td> 创建基于磁盘的临时表的成本，如果增大这个值的话会让优化器尽量少的创建基于磁盘的临时表。</td> </tr> <tr> <td> <code> disk_temptable_row_cost </code> </td> <td> <code> 1.0 </code> </td> <td> 向基于磁盘的临时表写入或读取一条记录的成本，如果增大这个值的话会让优化器尽量少的创建基于磁盘的临时表。</td> </tr> <tr> <td> <code> key_compare_cost </code> </td> <td> <code> 0.1 </code> </td> <td> 两条记录做比较操作的成本，多用在排序操作上，如果增大这个值的话会提升 <code> filesort </code> 的成本，让优化器可能更倾向于使用索引完成排序而不是 <code> filesort </code>。</td> </tr> <tr> <td> <code> memory_temptable_create_cost </code> </td> <td> <code> 2.0 </code> </td> <td> 创建基于内存的临时表的成本，如果增大这个值的话会让优化器尽量少的创建基于内存的临时表。</td> </tr> <tr> <td> <code> memory_temptable_row_cost </code> </td> <td> <code> 0.2 </code> </td> <td> 向基于内存的临时表写入或读取一条记录的成本，如果增大这个值的话会让优化器尽量少的创建基于内存的临时表。</td> </tr> <tr> <td> <code> row_evaluate_cost </code> </td> <td> <code> 0.2 </code> </td> <td> 这个就是我们之前一直使用的检测一条记录是否符合搜索条件的成本，增大这个值可能让优化器更倾向于使用索引而不是直接全表扫描。</td> </tr> </tbody> </table>


这些成本常数在 `server_cost` 中的初始值都是 `NULL`，意味着优化器会使用它们的默认值来计算某个操作的成本，如果我们想修改某个成本常数的值的话，可以更改这些配置。

### 1.3.2. mysql.engine_cost 表

`engine_cost表` 表中在存储引擎层进行的一些操作对应的 `成本常数`：

```sql

mysql> SELECT * FROM mysql.engine_cost;
+-------------+-------------+------------------------+------------+---------------------+---------+
| engine_name | device_type | cost_name              | cost_value | last_update         | comment |
+-------------+-------------+------------------------+------------+---------------------+---------+
| default     |           0 | io_block_read_cost     |       NULL | 2018-01-20 12:03:21 | NULL    |
| default     |           0 | memory_block_read_cost |       NULL | 2018-01-20 12:03:21 | NULL    |
+-------------+-------------+------------------------+------------+---------------------+---------+
2 rows in set (0.05 sec)

```

与 `server_cost` 相比，`engine_cost` 多了两个列：
- `engine_name` 列
    指成本常数适用的存储引擎名称。如果该值为 `default`，意味着对应的成本常数适用于所有的存储引擎。
- `device_type` 列
    指存储引擎使用的设备类型，这主要是为了区分常规的机械硬盘和固态硬盘，不过在 `MySQL 5.7.21` 这个版本中并没有对机械硬盘的成本和固态硬盘的成本作区分，所以该值默认是 `0`。

我们从 `engine_cost` 表中的内容可以看出来，目前支持的存储引擎成本常数只有两个：

<table> <thead> <tr> <th> 成本常数名称 </th> <th> 默认值 </th> <th> 描述 </th> </tr> </thead> <tbody> <tr> <td> <code> io_block_read_cost </code> </td> <td> <code> 1.0 </code> </td> <td> 对于 <code> InnoDB </code> 存储引擎来说，一个 <code> 页 </code> 就是一个块，不过对于 <code> MyISAM </code> 存储引擎来说，默认是以 <code> 4096 </code> 字节作为一个块的。增大这个值会加重 <code> I/O </code> 成本，可能让优化器更倾向于选择使用索引执行查询而不是执行全表扫描。</td> </tr> <tr> <td> <code> memory_block_read_cost </code> </td> <td> <code> 1.0 </code> </td> <td> 与上一个参数类似，只不过衡量的是从内存中读取一个块对应的成本。</td> </tr> </tbody> </table>

> [!question] 内存和磁盘效率一致？
> 为什么从内存中和从磁盘上读取一个块的默认成本是一样的？这主要是因为在 `MySQL` 目前的实现中，并不能准确预测某个查询需要访问的块中有哪些块已经加载到内存中，有哪些块还停留在磁盘上，所以模糊的认为不管这个块有没有加载到内存中，使用的成本都是 `1.0`，不过随着 `MySQL` 的发展，等到可以准确预测哪些块在磁盘上，那些块在内存中的那一天，这两个成本常数的默认值可能会改一改吧。

# 2. 成本统计数据的收集

查询成本的时候经常用到一些统计数据，比如通过 `SHOW TABLE STATUS` 可以看到关于表的统计数据，通过 `SHOW INDEX` 可以看到关于索引的统计数据，那么这些统计数据是怎么来的呢？

`InnoDB` 提供了两种存储统计数据的方式：
- 永久性的统计数据
    这种统计数据存储在磁盘上。
- 非永久性的统计数据
    这种统计数据存储在内存中，服务器重启之后，在某些适当的场景下才会重新收集这些统计数据。

系统变量 `innodb_stats_persistent` 来控制到底采用哪种方式去存储统计数据。

## 2.1. 基于磁盘的永久性统计数据

当我们选择把某个表以及该表索引的统计数据存放到磁盘上时，实际上是把这些统计数据存储到了两个表里：

```sql

mysql> SHOW TABLES FROM mysql LIKE 'innodb%';
+---------------------------+
| Tables_in_mysql (innodb%) |
+---------------------------+
| innodb_index_stats        |
| innodb_table_stats        |
+---------------------------+
2 rows in set (0.01 sec)

```

这两个表都位于 `mysql` 系统数据库下边，其中：
- `innodb_table_stats` 存储了关于表的统计数据，每一条记录对应着一个表的统计数据。
- `innodb_index_stats` 存储了关于索引的统计数据，每一条记录对应着一个索引的一个统计项的统计数据。

### 2.1.1. innodb_table_stats

<table> <thead> <tr> <th> 字段名 </th> <th> 描述 </th> </tr> </thead> <tbody> <tr> <td> <code> database_name </code> </td> <td> 数据库名 </td> </tr> <tr> <td> <code> table_name </code> </td> <td> 表名 </td> </tr> <tr> <td> <code> last_update </code> </td> <td> 本条记录最后更新时间 </td> </tr> <tr> <td> <code> n_rows </code> </td> <td> 表中记录的条数 </td> </tr> <tr> <td> <code> clustered_index_size </code> </td> <td> 表的聚簇索引占用的页面数量 </td> </tr> <tr> <td> <code> sum_of_other_index_sizes </code> </td> <td> 表的其他索引占用的页面数量 </td> </tr> </tbody> </table>

#### 2.1.1.1. n_rows 统计项的收集

`n_rows` 统计过程大致如下：

按照一定算法选取几个叶子节点页面，计算每个页面中主键值记录数量，然后计算平均一个页面中主键值的记录数量乘以全部叶子节点的数量就算是该表的 `n_rows` 值。

 **`n_rows` 值精确与否取决于统计时采样的页面数量**。`innodb_stats_persistent_sample_pages` 的系统变量来控制使用永久性的统计数据时，计算统计数据时采样的页面数量，该系统变量的默认值是 `20`。

可以单独设置某个表的采样页面的数量，在创建或修改表的时候通过指定 `STATS_SAMPLE_PAGES` 属性来指明该表的统计数据存储方式，默认使用系统变量 `innodb_stats_persistent_sample_pages` 的值作为该属性的值。

#### 2.1.1.2. clustered_index_size 和 sum_of_other_index_sizes 统计项的收集

这两个统计项的收集过程如下：
1. 从数据字典里找到表的各个索引对应的根页面位置。
    系统表 `SYS_INDEXES` 里存储了各个索引对应的根页面信息。
2. 从根页面的 `Page Header` 里找到叶子节点段和非叶子节点段对应的 `Segment Header`。
    在每个索引的根页面的 `Page Header` 部分都有两个字段：
    - `PAGE_BTR_SEG_LEAF` ：表示 B+树叶子段的 `Segment Header` 信息。
    - `PAGE_BTR_SEG_TOP` ：表示 B+树非叶子段的 `Segment Header` 信息。
3. 从叶子节点段和非叶子节点段的 `Segment Header` 中找到这两个段对应的 `INODE Entry` 结构。

	![](https://r2.129870.xyz/img/202209090023931.png)
4. 从对应的 `INODE Entry` 结构中可以找到该段对应所有零散的页面地址以及 `FREE`、`NOT_FULL`、`FULL` 链表的基节点。

	![](https://r2.129870.xyz/img/202209090023488.png)

5. 直接统计零散的页面有多少个，然后从那三个链表的 `List Length` 字段中读出该段占用的区的大小，每个区占用 `64` 个页，所以就可以统计出整个段占用的页面。

	![](https://r2.129870.xyz/img/202209090023072.png)

6. 分别计算聚簇索引的叶子结点段和非叶子节点段占用的页面数，它们的和就是 `clustered_index_size` 的值，按照同样的套路把其余索引占用的页面数都算出来，加起来之后就是 `sum_of_other_index_sizes` 的值。

一个段的数据在非常多时（超过 32 个页面），会以 `区` 为单位来申请空间，以区为单位申请的空间中有一些页可能并没有使用，但是在统计 `clustered_index_size` 和 `sum_of_other_index_sizes` 时都把它们算进去了，所以实际上聚簇索引和其他的索引占用的页面数可能比这两个值要小一些。

### 2.1.2. innodb_index_stats

<table> <thead> <tr> <th> 字段名 </th> <th> 描述 </th> </tr> </thead> <tbody> <tr> <td> <code> database_name </code> </td> <td> 数据库名 </td> </tr> <tr> <td> <code> table_name </code> </td> <td> 表名 </td> </tr> <tr> <td> <code> index_name </code> </td> <td> 索引名 </td> </tr> <tr> <td> <code> last_update </code> </td> <td> 本条记录最后更新时间 </td> </tr> <tr> <td> <code> stat_name </code> </td> <td> 统计项的名称 </td> </tr> <tr> <td> <code> stat_value </code> </td> <td> 对应的统计项的值 </td> </tr> <tr> <td> <code> sample_size </code> </td> <td> 为生成统计数据而采样的页面数量 </td> </tr> <tr> <td> <code> stat_description </code> </td> <td> 对应的统计项的描述 </td> </tr> </tbody> </table>

innodb_index_stats 表的每条记录代表着一个索引的一个统计项。

```sql
mysql> SELECT * FROM mysql.innodb_index_stats WHERE table_name = 'single_table';
```

| database_name | table_name   | index_name   | last_update         | stat_name    | stat_value | sample_size | stat_description                  |
|---------------|--------------|--------------|---------------------|--------------|------------|-------------|-----------------------------------|
| xiaohaizi     | single_table | PRIMARY      | 2018-12-14 14:24:46 | n_diff_pfx01 |       9693 |          20 | id                                |
| xiaohaizi     | single_table | PRIMARY      | 2018-12-14 14:24:46 | n_leaf_pages |         91 |        NULL | Number of leaf pages in the index |
| xiaohaizi     | single_table | PRIMARY      | 2018-12-14 14:24:46 | size         |         97 |        NULL | Number of pages in the index      |
| xiaohaizi     | single_table | idx_key1     | 2018-12-14 14:24:46 | n_diff_pfx01 |        968 |          28 | key1                              |
| xiaohaizi     | single_table | idx_key1     | 2018-12-14 14:24:46 | n_diff_pfx02 |      10000 |          28 | key1, id                           |
| xiaohaizi     | single_table | idx_key1     | 2018-12-14 14:24:46 | n_leaf_pages |         28 |        NULL | Number of leaf pages in the index |
| xiaohaizi     | single_table | idx_key1     | 2018-12-14 14:24:46 | size         |         29 |        NULL | Number of pages in the index      |
| xiaohaizi     | single_table | idx_key2     | 2018-12-14 14:24:46 | n_diff_pfx01 |      10000 |          16 | key2                              |
| xiaohaizi     | single_table | idx_key2     | 2018-12-14 14:24:46 | n_leaf_pages |         16 |        NULL | Number of leaf pages in the index |
| xiaohaizi     | single_table | idx_key2     | 2018-12-14 14:24:46 | size         |         17 |        NULL | Number of pages in the index      |
| xiaohaizi     | single_table | idx_key3     | 2018-12-14 14:24:46 | n_diff_pfx01 |        799 |          31 | key3                              |
| xiaohaizi     | single_table | idx_key3     | 2018-12-14 14:24:46 | n_diff_pfx02 |      10000 |          31 | key3, id                           |
| xiaohaizi     | single_table | idx_key3     | 2018-12-14 14:24:46 | n_leaf_pages |         31 |        NULL | Number of leaf pages in the index |
| xiaohaizi     | single_table | idx_key3     | 2018-12-14 14:24:46 | size         |         32 |        NULL | Number of pages in the index      |
| xiaohaizi     | single_table | idx_key_part | 2018-12-14 14:24:46 | n_diff_pfx01 |       9673 |          64 | key_part1                         |
| xiaohaizi     | single_table | idx_key_part | 2018-12-14 14:24:46 | n_diff_pfx02 |       9999 |          64 | key_part1, key_part2               |
| xiaohaizi     | single_table | idx_key_part | 2018-12-14 14:24:46 | n_diff_pfx03 |      10000 |          64 | key_part1, key_part2, key_part3     |
| xiaohaizi     | single_table | idx_key_part | 2018-12-14 14:24:46 | n_diff_pfx04 |      10000 |          64 | key_part1, key_part2, key_part3, id  |
| xiaohaizi     | single_table | idx_key_part | 2018-12-14 14:24:46 | n_leaf_pages |         64 |        NULL | Number of leaf pages in the index |
| xiaohaizi     | single_table | idx_key_part | 2018-12-14 14:24:46 | size         |         97 |        NULL | Number of pages in the index      |

关键属性含义如下：

1. `index_name` 记录索引名称
2. 针对 `index_name` 列相同的记录，`stat_name` 表示针对该索引的统计项名称，`stat_value` 展示的是该索引在该统计项上的值，`stat_description` 指的是来描述该统计项的含义：
    - `n_leaf_pages` ：表示该索引的叶子节点占用多少页面。
    - `size` ：表示该索引共占用多少页面。
    - `n_diff_pfx_NN` ：表示对应的索引列不重复的值有多少。

        对于普通的二级索引，并不能保证它的索引列值是唯一的，所以此时只有在索引列上加上主键值才可以区分两条索引列值都一样的二级索引记录。因此对于普通索引会多出一条主键的 part 部分。

3. 在计算某些索引列中包含多少不重复值时，需要对一些叶子节点页面进行采样，`sample_size` 列就表明了采样的页面数量是多少。

### 2.1.3. 定期更新统计数据

统计数据有两种更新的方式：
- 开启 `innodb_stats_auto_recalc`。
    系统变量 `innodb_stats_auto_recalc` 决定着服务器是否自动重新计算统计数据，它的默认值是 `ON`，每个表都可以单独设置此属性。
    
    在自动重新统计开关打开的前提下，**如果某个表的发生变动的记录数量超过了表大小的 `10%`，那么服务器会重新进行一次统计数据的计算**，并且更新 `innodb_table_stats` 和 `innodb_index_stats` 表，该更新过程是异步发生的。

- 手动调用 `ANALYZE TABLE` 语句来更新统计信息
    
    ANALYZE TABLE 语句会立即重新计算统计数据，因此尽量在系统负载比较低的情况下执行该命令。

## 2.2. 基于内存的非永久性统计数据

当我们把系统变量 `innodb_stats_persistent` 的值设置为 `OFF` 时，之后创建的表的统计数据默认就都是非永久性的了，或者我们直接在创建表或修改表时设置 `STATS_PERSISTENT` 属性的值为 `0`，那么该表的统计数据就是非永久性的了。

与永久性的统计数据不同，非永久性的统计数据采样的页面数量是由 `innodb_stats_transient_sample_pages` 控制的，这个系统变量的默认值是 `8`。

另外，由于非永久性的统计数据经常更新，所以导致 `MySQL` 查询优化器计算查询成本的时候依赖的是经常变化的统计数据，也就会生成经常变化的执行计划。

# 3. 基于规则的优化

## 3.1. 条件化简

### 3.1.1. 移除多余括号

```txt
((a = 5 AND b = c) OR ((a > c) AND (c < 5)))
(a = 5 and b = c) OR (a > c AND c < 5)
```

### 3.1.2. 常量传递（constant_propagation）

有时候某个表达式是某个列和某个常量做等值匹配：

```txt
a = 5
```

当这个表达式和其他涉及列 `a` 的表达式使用 `AND` 连接起来时，可以将其他表达式中的 `a` 的值替换为 `5`，比如这样：

```css
a = 5 AND b > a
```

就可以被转换为：

```css
a = 5 AND b > 5
```

### 3.1.3. 等值传递（equality_propagation）

有时候多个列之间存在等值匹配的关系，比如这样：

```css
a = b and b = c and c = 5
```

这个表达式可以被简化为：

```css
a = 5 and b = 5 and c = 5
```

### 3.1.4. 移除没用的条件（trivial_condition_removal）

对于一些明显永远为 `TRUE` 或者 `FALSE` 的表达式，优化器会移除掉它们。

### 3.1.5. 表达式计算

在查询开始执行之前，如果表达式中只包含常量的话，它的值会被先计算出来。

### 3.1.6. HAVING 子句和 WHERE 子句的合并

如果查询语句中没有出现诸如 `SUM`、`MAX` 等等的聚集函数以及 `GROUP BY` 子句，优化器就把 `HAVING` 子句和 `WHERE` 子句合并起来。

### 3.1.7. 常量表检测

`MySQL` 认为以下两种查询运行的特别快：
- 查询的表中一条记录没有，或者只有一条记录。
    由于 InnoDB 的统计数据数据不准确，所以这一条规则只能适用于使用 Memory 或者 MyISAM 存储引擎的表。
- 使用主键等值匹配或者唯一二级索引列等值匹配作为搜索条件来查询某个表。

这两种方式查询的表称之为 `常量表`（英文名：`constant tables`）。优化器在分析一个查询语句时，先首先执行常量表查询，然后把查询中涉及到该表的条件全部替换成常数，最后再分析其余表的查询成本：

```sql
SELECT * FROM table1 INNER JOIN table2
    ON table1.column1 = table2.column2 
    WHERE table1.primary_key = 1;
```

在这个查询中 `table1` 表相当于 `常量表`，在分析对 `table2` 表的查询成本之前，就会执行对 `table1` 表的查询，并把查询中涉及 `table1` 表的条件都替换掉：

```sql
SELECT table1表记录的各个字段的常量值, table2.* FROM table1 INNER JOIN table2 
    ON table1表column1列的常量值 = table2.column2;
```

## 3.2. 外连接消除

 `外连接` 由于连接顺序固定无法优化表的连接顺序，但是可以优化值不为 NULL 的查询，只要我们**在搜索条件中指定关于被驱动表相关列的值不为 `NULL`，那么外连接中在被驱动表中找不到符合 `ON` 子句条件的驱动表记录也就被排除出最后的结果集了**。例如以下查询：

```sql
mysql> SELECT * FROM t1 LEFT JOIN t2 ON t1.m1 = t2.m2 WHERE t2.n2 IS NOT NULL;
+------+------+------+------+
| m1   | n1   | m2   | n2   |
+------+------+------+------+
|    2 | b    |    2 | b    |
|    3 | c    |    3 | c    |
+------+------+------+------+
2 rows in set (0.01 sec)
```

我们把这种在外连接查询中，指定的 `WHERE` 子句中包含被驱动表中的列不为 `NULL` 值的条件称之为 `空值拒绝`（英文名：`reject-NULL`）。**在被驱动表的 WHERE 子句符合空值拒绝的条件后，外连接和内连接可以相互转换。这种转换带来的好处就是查询优化器可以通过评估表的不同连接顺序的成本，选出成本最低的那种连接顺序来执行查询。**

## 3.3. 子查询优化

### 3.3.1. 子查询分类

#### 3.3.1.1. 按返回结果集分类

因为子查询本身也算是一个查询，所以可以按照它们返回的不同结果集类型而把这些子查询分为不同的类型：

- 标量子查询
    只返回一个单一值的子查询称之为 `标量子查询`。
- 行子查询
    只返回一条记录的子查询。
- 列子查询
    查询一个列的多条数据。
- 表子查询
    既包含很多条记录，又包含很多个列。

#### 3.3.1.2. 按与外层查询关系分类

- 不相关子查询
	子查询可以单独运行出结果而不依赖于外层查询的值，称之为 `不相关子查询`。
- 相关子查询
	子查询的执行需要依赖于外层查询的值，称之为 `相关子查询`。

### 3.3.2. 子查询使用

#### 3.3.2.1. 布尔表达式

使用 =、>、<、>=、<=、<>、!=、<=> 作为布尔表达式的操作符。

#### 3.3.2.2. \[NOT] IN/ANY/SOME/ALL 子查询

- `IN` 或者 `NOT IN`
    用来判断某个操作数在不在由子查询结果集组成的集合中。
- `ANY/SOME`（`ANY` 和 `SOME` 是同义词）
    子查询结果集中存在某个值和给定的操作数做 `comparison_operator` 比较结果为 `TRUE`，那么整个表达式的结果就为 `TRUE`，否则整个表达式的结果就为 `FALSE`。    
- `ALL`
    子查询结果集中所有的值和给定的操作数做 `comparison_operator` 比较结果为 `TRUE`，那么整个表达式的结果就为 `TRUE`，否则整个表达式的结果就为 `FALSE`。

#### 3.3.2.3. EXISTS 子查询

仅仅需要判断子查询的结果集中是否有记录，而不在乎它的记录具体数据，可以使用 `EXISTS` 或者 `NOT EXISTS` 。

### 3.3.3. 子查询注意事项

- 子查询必须用小括号扩起来。
- 在 `SELECT` 子句中的子查询必须是标量子查询。
- 在想要得到标量子查询或者行子查询，但又不能保证子查询的结果集只有一条记录时，应该使用 `LIMIT 1` 语句来限制记录数量。
- 对于 `[NOT] IN/ANY/SOME/ALL` 子查询来说，子查询中不允许有 `LIMIT` 语句。
	正因为 `[NOT] IN/ANY/SOME/ALL` 子查询不支持 `LIMIT` 语句，所以子查询中的这些语句也就是多余的了：
    - `ORDER BY` 子句
    - `DISTINCT` 语句
    - 没有聚集函数以及 `HAVING` 子句的 `GROUP BY` 子句。
    对于这些冗余的语句，查询优化器会直接进行忽略。
- 不允许在一条语句中增删改某个表的记录时同时还对该表进行子查询。
    ```sql
    DELETE FROM t1 WHERE m1 < (SELECT MAX(m1) FROM t1);
    ```

### 3.3.4. 无优化的子查询执行方式

对于未经过任何优化的子查询执行过程如下：
- 不相关子查询
    ```sql   
    SELECT * FROM s1 
        WHERE key1 IN (SELECT common_field FROM s2);
    ```

    1. 先单独执行 `(SELECT common_field FROM s2)` 这个子查询。
    2. 然后在将上一步子查询得到的结果当作外层查询的参数再执行外层查询 `SELECT * FROM s1 WHERE key1 IN (...)`。
- 相关子查询
  
    ```sql
    SELECT * FROM s1 
        WHERE key1 IN (SELECT common_field FROM s2 WHERE s1.key2 = s2.key2);
    ```

    这个查询中的子查询中出现了 `s1.key2 = s2.key2` 这样的条件，意味着该子查询的执行依赖着外层查询的值，这个查询的执行方式是这样的：
    1. 先从外层查询中获取一条记录
    2. 从上一步骤中获取的那条记录中找出子查询中涉及到的值，然后执行子查询
    3. 最后根据子查询的查询结果来检测外层查询 `WHERE` 子句的条件是否成立，如果成立，就把外层查询的那条记录加入到结果集，否则就丢弃。
    4. 重复执行直到所有数据扫描完毕

对于标量子查询和行子查询，它们的实现方式就是无优化的执行方式。**因为只有一条数据，所以先将该数据查出来再进行外层查询就是最优的方式**。

### 3.3.5. 物化表优化 IN 子查询

#### 3.3.5.1. 物化表的提出

对于不相关的 `IN` 子查询来说，如果子查询的结果集中的记录条数很少，那么把子查询和外层查询分别看成两个单独的单表查询效率还是蛮高的，但是如果单独执行子查询后的结果集太多的话，就会导致这些问题：

- 结果集太多，可能内存中都放不下。
- 对于外层查询来说，如果子查询的结果集太多，那就意味着 `IN` 子句中的参数特别多，这就导致：
    - 无法有效的使用索引，只能对外层查询进行全表扫描。
    - 在对外层查询执行全表扫描时，由于 `IN` 子句中的参数太多，这会导致检测一条记录是否符合和 `IN` 子句中的参数匹配花费的时间太长。

因此实际上**未直接将不相关子查询的结果集当作外层查询的参数，而是将该结果集写入一个临时表里**：

1. 临时表的列即子查询结果集
2. 写入临时表的记录会被去重
    对临时表中记录的所有列建立主键或者唯一索引即可去重。
3. 为临时表建立索引
	一般情况下子查询结果集不会太大，所以会为它建立基于内存的使用 `Memory` 存储引擎的临时表，而且会为该表建立哈希索引。
	
	如果子查询的结果集非常大，超过了系统变量 `tmp_table_size` 或者 `max_heap_table_size`，临时表会转而使用基于磁盘的存储引擎来保存结果集中的记录，索引类型也对应转变为 `B+` 树索引。

上述将子查询结果集中的记录保存到临时表的过程称之为 `物化`（英文名：`Materialize`）。存储子查询结果集的临时表称之为 `物化表`。**正因为物化表中的记录都建立了索引（基于内存的物化表有哈希索引，基于磁盘的有 B+树索引），通过索引执行 `IN` 语句判断某个操作数在不在子查询结果集中变得非常快，从而提升了子查询语句的性能**。

#### 3.3.5.2. 物化表转连接

```sql

SELECT * FROM s1 
    WHERE key1 IN (SELECT common_field FROM s2 WHERE key3 = 'a');
    
```

对于物化表的查询可以从下边两种角度来看待：

- 从表 `s1` 的角度来看待，：对于 `s1` 表中的每条记录来说，如果该记录的 `key1` 列的值在子查询对应的物化表中，则该记录会被加入最终的结果集：

	![](https://r2.129870.xyz/img/202209130043313.png)

- 从子查询物化表的角度来看待：对于子查询物化表的每个值来说，如果能在 `s1` 表中找到对应的 `key1` 列的值与该值相等的记录，那么就把这些记录加入到最终的结果集：

    ![](https://r2.129870.xyz/img/202209130043593.png)


即查询的本质就相当于表 `s1` 和子查询物化表 `materialized_table` 进行内连接：

```sql
SELECT s1.* FROM s1 INNER JOIN materialized_table ON key1 = m_val;
```

转化成内连接之后，**查询优化器可以评估不同连接顺序需要的成本是多少，选取成本最低的那种查询方式执行查询**：

- 如果使用 `s1` 表作为驱动表的话，总查询成本由下边几个部分组成：
    - 物化子查询时需要的成本
    - 扫描 `s1` 表时的成本
    - s1 表中的记录数量 × 通过 `m_val = xxx` 对 `materialized_table` 表进行单表访问的成本
- 如果使用 `materialized_table` 表作为驱动表的话，总查询成本由下边几个部分组成：
    - 物化子查询时需要的成本
    - 扫描物化表时的成本
    - 物化表中的记录数量 × 通过 `key1 = xxx` 对 `s1` 表进行单表访问的成本

`MySQL` 查询优化器会通过运算来选择上述成本更低的方案来执行查询。

### 3.3.6. semi-join 半连接优化子查询

#### 3.3.6.1. 半连接查询概念

将子查询进行物化之后再执行查询都会有建立临时表的成本，**能不能不进行物化操作直接把子查询转换为连接呢**？

```sql
SELECT * FROM s1 
    WHERE key1 IN (SELECT common_field FROM s2 WHERE key3 = 'a');
```

这个查询可以理解成：对于 `s1` 表中的某条记录，如果能在 `s2` 表中找到一条或多条记录，这些记录的 `common_field` 的值等于 `s1` 表记录的 `key1` 列的值，那么该条 `s1` 表的记录就会被加入到最终的结果集。这个过程其实和把 `s1` 和 `s2` 两个表连接起来的效果很像：

```sql
SELECT s1.* FROM s1 INNER JOIN s2 
    ON s1.key1 = s2.common_field 
    WHERE s2.key3 = 'a';
```

只不过不能保证对于 `s1` 表的某条记录来说，在 `s2` 表中有多少条记录满足 `s1.key1 = s2.common_field` 这个条件，不过我们可以分三种情况：

- 没有任何记录满足 这个条件，该记录不会加入到最后的结果集
- 有且只有 1 条记录满足这个条件，该记录会被加入最终的结果集。
- 至少有 2 条记录满足 这个条件，该记录会被多次加入最终的结果集

由于会出现多次加入结果集的情况，针对这类问题提出了一个新概念：`半连接`（`semi-join`）。

将 `s1` 表和 `s2` 表进行半连接的意思就是：对于 `s1` 表的某条记录来说，**我们只关心在 `s2` 表中是否存在与之匹配的记录，而不关心具体有多少条记录与之匹配**，最终的结果集中只保留 `s1` 表的记录。


```sql
SELECT s1.* FROM s1 SEMI JOIN s2
    ON s1.key1 = s2.common_field
    WHERE key3 = 'a';
```

#### 3.3.6.2. 半连接查询实现

概念是有了，怎么实现这种所谓的 `半连接` 呢？

##### 3.3.6.2.1. Table pullout （子查询中的表上拉）

当子查询的查询列表处只有主键或者唯一索引列时，可以直接把子查询中的表 `上拉` 到外层查询的 `FROM` 子句中，并把子查询中的搜索条件合并到外层查询的搜索条件中：

```sql
SELECT * FROM s1 
	WHERE key2 IN (SELECT key2 FROM s2 WHERE key3 = 'a');
```

由于 `key2` 列是 `s2` 表的唯一二级索引列，所以我们可以直接把 `s2` 表上拉到外层查询的 `FROM` 子句中，并且把子查询中的搜索条件合并到外层查询的搜索条件中，上拉之后的查询就是这样的：

```sql
SELECT s1.* FROM s1 INNER JOIN s2 
	ON s1.key2 = s2.key2 
	WHERE s2.key3 = 'a';
```

##### 3.3.6.2.2. DuplicateWeedout execution strategy （重复值消除）

对于这个查询来说：

```sql
SELECT * FROM s1 
	WHERE key1 IN (SELECT common_field FROM s2 WHERE key3 = 'a');
```

转换为半连接查询后，`s1` 表中的某条记录可能在 `s2` 表中有多条匹配的记录，所以该条记录可能多次被添加到最后的结果集中，为了消除重复，可以建立一个临时表：

```sql
CREATE TABLE tmp (
	id PRIMARY KEY
);
```

这样在执行连接查询的过程中，每当某条 `s1` 表中的记录要加入结果集时，就首先把这条记录的 `id` 和临时表数据做对比，这种使用临时表消除 `semi-join` 结果集中的重复值的方式称之为 `DuplicateWeedout`。

##### 3.3.6.2.3. LooseScan execution strategy （松散扫描）

```sql
SELECT * FROM s1 
	WHERE key3 IN (SELECT key1 FROM s2 WHERE key1 > 'a' AND key1 < 'b');
```

在子查询中，对于 `s2` 表的访问可以使用到 `key1` 列的索引，而恰好子查询的查询列表处就是 `key1` 列，这样在将该查询转换为半连接查询后，如果将 `s2` 作为驱动表执行查询的话，那么执行过程就是这样：

![](https://r2.129870.xyz/img/202209130054903.png)

在 `s2` 表的 `idx_key1` 索引中，值为 `'aa'` 的二级索引记录一共有 3 条，那么只需要取第一条的值到 `s1` 表中查找 `s1.key3 = 'aa'` 的记录，如果能在 `s1` 表中找到对应的记录，那么就把对应的记录加入到结果集。这种**虽然是扫描索引，但只取值相同的记录的第一条去做匹配操作的方式称之为 `松散扫描` **。

##### 3.3.6.2.4. Semi-join Materialization execution strategy

之前介绍的先把外层查询的 `IN` 子句中的不相关子查询进行物化，然后再进行外层查询的表和物化表的连接本质上也算是一种 `semi-join`，只不过由于物化表中没有重复的记录，所以可以直接将子查询转为连接查询。

##### 3.3.6.2.5. FirstMatch execution strategy （首次匹配）

`FirstMatch` 是一种最原始的半连接执行方式，和 [[Mysql的查询优化#3 3 4 无优化的子查询执行方式|无优化的子查询执行方式]]是一样的。先取一条外层查询的中的记录，然后到子查询的表中寻找符合匹配条件的记录，如果能找到一条，则将该外层查询的记录放入最终的结果集并且停止查找更多匹配的记录，如果找不到则把该外层查询的记录丢弃掉；然后再开始取下一条外层查询中的记录，重复上边这个过程。

对于某些使用 `IN` 语句的相关子查询，它也可以很方便的转为半连接：

```sql
SELECT * FROM s1 
    WHERE key1 IN (SELECT common_field FROM s2 WHERE s1.key3 = s2.key3);
```

```sql
SELECT s1.* FROM s1 SEMI JOIN s2 
    ON s1.key1 = s2.common_field AND s1.key3 = s2.key3;
```

然后就可以使用 `DuplicateWeedout`、`LooseScan`、`FirstMatch` 等半连接执行策略来执行查询，当然，如果子查询的查询列表处只有主键或者唯一二级索引列，还可以直接使用 `table pullout` 的策略来执行查询。但是**由于相关子查询并不是一个独立的查询，所以不能转换为物化表来执行查询**。

#### 3.3.6.3. 半连接适用条件

当然，并不是所有包含 `IN` 子查询的查询语句都可以转换为 `semi-join`，只有形如这样的查询才可以被转换为 `semi-join` ：

```sql
SELECT ... FROM outer_tables 
    WHERE expr IN (SELECT ... FROM inner_tables ...) AND ...
```

或者这样的形式也可以：

```sql
SELECT ... FROM outer_tables 
    WHERE (oe1, oe2, ...) IN (SELECT ie1, ie2, ... FROM inner_tables ...) AND ...
```

即只有符合下边这些条件的子查询才可以被转换为 `semi-join` ：

- 该子查询必须是和 `IN` 语句组成的布尔表达式，并且在外层查询的 `WHERE` 或者 `ON` 子句中出现。
- 外层查询也可以有其他的搜索条件，只不过和 `IN` 子查询的搜索条件必须使用 `AND` 连接起来。
- 该子查询必须是一个单一的查询，不能是由若干查询由 `UNION` 连接起来的形式。
- 该子查询不能包含 `GROUP BY` 或者 `HAVING` 语句或者聚集函数。

#### 3.3.6.4. 不适用半连接的子查询优化

对于一些不能将子查询转位 `semi-join` 的情况，典型的比如下边这几种：
- 外层查询的 WHERE 条件中有其他搜索条件与 IN 子查询组成的布尔表达式使用 `OR` 连接起来
-   使用 `NOT IN` 而不是 `IN` 的情况
-   在 `SELECT` 子句中的 IN 子查询的情况
-   子查询中包含 `GROUP BY`、`HAVING` 或者聚集函数的情况
-   子查询中包含 `UNION` 的情况

对于这类情况的优化方式如下：

- 对于不相关子查询来说，可以尝试把它们物化之后再参与查询
- 不管子查询是相关的还是不相关的，都可以把 `IN` 子查询尝试转为 `EXISTS` 子查询
    其实对于任意一个 IN 子查询来说，都可以被转为 `EXISTS` 子查询，通用的例子如下：

    ```sql
    outer_expr IN (SELECT inner_expr FROM ... WHERE subquery_where)
    ```

    可以被转换为：

    ```sql
    EXISTS (SELECT inner_expr FROM ... WHERE subquery_where AND outer_expr=inner_expr)
    ```
    
    当然这个过程中有一些特殊情况，比如在 `outer_expr` 或者 `inner_expr` 值为 `NULL` 的情况下就比较特殊。因为有 `NULL` 值作为操作数的表达式结果往往是 `NULL`，而 `EXISTS` 子查询的结果肯定是 `TRUE` 或者 `FASLE`。
    
    由于大部分使用 `IN` 子查询的场景是把它放在 `WHERE` 或者 `ON` 子句中，而 `WHERE` 或者 `ON` 子句是不区分 `NULL` 和 `FALSE` 的，所以只要 `IN` 子查询是放在 `WHERE` 或者 `ON` 子句中的，那么 `IN -> EXISTS` 的转换就是没问题的。
    
    说了这么多，为啥要转换呢？这是因为**不转换的话可能用不到索引**：

    ```sql
    SELECT * FROM s1
        WHERE key1 IN (SELECT key3 FROM s2 where s1.common_field = s2.common_field) 
            OR key2 > 1000;   
    ```
    
    这个查询中的子查询是一个相关子查询，而且子查询执行的时候不能使用到索引，但是将它转为 `EXISTS` 子查询后却可以使用到索引：
    
    ```sql
    SELECT * FROM s1
        WHERE EXISTS (SELECT 1 FROM s2 where s1.common_field = s2.common_field AND s2.key3 = s1.key1) 
            OR key2 > 1000; 
    ```
    
    转为 `EXISTS` 子查询时便可以使用到 `s2` 表的 `idx_key3` 索引了。

    **需要注意的是，如果 `IN` 子查询不满足转换为 `semi-join` 的条件，又不能转换为物化表或者转换为物化表的成本太大，那么它就会被转换为 `EXISTS` 查询**。

## 3.4. 派生表查询优化

把子查询放在外层查询的 `FROM` 子句后，那么这个子查询的结果相当于一个 `派生表`：

```sql
SELECT * FROM  (
        SELECT id AS d_id,  key3 AS d_key3 FROM s2 WHERE key1 = 'a'
    ) AS derived_s1 WHERE d_key3 = 'a'; 
```

对于含有 `派生表` 的查询，`MySQL` 提供了两种执行策略：
- 派生表物化。
    可以将派生表的结果集写到一个内部的临时表中，然后就把这个物化表当作普通表一样参与查询。在对派生表进行物化时使用了一种称为 **`延迟物化` 的策略，也就是在查询中真正使用到派生表时才回去尝试物化派生表**：

	```sql
	    SELECT * FROM (
	            SELECT * FROM s1 WHERE key1 = 'a'
	        ) AS derived_s1 INNER JOIN s2
	        ON derived_s1.key1 = s2.key1
	        WHERE s2.key2 = 1;
	```

    只有当 `s2` 表中找出满足 `s2.key2 = 1` 的记录才会对内部的查询进行物化。
- 将派生表和外层的表合并

    ```sql
    SELECT * FROM (SELECT * FROM s1 WHERE key1 = 'a') AS derived_s1;
    ```

    本质和下边这个语句是等价的：

    ```sql
    SELECT * FROM s1 WHERE key1 = 'a';
    ```
    
    对于一些稍微复杂的包含派生表的语句：
    
    ```sql
    SELECT * FROM (
            SELECT * FROM s1 WHERE key1 = 'a'
        ) AS derived_s1 INNER JOIN s2
        ON derived_s1.key1 = s2.key1
        WHERE s2.key2 = 1;
    ```
    
    可以将派生表与外层查询的表合并，然后将派生表中的搜索条件放到外层查询的搜索条件中：
    
    ```sql
    SELECT * FROM s1 INNER JOIN s2 
        ON s1.key1 = s2.key1
        WHERE s1.key1 = 'a' AND s2.key2 = 1;
    ```
    
    这样通过将外层查询和派生表合并的方式成功的消除了派生表，也就意味着我们没必要再付出创建和访问临时表的成本了。
    
    可是并不是所有带有派生表的查询都能被成功的和外层查询合并，当派生表中有这些语句就不可以和外层查询合并：
    -   聚集函数，比如 `MAX()`、`MIN()`、`SUM()` 等
    -   DISTINCT
    -   GROUP BY
    -   HAVING
    -   LIMIT
    -   UNION 或者 UNION ALL
    -   派生表对应的子查询的 `SELECT` 子句中含有另一个子查询
    -   ... 还有些不常用的情况

所以 `MySQL` 在执行带有派生表的时候，优先尝试把派生表和外层查询合并掉，如果不行的话，再把派生表物化掉执行查询。

# 4. explain 详解

## 4.1. explain 结果的含义

<table> <thead> <tr> <th> 列名 </th> <th> 描述 </th> </tr> </thead> <tbody> <tr> <td> <code> id </code> </td> <td> 在一个大的查询语句中每个 <code> SELECT </code> 关键字都对应一个唯一的 <code> id </code> </td> </tr> <tr> <td> <code> select_type </code> </td> <td> <code> SELECT </code> 关键字对应的那个查询的类型 </td> </tr> <tr> <td> <code> table </code> </td> <td> 表名 </td> </tr> <tr> <td> <code> partitions </code> </td> <td> 匹配的分区信息 </td> </tr> <tr> <td> <code> type </code> </td> <td> 针对单表的访问方法 </td> </tr> <tr> <td> <code> possible_keys </code> </td> <td> 可能用到的索引 </td> </tr> <tr> <td> <code> key </code> </td> <td> 实际上使用的索引 </td> </tr> <tr> <td> <code> key_len </code> </td> <td> 实际使用到的索引长度 </td> </tr> <tr> <td> <code> ref </code> </td> <td> 当使用索引列等值查询时，与索引列进行等值匹配的对象信息 </td> </tr> <tr> <td> <code> rows </code> </td> <td> 预估的需要读取的记录条数 </td> </tr> <tr> <td> <code> filtered </code> </td> <td> 某个表经过搜索条件过滤后剩余记录条数的百分比 </td> </tr> <tr> <td> <code> Extra </code> </td> <td> 一些额外的信息 </td> </tr> </tbody> </table>

### 4.1.1. `table`

不论查询语句多复杂，**到最后也是需要对每个表进行单表访问的**，EXPLAIN 语句输出的每条记录都对应着某个单表的访问方法，该条记录的 table 列代表着该表的表名。

### 4.1.2. `id`

查询语句中每出现一个 `SELECT` 关键字，就会为它分配一个唯一的 `id` 值。

对于连接查询来说，一个 `SELECT` 关键字后边的 `FROM` 子句中可以跟随多个表，所以在连接查询的执行计划中，每个表都会对应一条记录，但是这些记录的 id 值都是相同的。

对于包含子查询的查询语句来说，就可能涉及多个 `SELECT` 关键字，所以在包含子查询的查询语句的执行计划中，每个 `SELECT` 关键字都会对应一个唯一的 `id` 值。

查询优化器可能对涉及子查询的查询语句进行重写从而转换为连接查询。如果**查询语句是一个子查询，但是执行计划中子查询涉及表对应的记录的 `id` 值相同的话，这就表明了查询优化器将子查询转换为了连接查询**。

### 4.1.3. `select_type`

每一个 `SELECT` 关键字代表的小查询都定义了一个称之为 `select_type` 的属性，只要知道了某个小查询的 `select_type` 属性，就知道了这个小查询在整个大查询中扮演了一个什么角色：

<table> <thead> <tr> <th> 名称 </th> <th> 描述 </th> </tr> </thead> <tbody> <tr> <td> <code> SIMPLE </code> </td> <td> Simple SELECT (not using UNION or subqueries) </td> </tr> <tr> <td> <code> PRIMARY </code> </td> <td> Outermost SELECT </td> </tr> <tr> <td> <code> UNION </code> </td> <td> Second or later SELECT statement in a UNION </td> </tr> <tr> <td> <code> UNION RESULT </code> </td> <td> Result of a UNION </td> </tr> <tr> <td> <code> SUBQUERY </code> </td> <td> First SELECT in subquery </td> </tr> <tr> <td> <code> DEPENDENT SUBQUERY </code> </td> <td> First SELECT in subquery, dependent on outer query </td> </tr> <tr> <td> <code> DEPENDENT UNION </code> </td> <td> Second or later SELECT statement in a UNION, dependent on outer query </td> </tr> <tr> <td> <code> DERIVED </code> </td> <td> Derived table </td> </tr> <tr> <td> <code> MATERIALIZED </code> </td> <td> Materialized subquery </td> </tr> <tr> <td> <code> UNCACHEABLE SUBQUERY </code> </td> <td> A subquery for which the result cannot be cached and must be re-evaluated for each row of the outer query </td> </tr> <tr> <td> <code> UNCACHEABLE UNION </code> </td> <td> The second or later select in a UNION that belongs to an uncacheable subquery (see UNCACHEABLE SUBQUERY) </td> </tr> </tbody> </table>

- `SIMPLE`
    查询语句中不包含 `UNION` 或者子查询的查询都算作是 `SIMPLE` 类型。
- 联合查询
    - `PRIMARY`
        对于包含 `UNION`、`UNION ALL` 或者子查询的大查询来说，它是由几个小查询组成的，其中最左边的那个查询的 `select_type` 值就是 `PRIMARY`。
    - `UNION`
        对于包含 `UNION` 或者 `UNION ALL` 的大查询来说，它是由几个小查询组成的，其中除了最左边的那个小查询以外，其余的小查询的 `select_type` 值就是 `UNION`。
    - `UNION RESULT`
        `MySQL` 选择使用临时表来完成 `UNION` 查询的去重工作，针对该临时表的查询的 `select_type` 就是 `UNION RESULT`。
    - `DEPENDENT UNION`
        在包含 `UNION` 或者 `UNION ALL` 的大查询中，如果各个小查询都依赖于外层查询的话，那除了最左边的那个小查询之外，其余的小查询的 `select_type` 的值就是 `DEPENDENT UNION`。
- 子查询 
    - `SUBQUERY`
        如果包含子查询的查询语句不能够转为对应的 `semi-join` 的形式，并且该子查询是不相关子查询，并且查询优化器决定采用将该子查询物化的方案来执行该子查询时，该子查询的第一个 `SELECT` 关键字代表的那个查询的 `select_type` 就是 `SUBQUERY`。被物化的子查询只需要执行一遍。
    - `DEPENDENT SUBQUERY`
        如果包含子查询的查询语句不能够转为对应的 `semi-join` 的形式，并且该子查询是相关子查询，则该子查询的第一个 `SELECT` 关键字代表的那个查询的 `select_type` 就是 `DEPENDENT SUBQUERY`。
    
        select_type 为 `DEPENDENT SUBQUERY` 的查询可能会被执行多次。
    -  `MATERIALIZED`
        当查询优化器在执行包含子查询的语句时，选择将子查询物化之后与外层查询进行连接查询时，该子查询对应的 `select_type` 属性就是 `MATERIALIZED`。
- 派生表查询 `DERIVED`
    对于采用物化的方式执行的包含派生表的查询，该派生表对应的子查询的 `select_type` 就是 `DERIVED`。
- `UNCACHEABLE SUBQUERY`
    不常用。
- `UNCACHEABLE UNION`
    不常用。

### 4.1.4. `partitions`

查询的表归属于哪一个 [[Mysql大纲#2.7. Mysql 分区表|分区]]。

### 4.1.5. `type`

执行计划的一条记录就代表着 `MySQL` 对某个表的执行查询时的[[Mysql大纲#3.5. 索引的访问类型|访问方法]]，其中的 `type` 列就表明了这个访问方法。

### 4.1.6. `possible_keys` 和 `key`

`possible_keys` 列表示在某个查询语句中，对某个表执行单表查询时可能用到的索引有哪些，`key` 列表示实际用到的索引有哪些（有哪些而不是有哪个，因为可能使用[[Mysql大纲#3 7 索引合并|索引合并]]从而使用到多个索引）。

possible_keys 列中的值并不是越多越好，可能使用的索引越多，查询优化器计算查询成本时就得花费更长时间，所以如果可以的话，尽量删除那些用不到的索引。

### 4.1.7. `key_len`

`key_len` 列表示当优化器决定使用某个索引执行查询时，该索引记录的最大长度。

### 4.1.8. `ref`

当使用索引列等值匹配的条件去执行查询时，也就是在[[Mysql大纲#3 5 索引的访问类型|访问类型]]是 `const`、`eq_ref`、`ref`、`ref_or_null`、`unique_subquery`、`index_subquery` 其中之一时，`ref` 列展示的就是与索引列作等值匹配的一个常数或者是某个列。

### 4.1.9. `rows`

如果查询优化器决定使用全表扫描的方式对某个表执行查询时，执行计划的 `rows` 列就代表预计需要扫描的行数，如果使用索引来执行查询时，执行计划的 `rows` 列就代表预计扫描的索引记录行数。

### 4.1.10. `filtered`

```sql
mysql> EXPLAIN SELECT * FROM s1 WHERE key1 > 'z' AND common_field = 'a';
```

| id | select_type | table | partitions | type  | possible_keys | key      | key_len | ref  | rows | filtered | Extra                              |
|----|-------------|-------|------------|-------|---------------|----------|---------|------|------|----------|------------------------------------|
|  1 | SIMPLE      | s1    | NULL       | range | idx_key1      | idx_key1 | 303     | NULL |  266 |    10.00 | Using index condition; Using where |

`filtered` 列就代表查询优化器预测在这 `266` 条记录中，**有多少条记录满足其余的搜索条件**，也就是 `common_field = 'a'` 这个条件的百分比。

### 4.1.11. `Extra`

`Extra` 列是用来说明一些额外信息的，可以通过这些额外信息来更准确的理解 `MySQL` 到底将如何执行给定的查询语句。`MySQL` 提供的额外信息有好几十个，我们只挑一些平时常见的或者比较重要的额外信息介绍。

#### 4.1.11.1. No tables used

当查询语句的没有 `FROM` 子句时将会提示该额外信息。

#### 4.1.11.2. Impossible WHERE

查询语句的 `WHERE` 子句永远为 `FALSE` 时将会提示该额外信息。

#### 4.1.11.3. No matching min/max row

当查询列表处有 `MIN` 或者 `MAX` 聚集函数，但是并没有符合 `WHERE` 子句中的搜索条件的记录时，将会提示该额外信息。

#### 4.1.11.4. Using index

查询列表以及搜索条件中只包含属于某个索引的列，也就是在可以使用[[Mysql大纲#3.2.1. 索引覆盖|索引覆盖]]的情况下，在 `Extra` 列将会提示该额外信息。

#### 4.1.11.5. Using index condition

如果在查询语句的执行过程中将要使用 [[Mysql大纲#3 2 2 索引下推|索引条件下推]]这个特性，在 `Extra` 列中将会显示 `Using index condition`。

#### 4.1.11.6. `Using where`

当某个搜索条件**需要在 server 层进行判断时**，在 `Extra` 列中会提示 `Using where`。

对于聚簇索引来说，是用不到 `索引条件下推` 特性的（索引下推是为了减少回表），所以所有的搜索条件都得在 `server层` 进行处理。

#### 4.1.11.7. `Using join buffer (Block Nested Loop)`

在连接查询执行过程中，当被驱动表不能有效的利用索引加快访问速度，`MySQL` 一般会为其分配一块名叫 `join buffer` 的内存块来加快查询速度，也就是我们所讲的 [[Mysql大纲#2 2 2 2 被驱动表无索引的情况|基于块的嵌套循环算法]]。

#### 4.1.11.8. `Not exists`

当我们使用左（外）连接时，如果 `WHERE` 子句中包含要求被驱动表的某个列等于 `NULL` 值的搜索条件，而且那个列又是不允许存储 `NULL` 值的，那么在该表的执行计划的 `Extra` 列就会提示 `Not exists` 额外信息。

#### 4.1.11.9. `Using intersect(...)`、`Using union(...)` 和 `Using sort_union(...)`

如果执行计划的 `Extra` 列出现了 `Using intersect(...)` 提示，说明准备使用 `Intersect` [[Mysql大纲#3 7 索引合并|索引合并]]的方式执行查询，括号中的 `...` 表示需要进行索引合并的索引名称；如果出现了 `Using union(...)` 提示，说明准备使用 `Union` 索引合并的方式执行查询；出现了 `Using sort_union(...)` 提示，说明准备使用 `Sort-Union` 索引合并的方式执行查询。

#### 4.1.11.10. `Zero limit`

当我们的 `LIMIT` 子句的参数为 `0` 时，表示不打算从表中读出任何记录，将会提示该额外信息.

#### 4.1.11.11. `Using filesort`

有一些情况下对结果集中的记录进行排序是可以使用到索引的，但是很多情况下排序操作无法使用到索引，只能在内存中（记录较少的时候）或者磁盘中（记录较多的时候）进行排序。这种在内存中或者磁盘上进行排序的方式统称为文件排序。如果某个查询需要使用文件排序的方式执行查询，就会在执行计划的 `Extra` 列中显示 `Using filesort` 提示。

#### 4.1.11.12. `Using temporary`

在许多查询的执行过程中，`MySQL` 可能会借助临时表来完成一些功能，比如去重、排序之类的。如果查询中使用到了内部的临时表，在执行计划的 `Extra` 列将会显示 `Using temporary` 提示。

#### 4.1.11.13. `Start temporary, End temporary`

查询优化器会优先尝试将 `IN` 子查询转换成 [[Mysql的查询优化#3 3 6 semi-join 半连接优化子查询|semi-join]]。而 `semi-join` 又有好多种执行策略，当执行策略为 [[Mysql的查询优化#3 3 6 2 2 DuplicateWeedout execution strategy （重复值消除）|DuplicateWeedout]] 时，也就是**通过建立临时表来实现为外层查询中的记录进行去重操作时，驱动表查询执行计划的 `Extra` 列将显示 `Start temporary` 提示，被驱动表查询执行计划的 `Extra` 列将显示 `End temporary` 提示**。

#### 4.1.11.14. `LooseScan`

在将 `In` 子查询转为 `semi-join` 时，如果采用的是 [[Mysql的查询优化#3 3 6 2 3 LooseScan execution strategy （松散扫描）|LooseScan]] 执行策略，则在驱动表执行计划的 `Extra` 列就是显示 `LooseScan` 提示。

#### 4.1.11.15. `FirstMatch(tbl_name)`

在将 `In` 子查询转为 `semi-join` 时，如果采用的是 [[Mysql的查询优化#3 3 6 2 5 FirstMatch execution strategy （首次匹配）|FirstMatch]] 执行策略，则在被驱动表执行计划的 `Extra` 列就是显示 `FirstMatch(tbl_name)` 提示。

## 4.2. JSON 结构的 explain 结果

我们上边介绍的 `EXPLAIN` 语句输出中缺少了一个衡量执行计划好坏的重要属性 —— 成本。在 `EXPLAIN` 单词和真正的查询语句中间加上 `FORMAT=JSON`。

这样我们就可以得到一个 `json` 格式的执行计划，里边儿包含该计划花费的成本。

## 4.3. Extented EXPLAIN

在使用 `EXPLAIN` 语句查看了某个查询的执行计划后，紧接着还可以使用 `SHOW WARNINGS` 语句查看与这个查询的执行计划有关的一些扩展信息，比如这样：

```sql

mysql> EXPLAIN SELECT s1.key1, s2.key1 FROM s1 LEFT JOIN s2 ON s1.key1 = s2.key1 WHERE s2.common_field IS NOT NULL;
+----+-------------+-------+------------+------+---------------+----------+---------+-------------------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key      | key_len | ref               | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+----------+---------+-------------------+------+----------+-------------+
|  1 | SIMPLE      | s2    | NULL       | ALL  | idx_key1      | NULL     | NULL    | NULL              | 9954 |    90.00 | Using where |
|  1 | SIMPLE      | s1    | NULL       | ref  | idx_key1      | idx_key1 | 303     | xiaohaizi.s2.key1 |    1 |   100.00 | Using index |
+----+-------------+-------+------------+------+---------------+----------+---------+-------------------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)

mysql> SHOW WARNINGS\G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `xiaohaizi`.`s1`.`key1` AS `key1`,`xiaohaizi`.`s2`.`key1` AS `key1` from `xiaohaizi`.`s1` join `xiaohaizi`.`s2` where ((`xiaohaizi`.`s1`.`key1` = `xiaohaizi`.`s2`.`key1`) and (`xiaohaizi`.`s2`.`common_field` is not null))
1 row in set (0.00 sec)

```

`SHOW WARNINGS` 展示出来的信息有三个字段，分别是 `Level`、`Code`、`Message`。最常见的就是 `Code` 为 `1003` 的信息，`Message` 字段展示的信息类似于查询优化器将我们的查询语句重写后的语句。

比如查询本来是一个左（外）连接查询，但是有一个 `s2.common_field IS NOT NULL` 的条件，着就会导致查询优化器把左（外）连接查询优化为内连接查询，从 `SHOW WARNINGS` 的 `Message` 字段也可以看出来，原本的 `LEFT JOIN` 已经变成了 `JOIN`。

 `Message` 字段展示的信息类似于查询优化器将原始的查询语句重写后的语句，并不是等价于，也就是说 `Message` 字段展示的信息并不是标准的查询语句，在很多情况下并不能直接运行，它只能作为帮助我们理解查 `MySQL` 将如何执行查询语句的一个参考依据而已。

## 4.4. optimizer trace

`optimizer trace` 可以查看优化器生成执行计划的整个过程，这个功能的开启与关闭由系统变量 `optimizer_trace` 决定，默认是关闭的：

```sql
mysql> SHOW VARIABLES LIKE 'optimizer_trace';
+-----------------+--------------------------+
| Variable_name   | Value                    |
+-----------------+--------------------------+
| optimizer_trace | enabled=off,one_line=off |
+-----------------+--------------------------+
1 row in set (0.02 sec)
```


打开后，查询语句执行完成可以到 `information_schema` 数据库下的 `OPTIMIZER_TRACE` 表中查看完整的优化过程。这个 `OPTIMIZER_TRACE` 表有 4 个列，分别是：

- `QUERY` ：查询语句。
- `TRACE` ：表示优化过程的 JSON 格式文本。
- `MISSING_BYTES_BEYOND_MAX_MEM_SIZE` ：由于优化过程可能会输出很多，如果超过某个限制时，多余的文本将不会被显示，这个字段展示了被忽略的文本字节数。
- `INSUFFICIENT_PRIVILEGES` ：表示是否没有权限查看优化过程，默认值是 0，只有某些特殊情况下才会是 `1`。

使用 `optimizer trace` 功能的步骤总结如下：

1. 打开optimizer trace功能 (默认情况下它是关闭的):
    `SET optimizer_trace="enabled=on";`
2. 进行查询
    `SELECT ...;` 
3. 从OPTIMIZER_TRACE表中查看上一个查询的优化过程
    `SELECT * FROM information_schema.OPTIMIZER_TRACE;`
