#数据库 #Mysql 

# 1. redo 日志

在 [[Mysql的存储结构#6 Mysql 的缓冲区（Buffer Pool）|Buffer Pool 缓冲区]]中，对数据的[[Mysql的存储结构#6 1 3 flush 链表的管理|修改体现在缓冲区]]，那么如何保证 `持久性` 呢？一个很简单的做法就是在事务提交完成之前把该事务所修改的所有页面都刷新到磁盘，但是这个简单粗暴的做法有些问题：

- 刷新一个完整的数据页太浪费了

    有时候我们仅仅修改了某个页面中的一个字节，但是我们知道在 `InnoDB` 中是以页为单位来进行磁盘 IO 的，也就是说我们在该事务提交时不得不将一个完整的页面从内存中刷新到磁盘，我们又知道一个页面默认是 16KB 大小，只修改一个字节就要刷新 16KB 的数据到磁盘上显然是太浪费了。

- 随机 IO 刷起来比较慢

    一个事务可能包含很多语句，即使是一条语句也可能修改许多页面，倒霉催的是该事务修改的这些页面可能并不相邻，这就意味着在将某个事务修改的 `Buffer Pool` 中的页面刷新到磁盘时，需要进行很多的随机 IO，随机 IO 比顺序 IO 要慢，尤其对于传统的机械硬盘来说。

我们只是想让已经提交了的事务对数据库中数据所做的修改永久生效，即使后来系统崩溃，在重启后也能把这种修改恢复出来。所以我们其实没有必要在每次事务提交时就把该事务在内存中修改过的全部页面刷新到磁盘，只需要把修改了哪些东西记录一下就好，比方说记录下某个事务将系统表空间中的第 100 号页面中偏移量为 1000 处的那个字节的值 `1` 改成 `2`。

这样我们在事务提交时，把上述内容刷新到磁盘中，即使之后系统崩溃了，重启之后只要按照上述内容所记录的步骤重新更新一下数据页，那么该事务对数据库中所做的修改又可以被恢复出来，也就意味着满足 `持久性` 的要求。

因为在系统崩溃重启时需要按照上述内容所记录的步骤重新更新数据页，所以上述内容也被称之为 `重做日志`，英文名为 `redo log`。与在事务提交时将所有修改过的内存中的页面刷新到磁盘中相比，只将该事务执行过程中产生的 `redo` 日志刷新到磁盘的好处如下：

-   `redo` 日志占用的空间非常小

    存储表空间 ID、页号、偏移量以及需要更新的值所需的存储空间是很小的，关于 `redo` 日志的格式我们稍后会详细唠叨，现在只要知道一条 `redo` 日志占用的空间不是很大就好了。

-   `redo` 日志是顺序写入磁盘的

    在执行事务的过程中，每执行一条语句，就可能产生若干条 `redo` 日志，这些日志是按照产生的顺序写入磁盘的，也就是使用顺序 IO。

## 1.1. redo 日志格式

`redo` 日志本质上只是记录了一下事务对数据库做了哪些修改。 Mysql 针对事务对数据库的不同修改场景定义了多种类型的 `redo` 日志，但是绝大部分类型的 `redo` 日志都有下边这种通用的结构：

- `type` ：该条 `redo` 日志的类型。

    在 `MySQL 5.7.21` 这个版本中，设计 `InnoDB` 的大叔一共为 `redo` 日志设计了 53 种不同的类型。

- `space ID` ：表空间 ID。

- `page number` ：页号。

- `data` ：该条 `redo` 日志的具体内容。

### 1.1.1. 简单的 redo 日志类型

某些简单的 redo 日志只需要记录哪个页面哪个偏移量处修改了几个字节的值，具体被修改的内容是什么即可。Mysql 把这种极其简单的 `redo` 日志称之为 `物理日志`，并且根据在页面中写入数据的多少划分了几种不同的 `redo` 日志类型：

- `MLOG_1BYTE`（`type` 字段对应的十进制数字为 `1`）：表示在页面的某个偏移量处写入 1 个字节的 `redo` 日志类型。

- `MLOG_2BYTE`（`type` 字段对应的十进制数字为 `2`）：表示在页面的某个偏移量处写入 2 个字节的 `redo` 日志类型。

- `MLOG_4BYTE`（`type` 字段对应的十进制数字为 `4`）：表示在页面的某个偏移量处写入 4 个字节的 `redo` 日志类型。

- `MLOG_8BYTE`（`type` 字段对应的十进制数字为 `8`）：表示在页面的某个偏移量处写入 8 个字节的 `redo` 日志类型。

- `MLOG_WRITE_STRING`（`type` 字段对应的十进制数字为 `30`）：表示在页面的某个偏移量处写入一串数据。

如自增主键的 `Max Row ID` 属性实际占用 8 个字节的存储空间，所以在修改页面中的该属性时，会记录一条类型为 `MLOG_8BYTE` 的 `redo` 日志，`MLOG_8BYTE` 的 `redo` 日志结构如下所示：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190003003.png)

其余 `MLOG_1BYTE`、`MLOG_2BYTE`、`MLOG_4BYTE` 类型的 `redo` 日志结构和 `MLOG_8BYTE` 的类似，只不过具体数据中包含对应个字节的数据罢了。`MLOG_WRITE_STRING` 类型的 `redo` 日志表示写入一串数据，但是因为不能确定写入的具体数据占用多少字节，所以需要在日志结构中添加一个 `len` 字段：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190003543.png)

