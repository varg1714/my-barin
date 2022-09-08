#数据库 #Mysql 

# 1. 基于成本的优化

`MySQL` 执行一个查询可以有不同的执行方案，它会选择其中成本最低，或者说代价最低的那种方案去真正的执行查询。不过我们之前对 `成本` 的描述是非常模糊的，其实在 `MySQL` 中一条查询语句的执行成本是由下边这两个方面组成的：

- `I/O` 成本
    
    我们的表经常使用的 `MyISAM`、`InnoDB` 存储引擎都是将数据和索引都存储到磁盘上的，当我们想查询表中的记录时，需要先把数据或者索引加载到内存中然后再操作。这个从磁盘到内存这个加载的过程损耗的时间称之为 `I/O` 成本。
    
- `CPU` 成本
    
    读取以及检测记录是否满足对应的搜索条件、对结果集进行排序等这些操作损耗的时间称之为 `CPU` 成本。
    

对于 `InnoDB` 存储引擎来说，页是磁盘和内存之间交互的基本单位，设计 `MySQL` 的大叔规定读取一个页面花费的成本默认是 `1.0`，读取以及检测一条记录是否符合搜索条件的成本默认是 `0.2`。`1.0`、`0.2` 这些数字称之为 `成本常数`，这两个成本常数我们最常用到，除此之外还有一切其他的成本常数。

## 1.1. 单表查询成本

在一条单表查询语句真正执行之前，`MySQL` 的查询优化器会找出执行该语句所有可能使用的方案，对比之后找出成本最低的方案，这个成本最低的方案就是所谓的 `执行计划`，之后才会调用存储引擎提供的接口真正的执行查询，这个过程总结一下就是这样：

1.  根据搜索条件，找出所有可能使用的索引。

2.  计算全表扫描的代价。

3.  计算使用不同索引执行查询的代价。

1.  对比各种执行方案的代价，找出成本最低的那一个。

### 1.1.1. 全表查询成本

对于 `InnoDB` 存储引擎来说，全表扫描的意思就是把聚簇索引中的记录都依次和给定的搜索条件做一下比较，把符合搜索条件的记录加入到结果集，所以需要将聚簇索引对应的页面加载到内存中，然后再检测记录是否符合搜索条件。由于查询成本= `I/O` 成本+ `CPU` 成本，所以计算全表扫描的代价需要两个信息：

-   聚簇索引占用的页面数

-   该表中的记录数

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

虽然出现了很多统计选项，但我们目前只关心两个：

- `Rows`

	本选项表示表中的记录条数。对于使用 `MyISAM` 存储引擎的表来说，该值是准确的，对于使用 `InnoDB` 存储引擎的表来说，该值是一个估计值。

- `Data_length`

	本选项表示表占用的存储空间字节数。使用 `MyISAM` 存储引擎的表来说，该值就是数据文件的大小，对于使用 `InnoDB` 存储引擎的表来说，该值就相当于聚簇索引占用的存储空间大小，也就是说可以这样计算该值的大小：

	```txt
	
	Data_length = 聚簇索引的页面数量 x 每个页面的大小
	
	```

	我们的 `single_table` 使用默认 `16KB` 的页面大小，而上边查询结果显示 `Data_length` 的值是 `1589248`，所以我们可以反向来推导出 `聚簇索引的页面数量` ：

	```txt

	聚簇索引的页面数量 = 1589248 ÷ 16 ÷ 1024 = 97
	
	```

我们现在已经得到了聚簇索引占用的页面数量以及该表记录数的估计值，所以就可以计算全表扫描成本了，但是设计 `MySQL` 的大叔在真实计算成本时会进行一些 `微调`。现在可以看一下全表扫描成本的计算过程：

* `I/O` 成本
    
    ```txt
    
    97 x 1.0 + 1.1 = 98.1
    
    ```
    
    `97` 指的是聚簇索引占用的页面数，`1.0` 指的是加载一个页面的成本常数，后边的 `1.1` 是一个微调值，我们不用在意。
    
* `CPU` 成本：
    
    ```txt
    
    9693 x 0.2 + 1.0 = 1939.6
    
    ```
    
    `9693` 指的是统计数据中表的记录数，对于 `InnoDB` 存储引擎来说是一个估计值，`0.2` 指的是访问一条记录所需的成本常数，后边的 `1.0` 是一个微调值，我们不用在意。
    
* 总成本：
    
    ```txt
    
    98.1 + 1939.6 = 2037.7
    
    ```
    

综上所述，对于 `single_table` 的全表扫描所需的总成本就是 `2037.7`。

### 1.1.2. 使用索引的成本

#### 1.1.2.1. 基于索引页估算记录条数

