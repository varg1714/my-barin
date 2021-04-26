> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/wSlNZcQkax-2KZCNEHOYLA)

说在前面的话  

本文是用来系统阐述在 MySQL 中，不同语句在各种条件下的加锁情况，并不是解释各种锁是什么（或者说加锁的本质是什么），大家如果不理解什么是`MVCC`、`ReadView`、`正经记录锁`、`gap锁`、`next-key锁`、`插入意向锁`这些概念的，可以参考`MySQL`的官方文档，或者直接参照《MySQL 是怎样运行的：从根儿上理解 MySQL》这本小册（里边有比官方文档更贴心，更详细的解释，文章中涉及到的所有概念均在小册中有详细解释，有疑惑，并且有兴趣的同学可以扫描下边二维码看看）：

![](https://mmbiz.qpic.cn/mmbiz_png/RLmbWWew55GM3XXtto9wTRRBb4WCDIUkVLtdRQQCPfvR0jvdzibUQA2iaUOjGaUSZvzIIcVgYR1DiaJvFQHIlC6CA/640?wx_fmt=png)
建议：

1. 本篇文章不适合碎片化时间阅读，最好使用电脑观看，或者将字体跳到最小效果好一些
2.  可能一下子看不完，关注 + 收藏 + 好看 + 转发一波  

3. 不要跳着看  

**事前准备**  

建立一个存储三国英雄的`hero`表：

```sql
CREATE TABLE hero (   
number INT,    
name VARCHAR(100),    
country varchar(100),    
PRIMARY KEY (number),    
KEY idx_name (name)) Engine=InnoDB CHARSET=utf8;
```

然后向这个表里插入几条记录：

```sql
INSERT INTO hero VALUES (1, 'l刘备', '蜀'),
(3, 'z诸葛亮', '蜀'), (8, 'c曹操', '魏'),
(15, 'x荀彧', '魏'), (20, 's孙权', '吴');
```

然后现在`hero`表就有了两个索引（一个二级索引，一个聚簇索引），示意图如下：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131229.jpeg)

#数据库 
# 语句加锁分析
----------

其实啊，“XXX 语句该加什么锁” 本身就是个伪命题，一条语句需要加的锁受到很多条件制约，比方说：

*   事务的隔离级别
    
*   语句执行时使用的索引（比如聚簇索引、唯一二级索引、普通二级索引）
    
*   查询条件（比方说`=`、`=<`、`>=`等等）
    
*   具体执行的语句类型
    

在继续详细分析语句的加锁过程前，大家一定要有一个全局概念：`加锁`只是解决并发事务执行过程中引起的`脏写`、`脏读`、`不可重复读`、`幻读`这些问题的一种解决方案（`MVCC`算是一种解决`脏读`、`不可重复读`、`幻读`这些问题的一种解决方案），一定要意识到`加锁`的出发点是为了解决这些问题，不同情景下要解决的问题不一样，才导致加的锁不一样，千万不要为了加锁而加锁，容易把自己绕进去。当然，有时候因为`MySQL`具体的实现而导致一些情景下的加锁有些不太好理解，这就得我们死记硬背了～

我们这里把语句分为 3 种大类：普通的`SELECT`语句、锁定读的语句、`INSERT`语句，我们分别看一下。

## 普通的 SELECT 语句

普通的`SELECT`语句在：

*   `READ UNCOMMITTED`隔离级别下，不加锁，直接读取记录的最新版本，可能发生`脏读`、`不可重复读`和`幻读`问题。
    
*   `READ COMMITTED`隔离级别下，不加锁，在每次执行普通的`SELECT`语句时都会生成一个`ReadView`，这样解决了`脏读`问题，但没有解决`不可重复读`和`幻读`问题。
    
*   `REPEATABLE READ`隔离级别下，不加锁，只在第一次执行普通的`SELECT`语句时生成一个`ReadView`，这样把`脏读`、`不可重复读`和`幻读`问题都解决了。
    
    不过这里有一个小插曲：
    
    ```sql
    # 事务T1，REPEATABLE READ隔离级别下
	mysql> BEGIN;
	Query OK, 0 rows affected (0.00 sec)
	mysql> SELECT * FROM hero WHERE number = 30;
	Empty set (0.01 sec)
	# 此时事务T2执行了：INSERT INTO hero VALUES(30, 'g关羽', '魏'); 并提交
	mysql> UPDATE hero SET country = '蜀' WHERE number = 30;
	Query OK, 1 row affected (0.01 sec)Rows matched: 1  Changed: 1  Warnings: 0
	mysql> SELECT * FROM hero WHERE number = 30;
	+--------+---------+---------+| number | name    | country |+--------+---------+---------+|     30 | g关羽   | 蜀      |+--------+---------+---------+1 row in set (0.01 sec)
    ```
    
    在`REPEATABLE READ`隔离级别下，`T1`第一次执行普通的`SELECT`语句时生成了一个`ReadView`，之后`T2`向`hero`表中新插入了一条记录便提交了，`ReadView`并不能阻止`T1`执行`UPDATE`或者`DELETE`语句来对改动这个新插入的记录（因为`T2`已经提交，改动该记录并不会造成阻塞），但是这样一来这条新记录的`trx_id`隐藏列就变成了`T1`的`事务id`，之后`T1`中再使用普通的`SELECT`语句去查询这条记录时就可以看到这条记录了，也就把这条记录返回给客户端了。因为这个特殊现象的存在，你也可以认为`InnoDB`中的`MVCC`并不能完完全全的禁止幻读。
    
*   `SERIALIZABLE`隔离级别下，需要分为两种情况讨论：
    
*   在系统变量`autocommit=0`时，也就是禁用自动提交时，普通的`SELECT`语句会被转为`SELECT ... LOCK IN SHARE MODE`这样的语句，也就是在读取记录前需要先获得记录的`S锁`，具体的加锁情况和`REPEATABLE READ`隔离级别下一样，我们后边再分析。
    
*   在系统变量`autocommit=1`时，也就是启用自动提交时，普通的`SELECT`语句并不加锁，只是利用`MVCC`来生成一个`ReadView`去读取记录。
    
    为啥不加锁呢？因为启用自动提交意味着一个事务中只包含一条语句，一条语句也就没有啥`不可重复读`、`幻读`这样的问题了。
    

## 锁定读的语句

我们把下边四种语句放到一起讨论：

*   语句一：`SELECT ... LOCK IN SHARE MODE;`
    
*   语句二：`SELECT ... FOR UPDATE;`
    
*   语句三：`UPDATE ...`
    
*   语句四：`DELETE ...`
    

我们说`语句一`和`语句二`是`MySQL`中规定的两种`锁定读`的语法格式，而`语句三`和`语句四`由于在执行过程需要首先定位到被改动的记录并给记录加锁，也可以被认为是一种`锁定读`。