PS：只要将 MLOG_WRITE_STRING 类型的 redo 日志的 len 字段填充上 1、2、4、8 这些数字，就可以分别替代 MLOG_1BYTE、MLOG_2BYTE、MLOG_4BYTE、MLOG_8BYTE 这些类型的 redo 日志。

### 1.1.2. 复杂的 redo 日志

有时候执行一条语句会修改非常多的页面，包括系统数据页面和用户数据页面（用户数据指的就是聚簇索引和二级索引对应的 `B+` 树）。以一条 `INSERT` 语句为例，它除了要向 `B+` 树的页面中插入数据，也可能更新系统数据 `Max Row ID` 的值，不过对于我们用户来说，平时更关心的是语句对 `B+` 树所做更新：

- 表中包含多少个索引，一条 `INSERT` 语句就可能更新多少棵 `B+` 树。

- 针对某一棵 `B+` 树来说，既可能更新叶子节点页面，也可能更新内节点页面，也可能创建新的页面（在该记录插入的叶子节点的剩余空间比较少，不足以存放该记录时，会进行页面的分裂，在内节点页面中添加 `目录项记录`）。

一个数据页中除了存储实际的记录之后，还有什么 `File Header`、`Page Header`、`Page Directory` 等等部分，所以每往叶子节点代表的数据页里插入一条记录时，还有其他很多地方会可能跟着更新，如以下信息：

