#数据库 #Mysql 

# 1. Mysql 的编码集

## 1.1. Mysql 支持的编码集

`utf8` 字符集表示一个字符需要使用 1～4 个字节，但是我们常用的一些字符使用 1～3 个字节就可以表示了。而在 `MySQL` 中字符集表示一个字符所用最大字节长度在某些方面会影响系统的存储和性能，所以设计 `MySQL` 的大叔偷偷的定义了两个概念：

- `utf8mb3` ：阉割过的 `utf8` 字符集，只使用 1～3 个字节表示字符。

- `utf8mb4` ：正宗的 `utf8` 字符集，使用 1～4 个字节表示字符。

有一点需要大家十分的注意，在 `MySQL` 中 `utf8` 是 `utf8mb3` 的别名，所以之后在 `MySQL` 中提到 `utf8` 就意味着使用 1~3 个字节来表示一个字符，如果大家有使用 4 字节编码一个字符的情况，比如存储一些 emoji 表情啥的，那请使用 `utf8mb4`。

其实准确的说，utf8 只是 Unicode 字符集的一种编码方案，Unicode 字符集可以采用 utf8、utf16、utf32 这几种编码方案，utf8 使用 1～4 个字节编码一个字符，utf16 使用 2 个或 4 个字节编码一个字符，utf32 使用 4 个字节编码一个字符。

通过语句 `SHOW (CHARACTER SET|CHARSET) [LIKE 匹配的模式];` 可以查询 Mysql 支持的字符集。

## 1.2. Mysql 字符集的比较规则

比较规则的作用通常体现比较字符串大小的表达式以及对某个字符串列进行排序中。一种字符集可能对应着若干种比较规则，`MySQL` 支持的字符集就已经非常多了，所以支持的比较规则更多，我们先只查看一下 `utf8` 字符集下的比较规则：

```sql

mysql> SHOW COLLATION LIKE 'utf8\_%';
+--------------------------+---------+-----+---------+----------+---------+
| Collation                | Charset | Id  | Default | Compiled | Sortlen |
+--------------------------+---------+-----+---------+----------+---------+
| utf8_general_ci          | utf8    |  33 | Yes     | Yes      |       1 |
| utf8_bin                 | utf8    |  83 |         | Yes      |       1 |
| utf8_unicode_ci          | utf8    | 192 |         | Yes      |       8 |
| utf8_icelandic_ci        | utf8    | 193 |         | Yes      |       8 |
| utf8_latvian_ci          | utf8    | 194 |         | Yes      |       8 |
| utf8_romanian_ci         | utf8    | 195 |         | Yes      |       8 |
| utf8_slovenian_ci        | utf8    | 196 |         | Yes      |       8 |
| utf8_polish_ci           | utf8    | 197 |         | Yes      |       8 |
| utf8_estonian_ci         | utf8    | 198 |         | Yes      |       8 |
| utf8_spanish_ci          | utf8    | 199 |         | Yes      |       8 |
| utf8_swedish_ci          | utf8    | 200 |         | Yes      |       8 |
| utf8_turkish_ci          | utf8    | 201 |         | Yes      |       8 |
| utf8_czech_ci            | utf8    | 202 |         | Yes      |       8 |
| utf8_danish_ci           | utf8    | 203 |         | Yes      |       8 |
| utf8_lithuanian_ci       | utf8    | 204 |         | Yes      |       8 |
| utf8_slovak_ci           | utf8    | 205 |         | Yes      |       8 |
| utf8_spanish2_ci         | utf8    | 206 |         | Yes      |       8 |
| utf8_roman_ci            | utf8    | 207 |         | Yes      |       8 |
| utf8_persian_ci          | utf8    | 208 |         | Yes      |       8 |
| utf8_esperanto_ci        | utf8    | 209 |         | Yes      |       8 |
| utf8_hungarian_ci        | utf8    | 210 |         | Yes      |       8 |
| utf8_sinhala_ci          | utf8    | 211 |         | Yes      |       8 |
| utf8_german2_ci          | utf8    | 212 |         | Yes      |       8 |
| utf8_croatian_ci         | utf8    | 213 |         | Yes      |       8 |
| utf8_unicode_520_ci      | utf8    | 214 |         | Yes      |       8 |
| utf8_vietnamese_ci       | utf8    | 215 |         | Yes      |       8 |
| utf8_general_mysql500_ci | utf8    | 223 |         | Yes      |       1 |
+--------------------------+---------+-----+---------+----------+---------+
27 rows in set (0.00 sec)

```

名称后缀意味着该比较规则是否区分语言中的重音、大小写啥的，具体可以用的值如下：

<table> <thead> <tr> <th> 后缀 </th> <th> 英文释义 </th> <th> 描述 </th> </tr> </thead> <tbody> <tr> <td> <code> _ai </code> </td> <td> <code> accent insensitive </code> </td> <td> 不区分重音 </td> </tr> <tr> <td> <code> _as </code> </td> <td> <code> accent sensitive </code> </td> <td> 区分重音 </td> </tr> <tr> <td> <code> _ci </code> </td> <td> <code> case insensitive </code> </td> <td> 不区分大小写 </td> </tr> <tr> <td> <code> _cs </code> </td> <td> <code> case sensitive </code> </td> <td> 区分大小写 </td> </tr> <tr> <td> <code> _bin </code> </td> <td> <code> binary </code> </td> <td> 以二进制方式比较 </td> </tr> </tbody> </table>
   
比如 `utf8_general_ci` 这个比较规则是以 `ci` 结尾的，说明不区分大小写。

每种字符集对应若干种比较规则，每种字符集都有一种默认的比较规则，`SHOW COLLATION` 的返回结果中的 `Default` 列的值为 `YES` 的就是该字符集的默认比较规则，比方说 `utf8` 字符集默认的比较规则就是 `utf8_general_ci`。

`MySQL` 有 4 个级别的字符集和比较规则，分别是：

- 服务器级别

- 数据库级别

- 表级别

- 列级别

可以使用 sql 语句更改数据库的字符集与比较规则，在转换列的字符集时需要注意，**如果转换前列中存储的数据不能用转换后的字符集进行表示会发生错误**。比方说原先列使用的字符集是 utf8，列中存储了一些汉字，现在把列的字符集转换为 ascii 的话就会出错，因为 ascii 字符集并不能表示汉字字符。

## 1.3. 服务器与客户端之间的字符集转换

从发送请求到接收结果过程中发生的字符集转换：

- 客户端使用操作系统的字符集编码请求字符串，向服务器发送的是经过编码的一个字节串。