假设 `idx_key2` 对应的搜索条件是：`key2 > 10 AND key2 < 1000`，也就是说对应的范围区间就是：`(10, 1000)`。

对于使用 `二级索引 + 回表` 方式的查询，设计 `MySQL` 的大叔计算这种查询的成本依赖两个方面的数据：

* 范围区间数量
    
    不论某个范围区间的二级索引到底占用了多少页面，查询优化器粗暴的认为读取索引的一个范围区间的 `I/O` 成本和读取一个页面是相同的。本例中使用 `idx_key2` 的范围区间只有一个：`(10, 1000)`，所以相当于访问这个范围区间的二级索引付出的 `I/O` 成本就是：
    
    ```txt
    
    1 x 1.0 = 1.0
    
    ```
    
* 需要回表的记录数
    
    优化器需要计算二级索引的某个范围区间到底包含多少条记录，对于本例来说就是要计算 `idx_key2` 在 `(10, 1000)` 这个范围区间中包含多少二级索引记录，计算过程是这样的：
    
    1. 先根据 `key2 > 10` 这个条件访问一下 `idx_key2` 对应的 `B+` 树索引，找到满足 `key2 > 10` 这个条件的第一条记录，我们把这条记录称之为 `区间最左记录`。在 `B+` 数树中定位一条记录的过程是常数级别的，所以这个过程的性能消耗是可以忽略不计的。

    2. 然后再根据 `key2 < 1000` 这个条件继续从 `idx_key2` 对应的 `B+` 树索引中找出最后一条满足这个条件的记录，我们把这条记录称之为 `区间最右记录`，这个过程的性能消耗也可以忽略不计的。

    3. 如果 `区间最左记录` 和 `区间最右记录` 相隔不太远（在 `MySQL 5.7.21` 这个版本里，只要相隔不大于 10 个页面即可），那就可以精确统计出满足 `key2 > 10 AND key2 < 1000` 条件的二级索引记录条数。否则只沿着 `区间最左记录` 向右读 10 个页面，计算平均每个页面中包含多少记录，然后用这个平均值乘以 `区间最左记录` 和 `区间最右记录` 之间的页面数量就可以了。那么问题又来了，怎么估计 `区间最左记录` 和 `区间最右记录` 之间有多少个页面呢？解决这个问题还得回到 `B+` 树索引的结构中来：
        
        ![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209082349305.png)

        如图，我们假设 `区间最左记录` 在 `页b` 中，`区间最右记录` 在 `页c` 中，那么我们想计算 `区间最左记录` 和 `区间最右记录` 之间的页面数量就相当于计算 `页b` 和 `页c` 之间有多少页面，而每一条 `目录项记录` 都对应一个数据页，所以计算 `页b` 和 `页c` 之间有多少页面就相当于计算它们父节点（也就是页 a）中对应的目录项记录之间隔着几条记录。在一个页面中统计两条记录之间有几条记录的成本就贼小了。
        
        如果 `页b` 和 `页c` 之间的页面实在太多，以至于 `页b` 和 `页c` 对应的目录项记录都不在一个页面中？则可以继续递归，也就是再统计 `页b` 和 `页c` 对应的目录项记录所在页之间有多少个页面。之前我们说过一个 `B+` 树有 4 层高已经很了不得了，所以这个统计过程也不是很耗费性能。

    知道了如何统计二级索引某个范围区间的记录数之后，就需要回到现实问题中来，根据上述算法测得 `idx_key2` 在区间 `(10, 1000)` 之间大约有 `95` 条记录。读取这 `95` 条二级索引记录需要付出的 `CPU` 成本就是：
    
    ```txt
    
    95 x 0.2 + 0.01 = 19.01
    
    ```
    
    其中 `95` 是需要读取的二级索引记录条数，`0.2` 是读取一条记录成本常数，`0.01` 是微调。
    
    在通过二级索引获取到记录之后，还需要干两件事儿：
    
    *   根据这些记录里的主键值到聚簇索引中做回表操作
        
        `MySQL` 评估回表操作的 `I/O` 成本很豪放，他们认为每次回表操作都相当于访问一个页面，也就是说二级索引范围区间有多少记录，就需要进行多少次回表操作，也就是需要进行多少次页面 `I/O`。我们上边统计了使用 `idx_key2` 二级索引执行查询时，预计有 `95` 条二级索引记录需要进行回表操作，所以回表操作带来的 `I/O` 成本就是：
        
        ```txt
        
        95 x 1.0 = 95.0
        
        ```
        
        其中 `95` 是预计的二级索引记录数，`1.0` 是一个页面的 `I/O` 成本常数。
        
    *   回表操作后得到的完整用户记录，然后再检测其他搜索条件是否成立
        
        回表操作的本质就是通过二级索引记录的主键值到聚簇索引中找到完整的用户记录，然后再检测除 `key2 > 10 AND key2 < 1000` 这个搜索条件以外的搜索条件是否成立。因为我们通过范围区间获取到二级索引记录共 `95` 条，也就对应着聚簇索引中 `95` 条完整的用户记录，读取并检测这些完整的用户记录是否符合其余的搜索条件的 `CPU` 成本如下：
        
        在定位到这些完整的用户记录后，需要检测除 `key2 > 10 AND key2 < 1000` 这个搜索条件以外的搜索条件是否成立，这个比较过程花费的 `CPU` 成本就是：
        
        ```txt
        
        95 x 0.2 = 19.0
        
        ```
        
        其中 `95` 是待检测记录的条数，`0.2` 是检测一条记录是否符合给定的搜索条件的成本常数。
        