- 可能更新 [[Mysql的存储结构#3 3 Page Directory（页目录）|Page Directory]] 中的槽信息。

- [[Mysql的存储结构#3 4 Page Header（页面头部）|Page Header]] 中的各种页面统计信息，比如 `PAGE_N_DIR_SLOTS` 表示的槽数量可能会更改，`PAGE_HEAP_TOP` 代表的还未使用的空间最小地址可能会更改，`PAGE_N_HEAP` 代表的本页面中的记录数量可能会更改等。

- 数据页里的记录是按照索引列从小到大的顺序组成一个单向链表的，每插入一条记录，还需要更新上一条记录的[[Mysql的存储结构#2 1 4 记录头信息|记录头信息]]中的 `next_record` 属性来维护这个单向链表。

画一个简易的示意图就像是这样：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190008562.png)

针对这种复杂的情况，这时我们如果使用上边介绍的简单的物理 `redo` 日志来记录这些修改时，可以有两种解决方案：

- 方案一：在每个修改的地方都记录一条 `redo` 日志。

    也就是如上图所示，有多少个加粗的块，就写多少条物理 `redo` 日志。这样子可能记录的 `redo` 日志占用的空间都比整个页面占用的空间都多了。

- 方案二：将整个页面的 `第一个被修改的字节` 到 `最后一个修改的字节` 之间所有的数据当成是一条物理 `redo` 日志中的具体数据。

    从图中也可以看出来，`第一个被修改的字节` 到 `最后一个修改的字节` 之间仍然有许多没有修改过的数据，我们把这些没有修改的数据也加入到 `redo` 日志中去会很浪费。

正因为上述两种使用物理 `redo` 日志的方式来记录某个页面中做了哪些修改比较浪费，Mysql 提出了一些新的 `redo` 日志类型，比如：

- `MLOG_REC_INSERT`（对应的十进制数字为 `9`）：表示插入一条使用非紧凑行格式的记录时的 `redo` 日志类型。

- `MLOG_COMP_REC_INSERT`（对应的十进制数字为 `38`）：表示插入一条使用[[Mysql的存储结构#2 1 COMPACT 格式|紧凑行格式]]的记录时的 `redo` 日志类型。

- `MLOG_COMP_PAGE_CREATE`（`type` 字段对应的十进制数字为 `58`）：表示创建一个存储紧凑行格式记录的页面的 `redo` 日志类型。

- `MLOG_COMP_REC_DELETE`（`type` 字段对应的十进制数字为 `42`）：表示删除一条使用紧凑行格式记录的 `redo` 日志类型。

- `MLOG_COMP_LIST_START_DELETE`（`type` 字段对应的十进制数字为 `44`）：表示从某条给定记录开始删除页面中的一系列使用紧凑行格式记录的 `redo` 日志类型。

- `MLOG_COMP_LIST_END_DELETE`（`type` 字段对应的十进制数字为 `43`）：与 `MLOG_COMP_LIST_START_DELETE` 类型的 `redo` 日志呼应，表示删除一系列记录直到 `MLOG_COMP_LIST_END_DELETE` 类型的 `redo` 日志对应的记录为止。

	数据页中的记录是按照索引列大小的顺序组成单向链表的，有时候我们会有删除索引列的值在某个区间范围内的所有记录的需求，这时候如果我们每删除一条记录就写一条 redo 日志的话，效率可能有点低，所以提出 `MLOG_COMP_LIST_START_DELETE` 和 `MLOG_COMP_LIST_END_DELETE` 类型的 redo 日志，可以很大程度上减少 redo 日志的条数。

- `MLOG_ZIP_PAGE_COMPRESS`（`type` 字段对应的十进制数字为 `51`）：表示压缩一个数据页的 `redo` 日志类型。

这些类型的 `redo` 日志既包含 `物理` 层面的意思，也包含 `逻辑` 层面的意思，具体指：

- 物理层面看，这些日志都指明了对哪个表空间的哪个页进行了修改。

- 逻辑层面看，在系统崩溃重启时，并不能直接根据这些日志里的记载，将页面内的某个偏移量处恢复成某个数据，而是需要调用一些事先准备好的函数，执行完这些函数后才可以将页面恢复成系统崩溃前的样子。

我们以类型为 `MLOG_COMP_REC_INSERT` 这个代表插入一条使用紧凑行格式的记录时的 `redo` 日志为例来说明一下我们上边所说的 `物理` 层面和 `逻辑` 层面的含义：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190013101.png)

很显然这个类型为 `MLOG_COMP_REC_INSERT` 的 `redo` 日志并没有记录 `PAGE_N_DIR_SLOTS` 的值修改为了啥，`PAGE_HEAP_TOP` 的值修改为了啥，`PAGE_N_HEAP` 的值修改为了啥等等这些信息，而只是把在本页面中插入一条记录所有必备的要素记了下来，之后系统崩溃重启时，服务器会调用相关向某个页面插入一条记录的那个函数，而 `redo` 日志中的那些数据就可以被当成是调用这个函数所需的参数，在调用完该函数后，页面中的 `PAGE_N_DIR_SLOTS`、`PAGE_HEAP_TOP`、`PAGE_N_HEAP` 等等的值也就都被恢复到系统崩溃前的样子了。这就是所谓的 `逻辑` 日志的意思。

## 1.2. Mini-Transaction

### 1.2.1. redo 日志组

语句在执行过程中可能修改若干个页面。比如我们前边说的一条 `INSERT` 语句可能修改系统表空间页号为 `7` 的页面的 `Max Row ID` 属性（当然也可能更新别的系统页面，只不过我们没有都列举出来而已），还会更新聚簇索引和二级索引对应 `B+` 树中的页面。由于对这些页面的更改都发生在 `Buffer Pool` 中，所以在修改完页面之后，需要记录一下相应的 `redo` 日志。在执行语句的过程中产生的 `redo` 日志被设计 `InnoDB` 的大叔人为的划分成了若干个不可分割的组，比如：

- 更新 `Max Row ID` 属性时产生的 `redo` 日志是不可分割的。

- 向聚簇索引对应 `B+` 树的页面中插入一条记录时产生的 `redo` 日志是不可分割的。

- 向某个二级索引对应 `B+` 树的页面中插入一条记录时产生的 `redo` 日志是不可分割的。

Mysql 规定在执行这些需要保证原子性的操作时必须以 `组` 的形式来记录的 `redo` 日志，在进行系统崩溃重启恢复时，针对某个组中的 `redo` 日志，要么把全部的日志都恢复掉，要么一条也不恢复。怎么做到的呢？这得分情况讨论：

- 组内包含多个 redo 日志

	有的需要保证原子性的操作会生成多条 `redo` 日志，比如向某个索引对应的 `B+` 树中进行一次悲观插入就需要生成许多条 `redo` 日志。
	
	如何把这些 `redo` 日志划分到一个组里边儿呢？Mysql 在某组 `redo` 日志结尾后边加上一条特殊类型的 `redo` 日志，该类型名称为 `MLOG_MULTI_REC_END`，`type` 字段对应的十进制数字为 `31`，该类型的 `redo` 日志结构很简单，只有一个 `type` 字段：
	
	![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190022227.png)
	
	所以某个需要保证原子性的操作产生的一系列 `redo` 日志必须要以一个类型为 `MLOG_MULTI_REC_END` 结尾，就像这样：
	
	![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190022311.png)
	
	这样在系统崩溃重启进行恢复时，只有当解析到类型为 `MLOG_MULTI_REC_END` 的 `redo` 日志，才认为解析到了一组完整的 `redo` 日志，才会进行恢复。否则的话直接放弃前边解析到的 `redo` 日志。

- 组内仅一条日志

	有的需要保证原子性的操作只生成一条 `redo` 日志，比如更新 `Max Row ID` 属性的操作就只会生成一条 `redo` 日志。

	其实在一条日志后边跟一个类型为 `MLOG_MULTI_REC_END` 的 `redo` 日志也是可以的，不过设计 `InnoDB` 的大叔比较勤俭节约，他们不想浪费一个比特位。由于 `redo` 日志的类型就是几十种，是小于 `127` 这个数字的，也就是说我们用 7 个比特位就足以包括所有的 `redo` 日志类型，而 `type` 字段其实是占用 1 个字节的，也就是说我们可以省出来一个比特位用来表示该需要保证原子性的操作只产生单一的一条 `redo` 日志，示意图如下：

	![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190024509.png)

	如果 `type` 字段的第一个比特位为 `1`，代表该需要保证原子性的操作只产生了单一的一条 `redo` 日志，否则表示该需要保证原子性的操作产生了一系列的 `redo` 日志。

### 1.2.2. Mini-Transaction 的概念

设计 `MySQL` 的大叔把对底层页面中的一次原子访问的过程称之为一个 `Mini-Transaction`，简称 `mtr`，比如上边所说的修改一次 `Max Row ID` 的值算是一个 `Mini-Transaction`，向某个索引对应的 `B+` 树中插入一条记录的过程也算是一个 `Mini-Transaction`。通过上边的叙述我们也知道，一个所谓的 `mtr` 可以包含一组 `redo` 日志，在进行崩溃恢复时这一组 `redo` 日志作为一个不可分割的整体。

一个事务可以包含若干条语句，每一条语句其实是由若干个 `mtr` 组成，每一个 `mtr` 又可以包含若干条 `redo` 日志，画个图表示它们的关系就是这样：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190025616.png)

## 1.3. redo 日志写入过程

### 1.3.1. redo 日志 block 块

为了更好的进行系统崩溃恢复，Mysql 把通过 `mtr` 生成的 `redo` 日志都放在了大小为 `512字节` 的 `页` 中。为了和我们前边提到的表空间中的页做区别，我们这里把用来存储 `redo` 日志的页称为 `block`。一个 `redo log block` 的示意图如下：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190026884.png)

真正的 `redo` 日志都是存储到占用 `496` 字节大小的 `log block body` 中，图中的 `log block header` 和 `log block trailer` 存储的是一些管理信息：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190026412.png)

其中 `log block header` 的几个属性的意思分别如下：

- `LOG_BLOCK_HDR_NO`

	每一个 block 都有一个大于 0 的唯一标号，本属性就表示该标号值。

- `LOG_BLOCK_HDR_DATA_LEN`

	表示 block 中已经使用了多少字节，初始值为 `12`（因为 `log block body` 从第 12 个字节处开始）。随着往 block 中写入的 redo 日志越来也多，本属性值也跟着增长。如果 `log block body` 已经被全部写满，那么本属性的值被设置为 `512`。

- `LOG_BLOCK_FIRST_REC_GROUP`

	一条 `redo` 日志也可以称之为一条 `redo` 日志记录（`redo log record`），一个 `mtr` 会生产多条 `redo` 日志记录，这些 `redo` 日志记录被称之为一个 `redo` 日志记录组（`redo log record group`）。`LOG_BLOCK_FIRST_REC_GROUP` 就代表该 block 中第一个 `mtr` 生成的 `redo` 日志记录组的偏移量（其实也就是这个 block 里第一个 `mtr` 生成的第一条 `redo` 日志的偏移量）。

- `LOG_BLOCK_CHECKPOINT_NO`

	表示所谓的 `checkpoint` 的序号。

- `log block trailer` 中属性的意思如下：

	- `LOG_BLOCK_CHECKSUM` ：表示 block 的校验值，用于正确性校验。

### 1.3.2. redo 日志缓冲区

Mysql 为了解决磁盘速度过慢的问题而引入了 `Buffer Pool`。同理，写入 `redo` 日志时也不能直接直接写到磁盘上，实际上在服务器启动时就向操作系统申请了一大片称之为 `redo log buffer` 的连续内存空间，翻译成中文就是 `redo日志缓冲区`，我们也可以简称为 `log buffer`。这片内存空间被划分成若干个连续的 `redo log block`，就像这样：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190028627.png)

向 `log buffer` 中写入 `redo` 日志的过程是顺序的，也就是先往前边的 block 中写，当该 block 的空闲空间用完之后再往下一个 block 中写。为了定位该在 `log buffer` 中在哪个 `block` 的哪个偏移量处写入 `redo` 日志，Mysql 提供了一个称之为 `buf_free` 的全局变量，该变量指明后续写入的 `redo` 日志应该写入到 `log buffer` 中的哪个位置，如图所示：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190029079.png)

不同的事务可能是并发执行的，每当一个 `mtr` 执行完成时，伴随该 `mtr` 生成的一组 `redo` 日志就需要被复制到 `log buffer` 中，也就是说不同事务的 `mtr` 可能是交替写入 `log buffer` 的（为了美观，我们把一个 `mtr` 中产生的所有的 `redo` 日志当作一个整体来画）：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190030328.png)