### READ UNCOMMITTED/READ COMMITTED 隔离级别下

在`READ UNCOMMITTED`下语句的加锁方式和`READ COMMITTED`隔离级别下语句的加锁方式基本一致，所以就放到一块儿说了。值得注意的是，采用`加锁`方式解决并发事务带来的问题时，其实`脏读`和`不可重复读`在任何一个隔离级别下都不会发生（因为`读-写`操作需要排队进行）。

#### 对于使用主键进行等值查询的情况

*   使用`SELECT ... LOCK IN SHARE MODE`来为记录加锁，比方说：
    
    ```sql
    SELECT * FROM hero WHERE number = 8 LOCK IN SHARE MODE;
    ```
    
    这个语句执行时只需要访问一下聚簇索引中`number`值为`8`的记录，所以只需要给它加一个`S型正经记录锁`就好了，如图所示：
    
    ![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131237.jpeg)
*   使用`SELECT ... FOR UPDATE`来为记录加锁，比方说：
    
    ```sql
    SELECT * FROM hero WHERE number = 8 FOR UPDATE;
    ```
    
    这个语句执行时只需要访问一下聚簇索引中`number`值为`8`的记录，所以只需要给它加一个`X型正经记录锁`就好了，如图所示：
    
    ![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131241.jpeg)
    
    > 小贴士： 为了区分 S 锁和 X 锁，我们之后在示意图中就把加了 S 锁的记录染成蓝色，把加了 X 锁的记录染成紫色。
    
*   使用`UPDATE ...`来为记录加锁，比方说：
    
    ```sql
    UPDATE hero SET country = '汉' WHERE number = 8;
    ```
    
    这条`UPDATE`语句并没有更新二级索引列，加锁方式和上边所说的`SELECT ... FOR UPDATE`语句一致。
    
    如果`UPDATE`语句中更新了二级索引列，比方说：
    
    ```sql
    UPDATE hero SET name = 'cao曹操' WHERE number = 8;
    ```
    
    该语句的实际执行步骤是首先更新对应的`number`值为`8`的聚簇索引记录，再更新对应的二级索引记录，所以加锁的步骤就是：
    

1.  为`number`值为`8`的聚簇索引记录加上`X型正经记录锁`（该记录对应的）。
    
2.  为该聚簇索引记录对应的`idx_name`二级索引记录（也就是`name`值为`'c曹操'`，`number`值为`8`的那条二级索引记录）加上`X型正经记录锁`。
    

画个图就是这样：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131245.jpeg)

> 小贴士： 我们用带圆圈的数字来表示为各条记录加锁的顺序。

*   使用`DELETE ...`来为记录加锁，比方说：
    
    ```sql
    DELETE FROM hero WHERE number = 8;
    ```
    
    我们平时所说的 “DELETE 表中的一条记录” 其实意味着对聚簇索引和所有的二级索引中对应的记录做`DELETE`操作，本例子中就是要先把`number`值为`8`的聚簇索引记录执行`DELETE`操作，然后把对应的`idx_name`二级索引记录删除，所以加锁的步骤和上边更新带有二级索引列的`UPDATE`语句一致，就不画图了。
    

#### 对于使用主键进行范围查询的情况

*   使用`SELECT ... LOCK IN SHARE MODE`来为记录加锁，比方说：
    
    ```sql
    SELECT * FROM hero WHERE number <= 8 LOCK IN SHARE MODE;
    ```
    
    这个语句看起来十分简单，但它的执行过程还是有一丢丢小复杂的：
    

1.  先到聚簇索引中定位到满足`number <= 8`的第一条记录，也就是`number`值为`1`的记录，然后为其加锁。
    
2.  判断一下该记录是否符合`索引条件下推`中的条件。
    
    我们前边介绍过一个称之为`索引条件下推`（ `Index Condition Pushdown`，简称`ICP`）的功能，也就是把查询中与被使用索引有关的查询条件下推到存储引擎中判断，而不是返回到`server`层再判断。不过需要注意的是，`索引条件下推`只是为了减少回表次数，也就是减少读取完整的聚簇索引记录的次数，从而减少`IO`操作。而对于`聚簇索引`而言不需要回表，它本身就包含着全部的列，也起不到减少`IO`操作的作用，所以设计`InnoDB`的大叔们规定这个`索引条件下推`特性只适用于`二级索引`。也就是说在本例中与被使用索引有关的条件是：`number <= 8`，而`number`列又是聚簇索引列，所以本例中并没有符合`索引条件下推`的查询条件，自然也就不需要判断该记录是否符合`索引条件下推`中的条件。
    
3.  判断一下该记录是否符合范围查询的边界条件
    
    因为在本例中是利用主键`number`进行范围查询，设计`InnoDB`的大叔规定每从聚簇索引中取出一条记录时都要判断一下该记录是否符合范围查询的边界条件，也就是`number <= 8`这个条件。如果符合的话将其返回给`server层`继续处理，否则的话需要释放掉在该记录上加的锁，并给`server层`返回一个查询完毕的信息。
    
    对于`number`值为`1`的记录是符合这个条件的，所以会将其返回到`server层`继续处理。
    
4.  将该记录返回到`server层`继续判断。
    
    `server层`如果收到存储引擎层提供的查询完毕的信息，就结束查询，否则继续判断那些没有进行`索引条件下推`的条件，在本例中就是继续判断`number <= 8`这个条件是否成立。噫，不是在第 3 步中已经判断过了么，怎么在这又判断一回？是的，设计`InnoDB`的大叔采用的策略就是这么简单粗暴，把凡是没有经过`索引条件下推`的条件都需要放到`server`层再判断一遍。如果该记录符合剩余的条件（没有进行`索引条件下推`的条件），那么就把它发送给客户端，不然的话需要释放掉在该记录上加的锁。
    
5.  然后刚刚查询得到的这条记录（也就是`number`值为`1`的记录）组成的单向链表继续向后查找，得到了`number`值为`3`的记录，然后重复第`2`，`3`，`4`、`5`这几个步骤。
    

> 小贴士： 上述步骤是在 MySQL 5.7.21 这个版本中验证的，不保证其他版本有无出入。

但是这个过程有个问题，就是当找到`number`值为`8`的那条记录的时候，还得向后找一条记录（也就是`number`值为`15`的记录），在存储引擎读取这条记录的时候，也就是上述的第`1`步中，就得为这条记录加锁，然后在第 3 步时，判断该记录不符合`number <= 8`这个条件，又要释放掉这条记录的锁，这个过程导致`number`值为`15`的记录先被加锁，然后把锁释放掉，过程就是这样：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131250.jpeg)

这个过程有意思的一点就是，如果你先在事务`T1`中执行：

```sql
# 事务T1BEGIN;SELECT * FROM hero WHERE number <= 8 LOCK IN SHARE MODE;
```

然后再到事务`T2`中执行：

