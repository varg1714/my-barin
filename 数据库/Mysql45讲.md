#数据库 #Mysql

# 1. Mysql45 讲

## 1.1. Sql 语句在 Mysql 的执行过程

 ![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20220508232841.png)

- 缓存在 8.0 中已经移除了，不建议使用缓存。Mysql 维护缓存有非常大的性能开销。

- 权限的判断是在执行器的时候进行的。因为部分操作 Mysql 无法在前面的步骤进行，比如触发器只能到执行阶段才能确认。

- 在分析器就完成了 SQL 语句的检查。分析器处理语法和解析查询，生成一颗解析树。进而检查解析树是否合法，列名是否有歧义等。通过则生成新的解析树再交给优化器。

## 1.2. 一条更新语句在数据库的执行过程

### 1.2.1. SQL 执行过程

图中浅色框表示是在 InnoDB 内部执行的，深色框表示是在执行器中执行的：
![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20220508234920.png)

#### 1.2.1.1. Binlog 日志

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20220513230808.png)

系统给 binlog cache 分配了一片内存，每个线程一个，但是共用同一份 binlog 文件。参数 `binlog_cache_size` 控制单个线程内 binlog cache 所占内存的大小。若超过该参数值，就要暂存到磁盘的临时文件 (不是最终的 binlog 文件)。事务提交时，执行器把 binlog cache 里的完整事务写入 binlog，并清空 binlog cache。所以如果经常有大事物的化可以考虑增大这个值来减少刷盘的次数。

binlog 写盘状态 ：

- 内存中

- write

	把日志写入到文件系统的 page cache，并没有把数据持久化到磁盘，所以速度较快
	
- fsync

    将数据持久化到磁盘。一般认为 fsync 才占磁盘的 IOPS

write 和 fsync 的时机，由参数 `sync_binlog` 控制：

-   sync_binlog=0，每次提交事务都只 write，不 fsync

-   sync_binlog=1，每次提交事务都会执行 fsync

-   sync_binlog=N (N>1)，每次提交事务都 write，但累积 N 个事务后才 fsync

因此，在出现 I/O 瓶颈的场景，将 `sync_binlog` 设置成一个较大值，可提升性能。在实际的业务场景中，考虑到丢失日志量的可控性，一般不建议将这个参数设成 0，推荐将其设置为 100~1000 中的某个数值。

但将 `sync_binlog` 设置为 N，对应的风险是：若主机发生异常重启，会丢失最近 N 个事务的 binlog 日志。

binlog 日志的格式：

- statement 格式

	语句级的格式，使用的是原始 SQL 存储。这就会存在问题，binlog 日志是按提交顺序存储的，如果提交顺序与执行顺序有差别，就会导致数据更新出问题。或者主备之间索引选择不一致，同样会导致数据更新出错。
	
- row 格式

	基于原始数据存储，数据是什么样子就存储成什么样子。可以通过命令 `show binlog events;` 查看当前 binlog 日志文件：
	
	![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20220514144956.png)

	binlog 日志文件在 row 格式下如上所示，如果需要查看更为详细的信息通过解析 binlog 日志查看，可以执行以下命令：`/var/lib/mysql# mysqlbinlog -vv binlog.000001 --start-position=3103917;`

	![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20220514145948.png)

	binlog_row_image 的默认配置是 FULL，因此 Delete_event 里面，包含了删掉的行的所有字段的值。如果把 binlog_row_image 设置为 MINIMAL，则只会记录必要的信息。

- mixed 格式

	- 因为有些 statement 格式的 binlog 可能会导致主备不一致，所以要使用 row 格式。
	
	- 但 row 格式的缺点是，很占空间。比如你用一个 delete 语句删掉 10 万行数据，用 statement 的话就是一个 SQL 语句被记录到 binlog 中，占用几十个字节的空间。但如果用 row 格式的 binlog，就要把这 10 万条记录都写到 binlog 中。这样做，不仅会占用更大的空间，同时写 binlog 也要耗费 IO 资源，影响执行速度。
	
	- 所以，MySQL 就取了个折中方案，也就是有了 mixed 格式的 binlog。Mixed 格式的意思是，MySQL 自己会判断这条 SQL 语句是否可能引起主备不一致，如果有可能，就用 row 格式，否则就用 statement 格式。也就是说，mixed 格式可以利用 statment 格式的优点，同时又避免了数据不一致的风险。

#### 1.2.1.2. Redolog 日志

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20220513230832.png)

