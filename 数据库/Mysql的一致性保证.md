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

# 2. uodo 日志

## 2.1. `trx_id` 事务 ID

`InnoDB` 记录行格式中：聚簇索引的记录除了会保存完整的用户数据以外，而且还会自动添加名为 `trx_id`、`roll_pointer` 的隐藏列，如果用户没有在表中定义主键以及 UNIQUE 键，还会自动添加一个名为 `row_id` 的隐藏列。所以一条记录在页面中的真实结构看起来就是这样的：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192249305.png)

`trx_id` 列就是某个对这个聚簇索引记录做改动的语句所在的事务对应的 `事务id` 而已（此处的改动可以是 `INSERT`、`DELETE`、`UPDATE` 操作）。

## 2.2. undo 日志格式

为了实现事务的 `原子性`，`InnoDB` 存储引擎在实际进行增、删、改一条记录时，都需要先把对应的 `undo日志` 记下来，一般每对一条记录做一次改动，就对应着一条 `undo日志`，但在某些更新记录的操作中，也可能会对应着 2 条 `undo日志`。

一个事务在执行过程中可能新增、删除、更新若干条记录，也就是说需要记录很多条对应的 `undo日志`，这些 `undo日志` 会被从 `0` 开始编号，`第0号undo日志`、`第1号undo日志`、...、`第n号undo日志` 等，这个编号也被称之为 `undo no`。

这些 `undo日志` 是被记录到[[Mysql的存储结构#3 5 File Header（文件头部）|类型]]为 `FIL_PAGE_UNDO_LOG`（对应的十六进制是 `0x0002`）的页面中。这些页面可以从系统表空间中分配，也可以从一种专门存放 `undo日志` 的表空间，也就是所谓的 `undo tablespace` 中分配。

### 2.2.1. INSERT 操作的 undo 日志

当我们向表中插入一条记录时这条记录被放到了一个数据页中，如果希望回滚这个插入操作，那么把这条记录删除就好了，也就是说在写对应的 `undo` 日志时，主要是把这条记录的主键信息记上。所以 Mysql 设计了一个类型为 `TRX_UNDO_INSERT_REC` 的 `undo日志`，它的完整结构如下图所示：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192253198.png)

当我们向某个表中插入一条记录时，实际上需要向聚簇索引和所有的二级索引都插入一条记录。不过记录 undo 日志时，我们只需要考虑向聚簇索引插入记录时的情况就好了，因为聚簇索引记录和二级索引记录是一一对应的，我们**在回滚插入操作时，只需要知道这条记录的主键信息，然后根据主键信息做对应的删除操作，做删除操作时就会顺带着把所有二级索引中相应的记录也删除掉**。

#### 2.2.1.1. `roll_pointer` 隐藏列的含义

`roll_pointer` 这个占用 `7` 个字节的字段本质上就是一个指向记录对应的 `undo日志` 的一个指针。比方说我们向表里插入了 2 条记录，每条记录都有与其对应的一条 `undo日志`。记录被存储到了类型为 `FIL_PAGE_INDEX` 的页面中，`undo日志` 被存放到了类型为 `FIL_PAGE_UNDO_LOG` 的页面中。效果如图所示：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192256728.png)

### 2.2.2. DELETE 操作对应的 undo 日志