- 服务器将客户端发送来的字节串采用 `character_set_client` 代表的字符集进行解码，将解码后的字符串再按照 `character_set_connection` 代表的字符集进行编码。

- 如果 `character_set_connection` 代表的字符集和具体操作的列使用的字符集一致，则直接进行相应操作，否则的话需要将请求中的字符串从 `character_set_connection` 代表的字符集转换为具体操作的列使用的字符集之后再进行操作。

- 将从某个列获取到的字节串从该列使用的字符集转换为 `character_set_results` 代表的字符集后发送到客户端。

- 客户端使用操作系统的字符集解析收到的结果集字节串。

在这个过程中各个系统变量的含义如下：

<table> <thead> <tr> <th> 系统变量 </th> <th> 描述 </th> </tr> </thead> <tbody> <tr> <td> <code> character_set_client </code> </td> <td> 服务器解码请求时使用的字符集 </td> </tr> <tr> <td> <code> character_set_connection </code> </td> <td> 服务器处理请求时会把请求字符串从 <code> character_set_client </code> 转为 <code> character_set_connection </code> </td> </tr> <tr> <td> <code> character_set_results </code> </td> <td> 服务器向客户端返回数据时使用的字符集 </td> </tr> </tbody> </table>    

一般情况下要使用**保持这三个变量的值和客户端使用的字符集相同**。

# 2. Mysql 记录的存储结构

`InnoDB` 是一个将表中的数据存储到磁盘上的存储引擎，所以即使关机后重启我们的数据还是存在的。而真正处理数据的过程是发生在内存中的，所以需要把磁盘中的数据加载到内存中，如果是处理写入或修改请求的话，还需要把内存中的内容刷新到磁盘上。而我们知道读写磁盘的速度非常慢，和内存读写差了几个数量级，所以当我们想从表中获取某些记录时，`InnoDB` 存储引擎需要一条一条的把记录从磁盘上读出来么？不，那样会慢死，`InnoDB` 采取的方式是：将数据划分为若干个页，以页作为磁盘和内存之间交互的基本单位，**InnoDB 中页的大小一般为 16KB**。也就是在一般情况下，一次最少从磁盘中读取 16KB 的内容到内存中，一次最少把内存中的 16KB 内容刷新到磁盘中。

我们平时是以记录为单位来向表中插入数据的，这些记录在磁盘上的存放方式也被称为 `行格式` 或者 `记录格式`。设计 `InnoDB` 存储引擎的大叔们到现在为止设计了 4 种不同类型的 `行格式`，分别是 `Compact`、`Redundant`、`Dynamic` 和 `Compressed` 行格式。

我们可以在创建或修改表的语句中指定 `行格式` ：

```sql

CREATE TABLE 表名 (列的信息) ROW_FORMAT=行格式名称
   
ALTER TABLE 表名 ROW_FORMAT=行格式名称

```

比如我们在 `xiaohaizi` 数据库里创建一个演示用的表 `record_format_demo`，可以这样指定它的 `行格式` ：

```sql

mysql> USE xiaohaizi;
Database changed

mysql> CREATE TABLE record_format_demo (
    ->     c1 VARCHAR(10),
    ->     c2 VARCHAR(10) NOT NULL,
    ->     c3 CHAR(10),
    ->     c4 VARCHAR(10)
    -> ) CHARSET=ascii ROW_FORMAT=COMPACT;
Query OK, 0 rows affected (0.03 sec)

```

可以看到我们刚刚创建的这个表的 `行格式` 就是 `Compact`，另外，我们还显式指定了这个表的字符集为 `ascii`，因为 `ascii` 字符集只包括空格、标点符号、数字、大小写字母和一些不可见字符，所以我们的汉字是不能存到这个表里的。我们现在向这个表中插入两条记录：

```sql

mysql> SELECT * FROM record_format_demo;
+------+-----+------+------+
| c1   | c2  | c3   | c4   |
+------+-----+------+------+
| aaaa | bbb | cc   | d    |
| eeee | fff | NULL | NULL |
+------+-----+------+------+
2 rows in set (0.00 sec)

mysql>

```

## 2.1. COMPACT 格式

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042217042.png)

COMPACT 格式一条完整的记录其实可以被分为 `记录的额外信息` 和 `记录的真实数据` 两大部分。

### 2.1.1. 记录的额外信息

这部分信息是服务器为了描述这条记录而不得不额外添加的一些信息，这些额外信息分为 3 类，分别是 `变长字段长度列表`、`NULL值列表` 和 `记录头信息`。

### 2.1.2. 变长字段列表

`MySQL` 支持一些变长的数据类型，比如 `VARCHAR(M)`、`VARBINARY(M)`、各种 `TEXT` 类型，各种 `BLOB` 类型，我们也可以把拥有这些数据类型的列称为 `变长字段`，变长字段中存储多少字节的数据是不固定的，所以我们在存储真实数据的时候需要顺便把这些数据占用的字节数也存起来，这样才不至于把 `MySQL` 服务器搞懵，所以这些变长字段占用的存储空间分为两部分：

1. 真正的数据内容
2. 占用的字节数

在 `Compact` 行格式中，把所有变长字段的真实数据占用的字节长度都存放在记录的开头部位，从而形成一个变长字段长度列表，各变长字段数据占用的字节数按照列的顺序**逆序存放**。

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042223666.png)
![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042223432.png)

由于第一行记录中 `c1`、`c2`、`c4` 列中的字符串都比较短，也就是说内容占用的字节数比较小，用 1 个字节就可以表示，但是如果变长列的内容占用的字节数比较多，可能就需要用 2 个字节来表示。具体用 1 个还是 2 个字节来表示真实数据占用的字节数，`InnoDB` 有它的一套规则，我们首先声明一下 `W`、`M` 和 `L` 的意思：

1. 假设某个字符集中表示一个字符最多需要使用的字节数为 `W`，也就是使用 `SHOW CHARSET` 语句的结果中的 `Maxlen` 列，比方说 `utf8` 字符集中的 `W` 就是 `3`，`gbk` 字符集中的 `W` 就是 `2`，`ascii` 字符集中的 `W` 就是 `1`。

2. 对于变长类型 `VARCHAR(M)` 来说，这种类型表示能存储最多 `M` 个字符（注意是字符不是字节），所以这个类型能表示的字符串最多占用的字节数就是 `M×W`。

3. 假设它实际存储的字符串占用的字节数是 `L`。