```sql
# 事务T2BEGIN;SELECT * FROM hero WHERE number = 15 FOR UPDATE;
```

是没有问题的，因为在`T2`执行时，事务`T1`已经释放掉了`number`值为`15`的记录的锁，但是如果你先执行`T2`，再执行`T1`，由于`T2`已经持有了`number`值为`15`的记录的锁，事务`T1`将因为获取不到这个锁而等待。

我们再看一个使用主键进行范围查询的例子：

```sql
SELECT * FROM hero WHERE number >= 8 LOCK IN SHARE MODE;
```

这个语句的执行过程其实和我们举的上一个例子类似。也是先到聚簇索引中定位到满足`number >= 8`这个条件的第一条记录，也就是`number`值为`8`的记录，然后就可以沿着由记录组成的单向链表一路向后找，每找到一条记录，就会为其加上锁，然后判断该记录符不符合范围查询的边界条件，不过这里的边界条件比较特殊：`number >= 8`，只要记录不小于 8 就算符合边界条件，所以判断和没判断是一样一样的。最后把这条记录返回给`server层`，`server层`再判断`number >= 8`这个条件是否成立，如果成立的话就发送给客户端，否则的话就结束查询。不过`InnoDB`存储引擎找到索引中的最后一条记录，也就是`Supremum`伪记录之后，在存储引擎内部就可以立即判断这是一条伪记录，不必要返回给`server层`处理，也没必要给它也加上锁（也就是说在第 1 步中就压根儿没给这条记录加锁）。整个过程会给`number`值为`8`、`15`、`20`这三条记录加上`S型正经记录锁`，画个图表示一下就是这样：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131256.jpeg)*   使用`SELECT ... FOR UPDATE`语句来为记录加锁：
    
    和`SELECT ... IN SHARE MODE`语句类似，只不过加的是`X型正经记录锁`。

*   使用`UPDATE ...`来为记录加锁，比方说：
    
    ```sql
    UPDATE hero SET country = '汉' WHERE number >= 8;
    ```
    
    这条`UPDATE`语句并没有更新二级索引列，加锁方式和上边所说的`SELECT ... FOR UPDATE`语句一致。
    
    如果`UPDATE`语句中更新了二级索引列，比方说：
    
    ```sql
    UPDATE hero SET name = 'cao曹操' WHERE number >= 8;
    ```
    
    这时候会首先更新聚簇索引记录，再更新对应的二级索引记录，所以加锁的步骤就是：
    

1.  为`number`值为`8`的聚簇索引记录加上`X型正经记录锁`。
    
2.  然后为上一步中的记录索引记录对应的`idx_name`二级索引记录加上`X型正经记录锁`。
    
3.  为`number`值为`15`的聚簇索引记录加上`X型正经记录锁`。
    
4.  然后为上一步中的记录索引记录对应的`idx_name`二级索引记录加上`X型正经记录锁`。
    
5.  为`number`值为`20`的聚簇索引记录加上`X型正经记录锁`。
    
6.  然后为上一步中的记录索引记录对应的`idx_name`二级索引记录加上`X型正经记录锁`。
    

画个图就是这样：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131302.jpeg)

如果是下边这个语句：

```sql
UPDATE hero SET namey = '汉' WHERE number <= 8;
```

则会对`number`值为`1`、`3`、`8`聚簇索引记录以及它们对应的二级索引记录加`X型正经记录锁`，加锁顺序和上边语句中的加锁顺序类似，都是先对一条聚簇索引记录加锁后，再给对应的二级索引记录加锁。之后会继续对`number`值为`15`的聚簇索引记录加锁，但是随后`InnoDB`存储引擎判断它不符合边界条件，随即会释放掉该聚簇索引记录上的锁（注意这个过程中没有对`number`值为`15`的聚簇索引记录对应的二级索引记录加锁）。具体示意图就不画了。

*   使用`DELETE ...`来为记录加锁，比方说：
    
    ```sql
    DELETE FROM hero WHERE number >= 8;
    ```
    
    和
    
    ```sql
    DELETE FROM hero WHERE number <= 8;
    ```
    
    这两个语句的加锁情况和更新带有二级索引列的`UPDATE`语句一致，就不画图了。
    

#### 对于使用二级索引进行等值查询的情况

> 小贴士： 在 READ UNCOMMITTED 和 READ COMMITTED 隔离级别下，使用普通的二级索引和唯一二级索引进行加锁的过程是一样的，所以我们也就不分开讨论了。

*   使用`SELECT ... LOCK IN SHARE MODE`来为记录加锁，比方说：
    
    ```sql
    SELECT * FROM hero WHERE name = 'c曹操' LOCK IN SHARE MODE;
    ```
    
    这个语句的执行过程是先通过二级索引`idx_name`定位到满足`name = 'c曹操'`条件的二级索引记录，然后进行回表操作。所以先要对二级索引记录加`S型正经记录锁`，然后再给对应的聚簇索引记录加`S型正经记录锁`，示意图如下：
    
    ![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131307.jpeg)
    
    
    这里需要再次强调一下这个语句的加锁顺序：
    

1.  先对`name`列为`'c曹操'`二级索引记录进行加锁。
    
2.  再对相应的聚簇索引记录进行加锁
    

> 小贴士： 我们知道 idx_name 是一个普通的二级索引，到 idx_name 索引中定位到满足 name= 'c 曹操'这个条件的第一条记录后，就可以沿着这条记录一路向后找。可是从我们上边的描述中可以看出来，并没有对下一条二级索引记录进行加锁，这是为什么呢？<font style = "color:orange">这是因为设计 InnoDB 的大叔对等值匹配的条件有特殊处理，</font>他们规定在 InnoDB 存储引擎层查找到当前记录的下一条记录时，在对其加锁前就直接判断该记录是否满足等值匹配的条件，如果不满足直接返回（也就是不加锁了），否则的话需要将其加锁后再返回给 server 层。所以这里也就不需要对下一条二级索引记录进行加锁了。

现在要介绍一个非常有趣的事情，我们假设上边这个语句在事务`T1`中运行，然后事务`T2`中运行下边一个我们之前介绍过的语句：

```sql
UPDATE hero SET name = '曹操' WHERE number = 8;
```

这两个语句都是要对`number`值为`8`的聚簇索引记录和对应的二级索引记录加锁，但是不同点是加锁的顺序不一样。这个`UPDATE`语句是先对聚簇索引记录进行加锁，后对二级索引记录进行加锁，如果在不同事务中运行上述两个语句，可能发生一种贼奇妙的事情 ——

*   事务`T2`持有了聚簇索引记录的锁，事务`T1`持有了二级索引记录的锁。
    
*   事务`T2`在等待获取二级索引记录上的锁，事务`T1`在等待获取聚簇索引记录上的锁。
    