插入到页面中的记录会根据记录头信息中的 `next_record` 属性组成一个单向链表，我们把这个链表称之为 `正常记录链表`；在数据页结构中，被删除的记录其实也会根据[[Mysql的存储结构#2 1 4 记录头信息|记录头信息]]中的 `next_record` 属性组成一个链表，只不过这个链表中的记录占用的存储空间可以被重新利用，所以也称这个链表为 `垃圾链表`。[[Mysql的存储结构#3 4 Page Header（页面头部）|Page Header]] 部分有一个称之为 `PAGE_FREE` 的属性，它指向由被删除记录组成的垃圾链表中的头节点。

我们假设某个数据页面中的记录分布情况是这样的：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192259999.png)

从图中可以看出，`正常记录链表` 中包含了 3 条正常记录，`垃圾链表` 里包含了 2 条已删除记录，在 `垃圾链表` 中的这些记录占用的存储空间可以被重新利用。页面的 `Page Header` 部分的 `PAGE_FREE` 属性的值代表指向 `垃圾链表` 头节点的指针。假设现在我们准备使用 `DELETE` 语句把 `正常记录链表` 中的最后一条记录给删除掉，其实这个删除的过程需要经历两个阶段：

1. 阶段一：仅仅将记录的 `delete_mask` 标识位设置为 `1`

	其他字段不做修改（其实会修改记录的 `trx_id`、`roll_pointer` 这些隐藏列的值），Mysql 把这个阶段称之为 `delete mark`。此时被删除的记录并没有被加入到 `垃圾链表`，也就是此时记录处于一个 `中间状态`，在删除语句所在的事务提交之前，被删除的记录一直都处于这种所谓的 `中间状态`。

	![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192302780.png)

2. 阶段二：当该删除语句所在的事务提交之后，会有专门的线程后来真正的把记录删除掉。

	所谓真正的删除就是把该记录从 `正常记录链表` 中移除，并且加入到 `垃圾链表` 中，然后还要调整一些页面的其他信息，比如页面中的用户记录数量 `PAGE_N_RECS`、上次插入记录的位置 `PAGE_LAST_INSERT`、垃圾链表头节点的指针 `PAGE_FREE`、页面中可重用的字节数量 `PAGE_GARBAGE`、还有页目录的一些信息等等。设计 `InnoDB` 的大叔把这个阶段称之为 `purge`。

	![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192302031.png)
	
	把 `阶段二` 执行完了，这条记录就算是真正的被删除掉了。这条已删除记录占用的存储空间也可以被重新利用了。

	将被删除记录加入到 `垃圾链表` 时，实际上加入到链表的头节点处，会跟着修改 `PAGE_FREE` 属性的值。页面的 `Page Header` 部分有一个 `PAGE_GARBAGE` 属性，该属性记录着当前页面中可重用存储空间占用的总字节数。每当有已删除记录被加入到垃圾链表后，都会把这个 `PAGE_GARBAGE` 属性的值加上该已删除记录占用的存储空间大小。
	
	`PAGE_FREE` 指向垃圾链表的头节点，之后每当新插入记录时，首先判断 `PAGE_FREE` 指向的头节点代表的已删除记录占用的存储空间是否足够容纳这条新插入的记录，**如果不可以容纳，就直接向页面中申请新的空间来存储这条记录**（并不会尝试遍历整个垃圾链表，找到一个可以容纳新记录的节点）。如果可以容纳，那么直接重用这条已删除记录的存储空间，并且把 `PAGE_FREE` 指向垃圾链表中的下一条已删除记录。
	
	但是这里有一个问题，如果新插入的那条记录占用的存储空间大小小于垃圾链表的头节点占用的存储空间大小，那就意味头节点对应的记录占用的存储空间里有一部分空间用不到，这部分空间就被称之为碎片空间。那这些碎片空间岂不是永远都用不到了么？其实也不是，这些碎片空间占用的存储空间大小会被统计到 `PAGE_GARBAGE` 属性中，这些碎片空间在整个页面快使用完前并不会被重新利用，不过当页面快满时，如果再插入一条记录，此时页面中并不能分配一条完整记录的空间，这时候会首先看一看 `PAGE_GARBAGE` 的空间和剩余可利用的空间加起来是不是可以容纳下这条记录，如果可以的话，InnoDB 会尝试重新组织页内的记录，重新组织的过程就是先开辟一个临时页面，把页面内的记录依次插入一遍，因为依次插入时并不会产生碎片，之后再把临时页面的内容复制到本页面，这样就可以把那些碎片空间都解放出来（很显然重新组织页面内的记录比较耗费性能）。

从上边的描述中我们也可以看出来，**在删除语句所在的事务提交之前，只会经历 `阶段一`，也就是 `delete mark` 阶段**（提交之后我们就不用回滚了，所以只需考虑对删除操作的 `阶段一` 做的影响进行回滚）。Mysql 设计了一种称之为 `TRX_UNDO_DEL_MARK_REC` 类型的 `undo日志`，它的完整结构如下图所示：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192308065.png)

上边的这个类型为 `TRX_UNDO_DEL_MARK_REC` 的 `undo日志` 中的属性，特别注意一下这几点：

- 在对一条记录进行 `delete mark` 操作前，需要把该记录的旧的 `trx_id` 和 `roll_pointer` 隐藏列的值都给记到对应的 `undo日志` 中来，这样可以通过 `undo日志` 的 `old roll_pointer` 找到记录在修改之前对应的 `undo` 日志。

	![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192309912.png)

- 与类型为 `TRX_UNDO_INSERT_REC` 的 `undo日志` 不同，类型为 `TRX_UNDO_DEL_MARK_REC` 的 `undo` 日志还多了一个 `索引列各列信息` 的内容。

	也就是说如果某个列被包含在某个索引中，那么它的相关信息就应该被记录到这个 `索引列各列信息` 部分，所谓的相关信息包括该列在记录中的位置（用 `pos` 表示），该列占用的存储空间大小（用 `len` 表示），该列实际值（用 `value` 表示）。
	
	所以 `索引列各列信息` 存储的内容实质上就是 `<pos, len, value>` 的一个列表。这部分信息主要是用在事务提交后，对该 `中间状态记录` 做真正删除的阶段二，也就是 `purge` 阶段中使用的。

当一条数据删除后，这个 `delete mark` 操作对应的 `undo日志` 的结构就是这样：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192311019.png)

### 2.2.3. UPDATE 操作对应的 undo 日志

在执行 `UPDATE` 语句时，`InnoDB` 对更新主键和不更新主键这两种情况有截然不同的处理方案。

#### 2.2.3.1. 不更新主键

在不更新主键的情况下，又可以细分为被更新的列占用的存储空间不发生变化和发生变化的情况。

##### 2.2.3.1.1. 就地更新（in-place update）

更新记录时，对于被更新的每个列来说，如果更新后的列和更新前的列占用的存储空间都一样大，那么就可以进行 `就地更新`，也就是直接在原记录的基础上修改对应列的值。有任何一个被更新的列更新前比更新后占用的存储空间大，或者更新前比更新后占用的存储空间小都不能进行 `就地更新`。

##### 2.2.3.1.2. 删除后插入

在不更新主键的情况下，如果有任何一个被更新的列更新前和更新后占用的存储空间大小不一致，那么就需要先把这条旧的记录从聚簇索引页面中删除掉，然后再根据更新后列的值创建一条新的记录插入到页面中。

