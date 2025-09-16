---
source: https://mp.weixin.qq.com/s/SaaHKPnNUSvDkmwKtip3HA
create: 2024-01-15 15:41
read: true
knowledge: true
knowledge-date: 2025-09-16
tags:
  - 计算机原理
summary: "[[阅读中/阅读总结/文章收藏/CPU 是如何与内存交互的|CPU 是如何与内存交互的]]"
---

# CPU 是如何与内存交互的

![](https://mmbiz.qpic.cn/mmbiz_gif/j3gficicyOvasIjZpiaTNIPReJVWEJf7UGpmokI3LL4NbQDb8fO48fYROmYPXUhXFN8IdDqPcI1gA6OfSLsQHxB4w/640?wx_fmt=gif)

作者：bearluo，腾讯 IEG 运营开发工程师

这篇文章主要整理了一下计算机中的内存结构，以及 CPU 是如何读写内存中的数据的，如何维护 CPU 缓存中的数据一致性。什么是虚拟内存，以及它存在的必要性。如有不对请多多指教。

## 1. 概述

目前在计算机中，主要有两大存储器 SRAM 和 DRAM。主存储器是由 DRAM 实现的，也就是我们常说的内存，在 CPU 里通常会有 L1、L2、L3 这样三层高速缓存是用 SRAM 实现的。

SRAM 被称为 “静态” 存储器，是因为只要处在通电状态，里面的数据就可以保持存在。而一旦断电，里面的数据就会丢失了。

目前 SRAM 主要集成在 CPU 里面，每个 CPU 核心都有一块属于自己的 L1 高速缓存，通常分成指令缓存和数据缓存，分开存放 CPU 使用的指令和数据。L2 的 Cache 同样是每个 CPU 核心都有的，不过它往往不在 CPU 核心的内部。所以，L2 cache 的访问速度会比 L1 稍微慢一些。而 L3 cache ，则通常是多个 CPU 核心共用的。

在 DRAM 中存储单元使用电容保存电荷的方式来存储数据，电容会不断漏电，所以需要定时刷新充电，才能保持数据不丢失，这也是被称为 “动态” 存储器的原因。由于存储 DRAM 一个 bit 只需要一个晶体管，所以存储的数据也大很多。

我们来看一些他们的速度：

*   • L1 的存取速度：**4 个 CPU 时钟周期**
    
*   • L2 的存取速度：**11 个 CPU 时钟周期**
    
*   • L3 的存取速度：**39 个 CPU 时钟周期**
    
*   • DRAM 内存的存取速度**：107 个 CPU 时钟周期**

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvatnhl3sxFBiblZPd2r6eEjOqqVr3g7jxLJWsCIjymMZE5sfG0JJrvONgGoaIibRicvLhBUP6tP6TxEBA/640?wx_fmt=jpeg)

## 2. CPU cache

### 2.1. cache 结构

上面我们说了，对于 CPU 来说，SRAM 被称为 CPU 的 cache，CPU 每次获取数据都会先访问 cache，如果获取不到数据则把数据加载到 cache 中进行访问。因为 cache 的大小是远远小于主存的，所以还需要在 cache 和主存之间维护一个映射关系，这样才能正确找到数据。

一种简单的方式是**直接映射**，有点像我们把数据找出来，直接放入到 map 中进行存储一样，映射通过地址和 cache 的数量进行取模后获取到 cache 中主存的地址去获取数据。