所以确定使用 1 个字节还是 2 个字节表示真正字符串占用的字节数的规则就是这样：

- 如果 `M×W <= 255`，那么使用 1 个字节来表示真正字符串占用的字节数。

	也就是说 InnoDB 在读记录的变长字段长度列表时先查看表结构，如果某个变长字段允许存储的最大字节数不大于 255 时，可以认为只使用 1 个字节来表示真正字符串占用的字节数。

- 如果 `M×W > 255`，则分为两种情况：

	- 如果 `L <= 127`，则用 1 个字节来表示真正字符串占用的字节数。
	
	- 如果 `L > 127`，则用 2 个字节来表示真正字符串占用的字节数。

	InnoDB 在读记录的变长字段长度列表时先查看表结构，如果某个变长字段允许存储的最大字节数大于 255 时，该怎么区分它正在读的某个字节是一个单独的字段长度还是半个字段长度呢？
	
	设计 InnoDB 的大叔使用该字节的第一个二进制位作为标志位：如果该字节的第一个位为 0，那该字节就是一个单独的字段长度（使用一个字节表示不大于 127 的二进制的第一个位都为 0），如果该字节的第一个位为 1，那该字节就是半个字段长度。对于一些占用字节数非常多的字段，比方说某个字段长度大于了 16KB，那么如果该记录在单个页面中无法存储时，InnoDB 会把一部分数据存放到所谓的溢出页中（我们后边会唠叨），在变长字段长度列表处只存储留在本页面中的长度，所以使用两个字节也可以存放下来。

总结一下就是说：如果该可变字段允许存储的最大字节数（`M×W`）超过 255 字节并且真实存储的字节数（`L`）超过 127 字节，则使用 2 个字节，否则使用 1 个字节。

另外需要注意的一点是，变长字段长度列表中只存储值为 _**非 NULL**_ 的列内容占用的长度，值为 _**NULL**_ 的列的长度是不储存的。

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042228572.png)

**并不是所有记录都有这个变长字段长度列表部分**，比方说表中所有的列都不是变长的数据类型的话，这一部分就不需要有。

### 2.1.3. NULL 值列表

表中的某些列可能存储 `NULL` 值，如果把这些 `NULL` 值都放到 `记录的真实数据` 中存储会很占地方，所以 `Compact` 行格式把这些值为 `NULL` 的列统一管理起来，存储到 `NULL` 值列表中，它的处理过程是这样的：

1. 首先统计表中允许存储 `NULL` 的列有哪些。

    主键列、被 `NOT NULL` 修饰的列都是不可以存储 `NULL` 值的，所以在统计的时候不会把这些列算进去。

2. 如果表中没有允许存储 _**NULL**_ 的列，则 _NULL 值列表_ 也不存在了，否则将每个允许存储 `NULL` 的列对应一个二进制位，二进制位按照列的顺序**逆序排列**，二进制位表示的意义如下：

    - 二进制位的值为 `1` 时，代表该列的值为 `NULL`。
	
    - 二进制位的值为 `0` 时，代表该列的值不为 `NULL`。

3. `MySQL` 规定 `NULL值列表` 必须用整数个字节的位表示，如果使用的二进制位个数不是整数个字节，则在字节的高位补 `0`。

所以对于上面两条数据，他们的 _NULL 值列表_ 如下：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042233266.png)
![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042233752.png)
![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042233037.png)

### 2.1.4. 记录头信息

除了 `变长字段长度列表`、`NULL值列表` 之外，还有一个用于描述记录的 `记录头信息`，它是由固定的 `5` 个字节组成。`5` 个字节也就是 `40` 个二进制位，不同的位代表不同的意思，如图：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042234643.png)

这些二进制位代表的详细信息如下表：

<table> <thead> <tr> <th> 名称 </th> <th> 大小（单位：bit）</th> <th> 描述 </th> </tr> </thead> <tbody> <tr> <td> <code> 预留位 1 </code> </td> <td> <code> 1 </code> </td> <td> 没有使用 </td> </tr> <tr> <td> <code> 预留位 2 </code> </td> <td> <code> 1 </code> </td> <td> 没有使用 </td> </tr> <tr> <td> <code> delete_mask </code> </td> <td> <code> 1 </code> </td> <td> 标记该记录是否被删除 </td> </tr> <tr> <td> <code> min_rec_mask </code> </td> <td> <code> 1 </code> </td> <td> B + 树的每层非叶子节点中的最小记录都会添加该标记 </td> </tr> <tr> <td> <code> n_owned </code> </td> <td> <code> 4 </code> </td> <td> 表示当前记录拥有的记录数 </td> </tr> <tr> <td> <code> heap_no </code> </td> <td> <code> 13 </code> </td> <td> 表示当前记录在记录堆的位置信息 </td> </tr> <tr> <td> <code> record_type </code> </td> <td> <code> 3 </code> </td> <td> 表示当前记录的类型，<code> 0 </code> 表示普通记录，<code> 1 </code> 表示 B + 树非叶子节点记录，<code> 2 </code> 表示最小记录，<code> 3 </code> 表示最大记录 </td> </tr> <tr> <td> <code> next_record </code> </td> <td> <code> 16 </code> </td> <td> 表示下一条记录的相对位置 </td> </tr> </tbody> </table>

我们现在直接看一下 `record_format_demo` 中的两条记录的 `头信息` 分别是什么：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042237473.png)

### 2.1.5. 记录的真实数据

对于 `record_format_demo` 表来说，`记录的真实数据` 除了 `c1`、`c2`、`c3`、`c4` 这几个我们自己定义的列的数据以外，`MySQL` 会为每个记录默认的添加一些列（也称为 `隐藏列`），具体的列如下：

<table> <thead> <tr> <th> 列名 </th> <th> 是否必须 </th> <th> 占用空间 </th> <th> 描述 </th> </tr> </thead> <tbody> <tr> <td> <code> row_id </code> </td> <td> 否 </td> <td> <code> 6 </code> 字节 </td> <td> 行 ID，唯一标识一条记录 </td> </tr> <tr> <td> <code> transaction_id </code> </td> <td> 是 </td> <td> <code> 6 </code> 字节 </td> <td> 事务 ID </td> </tr> <tr> <td> <code> roll_pointer </code> </td> <td> 是 </td> <td> <code> 7 </code> 字节 </td> <td> 回滚指针 </td> </tr> </tbody> </table>

实际上这几个列的真正名称其实是：DB_ROW_ID、DB_TRX_ID、DB_ROLL_PTR，我们为了美观才写成了 row_id、transaction_id 和 roll_pointer。