这里所说的 `删除` 并不是 `delete mark` 操作，而是真正的删除掉，也就是把这条记录从 `正常记录链表` 中移除并加入到 `垃圾链表` 中，并且修改页面中相应的统计信息（比如 `PAGE_FREE`、`PAGE_GARBAGE` 等这些信息）。不过这里做真正删除操作的线程并不是在 `DELETE` 语句中做 `purge` 操作时使用的另外专门的线程，而是由用户线程同步执行真正的删除操作，真正删除之后紧接着就要根据各个列更新后的值创建的新记录插入。

这里如果新创建的记录占用的存储空间大小不超过旧记录占用的空间，那么可以直接重用被加入到 `垃圾链表` 中的旧记录所占用的存储空间，否则的话需要在页面中新申请一段空间以供新记录使用，如果本页面内已经没有可用的空间的话，那就需要进行页面分裂操作，然后再插入新记录。

针对 `UPDATE` 不更新主键的情况（包括上边所说的就地更新和先删除旧记录再插入新记录），Mysql 设计了一种类型为 `TRX_UNDO_UPD_EXIST_REC` 的 `undo日志`，它的完整结构如下：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192315254.png)

大部分属性和 `TRX_UNDO_DEL_MARK_REC` 类型的 `undo日志` 是类似的，不过还是要注意这么几点：

- `n_updated` 属性表示本条 `UPDATE` 语句执行后将有几个列被更新，后边跟着的 `<pos, old_len, old_value>` 分别表示被更新列在记录中的位置、更新前该列占用的存储空间大小、更新前该列的真实值。

- 如果在 `UPDATE` 语句中更新的列包含索引列，那么也会添加 `索引列各列信息` 这个部分，否则的话是不会添加这个部分的。

#### 2.2.3.2. 更新主键

在聚簇索引中，记录是按照主键值的大小连成了一个单向链表的，如果我们更新了某条记录的主键值，意味着这条记录在聚簇索引中的位置将会发生改变。新旧两条记录在聚簇索引中就有可能离得非常远，甚至中间隔了好多个页面。针对 `UPDATE` 语句中更新了记录主键值的这种情况，`InnoDB` 在聚簇索引中分了两步处理：

1. 将旧记录进行 `delete mark` 操作

	这里是 delete mark 操作！也就是说在 `UPDATE` 语句所在的事务提交前，对旧记录只做一个 `delete mark` 操作，在事务提交后才由专门的线程做 purge 操作，把它加入到垃圾链表中。这里一定要和我们上边所说的在不更新记录主键值时，先真正删除旧记录，再插入新记录的方式区分开！

	为什么更新主键时采取 `delete mark` 操作，而不更新主键采取物理清除的方式呢？**因为在主键未更新的情况下可以继续通过该主键定位到数据（定位到新数据，再通过版本链获取老数据），因此可以删除；而主键更新的情况下需要使用老主键定位，所以需要将其保留。**

2. 根据更新后各列的值创建一条新记录，并将其插入到聚簇索引中

	由于更新后的记录主键值发生了改变，所以需要重新从聚簇索引中定位这条记录所在的位置，然后把它插进去。

针对 `UPDATE` 语句更新记录主键值的这种情况，在对该记录进行 `delete mark` 操作前，会记录一条类型为 `TRX_UNDO_DEL_MARK_REC` 的 `undo日志`；之后插入新记录时，会记录一条类型为 `TRX_UNDO_INSERT_REC` 的 `undo日志`，也就是说每对一条记录的主键值做改动时，会记录 2 条 `undo日志`。

## 2.3. undo 日志存储及使用

### 2.3.1. undo 日志页面

