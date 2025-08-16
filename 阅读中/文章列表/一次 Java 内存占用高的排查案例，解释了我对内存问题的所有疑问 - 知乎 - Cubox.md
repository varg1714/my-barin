---
source: https://cubox.pro/my/card?id=7172219807759073398
create: 2025-08-14 20:45
read: true
knowledge: true
knowledge-date: 2025-08-14
tags:
  - JVM
---
## 问题现象

7 月 25 号，我们一服务的内存占用较高，约 13G，容器总内存 16G，占用约 85%，触发了内存报警 (阈值 85%)，而我们是按容器内存 60%(9.6G) 的比例配置的 JVM 堆内存。看了下其它服务，同样的堆内存配置，它们内存占用约 70%~79%，此服务比其它服务内存占用稍大。

![](https://pic2.zhimg.com/v2-4f0418c878c958ef3330b7010e2c21e5_r.jpg)

  
那为什么此服务内存占用稍大呢，它存在内存泄露吗？

## 排查步骤

1. 检查 Java 堆占用与 gc 情况

![](https://pic2.zhimg.com/v2-d56a898280c298389a9f8fd41f5d53ed_r.jpg)

![](https://pic3.zhimg.com/v2-8adecf2df79a44df94f8842931daf6de_r.jpg)

可见堆使用情况正常。

## 2. 检查非堆占用情况

查看监控仪表盘，如下：  

![](https://pic1.zhimg.com/v2-d7e2a3087f8ff7623723a0b46eb66c04_r.jpg)

  
arthas 的 memory 命令查看，如下：  

![](https://pic1.zhimg.com/v2-f5cf456e331779ead44ee7f69b8ee0e8_r.jpg)

  
可见非堆内存占用也正常。

## 3. 检查 native 内存

Linux 进程的内存布局，如下：  

![](https://pic1.zhimg.com/v2-68da9b145bdf3f4ff177b0d7bbac89a0_b.jpg)

  
linux 进程启动时，有代码段、数据段、堆 (Heap)、栈(Stack) 及内存映射段，在运行过程中，应用程序调用 malloc、mmap 等 C 库函数来使用内存，C 库函数内部则会视情况通过 brk 系统调用扩展堆或使用 mmap 系统

创建新的内存映射段。

而通过 pmap 命令，就可以查看进程的内存布局，它的输出样例如下：  

![](https://pic1.zhimg.com/v2-77a0d9ead99b544fbb184a7916c87f7c_r.jpg)

可以发现，进程申请的所有虚拟内存段，都在 pmap 中能够找到，相关字段解释如下：

*   Address：表示此内存段的起始地址
*   Kbytes：表示此内存段的大小 (ps：这是虚拟内存)
*   RSS：表示此内存段实际分配的物理内存，这是由于 Linux 是延迟分配内存的，进程调用 malloc 时 Linux 只是分配了一段虚拟内存块，直到进程实际读写此内存块中部分时，Linux 会通过缺页中断真正分配物理内存。
*   Dirty：此内存段中被修改过的内存大小，使用 mmap 系统调用申请虚拟内存时，可以关联到某个文件，也可不关联，当关联了文件的内存段被访问时，会自动读取此文件的数据到内存中，若此段某一页内存数据后被更改，即为 Dirty，而对于非文件映射的匿名内存段 (anon)，此列与 RSS 相等。
*   Mode：内存段是否可读 (r) 可写 (w) 可执行(x)
*   Mapping：内存段映射的文件，匿名内存段显示为 anon，非匿名内存段显示文件名 (加 - p 可显示全路径)。

因此，我们可以找一些内存段，来看看这些内存段中都存储的什么数据，来确定是否有泄露。但 jvm 一般有非常多的内存段，重点检查哪些内存段呢？  
有两种思路，如下：  

1.  检查那些占用内存较大的内存段，如下：

```
jcmd 1 GC.heap_info
```

![](https://pic4.zhimg.com/v2-6a66a9c23d9e25c0d660f82b8b1b885f_r.jpg)

  
可以发现我们进程有非常多的 64M 的内存块，而我同时看了看其它 java 服务，发现 64M 内存块则少得多。  

1.  检查一段时间后新增了哪些内存段，或哪些变大了，如下：  
    在不同的时间点多次保存 pmap 命令的输出，然后通过文本对比工具查看两个时间点内存段分布的差异。

```
jstat -gcutil 1 1000
```

![](https://pic4.zhimg.com/v2-3fe2c5a46ee9348ddc227ae5b6f9877f_r.jpg)

```
pmap -x 1 | sort -nrk3 | less
```

![](https://pic3.zhimg.com/v2-e7db3cf47be49ef25fe38b0889391ab6_r.jpg)

  
可以看到，一段时间后，新分配了一些内存段，看看这些变化的内存段里存的是什么内容！  
tail -c +$((0x00007face0000000+1)) /proc/1/mem|head -c $((11616*1024))|strings|less -S  
说明：  

1.  Linux 将进程内存虚拟为伪文件 / proc/$pid/mem，通过它即可查看进程内存中的数据。
2.  tail 用于偏移到指定内存段的起始地址，即 pmap 的第一列，head 用于读取指定大小，即 pmap 的第二列。
3.  strings 用于找出内存中的字符串数据，less 用于查看 strings 输出的字符串。  
    

![](https://pic3.zhimg.com/v2-dcdf323561c3974db6a4d0ccf1656d86_r.jpg)

  
通过查看各个可疑内存段，发现有不少类似我们一自研消息队列的响应格式数据，通过与消息队列团队合作，找到了相关的消息 topic，并最终与相关研发确认了此 topic 消息最近刚迁移到此服务中。

## 4. 检查发 http 请求代码

由于发送消息是走 http 接口，故我在工程中搜索调用 http 接口的相关代码，发现一处代码中创建的流对象没有关闭，而 GZIPInputStream 这个类刚好会直接分配到 native 内存。  

![](https://pic1.zhimg.com/v2-f455df10102fde66d7b30a389102dbc4_r.jpg)

##   
其它方法

本次问题，通过检查内存中的数据找到了问题，还是有些碰运气的。这需要内存中刚好有一些非常有代表性的字符串，因为非字符串的二进制数据，基本无法分析。

如果查看内存数据无法找到关键线索，还可尝试以下几个方法：

## 5. 开启 JVM 的 NMT 原生内存追踪功能

添加 JVM 参数`-XX:NativeMemoryTracking=detail`开启，使用 jcmd 查看，如下：  
jcmd 1 VM.native_memory  

![](https://pic2.zhimg.com/v2-647253080a565699a99afb2d8ead0e19_r.jpg)

  
NMT 只能观察到 JVM 管理的内存，像通过 JNI 机制直接调用 malloc 分配的内存，则感知不到。

## 6. 检查被 glibc 内存分配器缓存的内存

JVM 等原生应用程序调用的 malloc、free 函数，实际是由基础 C 库 libc 提供的，而 linux 系统则提供了 brk、mmap、munmap 这几个系统调用来分配虚拟内存，所以 libc 的 malloc、free 函数实际是基于这些系统调用实现的。

由于系统调用有一定的开销，为减小开销，libc 实现了一个类似内存池的机制，在 free 函数调用时将内存块缓存起来不归还给 linux，直到缓存内存量到达一定条件才会实际执行归还内存的系统调用。

所以进程占用内存比理论上要大些，一定程度上是正常的。  

![](https://pic3.zhimg.com/v2-591a30f4f87cbc50b55956c889f2fcba_r.jpg)

  
**malloc_stats 函数**  
通过如下命令，可以确认 glibc 库缓存的内存量，如下：

```
pmap -x 1 > pmap-`date +%F-%H-%M-%S`.log
```

![](https://pic4.zhimg.com/v2-44d34b3775eb2c78c8692ebeda77facf_b.jpg)

  
如上，Total (incl. mmap) 表示 glibc 分配的总体情况 (包含 mmap 分配的部分)，其中 system bytes 表示 glibc 从操作系统中申请的虚拟内存总大小，in use bytes 表示 JVM 正在使用的内存总大小 (即调用 glibc 的 malloc 函数后且没有 free 的内存)。

可以发现，glibc 缓存了快 500m 的内存。  

注：当我对 jvm 进程中执行 malloc_stats 后，我发现它显示的 in use bytes 要少得多，经过检查 JVM 代码，发现 JVM 在为 Java Heap、Metaspace 分配内存时，是直接通过 mmap 函数分配的，而这个函数是直接封装的 mmap 系统调用，不走 glibc 内存分配器，故 in use bytes 会小很多。

**malloc_trim 函数**  
glibc 实现了 malloc_trim 函数，通过 brk 或 madvise 系统调用，归还被 glibc 缓存的内存，如下：

```
icdiff pmap-2023-07-27-09-46-36.log pmap-2023-07-28-09-29-55.log | less -SR
```

![](https://pic3.zhimg.com/v2-79e686d114321ebd58895433ee2cc23e_r.jpg)

![](https://pic2.zhimg.com/v2-f8552989204b0139e08ede69a1d5d489_r.jpg)

  
可以发现，执行 malloc_trim 后，RSS 减少了约 250m 内存，可见内存占用高并不是因为 glibc 缓存了内存。

注：通过 gdb 调用 C 函数，会有一定概率造成 jvm 进程崩溃，需谨慎执行。

## 7. 使用 tcmalloc 或 jemalloc 的内存泄露检测工具

glibc 的默认内存分配器为 ptmalloc2，但 Linux 提供了 LD_PRELOAD 机制，使得我们可以更换为其它的内存分配器，如业内比较成熟的 tcmalloc 或 jemalloc。

这两个内存分配器除了实现了内存分配功能外，还提供了内存泄露检测的能力，它们通过 hook 进程的 malloc、free 函数调用，然后找到那些调用了 malloc 后一直没有 free 的地方，那么这些地方就可能是内存泄露点。

```
# 查看glibc内存分配情况，会输出到进程标准错误中
gdb -q -batch -ex 'call malloc_stats()' -p 1
```

![](https://pic1.zhimg.com/v2-c906b193a70feb8e41d5665f24eb1e9c_r.jpg)

  
tcmalloc 下载地址：[github.com/gperftools/…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fgperftools%2Fgperftools)  
如上，可以发现内存泄露点来自 Inflater 对象的 init 和 inflateBytes 方法，而这些方法是通过 JNI 调用实现的，它会申请 native 内存，经过检查代码，发现 GZIPInputStream 确实会创建并使用 Inflater 对象，如下：  

![](https://pic3.zhimg.com/v2-1d572f65c2b087357dd47f8d5914e0a2_r.jpg)

  
而它的 close 方法，会调用 Inflater 的 end 方法来归还 native 内存，由于我们没有调用 close 方法，故相关联的 native 内存无法归还。  

![](https://pic1.zhimg.com/v2-b8facee2a03c644ded7e969a45c92454_b.jpg)

  
可以发现，tcmalloc 的泄露检测只能看到 native 栈，如想看到 Java 栈，可考虑配合使用 arthas 的 profile 命令，如下：

```
# 回收glibc缓存的内存
gdb -q -batch -ex 'call malloc_trim(0)' -p 1
```

## 如果代码不修复，内存会一直涨吗？

经过查看代码，发现 Inflater 实现了 finalize 方法，而 finalize 方法调用了 end 方法。

也就是说，若 GC 时 Inflater 对象被回收，相关联的原生内存是会被 free 的，所以内存会一直涨下去导致进程被 oom kill 吗？maybe，这取决于 GC 触发的阈值，即在 GC 触发前 JVM 中会保留的垃圾 Inflater 对象数量，保留得越多 native 内存占用越大。  

![](https://pic2.zhimg.com/v2-99aa0a1f9295691157b536db8c4fc2d1_r.jpg)

  
但我发现一个有趣现象，我通过 jcmd 强行触发了一次 Full GC，如下：

理论上 native 内存应该会 free，但我通过 top 观察进程 rss，发现基本没有变化，但我检查 malloc_stats 的输出，发现 in use bytes 确实少了许多，这说明 Full GC 后，JVM 确实归还了 Inflater 对象关联的原生内存，但它们都被 glibc 缓存起来了，并没有归还给操作系统。

于是我再执行了一次 malloc_trim，强制 glibc 归还缓存的内存，发现进程的 rss 降了下来。

编码最佳实践

这个问题是由于 InputStream 流对象未关闭导致的，在 Java 中流对象 (FileInputStream)、网络连接对象(Socket) 一般都关联了原生资源，记得在 finally 中调用 close 方法归还原生资源。

而 GZIPInputstream、Inflater 是 JVM 堆外内存泄露的常见问题点，review 代码发现有使用这些类时，需要保持警惕。

## JVM 内存常见疑问

为什么我设置了 - Xmx 为 10G，top 中看到的 rss 却大于 10G？

根据上面的介绍，JVM 内存占用分布大概如下：  

![](https://pic3.zhimg.com/v2-fa525eb0a0e3d8fa1380e27ecbf6224e_r.jpg)

  
可以发现，JVM 内存占用主要包含如下部分：

1.  Java 堆，-Xmx 选项限制的就是 Java 堆的大小，可通过 jcmd 命令观测。
2.  Java 非堆，包含 Metaspace、Code Cache、直接内存 (DirectByteBuffer、MappedByteBuffer)、Thread、GC，它可通过 arthas memory 命令或 NMT 原生内存追踪观测。
3.  native 分配内存，即直接调用 malloc 分配的，如 JNI 调用、磁盘与网络 io 操作等，可通过 pmap 命令、malloc_stats 函数观测，或使用 tcmalloc 检测泄露点。
4.  glibc 缓存的内存，即 JVM 调用 free 后，glibc 库缓存下来未归还给操作系统的部分，可通过 pmap 命令、malloc_stats 函数观测。

所以 - Xmx 的值，一定要小于容器 / 物理机的内存限制，根据经验，一般设置为容器 / 物理机内存的 65% 左右较为安全，可考虑使用比例的方式代替 - Xms 与 - Xmx，如下：  
-XX:MaxRAMPercentage=65.0 -XX:InitialRAMPercentage=65.0 -XX:MinRAMPercentage=65.0  
top 中 VIRT 与 RES 是什么区别？  

![](https://pic4.zhimg.com/v2-60214ee8c52af6504f9af44652c9c173_r.jpg)

*   VIRT：进程申请的虚拟内存总大小。
*   RES：进程在读写它申请的虚拟内存页面后，会触发 Linux 的内存缺页中断，进而导致 Linux 为该页分配实际内存，即 RSS，在 top 中叫 RES。
*   SHR：进程间共享的内存，如 libc.so 这个 C 动态库，几乎会被所有进程加载到各自的虚拟内存空间并使用，但 Linux 实际只分配了一份内存，各个进程只是通过内存页表关联到此内存而已，注意，RSS 指标一般也包含 SHR。

通过 top、ps 或 pidstat 可查询进程的缺页中断次数，如下：  
top 中可以通过 f 交互指令，将 mMin、mMaj 列显示出来。  

![](https://pic4.zhimg.com/v2-0bd1b2ffdb19c1961a08dc0ac8f14fef_r.jpg)

![](https://pic1.zhimg.com/v2-ecfede6572f24eb7b78e55488e18aca8_b.jpg)

![](https://pic3.zhimg.com/v2-505f9f23822b6d63126471da59f1da0e_r.jpg)

  
minflt 表示轻微缺页，即 Linux 分配了一个内存页给进程，而 majflt 表示主要缺页，即 Linux 除了要分配内存页外，还需要从磁盘中读取数据到内存页，一般是内存 swap 到了磁盘后再访问，或使用了内存映射技术读取文件。

## 为什么 top 中 JVM 进程的 VIRT 列 (虚拟内存) 那么大？  

![](https://pic2.zhimg.com/v2-9fdeccbbbdc4b8e0925d07d77bb82f11_r.jpg)

  
可以看到，我们一 Java 服务，申请了约 30G 的虚拟内存，比 RES 实际内存 5.6G 大很多。

这是因为 glibc 为了解决多线程内存申请时的锁竞争问题，创建了多个内存分配区 Arena，然后每个 Arena 都有一把锁，特定的线程会 hash 到特定的 Arena 中去竞争锁并申请内存，从而减少锁开销。

但在 64 位系统里，每个 Arena 去系统申请虚拟内存的单位是 64M，然后按需拆分为小块分配给申请方，所以哪怕线程在此 Arena 中只申请了 1K 内存，glibc 也会为此 Arena 申请 64M。

64 位系统里 glibc 创建 Arena 数量的默认值为 CPU 核心数的 8 倍，而我们容器运行在 32 核的机器，故 glibc 会创建`32*8=256`个 Arena，如果每个 Arena 最少申请 64M 虚拟内存的话，总共申请的虚拟内存为`256*64M=16G`。  

![](https://pic4.zhimg.com/v2-b31ab6a3c99243977f613c88c7565c57_r.jpg)

  
然后 JVM 是直接通过 mmap 申请的堆、MetaSpace 等内存区域，不走 glibc 的内存分配器，这些加起来大约 14G，与走 glibc 申请的 16G 虚拟内存加起来，总共申请虚拟内存 30G！

当然，不必惊慌，这些只是虚拟内存而已，它们多一些并没有什么影响，毕竟 64 位进程的虚拟内存空间有 2^48 字节那么大！

## 为什么 jvm 启动后一段时间内内存占用越来越多，存在内存泄露吗？

如下，是我们一服务重启后运行快 2 天的内存占用情况，可以发现内存一直从 45% 涨到了 62%，8G 的容器，上涨内存大小为 1.36G！  

![](https://pic4.zhimg.com/v2-791b4d12eb7202664fae1b20795f3b17_r.jpg)

  
但我们这个服务其实没有内存泄露问题，因为 JVM 为堆申请的内存是虚拟内存，如 4.8G，但在启动后 JVM 一开始可能实际只使用了 3G 内存，导致 Linux 实际只分配了 3G。

然后在 gc 时，由于会复制存活对象到堆的空闲部分，如果正好复制到了以前未使用过的区域，就又会触发 Linux 进行内存分配，故一段时间内内存占用会越来越多，直到堆的所有区域都被 touch 到。  

![](https://pic3.zhimg.com/v2-c6a9d60e6456a760d8d2db363ac1bd0e_r.jpg)

  
而通过添加 JVM 参数`-XX:+AlwaysPreTouch`，可以让 JVM 为堆申请虚拟内存后，立即把堆全部 touch 一遍，使得堆区域全都被分配物理内存，而由于 Java 进程主要活动在堆内，故后续内存就不会有很大变化了，我们另一服务添加了此参数，内存表现如下：  

![](https://pic3.zhimg.com/v2-b36ed6b742aee54f599ab84bca7bf5ce_r.jpg)

  
可以看到，内存上涨幅度不到 2%，无此参数可以提高内存利用度，加此参数则会使应用运行得更稳定。

如我们之前一服务一周内会有 1 到 2 次 GC 耗时超过 2s，当我添加此参数后，再未出现过此情况。这是因为当无此参数时，若 GC 访问到了未读写区域，会触发 Linux 分配内存，大多数情况下此过程很快，但有极少数情况下会较慢，在 GC 日志中则表现为 sys 耗时较高。  

![](https://pic3.zhimg.com/v2-af8bb7401ecfcf3d0bedf6041f9d9d52_r.jpg)