这里需要提一下 `InnoDB` 表对主键的生成策略：优先使用用户自定义主键作为主键，**如果用户没有定义主键，则选取一个 `Unique` 键作为主键**，如果表中连 `Unique` 键都没有定义的话，则 `InnoDB` 会为表默认添加一个名为 `row_id` 的隐藏列作为主键。

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042239073.png)

### 2.1.6. CHAR (M) 列的存储格式

在 `Compact` 行格式下只会把变长类型的列的长度逆序存到 `变长字段长度列表` 中，如果采用变长的字符集（也就是表示一个字符需要的字节数不确定，比如 `gbk` 表示一个字符要 1～2 个字节、`utf8` 表示一个字符要 1~3 个字节等）的话，列的长度也会被存储到 `变长字段长度列表` 中。

比如我们修改一下 `record_format_demo` 表的字符集：

```sql

mysql> ALTER TABLE record_format_demo MODIFY COLUMN c3 CHAR(10) CHARACTER SET utf8;
Query OK, 2 rows affected (0.02 sec)
Records: 2  Duplicates: 0  Warnings: 0

```

修改该列字符集后记录的 `变长字段长度列表` 也发生了变化，如图：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042243072.png)
这就意味着：对于 _**CHAR (M)**_ 类型的列来说，当列采用的是定长字符集时，该列占用的字节数不会被加到变长字段长度列表，而如果采用变长字符集时，该列占用的字节数也会被加到变长字段长度列表。

另外有一点还需要注意，变长字符集的 `CHAR(M)` 类型的列要求至少占用 `M` 个字节，而 `VARCHAR(M)` 却没有这个要求。比方说对于使用 `utf8` 字符集的 `CHAR(10)` 的列来说，该列存储的数据字节长度的范围是 10～30 个字节。即使我们向该列中存储一个空字符串也会占用 `10` 个字节，这是怕将来更新该列的值的字节长度大于原有值的字节长度而小于 10 个字节时，可以在该记录处直接更新，而不是在存储空间中重新分配一个新的记录空间，导致原有的记录空间成为所谓的碎片。

我们知道对于 `VARCHAR(M)` 类型的列最多可以占用 `65535` 个字节。其中的 `M` 代表该类型最多存储的字符数量。`MySQL` 对一条记录占用的最大存储空间是有限制的，除了 `BLOB` 或者 `TEXT` 类型的列之外，其他所有的列（不包括隐藏列和记录头信息）占用的字节长度加起来不能超过 `65535` 个字节。所以 `MySQL` 服务器建议我们把存储类型改为 `TEXT` 或者 `BLOB` 的类型。这个 `65535` 个字节除了列本身的数据之外，还包括一些其他的数据（`storage overhead`），比如说我们为了存储一个 `VARCHAR(M)` 类型的列，其实需要占用 3 部分存储空间：

- 真实数据

- 真实数据占用字节的长度

- `NULL` 值标识，如果该列有 `NOT NULL` 属性则可以没有这部分存储空间

如果 `VARCHAR(M)` 类型的列使用的不是 `ascii` 字符集，那 `M` 的最大取值取决于该字符集表示一个字符最多需要的字节数。在列的值允许为 `NULL` 的情况下，`gbk` 字符集表示一个字符最多需要 `2` 个字节，那在该字符集下，`M` 的最大取值就是 `32766`（也就是：65532/2），也就是说最多能存储 `32766` 个字符；`utf8` 字符集表示一个字符最多需要 `3` 个字节，那在该字符集下，`M` 的最大取值就是 `21844`，就是说最多能存储 `21844`（也就是：65532/3）个字符。

上述所言在列的值允许为 NULL 的情况下，gbk 字符集下 M 的最大取值就是 32766，utf8 字符集下 M 的最大取值就是 21844，这都是在表中只有一个字段的情况下说的。**一个行中的所有列（不包括隐藏列和记录头信息）占用的字节长度加起来不能超过 65535 个字节！**

## 2.2. 行溢出

### 2.2.1. 行溢出现象

在 `Compact` 和 `Redundant` 行格式中，对于占用存储空间非常大的列，在 `记录的真实数据` 处只会存储该列的一部分数据，把剩余的数据分散存储在几个其他的页中，然后 `记录的真实数据` 处用 20 个字节存储指向这些页的地址（当然这 20 个字节中还包括这些分散在其他页面中的数据的占用的字节数），从而可以找到剩余数据所在的页，如图所示：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042303293.png)

从图中可以看出来，对于 `Compact` 和 `Redundant` 行格式来说，如果某一列中的数据非常多的话，在本记录的真实数据处只会存储该列的前 `768` 个字节的数据和一个指向其他页的地址，然后把剩下的数据存放到其他页中，这个过程也叫做 `行溢出`，存储超出 `768` 字节的那些页面也被称为 `溢出页`。画一个简图就是这样：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042304813.png)

最后需要注意的是，不只是 VARCHAR (M) 类型的列，其他的 TEXT、BLOB 类型的列在存储数据非常多的时候也会发生 `行溢出`。

### 2.2.2. 行溢出临界点

那发生 `行溢出` 的临界点是什么呢？也就是说在列存储多少字节的数据时就会发生 `行溢出`？

`MySQL` 中规定一个页中至少存放两行记录。以上边的 `varchar_size_demo` 表为例，它只有一个列 `c`，我们往这个表中插入两条记录，每条记录最少插入多少字节的数据才会 `行溢出` 的现象呢？这得分析一下页中的空间都是如何利用的。

- 每个页除了存放我们的记录以外，也需要存储一些额外的信息，乱七八糟的额外信息加起来需要 `132` 个字节的空间（现在只要知道这个数字就好了），其他的空间都可以被用来存储记录。

- 每个记录需要的额外信息是 `27` 字节。

	这 27 个字节包括下边这些部分：

	- 2 个字节用于存储真实数据的长度
	- 1 个字节用于存储列是否是 NULL 值
	- 5 个字节大小的头信息
	- 6 个字节的 `row_id` 列
	- 6 个字节的 `transaction_id` 列
	- 7 个字节的 `roll_pointer` 列

假设一个列中存储的数据字节数为 n，设计 `MySQL` 的大叔规定如果该列不发生溢出的现象，就需要满足下边这个式子：

```scss

132 + 2×(27 + n) < 16384

```

如果表中有多个列，那上边的式子和结论都需要改一改了，所以重点就是：你不用关注这个临界点是什么，只要知道如果我们一条记录的某个列中存储的数据占用的字节数非常多时，该列就可能成为 `溢出列`。