在介绍[[Mysql的存储结构#5 Mysql 的表空间|表空间]]的时候说过，表空间其实是由许许多多的页面构成的，页面默认大小为 `16KB`。这些页面有不同的类型，比如类型为 `FIL_PAGE_INDEX` 的页面用于存储聚簇索引以及二级索引，类型为 `FIL_PAGE_TYPE_FSP_HDR` 的页面用于存储表空间头部信息的，还有其他各种类型的页面，其中有一种称之为 `FIL_PAGE_UNDO_LOG` 类型的页面是专门用来存储 `undo日志` 的，这种类型的页面的通用结构如下图所示（以默认的 `16KB` 大小为例）：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192325621.png)

`Undo Page Header` 是 `Undo页面` 所特有的，我们来看一下它的结构：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192325479.png)

- `TRX_UNDO_PAGE_TYPE` ：本页面准备存储什么种类的 `undo日志`。

	我们前边介绍了好几种类型的 `undo日志`，它们可以被分为两个大类：

	- `TRX_UNDO_INSERT`（使用十进制 `1` 表示）：类型为 `TRX_UNDO_INSERT_REC` 的 `undo日志` 属于此大类，一般由 `INSERT` 语句产生，或者在 `UPDATE` 语句中有更新主键的情况也会产生此类型的 `undo日志`。

	- `TRX_UNDO_UPDATE`（使用十进制 `2` 表示），除了类型为 `TRX_UNDO_INSERT_REC` 的 `undo日志`，其他类型的 `undo日志` 都属于这个大类，比如 `TRX_UNDO_DEL_MARK_REC`、`TRX_UNDO_UPD_EXIST_REC`，一般由 `DELETE`、`UPDATE` 语句产生的 `undo日志` 属于这个大类。

	这个 `TRX_UNDO_PAGE_TYPE` 属性可选的值就是上边的两个，用来标记本页面用于存储哪个大类的 `undo日志`，不同大类的 `undo日志` 不能混着存储，比如一个 `Undo页面` 的 `TRX_UNDO_PAGE_TYPE` 属性值为 `TRX_UNDO_INSERT`，那么这个页面就只能存储类型为 `TRX_UNDO_INSERT_REC` 的 `undo日志`，其他类型的 `undo日志` 就不能放到这个页面中了。

	之所以把 undo 日志分成两个大类，是因为**类型为 TRX_UNDO_INSERT_REC 的 undo 日志在事务提交后可以直接删除掉，而其他类型的 undo 日志还需要为所谓的 MVCC 服务，不能直接删除掉**，对它们的处理需要区别对待。

- `TRX_UNDO_PAGE_START` ：表示在当前页面中是从什么位置开始存储 `undo日志` 的，或者说表示第一条 `undo日志` 在本页面中的起始偏移量。

- `TRX_UNDO_PAGE_FREE` ：与上边的 `TRX_UNDO_PAGE_START` 对应，表示当前页面中存储的最后一条 `undo` 日志结束时的偏移量，或者说从这个位置开始，可以继续写入新的 `undo日志`。

	![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192328749.png)

- `TRX_UNDO_PAGE_NODE` ：代表一个 `List Node` 结构，链表的普通节点。

### 2.3.2. Undo 页面链表

#### 2.3.2.1. 单个事务中的 Undo 页面链表

因为一个事务可能包含多个语句，而且一个语句可能对若干条记录进行改动，而对每条记录进行改动前，都需要记录 1 条或 2 条的 `undo日志`，所以在一个事务执行过程中可能产生很多 `undo日志`，这些日志可能一个页面放不下，需要放到多个页面中，这些页面就通过我们上边介绍的 `TRX_UNDO_PAGE_NODE` 属性连成了链表：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192329507.png)

链表中的第一个 `Undo页面` 称它为 `first undo page`，其余的 `Undo页面` 称之为 `normal undo page`，这是因为在 `first undo page` 中除了记录 `Undo Page Header` 之外，还会记录其他的一些管理信息。

在一个事务执行过程中，可能混着执行 `INSERT`、`DELETE`、`UPDATE` 语句，也就意味着会产生不同类型的 `undo日志`。但是我们前边又强调过，同一个 `Undo页面` 要么只存储 `TRX_UNDO_INSERT` 大类的 `undo日志`，要么只存储 `TRX_UNDO_UPDATE` 大类的 `undo日志`，不能混着存，所以在一个事务执行过程中就可能需要 2 个 `Undo页面` 的链表，一个称之为 `insert undo链表`，另一个称之为 `update undo链表` ：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192330031.png)

另外，Mysql 对普通表和临时表的记录改动时产生的 `undo日志` 要分别记录，所以在一个事务中最多有 4 个以 `Undo页面` 为节点组成的链表：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192331535.png)

当然，并不是在事务一开始就会为这个事务分配这 4 个链表，而是按需分配，啥时候需要啥时候再分配，不需要就不分配。

#### 2.3.2.2. 多个事务中的 Undo 页面链表

为了尽可能提高 `undo日志` 的写入效率，不同事务执行过程中产生的 undo 日志需要被写入到不同的 Undo 页面链表中，如以下的例子：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192332187.png)

如果有更多的事务，那就意味着可能会产生更多的 `Undo页面` 链表。

### 2.3.3. undo 日志具体写入过程

#### 2.3.3.1. Undo Log Segment Header

