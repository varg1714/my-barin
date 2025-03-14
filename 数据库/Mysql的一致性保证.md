#数据库 #Mysql 

# 1. redo 日志

在 [[Mysql的存储结构#6 Mysql 的缓冲区（Buffer Pool）|Buffer Pool 缓冲区]]中，对数据的[[Mysql的存储结构#6 1 3 flush 链表的管理|修改体现在缓冲区]] ，那么如何保证 `持久性` 呢？在事务提交完成之前把该事务所修改的所有页面都刷新到磁盘，但是这样性能并不好：

- 刷新一个完整的数据页太浪费了
    仅修改了页面中一部分数据但是需要以页为最小单位将其刷新到磁盘。
- 随机 IO 刷起来比较慢
    修改不相邻页面时刷入磁盘需要随机 IO。

只是想让**已经提交了的事务对数据库中数据所做的修改永久生效，即使后来系统崩溃，在重启后也能把这种修改恢复出来**。所以没有必要在每次事务提交时就把该事务在内存中修改过的全部页面刷新到磁盘，只需要把修改了哪些东西记录一下就好。将这个变化刷新到磁盘中即可满足持久性的需求。

**在系统崩溃重启时需要按照上述内容所记录的步骤重新更新数据页，所以上述内容也被称之为 `重做日志`，英文名为 `redo log`**，将 `redo` 日志刷新到磁盘的好处如下：

-   `redo` 日志占用的空间非常小
    存储表空间 ID、页号、偏移量以及需要更新的值所需的存储空间很小。
-   `redo` 日志是顺序写入磁盘的
    redo 日志是按照产生的顺序写入磁盘的。

## 1.1. redo 日志格式

`redo` 日志本质上只是记录了一下事务对数据库做了哪些修改：

- `type` ：该条 `redo` 日志的类型。
- `space ID` ：表空间 ID。
- `page number` ：页号。
- `data` ：该条 `redo` 日志的具体内容。

### 1.1.1. 简单的 redo 日志类型

简单的 redo 日志只需要记录哪个页面哪个偏移量处修改了几个字节的值，具体被修改的内容是什么即可。这种 `redo` 日志称之为 `物理日志`，如表示写了 1、2、4、8 字节类型的日志：

- `MLOG_1BYTE`
- `MLOG_2BYTE`
- `MLOG_4BYTE`
- `MLOG_8BYTE`
- `MLOG_WRITE_STRING`：表示在页面的某个偏移量处写入一串数据。

如自增主键的 `Max Row ID` 属性实际占用 8 个字节的存储空间，所以在修改页面中的该属性时，会记录一条类型为 `MLOG_8BYTE` 的 `redo` 日志，`MLOG_8BYTE` 的 `redo` 日志结构如下所示：

![](https://r2.129870.xyz/img/202209190003003.png)

`MLOG_WRITE_STRING` 类型的 `redo` 日志表示写入一串数据，但是因为不能确定写入的具体数据占用多少字节，所以需要在日志结构中添加一个 `len` 字段：

![](https://r2.129870.xyz/img/202209190003543.png)

### 1.1.2. 复杂的 redo 日志

一条语句会修改非常多的页面，包括系统数据页面和用户数据页面。以一条 `INSERT` 语句为例，它除了要向 `B+` 树的页面中插入数据，也可能更新系统数据 `Max Row ID` 的值等：

- 表中包含多少个索引，一条 `INSERT` 语句就可能更新多少棵 `B+` 树
- 针对某一棵 `B+` 树来说，既可能更新叶子节点页面，也可能更新非叶子节点页面，也可能创建新的页面

一个数据页中除了存储实际的记录，还有 `File Header`、`Page Header`、`Page Directory` 等元信息需要更新：

-  [[Mysql的存储结构#3 3 Page Directory（页目录）|Page Directory]] 的槽信息
- [[Mysql的存储结构#3 4 Page Header（页面头部）|Page Header]] 的页面统计信息
    比如 `PAGE_N_DIR_SLOTS` 表示的槽数量可能会更改，`PAGE_HEAP_TOP` 代表的还未使用的空间最小地址可能会更改，`PAGE_N_HEAP` 代表的本页面中的记录数量可能会更改等。
- 上一条记录的 [[Mysql的存储结构#2.1.1.3. 记录头信息|记录头信息]]中的 `next_record` 信息

![](https://r2.129870.xyz/img/202209190008562.png)

针对这种复杂的情况，如果使用简单的物理 `redo` 日志来记录这些修改时，可以有两种解决方案：

- 方案一：在每个修改的地方都记录一条 `redo` 日志。
    这样子可能记录的 `redo` 日志占用的空间都比整个页面占用的空间都多了。
- 方案二：将整个页面的 `第一个被修改的字节` 到 `最后一个修改的字节` 之间所有的数据当成是一条物理 `redo` 日志中的具体数据。
    `第一个被修改的字节` 到 `最后一个修改的字节` 之间仍然有许多没有修改过的数据，把这些没有修改的数据也加入到 `redo` 日志中去会很浪费。

正因为上述两种使用物理 `redo` 日志的方式来记录某个页面中做了哪些修改比较浪费，Mysql 提出了一些新的 `redo` 日志类型，比如：

- `MLOG_REC_INSERT`（对应的十进制数字为 `9`）：表示插入一条使用非紧凑行格式的记录时的 `redo` 日志类型。
- `MLOG_COMP_REC_INSERT`（对应的十进制数字为 `38`）：表示插入一条使用[[Mysql的存储结构#2 1 COMPACT 格式|紧凑行格式]] 的记录时的 `redo` 日志类型。
- `MLOG_COMP_PAGE_CREATE`（`type` 字段对应的十进制数字为 `58`）：表示创建一个存储紧凑行格式记录的页面的 `redo` 日志类型。
- `MLOG_COMP_REC_DELETE`（`type` 字段对应的十进制数字为 `42`）：表示删除一条使用紧凑行格式记录的 `redo` 日志类型。
- `MLOG_COMP_LIST_START_DELETE`（`type` 字段对应的十进制数字为 `44`）：表示从某条给定记录开始删除页面中的一系列使用紧凑行格式记录的 `redo` 日志类型。
- `MLOG_COMP_LIST_END_DELETE`（`type` 字段对应的十进制数字为 `43`）：与 `MLOG_COMP_LIST_START_DELETE` 类型的 `redo` 日志呼应，表示删除一系列记录直到 `MLOG_COMP_LIST_END_DELETE` 类型的 `redo` 日志对应的记录为止。
	数据页中的记录是按照索引列大小的顺序组成单向链表的，有时候我们会有删除索引列的值在某个区间范围内的所有记录的需求，这时候如果我们每删除一条记录就写一条 redo 日志的话，效率可能有点低，所以提出 `MLOG_COMP_LIST_START_DELETE` 和 `MLOG_COMP_LIST_END_DELETE` 类型的 redo 日志，可以很大程度上减少 redo 日志的条数。
- `MLOG_ZIP_PAGE_COMPRESS`（`type` 字段对应的十进制数字为 `51`）：表示压缩一个数据页的 `redo` 日志类型。

这些类型的 `redo` 日志**既包含 `物理` 层面的意思，也包含 `逻辑` 层面的意思**：

- 物理层面看，这些日志都指明了对哪个表空间的哪个页进行了修改。
- 逻辑层面看，在系统崩溃重启时，**并不能直接根据这些日志里的记载将页面内的某个偏移量处恢复成某个数据，而是需要调用一些事先准备好的函数，执行完这些函数后才可以将页面恢复成系统崩溃前的样子**。

以类型为 `MLOG_COMP_REC_INSERT` 这个代表插入一条使用紧凑行格式的记录时的 `redo` 日志为例来说明 `物理` 层面和 `逻辑` 层面的含义：

![](https://r2.129870.xyz/img/202209190013101.png)

这个类型为 `MLOG_COMP_REC_INSERT` 的 `redo` 日志并没有记录 `PAGE_N_DIR_SLOTS` 的值等数据修改后的值，而只是把在本页面中插入一条记录所有必备的要素记了下来。之后系统崩溃重启时，服务器会调用相关向某个页面插入一条记录的那个函数，而 `redo` 日志中的那些数据就可以被当成是调用这个函数所需的参数，在调用完该函数后，页面中的 `PAGE_N_DIR_SLOTS`、`PAGE_HEAP_TOP`、`PAGE_N_HEAP` 等等的值也就都被恢复到系统崩溃前的样子了。这就是所谓的 `逻辑` 日志的意思。

## 1.2. Mini-Transaction

### 1.2.1. redo 日志组

语句在执行过程中可能修改若干个页面而产生多个日志并会被划分成不同的组：

- 更新 `Max Row ID` 属性时产生的 `redo` 日志
- 向聚簇索引对应 `B+` 树的页面中插入一条记录时产生的 `redo` 日志
- 向某个二级索引对应 `B+` 树的页面中插入一条记录时产生的 `redo` 日志

在执行这些需要保证**原子性的操作时必须以 `组` 的形式来记录的 `redo` 日志，在进行系统崩溃重启恢复时，针对某个组中的 `redo` 日志，要么把全部的日志都恢复掉，要么一条也不恢复**。

- 组内包含多个 redo 日志
	有多条日志时，组 redo 日志结尾后边加上一条特殊类型的 redo 日志，该类型名称为 MLOG_MULTI_REC_END，type 字段对应的十进制数字为 31，该类型的 redo 日志结构很简单，只有一个 type 字段：

	![](https://r2.129870.xyz/img/202209190022227.png)

	所以某个需要保证原子性的操作产生的一系列 `redo` 日志必须要以一个类型为 `MLOG_MULTI_REC_END` 结尾：

	![](https://r2.129870.xyz/img/202209190022311.png)

	在系统崩溃重启进行恢复时，只有当解析到类型为 `MLOG_MULTI_REC_END` 的 `redo` 日志，才认为解析到了一组完整的 `redo` 日志。

- 组内仅一条日志

	若组内仅一条日志（如更新 `Max Row ID` 属性）则可以通过一个 bit 位进行区分：

	![](https://r2.129870.xyz/img/202209190024509.png)

	如果 `type` 字段的第一个比特位为 `1`，代表该需要保证原子性的操作只产生了单一的一条 `redo` 日志，否则表示该需要保证原子性的操作产生了一系列的 `redo` 日志。

### 1.2.2. Mini-Transaction 的概念

对底层页面中的一次原子访问的过程称之为一个 `Mini-Transaction`，简称 `mtr`，一个 `mtr` 可以包含一组 `redo` 日志，在进行崩溃恢复时这一组 `redo` 日志作为一个不可分割的整体。

一个事务可以包含若干条语句，每一条语句其实是由若干个 `mtr` 组成，每一个 `mtr` 又可以包含若干条 `redo` 日志：

![](https://r2.129870.xyz/img/202209190025616.png)

## 1.3. redo 日志写入过程

### 1.3.1. redo 日志 block 块

 `mtr` 生成的 `redo` 日志被放在了大小为 `512字节` 的 `页` 中。 `redo` 日志的页称为 `block`：

![](https://r2.129870.xyz/img/202209190026884.png)

`redo` 日志实际存储在 `496` 字节大小的 `log block body` 中：

![](https://r2.129870.xyz/img/202209190026412.png)

其中 `log block header` 包含以下属性：

- `LOG_BLOCK_HDR_NO`
	每一个 block 都有一个大于 0 的唯一标号，本属性就表示该标号值。
- `LOG_BLOCK_HDR_DATA_LEN`
	表示 block 中已经使用了多少字节。
- `LOG_BLOCK_FIRST_REC_GROUP`
	`LOG_BLOCK_FIRST_REC_GROUP` 代表该 block 中第一个 `mtr` 生成的 `redo` 日志记录组的偏移量（即 block 里第一个 `mtr` 生成的第一条 `redo` 日志的偏移量）。
- `LOG_BLOCK_CHECKPOINT_NO`
	 `checkpoint` 的序号。

`log block trailer` 中属性的意思如下：

- `LOG_BLOCK_CHECKSUM` ：表示 block 的校验值，用于正确性校验。

### 1.3.2. redo 日志缓冲区

`redo` 日志写入时会先写在 `redo log buffer` 的连续内存空间中，这片内存空间被划分成若干个连续的 `redo log block`：

![](https://r2.129870.xyz/img/202209190028627.png)

`log buffer` 中写入 `redo` 日志的过程是顺序的， `buf_free` 用于标记当前可写入位置：

![](https://r2.129870.xyz/img/202209190029079.png)

多个事务的 `mtr` 可能是交替写入 `log buffer` 的：

![](https://r2.129870.xyz/img/202209190030328.png)

## 1.4. redo 日志文件

### 1.4.1. redo 日志缓冲区刷盘时机

#### 1.4.1.1. 刷盘时机

 `log buffer` 中的日志在一些情况下它们会被刷新到磁盘里：

- `log buffer` 空间不足时
    如果当前写入 `log buffer` 的 `redo` 日志量已经占满了 `log buffer` 总容量（`innodb_log_buffer_size`）的大约一半左右，就需要把这些日志刷新到磁盘上。
- 事务提交后
    事务提交时为了保证持久性，必须要把修改这些页面对应的 `redo` 日志刷新到磁盘。
- 脏页刷新时
    将某个脏页刷新到磁盘前，会保证先将该脏页对应的 redo 日志刷新到磁盘中。
    > [!tip] redo 日志的刷新顺序
    > redo 日志是顺序刷新的，所以在将某个脏页对应的 redo 日志从 `redo log buffer` 刷新到磁盘时，也会保证将在其之前产生的 redo 日志也刷新到磁盘。

- 后台线程定时刷新
    后台有一个线程，每秒都会刷新一次 `log buffer` 中的 `redo` 日志到磁盘。
- 正常关闭服务器时
- 做 `checkpoint` 时

**因此 redo 日志的写盘并不一定是随着事务的提交才写入重做日志文件的，而是随着事务的开始逐步开始的。**

#### 1.4.1.2. 刷盘控制

`redo` 刷盘时机由 `innodb_flush_log_at_trx_commit` 系统变量控制：

- `0` ：事务提交时不立即向磁盘中同步 `redo` 日志，而是交给后台线程定时刷新
    后台线程每 N 秒将 redo log buffer 的记录写入 redo log 文件，并且将文件刷入硬件存储 1 次。N 由 `innodb_flush_log_at_timeout` 控制。
- `1` ：事务提交时需要将 `redo` 日志同步到磁盘，可以保证事务的 `持久性`。`1` 也是 `innodb_flush_log_at_trx_commit` 的默认值
- `2` ：事务提交时需要将 `redo` 日志写到操作系统的缓冲区中，但并不需要保证将日志真正的刷新到磁盘
    在操作系统未崩溃的前提下仍能保证持久性。

### 1.4.2. redo 日志文件组

`MySQL` 的数据目录（使用 `SHOW VARIABLES LIKE 'datadir'` 查看）下默认有两个名为 `ib_logfile0` 和 `ib_logfile1` 的日志文件，日志文件可以由以下系统变量配置：

- `innodb_log_group_home_dir`
     `redo` 日志文件所在的目录。
- `innodb_log_file_size`
    该参数指定了每个 `redo` 日志文件的大小，在 `MySQL 5.7.21` 这个版本中的默认值为 `48MB`，
- `innodb_log_files_in_group`
    `redo` 日志文件的个数，默认值为 2，最大值为 100。

磁盘上的 `redo` 日志文件可能有多个，以 `ib_logfile[数字]` 的形式进行命名。在将 `redo` 日志写入 `日志文件组` 时是一个循环写的过程：

![](https://r2.129870.xyz/img/202209190045531.png)

### 1.4.3. redo 日志文件格式

`log buffer` 和 `redo` 日志文件都是由若干个 `512` 字节大小的 block 组成。

`redo` 日志文件组中的每个文件由两部分组成：

- 前 2048 个字节，即前 4 个 block 是用来存储管理信息。
- 第 2048 字节往后用来存储 `log buffer` 中的 block 镜像。

日志循环写时实际上时写入以下部分：

![](https://r2.129870.xyz/img/202209190046839.png)

 `redo` 日志文件前 4 个lock 的格式：

![](https://r2.129870.xyz/img/202209190047929.png)

这 4 个 block 分别是：

- `log file header` ：描述该 `redo` 日志文件的一些整体属性

	![](https://r2.129870.xyz/img/202209190047194.png)

	各个属性的具体释义如下：

	<table> <thead> <tr> <th> 属性名 </th> <th> 长度（单位：字节）</th> <th> 描述 </th> </tr> </thead> <tbody> <tr> <td> <code> LOG_HEADER_FORMAT </code> </td> <td> <code> 4 </code> </td> <td> <code> redo </code> 日志的版本</td> </tr> <tr> <td> <code> LOG_HEADER_PAD1 </code> </td> <td> <code> 4 </code> </td> <td> 字节填充</td> </tr> <tr> <td> <code> LOG_HEADER_START_LSN </code> </td> <td> <code> 8 </code> </td> <td> 标记本 <code> redo </code> 日志文件开始的 LSN 值，也就是文件偏移量为 2048 字节处对应的 LSN 值）。</td> </tr> <tr> <td> <code> LOG_HEADER_CREATOR </code> </td> <td> <code> 32 </code> </td> <td> 一个字符串，标记本 <code> redo </code> 日志文件的创建者是谁。正常运行时该值为 <code> MySQL </code> 的版本号，比如：<code> "MySQL 5.7.21" </code>，使用 <code> mysqlbackup </code> 命令创建的 <code> redo </code> 日志文件的该值为 <code> "ibbackup" </code> 和创建时间。</td> </tr> <tr> <td> <code> LOG_BLOCK_CHECKSUM </code> </td> <td> <code> 4 </code> </td> <td> 本 block 的校验值</td> </tr> </tbody> </table>

- `checkpoint1` ：记录关于 `checkpoint` 的一些属性。

	![](https://r2.129870.xyz/img/202209190048041.png)

	各个属性的具体释义如下：

	<table> <thead> <tr> <th> 属性名 </th> <th> 长度（单位：字节）</th> <th> 描述 </th> </tr> </thead> <tbody> <tr> <td> <code> LOG_CHECKPOINT_NO </code> </td> <td> <code> 8 </code> </td> <td> 服务器做 <code> checkpoint </code> 的编号，每做一次 <code> checkpoint </code>，该值就加 1。</td> </tr> <tr> <td> <code> LOG_CHECKPOINT_LSN </code> </td> <td> <code> 8 </code> </td> <td> 服务器做 <code> checkpoint </code> 结束时对应的 <code> LSN </code> 值，系统崩溃恢复时将从该值开始。</td> </tr> <tr> <td> <code> LOG_CHECKPOINT_OFFSET </code> </td> <td> <code> 8 </code> </td> <td>  <code> LOG_CHECKPOINT_LSN </code> 值在 <code> redo </code> 日志文件组中的偏移量 </td> </tr> <tr> <td> <code> LOG_CHECKPOINT_LOG_BUF_SIZE </code> </td> <td> <code> 8 </code> </td> <td> 服务器在做 <code> checkpoint </code> 操作时对应的 <code> log buffer </code> 的大小 </td> </tr> <tr> <td> <code> LOG_BLOCK_CHECKSUM </code> </td> <td> <code> 4 </code> </td> <td> 本 block 的校验值，所有 block 都有，我们不关心 </td> </tr> </tbody> </table>

- 第三个 block 未使用
- `checkpoint2` ：结构和 `checkpoint1` 一样。

## 1.5. Log Sequence Number

Mysql 为记录已经写入的 `redo` 日志量，设计了一个称之为 `Log Sequence Number` 的全局变量，`日志序列号`，简称 `lsn`， 初始值为 `8704`。在**统计 `lsn` 的增长量时，是按照实际写入的日志量加上占用的 `log block header` 和 `log block trailer` 来计算的**。

 `lsn` 的值代表系统写入的 `redo` 日志量的一个总和，一个 `mtr` 中产生多少日志，`lsn` 的值就增加多少，**借此很容易计算某一个 `lsn` 值在 `redo` 日志文件组中的偏移量**。

### 1.5.1. flushed_to_disk_lsn

`redo` 日志首先写到 `log buffer` 中，之后才会被刷新到磁盘上的 `redo` 日志文件。 `buf_next_to_write` 全局变量标记当前 `log buffer` 中哪些日志被刷新到磁盘中了：

![](https://r2.129870.xyz/img/202209190052707.png)

 `lsn` 是表示当前系统中写入的 `redo` 日志量，这包括了写到 `log buffer` 而没有刷新到磁盘的日志。相应的，Mysql 提出了一个表示刷新到磁盘中的 `redo` 日志量的全局变量，称之为 `flushed_to_disk_lsn`。当有新的 `redo` 日志写入到 `log buffer` 时，首先 `lsn` 的值会增长，但 `flushed_to_disk_lsn` 不变，随后随着不断有 `log buffer` 中的日志被刷新到磁盘上，`flushed_to_disk_lsn` 的值也跟着增长。如果两者的值相同时，说明 log buffer 中的所有 redo 日志都已经刷新到磁盘中了。

### 1.5.2. flush 链表中的 LSN

一个 `mtr` 代表一次对底层页面的原子访问，在访问过程中可能会产生一组不可分割的 `redo` 日志，在 `mtr` 结束时，会把这一组 `redo` 日志写入到 `log buffer` 中。除此之外还会把在 mtr 执行过程中可能修改过的页面加入到 Buffer Pool 的 [[Mysql的存储结构#6 1 3 flush 链表的管理|flush 链表]]。

![](https://r2.129870.xyz/img/202209190057119.png)

当第一次修改某个缓存在 `Buffer Pool` 中的页面时，就会把这个页面对应的控制块插入到 `flush 链表` 的头部， **flush 链表中的脏页是按照页面的第一次修改时间从大到小进行排序的**。在这个过程中会在缓存页对应的控制块中记录两个关于页面何时修改的属性：

- `oldest_modification` ：页面初始被修改时对应的系统 `lsn` 值
- `newest_modification` ：页面最近一次修改后对应的系统 `lsn` 值

## 1.6. checkpoint

 `redo` 日志文件组容量是有限的，因此采取循环使用 `redo` 日志文件的方式，这就会造成最后写的 `redo` 日志与最开始写的 `redo` 日志 `追尾`。

**判断某些 redo 日志占用的磁盘空间是否可以覆盖的依据就是它对应的脏页是否已经刷新到磁盘里**。

Mysql 提出了一个全局变量 `checkpoint_lsn` 来代表当前系统中可以被覆盖的 `redo` 日志总量是多少，即这些 `redo 日志` 对应的脏页都已经刷新到了磁盘，这个变量初始值也是 `8704`。

`checkpoint_lsn` 由 `checkpoint` 操作生成：

1. 计算当前系统中可以被覆盖的 `redo` 日志对应的 `lsn` 最大值
    `redo` 日志可以被覆盖的前提是它对应的脏页被刷到了磁盘，因此只要计算出当前系统中被最早修改的脏页对应的 `oldest_modification` 值，就可以把该脏页的 `oldest_modification` 赋值给 `checkpoint_lsn`。
2. 将 `checkpoint_lsn` 和对应的 `redo` 日志文件组偏移量以及此次 `checkpint` 的编号写到日志文件的管理信息（`checkpoint1` 或者 `checkpoint2`）中
    写入以下三部分数据：
    -  `checkpoint_no`：每做一次 `checkpoint`，该变量的值就加 1
    - `checkpoint_lsn` 
    - `checkpoint_offset`：`checkpoint_lsn` 在 `redo` 日志文件组中对应的偏移量

    **关于 checkpoint 的信息只会被写到日志文件组的第一个日志文件的管理信息中**。当 `checkpoint_no` 的值是偶数时，就写到 `checkpoint1` 中，是奇数时，就写到 `checkpoint2` 中。

记录完 `checkpoint` 的信息之后，`redo` 日志文件组中各个 `lsn` 值的关系如下：

![](https://r2.129870.xyz/img/202209190101664.png)

### 1.6.1. 批量从 flush 链表中刷出脏页

后台的线程会对 `LRU 链表` 和 `flush 链表` 进行 [[Mysql的存储结构#6 1 5 刷新脏页到磁盘|刷脏操作]]，但是如果当前系统修改页面的操作十分频繁，系统 `lsn` 值增长就会过快。如果后台的刷脏操作不能将脏页刷出，那么系统无法及时做 `checkpoint`，可能就**需要用户线程同步的从 `flush 链表` 中把那些最早修改的脏页（`oldest_modification` 最小的脏页）刷新到磁盘**，这样这些脏页对应的 `redo` 日志就没用了，然后就可以去做 `checkpoint` 了。

### 1.6.2. 查看系统中的各种 LSN 值

可以使用 `SHOW ENGINE INNODB STATUS` 命令查看当前 `InnoDB` 存储引擎中的各种 `LSN` 值的情况：

- `Log sequence number` ：当前系统的 `lsn` 值，即当前系统已经写入的 `redo` 日志量，包括写入 `log buffer` 中的日志
- `Log flushed up to` ： `flushed_to_disk_lsn` 的值
- `Pages flushed up to` ： `flush 链表` 中被最早修改的那个页面对应的 `oldest_modification` 属性值
- `Last checkpoint at` ：当前系统的 `checkpoint_lsn` 值

> [!question]- 可以只使用 flushed_to_disk_lsn 不使用 checkpoint_lsn 吗？
>  flushed_to_disk_lsn 用于标记当前已刷新到磁盘的日志的位置，而 checkpoint_lsn 代表可被覆盖的 redo 日志的起点。日志被刷新到磁盘并不代表日志可以被覆盖，只有当日志对应的脏页被刷新到磁盘后该日志才可以被覆盖，因此 checkpoint_lsn 必不可少。

## 1.7. 崩溃恢复

### 1.7.1. 确定恢复的起点

**`checkpoint_lsn` 之前的 `redo` 日志都可以被覆盖，也就是说这些 `redo` 日志对应的脏页都已经被刷新到磁盘中了**。对于 `checkpoint_lsn` 之后的 `redo` 日志，它们对应的脏页刷盘状态不能确定，所以需要从 `checkpoint_lsn` 开始读取 `redo` 日志来恢复页面。

`redo` 日志文件组的第一个文件的管理信息中有两个 block 都存储了 `checkpoint_lsn` 的信息，选取 `checkpoint_no` 值更大的那个即可，借此就能拿到最近发生的 `checkpoint` 对应的 `checkpoint_lsn` 值以及它在 `redo` 日志文件组中的偏移量 `checkpoint_offset`。

### 1.7.2. 确定恢复的终点

![](https://r2.129870.xyz/img/202209190106291.png)

普通 block 的 `log block header` 部分有一个称之为 `LOG_BLOCK_HDR_DATA_LEN` 的属性记录了当前 block 里使用了多少字节的空间。对于被填满的 block 来说，该值永远为 `512`。如果该属性的值不为 `512`，那么它就是此次崩溃恢复中需要扫描的最后一个 block。

### 1.7.3. 恢复过程

假设现在的 `redo` 日志文件中有 5 条 `redo` 日志：

![](https://r2.129870.xyz/img/202209190107937.png)

按照 `redo` 日志的顺序依次扫描 `checkpoint_lsn` 之后的各条 redo 日志，按照日志中记载的内容将对应的页面恢复出来：

- 使用哈希表
    根据 `redo` 日志的 `space ID` 和 `page number` 属性计算出散列值，多个 `space ID` 和 `page number` 都相同的 `redo` 日志之间使用链表连接起来：

	![](https://r2.129870.xyz/img/202209190107318.png)

	之后就可以遍历哈希表，一次性将一个页面修复好（避免了很多读取页面的随机 IO），这样可以加快恢复速度。

	同一个页面的 `redo` 日志是按照生成时间顺序进行排序的，所以恢复的时候也是按照这个顺序进行恢复。

- 跳过已经刷新到磁盘的页面
    在最近做的一次 `checkpoint` 后，后台线程又不断的从 `LRU 链表` 和 `flush 链表` 中将一些脏页刷出 `Buffer Pool`。这些在 `checkpoint_lsn` 之后的 `redo` 日志，如果它们对应的脏页已经刷新到磁盘，那在恢复时也就没有必要根据 `redo` 日志的内容修改该页面了。

    那在恢复时怎么知道某个 `redo` 日志对应的脏页是否在崩溃发生时已经刷新到磁盘了呢？每个页面中 [[Mysql的存储结构#3 5 File Header（文件头部）|File Header]] 的部分有一个 `FIL_PAGE_LSN` 的属性，记载了最近一次修改页面时对应的 `lsn` 值（即页面控制块中的 `newest_modification` 值）。如果在做了某次 `checkpoint` 之后有脏页被刷新到磁盘中，那么该页对应的 `FIL_PAGE_LSN` 代表的 `lsn` 值肯定大于 `checkpoint_lsn` 的值，凡是符合这种情况的页面就不需要重复执行 lsn 值小于 `FIL_PAGE_LSN` 的 redo 日志了。

# 2. binlog 日志

## 2.1. binlog 存储位置

![](https://r2.129870.xyz/img/20220513230808.png)

系统给 binlog cache 分配了一片内存，每个线程一个，但是共用同一份 binlog 文件。参数 `binlog_cache_size` 控制单个线程内 binlog cache 所占内存的大小。若超过该参数值，就要暂存到磁盘的临时文件 (不是最终的 binlog 文件)。事务提交时，执行器把 binlog cache 里的完整事务写入 binlog，并清空 binlog cache。所以如果经常有大事物则可以考虑增大这个值来减少刷盘的次数。

## 2.2. binlog 写盘状态与控制

binlog 写盘状态 ：

- 内存中
- write cache
	把日志写入到文件系统的 `page cache`，并没有把数据持久化到磁盘。
- fsync
    将数据持久化到磁盘。

write 和 fsync 的时机，由参数 `sync_binlog` 控制：

-   sync_binlog=0，每次提交事务都只 write，不 fsync
-   sync_binlog=1，每次提交事务都会执行 fsync
-   sync_binlog=N (N>1)，每次提交事务都 write，但累积 N 个事务后才 fsync

因此，在出现 I/O 瓶颈的场景，将 `sync_binlog` 设置成一个较大值，可提升性能。在实际的业务场景中，考虑到丢失日志量的可控性，一般不建议将这个参数设成 0，推荐将其设置为 100~1000 中的某个数值。

但将 `sync_binlog` 设置为 N，对应的风险是：若主机发生异常重启，会丢失最近 N 个事务的 binlog 日志。

## 2.3. binlog 日志格式

binlog 日志的格式：

- statement 格式
	语句级的格式，原始 SQL 存储。binlog 日志是按提交顺序存储的，如果提交顺序与执行顺序有差别，就会导致数据更新出问题。或者主备之间索引选择不一致，同样会导致数据更新出错。
- row 格式
	基于原始数据存储。可以通过命令 `show binlog events;` 查看当前 binlog 日志文件：

	![](https://r2.129870.xyz/img/20220514144956.png)

	更为详细的信息通过解析 binlog 日志查看，可以执行以下命令：`/var/lib/mysql# mysqlbinlog -vv binlog.000001 --start-position=3103917;`

	![](https://r2.129870.xyz/img/20220514145948.png)

	`binlog_row_image` 的默认配置是 FULL，因此 `delete_event` 里面，包含了删掉的行的所有字段的值。如果把 `binlog_row_image` 设置为 `MINIMAL`，则只会记录必要的信息。

- mixed 格式
	因为有些 statement 格式的 binlog 可能会导致主备不一致，所以要使用 row 格式。但 row 格式很占空间，在 sql 影响条数很大的情况下，不仅会占用更大的日志空间，同时写 binlog 也要耗费 IO 资源，影响执行速度。

	mixed 为混合日志，MySQL 自动根据 SQL 语句是否可能引起主备不一致的可能性判断使用 row 格式或者 statement 格式。即 mixed 格式可以利用 statment 格式的优点，同时又避免了数据不一致的风险。

# 3. binlog 日志与 redo 日志的合作

图中浅色框表示是在 InnoDB 内部执行的，深色框表示是在执行器中执行的：

![](https://r2.129870.xyz/img/20220508234920.png)

## 3.1. redolog 和 binlog 两次提交的原因

这种做法叫做[[分布式大纲#2 4 3 XA 实现之 2PC 协议|二阶段提交]]，为了保持数据的一致性而产生的。第一阶段提交事务请求，协调者向参与者发送事务内容，参与者执行并返回结果。第二阶段决定是否提交事务操作，根据情况让参与者提交或者回滚请求。

Update 语句过程中在写完第一个日志后，第二个日志还没有写完期间发生了 crash，会出现什么情况呢?

1. 先写 redo log 后写 binlog
	redo 日志成功保存但是 binlog 日志保存失败，导致恢复时主库包含该数据的更改但是从库缺少 binlog 无法更新。

2. 先写 binlog 后写 redo log
	从库可以根据 binlog 日志正常更新，但是主库缺少 redo 日志而无法更新。

如果不使用“两阶段提交”，那么数据库的状态就有可能和用它的日志恢复出来的库的状态不一致。

## 3.2. 崩溃时的处理策略

- 在 redo log 写入磁盘后崩溃
    redo log 是 prepare 阶段，binlog 尚未写入，此时可以直接丢弃 redo log 日志。
- binlog 写入后崩溃
    此时 binlog 已经写入，准备将 redo log 更新为 commit 的时候崩溃。当崩溃恢复后，进行以下的处理：
    1. 判断恢复的 redo log 日志中是否有 commit 标识，如果有，认为事务已经提交，使用 redolog 日志更新内存中的数据。
        这么处理的原因是为了保持主从数据的一致性。当主库 binlog 日志产生后会同步到从库，从库就更新了数据。若主库恢复后数据不更新，那么就会造成数据的不一致。

        另一方面，binlog 日志已经写入磁盘就代表着事务已经提交，那么执行该事务也是合理的。

    2. 没有 commit 标识，进行 binlog 日志的判断
        1. binlog 日志不完整，回滚事务，**正是由于这种崩溃后仍会回滚事务的场景，所以才需要将该事务对应的 undo 日志也写入磁盘，否则无法对其回滚**。
        2. binlog 日志完整，进行 redo log 日志的提交。
        
        一个事务的 binlog 是有完整格式的：
        - Statement 格式的 binlog，最后会有 `COMMIT`。
        - Row 格式的 binlog，最后会有一个 `XID event`。

redo log 与 binlog 通过事务 id 进行关联。

## 3.3. redo log 与 binlog 同时使用的原因

1. redo log 是为了数据崩溃恢复使用的，binlog 是为了记录数据操作使用的。redo log 记录了数据页发生的变化方便在崩溃时恢复。而 binlog 记录了 sql 的执行操作，用于数据备份。
2. redo log 记录是循环写的，写到尾部后会回到头部继续写。而 binlog 是追加的。
3. 两阶段提交的原因是为了让 redo log 有办法处理 binlog 存储失败后的问题。当 binlog 存储失败后，redo log 可以不提交。而如果不采用两阶段提交，那么当 binlog 日志出现问题的时候 redo log 就回滚不了了。

## 3.4. 组提交策略

redo log 与 binlog 在事务提交时会发生两次写盘，为了优化这一阶段，可以采取组提交策略。对于 redo log 与 binlog 的操作争取一次写盘完成。

![](https://r2.129870.xyz/img/20220513231453.png)

在将 redo log 写入到磁盘时，刷盘动作延迟到 `binlog write` 到 `page cache` 之后，这样 binlog 与 redo log 就有机会一同写入到磁盘中。不过通常情况下第 3 步执行得会很快，所以 binlog 的 write 和 fsync 间的间隔时间短，导致能集合到一起持久化的 binlog 比较少，因此 binlog 的组提交的效果通常不如 redo log 的效果那么好。

如果想提升 binlog 组提交的效果，可以通过设置 `binlog_group_commit_sync_delay` 和 `binlog_group_commit_sync_no_delay_count` 来实现：

- `binlog_group_commit_sync_delay` 参数，表示延迟多少微秒后才调用 fsync; 
- `binlog_group_commit_sync_no_delay_count` 参数，表示累积多少次以后才调用 fsync。

这两个条件是或的关系，所以当 `binlog_group_commit_sync_delay` 设置为 0 的时候，`binlog_group_commit_sync_no_delay_count` 也无效了。

组提交策略原理：

- redo log 与 binlog 都是顺序写，磁盘的顺序写比随机写要快。
- 组提交策略，大幅度降低磁盘的 IOPS 消耗。大幅降低的本质是**将多个事务的 redo log 和 binlog 进行了合并写入**，如果只是将单个事务的 redo log 和 binglog 合并的话那么性能提升并不是很明显。

# 4. uodo 日志

## 4.1. `trx_id` 事务 ID

`InnoDB` 记录行格式中：聚簇索引的记录除了会保存完整的用户数据以外，而且还会自动添加名为 `trx_id`、`roll_pointer` 的隐藏列，如果用户没有在表中定义主键以及 UNIQUE 键，还会自动添加一个名为 `row_id` 的隐藏列：

![](https://r2.129870.xyz/img/202209192249305.png)

`trx_id` 列是对这个聚簇索引记录做改动的语句所在的事务对应的 `事务 id` 。

## 4.2. undo 日志格式

为了实现事务的 `原子性`，`InnoDB` 存储引擎在实际进行增、删、改一条记录时，都需要先把对应的 `undo 日志` 记下来，一次改动可能对应一条或者多条日志。

一个事务在执行过程中可能产生多条日志，编号递增，这个编号被称之为 `undo no`。

 `undo 日志` 被记录到[[Mysql的存储结构#3 5 File Header（文件头部）|页面类型]]为 `FIL_PAGE_UNDO_LOG`的页面中。这些页面可以从系统表空间中分配，也可以从一种专门存放 `undo 日志` 的表空间（`undo tablespace`）中分配。

### 4.2.1. INSERT 操作的 undo 日志

回滚插入操作，在对应的 `undo` 日志把这条记录的主键信息记上即可。所以 Mysql 设计了一个类型为 `TRX_UNDO_INSERT_REC` 的 `undo 日志`：

![](https://r2.129870.xyz/img/202209192253198.png)

当向某个表中插入一条记录时，实际上需要向聚簇索引和所有的二级索引都插入一条记录。不过记录 undo 日志时，只需要考虑向聚簇索引插入记录时的情况就好了，因为聚簇索引记录和二级索引记录是一一对应的，**在回滚插入操作时，只需要知道这条记录的主键信息，然后根据主键信息做对应的删除操作，做删除操作时就会顺带着把所有二级索引中相应的记录也删除掉**。

#### 4.2.1.1. `roll_pointer` 隐藏列的含义

`roll_pointer` 这个占用 `7` 个字节的字段就是一个指向记录对应的 `undo 日志` 的一个指针。比方说我们向表里插入了 2 条记录，每条记录都有与其对应的一条 `undo 日志`。记录被存储到了类型为 `FIL_PAGE_INDEX` 的页面中，`undo 日志` 被存放到了类型为 `FIL_PAGE_UNDO_LOG` 的页面中。效果如图所示：

![](https://r2.129870.xyz/img/202209192256728.png)

### 4.2.2. DELETE 操作对应的 undo 日志

插入到页面中的记录会根据记录头信息中的 `next_record` 属性组成一个单向链表称之为 `正常记录链表`；在数据页结构中，被删除的记录其实也会根据 [[Mysql的存储结构#2.1.1.3. 记录头信息|记录头信息]]中的 `next_record` 属性组成一个链表，只不过这个链表中的记录占用的存储空间可以被重新利用，所以也称这个链表为 `垃圾链表`。[[Mysql的存储结构#3 4 Page Header（页面头部）|Page Header]] 部分有一个称之为 `PAGE_FREE` 的属性，它指向由被删除记录组成的垃圾链表中的头节点。

假设某个数据页面中的记录分布情况如下：

![](https://r2.129870.xyz/img/202209192259999.png)

页面的 `Page Header` 部分的 `PAGE_FREE` 属性的值代表指向 `垃圾链表` 头节点的指针。当使用 `DELETE` 语句删除数据时需要经历两个阶段：

1. 阶段一：仅仅将记录的 `delete_mask` 标识位设置为 `1`
    其他字段不做修改（其实会修改记录的 `trx_id`、`roll_pointer` 这些隐藏列的值），Mysql 把这个阶段称之为 `delete mark`。**此时被删除的记录并没有被加入到 `垃圾链表`，也就是此时记录处于一个 `中间状态`，在删除语句所在的事务提交之前，被删除的记录一直都处于这种所谓的 `中间状态`**。

    ![](https://r2.129870.xyz/img/202209192302780.png)

    > [!note] 只标记不移动的原因
    > 由于该删除操作对应的事务没有提交，因此不能将其移动到垃圾链表中，否则其它事务无法在数据链表中找到该数据。
2. 阶段二：当该删除语句所在的事务提交之后，会有专门的线程后来真正的把记录删除掉。
    把该记录从 `正常记录链表` 中移除，并且加入到 `垃圾链表` 中，然后调整页面的元信息，比如页面中的用户记录数量 `PAGE_N_RECS`、上次插入记录的位置 `PAGE_LAST_INSERT`、垃圾链表头节点的指针 `PAGE_FREE`、页面中可重用的字节数量 `PAGE_GARBAGE`、还有页目录的一些信息等等。这个阶段称之为 `purge`。

    ![](https://r2.129870.xyz/img/202209192302031.png)

    把 `阶段二` 执行完了，这条记录就算是真正的被删除掉了。这条已删除记录占用的存储空间也可以被重新利用了。

    将被删除记录加入到 `垃圾链表` 时，实际上加入到链表的头节点处，并会修改 `PAGE_FREE` 属性的值。页面的 `Page Header` 部分有一个 `PAGE_GARBAGE` 属性，该属性记录着当前页面中可重用存储空间占用的总字节数。每**当有已删除记录被加入到垃圾链表后，都会把这个 `PAGE_GARBAGE` 属性的值加上该已删除记录占用的存储空间大小**。

    `PAGE_FREE` 指向垃圾链表的头节点，之后每当新插入记录时，首先判断 `PAGE_FREE` 指向的头节点代表的已删除记录占用的存储空间是否足够容纳这条新插入的记录，**如果不可以容纳，就直接向页面中申请新的空间来存储这条记录**（并不会尝试遍历整个垃圾链表，找到一个可以容纳新记录的节点）**。如果可以容纳，那么直接重用这条已删除记录的存储空间，并且把 `PAGE_FREE` 指向垃圾链表中的下一条已删除记录**。

    如果新插入的那条记录占用的存储空间大小小于垃圾链表的头节点占用的存储空间大小，那就意味头节点对应的记录占用的存储空间里有一部分空间用不到，这部分空间就被称之为碎片空间。
    
    这些碎片空间占用的存储空间大小会被统计到 `PAGE_GARBAGE` 属性中，当页面快满时，如果再插入一条记录，此时页面中并不能分配一条完整记录的空间，这时候会判断 `PAGE_GARBAGE` 的空间和剩余可利用的空间加起来是不是可以容纳下这条记录，如果可以的话，InnoDB 会**尝试重新组织页内的记录**。

    重新组织的过程就是先开辟一个临时页面，把页面内的记录依次插入一遍，因为依次插入时并不会产生碎片，之后再把临时页面的内容复制到本页面，从而把碎片空间利用上。

**在删除语句所在的事务提交之前，只会经历 `阶段一`，也就是 `delete mark` 阶段**（提交之后我们就不用回滚了，所以只需考虑对删除操作的 `阶段一` 做的影响进行回滚）。Mysql 设计了一种称之为 `TRX_UNDO_DEL_MARK_REC` 类型的 `undo 日志`，它的完整结构如下图所示：

![](https://r2.129870.xyz/img/202209192308065.png)

上边的这个类型为 `TRX_UNDO_DEL_MARK_REC` 的 `undo 日志` 中的属性，特别注意一下这几点：

- 在对一条记录进行 `delete mark` 操作前，需要把该记录的旧的 `trx_id` 和 `roll_pointer` 隐藏列的值都给记到对应的 `undo 日志` 中来，这样可以通过 `undo 日志` 的 `old roll_pointer` 找到记录在修改之前对应的 `undo` 日志。

	![](https://r2.129870.xyz/img/202209192309912.png)

- 与类型为 `TRX_UNDO_INSERT_REC` 的 `undo 日志` 不同，类型为 `TRX_UNDO_DEL_MARK_REC` 的 `undo` 日志还多了一个 `索引列各列信息` 的内容。

	也就是说如果某个列被包含在某个索引中，那么它的相关信息就应该被记录到这个 `索引列各列信息` 部分，所谓的相关信息包括该列在记录中的位置（用 `pos` 表示），该列占用的存储空间大小（用 `len` 表示），该列实际值（用 `value` 表示）。

	所以 `索引列各列信息` 存储的内容实质上就是 `<pos, len, value>` 的一个列表。这部分信息主要是用在事务提交后，对该 `中间状态记录` 做真正删除的阶段二，也就是 `purge` 阶段中使用的。

当一条数据删除后，这个 `delete mark` 操作对应的 `undo 日志` 的结构就是这样：

![](https://r2.129870.xyz/img/202209192311019.png)

### 4.2.3. UPDATE 操作对应的 undo 日志

在执行 `UPDATE` 语句时，`InnoDB` 对更新主键和不更新主键这两种情况有截然不同的处理方案。

#### 4.2.3.1. 不更新主键

在不更新主键的情况下，又可以细分为被更新的列占用的存储空间不发生变化和发生变化的情况。

##### 4.2.3.1.1. 就地更新（in-place update）

更新记录时，对于被更新的任意列来说，如果占用的存储空间未发生变化，那么就可以进行 `就地更新`。

##### 4.2.3.1.2. 删除后插入

在不更新主键的情况下，如果有任何一个被更新的列存储空间大小不一致，那么就需要先把这条旧的记录从聚簇索引页面中删除掉，然后再根据更新后列的值创建一条新的记录插入到页面中。

这里所说的 `删除` 并不是 `delete mark` 操作，而是真正的删除掉，把这条记录从 `正常记录链表` 中移除并加入到 `垃圾链表` 中，并且修改页面中相应的统计信息（比如 `PAGE_FREE`、`PAGE_GARBAGE` 等信息）。删除操作**由用户线程同步执行，真正删除之后紧接着就要根据各个列更新后的值创建的新记录插入**。

如果新创建的记录占用的存储空间大小不超过旧记录占用的空间，那么可以直接重用被加入到 `垃圾链表` 中的旧记录所占用的存储空间，否则的话需要在页面中新申请一段空间以供新记录使用，如果本页面内已经没有可用的空间的话，那就需要进行页面分裂操作，然后再插入新记录。

针对 `UPDATE` 不更新主键的情况（包括上边所说的就地更新和先删除旧记录再插入新记录），Mysql 设计了一种类型为 `TRX_UNDO_UPD_EXIST_REC` 的 `undo 日志`：

![](https://r2.129870.xyz/img/202209192315254.png)

大部分属性和 `TRX_UNDO_DEL_MARK_REC` 类型的 `undo 日志` 是类似的：

- `n_updated` 属性表示本条 `UPDATE` 语句执行后将有几个列被更新，后边跟着的 `<pos, old_len, old_value>` 分别表示被更新列在记录中的位置、更新前该列占用的存储空间大小、更新前该列的真实值。
- 如果在 `UPDATE` 语句中更新的列包含索引列，那么也会添加 `索引列各列信息` 这个部分，否则的话是不会添加这个部分的。

#### 4.2.3.2. 更新主键

在聚簇索引中记录按照主键值的大小连成了一个单向链表，如果更新某条记录的主键值，意味着这条记录在聚簇索引中的位置将会发生改变。更新操作在聚簇索引中分了两步处理：

1. 将旧记录进行 `delete mark` 操作
    这里是 delete mark 操作！也就是说在 `UPDATE` 语句所在的事务提交前，对**旧记录只做一个 `delete mark` 操作，在事务提交后才由专门的线程做 purge 操作，把它加入到垃圾链表中**。
    
    > [!note] 仅标记删除的原因
    > 为什么更新主键时采取 `delete mark` 操作，而不更新主键采取物理清除的方式呢？**因为在主键未更新的情况下可以继续通过该主键定位到数据（定位到新数据，再通过版本链获取老数据），因此可以删除；而主键更新的情况下需要使用老主键定位，所以需要将其保留。**
2. 根据更新后各列的值创建一条新记录，并将其插入到聚簇索引中
    由于更新后的记录主键值发生了改变，所以需要重新从聚簇索引中定位这条记录所在的位置，然后把它插进去。

针对 `UPDATE` 语句更新记录主键值的这种情况，在对该记录进行 `delete mark` 操作前，会记录一条类型为 `TRX_UNDO_DEL_MARK_REC` 的 `undo 日志`；之后插入新记录时，会记录一条类型为 `TRX_UNDO_INSERT_REC` 的 `undo 日志`，也就是说每对一条记录的主键值做改动时，会记录 2 条 `undo 日志`。

## 4.3. undo 日志存储及使用

### 4.3.1. undo 日志页面

 `undo 日志` 存储在 `FIL_PAGE_UNDO_LOG` 类型的页中：

![](https://r2.129870.xyz/img/202209192325621.png)

`Undo Page Header` 是 `Undo 页面` 所特有的：

![](https://r2.129870.xyz/img/202209192325479.png)

- `TRX_UNDO_PAGE_TYPE` ： `undo 日志` 类型
    -  `TRX_UNDO_INSERT`
        一般由 `INSERT` 语句产生，或者在 `UPDATE` 语句中有更新主键的情况也会产生此类型的 `undo 日志`。
    -  `TRX_UNDO_UPDATE`
        其他类型的 `undo 日志` 都属于这个大类，比如 `TRX_UNDO_DEL_MARK_REC`、`TRX_UNDO_UPD_EXIST_REC`，一般由 `DELETE`、`UPDATE` 语句产生的 `undo 日志` 属于这个大类。

    不同大类的 `undo 日志` 不能混着存储。之所以把 undo 日志分成两个大类，是因为**类型为 TRX_UNDO_INSERT_REC 的 undo 日志在事务提交后可以直接删除掉，而其他类型的 undo 日志还需要进行 MVCC 服务，不能直接删除掉**。

- `TRX_UNDO_PAGE_START` ：表示第一条 `undo 日志` 在本页面中的起始偏移量。
- `TRX_UNDO_PAGE_FREE` ：表示当前页面中存储的最后一条 `undo` 日志结束时的偏移量。
    ![](https://r2.129870.xyz/img/202209192328749.png)
- `TRX_UNDO_PAGE_NODE` ：代表一个 `List Node` 结构，链表的普通节点。

### 4.3.2. Undo 页面链表

#### 4.3.2.1. 单个事务中的 Undo 页面链表

一个事务执行过程中可能产生很多 `undo 日志`，需要放到多个页面中，这些页面通过 `TRX_UNDO_PAGE_NODE` 属性连成了链表：

![](https://r2.129870.xyz/img/202209192329507.png)

链表中的第一个 `Undo 页面` 称它为 `first undo page`，其余的 `Undo 页面` 称之为 `normal undo page`，在 `first undo page` 中除了记录 `Undo Page Header` 之外，还会记录其他的一些管理信息。

在一个事务执行过程中可能混着执行 `INSERT`、`DELETE`、`UPDATE` 语句，会产生不同类型的 `undo 日志`。同一个 `Undo 页面` 要么只存储 `TRX_UNDO_INSERT` 大类的 `undo 日志`，要么只存储 `TRX_UNDO_UPDATE` 大类的 `undo 日志`，所以在**一个事务执行过程中就可能需要 2 个 `Undo 页面` 的链表，一个称之为 `insert undo 链表`，另一个称之为 `update undo 链表`** ：

![](https://r2.129870.xyz/img/202209192330031.png)

另外，对**普通表和临时表的记录改动时产生的 `undo 日志` 要分别记录**，所以在一个事务中最多有 4 个以 `Undo 页面` 为节点组成的链表：

![](https://r2.129870.xyz/img/202209192331535.png)

事务进行过程中按需分配这些链表。

#### 4.3.2.2. 多个事务中的 Undo 页面链表

为了尽可能提高 `undo 日志` 的写入效率，不同事务执行过程中产生的 undo 日志会被写入到不同的 Undo 页面链表中：

![](https://r2.129870.xyz/img/202209192332187.png)

### 4.3.3. undo 日志具体写入过程

#### 4.3.3.1. Undo Log Segment Header

每一个 `Undo 页面` 链表都对应着一个[[Mysql的存储结构#5 2 1 独立表空间|段]]，称之为 `Undo Log Segment`，链表中的页面都是从这个段里边申请的。在 `Undo 页面` 链表的第一个页面 `first undo page` 中有一个 `Undo Log Segment Header` 部分，这个部分中包含了该链表对应的段的 `segment header` 信息以及其他的一些关于这个段的信息：

![](https://r2.129870.xyz/img/202209192334294.png)

 `Undo Log Segment Header` 结构如下：

![](https://r2.129870.xyz/img/202209192335369.png)

其中各个属性的意思如下：

- `TRX_UNDO_STATE` ：本 `Undo 页面` 链表状态
    - `TRX_UNDO_ACTIVE` ：活跃状态，一个活跃的事务正在往这个段里边写入 `undo 日志`。
    - `TRX_UNDO_CACHED` ：被缓存的状态，处在该状态的 `Undo 页面` 链表等待着之后被其他事务重用。
    - `TRX_UNDO_TO_FREE` ：对于 `insert undo` 链表来说，如果在它对应的事务提交之后，该链表不能被重用，那么就会处于这种状态。
    - `TRX_UNDO_TO_PURGE` ：对于 `update undo` 链表来说，如果在它对应的事务提交之后，该链表不能被重用，那么就会处于这种状态。
    - `TRX_UNDO_PREPARED` ：包含处于 `PREPARE` 阶段的事务产生的 `undo 日志`。
        事务的 PREPARE 阶段在分布式事务中才出现的。
- `TRX_UNDO_LAST_LOG` ：本 `Undo 页面` 链表中最后一个 `Undo Log Header` 的位置。
- `TRX_UNDO_FSEG_HEADER` ：本 `Undo 页面` 链表对应的段的 [[Mysql的存储结构#5 2 1 4 段的结构 INODE Entry|Segment Header]] 信息。
- `TRX_UNDO_PAGE_LIST` ：`Undo 页面` 链表的基节点。

#### 4.3.3.2. Undo Log Header

一个事务在向 `Undo 页面` 中写入 `undo 日志` 是连续写入的。写完一个 `Undo 页面` 后，再从段里申请一个新页面，然后把这个页面插入到 `Undo 页面` 链表中，继续往这个新申请的页面中写。

同一个事务向一个 `Undo 页面` 链表中写入的 `undo 日志` 算是一个组，在每写入一组 `undo 日志` 时，都会在这组 `undo 日志` 前先记录关于这个组的一些属性：

![](https://r2.129870.xyz/img/202209192341738.png)

`Undo Log Header` 具体的结构如下：

![](https://r2.129870.xyz/img/202209192341254.png)

- `TRX_UNDO_TRX_ID` ：本组 `undo 日志` 的事务 `id`。
- `TRX_UNDO_TRX_NO` ：事务提交后生成的一个序号，使用此序号来标记事务的提交顺序。
- `TRX_UNDO_DEL_MARKS` ：本组 `undo` 日志中是否包含由于 `Delete mark` 操作产生的 `undo 日志`。
- `TRX_UNDO_LOG_START` ：本组 `undo` 日志中第一条 `undo 日志` 的在页面中的偏移量。
- `TRX_UNDO_XID_EXISTS` ：本组 `undo 日志` 是否包含 XID 信息。
- `TRX_UNDO_DICT_TRANS` ：标记本组 `undo 日志` 是否由 DDL 语句产生。
- `TRX_UNDO_TABLE_ID` ：如果 `TRX_UNDO_DICT_TRANS` 为 true，那么本属性表示 DDL 语句操作的表的 `table id`。
- `TRX_UNDO_NEXT_LOG` ：下一组的 `undo 日志` 在页面中开始的偏移量。
- `TRX_UNDO_PREV_LOG` ：上一组的 `undo 日志` 在页面中开始的偏移量。
    一般来说一个 Undo 页面链表只存储一个事务执行过程中产生的一组 undo 日志，但是在某些情况下，可能会在一个事务提交之后，之后开启的事务重复利用这个 Undo 页面链表，这样就会导致一个 Undo 页面中可能存放多组 Undo 日志。
- `TRX_UNDO_HISTORY_NODE` ：一个 12 字节的 `List Node` 结构，代表一个称之为 `History` 链表的节点。

对于没有被重用的 `Undo 页面` 链表， `first undo page` 在真正写入 `undo 日志` 前，会填充 `Undo Page Header`、`Undo Log Segment Header`、`Undo Log Header` 这 3 个部分，之后才开始正式写入 `undo 日志`。对于其他的页面在真正写入 `undo 日志` 前，只会填充 `Undo Page Header`。链表的 `List Base Node` 存放到 `first undo page` 的 `Undo Log Segment Header` 部分：

![](https://r2.129870.xyz/img/202209192343558.png)

### 4.3.4. 重用 Undo 页面

为了能提高并发执行的多个事务写入 `undo 日志` 的性能，Mysql 为每个事务单独分配相应的 `Undo 页面` 链表。在事务修改数据比较少的情况下页面浪费空间大，于是**在事务提交后在某些情况下可以重用该事务的 `Undo 页面` 链表**：

1. 该链表中只包含一个 `Undo 页面`。
    只有在 `Undo 页面` 链表中只包含一个 `Undo 页面` 时，该链表才可以被下一个事务所重用。不复用更多页面的原因是避免上一个事务的页面无法被充分复用。
2. 该 `Undo 页面` 已经使用的空间小于整个页面空间的 3/4。

`Undo 页面` 链表按照存储的 `undo 日志` 所属的大类可以被分为 `insert undo 链表` 和 `update undo 链表` 两种，这两种链表在被重用时的策略也是不同的：

- `insert undo 链表`
    `insert undo 链表` 在事务提交之后就可以被清除掉。重用这个事务的 `insert undo 链表` 时，可以直接把之前事务写入的一组 `undo 日志` 覆盖掉，从头开始写入新事务的一组 `undo 日志`：

    ![](https://r2.129870.xyz/img/202209192345918.png)

    在重用 Undo 页面链表写入新的一组 undo 日志时，不仅会写入新的 Undo Log Header，还会适当调整 `Undo Page Header`、`Undo Log Segment Header`、`Undo Log Header` 中的一些属性，比如 `TRX_UNDO_PAGE_START`、`TRX_UNDO_PAGE_FREE` 等。

- `update undo 链表`
    在一个事务提交后，它的 `update undo 链表` 中的 `undo 日志` 也不能立即删除掉（这些日志用于 MVCC）。所以如果之后的事务想重用 `update undo 链表` 时，就不能覆盖之前事务写入的 `undo 日志`。这样就相当于在同一个 `Undo 页面` 中写入了多组的 `undo 日志`：

    ![](https://r2.129870.xyz/img/202209192347690.png)

### 4.3.5. 回滚段

#### 4.3.5.1. 回滚段的结构

为了管理 undo 日志的链表，Mysql 设计了一个称之为 `Rollback Segment Header` 的页面，在这个页面中存放了各个 `Undo 页面` 链表的 `frist undo page` 的 `页号`，这些 `页号` 称之为 `undo slot`。

![](https://r2.129870.xyz/img/202209192348444.png)

每一个 `Rollback Segment Header` 页面都对应着一个段，这个 `Rollback Segment` 里其实只有一个页面：

- `TRX_RSEG_MAX_SIZE` ：本 `Rollback Segment` 中管理的所有 `Undo 页面` 链表中的 `Undo 页面` 数量之和的最大值。
    本 `Rollback Segment` 中所有 `Undo 页面` 链表中的 `Undo 页面` 数量之和不能超过 `TRX_RSEG_MAX_SIZE` 代表的值。
- `TRX_RSEG_HISTORY_SIZE` ：`History` 链表占用的页面数量。
- `TRX_RSEG_HISTORY` ：`History` 链表的基节点。
- `TRX_RSEG_FSEG_HEADER` ：本 `Rollback Segment` 对应的 10 字节大小的 `Segment Header` 结构，通过它可以找到本段对应的 `INODE Entry`。
- `TRX_RSEG_UNDO_SLOTS` ：各个 `Undo 页面` 链表的 `first undo page` 的 `页号` 集合，也就是 `undo slot` 集合。
    一个页号占用 `4` 个字节，对于 `16KB` 大小的页面来说，这个 `TRX_RSEG_UNDO_SLOTS` 部分共存储了 `1024` 个 `undo slot`，所以共需 `1024 × 4 = 4096` 个字节。

#### 4.3.5.2. 从回滚段中申请 Undo 页面链表

初始时 `Rollback Segment Header` 的 `undo slot` 都被设置成了一个特殊的值：`FIL_NULL`，表示该 `undo slot` 不指向任何页面。

当有事务需要分配 `Undo 页面` 链表时，就从回滚段的第一个 `undo slot` 开始寻找状态不是 `FIL_NULL` 的页面：

- 如果是 `FIL_NULL`，那么在表空间中新创建一个段（也就是 `Undo Log Segment`），然后从段里申请一个页面作为 `Undo 页面` 链表的 `first undo page`，然后把该 `undo slot` 的值设置为刚刚申请的这个页面的页号，这样也就意味着这个 `undo slot` 被分配给了这个事务。
- 如果不是 `FIL_NULL`，说明该 `undo slot` 已经指向了一个 `undo 链表`。

一个 `Rollback Segment Header` 页面中包含 `1024` 个 `undo slot`，如果这 `1024` 个 `undo slot` 的值都不为 `FIL_NULL`，这就意味着这 `1024` 个 `undo slot` 都已经被分配给了某个事务，此时由于新事务无法再获得新的 `Undo 页面` 链表，就会回滚这个事务并且给用户报错：`Too many active concurrent transactions`。

当一个事务提交时：

- 如果 `undo slot` 指向的 `Undo 页面` 链表**符合被重用的条件**。
    该 `undo slot` 就设置为被缓存的状态，这时该 `Undo 页面` 链表的 `TRX_UNDO_STATE` 属性会被设置为 `TRX_UNDO_CACHED`。

    被缓存的 `undo slot` 都会被加入到一个链表，根据对应的 `Undo 页面` 链表的类型不同，也会被加入到不同的链表：

    - 如果对应的 `Undo 页面` 链表是 `insert undo 链表`，则该 `undo slot` 会被加入 `insert undo cached 链表`。
    - 如果对应的 `Undo 页面` 链表是 `update undo 链表`，则该 `undo slot` 会被加入 `update undo cached 链表`。

    一个回滚段就对应着上述两个 `cached 链表`，如果有**新事务要分配 `undo slot` 时，先从对应的 `cached 链表` 中找。如果没有被缓存的 `undo slot`，才会到回滚段的 `Rollback Segment Header` 页面中再去找**。

- 如果该 `undo slot` 指向的 `Undo 页面` 链表不符合被重用的条件：
    -  `insert undo 链表`：该 `Undo 页面` 链表的 `TRX_UNDO_STATE` 属性会被设置为 `TRX_UNDO_TO_FREE`，之后该 `Undo 页面` 链表对应的段会被释放掉，然后把该 `undo slot` 的值设置为 `FIL_NULL`。
    -  `update undo 链表`：该 `Undo 页面` 链表的 `TRX_UNDO_STATE` 属性会被设置为 `TRX_UNDO_TO_PRUGE`，则会将该 `undo slot` 的值设置为 `FIL_NULL`，然后**将本次事务写入的一组 `undo` 日志放到 `History 链表` 中**。

#### 4.3.5.3. 多个回滚段

一个事务执行过程中最多分配 `4` 个 `Undo 页面` 链表，而一个回滚段里只有 `1024` 个 `undo slot`，很显然 `undo slot` 的数量有点少。所以 Mysql 定义了 `128` 个回滚段，也就相当于有了 `128 × 1024 = 131072` 个 `undo slot`。**假设一个读写事务执行过程中只分配 `1` 个 `Undo 页面` 链表，那么就可以同时支持 `131072` 个读写事务并发执行**。

每个回滚段都对应着一个 `Rollback Segment Header` 页面，有 128 个回滚段，自然就要有 128 个 `Rollback Segment Header` 页面，这些页面的地址存放在 Mysql [[Mysql的存储结构#5 2 2 系统表空间|系统表空间]]的第 `5` 号页面中，该区域包含了 128 个 8 字节大小的格子：

![](https://r2.129870.xyz/img/202209192357369.png)

每个 8 字节的格子的构造就像这样：

![](https://r2.129870.xyz/img/202209192357396.png)

如图所示，每个 8 字节的格子其实由两部分组成：

- 4 字节大小的 `Space ID`，代表一个表空间的 ID。
- 4 字节大小的 `Page number`，代表一个页号。

每个 8 字节大小的指针指向某个表空间中的 `Rollback Segment Header`。要定位一个 `Rollback Segment Header` 还需要知道对应的表空间 ID，这也就意味着不同的回滚段可能分布在不同的表空间中。

![](https://r2.129870.xyz/img/202209192358989.png)

#### 4.3.5.4. 回滚段分类

128 个回滚段可以被分成两大类：

- 普通表回滚段：第 `0` 号、第 `33～127` 号回滚段
    其中**第 `0` 号回滚段必须在系统表空间中**，第 `33～127` 号回滚段既可以在系统表空间中，也可以在自己配置的 `undo` 表空间中。

    如果一个事务在执行过程中由于对普通表的记录做了改动需要分配 `Undo 页面` 链表时，必须从这一类的段中分配相应的 `undo slot`。

- 临时表回滚段：第 `1～32` 号回滚段
    这些回滚段必须在临时表空间（对应着数据目录中的 `ibtmp1` 文件）中。

也就是说如果一个事务在执行过程中既对普通表的记录做了改动，又对临时表的记录做了改动，那么需要为这个记录分配 2 个回滚段，再分别到这两个回滚段中分配对应的 `undo slot`。

针对普通表和临时表划分不同种类的 `回滚段` 的原因：**在修改针对普通表的回滚段中的 undo 页面时，需要记录对应的 redo 日志，而修改针对临时表的回滚段中的 undo 页面时，不需要记录对应的 redo 日志**。

#### 4.3.5.5. 为事务分配 Undo 页面链表详细过程

以事务对普通表的记录做改动为例，事务执行过程中分配 `Undo 页面` 链表时的完整过程如下：

1. 获取回滚段
    事务在执行过程中对普通表的记录首次做改动之前，首先会到系统表空间的第 `5` 号页面中分配一个回滚段（获取一个 `Rollback Segment Header` 页面的地址）。一旦某个回滚段被分配给了这个事务，那么之后该事务中再对普通表的记录做改动时，就不会重复分配了。

    使用  `round-robin` 的方式来分配回滚段，回滚段被轮着分配给不同的事务。
2. 查看是否有可重用页面
    在分配到回滚段后，首先看一下这个回滚段的两个 `cached 链表` 有没有已经缓存了的 `undo slot`，如果事务做的是 `INSERT` 操作，就去回滚段对应的 `insert undo cached 链表` 中看看有没有缓存的 `undo slot`；如果事务做的是 `DELETE` 操作，就去回滚段对应的 `update undo cached 链表` 中看看有没有缓存的 `undo slot`。如果有缓存的 `undo slot`，那么就把这个缓存的 `undo slot` 分配给该事务。
3. 无可重用页面，申请新页面
    如果没有缓存的 `undo slot` 可供分配，那么就要到 `Rollback Segment Header` 页面中找一个可用的 `undo slot` 分配给当前事务。
4. 配置 uodo 日志页
    找到可用的 `undo slot` 后，如果该 `undo slot` 是从 `cached 链表` 中获取的，那么它对应的 `Undo Log Segment` 已经分配了，否则的话需要重新分配一个 `Undo Log Segment`，然后从该 `Undo Log Segment` 中申请一个页面作为 `Undo 页面` 链表的 `first undo page`。
5. 写入 undo 日志
    事务把 `undo 日志` 写入到申请的 `Undo 页面` 链表中。