## 2.3. Dynamic 和 Compressed 行格式

`Dynamic` 和 `Compressed` 行格式，这俩行格式和 `Compact` 行格式挺像，只不过在处理 `行溢出` 数据时有点儿分歧，它们不会在记录的真实数据处存储字段真实数据的前 `768` 个字节，而是把所有的字节都存储到其他页面中，只在记录的真实数据处存储其他页面的地址，就像这样：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042308235.png)

`Compressed` 行格式和 `Dynamic` 不同的一点是，`Compressed` 行格式会采用压缩算法对页面进行压缩，以节省空间。

# 3. Mysql 的数据页

## 3.1. 数据页概览

数据页代表的这块 `16KB` 大小的存储空间可以被划分为多个部分，不同部分有不同的功能，各个部分如图所示：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042335106.png)

一个 `InnoDB` 数据页的存储空间大致被划分成了 `7` 个部分，有的部分占用的字节数是确定的，有的部分占用的字节数是不确定的。

<table> <thead> <tr> <th> 名称 </th> <th> 中文名 </th> <th> 占用空间大小 </th> <th> 简单描述 </th> </tr> </thead> <tbody> <tr> <td> <code> File Header </code> </td> <td> 文件头部 </td> <td> <code> 38 </code> 字节 </td> <td> 页的一些通用信息 </td> </tr> <tr> <td> <code> Page Header </code> </td> <td> 页面头部 </td> <td> <code> 56 </code> 字节 </td> <td> 数据页专有的一些信息 </td> </tr> <tr> <td> <code> Infimum + Supremum </code> </td> <td> 最小记录和最大记录 </td> <td> <code> 26 </code> 字节 </td> <td> 两个虚拟的行记录 </td> </tr> <tr> <td> <code> User Records </code> </td> <td> 用户记录 </td> <td> 不确定 </td> <td> 实际存储的行记录内容 </td> </tr> <tr> <td> <code> Free Space </code> </td> <td> 空闲空间 </td> <td> 不确定 </td> <td> 页中尚未使用的空间 </td> </tr> <tr> <td> <code> Page Directory </code> </td> <td> 页面目录 </td> <td> 不确定 </td> <td> 页中的某些记录的相对位置 </td> </tr> <tr> <td> <code> File Trailer </code> </td> <td> 文件尾部 </td> <td> <code> 8 </code> 字节 </td> <td> 校验页是否完整 </td> </tr> </tbody> </table>

## 3.2. 记录在页中的存储

在页的 7 个组成部分中，我们自己存储的记录会按照我们指定的 `行格式` 存储到 `User Records` 部分。但是在一开始生成页的时候，其实并没有 `User Records` 这个部分，每当我们插入一条记录，都会从 `Free Space` 部分，也就是尚未使用的存储空间中申请一个记录大小的空间划分到 `User Records` 部分，当 `Free Space` 部分的空间全部被 `User Records` 部分替代掉之后，也就意味着这个页使用完了，如果还有新的记录插入的话，就需要去申请新的页了，这个过程的图示如下：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042337859.png)

### 3.2.1. 数据的记录头信息

为了故事的顺利发展，我们先创建一个表：

```sql

mysql> CREATE TABLE page_demo(
    ->     c1 INT,
    ->     c2 INT,
    ->     c3 VARCHAR(10000),
    ->     PRIMARY KEY (c1)
    -> ) CHARSET=ascii ROW_FORMAT=Compact;
Query OK, 0 rows affected (0.03 sec)

```

这个新创建的 `page_demo` 表有 3 个列，其中 `c1` 和 `c2` 列是用来存储整数的，`c3` 列是用来存储字符串的。需要注意的是，我们把 _**c1**_ 列指定为主键，所以在具体的行格式中 InnoDB 就没必要为我们去创建那个所谓的 _**row_id**_ 隐藏列了。而且我们为这个表指定了 `ascii` 字符集以及 `Compact` 的行格式。所以这个表中记录的行格式示意图就是这样的：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042339797.png)

<table> <thead> <tr> <th> 名称 </th> <th> 大小（单位：bit）</th> <th> 描述 </th> </tr> </thead> <tbody> <tr> <td> <code> 预留位 1 </code> </td> <td> <code> 1 </code> </td> <td> 没有使用 </td> </tr> <tr> <td> <code> 预留位 2 </code> </td> <td> <code> 1 </code> </td> <td> 没有使用 </td> </tr> <tr> <td> <code> delete_mask </code> </td> <td> <code> 1 </code> </td> <td> 标记该记录是否被删除 </td> </tr> <tr> <td> <code> min_rec_mask </code> </td> <td> <code> 1 </code> </td> <td> B + 树的每层非叶子节点中的最小记录都会添加该标记 </td> </tr> <tr> <td> <code> n_owned </code> </td> <td> <code> 4 </code> </td> <td> 表示当前记录拥有的记录数 </td> </tr> <tr> <td> <code> heap_no </code> </td> <td> <code> 13 </code> </td> <td> 表示当前记录在记录堆的位置信息 </td> </tr> <tr> <td> <code> record_type </code> </td> <td> <code> 3 </code> </td> <td> 表示当前记录的类型，<code> 0 </code> 表示普通记录，<code> 1 </code> 表示 B + 树非叶节点记录，<code> 2 </code> 表示最小记录，<code> 3 </code> 表示最大记录 </td> </tr> <tr> <td> <code> next_record </code> </td> <td> <code> 16 </code> </td> <td> 表示下一条记录的相对位置 </td> </tr> </tbody> </table>

以一些数据例子为例说明记录头信息中各个字段的作用：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042340092.png)

- delete_mask

	这个属性标记着当前记录是否被删除，占用 1 个二进制位，值为 `0` 的时候代表记录并没有被删除，为 `1` 的时候代表记录被删除掉了。
	
	啥？被删除的记录还在 `页` 中么？是的，摆在台面上的和背地里做的可能大相径庭，你以为它删除了，可它还在真实的磁盘上。
	
	这些被删除的记录之所以不立即从磁盘上移除，是因为**移除它们之后把其他的记录在磁盘上重新排列需要性能消耗**，所以只是打一个删除标记而已，所有被删除掉的记录都会组成一个所谓的 `垃圾链表`，在这个链表中的记录占用的空间称之为所谓的 `可重用空间`，之后如果有新记录插入到表中的话，可能把这些被删除的记录占用的存储空间覆盖掉。

	将这个 delete_mask 位设置为 1 和将被删除的记录加入到垃圾链表中其实是两个阶段，我们后边在介绍事务的时候会详细唠叨删除操作的详细过程。