Mysql 规定，每一个 `Undo页面` 链表都对应着一个[[Mysql的存储结构#5 2 1 独立表空间|段]]，称之为 `Undo Log Segment`。也就是说链表中的页面都是从这个段里边申请的，所以他们在 `Undo页面` 链表的第一个页面，也就是上边提到的 `first undo page` 中设计了一个称之为 `Undo Log Segment Header` 的部分，这个部分中包含了该链表对应的段的 `segment header` 信息以及其他的一些关于这个段的信息，所以 `Undo` 页面链表的第一个页面其实长这样：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192334294.png)

这个 `Undo` 链表的第一个页面比普通页面多了个 `Undo Log Segment Header`，我们来看一下它的结构：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192335369.png)

其中各个属性的意思如下：

- `TRX_UNDO_STATE` ：本 `Undo页面` 链表处在什么状态。

	一个 `Undo Log Segment` 可能处在的状态包括：

    - `TRX_UNDO_ACTIVE` ：活跃状态，也就是一个活跃的事务正在往这个段里边写入 `undo日志`。

    - `TRX_UNDO_CACHED` ：被缓存的状态。处在该状态的 `Undo页面` 链表等待着之后被其他事务重用。

    - `TRX_UNDO_TO_FREE` ：对于 `insert undo` 链表来说，如果在它对应的事务提交之后，该链表不能被重用，那么就会处于这种状态。

    - `TRX_UNDO_TO_PURGE` ：对于 `update undo` 链表来说，如果在它对应的事务提交之后，该链表不能被重用，那么就会处于这种状态。

    - `TRX_UNDO_PREPARED` ：包含处于 `PREPARE` 阶段的事务产生的 `undo日志`。

		事务的 PREPARE 阶段是在所谓的分布式事务中才出现的。

- `TRX_UNDO_LAST_LOG` ：本 `Undo页面` 链表中最后一个 `Undo Log Header` 的位置。

- `TRX_UNDO_FSEG_HEADER` ：本 `Undo页面` 链表对应的段的 [[Mysql的存储结构#5 2 1 4 段的结构 INODE Entry|Segment Header]] 信息。

- `TRX_UNDO_PAGE_LIST` ：`Undo页面` 链表的基节点。

    `Undo页面` 的 `Undo Page Header` 部分有一个 12 字节大小的 `TRX_UNDO_PAGE_NODE` 属性，这个属性代表一个 `List Node` 结构。每一个 `Undo页面` 都包含 `Undo Page Header` 结构，这些页面就可以通过这个属性连成一个链表。这个 `TRX_UNDO_PAGE_LIST` 属性代表着这个链表的基节点，当然这个基节点只存在于 `Undo页面` 链表的第一个页面，也就是 `first undo page` 中。

#### 2.3.3.2. Undo Log Header

一个事务在向 `Undo页面` 中写入 `undo日志` 时的方式是十分简单暴力的，就是直接往里怼，写完一条紧接着写另一条，各条 `undo日志` 之间是亲密无间的。写完一个 `Undo页面` 后，再从段里申请一个新页面，然后把这个页面插入到 `Undo页面` 链表中，继续往这个新申请的页面中写。

Mysql 认为同一个事务向一个 `Undo页面` 链表中写入的 `undo日志` 算是一个组，比方说 `trx 1` 会分配 3 个 `Undo页面` 链表，也就会写入 3 个组的 `undo日志`；`trx 2` 会分配 2 个 `Undo页面` 链表，也就会写入 2 个组的 `undo日志`。在每写入一组 `undo日志` 时，都会在这组 `undo日志` 前先记录一下关于这个组的一些属性。

Mysql 把存储这些属性的地方称之为 `Undo Log Header`。所以 `Undo页面` 链表的第一个页面在真正写入 `undo日志` 前，其实都会被填充 `Undo Page Header`、`Undo Log Segment Header`、`Undo Log Header` 这 3 个部分，如图所示：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192341738.png)

这个 `Undo Log Header` 具体的结构如下：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192341254.png)

- `TRX_UNDO_TRX_ID` ：生成本组 `undo日志` 的事务 `id`。

- `TRX_UNDO_TRX_NO` ：事务提交后生成的一个需要序号，使用此序号来标记事务的提交顺序（先提交的此序号小，后提交的此序号大）。

- `TRX_UNDO_DEL_MARKS` ：标记本组 `undo` 日志中是否包含由于 `Delete mark` 操作产生的 `undo日志`。

- `TRX_UNDO_LOG_START` ：表示本组 `undo` 日志中第一条 `undo日志` 的在页面中的偏移量。

- `TRX_UNDO_XID_EXISTS` ：本组 `undo日志` 是否包含 XID 信息。

- `TRX_UNDO_DICT_TRANS` ：标记本组 `undo日志` 是不是由 DDL 语句产生的。

- `TRX_UNDO_TABLE_ID` ：如果 `TRX_UNDO_DICT_TRANS` 为真，那么本属性表示 DDL 语句操作的表的 `table id`。

- `TRX_UNDO_NEXT_LOG` ：下一组的 `undo日志` 在页面中开始的偏移量。

- `TRX_UNDO_PREV_LOG` ：上一组的 `undo日志` 在页面中开始的偏移量。

	一般来说一个 Undo 页面链表只存储一个事务执行过程中产生的一组 undo 日志，但是在某些情况下，可能会在一个事务提交之后，之后开启的事务重复利用这个 Undo 页面链表，这样就会导致一个 Undo 页面中可能存放多组 Undo 日志。TRX_UNDO_NEXT_LOG 和 TRX_UNDO_PREV_LOG 就是用来标记下一组和上一组 undo 日志在页面中的偏移量的。。
    
- `TRX_UNDO_HISTORY_NODE` ：一个 12 字节的 `List Node` 结构，代表一个称之为 `History` 链表的节点。

对于没有被重用的 `Undo页面` 链表来说，链表的第一个页面，也就是 `first undo page` 在真正写入 `undo日志` 前，会填充 `Undo Page Header`、`Undo Log Segment Header`、`Undo Log Header` 这 3 个部分，之后才开始正式写入 `undo日志`。对于其他的页面来说，也就是 `normal undo page` 在真正写入 `undo日志` 前，只会填充 `Undo Page Header`。链表的 `List Base Node` 存放到 `first undo page` 的 `Undo Log Segment Header` 部分，`List Node` 信息存放到每一个 `Undo页面` 的 `undo Page Header` 部分，所以画一个 `Undo页面` 链表的示意图就是这样：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192343558.png)

### 2.3.4. 重用 Undo 页面

我们前边说为了能提高并发执行的多个事务写入 `undo日志` 的性能，Mysql 决定为每个事务单独分配相应的 `Undo页面` 链表（最多可能单独分配 4 个链表）。但是这样也造成了一些问题，比如其实大部分事务执行过程中可能只修改了一条或几条记录，针对某个 `Undo页面` 链表只产生了非常少的 `undo日志`，这些 `undo日志` 可能只占用一丢丢存储空间，每开启一个事务就新创建一个 `Undo页面` 链表（虽然这个链表中只有一个页面）来存储这么一丢丢 `undo日志` 岂不是太浪费了么？于是决定在事务提交后在某些情况下重用该事务的 `Undo页面` 链表。一个 `Undo页面` 链表是否可以被重用的条件很简单：

