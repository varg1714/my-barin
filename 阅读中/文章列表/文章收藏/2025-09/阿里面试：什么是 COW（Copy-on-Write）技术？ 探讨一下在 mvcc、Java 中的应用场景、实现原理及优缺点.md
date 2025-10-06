---
source: https://mp.weixin.qq.com/s/wiizBKdvN8hBmO_nT_HXCw
create: 2025-09-11 21:13
read: true
knowledge: true
knowledge-date: 2025-09-11
summary: "[[COW（Copy-on-Write，写时复制） 技术]]"
tags:
  - 数据结构
---

FSAC 未来超级架构师

架构师总动员  
实现架构转型，再无中年危机

## 尼恩说在前面

在 40 岁老架构师尼恩的**读者交流群** (50+) 中，最近有小伙伴拿到了一线企业如字节、得物、阿里、滴滴、极兔、有赞、希音、百度、网易、美团、蚂蚁、得物的面试资格，遇到很多很重要的面试题：

什么是 COW（Copy-on-Write）技术？

探讨一下在 mvcc、Java 中的应用场景、实现原理及优缺点？

最近有小伙伴在**面试阿里**，都到了这个的面试题。小伙伴没回答好，支支吾吾的说了几句，面试官不满意，面试挂了。

所以，尼恩给大家做一下系统化、体系化的梳理，帮大家展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer 直提”。

当然，这道面试题，以及参考答案，也会收入咱们的《尼恩 Java 面试宝典 PDF》V175 版本，供后面的小伙伴参考，提升大家的 3 高架构、设计、开发水平。

《尼恩架构笔记》《尼恩高并发三部曲》《尼恩 Java 面试宝典》的 PDF，请到文末公号【技术自由圈】获取

## 一：COW 与高并发问题

“读多写少”  场景，  “读多 “  表示同一行数据每秒被读成百上千次，  “写少 “表示偶尔才改一次。

“读多写少”  场景，如何解决读操作和写操作的高并发问题呢？

传统做法：  写操作加排他锁，加锁后所有其他并发读写全部阻塞，吞吐量骤降。

Copy-on-Write（COW）就是冲着这个痛点来的解决方案。

COW 并非特指某一种技术，而是一种**数据更新思想**，核心逻辑是：

*   **读操作**：直接访问原始数据，全程无锁（或极轻量锁），保证读操作的高并发、低延迟。
    
*   **写操作**：不直接修改原始数据，而是先复制一份原始数据的 “副本”，在副本上完成修改，修改完成后再通过 “原子切换” 将访问入口指向新副本，最后异步回收旧副本。

### 1.1. COW（写时复制） 核心流程为：

*   初始状态多个线程**共享同一份数据副本**
    
*   当需要修改数据时，**复制出新副本**进行修改
    
*   修改完成后，**替换引用**指向新副本
    
*   **读操作直接访问原副本**，不受写干扰

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WOz15oExlBVFwjQqTXK3yMx93dsmcA842cNdYD913FQfHiaFSd4H3iaBNq5vLKOEdd2WMynyWxVibqbg/640?from=appmsg&watermark=1#imgIndex=0)

**cow  设计本质**：读 / 写分离 + 延迟复制，特别适合 **读多写少** 场景。

简言之：cow   执行流程是 **读走老路，写开新路**，通过 “空间换时间” 的方式，彻底解耦读、写操作的资源竞争。

当然，mvcc  cow 流程与经典 cow 流程稍微有不同 ：**写走老路，读开新路**，通过 “空间换时间” 的方式，彻底解耦读、写操作的资源竞争。如论如何 ,  都是通过 “空间换时间” 的方式，彻底解耦读、写操作的资源竞争。

### 1.2. 来一个简单的 cow demo

假设有一张表 `account(id, balance)`，热点行 `id=42`：

<table><thead><tr><td><span><strong><span leaf="">时间点</span></strong></span></td><td><span><strong><span leaf="">线程 A（读）</span></strong></span></td><td><span><strong><span leaf="">线程 B（读）</span></strong></span></td><td><span><strong><span leaf="">线程 C（写）</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">T0</span></span></section></td><td><section><span><span leaf="">想读 balance</span></span></section></td><td><section><span><span leaf="">想读 balance</span></span></section></td><td><section><span><span leaf="">想改 balance</span></span></section></td></tr><tr><td><section><span><span leaf="">传统锁</span></span></section></td><td><section><span><span leaf="">等待 C 写完</span></span></section></td><td><section><span><span leaf="">等待 C 写完</span></span></section></td><td><section><span><span leaf="">加排他锁</span></span></section></td></tr><tr><td><section><span><span leaf="">结果</span></span></section></td><td><section><span><span leaf="">读阻塞，QPS 掉到个位数</span></span></section></td><td><section><span leaf=""><br></span></section></td><td><section><span leaf=""><br></span></section></td></tr></tbody></table>