两个事务都分别持有一个锁，而且都在等待对方已经持有的那个锁，这种情况就是所谓的`死锁`，两个事务都无法运行下去，必须选择一个进行回滚，对性能影响比较大。  

*   使用`SELECT ... FOR UPDATE`语句时，比如：
    
    ```sql
    SELECT * FROM hero WHERE name = 'c曹操' FOR UPDATE;
    ```
    
    这种情况下与`SELECT ... LOCK IN SHARE MODE`语句的加锁情况类似，都是给访问到的二级索引记录和对应的聚簇索引记录加锁，只不过加的是`X型正经记录锁`罢了。
    
*   使用`UPDATE ...`来为记录加锁，比方说：
    
    与更新二级索引记录的`SELECT ... FOR UPDATE`的加锁情况类似，不过如果被更新的列中还有别的二级索引列的话，对应的二级索引记录也会被加锁。
    
*   使用`DELETE ...`来为记录加锁，比方说：
    
    与`SELECT ... FOR UPDATE`的加锁情况类似，不过如果表中还有别的二级索引列的话，对应的二级索引记录也会被加锁。
    

#### 对于使用二级索引进行范围查询的情况

*   使用`SELECT ... LOCK IN SHARE MODE`来为记录加锁，比方说：
    
    ```sql
    SELECT * FROM hero FORCE INDEX(idx_name)  WHERE name >= 'c曹操' LOCK IN SHARE MODE;
    ```
    
    > 小贴士： 因为优化器会计算使用二级索引进行查询的成本，在成本较大时可能选择以全表扫描的方式来执行查询，所以我们这里使用 FORCE INDEX(idx_name) 来强制使用二级索引 idx_name 来执行查询。
    
    这个语句的执行过程其实是先到二级索引中定位到满足`name >= 'c曹操'`的第一条记录，也就是`name`值为`c曹操`的记录，然后就可以沿着这条记录的链表一路向后找，从二级索引`idx_name`的示意图中可以看出，所有的用户记录都满足`name >= 'c曹操'`的这个条件，所以所有的二级索引记录都会被加`S型正经记录锁`，它们对应的聚簇索引记录也会被加`S型正经记录锁`。不过需要注意一下加锁顺序，对一条二级索引记录加锁完后，会接着对它相应的聚簇索引记录加锁，完后才会对下一条二级索引记录进行加锁，以此类推～ 画个图表示一下就是这样：
    
    ![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131315.jpeg)
    
    再来看下边这个语句：
    
    ```sql
    SELECT * FROM hero FORCE INDEX(idx_name) WHERE name <= 'c曹操' LOCK IN SHARE MODE;
    ```
    
    这个语句的加锁情况就有点儿有趣了。前边说在使用`number <= 8`这个条件的语句中，需要把`number`值为`15`的记录也加一个锁，之后又判断它不符合边界条件而把锁释放掉。而对于查询条件`name <= 'c曹操'`的语句来说，执行该语句需要使用到二级索引，而与二级索引相关的条件是可以使用`索引条件下推`这个特性的。设计`InnoDB`的大叔规定，如果一条记录不符合`索引条件下推`中的条件的话，直接跳到下一条记录（这个过程根本不将其返回到`server层`），如果这已经是最后一条记录，那么直接向`server层`报告查询完毕。但是这里头有个问题呀：先对一条记录加了锁，然后再判断该记录是不是符合索引条件下推的条件，如果不符合直接跳到下一条记录或者直接向 server 层报告查询完毕，这个过程中并没有把那条被加锁的记录上的锁释放掉呀！！！。本例中使用的查询条件是`name <= 'c曹操'`，在为`name`值为`'c曹操'`的二级索引记录以及它对应的聚簇索引加锁之后，会接着二级索引中的下一条记录，也就是`name`值为`'l刘备'`的那条二级索引记录，由于该记录不符合`索引条件下推`的条件，而且是范围查询的最后一条记录，会直接向`server层`报告查询完毕，重点是这个过程中并不会释放`name`值为`'l刘备'`的二级索引记录上的锁，也就导致了语句执行完毕时的加锁情况如下所示：
    
    ![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131319.jpeg)
    
    这样子会造成一个尴尬情况，假如`T1`执行了上述语句并且尚未提交，`T2`再执行这个语句：
    
    ```sql
    SELECT * FROM hero WHERE name = 'l刘备' FOR UPDATE;
    ```
    
    `T2`中的语句需要获取`name`值为`l刘备`的二级索引记录上的`X型正经记录锁`，而`T1`中仍然持有`name`值为`l刘备`的二级索引记录上的`S型正经记录锁`，这就造成了`T2`获取不到锁而进入等待状态。
    
    > 小贴士： 为啥不能释放不符合索引条件下推中的条件的二级索引记录上的锁呢？这个问题我也没想明白，人家就是这么规定的，如果有明白的小伙伴可以加我微信 xiaohaizi4919 来讨论一下哈～ 再强调一下，我使用的 MySQL 版本是 5.7.21，不保证其他版本中的加锁情景是否完全一致。
    
*   使用`SELECT ... FOR UPDATE`语句时：
    
    和`SELECT ... IN SHARE MODE`语句类似，只不过加的是`X型正经记录锁`。
    
*   使用`UPDATE ...`来为记录加锁，比方说：
    
    ```sql
    UPDATE hero SET country = '汉' WHERE name >= 'c曹操';
    ```
    
    > 小贴士： FORCE INDEX 只对 SELECT 语句起作用，UPDATE 语句虽然支持该语法，但实质上不起作用，DELETE 语句压根儿不支持该语法。
    
    假设该语句执行时使用了`idx_name`二级索引来进行`锁定读`，那么它的加锁方式和上边所说的`SELECT ... FOR UPDATE`语句一致。如果有其他二级索引列也被更新，那么也会为对应的二级索引记录进行加锁，就不赘述了。不过还有一个有趣的情况，比方说：
    
    ```sql
    UPDATE hero SET country = '汉' WHERE name <= 'c曹操';
    ```
    
    我们前边说的`索引条件下推`这个特性只适用于`SELECT`语句，也就是说`UPDATE`语句中无法使用，那么这个语句就会为`name`值为`'c曹操'`和`'l刘备'`的二级索引记录以及它们对应的聚簇索引进行加锁，之后在判断边界条件时发现`name`值为`'l刘备'`的二级索引记录不符合`name <= 'c曹操'`条件，再把该二级索引记录和对应的聚簇索引记录上的锁释放掉。这个过程如下图所示：
    
    ![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131325.jpeg)
*   使用`DELETE ...`来为记录加锁，比方说：
    
    ```sql
    DELETE FROM hero WHERE name >= 'c曹操';
    ```
    
    和
    
    ```
    DELETE FROM hero WHERE name <= 'c曹操';
    ```
    
    如果这两个语句采用二级索引来进行`锁定读`，那么它们的加锁情况和更新带有二级索引列的`UPDATE`语句一致，就不画图了。
    