- 作用

    防止在发生故障的时间点，尚有[脏页](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E8%2584%258F%25E9%25A1%25B5%2F5437502) 未写入磁盘（事务提交之后，数据写入磁盘之前宕机），在重启 mysql 服务的时候，根据 redolog 进行重做，从而达到事务的持久性这一特性。
	
- 内容：  

    物理格式的日志，记录的是物理数据页面的修改信息，其 redolog 是顺序写入 redolog file 的物理文件中去的。
	
- 生成时机：  

    事务开始之后就产生 redolog，redolog 的落盘并不是随着事务的提交才写入的，而是在事务的执行开始，便开始写入 redolog 文件中。
	
- 释放时机：  

    当对应事务的脏页写入到磁盘之后，redolog 的使命也就完成了，重做日志占用的空间就可以重用（被覆盖）。

Innodb 通过 redo Log 和 undo Log 可以保证以上两点。为了保证严格的 CrashSafe，必须要在每个事务提交的时候，将 redo log 写入硬件存储。这样做会牺牲一些性能，但是可靠性最好。为了平衡两者，InnoDB 提供了一个 `innodb_flush_log_at_trx_commit` 系统变量，用户可以根据应用的需求自行调整：

可配置刷入的时机：

- 0：每 N 秒将 redo log buffer 的记录写入 redo log 文件，并且将文件刷入硬件存储 1 次。N 由 `innodb_flush_log_at_timeout` 控制。

- 1：每个事务提交时，将记录从 redo log buffer 写入 redo log 文件，并且将文件刷入磁盘。

- 2：每个事务提交时，仅将记录从 redo log buffer 写入 redo log 文件。redo log 何时刷入硬件存储由操作系统和 `innodb_flush_log_at_timeout` 决定。这个选项可以保证在 MySQL 宕机，而操作系统正常工作时，数据的完整性。

Mysql 后台同步的时机：

重做日志有一个缓存区 Innodb_log_buffer，Innodb_log_buffer 的默认大小为 16Mb，Innodb 存储引擎先将重做日志写入 Innodb_log_buffer 中，然后会通过以下三种方式 Innodb 日志缓冲区的日志刷新到磁盘：

1. Master Thread 每秒一次执行刷新 Innodb_log_buffer 到重做日志文件。

2. 并行的事务提交时，顺带将该事务的 redo log buffer 持久化到磁盘。

	假设一个事务 A 执行到一半，已经写了一些 redo log 到 buffer，这时另外一个线程的事务 B 提交，若 `innodb_flush_log_at_trx_commit` 是 1，则事务 B 要把 redo log buffer 里的日志全部持久化到磁盘。这时，就会带上事务 A 在 redo log buffer 里的日志一起持久化到磁盘。
	
3. 当重做日志缓存可用空间少于 `innodb_log_buffer_size` 的一半时，重做日志缓存被刷新到重做日志文件。

	redolog buffer 占用的空间即将达到 innodb_log_buffer_size 的一半，后台线程会主动写盘。由于这个事务并未提交，所以这个写盘动作只是 write，没有调用 fsync，即只留在文件系统的 page cache。

通过 redo 日志将所有已经在存储引擎内部提交的事务应用 redo log 恢复，所有已经 prepare 但是没有 commit 的 transactions 将会应用 undo log 做 rollback。然后客户端连接时就能看到已经提交的数据存在数据库内，未提交被回滚地数据需要重新执行。

**因此重做日志的写盘，并不一定是随着事务的提交才写入重做日志文件的，而是随着事务的开始，逐步开始的。**

两阶段提交的过程，时序上 redo log 先 prepare，再写 binlog，最后再把 redo log commit。

若把 `innodb_flush_log_at_trx_commit` 置 1，则 redolog 在 prepare 阶段就要持久化一次，因为有一个崩溃恢复逻辑是要依赖于 prepare 的 redo log，再加上 binlog 来恢复的。

每 s 一次的后台轮询刷盘，再加上崩溃恢复，InnoDB 就认为 redo log 在 commit 时无需 fsync，只 write 到文件系统的 page cache 就够了。

通常我们说 MySQL 的“双 1”配置，指的就是 `sync_binlog`、`innodb_flush_log_at_trx_commit` 都是 1。即一个事务完整提交前，需要等待两次刷盘：

- redolog（prepare 阶段）

- binlog

#### 1.2.1.3. Binlog 与 redolog 产生与持久化时机

在事务执行的时候，就已经产生了 redolog 与 binlog，只不过这个时候的文件是在内存中，而不是持久化到磁盘中。
日志文件何时持久化到磁盘中呢，有以下几种情况：