1. 该链表中只包含一个 `Undo页面`。

    如果一个事务执行过程中产生了非常多的 `undo日志`，那么它可能申请非常多的页面加入到 `Undo页面` 链表中。在该事物提交后，如果将整个链表中的页面都重用，那就意味着即使新的事务并没有向该 `Undo页面` 链表中写入很多 `undo日志`，那该链表中也得维护非常多的页面，那些用不到的页面也不能被别的事务所使用，这样就造成了另一种浪费。所以设计 `InnoDB` 的大叔们规定，只有在 `Undo页面` 链表中只包含一个 `Undo页面` 时，该链表才可以被下一个事务所重用。

2. 该 `Undo页面` 已经使用的空间小于整个页面空间的 3/4。

我们前边说过，`Undo页面` 链表按照存储的 `undo日志` 所属的大类可以被分为 `insert undo链表` 和 `update undo链表` 两种，这两种链表在被重用时的策略也是不同的，我们分别看一下：

- `insert undo 链表`

	`insert undo链表` 中只存储类型为 `TRX_UNDO_INSERT_REC` 的 `undo日志`，这种类型的 `undo日志` 在事务提交之后就没用了，就可以被清除掉。所以在某个事务提交后，重用这个事务的 `insert undo链表` 时，可以直接把之前事务写入的一组 `undo日志` 覆盖掉，从头开始写入新事务的一组 `undo日志`，如下图所示：

	![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192345918.png)

	当然，在重用 Undo 页面链表写入新的一组 undo 日志时，不仅会写入新的 Undo Log Header，还会适当调整 `Undo Page Header`、`Undo Log Segment Header`、`Undo Log Header` 中的一些属性，比如 `TRX_UNDO_PAGE_START`、`TRX_UNDO_PAGE_FREE` 等。

- `update undo 链表`

	在一个事务提交后，它的 `update undo链表` 中的 `undo日志` 也不能立即删除掉（这些日志用于 MVCC）。所以如果之后的事务想重用 `update undo链表` 时，就不能覆盖之前事务写入的 `undo日志`。这样就相当于在同一个 `Undo页面` 中写入了多组的 `undo日志`，效果看起来就是这样：

	![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192347690.png)

### 2.3.5. 回滚段

#### 2.3.5.1. 回滚段的结构

我们现在知道一个事务在执行过程中最多可以分配 4 个 `Undo页面` 链表，在同一时刻不同事务拥有的 `Undo页面` 链表是不一样的，所以在同一时刻系统里其实可以有许许多多个 `Undo页面` 链表存在。为了更好的管理这些链表，Mysql 又设计了一个称之为 `Rollback Segment Header` 的页面，在这个页面中存放了各个 `Undo页面` 链表的 `frist undo page` 的 `页号`，他们把这些 `页号` 称之为 `undo slot`。

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192348444.png)

每一个 `Rollback Segment Header` 页面都对应着一个段，这个段就称为 `Rollback Segment`，翻译过来就是 `回滚段`。与我们之前介绍的各种段不同的是，这个 `Rollback Segment` 里其实只有一个页面。我们再来看看这个称之为 `Rollback Segment Header` 的页面的各个部分的含义都是啥意思：

- `TRX_RSEG_MAX_SIZE` ：本 `Rollback Segment` 中管理的所有 `Undo页面` 链表中的 `Undo页面` 数量之和的最大值。

	换句话说，本 `Rollback Segment` 中所有 `Undo页面` 链表中的 `Undo页面` 数量之和不能超过 `TRX_RSEG_MAX_SIZE` 代表的值。该属性的值默认为无限大，也就是我们想写多少 `Undo页面` 都可以。
	
	无限大其实也只是个夸张的说法，4 个字节能表示最大的数也就是 `0xFFFFFFFF`，但是 `0xFFFFFFFF` 这个数有特殊用途，所以实际上 `TRX_RSEG_MAX_SIZE` 的值为 `0xFFFFFFFE`。
    
- `TRX_RSEG_HISTORY_SIZE` ：`History` 链表占用的页面数量。

- `TRX_RSEG_HISTORY` ：`History` 链表的基节点。

- `TRX_RSEG_FSEG_HEADER` ：本 `Rollback Segment` 对应的 10 字节大小的 `Segment Header` 结构，通过它可以找到本段对应的 `INODE Entry`。

- `TRX_RSEG_UNDO_SLOTS` ：各个 `Undo页面` 链表的 `first undo page` 的 `页号` 集合，也就是 `undo slot` 集合。

    一个页号占用 `4` 个字节，对于 `16KB` 大小的页面来说，这个 `TRX_RSEG_UNDO_SLOTS` 部分共存储了 `1024` 个 `undo slot`，所以共需 `1024 × 4 = 4096` 个字节。

#### 2.3.5.2. 从回滚段中申请 Undo 页面链表

初始情况下，由于未向任何事务分配任何 `Undo页面` 链表，所以对于一个 `Rollback Segment Header` 页面来说，它的各个 `undo slot` 都被设置成了一个特殊的值：`FIL_NULL`（对应的十六进制就是 `0xFFFFFFFF`），表示该 `undo slot` 不指向任何页面。

随着时间的流逝，开始有事务需要分配 `Undo页面` 链表了，就从回滚段的第一个 `undo slot` 开始，看看该 `undo slot` 的值是不是 `FIL_NULL` ：