### 1.3. COW 的解决思路

把 “读不阻塞写” 拆成三步：

**1）共享只读页**

所有读线程，继续指向同一份 “老版本” 数据页，完全无锁。

**2） 写时复制**

写线程 C 想改 id=42 的  balance=1000→900：

*   先把 id=42 那一行所在的页复制一份 → 新页；
    
*   在新页上改 balance=900；
    
*   原页仍保持 balance=1000，读线程 A、B 继续访问。

**3）原子切换指针**

*   复制完成后，用一次原子操作把  “当前页指针”  从老页切到新页。
    
*   后续新进来的读线程看到最新值，老读线程仍读完自己的老页即可。

<table><thead><tr><td><span><strong><span leaf="">时间点</span></strong></span></td><td><span><strong><span leaf="">线程 A（读）</span></strong></span></td><td><span><strong><span leaf="">线程 B（读）</span></strong></span></td><td><span><strong><span leaf="">线程 C（写）</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">T0</span></span></section></td><td><section><span><span leaf="">读老页 1000</span></span></section></td><td><section><span><span leaf="">读老页 1000</span></span></section></td><td><section><span><span leaf="">复制新页</span></span></section></td></tr><tr><td><section><span><span leaf="">T1</span></span></section></td><td><section><span><span leaf="">继续 &nbsp; 读 1000</span></span></section></td><td><section><span><span leaf="">继续 &nbsp; 读 1000</span></span></section></td><td><section><span><span leaf="">改新页→900</span></span></section></td></tr><tr><td><section><span><span leaf="">T2</span></span></section></td><td><section><span><span leaf="">结束</span></span></section></td><td><section><span><span leaf="">结束</span></span></section></td><td><section><span><span leaf="">切换指针</span></span></section></td></tr><tr><td><section><span><span leaf="">T3</span></span></section></td><td><section><span><span leaf="">新读→900</span></span></section></td><td><section><span><span leaf="">新读→900</span></span></section></td><td><section><span><span leaf="">完成</span></span></section></td></tr></tbody></table>

结果：

*   读线程 A、B 全程无阻塞；
    
*   写线程 C 只承担一次 “复制成本”，不会拖垮整体 QPS；
    
*   一致性：老读看到老值，新读看到新值，互不干扰。

COW 把并发问题转化为  “延迟复制” 问题——读线程永远无锁地读旧页，写线程仅在真正修改时承担一次复制成本，从而在读多写少的高并发场景里实现 “读不阻塞写、写不阻塞读”。

尼恩提示 ：上面实例的 cow demo 是一个简单的演示案例，真正  mysql   mvcc  里边  COW 逻辑比上面的  cow  复杂的多。

mvcc 与经典的 cow 不同 ：**写走老路，读开新路**，通过 “空间换时间” 的方式，彻底解耦读、写操作的资源竞争。

mvcc 的 cow 涉及到涉及到 undo-log 版本链， readview 读视图，可以参考网盘里尼恩团队的《MVCC 学习圣经》 pdf

### 1.4、COW 如何解决读多写少的高并发问题？

COW 针对上述痛点，通过 “读写解耦” 和 “数据版本隔离”，从根源上彻底消除读写锁竞争，提升读并发。

COW 的核心优势是**读操作无锁**：

*   所有读请求始终访问 “当前有效” 的原始数据（旧副本），无需加锁，支持无限并发读，延迟极低（接近内存直接访问）。
    
*   写操作仅在 “复制副本” 和 “切换入口” 两个环节需要极短的原子操作（如指针切换），几乎不阻塞读请求；即使写操作耗时较长（如大对象修改），也不会影响正在进行的读请求。

**示例**：Java 中的 `CopyOnWriteArrayList`（替代 `ArrayList` 用于读多写少场景）：

*   读方法（如 `get()`）：直接访问内部数组，无任何锁；
    
*   写方法（如 `add()`）：先复制一份新数组，在新数组中添加元素，再通过 `volatile` 修饰的数组引用 “原子切换” 到新数组；