- 事务提交

	当事务提交的时候，日志文件会被持久化到磁盘中，即上述的 prepare 阶段开始。当事务提交的时候，redolog 首先写入磁盘，状态为 prepare。接着将 binlog 写入磁盘。最后将 redolog 状态改为 commit。
	
- 被动写入

	当日志文件是现在内存中创建的，当内存空间不足时，也可能被先写入磁盘。

### 1.2.2. Redolog 日志记录

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20220508235244.png)

1. Redo log 是 InnoDB 引擎特有的；binlog 是 MySQL 的 Server 层实现的，所有引擎都可以使用。

2. Redo log 是物理日志，记录的是“在某个数据页上做了什么修改”；binlog 是逻辑日志，记录的是这个语句的原始逻辑，比如“给 ID=2 这一行的 c 字段加 1 ”。

3. Redo log 是循环写的，空间固定会用完; binlog 是可以追加写入的。“追加写”是指 binlog 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。

### 1.2.3. Redolog 和 binlog 两次提交的原因

这种做法叫做二阶段提交，为了保持数据的一致性而产生的。第一阶段提交事务请求，协调者向参与者发送事务内容，参与者执行并返回结果。第二阶段决定是否提交事务操作，根据情况让参与者提交或者回滚请求。

Update 语句过程中在写完第一个日志后，第二个日志还没有写完期间发生了 crash，会出现什么情况呢?

1.  先写 redo log 后写 binlog 。

	假设在 redo log 写完，binlog 还没有写完的时候，MySQL 进程异常重启。由于我们前面说过的，redo log 写完之后，系统即使崩溃，仍然能够把数据恢复回来，所以恢复后这一行 c 的值是 1。  
	
	但是由于 binlog 没写完就 crash 了，这时候 binlog 里面就没有记录这个语句。因此，之后备份日志的时候，存起来的 binlog 里面就没有这条语句。然后你会发现，如果需要用这个 binlog 来恢复临时库的话，由于这个语句的 binlog 丢失，这个临时库就会少了这一次更新，恢复出来的这一行 c 的值就是 0，与原库的值不同。
 
2. 先写 binlog 后写 redo log 。

	如果在 binlog 写完之后 crash，由于 redo log 还没写，崩溃恢复。以后这个事务无效，所以这一行 c 的值是 0。但是 binlog 里面已经记录了“把 c 从 0 改成 1”这个日志。所以，在之后用 binlog 来恢复的时候就多了一个事务出来，恢复出来的这一行 c 的值就是 1，与原库的值不同。

可以看到，如果不使用“两阶段提交”，那么数据库的状态就有可能和用它的日志恢复出来的库的状态不一致。

### 1.2.4. 崩溃时的处理策略

- 在 redolog 写入磁盘后崩溃

	此时 redolog 是 prepare 阶段，binlog 尚未写入。因此对于这种情况可以直接丢弃 redolog 日志。
	
- Binlog 写入后崩溃

	此时 binlog 已经写入，准备将 redolog 更新为 commit 的时候崩溃。当崩溃恢复后，进行以下的处理：
	
	1. 判断恢复的 redolog 日志中是否有 commit 标识，如果有，认为事务已经提交，使用 redolog 日志更新内存中的数据。
	
		这么处理的原因是为了保持主从数据的一致性。当主库 binlog 日志产生后会同步到从库，从库就更新了数据。若主库恢复后数据不更新，那么就会造成数据的不一致。
		另一方面，binlog 日志已经写入磁盘就代表着事务已经提交，那么执行该事务也是合理的。
		
	2. 没有 commit 标识，进行 binlog 日志的判断
	
		1. Binlog 日志不完整，回滚事务。
		
		2. Binlog 日志完整，进行 redolog 日志的提交。
		
		一个事务的 binlog 是有完整格式的：
		
		- Statement 格式的 binlog，最后会有 COMMIT。
		
		- Row 格式的 binlog，最后会有一个 XID event。
		
	3. Redolog 与 binlog 通过事务 id 进行关联

### 1.2.5. Redolog 与 binlog 同时使用的原因

1. Redolog 是为了数据崩溃恢复使用的，binlog 是为了记录数据操作使用的。Redolog 记录了数据页发生的变化，方便在崩溃时恢复。而 binlog 记录了 sql 的执行操作，用于备份。

2. Redolog 记录是循环写的，比如 4G 的 redolog 日志空间，写到尾部后会回到头部继续写。而 binlog 是追加的。