所以本例中使用 `idx_key2` 执行查询的成本就如下所示：

*   `I/O` 成本：
    
    ```txt
    
    1.0 + 95 x 1.0 = 96.0 (范围区间的数量 + 预估的二级索引记录条数)
    
    ```
    
*   `CPU` 成本：
    
    ```txt
    
    95 x 0.2 + 0.01 + 95 x 0.2 = 38.01 （读取二级索引记录的成本 + 读取并检测回表后聚簇索引记录的成本）
    
    ```
    

综上所述，使用 `idx_key2` 执行查询的总成本就是：

```txt

96.0 + 38.01 = 134.01

```

#### 1.1.2.2. 基于索引统计数据估算记录条数

有时候使用索引执行查询时会有许多单点区间，比如使用 `IN` 语句就很容易产生非常多的单点区间，比如下边这个查询：

```sql

SELECT * FROM single_table WHERE key1 IN ('aa1', 'aa2', 'aa3', ... , 'zzz');

```

很显然，这个查询可能使用到的索引就是 `idx_key1`，由于这个索引并不是唯一二级索引，所以并不能确定一个单点区间对应的二级索引记录的条数有多少，需要我们去计算。

计算方式就是先获取索引对应的 `B+` 树的 `区间最左记录` 和 `区间最右记录`，然后再计算这两条记录之间有多少记录（记录条数少的时候可以做到精确计算，多的时候只能估算）。`MySQL` 把这种通过直接访问索引对应的 `B+` 树来计算某个范围区间对应的索引记录条数的方式称之为 `index dive`。

有零星几个单点区间的话，使用 `index dive` 的方式去计算这些单点区间对应的记录数也不是什么问题，可如果单点区间的数量非常多，`MySQL` 的查询优化器为了计算这些单点区间对应的索引记录条数，要进行大量的 `index dive` 操作，这性能损耗可就大了，搞不好计算这些单点区间对应的索引记录条数的成本比直接全表扫描的成本都大了。

`MySQL` 考虑到了这种情况，所以提供了一个系统变量 `eq_range_index_dive_limit`，我们看一下在 `MySQL 5.7.21` 中这个系统变量的默认值：

```txt

mysql> SHOW VARIABLES LIKE '%dive%';
+---------------------------+-------+
| Variable_name             | Value |
+---------------------------+-------+
| eq_range_index_dive_limit | 200   |
+---------------------------+-------+
1 row in set (0.08 sec)

```

也就是说如果我们的 `IN` 语句中的参数个数小于 200 个的话，将使用 `index dive` 的方式计算各个单点区间对应的记录条数，如果大于或等于 200 个的话，可就不能使用 `index dive` 了，要使用所谓的索引统计数据来进行估算。

像会为每个表维护一份统计数据一样，`MySQL` 也会为表中的每一个索引维护一份统计数据：

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

其实我们现在最在意的是 `Cardinality` 属性，`Cardinality` 直译过来就是 `基数` 的意思，表示索引列中不重复值的个数。比如对于一个一万行记录的表来说，某个索引列的 `Cardinality` 属性是 `10000`，那意味着该列中没有重复的值，如果 `Cardinality` 属性是 `1` 的话，就意味着该列的值全部是重复的。

不过需要注意的是，对于 InnoDB 存储引擎来说，使用 SHOW INDEX 语句展示出来的某个索引列的 Cardinality 属性是一个估计值，并不是精确的。

前边说道，当 `IN` 语句中的参数个数大于或等于系统变量 `eq_range_index_dive_limit` 的值的话，就不会使用 `index dive` 的方式计算各个单点区间对应的索引记录条数，而是使用索引统计数据，这里所指的 `索引统计数据` 指的是这两个值：

* 使用 `SHOW TABLE STATUS` 展示出的 `Rows` 值，也就是一个表中有多少条记录。