效果：即使有 1000 个读线程同时读取，1 个写线程修改，读线程也不会阻塞，写线程仅在复制和切换时耗时，整体并发效率极高。

## 二、COW  在 JDK 中的应用

Copy-On-Write（简称 COW）是程序设计中的一种优化策略，核心思路是：共享同一内容时，若需修改，先复制一份内容再修改，最后将引用指向新内容。

从 JDK 1.5 开始，Java 并发包提供了两种基于 COW 机制的容器：`CopyOnWriteArrayList` 和 `CopyOnWriteArraySet`，这类容器在很多并发场景中都能发挥重要作用。

### 2.1  CopyOnWriteArrayList 实现原理

`CopyOnWriteArrayList` 是 COW 思想的典型实现：内部通过一个指向数组的引用 `array` 存储数据，所有修改操作（如 `add`、`set`）都在数组的副本上进行。

当需要修改或新增元素时，不直接操作`array`指向的原数组，而是先复制原数组，在副本中完成修改，最后将`array`指向新副本。这样能确保修改操作不影响读操作，实现读写分离。

这样做的好处是我们可以对 CopyOnWrite 容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WOz15oExlBVFwjQqTXK3yMxt8BoFZQhlVc6pNcxaf6XVzUZ2ZuiblA9JPEpJMfSFE5NyPCML95hGqw/640?from=appmsg&watermark=1#imgIndex=1)

```
public class CopyOnWriteArrayList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable {    / 对所有的修改器（mutator）方法进行保护，访问器（accessor）方法并不需要保护 */    final transient ReentrantLock lock = new ReentrantLock();    / 内部对象数组,通过 getArray/setArray 方法去访问 /    private transient volatile Object[] array;    /获取内部对象数组/    final Object[] getArray() {        return array;    }    /设置内部对象数组    */    final void setArray(Object[] a) {        array = a;    }    //...省略其他代码}
```

**关键设计**：

*   `volatile array`：保证数组引用的内存可见性
    
*   `ReentrantLock`：协调并发写操作
    
*   通过`getArray()`/`setArray()`封装数组访问

### 2.2 无锁读操作实现

```
/* 操作内存的引用/private transient volatile Object[] array;public E get(int index) {    return get(getArray(), index);} //获取元素@SuppressWarnings("unchecked")private E get(Object[] a, int index) {    return (E) a[index];}//返回操作内存final Object[] getArray() {    return array;}
```

**优势**：读取完全不加锁，多个线程可以并行读，性能接近 ArrayList

### 2.3 写时复制基本流程

通过 add() 方法，看下 COW 的基本流程