3. 两阶段提交的原因是为了让 redolog 有办法处理 binlog 存储失败后的问题。当 binlog 存储失败后，redolog 可以不提交。而如果不采用两阶段提交，那么当 binlog 日志出现问题的时候 redolog 就回滚不了了。

### 1.2.6. 组提交策略

通过上面的分析我们可以知道，redolog 与 binlog 在事务提交时会发生两次写盘，为了优化这一阶段，我们可以采取组提交策略。对于 redolog 与 binlog 的操作争取一次写盘完成。

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20220513231453.png)

在将 redolog 写入到磁盘时，刷盘动作延迟到 binlog write 到 page cache 之后，这样 binlog 与 redolog 就有机会一同写入到磁盘中了。不过通常情况下第 3 步执行得会很快，所以 binlog 的 write 和 fsync 间的间隔时间短，导致能集合到一起持久化的 binlog 比较少，因此 binlog 的组提交的效果通常不如 redo log 的效果那么好。

如果你想提升 binlog 组提交的效果，可以通过设置 `binlog_group_commit_sync_delay` 和 `binlog_group_commit_sync_no_delay_count` 来实现：

- Binlog_group_commit_sync_delay 参数，表示延迟多少微秒后才调用 fsync; 

- Binlog_group_commit_sync_no_delay_count 参数，表示累积多少次以后才调用 fsync。

这两个条件是或的关系，也就是说只要有一个满足条件就会调用 fsync。所以，当 `binlog_group_commit_sync_delay` 设置为 0 的时候，`binlog_group_commit_sync_no_delay_count` 也无效了。

组提交策略原理：

- Redolog 与 binlog 都是顺序写，磁盘的顺序写比随机写要快。

- 组提交策略，大幅度降低磁盘的 IOPS 消耗。

## 1.3. 普通索引和唯一索引的选择

对于唯一索引，限制数据是唯一的，因此在执行插入或者修改操作的时候必须判断该数据是否重复。而对于普通索引则没有这个限制。

### 1.3.1. 普通索引和唯一索引查询的区别

- 唯一索引

	- 查询时通过 B+树查找，找到第一个满足条件的记录后就停止检索。
	
	- 更新时首先判断内存中是否有该数据，有的话就直接更新，否则需要加载到内存中然后更新。
	
- 普通索引

	- 查询时通过 B+树查找，找到满足条件的数据后还要判断下一条数据是否满足条件。大部分情况下这两条数据都在同一个页中，因此这个多出来的判断时间其实很小。但当下一条数据不在同一页，正好分页的时候，就需要多一次磁盘 IO 操作将其加载到页中了。当然这种情况比较少。
	
	- 更新时如果发现内存中有该数据，则直接更新内存。**内存中没有的话就在 ChangeBuffer 中记录该更新操作然后返回。**所以这个会比唯一索引的操作快很多。
	
	- ChangeBuffer 记录仅针对二级索引有效，**因为主键索引和唯一索引需要确定唯一性，所以就必须要把数据页加载到内存中进行检查。**当操作的二级索引不在内存中时就会缓存下来。等到下次读取该页时就与 ChangeBuffer 合并。

### 1.3.2. ChangeBuffer 的使用

ChangeBuffer 仅对普通索引生效。对于产生的 ChangeBuffer 数据，当数据被读取时，会从磁盘中加载然后与 ChangeBuffer 合并，从而产生最新的数据。后台也会定时刷新 ChangerBuffer 到缓存中。这个过程称为 ChangeBuffer 的 Merge。对于写后立即读的场景，使用 ChangeBuffer 会带来一定负担，所以如果数据库中所有表更新操作都会立即读取最新数据，那么关闭该功能反而会有不错的效果。

在 ChangeBuffer 产生后，就会将 ChangeBuffer 相关的操作写入 redolog 而后写入磁盘，因此无需担心 ChangeBuffer 丢失后造成的数据一致性问题。Redolog 主要节省的是随机写磁盘的 IO 消耗 (转成顺序写)，而 ChangeBuffer 主要节省的则是随机读磁盘的 IO 消耗。

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20220511222449.png)

## 1.4. 问题列表

### 1.4.1. 在 GTID 模式下，如果一个新的从库接上主库，但是需要的 binlog 已经没了，要怎么做？