* 使用 `SHOW INDEX` 语句展示出的 `Cardinality` 属性。

结合上一个 `Rows` 统计数据，我们可以针对索引列，计算出平均一个值重复多少次。

```txt

一个值的重复次数 ≈ Rows ÷ Cardinality

```

以 `single_table` 表的 `idx_key1` 索引为例，它的 `Rows` 值是 `9693`，它对应索引列 `key1` 的 `Cardinality` 值是 `968`，所以我们可以计算 `key1` 列平均单个值的重复次数就是：

```txt

9693 ÷ 968 ≈ 10（条）

```

此时再看上边那条查询语句：

```sql

SELECT * FROM single_table WHERE key1 IN ('aa1', 'aa2', 'aa3', ... , 'zzz');

```

假设 `IN` 语句中有 20000 个参数的话，就直接使用统计数据来估算这些参数需要单点区间对应的记录条数了，每个参数大约对应 `10` 条记录，所以总共需要回表的记录数就是：

```txt

20000 x 10 = 200000

```

使用统计数据来计算单点区间对应的索引记录条数可比 `index dive` 的方式简单多了，但是它的致命弱点就是：不精确！。使用统计数据算出来的查询成本与实际所需的成本可能相差非常大。

## 1.2. 连接查询的成本

### 1.2.1. 两表联合查询的成本

`MySQL` 中连接查询采用的是嵌套循环连接算法，驱动表会被访问一次，被驱动表可能会被访问多次，所以对于两表连接查询来说，它的查询成本由下边两个部分构成：

- 单次查询驱动表的成本
 
- 多次查询被驱动表的成本（具体查询多少次取决于对驱动表查询的结果集中有多少条记录）

我们把对驱动表进行查询后得到的记录条数称之为驱动表的 `扇出`（英文名：`fanout`）。很显然驱动表的扇出值越小，对被驱动表的查询次数也就越少，连接查询的总成本也就越低。当查询优化器想计算整个连接查询所使用的成本时，就需要计算出驱动表的扇出值。

**驱动表扇出值时需要靠猜**：

- 如果使用的是全表扫描的方式执行的单表查询，那么计算驱动表扇出时需要猜满足搜索条件的记录到底有多少条。

- 如果使用的是索引执行的单表扫描，那么计算驱动表扇出的时候需要猜满足除使用到对应索引的搜索条件外的其他搜索条件的记录有多少条。

设计 `MySQL` 的大叔把这个 `猜` 的过程称之为 `condition filtering`。

连接查询的成本计算公式是这样的：

```txt

连接查询总成本 = 单次访问驱动表的成本 + 驱动表扇出数 x 单次访问被驱动表的成本

```

对于左（外）连接和右（外）连接查询来说，它们的驱动表是固定的，所以想要得到最优的查询方案只需要分别为驱动表和被驱动表选择成本最低的访问方法。可是对于内连接来说，驱动表和被驱动表的位置是可以互换的，所以需要考虑两个方面的问题：

- 不同的表作为驱动表最终的查询成本可能是不同的，也就是需要考虑最优的表连接顺序。

- 然后分别为驱动表和被驱动表选择成本最低的访问方法。

### 1.2.2. 多表联合的成本

有 `n` 个表进行连接，`MySQL` 查询优化器要每一种连接顺序的成本都计算一遍么？那可是 `n!` 种连接顺序呀。其实真的是要都算一遍，不过 `MySQL` 想了很多办法减少计算非常多种连接顺序的成本的方法：

- 提前结束某种顺序的成本评估
    
    `MySQL` 在计算各种链接顺序的成本之前，会维护一个全局的变量，这个变量表示当前最小的连接查询成本。如果在分析某个连接顺序的成本时，该成本已经超过当前最小的连接查询成本，那就压根儿不对该连接顺序继续往下分析了。

	比方说 A、B、C 三个表进行连接，已经得到连接顺序 `ABC` 是当前的最小连接成本，比方说 `10.0`，在计算连接顺序 `BCA` 时，发现 `B` 和 `C` 的连接成本就已经大于 `10.0` 时，就不再继续往后分析 `BCA` 这个连接顺序的成本了。
    
- 系统变量 `optimizer_search_depth`
    
    为了防止无穷无尽的分析各种连接顺序的成本，设计 `MySQL` 的大叔们提出了 `optimizer_search_depth` 系统变量，如果连接表的个数小于该值，那么就继续穷举分析每一种连接顺序的成本，否则只对与 `optimizer_search_depth` 值相同数量的表进行穷举分析。很显然，该值越大，成本分析的越精确，越容易得到好的执行计划，但是消耗的时间也就越长，否则得到不是很好的执行计划，但可以省掉很多分析连接成本的时间。
    