从示意图中我们可以看出来，不同的 `mtr` 产生的一组 `redo` 日志占用的存储空间可能不一样，有的 `mtr` 产生的 `redo` 日志量很少，，有的 `mtr` 产生的 `redo` 日志量非常大。

## 1.4. redo 日志文件

### 1.4.1. redo 日志缓冲区刷盘时机

#### 1.4.1.1. 刷盘时机

我们前边说 `mtr` 运行过程中产生的一组 `redo` 日志在 `mtr` 结束时会被复制到 `log buffer` 中，可是这些日志总在内存里呆着也不是个办法，在一些情况下它们会被刷新到磁盘里，比如：

- `log buffer` 空间不足时

    `log buffer` 的大小是有限的（通过系统变量 `innodb_log_buffer_size` 指定），如果不停的往这个有限大小的 `log buffer` 里塞入日志，很快它就会被填满。Mysql 认为如果当前写入 `log buffer` 的 `redo` 日志量已经占满了 `log buffer` 总容量的大约一半左右，就需要把这些日志刷新到磁盘上。

- 事务提交时

    我们前边说过之所以使用 `redo` 日志主要是因为它占用的空间少，还是顺序写，在事务提交时可以不把修改过的 `Buffer Pool` 页面刷新到磁盘，但是为了保证持久性，必须要把修改这些页面对应的 `redo` 日志刷新到磁盘。

- 将某个脏页刷新到磁盘前，会保证先将该脏页对应的 redo 日志刷新到磁盘中（再一次强调，redo 日志是顺序刷新的，所以在将某个脏页对应的 redo 日志从 redo log buffer 刷新到磁盘时，也会保证将在其之前产生的 redo 日志也刷新到磁盘）。