- min_rec_mask

	B+树的每层非叶子节点中的最小记录都会添加该标记

- n_owned

	槽中记录数，用于记录当前记录数据所属槽中包含的记录数，具体在[[Mysql的存储结构#Page Directory（页目录）|页目录]]中使用。

- heap_no

	这个属性表示当前记录在本 `页` 中的位置，从图中可以看出来，我们插入的 4 条记录在本 `页` 中的位置分别是：`2`、`3`、`4`、`5`。是不是少了点啥？是的，怎么不见 `heap_no` 值为 `0` 和 `1` 的记录呢？

	`InnoDB` 自动给每个页里边儿加了两个记录，由于这两个记录并不是我们自己插入的，所以有时候也称为 `伪记录` 或者 `虚拟记录`。这两个伪记录一个代表 `最小记录`，一个代表 `最大记录`。

	不管我们向 `页` 中插入了多少自己的记录，设计 `InnoDB` 的大叔们都规定他们定义的两条伪记录分别为最小记录与最大记录。这两条记录的构造十分简单，都是由 5 字节大小的 `记录头信息` 和 8 字节大小的一个固定的部分组成的，如图所示：

	![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042345355.png)

	由于这两条记录不是我们自己定义的记录，所以它们并不存放在 `页` 的 `User Records` 部分，他们被单独放在一个称为 `Infimum + Supremum` 的部分，如图所示：

	![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042345733.png)

	从图中我们可以看出来，最小记录和最大记录的 `heap_no` 值分别是 `0` 和 `1`，也就是说它们的位置最靠前。

- record_type

	这个属性表示当前记录的类型，一共有 4 种类型的记录，`0` 表示普通记录，`1` 表示 B+树非叶节点记录，`2` 表示最小记录，`3` 表示最大记录。从图中我们也可以看出来，我们自己插入的记录就是普通记录，它们的 `record_type` 值都是 `0`，而最小记录和最大记录的 `record_type` 值分别为 `2` 和 `3`。

- next_record

	表示从当前记录的真实数据到下一条记录的真实数据的地址偏移量。比方说第一条记录的 `next_record` 值为 `32`，意味着从第一条记录的真实数据的地址处向后找 `32` 个字节便是下一条记录的真实数据。

	这其实是个 `链表`，可以通过一条记录找到它的下一条记录。但是需要注意的一点是，`下一条记录` 指得并不是按照我们插入顺序的下一条记录，而是按照主键值由小到大的顺序的下一条记录。而且规定 _**Infimum 记录（也就是最小记录）**_ 的下一条记录就是本页中主键值最小的用户记录，而本页中主键值最大的用户记录的下一条记录就是 _**Supremum 记录（也就是最大记录）**_。

	![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042347269.png)

	你会不会觉得 next_record 这个指针有点儿怪，为啥要指向记录头信息和真实数据之间的位置呢？为啥不干脆指向整条记录的开头位置，也就是记录的额外信息开头的位置呢？ 因为这个位置刚刚好，**向左读取就是记录头信息，向右读取就是真实数据**。我们前边还说过变长字段长度列表、NULL 值列表中的信息都是逆序存放，这样可以使记录中位置靠前的字段和它们对应的字段长度信息在内存中的距离更近，可能会提高高速缓存的命中率。

## 3.3. Page Directory（页目录）

现在我们了解了记录在页中按照主键值由小到大顺序串联成一个单链表，那如果我们想根据主键值查找页中的某条记录该咋办呢？

最笨的办法：从 `Infimum` 记录（最小记录）开始，沿着链表一直往后找，总有一天会找到（或者找不到）。在找的时候还能投机取巧，因为链表中各个记录的值是按照从小到大顺序排列的，所以当链表的某个节点代表的记录的主键值大于你想要查找的主键值时，你就可以停止查找了，因为该节点后边的节点的主键值依次递增。这个方法当记录数少的时候没啥问题，但当记录数多时性能表现不好。

我们平常想从一本书中查找某个内容的时候，一般会先看目录，找到需要查找的内容对应的书的页码，然后到对应的页码查看内容。设计 `InnoDB` 的大叔们为我们的记录也制作了一个类似的目录，他们的制作过程是这样的：

1. 将所有正常的记录（包括最大和最小记录，不包括标记为已删除的记录）划分为几个组。

2. 每个组的最后一条记录（也就是组内最大的那条记录）的头信息中的 `n_owned` 属性表示该记录拥有多少条记录，也就是该组内共有几条记录。

3. 将每个组的最后一条记录的地址偏移量单独提取出来按顺序存储到靠近 `页` 的尾部的地方，这个地方就是所谓的 `Page Directory`，也就是 `页目录`。页面目录中的这些地址偏移量被称为 `槽`（英文名：`Slot`），所以这个页面目录就是由 `槽` 组成的。

比方说现在的 `page_demo` 表中正常的记录共有 6 条，`InnoDB` 会把它们分成两组，第一组中只有一个最小记录，第二组中是剩余的 5 条记录：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042353242.png)
![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042353223.png)

设计 `InnoDB` 的大叔们对每个分组中的记录条数是有规定的：对于最小记录所在的分组只能有 _**1**_ 条记录，最大记录所在的分组拥有的记录条数只能在 _**1~8**_ 条之间，剩下的分组中记录的条数范围只能在是 _**4~8**_ 条之间。所以分组是按照下边的步骤进行的：

- 初始情况下一个数据页里只有最小记录和最大记录两条记录，它们分属于两个分组。

- 之后每插入一条记录，都会从 `页目录` 中找到主键值比本记录的主键值大并且差值最小的槽，然后把该槽对应的记录的 `n_owned` 值加 1，表示本组内又添加了一条记录，直到该组中的记录数等于 8 个。

- 在一个组中的记录数等于 8 个后再插入一条记录时，会将组中的记录拆分成两个组，一个组中 4 条记录，另一个 5 条记录。这个过程会在 `页目录` 中新增一个 `槽` 来记录这个新增分组中最大的那条记录的偏移量。

以更多的数据为例说明：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209042357931.png)

现在看怎么从这个 `页目录` 中查找记录。因为各个槽代表的记录的主键值都是从小到大排序的，所以我们可以使用所谓的 `二分法` 来进行快速查找。

在一个数据页中查找指定主键值的记录的过程分为两步：

1. 通过二分法确定该记录所在的槽，并找到该槽所在分组中主键值最小的那条记录。