- 根据某些规则压根儿就不考虑某些连接顺序
    
    即使是有上边两条规则的限制，但是分析多个表不同连接顺序成本花费的时间还是会很长，所以设计 `MySQL` 的大叔干脆提出了一些所谓的 `启发式规则`（就是根据以往经验指定的一些规则），凡是不满足这些规则的连接顺序压根儿就不分析，这样可以极大的减少需要分析的连接顺序的数量，但是也可能造成错失最优的执行计划。他们提供了一个系统变量 `optimizer_prune_level` 来控制到底是不是用这些启发式规则。

## 1.3. 调节成本常数

我们前边已经介绍了两个 `成本常数` ：

- 读取一个页面花费的成本默认是 `1.0`
- 检测一条记录是否符合搜索条件的成本默认是 `0.2`

其实除了这两个成本常数，`MySQL` 还支持好多呢，它们被存储到了 `mysql` 数据库（的两个表中：

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

### 1.3.1. mysql. server_cost 表

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

这个就是我们之前一直使用的检测一条记录是否符合搜索条件的成本，增大这个值可能让优化器更倾向于使用索引而不是直接全表扫描。

这些成本常数在 `server_cost` 中的初始值都是 `NULL`，意味着优化器会使用它们的默认值来计算某个操作的成本，如果我们想修改某个成本常数的值的话，可以更改这些配置。

### 1.3.2. mysql. engine_cost 表

`engine_cost表` 表中在存储引擎层进行的一些操作对应的 `成本常数`，具体内容如下：

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

<table> <thead> <tr> <th> 成本常数名称 </th> <th> 默认值 </th> <th> 描述 </th> </tr> </thead> <tbody> <tr> <td> <code> io_block_read_cost </code> </td> <td> <code> 1.0 </code> </td> <td> 从磁盘上读取一个块对应的成本。请注意我使用的是 <code> 块 </code>，而不是 <code> 页 </code> 这个词儿。对于 <code> InnoDB </code> 存储引擎来说，一个 <code> 页 </code> 就是一个块，不过对于 <code> MyISAM </code> 存储引擎来说，默认是以 <code> 4096 </code> 字节作为一个块的。增大这个值会加重 <code> I/O </code> 成本，可能让优化器更倾向于选择使用索引执行查询而不是执行全表扫描。</td> </tr> <tr> <td> <code> memory_block_read_cost </code> </td> <td> <code> 1.0 </code> </td> <td> 与上一个参数类似，只不过衡量的是从内存中读取一个块对应的成本。</td> </tr> </tbody> </table>


与上一个参数类似，只不过衡量的是从内存中读取一个块对应的成本。

大家看完这两个成本常数的默认值是不是有些疑惑，怎么从内存中和从磁盘上读取一个块的默认成本是一样的？这主要是因为在 `MySQL` 目前的实现中，并不能准确预测某个查询需要访问的块中有哪些块已经加载到内存中，有哪些块还停留在磁盘上，所以设计 `MySQL` 的大叔们很粗暴的认为不管这个块有没有加载到内存中，使用的成本都是 `1.0`，不过随着 `MySQL` 的发展，等到可以准确预测哪些块在磁盘上，那些块在内存中的那一天，这两个成本常数的默认值可能会改一改吧。

# 2. 成本统计数据的收集

查询成本的时候经常用到一些统计数据，比如通过 `SHOW TABLE STATUS` 可以看到关于表的统计数据，通过 `SHOW INDEX` 可以看到关于索引的统计数据，那么这些统计数据是怎么来的呢？它们是以什么方式收集的呢？本章将聚焦于 `InnoDB` 存储引擎的统计数据收集策略，看完本章大家就会明白为啥前边老说 `InnoDB` 的统计信息是不精确的估计值了。

`InnoDB` 提供了两种存储统计数据的方式：

- 永久性的统计数据

    这种统计数据存储在磁盘上，也就是服务器重启之后这些统计数据还在。

- 非永久性的统计数据

    这种统计数据存储在内存中，当服务器关闭时这些这些统计数据就都被清除掉了，等到服务器重启之后，在某些适当的场景下才会重新收集这些统计数据。

 `MySQL` 提供了系统变量 `innodb_stats_persistent` 来控制到底采用哪种方式去存储统计数据。在 `MySQL 5.6.6` 之前，`innodb_stats_persistent` 的值默认是 `OFF`，也就是说 `InnoDB` 的统计数据默认是存储到内存的，之后的版本中 `innodb_stats_persistent` 的值默认是 `ON`，也就是统计数据默认被存储到磁盘中。

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