- 后台线程不停的刷刷刷

    后台有一个线程，大约每秒都会刷新一次 `log buffer` 中的 `redo` 日志到磁盘。

- 正常关闭服务器时

- 做所谓的 `checkpoint` 时

- 其他的一些情况...

#### 1.4.1.2. 刷盘控制

为了保证事务的 `持久性`，用户线程在事务提交时需要将该事务执行过程中产生的所有 `redo` 日志都刷新到磁盘上。这一条要求太狠了，会很明显的降低数据库性能。如果有的同学对事务的 `持久性` 要求不是那么强烈的话，可以选择修改一个称为 `innodb_flush_log_at_trx_commit` 的系统变量的值，该变量有 3 个可选的值：

- `0` ：当该系统变量值为 0 时，表示在事务提交时不立即向磁盘中同步 `redo` 日志，这个任务是交给后台线程做的。

    这样很明显会加快请求处理速度，但是如果事务提交后服务器挂了，后台线程没有及时将 `redo` 日志刷新到磁盘，那么该事务对页面的修改会丢失。

- `1` ：当该系统变量值为 1 时，表示在事务提交时需要将 `redo` 日志同步到磁盘，可以保证事务的 `持久性`。`1` 也是 `innodb_flush_log_at_trx_commit` 的默认值。

- `2` ：当该系统变量值为 2 时，表示在事务提交时需要将 `redo` 日志写到操作系统的缓冲区中，但并不需要保证将日志真正的刷新到磁盘。

    这种情况下如果数据库挂了，操作系统没挂的话，事务的 `持久性` 还是可以保证的，但是操作系统也挂了的话，那就不能保证 `持久性` 了。

### 1.4.2. redo 日志文件组

`MySQL` 的数据目录（使用 `SHOW VARIABLES LIKE 'datadir'` 查看）下默认有两个名为 `ib_logfile0` 和 `ib_logfile1` 的文件，`log buffer` 中的日志默认情况下就是刷新到这两个磁盘文件中。如果我们对默认的 `redo` 日志文件不满意，可以通过下边几个启动参数来调节：

- `innodb_log_group_home_dir`

    该参数指定了 `redo` 日志文件所在的目录，默认值就是当前的数据目录。

- `innodb_log_file_size`

    该参数指定了每个 `redo` 日志文件的大小，在 `MySQL 5.7.21` 这个版本中的默认值为 `48MB`，

- `innodb_log_files_in_group`

    该参数指定 `redo` 日志文件的个数，默认值为 2，最大值为 100。

从上边的描述中可以看到，磁盘上的 `redo` 日志文件不只一个，而是以一个 `日志文件组` 的形式出现的。这些文件以 `ib_logfile[数字]`（`数字` 可以是 `0`、`1`、`2`...）的形式进行命名。在将 `redo` 日志写入 `日志文件组` 时，是从 `ib_logfile0` 开始写，如果 `ib_logfile0` 写满了，就接着 `ib_logfile1` 写，同理，`ib_logfile1` 写满了就去写 `ib_logfile2`，依此类推。如果写到最后一个文件该咋办？那就重新转到 `ib_logfile0` 继续写，所以整个过程如下图所示：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190045531.png)

### 1.4.3. redo 日志文件格式

`log buffer` 本质上是一片连续的内存空间，被划分成了若干个 `512` 字节大小的 `block`。将 log buffer 中的 redo 日志刷新到磁盘的本质就是把 block 的镜像写入日志文件中，所以 `redo` 日志文件其实也是由若干个 `512` 字节大小的 block 组成。

`redo` 日志文件组中的每个文件大小都一样，格式也一样，都是由两部分组成：

- 前 2048 个字节，也就是前 4 个 block 是用来存储一些管理信息的。

- 从第 2048 字节往后是用来存储 `log buffer` 中的 block 镜像的。

所以我们前边所说的 `循环` 使用 redo 日志文件，其实是从每个日志文件的第 2048 个字节开始算，画个示意图就是这样：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190046839.png)

普通 block 的格式我们在唠叨 `log buffer` 的时候都说过了，就是 `log block header`、`log block body`、`log block trialer` 这三个部分。这里需要介绍一下每个 `redo` 日志文件前 2048 个字节，也就是前 4 个特殊 block 的格式都是干嘛的：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190047929.png)

这 4 个 block 分别是：

- `log file header` ：描述该 `redo` 日志文件的一些整体属性

	![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190047194.png)

	各个属性的具体释义如下：

	<table> <thead> <tr> <th> 属性名 </th> <th> 长度（单位：字节）</th> <th> 描述 </th> </tr> </thead> <tbody> <tr> <td> <code> LOG_HEADER_FORMAT </code> </td> <td> <code> 4 </code> </td> <td> <code> redo </code> 日志的版本，在 <code> MySQL 5.7.21 </code> 中该值永远为 1 </td> </tr> <tr> <td> <code> LOG_HEADER_PAD1 </code> </td> <td> <code> 4 </code> </td> <td> 做字节填充用的，没什么实际意义，忽略～ </td> </tr> <tr> <td> <code> LOG_HEADER_START_LSN </code> </td> <td> <code> 8 </code> </td> <td> 标记本 <code> redo </code> 日志文件开始的 LSN 值，也就是文件偏移量为 2048 字节初对应的 LSN 值）。</td> </tr> <tr> <td> <code> LOG_HEADER_CREATOR </code> </td> <td> <code> 32 </code> </td> <td> 一个字符串，标记本 <code> redo </code> 日志文件的创建者是谁。正常运行时该值为 <code> MySQL </code> 的版本号，比如：<code> "MySQL 5.7.21" </code>，使用 <code> mysqlbackup </code> 命令创建的 <code> redo </code> 日志文件的该值为 <code> "ibbackup" </code> 和创建时间。</td> </tr> <tr> <td> <code> LOG_BLOCK_CHECKSUM </code> </td> <td> <code> 4 </code> </td> <td> 本 block 的校验值，所有 block 都有，我们不关心 </td> </tr> </tbody> </table>