```
public boolean add(E e) {    // 获取共享锁实例    final ReentrantLock lock = this.lock;    // 加锁保证写操作的排他性    lock.lock();    try {        // 获取当前数组快照        Object[] elements = getArray();        int len = elements.length;        // 创建新数组（长度+1）        Object[] newElements = Arrays.copyOf(elements, len + 1);        // 新增元素到末尾        newElements[len] = e;        // 原子切换数组引用        setArray(newElements);        return true;    } finally {        // 确保锁释放        lock.unlock();    }}
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WOz15oExlBVFwjQqTXK3yMxUWvKMyRYUonfzuPP1TAVzJ1ytHTtqBXpQVlouxzmFgd3W6sXOloOSQ/640?from=appmsg&watermark=1#imgIndex=2)

**关键步骤说明**：

**(1) 写操作需要先获取独占锁**

**(2) 复制当前数组创建新副本**

**(3) 在新副本上执行修改**

**(4) 原子更新底层数组引用**

**(5) 保证锁最终释放（finally 块）**

### 2.4 迭代器工作原理

```
static final class COWIterator<E> implements ListIterator<E> {    // 迭代器创建时的数组快照    private final Object[] snapshot;    private int cursor;    COWIterator(Object[] elements, int initialCursor) {        cursor = initialCursor;        snapshot = elements; // 绑定创建时的数组引用    }    public E next() {        if (!hasNext()) throw new NoSuchElementException();        return (E) snapshot[cursor++]; // 从快照中读取    }    // 禁止修改操作（抛出异常）    public void remove() {        throw new UnsupportedOperationException();    }    public void set(E e) {        throw new UnsupportedOperationException();    }}
```

**迭代器特性**：

**(1) 基于创建时的数据快照**

**(2) 不反映后续修改**

**(3) 禁止修改操作（保证快照一致性）**

**(4) 无并发控制开销**

## 三、COW 在 MySQL  MVCC 机制中是如何应用的？

MySQL 的 InnoDB 存储引擎使用 **多版本并发控制（MVCC）** 来实现高并发，而 MVCC 的实现核心正是 **Copy-on-Write 思想**。

### 3.1. MVCC 简介

MySQL  MVCC 通过 undo-log 版本链，为每一行数据维护多个版本来实现并发控制。

不同的事务在特定时刻看到的是数据的不同版本（快照），而不是直接操作同一份  B + 树上的实际的数据页。

这使得：

*   **读操作（SELECT）** 不会阻塞**写操作（UPDATE, DELETE）**。
    
*   **写操作** 不会阻塞 **读操作（SELECT）**。

当然，mvcc  cow 流程与经典 cow 流程稍微有不同 ：**写走老路，读开新路**，通过 “空间换时间” 的方式，彻底解耦读、写操作的资源竞争。

如论如何 ,  都是通过 “空间换时间” 的方式，彻底解耦读、写操作的资源竞争。

### 3.2. COW 在 MVCC 中的具体实现

InnoDB 通过以下方式实现 COW：

#### a. 隐藏列

InnoDB 为每行记录添加了两个（或三个）隐藏的系统字段：

*   `DB_TRX_ID`（6 字节）：**事务 ID**。记录最后一次插入或更新该行的事务 ID。
    
*   `DB_ROLL_PTR`（7 字节）：**回滚指针**。**这个指针是实现 COW 和 MVCC 的关键**。它指向该行数据的前一个版本在 `Undo Log`中的位置。
    
*   `DB_ROW_ID`（6 字节）：行 ID（如果表没有主键，InnoDB 会自动生成一个聚簇索引基于此列）。

#### b. 更新操作的过程（COW 的体现）

当一个事务（假设事务 ID 为 `100`）想要 `UPDATE`某一行时，InnoDB 并不会直接覆盖那行数据。

其过程如下：

**1. Copy（复制）**： 

 - InnoDB 会先将该行数据的**当前版本**（包括所有列的数据）复制到 **Undo Log** 中。这个副本就是该行数据的一个 “旧版本”。 
- 这个旧版本通过一个**回滚指针**（`DB_ROLL_PTR`）链式地连接起来，最新的版本在表空间中，旧的版本都在 Undo Log 里。

**2. Write（写入）**： 

 - 现在，InnoDB 才在**数据库表空间的实际数据页上**修改这行数据。 
- 更新这行数据的 `DB_TRX_ID`字段为当前事务 ID `100`。 
- 更新这行数据的 `DB_ROLL_PTR`字段，使其指向刚刚在 Undo Log 中创建的**旧版本数据**。

**这个过程完美体现了 COW：**

**写时复制（Copy-on-Write）**：只有在进行**写操作（UPDATE）** 时，才**复制（Copy）** 一个旧版本到 Undo Log，然后才**写入（Write）** 新数据。

不过，mvcc  cow 流程与经典 cow 流程稍微有不同 ：**写走老路 (实际的数据页)，读开新路 （ 复制到 Undo Log 副本，可能是多个副本）**，也通过 “空间换时间” 的方式，彻底解耦读、写操作的资源竞争。

#### c. 读操作的过程（MVCC 的体现）

当一个事务（假设是只读事务）执行 `SELECT`语句时：

**(1) InnoDB 会为该事务提供一个一致性读视图（Read View）。这个视图决定了当前事务能看到哪些版本的数据。**

**(2) 事务会从数据页中的最新行数据开始，查看其 `DB_TRX_ID`。**

**(3) 如果该 `DB_TRX_ID`表示它是由一个在当前事务开始之后才启动的、或者尚未提交的事务修改的（即，该版本对当前事务不可见），那么事务就会顺着回滚指针（DB_ROLL_PTR） 找到 Undo Log 中的旧版本。**

**(4) 它会重复这个检查过程，直到找到一个对其可见的数据版本（即，该版本是由已提交的、且在当前事务开始之前就存在的事务修改的）为止。**

### 3.3. 快照读与当前读

#### 3.3.1  快照读

快照读是 **基于 MVCC 实现的一致性非锁定读**。

**哪些操作是快照读？** 最典型的就是**不加锁的普通 SELECT 语句**。

```
SELECT * FROM table_name WHERE ...;
```

**特点：**

*   读取的是历史版本，可能不是最新的数据。
    
*   读写不冲突，性能高。

**快照读，读的是什么？**

它读取的不是数据的最新物理状态，而是**记录在某个时间点的快照版本**（通常是事务开始时的版本）。这个快照数据来源于 **Undo Log**。

**快照读的执行流程？**

当一个事务执行快照读时，InnoDB 会为其生成一个 **Read View**（读视图）。这个 Read View 决定了这个事务能看到哪些版本的数据：

*   在 `READ COMMITTED`隔离级别下，每次快照读都会生成一个新的 Read View。
    
*   在 `REPEATABLE READ`隔离级别下，只有在第一次快照读时生成一个 Read View，后续整个事务都使用这个相同的 Read View，从而保证可重复读。

**快照读的锁行为：** 不加锁 （因此是 “非锁定读”）。它通过多版本机制来避免读取被其他事务锁定的数据，从而实现了读写不阻塞，大大提高了并发性。

#### 3.3.2  当前读

当前读是 **锁定读**，它读取的是数据的最新提交版本。

*   **它读的是什么？**

它总是读取**记录的最新版本**，并且会保证在读取后，其他并发事务不能修改这条记录，直到当前事务结束。

*   **如何实现的？**

通过**加锁**来实现。当前读在读取数据时，会为读取的数据加上锁，以防止其他事务修改它。

### 3.4. COW  与 MVCC  关系

<table><thead><tr><td><span><strong><span leaf="">特性</span></strong></span></td><td><span><strong><span leaf="">Copy-on-Write (COW)</span></strong></span></td><td><span><strong><span leaf="">MySQL InnoDB MVCC</span></strong></span></td></tr></thead><tbody><tr><td><section><span><strong><span leaf="">核心思想</span></strong></span></section></td><td><section><span><span leaf="">写时复制副本，读时共享原数据。</span><strong><span leaf="">读旧版本，写新版本</span></strong></span></section></td><td><section><span><span leaf="">写时创建旧版本副本，读时根据需要选择版本。&nbsp;</span><strong><span leaf="">写旧版本，读可看见的新版本</span></strong></span></section></td></tr><tr><td><section><span><strong><span leaf="">“复制” 发生时机</span></strong></span></section></td><td><section><span><span leaf="">写操作发生时</span></span></section></td><td><section><span><strong><span leaf="">UPDATE/DELETE</span></strong><span leaf="">&nbsp; 操作发生时</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">“复制” 的位置</span></strong></span></section></td><td><section><span><span leaf="">内存或磁盘的某个副本空间</span></span></section></td><td><section><span><strong><span leaf="">Undo Log</span></strong><span leaf="">（回滚段）</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">“原数据”</span></strong></span></section></td><td><section><span><span leaf="">原数据 = &nbsp; 原始数据页</span></span></section></td><td><section><span><span leaf="">原数据 = &nbsp; 数据页中的最新版本</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">“副本”</span></strong></span></section></td><td><section><span><span leaf="">副本 = 数据的拷贝</span></span></section></td><td><section><span><span leaf="">副本 = &nbsp; 旧版本 &nbsp; 的</span><strong><span leaf="">副本</span></strong><span leaf="">，存储在 Undo Log 中</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">目的</span></strong></span></section></td><td><section><span><span leaf="">优化性能，减少不必要的拷贝，保证读一致性</span></span></section></td><td><section><span><span leaf="">实现高并发，避免读写冲突，提供一致性非锁定读</span></span></section></td></tr></tbody></table>

MySQL InnoDB 的 MVCC 机制是 **Copy-on-Write 思想在数据库领域的经典实现**。

MVCC   通过在执行写操作（UPDATE/DELETE）时，将旧数据副本写入 Undo Log（**Copy**），然后修改当前数据（**Write**），从而为每个事务提供了一致性的数据视图，极大地提升了数据库的并发性能。

mvcc 的 cow 涉及到涉及到 undo-log 版本链， readview 读视图，可以参考网盘里尼恩团队的《MVCC 学习圣经》 pdf

## 四、COW  并非 “银弹” ( COW   局限性) ：

COW 虽适配读多写少场景，但也存在明显短板，需根据业务场景权衡：

**1. 写操作成本高**：

复制瞬间需要两份页，写完旧页即可回收，写的时候，可能 **内存瞬时翻倍**。

每次写操作都需复制数据，如果 copy 数据量大（如 Java 的 `CopyOnWriteArrayList` 若存储 10 万条数据，一次 `add()` 需复制 10 万条数据到新数组），写频繁场景（如每秒 1000 次写）会导致 CPU、内存开销剧增，反而性能下降。

**2. 内存占用高**：

写操作过程中，旧副本和新副本会同时存在（直到旧副本被回收），若数据量极大（如 GB 级对象），可能引发内存溢出（OOM）。

**3. 读操作存在 “延迟一致性”**：

读操作看到的是 “旧版本” 数据，直到写操作完成并切换副本后，才能读到新数据。

若业务要求 “实时一致性”（如金融交易金额查询），COW 不适用。

**4. COW   不适合写多 + 读少场景**：

写多 + 读少场景，如果每毫秒都在改同一行，复制开销反而成为瓶颈，不适合使用 cow。

那么，写多读少场景，如何提高高并发呢，这个可以找尼恩来探讨，交流。

## 说在最后：有问题找老架构取经‍

只要按照上面的尼恩团队梳理的方案去作答，你的答案不是 100 分，而是 120 分。面试官一定是心满意足，五体投地。

按照尼恩的梳理，进行深度回答，可以充分展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer 直提”。

在面试之前，建议大家系统化的刷一波 5000 页《[尼恩 Java 面试宝典 PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》，里边有大量的大厂真题、面试难题、架构难题。

很多小伙伴刷完后，吊打面试官，大厂横着走。

在刷题过程中，如果有啥问题，大家可以来找 40 岁老架构师尼恩交流。

另外，如果没有面试机会，可以找尼恩来改简历、做帮扶。前段时间，**[刚指导一个小伙 暴涨 200%（涨 2 倍），29 岁 / 7 年 / 双非一本 ， 从 13K 一次涨到 37K ，逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486433&idx=1&sn=0f630e8fc7620b3bf0e054827b48d91e&scene=21#wechat_redirect)。**

狠狠卷，实现 “offer 自由” 很容易的，前段时间一个武汉的跟着尼恩卷了 2 年的小伙伴，在极度严寒 / 痛苦被裁的环境下， offer 拿到手软，实现真正的 “offer 自由” 。

## 惊天大逆袭： 通过  Java+AI  实现弯道超车，完成转架构 

 [会 AI 的程序员，工资暴涨 50%！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486827&idx=1&sn=db3f268c6752ddb074a98c7b13a84011&scene=21#wechat_redirect)

[极速上岸： 被裁 后， 8 天 拿下 京东，狠涨 一倍 年薪 48W， 小伙伴 就是 做对了一件事](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486913&idx=1&sn=edd6774e9d17c39327e8dcb95fdd68d9&scene=21#wechat_redirect)

[外包 + 二本 可以进 美团： 26 岁小 2 本 一步登天， 进了顶奢大厂（ 美团） ， 太爽了](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486904&idx=1&sn=e7c878f99ed101ebfbd1d80ec0e07401&scene=21#wechat_redirect)

[暴涨 150%，4 年 CRUD 一步登天， 进  ‘宇宙厂’， 26 岁 小伙 6 个月 大逆袭](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486898&idx=1&sn=c1f7b50eb13ad990a19f099d9e608231&scene=21#wechat_redirect)

[Java+Al 大逆袭 1： 34 岁无路可走，一个月翻盘，拿 3 个架构 offer，靠  Java+Al   逆天改命！！！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486848&idx=1&sn=ff43d058271b0801f84aba3f3d957855&scene=21#wechat_redirect)

java+AI 大逆袭 2：[：3 年 程序媛 被裁， 25W-》40W 上岸， 逆涨 60%。 Java+AI 太神了， 架构小白 2 个月逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486858&idx=1&sn=9cd122c18d166433b43d0952d3a7a7a8&scene=21#wechat_redirect)

[Java+AI 逆袭 ： 36 岁 / 失业 7 个月 / 彻底绝望 。狠卷 3 个月 Java+AI ，终于逆风翻盘，顺利 上岸](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486870&idx=1&sn=990a540a155cdb8254ebce6c75816882&scene=21#wechat_redirect)

[Java+AI 逆袭 ： 闲了一年，41 岁 / 失业 12 个月 / 彻底绝望 。狠卷 2 个月 Java+AI ，终于逆风翻盘](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486880&idx=1&sn=a37033157c766ac5ae7c9195fe776693&scene=21#wechat_redirect)

## 冲大厂案例：  全网顶尖、高薪案例，进大厂拿高薪，实现薪酬腾飞、人生逆袭 

[涨一倍：从 30 万 涨 60 万，3 年经验小伙 冲大厂成功，逆天了 ！！！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486818&idx=1&sn=ec5ff9c809133d903bdecddcd25c440e&scene=21#wechat_redirect)

[阿里 + 美团 offer：25 岁 屡战屡败 绝望至极。找尼恩转架构升级，1 个月拿到阿里 + 美团 offer，逆天改命年薪 50W](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486794&idx=1&sn=fc1a1f843b0a91b61f7124ac0c2b51e2&scene=21#wechat_redirect)

[阿里 offer：6 年一本 不想 混小厂了。狠卷 1 年  拿到 得物 + 阿里 offer ， 彻底上岸 ，逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486782&idx=1&sn=52ed05742c8afe7abf3b50405e78c853&scene=21#wechat_redirect)

## 大龄逆袭的案例： 大龄被裁，快速上岸的，远离没有 offer 的焦虑、恐慌 

[47 岁超级大龄，被裁员后 找尼恩辅导收  2 个 offer，一个 40 多 W。 35 岁之后，只要 技术好，还是有饭吃，关键是找对方向，找对路子](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486323&idx=1&sn=a088584f97b657add9a058c149b72067&scene=21#wechat_redirect)

[大龄不难：39 岁 / 15 年老码农，15 天时间 40W 上岸，管一个 team，不用去 铁人三项了！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486310&idx=1&sn=505e479da54cf1365afd4d39bddc18bd&scene=21#wechat_redirect)

## 草根逆袭， 100W 年薪天花板案例。他们如何实现薪酬腾飞、人生逆袭？ 

[专科生 100 年薪 ：35 岁专科 草根逆袭，2 线城市年薪 100W 逆天改命， 从 超低起点 塔基（8W）--》塔腰 -》塔尖（100W）](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486860&idx=1&sn=940405ff775e014bac52ba0636ea69e7&scene=21#wechat_redirect)

## [年薪 100W 的底层逻辑：](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486315&idx=1&sn=52cc0a640c1499115f5a45b82097d857&chksm=97b57ae3a0c2f3f502293e84cde4bc892c45d06dee3579471a522ab497fb53e32640fb00c339&scene=21#wechat_redirect) [大厂被裁，他们两个，如何实现年薪百万？](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486315&idx=1&sn=52cc0a640c1499115f5a45b82097d857&chksm=97b57ae3a0c2f3f502293e84cde4bc892c45d06dee3579471a522ab497fb53e32640fb00c339&scene=21#wechat_redirect) 

**[年薪 100W](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)****[：](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)**[40](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)[岁小伙，被裁 6 个月，猛卷 3 月，100W 逆袭 ，秘诀：升级首席架构 / 总架构](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)

[最新的 100W 案例：环境太糟，如何升 P8 级，年入 100W？](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486369&idx=1&sn=0b32e8907dbcf26eaf33ea3f37ddd099&chksm=97b57a29a0c2f33f468d29dafe739204543873efb459f324871c3c68a3ecc178b1a9ab08a2fa&scene=21#wechat_redirect)

**职业救助站**

实现职业转型，极速上岸

![](https://mmbiz.qpic.cn/mmbiz_jpg/9qY2Gt3PBs3icE6WcM4nBOdRRcczOW4wkNDgCMZhBfqeet9suwGHfgQduP33YYqqZLOppecMI72Pv5ibNkPeIk6g/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=3)

关注**职业救助站**公众号，获取每天职业干货  
助您实现**职业转型、职业升级、极速上岸**  
---------------------------------

**技术自由圈**

实现架构转型，再无中年危机

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/xlgvgPaib7WMtyFcJx45oIeWL1d6AENcdoAvuTPuq46E9n8pPe4SHXoSicuEVvhibHRSIzmEBPlC7POcDaqD6ID7w/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=4)

关注**技术自由圈**公众号，获取每天技术千货  
一起成为牛逼的**未来超级架构师**

**几十篇架构笔记、5000 页面试宝典、20 个技术圣经  
请加尼恩个人微信免费拿走**

**暗号，请在公众号后台发送消息：领电子书**

如有收获，请点击底部的 " 在看 "和" 赞 "，谢谢