为啥老强调 `n_rows` 这个统计项的值是估计值呢？现在就来揭晓答案。`InnoDB` 统计一个表中有多少行记录的套路是这样的：

按照一定算法（并不是纯粹随机的）选取几个叶子节点页面，计算每个页面中主键值记录数量，然后计算平均一个页面中主键值的记录数量乘以全部叶子节点的数量就算是该表的 `n_rows` 值（真实的计算过程比这个稍微复杂一些）。

可以看出来这个 `n_rows` 值精确与否取决于统计时采样的页面数量。`MySQL` 很贴心的为我们准备了一个名为 `innodb_stats_persistent_sample_pages` 的系统变量来控制使用永久性的统计数据时，计算统计数据时采样的页面数量。该值设置的越大，统计出的 `n_rows` 值越精确，但是统计耗时也就最久；该值设置的越小，统计出的 `n_rows` 值越不精确，但是统计耗时特别少。所以在实际使用是需要我们去权衡利弊，该系统变量的默认值是 `20`。

我们前边说过，不过 `InnoDB` 默认是以表为单位来收集和存储统计数据的，我们也可以单独设置某个表的采样页面的数量，设置方式就是在创建或修改表的时候通过指定 `STATS_SAMPLE_PAGES` 属性来指明该表的统计数据存储方式：

如果我们在创建表的语句中并没有指定 `STATS_SAMPLE_PAGES` 属性的话，将默认使用系统变量 `innodb_stats_persistent_sample_pages` 的值作为该属性的值。

#### 2.1.1.2. clustered_index_size 和 sum_of_other_index_sizes 统计项的收集

统计这两个数据需要大量用到 `InnoDB` 表空间的知识。

这两个统计项的收集过程如下：

1. 从数据字典里找到表的各个索引对应的根页面位置。
    
    系统表 `SYS_INDEXES` 里存储了各个索引对应的根页面信息。
    
2. 从根页面的 `Page Header` 里找到叶子节点段和非叶子节点段对应的 `Segment Header`。
    
    在每个索引的根页面的 `Page Header` 部分都有两个字段：
    
    - `PAGE_BTR_SEG_LEAF` ：表示 B+树叶子段的 `Segment Header` 信息。

    - `PAGE_BTR_SEG_TOP` ：表示 B+树非叶子段的 `Segment Header` 信息。

3. 从叶子节点段和非叶子节点段的 `Segment Header` 中找到这两个段对应的 `INODE Entry` 结构。

	![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209090023931.png)
4. 从对应的 `INODE Entry` 结构中可以找到该段对应所有零散的页面地址以及 `FREE`、`NOT_FULL`、`FULL` 链表的基节点。

	![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209090023488.png)

5. 直接统计零散的页面有多少个，然后从那三个链表的 `List Length` 字段中读出该段占用的区的大小，每个区占用 `64` 个页，所以就可以统计出整个段占用的页面。

	![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209090023072.png)

6. 分别计算聚簇索引的叶子结点段和非叶子节点段占用的页面数，它们的和就是 `clustered_index_size` 的值，按照同样的套路把其余索引占用的页面数都算出来，加起来之后就是 `sum_of_other_index_sizes` 的值。

这里需要大家注意一个问题，我们说一个段的数据在非常多时（超过 32 个页面），会以 `区` 为单位来申请空间，这里头的问题是以区为单位申请空间中有一些页可能并没有使用，但是在统计 `clustered_index_size` 和 `sum_of_other_index_sizes` 时都把它们算进去了，所以说聚簇索引和其他的索引占用的页面数可能比这两个值要小一些。

### 2.1.2. innodb_index_stats

<table> <thead> <tr> <th> 字段名 </th> <th> 描述 </th> </tr> </thead> <tbody> <tr> <td> <code> database_name </code> </td> <td> 数据库名 </td> </tr> <tr> <td> <code> table_name </code> </td> <td> 表名 </td> </tr> <tr> <td> <code> index_name </code> </td> <td> 索引名 </td> </tr> <tr> <td> <code> last_update </code> </td> <td> 本条记录最后更新时间 </td> </tr> <tr> <td> <code> stat_name </code> </td> <td> 统计项的名称 </td> </tr> <tr> <td> <code> stat_value </code> </td> <td> 对应的统计项的值 </td> </tr> <tr> <td> <code> sample_size </code> </td> <td> 为生成统计数据而采样的页面数量 </td> </tr> <tr> <td> <code> stat_description </code> </td> <td> 对应的统计项的描述 </td> </tr> </tbody> </table>

innodb_index_stats 表的每条记录代表着一个索引的一个统计项。

可能这会大家有些懵逼这个统计项到底指什么，别着急，我们直接看一下关于 `single_table` 表的索引统计数据都有些什么：