- `checkpoint1` ：记录关于 `checkpoint` 的一些属性。

	![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190048041.png)

	各个属性的具体释义如下：
	
	<table> <thead> <tr> <th> 属性名 </th> <th> 长度（单位：字节）</th> <th> 描述 </th> </tr> </thead> <tbody> <tr> <td> <code> LOG_CHECKPOINT_NO </code> </td> <td> <code> 8 </code> </td> <td> 服务器做 <code> checkpoint </code> 的编号，每做一次 <code> checkpoint </code>，该值就加 1。</td> </tr> <tr> <td> <code> LOG_CHECKPOINT_LSN </code> </td> <td> <code> 8 </code> </td> <td> 服务器做 <code> checkpoint </code> 结束时对应的 <code> LSN </code> 值，系统崩溃恢复时将从该值开始。</td> </tr> <tr> <td> <code> LOG_CHECKPOINT_OFFSET </code> </td> <td> <code> 8 </code> </td> <td> 上个属性中的 <code> LSN </code> 值在 <code> redo </code> 日志文件组中的偏移量 </td> </tr> <tr> <td> <code> LOG_CHECKPOINT_LOG_BUF_SIZE </code> </td> <td> <code> 8 </code> </td> <td> 服务器在做 <code> checkpoint </code> 操作时对应的 <code> log buffer </code> 的大小 </td> </tr> <tr> <td> <code> LOG_BLOCK_CHECKSUM </code> </td> <td> <code> 4 </code> </td> <td> 本 block 的校验值，所有 block 都有，我们不关心 </td> </tr> </tbody> </table>

- 第三个 block 未使用，忽略～

- `checkpoint2` ：结构和 `checkpoint1` 一样。

## 1.5. Log Sequence Number

Mysql 为记录已经写入的 `redo` 日志量，设计了一个称之为 `Log Sequence Number` 的全局变量，翻译过来就是：`日志序列号`，简称 `lsn`。`lsn` 初始值为 `8704`（也就是一条 `redo` 日志也没写入时，`lsn` 的值为 `8704`）。

我们知道在向 `log buffer` 中写入 `redo` 日志时不是一条一条写入的，而是以一个 `mtr` 生成的一组 `redo` 日志为单位进行写入的。而且实际上是把日志内容写在了 `log block body` 处。但是在统计 `lsn` 的增长量时，是按照实际写入的日志量加上占用的 `log block header` 和 `log block trailer` 来计算的。

因为 `lsn` 的值是代表系统写入的 `redo` 日志量的一个总和，一个 `mtr` 中产生多少日志，`lsn` 的值就增加多少（当然有时候要加上 `log block header` 和 `log block trailer` 的大小），这样 `mtr` 产生的日志写到磁盘中时，很容易计算某一个 `lsn` 值在 `redo` 日志文件组中的偏移量。

### 1.5.1. flushed_to_disk_lsn

`redo` 日志是首先写到 `log buffer` 中，之后才会被刷新到磁盘上的 `redo` 日志文件。所以 Mysql 提出了一个称之为 `buf_next_to_write` 的全局变量，标记当前 `log buffer` 中已经有哪些日志被刷新到磁盘中了：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190052707.png)

我们前边说 `lsn` 是表示当前系统中写入的 `redo` 日志量，这包括了写到 `log buffer` 而没有刷新到磁盘的日志。相应的，Mysql 提出了一个表示刷新到磁盘中的 `redo` 日志量的全局变量，称之为 `flushed_to_disk_lsn`。系统第一次启动时，该变量的值和初始的 `lsn` 值是相同的，都是 `8704`。随着系统的运行，`redo` 日志被不断写入 `log buffer`，但是并不会立即刷新到磁盘，`lsn` 的值就和 `flushed_to_disk_lsn` 的值拉开了差距。

当有新的 `redo` 日志写入到 `log buffer` 时，首先 `lsn` 的值会增长，但 `flushed_to_disk_lsn` 不变，随后随着不断有 `log buffer` 中的日志被刷新到磁盘上，`flushed_to_disk_lsn` 的值也跟着增长。如果两者的值相同时，说明 log buffer 中的所有 redo 日志都已经刷新到磁盘中了。

### 1.5.2. flush 链表中的 LSN