- 如果业务允许主从不一致的情况，那么可以在主库上先执行 `show global variables like ‘gtid_purged’;`，得到主库已经删除的 GTID 集合，假设是 gtid_purged1；然后先在从库上执行 `reset master`，再执行 `set global gtid_purged =‘gtid_purged1’;` 最后执行 `start slave`，就会从主库现存的 binlog 开始同步。Binlog 缺失的那一部分，数据在从库上就可能会有丢失，造成主从不一致。

- 如果需要主从数据一致的话，最好还是通过重新搭建从库来做。

- 如果有其他的从库保留有全量的 binlog 的话，可以把新的从库先接到这个保留了全量 binlog 的从库，追上日志以后，如果有需要，再接回主库。

- 如果 binlog 有备份的情况，可以先在从库上应用缺失的 binlog，然后再执行 `start slave`。

### 1.4.2. MySQL 是怎么快速定位 binlog 里面的某一个 GTID 位置的？

在 binlog 文件头部的 Previous_gtids 可以解决这个问题。

### 1.4.3. 若采用 GTID 等定位方法做读写分离，对大表做 DDL 操作该如何进行？

假设，这条语句在主库上要执行 10 分钟，提交后传到备库就要 10 分钟（典型的大事务）。那么，在主库 DDL 之后再提交的事务的 GTID，去备库查的时候，就会等 10 分钟才出现。这样，这个读写分离机制在这 10 分钟之内都会超时，然后走主库。这种预期内的操作，应该在业务低峰期的时候，确保主库能够支持所有业务查询，然后把读请求都切到主库，再在主库上做 DDL。等备库延迟追上以后，再把读请求切回备库。

### 1.4.4. 如果一个事务被 kill 之后，持续处于回滚状态，从恢复速度的角度看，你是应该重启等它执行结束，还是应该强行重启整个 MySQL 进程?

因为重启之后该做的回滚动作还是不能少的，所以从恢复速度的角度来说，应该让它自己结束。

当然，如果这个语句可能会占用别的锁，或者由于占用 IO 资源过多，从而影响到了别的语句执行的话，就需要先做主备切换，切到新主库提供服务。

切换之后别的线程都断开了连接，自动停止执行。接下来还是等它自己执行完成。这个操作属于我们在文章中说到的，减少系统压力，加速终止逻辑。

### 1.4.5. 如果客户端由于压力过大，迟迟不能接收数据，会对服务端造成什么严重的影响？

这个问题的核心是，造成了“长事务”。至于长事务的影响，就要结合我们前面文章中提到的锁、MVCC 的知识点了。

如果前面的语句有更新，意味着它们在占用着行锁，会导致别的语句更新被锁住。

当然读的事务也有问题，就是会导致 undo log 不能被回收，导致回滚段空间膨胀。

### 1.4.6. 以下语句如何建立索引及关联查询合适？

```sql

select * from t1 join t2 on(t1.a=t2.a) join t3 on (t2.b=t3.b) where t1.c>=X and t2.c>=Y and t3.c>=Z;

```

第一原则是要尽量使用 BKA 算法。需要注意的是，使用 BKA 算法的时候，并不是“先计算两个表 join 的结果，再跟第三个表 join”，而是直接嵌套查询的。

具体实现是：在 t1. c>=X、t2. c>=Y、t3. c>=Z 这三个条件里，选择一个经过过滤以后，数据最少的那个表，作为第一个驱动表。

此时，可能会出现如下两种情况：

- 第一种情况，如果选出来是表 t1 或者 t3，那剩下的部分就固定了。

	- 如果驱动表是 t1，则连接顺序是 t1->t2->t3，要在**被驱动表字段创建上索引**，也就是 t2. a 和 t3. b 上创建索引；
	
	- 如果驱动表是 t3，则连接顺序是 t3->t2->t1，需要在 t2. b 和 t1. a 上创建索引。
	
	- 同时，我们还需要在第一个驱动表的字段 c 上创建索引。
	
- 第二种情况是，如果选出来的第一个驱动表是表 t2 的话，则需要评估另外两个条件的过滤效果。

总之，整体的思路就是，尽量让每一次参与 join 的驱动表的数据集，越小越好，因为这样我们的驱动表就会越小。

### 1.4.7. 为什么不能用 rename 修改临时表的改名？

在实现上，执行 rename table 语句的时候，要求按照“库名 / 表名.frm”的规则去磁盘找文件，但是临时表在磁盘上的 frm 文件是放在 tmpdir 目录下的，并且文件名的规则是`“#sql{进程 id}\_{线程 id}\_ 序列号.frm”`，因此会报“找不到文件名”的错误。