```

mysql> SELECT * FROM mysql.innodb_index_stats WHERE table_name = 'single_table';
+---------------+--------------+--------------+---------------------+--------------+------------+-------------+-----------------------------------+
| database_name | table_name   | index_name   | last_update         | stat_name    | stat_value | sample_size | stat_description                  |
+---------------+--------------+--------------+---------------------+--------------+------------+-------------+-----------------------------------+
| xiaohaizi     | single_table | PRIMARY      | 2018-12-14 14:24:46 | n_diff_pfx01 |       9693 |          20 | id                                |
| xiaohaizi     | single_table | PRIMARY      | 2018-12-14 14:24:46 | n_leaf_pages |         91 |        NULL | Number of leaf pages in the index |
| xiaohaizi     | single_table | PRIMARY      | 2018-12-14 14:24:46 | size         |         97 |        NULL | Number of pages in the index      |
| xiaohaizi     | single_table | idx_key1     | 2018-12-14 14:24:46 | n_diff_pfx01 |        968 |          28 | key1                              |
| xiaohaizi     | single_table | idx_key1     | 2018-12-14 14:24:46 | n_diff_pfx02 |      10000 |          28 | key1,id                           |
| xiaohaizi     | single_table | idx_key1     | 2018-12-14 14:24:46 | n_leaf_pages |         28 |        NULL | Number of leaf pages in the index |
| xiaohaizi     | single_table | idx_key1     | 2018-12-14 14:24:46 | size         |         29 |        NULL | Number of pages in the index      |
| xiaohaizi     | single_table | idx_key2     | 2018-12-14 14:24:46 | n_diff_pfx01 |      10000 |          16 | key2                              |
| xiaohaizi     | single_table | idx_key2     | 2018-12-14 14:24:46 | n_leaf_pages |         16 |        NULL | Number of leaf pages in the index |
| xiaohaizi     | single_table | idx_key2     | 2018-12-14 14:24:46 | size         |         17 |        NULL | Number of pages in the index      |
| xiaohaizi     | single_table | idx_key3     | 2018-12-14 14:24:46 | n_diff_pfx01 |        799 |          31 | key3                              |
| xiaohaizi     | single_table | idx_key3     | 2018-12-14 14:24:46 | n_diff_pfx02 |      10000 |          31 | key3,id                           |
| xiaohaizi     | single_table | idx_key3     | 2018-12-14 14:24:46 | n_leaf_pages |         31 |        NULL | Number of leaf pages in the index |
| xiaohaizi     | single_table | idx_key3     | 2018-12-14 14:24:46 | size         |         32 |        NULL | Number of pages in the index      |
| xiaohaizi     | single_table | idx_key_part | 2018-12-14 14:24:46 | n_diff_pfx01 |       9673 |          64 | key_part1                         |
| xiaohaizi     | single_table | idx_key_part | 2018-12-14 14:24:46 | n_diff_pfx02 |       9999 |          64 | key_part1,key_part2               |
| xiaohaizi     | single_table | idx_key_part | 2018-12-14 14:24:46 | n_diff_pfx03 |      10000 |          64 | key_part1,key_part2,key_part3     |
| xiaohaizi     | single_table | idx_key_part | 2018-12-14 14:24:46 | n_diff_pfx04 |      10000 |          64 | key_part1,key_part2,key_part3,id  |
| xiaohaizi     | single_table | idx_key_part | 2018-12-14 14:24:46 | n_leaf_pages |         64 |        NULL | Number of leaf pages in the index |
| xiaohaizi     | single_table | idx_key_part | 2018-12-14 14:24:46 | size         |         97 |        NULL | Number of pages in the index      |
+---------------+--------------+--------------+---------------------+--------------+------------+-------------+-----------------------------------+
20 rows in set (0.03 sec)

```

这个结果有点儿多，正确查看这个结果的方式是这样的：

1. 先查看 `index_name` 列，这个列说明该记录是哪个索引的统计信息，从结果中我们可以看出来，`PRIMARY` 索引（也就是主键）占了 3 条记录，`idx_key_part` 索引占了 6 条记录。