我们知道一个 `mtr` 代表一次对底层页面的原子访问，在访问过程中可能会产生一组不可分割的 `redo` 日志，在 `mtr` 结束时，会把这一组 `redo` 日志写入到 `log buffer` 中。除此之外，在 `mtr` 结束时还有一件非常重要的事情要做，就是把在 mtr 执行过程中可能修改过的页面加入到 Buffer Pool 的 [[Mysql的存储结构#6 1 3 flush 链表的管理|flush 链表]]。

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190057119.png)

当第一次修改某个缓存在 `Buffer Pool` 中的页面时，就会把这个页面对应的控制块插入到 `flush链表` 的头部，之后再修改该页面时由于它已经在 `flush` 链表中了，就不再次插入了。也就是说 flush 链表中的脏页是按照页面的第一次修改时间从大到小进行排序的。在这个过程中会在缓存页对应的控制块中记录两个关于页面何时修改的属性：

- `oldest_modification` ：如果某个页面被加载到 `Buffer Pool` 后进行第一次修改，那么就将修改该页面的 `mtr` 开始时对应的 `lsn` 值写入这个属性。

- `newest_modification` ：每修改一次页面，都会将修改该页面的 `mtr` 结束时对应的 `lsn` 值写入这个属性。也就是说该属性表示页面最近一次修改后对应的系统 `lsn` 值。

## 1.6. checkpoint

有一个很不幸的事实就是我们的 `redo` 日志文件组容量是有限的，我们不得不选择循环使用 `redo` 日志文件组中的文件，但是这会造成最后写的 `redo` 日志与最开始写的 `redo` 日志 `追尾`。
redo 日志只是为了系统崩溃后恢复脏页用的，如果对应的脏页已经刷新到了磁盘，也就是说即使现在系统崩溃，那么在重启后也用不着使用 redo 日志恢复该页面了，所以该 redo 日志也就没有存在的必要了，那么它占用的磁盘空间就可以被后续的 redo 日志所重用。也就是说：**判断某些 redo 日志占用的磁盘空间是否可以覆盖的依据就是它对应的脏页是否已经刷新到磁盘里**。

Mysql 提出了一个全局变量 `checkpoint_lsn` 来代表当前系统中可以被覆盖的 `redo` 日志总量是多少，这个变量初始值也是 `8704`。比方说现在 `页a` 被刷新到了磁盘，`mtr_1` 生成的 `redo` 日志就可以被覆盖了，所以我们可以进行一个增加 `checkpoint_lsn` 的操作，我们把这个过程称之为做一次 `checkpoint`。做一次 `checkpoint` 其实可以分为两个步骤：

1. 计算一下当前系统中可以被覆盖的 `redo` 日志对应的 `lsn` 值最大是多少。

    `redo` 日志可以被覆盖，意味着它对应的脏页被刷到了磁盘，只要我们计算出当前系统中被最早修改的脏页对应的 `oldest_modification` 值，那凡是在系统 lsn 值小于该节点的 oldest_modification 值时产生的 redo 日志都是可以被覆盖掉的，我们就把该脏页的 `oldest_modification` 赋值给 `checkpoint_lsn`。

2. 将 `checkpoint_lsn` 和对应的 `redo` 日志文件组偏移量以及此次 `checkpint` 的编号写到日志文件的管理信息（就是 `checkpoint1` 或者 `checkpoint2`）中。
    
    Mysql 维护了一个目前系统做了多少次 `checkpoint` 的变量 `checkpoint_no`，每做一次 `checkpoint`，该变量的值就加 1。我们前边说过计算一个 `lsn` 值对应的 `redo` 日志文件组偏移量是很容易的，所以可以计算得到该 `checkpoint_lsn` 在 `redo` 日志文件组中对应的偏移量 `checkpoint_offset`，然后把这三个值都写到 `redo` 日志文件组的管理信息中。
    
    每一个 `redo` 日志文件都有 `2048` 个字节的管理信息，但是上述关于 checkpoint 的信息只会被写到日志文件组的第一个日志文件的管理信息中。不过我们是存储到 `checkpoint1` 中还是 `checkpoint2` 中呢？当 `checkpoint_no` 的值是偶数时，就写到 `checkpoint1` 中，是奇数时，就写到 `checkpoint2` 中。

记录完 `checkpoint` 的信息之后，`redo` 日志文件组中各个 `lsn` 值的关系就像这样：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190101664.png)

### 1.6.1. 批量从 flush 链表中刷出脏页