- 如果是 `FIL_NULL`，那么在表空间中新创建一个段（也就是 `Undo Log Segment`），然后从段里申请一个页面作为 `Undo页面` 链表的 `first undo page`，然后把该 `undo slot` 的值设置为刚刚申请的这个页面的页号，这样也就意味着这个 `undo slot` 被分配给了这个事务。

- 如果不是 `FIL_NULL`，说明该 `undo slot` 已经指向了一个 `undo链表`，也就是说这个 `undo slot` 已经被别的事务占用了，那就跳到下一个 `undo slot`，判断该 `undo slot` 的值是不是 `FIL_NULL`，重复上边的步骤。

一个 `Rollback Segment Header` 页面中包含 `1024` 个 `undo slot`，如果这 `1024` 个 `undo slot` 的值都不为 `FIL_NULL`，这就意味着这 `1024` 个 `undo slot` 都已经被分配给了某个事务，此时由于新事务无法再获得新的 `Undo页面` 链表，就会回滚这个事务并且给用户报错：`Too many active concurrent transactions`。

当一个事务提交时，它所占用的 `undo slot` 有两种命运：

- 如果该 `undo slot` 指向的 `Undo页面` 链表符合被重用的条件（就是我们上边说的 `Undo页面` 链表只占用一个页面并且已使用空间小于整个页面的 3/4）。
    
    该 `undo slot` 就设置为被缓存的状态，这时该 `Undo页面` 链表的 `TRX_UNDO_STATE` 属性（该属性在 `first undo page` 的 `Undo Log Segment Header` 部分）会被设置为 `TRX_UNDO_CACHED`。

    被缓存的 `undo slot` 都会被加入到一个链表，根据对应的 `Undo页面` 链表的类型不同，也会被加入到不同的链表：

    - 如果对应的 `Undo页面` 链表是 `insert undo链表`，则该 `undo slot` 会被加入 `insert undo cached链表`。

    - 如果对应的 `Undo页面` 链表是 `update undo链表`，则该 `undo slot` 会被加入 `update undo cached链表`。

    一个回滚段就对应着上述两个 `cached链表`，如果有新事务要分配 `undo slot` 时，先从对应的 `cached链表` 中找。如果没有被缓存的 `undo slot`，才会到回滚段的 `Rollback Segment Header` 页面中再去找。

- 如果该 `undo slot` 指向的 `Undo页面` 链表不符合被重用的条件，那么针对该 `undo slot` 对应的 `Undo页面` 链表类型不同，也会有不同的处理：

    - 如果对应的 `Undo页面` 链表是 `insert undo链表`，则该 `Undo页面` 链表的 `TRX_UNDO_STATE` 属性会被设置为 `TRX_UNDO_TO_FREE`，之后该 `Undo页面` 链表对应的段会被释放掉（也就意味着段中的页面可以被挪作他用），然后把该 `undo slot` 的值设置为 `FIL_NULL`。

    - 如果对应的 `Undo页面` 链表是 `update undo链表`，则该 `Undo页面` 链表的 `TRX_UNDO_STATE` 属性会被设置为 `TRX_UNDO_TO_PRUGE`，则会将该 `undo slot` 的值设置为 `FIL_NULL`，然后将本次事务写入的一组 `undo` 日志放到所谓的 `History链表` 中（需要注意的是，这里并不会将 `Undo页面` 链表对应的段给释放掉，因为这些 `undo` 日志还会被使用到 MVCC 中）。

#### 2.3.5.3. 多个回滚段

我们说一个事务执行过程中最多分配 `4` 个 `Undo页面` 链表，而一个回滚段里只有 `1024` 个 `undo slot`，很显然 `undo slot` 的数量有点少。所以 Mysql 定义了 `128` 个回滚段，也就相当于有了 `128 × 1024 = 131072` 个 `undo slot`。假设一个读写事务执行过程中只分配 `1` 个 `Undo页面` 链表，那么就可以同时支持 `131072` 个读写事务并发执行。

每个回滚段都对应着一个 `Rollback Segment Header` 页面，有 128 个回滚段，自然就要有 128 个 `Rollback Segment Header` 页面，这些页面的地址总得找个地方存一下吧！于是 Mysql [[Mysql的存储结构#5 2 2 系统表空间|系统表空间]]的第 `5` 号页面的某个区域包含了 128 个 8 字节大小的格子：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192357369.png)

每个 8 字节的格子的构造就像这样：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192357396.png)

如图所示，每个 8 字节的格子其实由两部分组成：

-4 字节大小的 `Space ID`，代表一个表空间的 ID。

- 4 字节大小的 `Page number`，代表一个页号。

也就是说每个 8 字节大小的 `格子` 相当于一个指针，指向某个表空间中的某个页面，这些页面就是 `Rollback Segment Header`。这里需要注意的一点事，要定位一个 `Rollback Segment Header` 还需要知道对应的表空间 ID，这也就意味着不同的回滚段可能分布在不同的表空间中。

所以通过上边的叙述我们可以大致清楚，在系统表空间的第 `5` 号页面中存储了 128 个 `Rollback Segment Header` 页面地址，每个 `Rollback Segment Header` 就相当于一个回滚段。在 `Rollback Segment Header` 页面中，又包含 `1024` 个 `undo slot`，每个 `undo slot` 都对应一个 `Undo页面` 链表。我们画个示意图：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202209192358989.png)