2. 通过记录的 `next_record` 属性遍历该槽所在的组中的各个记录。

5 个槽的编号分别是：`0`、`1`、`2`、`3`、`4`，所以初始情况下最低的槽就是 `low=0`，最高的槽就是 `high=4`。比方说我们想找主键值为 `6` 的记录，过程是这样的：

1. 计算中间槽的位置：`(0+4)/2=2`，所以查看 `槽2` 对应记录的主键值为 `8`，又因为 `8 > 6`，所以设置 `high=2`，`low` 保持不变。

2. 重新计算中间槽的位置：`(0+2)/2=1`，所以查看 `槽1` 对应的主键值为 `4`，又因为 `4 < 6`，所以设置 `low=1`，`high` 保持不变。

3. 因为 `high - low` 的值为 1，所以确定主键值为 `6` 的记录在 `槽2` 对应的组中。此刻我们需要找到 `槽2` 中主键值最小的那条记录，然后沿着单向链表遍历 `槽2` 中的记录。但是我们前边又说过，每个槽对应的记录都是该组中主键值最大的记录，这里 `槽2` 对应的记录是主键值为 `8` 的记录，怎么定位一个组中最小的记录呢？别忘了各个槽都是挨着的，我们可以很轻易的拿到 `槽1` 对应的记录（主键值为 `4`），该条记录的下一条记录就是 `槽2` 中主键值最小的记录，该记录的主键值为 `5`。所以我们可以从这条主键值为 `5` 的记录出发，遍历 `槽2` 中的各条记录，直到找到主键值为 `6` 的那条记录即可。由于一个组中包含的记录条数只能是 1~8 条，所以遍历一个组中的记录的代价是很小的。

## 3.4. Page Header（页面头部）

设计 `InnoDB` 的大叔们为了能得到一个数据页中存储的记录的状态信息，比如本页中已经存储了多少条记录，第一条记录的地址是什么，页目录中存储了多少个槽等等，特意在页中定义了一个叫 `Page Header` 的部分，它是 `页` 结构的第二部分，这个部分占用固定的 `56` 个字节，专门存储各种状态信息：

<table> <thead> <tr> <th> 名称 </th> <th> 占用空间大小 </th> <th> 描述 </th> </tr> </thead> <tbody> <tr> <td> <code> PAGE_N_DIR_SLOTS </code> </td> <td> <code> 2 </code> 字节 </td> <td> 在页目录中的槽数量 </td> </tr> <tr> <td> <code> PAGE_HEAP_TOP </code> </td> <td> <code> 2 </code> 字节 </td> <td> 还未使用的空间最小地址，也就是说从该地址之后就是 <code> Free Space </code> </td> </tr> <tr> <td> <code> PAGE_N_HEAP </code> </td> <td> <code> 2 </code> 字节 </td> <td> 本页中的记录的数量（包括最小和最大记录以及标记为删除的记录）</td> </tr> <tr> <td> <code> PAGE_FREE </code> </td> <td> <code> 2 </code> 字节 </td> <td> 第一个已经标记为删除的记录地址（各个已删除的记录通过 <code> next_record </code> 也会组成一个单链表，这个单链表中的记录可以被重新利用）</td> </tr> <tr> <td> <code> PAGE_GARBAGE </code> </td> <td> <code> 2 </code> 字节 </td> <td> 已删除记录占用的字节数 </td> </tr> <tr> <td> <code> PAGE_LAST_INSERT </code> </td> <td> <code> 2 </code> 字节 </td> <td> 最后插入记录的位置 </td> </tr> <tr> <td> <code> PAGE_DIRECTION </code> </td> <td> <code> 2 </code> 字节 </td> <td> 记录插入的方向 </td> </tr> <tr> <td> <code> PAGE_N_DIRECTION </code> </td> <td> <code> 2 </code> 字节 </td> <td> 一个方向连续插入的记录数量 </td> </tr> <tr> <td> <code> PAGE_N_RECS </code> </td> <td> <code> 2 </code> 字节 </td> <td> 该页中记录的数量（不包括最小和最大记录以及被标记为删除的记录）</td> </tr> <tr> <td> <code> PAGE_MAX_TRX_ID </code> </td> <td> <code> 8 </code> 字节 </td> <td> 修改当前页的最大事务 ID，该值仅在二级索引中定义 </td> </tr> <tr> <td> <code> PAGE_LEVEL </code> </td> <td> <code> 2 </code> 字节 </td> <td> 当前页在 B + 树中所处的层级 </td> </tr> <tr> <td> <code> PAGE_INDEX_ID </code> </td> <td> <code> 8 </code> 字节 </td> <td> 索引 ID，表示当前页属于哪个索引 </td> </tr> <tr> <td> <code> PAGE_BTR_SEG_LEAF </code> </td> <td> <code> 10 </code> 字节 </td> <td> B + 树叶子段的头部信息，仅在 B + 树的 Root 页定义 </td> </tr> <tr> <td> <code> PAGE_BTR_SEG_TOP </code> </td> <td> <code> 10 </code> 字节 </td> <td> B + 树非叶子段的头部信息，仅在 B + 树的 Root 页定义 </td> </tr> </tbody> </table>

其中前半段字段含义已经介绍过了，这里我们先唠叨一下 `PAGE_DIRECTION` 和 `PAGE_N_DIRECTION` 的意思：

- PAGE_DIRECTION
    
    假如新插入的一条记录的主键值比上一条记录的主键值大，我们说这条记录的插入方向是右边，反之则是左边。用来表示最后一条记录插入方向的状态就是 `PAGE_DIRECTION`。
    
- PAGE_N_DIRECTION
    
    假设连续几次插入新记录的方向都是一致的，`InnoDB` 会把沿着同一个方向插入记录的条数记下来，这个条数就用 `PAGE_N_DIRECTION` 这个状态表示。当然，如果最后一条记录的插入方向改变了的话，这个状态的值会被清零重新统计。

## 3.5. File Header（文件头部）

`Page Header` 是专门针对 `数据页` 记录的各种状态信息，比方说页里头有多少个记录，有多少个槽。我们现在描述的 `File Header` 针对各种类型的页都通用，也就是说不同类型的页都会以 `File Header` 作为第一个组成部分，它描述了一些针对各种页都通用的一些信息，比方说这个页的编号是多少，它的上一个页、下一个页是谁。这个部分占用固定的 `38` 个字节，是由下边这些内容组成的：