#### 全表扫描的情况

比方说：

```sql
SELECT * FROM hero WHERE country  = '魏' LOCK IN SHARE MODE;
```

由于`country`列上未建索引，所以只能采用全表扫描的方式来执行这条查询语句，存储引擎每读取一条聚簇索引记录，就会为这条记录加锁一个`S型正常记录锁`，然后返回给`server层`，如果`server层`判断`country = '魏'`这个条件是否成立，如果成立则将其发送给客户端，否则会释放掉该记录上的锁，画个图就像这样：

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210414131331.jpeg)

使用`SELECT ... FOR UPDATE`进行加锁的情况与上边类似，只不过加的是`X型正经记录锁`，就不赘述了。

对于`UPDATE ...`和`DELETE ...`的语句来说，在遍历聚簇索引中的记录，都会为该聚簇索引记录加上`X型正经记录锁`，然后：

*   如果该聚簇索引记录不满足条件，直接把该记录上的锁释放掉。
    
*   如果该聚簇索引记录满足条件，则会对相应的二级索引记录加上`X型正经记录锁`（<font style = "color:red">DELETE语句会对所有二级索引列加锁，UPDATE语句只会为更新的二级索引列对应的二级索引记录加锁（这个加锁指的是如果update语句更新的值涉及到二姐索引列，则会对二级索引进行加锁，否则仅对聚簇索引加锁。）</font>）。

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ODbju9fjB5QFEN8IIYp__A)

说在前面的话  

本文是用来系统阐述在 MySQL 中，不同语句在各种条件下的加锁情况，并不是解释各种锁是什么（或者说加锁的本质是什么），大家如果不理解什么是`MVCC`、`ReadView`、`正经记录锁`、`gap锁`、`next-key锁`、`插入意向锁`这些概念的，可以参考`MySQL`的官方文档，或者直接参照《MySQL 是怎样运行的：从根儿上理解 MySQL》这本小册（里边有比官方文档更贴心，更详细的解释，文章中涉及到的所有概念均在小册中有详细解释，有疑惑，并且有兴趣的同学可以扫描下边二维码看看）：