#### 2.3.5.4. 回滚段分类

我们把这 128 个回滚段给编一下号，最开始的回滚段称之为 `第0号回滚段`，之后依次递增，最后一个回滚段就称之为 `第127号回滚段`。这 128 个回滚段可以被分成两大类：

- 第 `0` 号、第 `33～127` 号回滚段属于一类。其中第 `0` 号回滚段必须在系统表空间中，第 `33～127` 号回滚段既可以在系统表空间中，也可以在自己配置的 `undo` 表空间中。

    如果一个事务在执行过程中由于对普通表的记录做了改动需要分配 `Undo页面` 链表时，必须从这一类的段中分配相应的 `undo slot`。

- 第 `1～32` 号回滚段属于一类。这些回滚段必须在临时表空间（对应着数据目录中的 `ibtmp1` 文件）中。

    如果一个事务在执行过程中由于对临时表的记录做了改动需要分配 `Undo页面` 链表时，必须从这一类的段中分配相应的 `undo slot`。

也就是说如果一个事务在执行过程中既对普通表的记录做了改动，又对临时表的记录做了改动，那么需要为这个记录分配 2 个回滚段，再分别到这两个回滚段中分配对应的 `undo slot`。

为啥要把针对普通表和临时表来划分不同种类的 `回滚段` 呢？这个还得从 `Undo页面` 本身说起，我们说 `Undo页面` 其实是类型为 `FIL_PAGE_UNDO_LOG` 的页面的简称，说到底它也是一个普通的页面。我们前边说过，在修改页面之前一定要先把对应的 `redo日志` 写上，这样在系统奔溃重启时才能恢复到奔溃前的状态。

我们向 `Undo页面` 写入 `undo日志` 本身也是一个写页面的过程，Mysql 为此还设计了许多种 `redo日志` 的类型，比方说 `MLOG_UNDO_HDR_CREATE`、`MLOG_UNDO_INSERT`、`MLOG_UNDO_INIT` 等，也就是说我们对 `Undo页面` 做的任何改动都会记录相应类型的 `redo日志`。但是对于临时表来说，因为修改临时表而产生的 `undo日志` 只需要在系统运行过程中有效，如果系统奔溃了，那么在重启时也不需要恢复这些 `undo` 日志所在的页面，所以在写针对临时表的 `Undo页面` 时，并不需要记录相应的 `redo日志`。

总结一下针对普通表和临时表划分不同种类的 `回滚段` 的原因：在修改针对普通表的回滚段中的 Undo 页面时，需要记录对应的 redo 日志，而修改针对临时表的回滚段中的 Undo 页面时，不需要记录对应的 redo 日志。

#### 2.3.5.5. 为事务分配 Undo 页面链表详细过程

接下来我们以事务对普通表的记录做改动为例，给大家梳理一下事务执行过程中分配 `Undo页面` 链表时的完整过程：

1. 获取回滚段

	事务在执行过程中对普通表的记录首次做改动之前，首先会到系统表空间的第 `5` 号页面中分配一个回滚段（其实就是获取一个 `Rollback Segment Header` 页面的地址）。一旦某个回滚段被分配给了这个事务，那么之后该事务中再对普通表的记录做改动时，就不会重复分配了。
    
    使用 `round-robin`（循环使用）方式来分配回滚段。比如当前事务分配了第 `0` 号回滚段，那么下一个事务就要分配第 `33` 号回滚段，下下个事务就要分配第 `34` 号回滚段，简单一点的说就是这些回滚段被轮着分配给不同的事务。
    
2. 查看是否有可重用页面

	在分配到回滚段后，首先看一下这个回滚段的两个 `cached链表` 有没有已经缓存了的 `undo slot`，比如如果事务做的是 `INSERT` 操作，就去回滚段对应的 `insert undo cached链表` 中看看有没有缓存的 `undo slot`；如果事务做的是 `DELETE` 操作，就去回滚段对应的 `update undo cached链表` 中看看有没有缓存的 `undo slot`。如果有缓存的 `undo slot`，那么就把这个缓存的 `undo slot` 分配给该事务。

3. 无可重用页面，申请新页面

	如果没有缓存的 `undo slot` 可供分配，那么就要到 `Rollback Segment Header` 页面中找一个可用的 `undo slot` 分配给当前事务。

    从 `Rollback Segment Header` 页面中分配可用的 `undo slot` 的方式我们上边也说过了，就是从第 `0` 个 `undo slot` 开始，如果该 `undo slot` 的值为 `FIL_NULL`，意味着这个 `undo slot` 是空闲的，就把这个 `undo slot` 分配给当前事务，否则查看第 `1` 个 `undo slot` 是否满足条件，依次类推，直到最后一个 `undo slot`。如果这 `1024` 个 `undo slot` 都没有值为 `FIL_NULL` 的情况，就直接报错喽（一般不会出现这种情况）～

4. 配置 uodo 日志页

	找到可用的 `undo slot` 后，如果该 `undo slot` 是从 `cached链表` 中获取的，那么它对应的 `Undo Log Segment` 已经分配了，否则的话需要重新分配一个 `Undo Log Segment`，然后从该 `Undo Log Segment` 中申请一个页面作为 `Undo页面` 链表的 `first undo page`。

5. 写入 undo 日志

	然后事务就可以把 `undo日志` 写入到上边申请的 `Undo页面` 链表了！