2. 针对 `index_name` 列相同的记录，`stat_name` 表示针对该索引的统计项名称，`stat_value` 展示的是该索引在该统计项上的值，`stat_description` 指的是来描述该统计项的含义的。我们来具体看一下一个索引都有哪些统计项：
    
    - `n_leaf_pages` ：表示该索引的叶子节点占用多少页面。

    - `size` ：表示该索引共占用多少页面。

    - `n_diff_pfx**NN**` ：表示对应的索引列不重复的值有多少。其中的 `NN` 长得有点儿怪呀，啥意思呢？
        
        其实 `NN` 可以被替换为 `01`、`02`、`03`... 这样的数字。比如对于 `idx_key_part` 来说：
        
        - `n_diff_pfx01` 表示的是统计 `key_part1` 这单单一个列不重复的值有多少。

        - `n_diff_pfx02` 表示的是统计 `key_part1、key_part2` 这两个列组合起来不重复的值有多少。

        - `n_diff_pfx03` 表示的是统计 `key_part1、key_part2、key_part3` 这三个列组合起来不重复的值有多少。

        - `n_diff_pfx04` 表示的是统计 `key_part1、key_part2、key_part3、id` 这四个列组合起来不重复的值有多少。

        这里需要注意的是，对于普通的二级索引，并不能保证它的索引列值是唯一的，比如对于 idx_key1 来说，key1 列就可能有很多值重复的记录。此时只有在索引列上加上主键值才可以区分两条索引列值都一样的二级索引记录。对于主键和唯一二级索引则没有这个问题，它们本身就可以保证索引列值的不重复，所以也不需要再统计一遍在索引列后加上主键值的不重复值有多少。比如上边的 idx_key1 有 n_diff_pfx01、n_diff_pfx02 两个统计项，而 idx_key2 却只有 n_diff_pfx01 一个统计项。

3. 在计算某些索引列中包含多少不重复值时，需要对一些叶子节点页面进行采样，`sample_size` 列就表明了采样的页面数量是多少。

### 2.1.3. 定期更新统计数据

随着我们不断的对表进行增删改操作，表中的数据也一直在变化，`innodb_table_stats` 和 `innodb_index_stats` 表里的统计数据也会产生相应变化。设计 `MySQL` 的大叔提供了如下两种更新统计数据的方式：

- 开启 `innodb_stats_auto_recalc`。
    
    系统变量 `innodb_stats_auto_recalc` 决定着服务器是否自动重新计算统计数据，它的默认值是 `ON`，也就是该功能默认是开启的。每个表都维护了一个变量，该变量记录着对该表进行增删改的记录条数，如果发生变动的记录数量超过了表大小的 `10%`，并且自动重新计算统计数据的功能是打开的，那么服务器会重新进行一次统计数据的计算，并且更新 `innodb_table_stats` 和 `innodb_index_stats` 表。不过自动重新计算统计数据的过程是异步发生的，也就是即使表中变动的记录数超过了 `10%`，自动重新计算统计数据也不会立即发生，可能会延迟几秒才会进行计算。
    
    再一次强调，`InnoDB` 默认是以表为单位来收集和存储统计数据的，我们也可以单独为某个表设置是否自动重新计算统计数的属性，设置方式就是在创建或修改表的时候通过指定 `STATS_AUTO_RECALC` 属性来指明该表的统计数据存储方式：

    当 `STATS_AUTO_RECALC=1` 时，表明我们想让该表自动重新计算统计数据，当 `STATS_AUTO_RECALC=0` 时，表明不想让该表自动重新计算统计数据。如果我们在创建表时未指定 `STATS_AUTO_RECALC` 属性，那默认采用系统变量 `innodb_stats_auto_recalc` 的值作为该属性的值。
    
- 手动调用 `ANALYZE TABLE` 语句来更新统计信息
    
    如果 `innodb_stats_auto_recalc` 系统变量的值为 `OFF` 的话，我们也可以手动调用 `ANALYZE TABLE` 语句来重新计算统计数据，比如我们可以这样更新关于 `single_table` 表的统计数据：

    需要注意的是，ANALYZE TABLE 语句会立即重新计算统计数据，也就是这个过程是同步的，在表中索引多或者采样页面特别多时这个过程可能会特别慢，请不要没事儿就运行一下 `ANALYZE TABLE` 语句，最好在业务不是很繁忙的时候再运行。

## 2.2. 基于内存的非永久性统计数据

当我们把系统变量 `innodb_stats_persistent` 的值设置为 `OFF` 时，之后创建的表的统计数据默认就都是非永久性的了，或者我们直接在创建表或修改表时设置 `STATS_PERSISTENT` 属性的值为 `0`，那么该表的统计数据就是非永久性的了。

与永久性的统计数据不同，非永久性的统计数据采样的页面数量是由 `innodb_stats_transient_sample_pages` 控制的，这个系统变量的默认值是 `8`。

另外，由于非永久性的统计数据经常更新，所以导致 `MySQL` 查询优化器计算查询成本的时候依赖的是经常变化的统计数据，也就会生成经常变化的执行计划，这个可能让大家有些懵逼。不过最近的 `MySQL` 版本都不咋用这种基于内存的非永久性统计数据了，所以我们也就不深入唠叨它了。