![](https://mmbiz.qpic.cn/mmbiz_png/RLmbWWew55GM3XXtto9wTRRBb4WCDIUkVLtdRQQCPfvR0jvdzibUQA2iaUOjGaUSZvzIIcVgYR1DiaJvFQHIlC6CA/640?wx_fmt=png)建议：        1. 本篇文章不适合碎片化时间阅读，最好使用电脑观看，或者将字体跳到最小效果好一些        2. 可能一下子看不完，关注 + 收藏 + 好看 + 转发一波  

        3. 不要跳着看

        4. 一定要先看过上一篇文章：[超全面 MySQL 语句加锁分析（上篇）（求转）](http://mp.weixin.qq.com/s?__biz=MzIxNTQ3NDMzMw==&mid=2247484169&idx=1&sn=f06eac890ea0f0810cedd6a2ca62fdd3&chksm=97968afba0e103ed979f2c0e75448cc78c42b094a895e1d43eb9cd81da97c013146621e422ea&scene=21#wechat_redirect)  

### REPEATABLE READ 隔离级别下

采用`加锁`的方式解决并发事务产生的问题时，`REPEATABLE READ`隔离级别与`READ UNCOMMITTED`和`READ COMMITTED`这两个隔离级别相比，最主要的就是要解决`幻读`问题，`幻读`问题的解决还得靠我们之前讲过的`gap锁`。

#### 对于使用主键进行等值查询的情况

*   使用`SELECT ... LOCK IN SHARE MODE`来为记录加锁，比方说：
    
    ```
    SELECT * FROM hero WHERE number = 8 LOCK IN SHARE MODE;
    ```
    
    我们知道主键具有唯一性，如果在一个事务中第一次执行上述语句时将得到的结果集中包含一条记录，第二次执行上述语句前肯定不会有别的事务插入多条`number`值为`8`的记录（主键具有唯一性），也就是说一个事务中两次执行上述语句并不会发生幻读，这种情况下和`READ UNCOMMITTED／READ COMMITTED`隔离级别下一样，我们只需要为这条`number`值为`8`的记录加一个`S型正经记录锁`就好了，如图所示：
    
    ![](https://mmbiz.qpic.cn/mmbiz/RLmbWWew55Ea7XI8DjgG6TqefjLcfkqRRBrR8vcrUrmlunXL71eBQZ1XOvgKaqhCtR7H4bia831ygXBXJiboZ14Q/640?wx_fmt=other)
    
    但是如果我们要查询主键值不存在的记录，比方说：
    
    ```
    SELECT * FROM hero WHERE number = 7 LOCK IN SHARE MODE;
    ```
    
    由于`number`值为`7`的记录不存在，为了禁止`幻读`现象（也就是避免在同一事务中下一次执行相同语句时得到的结果集中包含`number`值为`7`的记录），在当前事务提交前我们需要预防别的事务插入`number`值为`7`的新记录，所以需要在`number`值为`8`的记录上加一个`gap锁`，也就是不允许别的事务插入`number`值在`(3, 8)`这个区间的新记录。画个图表示一下：
    
    ![](https://mmbiz.qpic.cn/mmbiz/RLmbWWew55Ea7XI8DjgG6TqefjLcfkqRQibHCdibGVW5tUgYgeVNeX0UqxL1edxTWBhE6uCHQWEpm3Dyyyp5vCmg/640?wx_fmt=other)
    
    如果在`READ UNCOMMITTED／READ COMMITTED`隔离级别下一样查询了一条主键值不存在的记录，那么什么锁也不需要加，因为在`READ UNCOMMITTED／READ COMMITTED`隔离级别下，并不需要禁止`幻读`问题。
    
*   其余语句使用主键进行等值查询的情况与`READ UNCOMMITTED／READ COMMITTED`隔离级别下的情况类似，这里就不赘述了。
    

#### 对于使用主键进行范围查询的情况

*   使用`SELECT ... LOCK IN SHARE MODE`语句来为记录加锁，比方说：
    
    ```
    SELECT * FROM hero WHERE number >= 8 LOCK IN SHARE MODE;
    ```
    
    因为要解决幻读问题，所以需要禁止别的事务插入`number`值符合`number >= 8`的记录，又因为主键本身就是唯一的，所以我们不用担心在`number`值为`8`的前边有新记录插入，只需要保证不要让新记录插入到`number`值为`8`的后边就好了，所以：
    

*   为`number`值为`8`的聚簇索引记录加一个`S型正经记录锁`。
    
*   为`number`值大于`8`的所有聚簇索引记录都加一个`S型next-key锁`（包括`Supremum`伪记录）。
    

画个图就是这样子：

![](https://mmbiz.qpic.cn/mmbiz/RLmbWWew55Ea7XI8DjgG6TqefjLcfkqRYex6Z4zPF0DNV4c2EVoTjRBWnVPib1R7Eq3YHrp32Tyc7rjllTTTMBQ/640?wx_fmt=other)

> 小贴士： 为什么不给 Supremum 记录加 gap 锁，而要加 next-key 锁呢？其实设计 InnoDB 的大叔在处理 Supremum 记录上加的 next-key 锁时就是当作 gap 锁看待的，只不过为了节省锁结构（我们前边说锁的类型不一样的话不能被放到一个锁结构中）才这么做的而已，大家不必在意。

与`READ UNCOMMITTED/READ COMMITTED`隔离级别类似，在`REPEATABLE READ`隔离级别下，下边这个范围查询也是有点特殊：

```sql
SELECT * FROM hero WHERE number <= 8 LOCK IN SHARE MODE;
```

这个语句的执行过程我们在之前唠叨过，在`READ UNCOMMITTED/READ COMMITTED`隔离级别下，这个语句会为`number`值为`1`、`3`、`8`、`15`这 4 条记录都加上`S型正经记录锁`，然后由于`number`值为`15`的记录不满足边界条件`number <= 8`，随后便把这条记录的锁释放掉。在`REPEATABLE READ`隔离级别下的加锁过程与之类似，不过会为`1`、`3`、`8`、`15`这 4 条记录都加上`S型next-key锁`，但是有一点需要大家十分注意：REPEATABLE READ 隔离级别下，在判断 number 值为 15 的记录不满足边界条件 number <= 8 后，并不会去释放加在该记录上的锁！！！ 所以在`REPEATABLE READ`隔离级别下，该语句的加锁示意图就如下所示：

![](https://mmbiz.qpic.cn/mmbiz/RLmbWWew55Ea7XI8DjgG6TqefjLcfkqRTIt5kR5Ht9FLmOTNvmFvs2pYecNHZha7kU9ibFZWl3WWdBz3kAD0Pfg/640?wx_fmt=other)

这样如果别的事务想要插入的新记录的`number`值在`(-∞, 1]`、`(1, 3]`、`(3, 8]`、`(8, 15]`之间的话，是会进入等待状态的。

> 小贴士： 很显然这么粗暴的做法导致的一个后果就是别的事务竟然不允许插入 number 值在 (8, 15] 这个区间中的新记录，甚至不允许别的事务再获取 number 值为 15 的记录上的锁，而理论上只需要禁止别的事务插入 number 值在 (-∞, 8] 之间的新记录就好。

<br/>
<font style = "color:red">上述情况在mysql8.17版本中未出现，在8.17版本中执行上述语句后，会在`(-∞, 1]`、`(1, 3]`、`(3, 8]`加上next-key锁，但是在(8,15)之间则并未加锁。若是执行以下sql的话，则情况会有所不同:</font>

```sql
SELECT * FROM hero WHERE number <= 9 LOCK IN SHARE MODE;
```

<font style = "color:red">在这种情况下，会在`(-∞, 1]`、`(1, 3]`、`(3, 8]`加上next-key锁，同时会在(8,15)加上gap锁，不允许在(8,15)的区间内插入数据。但若是操作number = 15 的这条数据的话，是没有问题的。</font>

*   使用`SELECT ... FOR UPDATE`语句来为记录加锁：
    
    和`SELECT ... LOCK IN SHARE MODE`语句类似，只不过需要将上边提到的`S型next-key锁`替换成`X型next-key锁`。
    
*   使用`UPDATE ...`来为记录加锁：
    
    如果`UPDATE`语句未更新二级索引列，比方说：
    
    ```
    UPDATE hero SET country = '汉' WHERE number >= 8;
    ```
    
    这条`UPDATE`语句并没有更新二级索引列，加锁方式和上边所说的`SELECT ... FOR UPDATE`语句一致。
    
    如果`UPDATE`语句中更新了二级索引列，比方说：
    
    ```
    UPDATE hero SET name = 'cao曹操' WHERE number >= 8;
    ```
    
    对聚簇索引记录加锁的情况和`SELECT ... FOR UPDATE`语句一致，也就是对`number`值为`8`的聚簇索引记录加`X型正经记录锁`，对`number`值`15`、`20`的聚簇索引记录以及`Supremum`记录加`X型next-key锁`。但是因为也要更新二级索引`idx_name`，所以也会对`number`值为`8`、`15`、`20`的聚簇索引记录对应的`idx_name`二级索引记录加`X型正经记录锁`，画个图表示一下：
    
    ![](https://mmbiz.qpic.cn/mmbiz/RLmbWWew55Ea7XI8DjgG6TqefjLcfkqRqLPpVIRPEubOww9PrJKrwHgjcQnUlkMESxNNibxQiauhzJdXR3P7Ozjg/640?wx_fmt=other)
    
    如果是下边这个语句：
    
    ```
    UPDATE hero SET name = 'cao曹操' WHERE number <= 8;
    ```
    
    则会对`number`值为`1`、`3`、`8`、`15`的聚簇索引记录加`X型next-key`，其中`number`值为`15`的聚簇索引记录不满足`number <= 8`的边界条件，虽然在`REPEATABLE READ`隔离级别下不会将它的锁释放掉<font style = "color:red">(在mysql8.17版本中是会释放掉记录15上的锁的,[[超全面Mysql语句加锁分析#锁定读的语句#REPEATABLE READ 隔离级别下#对于使用主键进行范围查询的情况|原理同上]])</font>，但是也并不会对这条聚簇索引记录对应的二级索引记录加锁，也就是说只会为`number`值为`1`、`3`、`8`的聚簇索引记录对应的`idx_name`二级索引记录加`X型正经记录锁`，加锁示意图如下所示：
    
    ![](https://mmbiz.qpic.cn/mmbiz/RLmbWWew55Ea7XI8DjgG6TqefjLcfkqRjFrPQSXFIjCSibKQia0wmibyM4QiaQ93JzCxa6dHffKKibElgS1icfsL6HmQ/640?wx_fmt=other)
*   使用`DELETE ...`来为记录加锁，比方说：
    
    ```
    DELETE FROM hero WHERE number >= 8;
    ```
    
    和
    
    ```
    DELETE FROM hero WHERE number <= 8;
    ```
    
    这两个语句的加锁情况和更新带有二级索引列的`UPDATE`语句一致，就不画图了。
    

#### 对于使用唯一二级索引进行等值查询的情况

由于`hero`表并没有唯一二级索引，我们把原先的`idx_name`修改为一个唯一二级索引`uk_name`：

```
ALTER TABLE hero DROP INDEX idx_name, ADD UNIQUE KEY uk_name (name);
```

*   使用`SELECT ... LOCK IN SHARE MODE`语句来为记录加锁，比方说：
    
    ```
    SELECT * FROM hero WHERE name = 'c曹操' LOCK IN SHARE MODE;
    ```
    
    由于唯一二级索引具有唯一性，如果在一个事务中第一次执行上述语句时将得到一条记录，第二次执行上述语句前肯定不会有别的事务插入多条`name`值为`'c曹操'`的记录（二级索引具有唯一性），也就是说一个事务中两次执行上述语句并不会发生幻读，这种情况下和`READ UNCOMMITTED／READ COMMITTED`隔离级别下一样，我们只需要为这条`name`值为`'c曹操'`的二级索引记录加一个`S型正经记录锁`，然后再为它对应的聚簇索引记录加一个`S型正经记录锁`就好了，我们画个图看看：
    
    ![](https://mmbiz.qpic.cn/mmbiz/RLmbWWew55Ea7XI8DjgG6TqefjLcfkqRvEHLJwA6I6K40KYJBlEPGFJ1m8LiaP7odLIibM75ZxJqWzcavTGvAfcw/640?wx_fmt=other)
    
    注意加锁顺序，是先对二级索引记录加锁，再对聚簇索引加锁。
    
    如果对唯一二级索引列进行等值查询的记录并不存在，比如：
    
    ```
    SELECT * FROM hero WHERE name = 'g关羽' LOCK IN SHARE MODE;
    ```
    
    为了禁止幻读，所以需要保证别的事务不能再插入`name`值为`'g关羽'`的新记录。在唯一二级索引`uk_name`中，键值比`'g关羽'`大的第一条记录的键值为`l刘备`，所以需要在这条二级索引记录上加一个`gap锁`，如图所示：
    
    ![](https://mmbiz.qpic.cn/mmbiz/RLmbWWew55Ea7XI8DjgG6TqefjLcfkqRRGDglDlcEOcWQ6PMsCEB4wCck2RGXU5JeFXWUNC5M64FwtIQNffGWQ/640?wx_fmt=other)
    
    注意，这里只对二级索引记录进行加锁，并不会对聚簇索引记录进行加锁。
    
*   使用`SELECT ... FOR UPDATE`语句来为记录加锁，比如：
    
    和`SELECT ... LOCK IN SHARE MODE`语句类似，只不过加的是`X型正经记录锁`。
    
*   使用`UPDATE ...`来为记录加锁，比方说：
    
    与`SELECT ... FOR UPDATE`的加锁情况类似，不过如果被更新的列中还有别的二级索引列的话，这些对应的二级索引记录也会被加`X型正经记录锁`。
    
*   使用`DELETE ...`来为记录加锁，比方说：
    
    与`SELECT ... FOR UPDATE`的加锁情况类似，不过如果表中还有别的二级索引列的话，这些对应的二级索引记录也会被加`X型正经记录锁`。
    

#### 对于使用唯一二级索引进行范围查询的情况

*   使用`SELECT ... LOCK IN SHARE MODE`语句来为记录加锁，比方说：
    
    ```sql
    SELECT * FROM hero FORCE INDEX(uk_name) WHERE name >= 'c曹操' LOCK IN SHARE MODE;
    ```
    
    这个语句的执行过程其实是先到二级索引中定位到满足`name >= 'c曹操'`的第一条记录，也就是`name`值为`c曹操`的记录，然后就可以沿着由记录组成的单向链表一路向后找。从二级索引`idx_name`的示意图中可以看出，所有的用户记录都满足`name >= 'c曹操'`的这个条件，所以所有的二级索引记录都会被加`S型next-key锁`，它们对应的聚簇索引记录也会被加`S型正经记录锁`，二级索引的最后一条`Supremum`记录也会被加`S型next-key锁`。不过需要注意一下加锁顺序，对一条二级索引记录加锁完后，会接着对它响应的聚簇索引记录加锁，完后才会对下一条二级索引记录进行加锁，以此类推～ 画个图表示一下就是这样：
    
    ![](https://mmbiz.qpic.cn/mmbiz/RLmbWWew55Ea7XI8DjgG6TqefjLcfkqRmR3WicDLsEnX8SDMdJGseZtg8yw5qwTo2v24gMnRTNtWAtgO0Cd0BOA/640?wx_fmt=other)
    
    稍等一下，不是说`uk_name`是唯一二级索引么？唯一二级索引本身就能保证其自身的值是唯一的，那为啥还要给`name`值为`'c曹操'`的记录加上`S型next-key锁`，而不是`S型正经记录锁`呢？其实我也不知道，按理说只需要给这条二级索引记录加`S型正经记录锁`就好了，我也没想明白设计`InnoDB`的大叔是怎么想的，有知道的小伙伴赶紧添加我微信：`xiaohaizi4919`联系我哈（聊八卦的同学请勿添加）～
    
    再来看下边这个语句：
    
    ```
    SELECT * FROM hero WHERE name <= 'c曹操' LOCK IN SHARE MODE;
    ```
    
    这个语句先会为`name`值为`'c曹操'`的二级索引记录加`S型next-key锁`以及它对应的聚簇索引记录加`S型正经记录锁`。然后还要给`name`值为`'l刘备'`的二级索引记录加`S型next-key锁`，`name`值为`'l刘备'`的二级索引记录不满足索引条件下推的`name <= 'c曹操'`条件，压根儿不会释放掉该记录的锁就直接报告`server层`查询完毕了。这样可以禁止其他事务插入`name`值在`('c曹操', 'l刘备')`之间的新记录，从而防止幻读产生。所以这个过程的加锁示意图如下：
    
    ![](https://mmbiz.qpic.cn/mmbiz/RLmbWWew55Ea7XI8DjgG6TqefjLcfkqRMcdftqYqLxiaa0zn3LVSTz2M6uVouMwiaoJaZ5N8TuM865PV5fHKv4zg/640?wx_fmt=other)
    
    这里大家要注意一下，设计`InnoDB`的大叔在这里给`name`值为`'l刘备'`的二级索引记录加的是`S型next-key锁`，而不是简单的`gap锁`。
    
*   使用`SELECT ... FOR UPDATE`语句来为记录加锁：
    
    和`SELECT ... LOCK IN SHARE MODE`语句类似，只不过加的是`X型正经记录锁`。
    
*   使用`UPDATE ...`来为记录加锁，比方说：
    
    ```
    UPDATE hero SET country = '汉' WHERE name >= 'c曹操';
    ```
    
    假设该语句执行时使用了`uk_name`二级索引来进行`锁定读`（如果二级索引扫描的记录太多，也可能因为成本过大直接使用全表扫描的方式进行`锁定读`），而这条`UPDATE`语句并没有更新二级索引列，那么它的加锁方式和上边所说的`SELECT ... FOR UPDATE`语句一致。如果有其他二级索引列也被更新，那么也会为这些二级索引记录进行加锁，就不赘述了。不过还需要强调一种情况，比方说：
    
    ```
    UPDATE hero SET country = '汉' WHERE name <= 'c曹操';
    ```
    
    我们前边说的`索引条件下推`这个特性只适用于`SELECT`语句，也就是说`UPDATE`语句中无法使用，无法使用`索引条件下推`这个特性时需要先进行回表操作，那么这个语句就会为`name`值为`'c曹操'`和`'l刘备'`的二级索引记录加`X型next-key锁`，对它们对应的聚簇索引记录进行加`X型正经记录锁`。不过之后在判断边界条件时，虽然`name`值为`'l刘备'`的二级索引记录不符合`name <= 'c曹操'`的边界条件，但是在 REPEATABLE READ 隔离级别下并不会释放该记录上加的锁，整个过程的加锁示意图就是：
    
    ![](https://mmbiz.qpic.cn/mmbiz/RLmbWWew55Ea7XI8DjgG6TqefjLcfkqRrwg6LkAsgm5viaxHtB1Jziavp77ZOuoOEc2JsGmD1IqWIPdib6XI3rCUw/640?wx_fmt=other)
*   使用`DELETE ...`来为记录加锁，比方说：
    
    ```
    DELETE FROM hero WHERE name >= 'c曹操';
    ```
    
    和
    
    ```
    DELETE FROM hero WHERE name <= 'c曹操';
    ```
    
    如果这两个语句采用二级索引来进行`锁定读`，那么它们的加锁情况和更新带有二级索引列的`UPDATE`语句一致，就不画图了。
    

#### 对于使用普通二级索引进行等值查询的情况

我们再把上边的唯一二级索引`uk_name`改回普通二级索引`idx_name`：

```
ALTER TABLE hero DROP INDEX uk_name, ADD INDEX idx_name (name);
```

*   使用`SELECT ... LOCK IN SHARE MODE`语句来为记录加锁，比方说：
    
    ```
    SELECT * FROM hero WHERE name = 'c曹操' LOCK IN SHARE MODE;
    ```
    
    由于普通的二级索引没有唯一性，所以一个事务在执行上述语句之后，要阻止别的事务插入`name`值为`'c曹操'`的新记录，设计`InnoDB`的大叔采用下边的方式对上述语句进行加锁：
    

*   对所有`name`值为`'c曹操'`的二级索引记录加`S型next-key锁`，它们对应的聚簇索引记录加`S型正经就锁`。
    
*   对最后一个`name`值为`'c曹操'`的二级索引记录的下一条二级索引记录加`gap锁`。
    

所以整个加锁示意图就如下所示：

![](https://mmbiz.qpic.cn/mmbiz/RLmbWWew55Ea7XI8DjgG6TqefjLcfkqRlvib7J5esN9ptMoSauqMiaXP0U1zLKbEiadmECRv1cvxccsS0mOfL5x7A/640?wx_fmt=other)

如果对普通二级索引等值查询的值并不存在，比如：

```
SELECT * FROM hero WHERE name = 'g关羽' LOCK IN SHARE MODE;
```

加锁方式和我们上边唠叨过的唯一二级索引的情况是一样的，就不赘述了。

*   使用`SELECT ... FOR UPDATE`语句来为记录加锁，比如：
    
    和`SELECT ... LOCK IN SHARE MODE`语句类似，只不过加的是`X型正经记录锁`。
    
*   使用`UPDATE ...`来为记录加锁，比方说：
    
    与`SELECT ... FOR UPDATE`的加锁情况类似，不过如果被更新的列中还有别的二级索引列的话，这些对应的二级索引记录也会被加锁。
    
*   使用`DELETE ...`来为记录加锁，比方说：
    
    与`SELECT ... FOR UPDATE`的加锁情况类似，不过如果表中还有别的二级索引列的话，这些对应的二级索引记录也会被加锁。
    

#### 对于使用普通二级索引进行范围查询的情况

与唯一二级索引的加锁情况类似，就不多唠叨了哈～

### 全表扫描的情况

比方说：

```
SELECT * FROM hero WHERE country  = '魏' LOCK IN SHARE MODE;
```

由于`country`列上未建索引，所以只能采用全表扫描的方式来执行这条查询语句，存储引擎每读取一条聚簇索引记录，就会为这条记录加锁一个`S型next-key锁`，然后返回给`server层`，如果`server层`判断`country = '魏'`这个条件是否成立，如果成立则将其发送给客户端，否则会向`InnoDB`存储引擎发送释放掉该记录上的锁的消息，不过在 REPEATABLE READ 隔离级别下，InnoDB 存储引擎并不会真正的释放掉锁，所以聚簇索引的全部记录都会被加锁，并且在事务提交前不释放。画个图就像这样：

![](https://mmbiz.qpic.cn/mmbiz/RLmbWWew55Ea7XI8DjgG6TqefjLcfkqR7kHuxq9WaRAfdJibqLNU2enTLZLVg1GW25xSH78XkH3jHDt7IBSWQZA/640?wx_fmt=other)

大家看到了么：全部记录都被加了 next-key 锁！此时别的事务别说想向表中插入啥新记录了，就是对某条记录加`X`锁都不可以，这种情况下会极大影响访问该表的并发事务处理能力，所以如果可能的话，尽可能为表建立合适的索引吧～

使用`SELECT ... FOR UPDATE`进行加锁的情况与上边类似，只不过加的是`X型正经记录锁`，就不赘述了。

对于`UPDATE ...`语句来说，加锁情况与`SELECT ... FOR UPDATE`类似，不过如果被更新的列中还有别的二级索引列的话，这些对应的二级索引记录也会被加`X型正经记录锁`。

和`DELETE ...`的语句来说，加锁情况与`SELECT ... FOR UPDATE`类似，不过如果表中还有别的二级索引列的话，这些对应的二级索引记录也会被加`X型正经记录锁`。

**未完待续**

下一章节继续唠叨 INSERT 语句的加锁情况，敬请期待。如果文中有任何问题，请联系作者：xiaohaizi4919（正经微信，扯犊子的请勿添加）。

关注小青蛙，全都是技术干货哈：  

![](https://mmbiz.qpic.cn/mmbiz_png/RLmbWWew55GM3XXtto9wTRRBb4WCDIUkaHqWq17HDQR2vqq7ov7hXicaMZ0VQQtg55EfbQYsiapbJ4icU23x2c3Dg/640?wx_fmt=png)

**原文链接**

大家可以点击原文链接，查看《MySQL 是怎样运行的：从根儿上理解 MySQL》的完整内容，通俗到爆炸💥的 MySQL 进阶读物，原来学习可以这么有趣（也可以扫下边的二维码直接观看）：

![](https://mmbiz.qpic.cn/mmbiz_png/RLmbWWew55GM3XXtto9wTRRBb4WCDIUkVLtdRQQCPfvR0jvdzibUQA2iaUOjGaUSZvzIIcVgYR1DiaJvFQHIlC6CA/640?wx_fmt=png)