<table> <thead> <tr> <th> 名称 </th> <th> 占用空间大小 </th> <th> 描述 </th> </tr> </thead> <tbody> <tr> <td> <code> FIL_PAGE_SPACE_OR_CHKSUM </code> </td> <td> <code> 4 </code> 字节 </td> <td> 页的校验和（checksum 值）</td> </tr> <tr> <td> <code> FIL_PAGE_OFFSET </code> </td> <td> <code> 4 </code> 字节 </td> <td> 页号 </td> </tr> <tr> <td> <code> FIL_PAGE_PREV </code> </td> <td> <code> 4 </code> 字节 </td> <td> 上一个页的页号 </td> </tr> <tr> <td> <code> FIL_PAGE_NEXT </code> </td> <td> <code> 4 </code> 字节 </td> <td> 下一个页的页号 </td> </tr> <tr> <td> <code> FIL_PAGE_LSN </code> </td> <td> <code> 8 </code> 字节 </td> <td> 页面被最后修改时对应的日志序列位置（英文名是：Log Sequence Number）</td> </tr> <tr> <td> <code> FIL_PAGE_TYPE </code> </td> <td> <code> 2 </code> 字节 </td> <td> 该页的类型 </td> </tr> <tr> <td> <code> FIL_PAGE_FILE_FLUSH_LSN </code> </td> <td> <code> 8 </code> 字节 </td> <td> 仅在系统表空间的一个页中定义，代表文件至少被刷新到了对应的 LSN 值 </td> </tr> <tr> <td> <code> FIL_PAGE_ARCH_LOG_NO_OR_SPACE_ID </code> </td> <td> <code> 4 </code> 字节 </td> <td> 页属于哪个表空间 </td> </tr> </tbody> </table>

对照着这个表格，我们看几个目前比较重要的部分：

- FIL_PAGE_SPACE_OR_CHKSUM
    
    这个代表当前页面的校验和（checksum）。
    
-   FIL_PAGE_OFFSET
    
    每一个 `页` 都有一个单独的页号，就跟你的身份证号码一样，`InnoDB` 通过页号来可以唯一定位一个 `页`。
    
-   FIL_PAGE_TYPE
    
    这个代表当前 `页` 的类型，我们前边说过，`InnoDB` 为了不同的目的而把页分为不同的类型，我们上边介绍的其实都是存储记录的 `数据页`，其实还有很多别的类型的页，具体如下表：

	<table> <thead> <tr> <th> 类型名称 </th> <th> 十六进制 </th> <th> 描述 </th> </tr> </thead> <tbody> <tr> <td> <code> FIL_PAGE_TYPE_ALLOCATED </code> </td> <td> 0x0000 </td> <td> 最新分配，还没使用 </td> </tr> <tr> <td> <code> FIL_PAGE_UNDO_LOG </code> </td> <td> 0x0002 </td> <td> Undo 日志页 </td> </tr> <tr> <td> <code> FIL_PAGE_INODE </code> </td> <td> 0x0003 </td> <td> 段信息节点 </td> </tr> <tr> <td> <code> FIL_PAGE_IBUF_FREE_LIST </code> </td> <td> 0x0004 </td> <td> Insert Buffer 空闲列表 </td> </tr> <tr> <td> <code> FIL_PAGE_IBUF_BITMAP </code> </td> <td> 0x0005 </td> <td> Insert Buffer 位图 </td> </tr> <tr> <td> <code> FIL_PAGE_TYPE_SYS </code> </td> <td> 0x0006 </td> <td> 系统页 </td> </tr> <tr> <td> <code> FIL_PAGE_TYPE_TRX_SYS </code> </td> <td> 0x0007 </td> <td> 事务系统数据 </td> </tr> <tr> <td> <code> FIL_PAGE_TYPE_FSP_HDR </code> </td> <td> 0x0008 </td> <td> 表空间头部信息 </td> </tr> <tr> <td> <code> FIL_PAGE_TYPE_XDES </code> </td> <td> 0x0009 </td> <td> 扩展描述页 </td> </tr> <tr> <td> <code> FIL_PAGE_TYPE_BLOB </code> </td> <td> 0x000A </td> <td> 溢出页 </td> </tr> <tr> <td> <code> FIL_PAGE_INDEX </code> </td> <td> 0x45BF </td> <td> 索引页，也就是我们所说的 <code> 数据页 </code> </td> </tr> </tbody> </table>

- FIL_PAGE_PREV 和 FIL_PAGE_NEXT

`InnoDB` 都是以页为单位存放数据的，`FIL_PAGE_PREV` 和 `FIL_PAGE_NEXT` 就分别代表本页的上一个和下一个页的页号。这样通过建立一个双向链表把许许多多的页就都串联起来了，而无需这些页在物理上真正连着。

需要注意的是，并不是所有类型的页都有上一个和下一个页的属性，不过 `数据页`（也就是类型为 `FIL_PAGE_INDEX` 的页）是有这两个属性的，所以所有的数据页其实是一个双链表，就像这样：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209050009280.png)

## 3.6. File Trailer（文件尾部）

`InnoDB` 存储引擎会把数据存储到磁盘上，但是磁盘速度太慢，需要以 `页` 为单位把数据加载到内存中处理，如果该页中的数据在内存中被修改了，那么在修改后的某个时间需要把数据同步到磁盘中。但是在同步了一半的时候中断电了咋办，这不是莫名尴尬么？

为了检测一个页是否完整（也就是在同步的时候有没有发生只同步一半的尴尬情况），设计 `InnoDB` 的大叔们在每个页的尾部都加了一个 `File Trailer` 部分，这个部分由 `8` 个字节组成，可以分成 2 个小部分：

- 前 4 个字节代表页的校验和
    
    这个部分是和 `File Header` 中的校验和相对应的。每当一个页面在内存中修改了，在同步之前就要把它的校验和算出来，因为 `File Header` 在页面的前边，所以校验和会被首先同步到磁盘，当完全写完时，校验和也会被写到页的尾部，如果完全同步成功，则页的首部和尾部的校验和应该是一致的。如果写了一半儿断电了，那么在 `File Header` 中的校验和就代表着已经修改过的页，而在 `File Trailer` 中的校验和代表着原先的页，二者不同则意味着同步中间出了错。
    
-   后 4 个字节代表页面被最后修改时对应的日志序列位置（LSN）
    
    这个部分也是为了校验页的完整性的，只不过我们目前还没说 `LSN` 是个什么意思，所以大家可以先不用管这个属性。
    
这个 `File Trailer` 与 `File Header` 类似，都是所有类型的页通用的。