```
（地址）mod（cache 中的块数）
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvatnhl3sxFBiblZPd2r6eEjOqicD4WXgwib8oegmSzALdpLaPibfP4v2D8U5ibGBCgtqfmQOibQCmlSpxbOg/640?wx_fmt=jpeg)

上图中画了一个地址去找 cache 的情况。对于 cache 来说可以划分为：

索引：用来取模找到对应的 cache 行；

有效位：cache 初始值一开始是空的，这个字段标记 cache 行是否有数据；

标记：用来和地址进行比较是否是对应的数据；

数据：表示存储的实际的数据，可以通过地址的偏移来获取到对应的数据；

比如对于这个 cache 有 1024 个字，即 4KB。假设有一个 32 位的地址要去 cache 中查找数据，那么会取地址 10 位进行取模找到对应的 cache 行，然后取出 20 位与 cahce 标记位进行比较，如果相等，并且有效位开启，那么对应请求地址在 cache 中命中。否则，发生缺失。

由于 CPU 在读取数据的时候，并不是要读取一整个 Block，而是读取一个他需要的数据片段， cache 中命中之后会根据低两位的偏移去数据里面索引到对应的字。

除了上面说的**直接映射**以外还有**组相联**和**全相联**。

**组相联**就是使用组索引代替了原来的索引，下图中表示每组有 2 行数据，通过组索引找到对应的数据行之后通过有效位和标记对组中每一行进行检索，如果能匹配上就说明命中。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvatnhl3sxFBiblZPd2r6eEjOq7cBMWf1W2H7bewv31bzOUBsfES9JpbRF7SeA1kJGQbalSXg8icIYAAg/640?wx_fmt=jpeg)

### 2.2. cache 读写操作

先来看看读操作，cache 在初始状态的时候是空的，这个时候去读数据会产生 **cache 缺失（cache miss）**。cache 控制器会检测到缺失的发生，然后从主存中（或低一级 cache）中取回所需数据。如果命中，那么就会直接使用。

当 cache 缺失时，对于乱序执行处理器而言依然能执行一些其他指令，但是对于顺序执行处理器，当 cache 缺失时会被阻塞，临时寄存器和程序员可见的寄存器中的内容基本被冻结。

再来看看写操作，因为 cache 是由多级组成，所以写策略一般而言有两种：**写直达（write-through）**和**写回（write-back）**。通过这两种策略使在 cache 中写入的数据和主存中的数据保持一致。

**写直达**就是在将数据写入 cache 之后同时将这个数据立马写入到主存中，但是由于主存和 cache 本身性能差异，那么每次在写入主存的时候都将花费大量的时间。解决办法就是加一层写缓冲（write buffer），这样 CPU 在将数据写入 cache 和缓冲之后可以继续执行，等到缓冲写入到主存中再释放。

但是如果写入速度大于缓冲释放速度，那么还是会阻塞 CPU 执行。那么可以考虑一下**写回**策略，这种机制会在每次写入的时候仅将新值写入 cache 中，只有当修改过的块被替换时才需要写到较低层存储结构中。

### 2.3. 一致性与 MESI 协议

由于现在都是多核 CPU，并且 cache 分了多级，并且数据存在共享的情况，所以需要一种机制保证在不同的核中看到的 cache 数据必须时一致的。最常用来处理多核 CPU 之间的缓存一致性协议就是 **MESI 协议**。

**MESI 协议**，是一种叫作写失效（Write Invalidate）的协议。在写失效协议里，只有一个 CPU 核心负责写入数据，其他的核心，只是同步读取到这个写入。在这个 CPU 核心写入 cache 之后，它会去广播一个 “失效” 请求告诉所有其他的 CPU 核心。

MESI 协议对应的四个不同的标记，分别是：

*   • M：代表已修改（Modified）
    
*   • E：代表独占（Exclusive）
    
*   • S：代表共享（Shared）
    
*   • I：代表已失效（Invalidated）

“已修改”和 “已失效” 比较容易理解，我们来看看 独占”和“共享” 两个状态。

在独占状态下，对应的 cache Line 只加载到了当前 CPU 核所拥有的 cache 里。其他的 CPU 核，并没有加载对应的数据到自己的 cache 里。这个时候，如果要向独占的 cache Block 写入数据，我们可以自由地写入数据，而不需要告知其他 CPU 核。

那么对应的，共享状态就是在多核中同时加载了同一份数据。所以在共享状态下想要修改数据要先向所有的其他 CPU 核心广播一个请求，要求先把其他 CPU 核心里面的 cache ，都变成无效的状态，然后再更新当前 cache 里面的数据。

## 3. 虚拟内存

### 3.1. 虚拟内存映射

在我们日常使用的 Linux 或者 Windows 操作系统下，程序并不能直接访问物理内存。程序都是通过虚拟地址 VA（virtual address）用地址转换翻译成 PA 物理地址（physical address）才能获取到数据。也就是说 CPU 操作的实际上是一个虚拟地址 VA。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvatnhl3sxFBiblZPd2r6eEjOqYKNwaapx9mf6U1LSQvic3p9ElxA1UAUBojxqJV6dZAHjyZ3ZyribbZfw/640?wx_fmt=jpeg)

如上图，CPU 访问主存的时候会将一个虚拟地址（virtual address）被内存管理单元（Memory Management Unint, MMU）进行翻译成物理地址 PA（physical address） 才能访问。

想要把虚拟内存地址，映射到物理内存地址，最直观的办法，就是来建一张映射表。这个映射表在计算机中叫页表（Page Table）。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvatnhl3sxFBiblZPd2r6eEjOqPiaTBQxTAul3RaCu1dicyfAibzq79BXHdrnIWEianicQ4Lnic6HyYhhFFptg/640?wx_fmt=jpeg)

在查找页表的时候，会将虚拟地址分成页号（Directory）和偏移量（Offset）两个部分。前面的高位，表示内存地址的页号。后面的低位，表示内存地址里面的偏移量。

查找方式和上面说的组相联类似，首先使用虚拟页号作为索引去页表中找到对应的物理页号，页表中还会有 1 位表示有效位，如果该位无效就不在主存中，发生一次缺页；如果有效，那么就可以拿到对应的物理页号获取到对应的物理页位置，再根据偏移量得到物理内存地址。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvatnhl3sxFBiblZPd2r6eEjOqiczvtrvq3QKVpJEBqicZ51Guma61eANgTjDqwUicxdEg2UK6NtffCFQyA/640?wx_fmt=jpeg)

如果有效位关闭，那么该页就只存在磁盘上的某个指定的磁盘地址。缺页会触发缺页异常，然后在闪存或磁盘中找到该页，将其放入到主存 DRAM 中。

如果主存满了，那么会选择一个牺牲页，大多数操作系统会使用 LRU 替换策略来进行页的替换。操作系统会查找最少使用的页，被替换的页会写入磁盘的交换区（swap 分区）。swap 分区通常被称为交换分区，这是一块特殊的硬盘空间，即当实际内存不够用的时候，操作系统会从内存中取出一部分暂时不用的数据，放在交换分区中，从而为当前运行的程序腾出足够的内存空间。

在下图中假设选择将存放在主存中的 VP6 进行替换，将 VP6 替换为 VP3。如果被替代的 VP6 已经被修改了，那么内核会将它复制回磁盘。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvatnhl3sxFBiblZPd2r6eEjOq7q6BhhQLSIlBfMtfdjbeIT6AP4ASHCIV3eInxwSp1VD8ucdGYG5EAg/640?wx_fmt=jpeg)

由于局部性（locality）的存在，程序一般而言会在一个较小的活动页面集合上工作，页的切换开销只存在于程序启动时将页面调度到内存中，接下来的程序都会页命中。但是如果代码的工作集太大，超过了物理内存大小，那么页面就会不停地换进换出，产生抖动。

### 3.2. 多级页表

假设我们现在是一个 32 位的地址空间、4KB 的页面和一个 4 字节的 PTE，那么需要一个 4MB 的页表常驻在内存中，并且这个页表是每个进程都独占一份，所以会造成很大的内存浪费，我们需要一种方式来优化我们的页表空间存储。

想一下虚拟内存空间结构，整个 4 GB 的空间，操作系统用了 1 GB，从地址 `0XC0000000` 到 `0XFFFFFFFF`, 剩余 3 GB 留给用户空间，其实很多程序根本用不到这么大的空间，对于 64 位系统，每个进程都会拥有 256 TiB 的内存空间，那就更加用不上了。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvatnhl3sxFBiblZPd2r6eEjOqcZlOUwRE1kHeCysoqVlFPZI6fiahhwByld48fzMppPmHes38JnCwV9w/640?wx_fmt=jpeg)

那么对于用不上的空间，我们可以不可以不把它加载到页表里面，等到用这块空间的时候才在页表里面给它分配一个页表项，是不是就可以节省大量空间。

在程序运行的时候，内存地址从顶部往下，不断分配占用的栈的空间。而堆的空间，内存地址则是从底部往上，是不断分配占用的。所以，在一个实际的程序进程里面，虚拟内存占用的地址空间，通常是两段连续的空间。而多级页表，就特别适合这样的内存地址分布。

假设 32 位虚拟地址空间被划分位 4KB 每页，每个条目都是 4 字节，那么我们可以让第一级页表中的每个 PTE （页表项 page table entry）负责映射虚拟地址空间中一个 4MB 的片，这个片由 1024 个连续页面组成，表示二级页表。如果地址空间是 4GB，那么 1024 个一级页表项就可以覆盖整个空间。

如下图所示，内存前 2K 个页面给代码和数据，接下来 6K 个页面未分配，在接下来 1023 个页面也未分配，接下来一个页面分配给用户栈。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvatnhl3sxFBiblZPd2r6eEjOqibHWFSG56jRKeI4Cg7t4mmsTiapc11JwpcRib2K685L1B6pk4wAXfNB3w/640?wx_fmt=jpeg)

这种方法从两个方面减少了内存占用。第一，如果一级页表中的一个 PTE 是空的，那么相应的二级页表就根本不会存在。由于很多程序占用内存实际远小于页表所能表示的大小，所以可以节约很大空间的页表项资源；第二，只有一级页表才需要总是在主存中，二级页表会在需要的时候创建或销毁，只有最经常使用的二级页表才需要缓存在主存中，这就减少了主存的压力。

Linux 在 2.6.10 中引入了四层的页表辅助虚拟地址的转换：

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvatnhl3sxFBiblZPd2r6eEjOq4ILbvFvMfnZv0zGKkmdAKa64fD0b5zqlia8SsfdzJAFKBAQ7Sf0kR1g/640?wx_fmt=jpeg)

首先会找到 4 级页表里面对应的条目（Entry）。这个条目里存放的是一张 3 级页表所在的位置。4 级页面里面的每一个条目，都对应着一张 3 级页表，所以我们可能有多张 3 级页表。

找到对应这张 3 级页表之后，我们用 3 级索引去找到对应的 3 级索引的条目。3 级索引的条目再会指向一个 2 级页表。依次拿到 1 级页表里面存储的物理页号，我们同样可以用 “页号 + 偏移量” 的方式，来获取最终的物理内存地址。

### 3.3. TLB 加速地址转换

对于一个页命中的数据获取过程通常来说，如果没有 TLB 加速是这样的：

1.  CPU 生成一个虚拟地址，并把它传给 MMU；
    
2.  MMU 生成页表项地址 PTEA，并从高速缓存 / 主存请求获取页表项 PTE；
    
3.  高速缓存 / 主存向 MMU 返回 PTE；
    
4.  MMU 构造物理地址 PA，并把它传给高速缓存 / 主存；
    
5.  高速缓存 / 主存返回所请求的数据给 CPU。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvatnhl3sxFBiblZPd2r6eEjOq0MQpyx1WSQibrn8icNqcShuRRy6ZjQFNwytm0rzWBHP1ZkdKJTH1FOPg/640?wx_fmt=jpeg)

一次简单的数据获取需要多次经过多次与内存的交互，如果是 4 级页表，那么就需要访问 4 次内存才能获取到对应的物理页号。如果是缺页，还需要有一个 PTE 的置换或加载过程。在开头也讲了，访问内存的性能其实很低的，实际上这严重影响了 CPU 处理性能。

程序所需要使用的指令，都顺序存放在虚拟内存里面。我们执行的指令，也是一条条顺序执行下去的。也就是说，我们对于指令地址的访问，存在前面几讲所说的 “空间局部性” 和“时间局部性”，而需要访问的数据也是一样的。我们连续执行了 5 条指令。因为内存地址都是连续的，所以我们可以通过加缓存的方法，把之前内存转换的地址缓存下来，减少与内存的交互。

加的这一层就是**缓存芯片 TLB** （Translation Lookaside Buffer），它里面每一行保存着一个由单个 PTE 组成的块。

那么查询数据的过程就变成了：

1.  CPU 产生一个虚拟地址；
    
2.  MMU 从 TLB 中取出相应的 PTE；
    
3.  MMU 将这个虚拟地址翻译成一个物理地址，并且将它发送到高速缓存 / 主存；
    
4.  高速缓存 / 主存将所请求的数据字返回给 CPU；

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvatnhl3sxFBiblZPd2r6eEjOq0GyuKMAnIfpxSM6Yh2xUiba7oNSDtU0ibLvmBEtffrEbXQU89qrySIsg/640?wx_fmt=jpeg)

### 3.4. 最后来看看为什么需要虚拟内存？

讲完了什么是虚拟内存，我们最后讲讲虚拟内存的必要性。

由于操作虚拟内存实际上就是操作页表，从上面讲解我们知道，页表的大小其实和物理内存没有关系，当物理内存不够用时可以通过页缺失来将需要的数据**置换**到内存中，内存中只需要存放众多程序中活跃的那部分，不需要将整个程序加载到内存里面，这可以让小内存的机器也可以运行程序。

虚拟内存可以为正在运行的进程提供独立的内存空间，制造一种每个进程的内存都是独立的假象。虚拟内存空间只是操作系统中的逻辑结构，通过多层的页表结构来转换虚拟地址，可以让多个进程可以通过虚拟内存**共享物理内存**。

并且独立的虚拟内存空间也会**简化内存**的分配过程，当用户程序向操作系统申请堆内存时，操作系统可以分配几个连续的虚拟页，但是这些虚拟页可以对应到物理内存中不连续的页中。

再来就是提供了**内存保护机制**。任何现代计算机系统必须为操作系统提供手段来控制对内存系统的访问。虚拟内存中页表中页存放了读权限、写权限和执行权限。内存管理单元可以决定当前进程是否有权限访问目标的物理内存，这样我们就最终将权限管理的功能全部收敛到虚拟内存系统中，减少了可能出现风险的代码路径。

## 4. 总结

从上面我们可以知道 CPU 的缓存结构一般由 L1、L2、L3 三层缓存结构组成，CPU 读取数据只与缓存交互，不会直接访问主存，所以 CPU 缓存和主存之间维护了一套映射关系。当被查找的数据发生缺失时，需要等待数据从主存加载到缓存中，如果缓存满了，那么还需要进行淘汰。如果被淘汰的数据是脏数据，那么还需要写回到主存中，写的策略有写直达（write-through）和写回（write-back）。

由于现在计算机中的 CPU 都是多核的，并且缓存数据是由多核共享的，所以就有了类似 MESI 这样的协议来维护一个状态机保证数据在多核之间是一致的。

为了访问数据安全，便捷，迅速所以加了一层虚拟内存，每个程序在启动的时候都会维护一个页表，这个页表维护了一套映射关系。CPU 操作的实际上是虚拟地址，每次需要 MMU 将虚拟地址在页表上映射成物理地址后查找数据。并且为了节省内存所以设计了多级页表，为了从页表中查找数据更快加了一个缓存芯片 TLB。

### 4.1. Reference

《深入理解计算机系统》

《深入浅出计算机组成原理》

《计算机组成与设计：硬件软件接口》

https://draveness.me/whys-the-design-os-virtual-memory/

https://people.freebsd.org/~lstewart/articles/cpumemory.pdf