我们在介绍 `Buffer Pool` 的时候说过，一般情况下都是后台的线程在对 `LRU链表` 和 `flush链表` 进行[[Mysql的存储结构#6 1 5 刷新脏页到磁盘|刷脏操作]]，这主要因为刷脏操作比较慢，不想影响用户线程处理请求。但是如果当前系统修改页面的操作十分频繁，这样就导致写日志操作十分频繁，系统 `lsn` 值增长过快。如果后台的刷脏操作不能将脏页刷出，那么系统无法及时做 `checkpoint`，可能就需要用户线程同步的从 `flush链表` 中把那些最早修改的脏页（`oldest_modification` 最小的脏页）刷新到磁盘，这样这些脏页对应的 `redo` 日志就没用了，然后就可以去做 `checkpoint` 了。

### 1.6.2. 查看系统中的各种 LSN 值

我们可以使用 `SHOW ENGINE INNODB STATUS` 命令查看当前 `InnoDB` 存储引擎中的各种 `LSN` 值的情况：

- `Log sequence number` ：代表系统中的 `lsn` 值，也就是当前系统已经写入的 `redo` 日志量，包括写入 `log buffer` 中的日志。

- `Log flushed up to` ：代表 `flushed_to_disk_lsn` 的值，也就是当前系统已经写入磁盘的 `redo` 日志量。

- `Pages flushed up to` ：代表 `flush链表` 中被最早修改的那个页面对应的 `oldest_modification` 属性值。

- `Last checkpoint at` ：当前系统的 `checkpoint_lsn` 值。

## 1.7. 崩溃恢复

### 1.7.1. 确定恢复的起点

我们前边说过，`checkpoint_lsn` 之前的 `redo` 日志都可以被覆盖，也就是说这些 `redo` 日志对应的脏页都已经被刷新到磁盘中了，既然它们已经被刷盘，我们就没必要恢复它们了。对于 `checkpoint_lsn` 之后的 `redo` 日志，它们对应的脏页可能没被刷盘，也可能被刷盘了，我们不能确定，所以需要从 `checkpoint_lsn` 开始读取 `redo` 日志来恢复页面。

当然，`redo` 日志文件组的第一个文件的管理信息中有两个 block 都存储了 `checkpoint_lsn` 的信息，我们当然是要选取最近发生的那次 checkpoint 的信息。衡量 `checkpoint` 发生时间早晚的信息就是所谓的 `checkpoint_no`，我们只要把 `checkpoint1` 和 `checkpoint2` 这两个 block 中的 `checkpoint_no` 值读出来比一下大小，哪个的 `checkpoint_no` 值更大，说明哪个 block 存储的就是最近的一次 `checkpoint` 信息。这样我们就能拿到最近发生的 `checkpoint` 对应的 `checkpoint_lsn` 值以及它在 `redo` 日志文件组中的偏移量 `checkpoint_offset`。

### 1.7.2. 确定恢复的终点

`redo` 日志恢复的起点确定了，那终点是哪个呢？这个还得从 [[Mysql的一致性保证#redo 日志 block 块|block 的结构]]说起。我们说在写 `redo` 日志的时候都是顺序写的，写满了一个 block 之后会再往下一个 block 中写：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190106291.png)

普通 block 的 `log block header` 部分有一个称之为 `LOG_BLOCK_HDR_DATA_LEN` 的属性，该属性值记录了当前 block 里使用了多少字节的空间。对于被填满的 block 来说，该值永远为 `512`。如果该属性的值不为 `512`，那么就是它了，它就是此次崩溃恢复中需要扫描的最后一个 block。

### 1.7.3. 恢复过程

确定了需要扫描哪些 `redo` 日志进行崩溃恢复之后，接下来就是怎么进行恢复了。假设现在的 `redo` 日志文件中有 5 条 `redo` 日志，如图：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190107937.png)

由于 `redo 0` 在 `checkpoint_lsn` 后边，恢复时可以不管它。我们现在可以按照 `redo` 日志的顺序依次扫描 `checkpoint_lsn` 之后的各条 redo 日志，按照日志中记载的内容将对应的页面恢复出来。这样没什么问题，不过 Mysql 还是想了一些办法加快这个恢复的过程：

- 使用哈希表

    根据 `redo` 日志的 `space ID` 和 `page number` 属性计算出散列值，把 `space ID` 和 `page number` 相同的 `redo` 日志放到哈希表的同一个槽里，如果有多个 `space ID` 和 `page number` 都相同的 `redo` 日志，那么它们之间使用链表连接起来，按照生成的先后顺序链接起来的，如图所示：

	![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209190107318.png)

	之后就可以遍历哈希表，因为对同一个页面进行修改的 `redo` 日志都放在了一个槽里，所以可以一次性将一个页面修复好（避免了很多读取页面的随机 IO），这样可以加快恢复速度。
	
	另外需要注意一点的是，同一个页面的 `redo` 日志是按照生成时间顺序进行排序的，所以恢复的时候也是按照这个顺序进行恢复，如果不按照生成时间顺序进行排序的话，那么可能出现错误。

- 跳过已经刷新到磁盘的页面
    
    我们前边说过，`checkpoint_lsn` 之前的 `redo` 日志对应的脏页确定都已经刷到磁盘了，但是 `checkpoint_lsn` 之后的 `redo` 日志我们不能确定是否已经刷到磁盘，主要是因为在最近做的一次 `checkpoint` 后，可能后台线程又不断的从 `LRU链表` 和 `flush链表` 中将一些脏页刷出 `Buffer Pool`。这些在 `checkpoint_lsn` 之后的 `redo` 日志，如果它们对应的脏页在崩溃发生时已经刷新到磁盘，那在恢复时也就没有必要根据 `redo` 日志的内容修改该页面了。
    
    那在恢复时怎么知道某个 `redo` 日志对应的脏页是否在崩溃发生时已经刷新到磁盘了呢？这还得从页面的结构说起，我们前边说过每个页面都有一个称之为 [[Mysql的存储结构#3 5 File Header（文件头部）|File Header]] 的部分，在 `File Header` 里有一个称之为 `FIL_PAGE_LSN` 的属性，该属性记载了最近一次修改页面时对应的 `lsn` 值（其实就是页面控制块中的 `newest_modification` 值）。如果在做了某次 `checkpoint` 之后有脏页被刷新到磁盘中，那么该页对应的 `FIL_PAGE_LSN` 代表的 `lsn` 值肯定大于 `checkpoint_lsn` 的值，凡是符合这种情况的页面就不需要重复执行 lsn 值小于 `FIL_PAGE_LSN` 的 redo 日志了，所以更进一步提升了崩溃恢复的速度。

