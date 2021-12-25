#数据库
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码，原文地址 [my.oschina.net](https://my.oschina.net/u/3677838/blog/4822404)

编辑：业余草

来源：https://www.xttblog.com/?p=4911

**前言**

  

---

此篇博客主要是讲述 MySql (仅限 Innodb) 的两阶段加锁 (2PL) 协议, 而非两阶段提交 ([[分布式/二阶段协议|2PC]]) 协议, 区别如下:

*   **2PL 两阶段加锁协议**: 主要用于单机事务中的一致性与隔离性。整个事务分为两个阶段，前一个阶段为加锁，后一个阶段为解锁。在加锁阶段，事务只能加锁，也可以操作数据，但不能解锁，直到事务释放第一个锁，就进入解锁阶段，此过程中事务只能解锁，也可以操作数据，不能再加锁。两阶段锁协议使得事务具有较高的并发度，因为解锁不必发生在事务结尾。它的不足是没有解决死锁的问题，因为它在加锁阶段没有顺序要求。如两个事务分别申请了 A, B 锁，接着又申请对方的锁，此时进入死锁状态
    
*   **2PC**, 两阶段提交协议: 主要用于分布式事务。
    

MySql 本身针对性能，还有一个 MVCC (多版本控制) 控制, 本文不考虑此种技术，仅仅考虑 MySql 本身的加锁协议。

**什么时候会加锁**

  

---

在对记录更新操作或者 (select for update、lock in share model) 时，会对记录加锁 (有共享锁、排它锁、意向锁、gap 锁、nextkey 锁等等), 本文为了简单考虑，不考虑锁的种类。

**什么是两阶段加锁**

  

---

在一个事务里面，分为加锁 (lock) 阶段和解锁 (unlock) 阶段, 也即所有的 lock 操作都在 unlock 操作之前, 如下图所示:  
![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210406215717.png)

**为什么需要两阶段加锁**

引入 2PL 是为了保证事务的隔离性，即<font color="orange">多个事务在并发的情况下等同于串行的执行</font>。在数学上证明了如下的封锁定理:

**如果事务是良构的且是两阶段的，那么任何一个合法的调度都是隔离的。**

两阶段加锁协议事务 `可串形化调度` 的充分条件，满足两阶段加锁协议的事务可以被串形化调度，即做到事务的隔离。

具体的数学推到过程可以参照 <<事务处理: 概念与技术>> 这本书的 7.5.8.2 节.  
此书乃是关于数据库事务的圣经，无需解释 (中文翻译虽然晦涩，也能坚持读下去, 强烈推荐)

**工程实践中的两阶段加锁 - S2PL**

  

---

在实际情况下，SQL 是千变万化、条数不定的, 数据库很难在事务中判定什么是加锁阶段，什么是解锁阶段。于是引入了 S2PL (Strict-2PL), 即:

**在事务中只有提交 (commit) 或者回滚 (rollback) 时才是解锁阶段，其余时间为加锁阶段。**

如下图所示:  
![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210406215726.png)  
这样的话，在实际的数据库中就很容易实现了。

**两阶段加锁对性能的影响**

  

---

上面很好的解释了两阶段加锁，现在我们分析下其对性能的影响。考虑下面两种不同的扣减库存的方案:

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210406215741.png)

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210406215758.png)

由于在同一个事务之内，这几条对数据库的操作应该是等价的。但在两阶段加锁下的性能确是有比较大的差距。两者方案的时序如下图所示:  
![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210406215814.png)

**由于库存往往是最重要的热点，是整个系统的瓶颈。那么如果采用第二种方案的话, tps 应该理论上能够提升 3rt/rt=3 倍。这还仅仅是业务就只有三条 SQL 的情况下，多一条 sql 就多一次 rt, 就多一倍的时间。**

值得注意的是：

**在更新到数据库的那个时间点才算锁成功，提交到数据库的时候才算解锁成功，这两个 round_trip 的前半段是不会计算在内的。**

如下图所示:  
![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210406215821.png)  
当前只考虑网络时延，不考虑数据库和应用本身的时间消耗。

**依据 S2PL 的性能优化**

**从上面的例子中, 可以看出，需要把最热点的记录，放到事务最后，这样可以显著的提高吞吐量。更进一步: 越热点记录离事务的终点越近 (无论是 commit 还是 rollback)**

笔者认为，先后顺序如下图:

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210406215847.png)

 

**避免死锁**

这也是任何 SQL 加锁不可避免的。上文提到了按照记录 Key 的热度在事务中倒序排列。那么写代码的时候任何可能并发的 SQL 都必须按照这种顺序来处理，不然会造成死锁。如下图所示: ![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210406215854.png)

 

**select for update 和 update where 谓词计算**

我们可以直接将一些简单的判断逻辑写到 update 的谓词里面，以减少加锁时间，考虑下面两种方案:  
**方案 1:**

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210406215906.png)  

**方案 2:**

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210406215912.png)

如图所示:

 ![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20210406215926.png)  
可以看到，通过在 update 中加谓词计算，少了 1rt 的时间。

**由于 update 在执行过程中对符合谓词条件的记录加的是和 select for update 一致的排它锁 (具体的锁类型较为复杂，不在这里描述), 所以两者效果一样。**

**总结**

MySql 采用两阶段加锁协议实现隔离性和一致性，我们只有深入的去理解这种协议，才能更好的对我们的 SQL 进行优化，增加系统的吞吐量。

![](https://oscimg.oschina.net/oscnet/167a74a2-ca57-45ee-9c87-a0ebfaceca26.jpg)

本文分享自微信公众号 - 业余草（yyucao）。  
如有侵权，请联系 support@oschina. Cn 删除。  
本文参与 “[OSC 源创计划](https://www.oschina.net/sharing-plan)”，欢迎正在阅读的你也加入，一起分享。