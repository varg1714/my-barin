---
source: https://cubox.pro/my/card?id=7172204839865156151
create: 2025-08-14 20:46
read: true
knowledge: true
knowledge-date: 2025-08-15
tags:
  - JVM
summary: "[[阅读中/阅读总结/文章收藏/2025-09/Spring Boot 引起的 “堆外内存泄漏” 排查及经验总结 - 美团技术团队 - Cubox|Spring Boot 引起的 “堆外内存泄漏” 排查及经验总结 - 美团技术团队 - Cubox]]"
---
## 背景

为了更好地实现对项目的管理，我们将组内一个项目迁移到 MDP 框架（基于 Spring Boot），随后我们就发现系统会频繁报出 Swap 区域使用量过高的异常。笔者被叫去帮忙查看原因，发现配置了 4G 堆内内存，但是实际使用的物理内存竟然高达 7G，确实不正常。JVM 参数配置是 “-XX:MetaspaceSize=256M -XX:MaxMetaspaceSize=256M -XX:+AlwaysPreTouch -XX:ReservedCodeCacheSize=128m -XX:InitialCodeCacheSize=128m, -Xss512k -Xmx4g -Xms4g,-XX:+UseG1GC -XX:G1HeapRegionSize=4M”，实际使用的物理内存如下图所示：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fawps-assets.meituan.net%2Fmit-x%2Fblog-images-bundle-2019a%2F0834c647.png&valid=true)

top 命令显示的内存情况

## 排查过程

### 1. 使用 Java 层面的工具定位内存区域（堆内内存、Code 区域或者使用 unsafe.allocateMemory 和 DirectByteBuffer 申请的堆外内存）

笔者在项目中添加`-XX:NativeMemoryTracking=detail`JVM 参数重启项目，使用命令`jcmd pid VM.native_memory detail`查看到的内存分布如下：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fawps-assets.meituan.net%2Fmit-x%2Fblog-images-bundle-2019a%2F34b8e79d.png&valid=true)

jcmd 显示的内存情况

发现命令显示的 committed 的内存小于物理内存，因为 jcmd 命令显示的内存包含堆内内存、Code 区域、通过 unsafe.allocateMemory 和 DirectByteBuffer 申请的内存，但是不包含其他 Native Code（C 代码）申请的堆外内存。所以猜测是使用 Native Code 申请内存所导致的问题。

为了防止误判，笔者使用了 pmap 查看内存分布，发现大量的 64M 的地址；而这些地址空间不在 jcmd 命令所给出的地址空间里面，基本上就断定就是这些 64M 的内存所导致。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fawps-assets.meituan.net%2Fmit-x%2Fblog-images-bundle-2019a%2F14063492.png&valid=true)

pmap 显示的内存情况

### 2. 使用系统层面的工具定位堆外内存

因为笔者已经基本上确定是 Native Code 所引起，而 Java 层面的工具不便于排查此类问题，只能使用系统层面的工具去定位问题。

#### 首先，使用了 gperftools 去定位问题

gperftools 的使用方法可以参考 [gperftools](https://github.com/gperftools/gperftools)，gperftools 的监控如下：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fawps-assets.meituan.net%2Fmit-x%2Fblog-images-bundle-2019a%2Fec7740f1.png&valid=true)

gperftools 监控

从上图可以看出：使用 malloc 申请的的内存最高到 3G 之后就释放了，之后始终维持在 700M-800M。笔者第一反应是：难道 Native Code 中没有使用 malloc 申请，直接使用 mmap/brk 申请的？（gperftools 原理就使用动态链接的方式替换了操作系统默认的内存分配器（glibc）。）

#### 然后，使用 strace 去追踪系统调用

因为使用 gperftools 没有追踪到这些内存，于是直接使用命令 “strace -f -e”brk,mmap,munmap” -p pid” 追踪向 OS 申请内存请求，但是并没有发现有可疑内存申请。strace 监控如下图所示:

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fawps-assets.meituan.net%2Fmit-x%2Fblog-images-bundle-2019a%2Fd865697e.jpg&valid=true)

strace 监控

#### 接着，使用 GDB 去 dump 可疑内存

因为使用 strace 没有追踪到可疑内存申请；于是想着看看内存中的情况。就是直接使用命令`gdp -pid pid`进入 GDB 之后，然后使用命令`dump memory mem.bin startAddress endAddress`dump 内存，其中 startAddress 和 endAddress 可以从 / proc/pid/smaps 中查找。然后使用`strings mem.bin`查看 dump 的内容，如下：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fawps-assets.meituan.net%2Fmit-x%2Fblog-images-bundle-2019a%2F94def72f.jpg&valid=true)

gperftools 监控

从内容上来看，像是解压后的 JAR 包信息。读取 JAR 包信息应该是在项目启动的时候，那么在项目启动之后使用 strace 作用就不是很大了。所以应该在项目启动的时候使用 strace，而不是启动完成之后。

#### 再次，项目启动时使用 strace 去追踪系统调用

项目启动使用 strace 追踪系统调用，发现确实申请了很多 64M 的内存空间，截图如下：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fawps-assets.meituan.net%2Fmit-x%2Fblog-images-bundle-2019a%2F025d9f15.png&valid=true)

strace 监控

使用该 mmap 申请的地址空间在 pmap 对应如下：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fawps-assets.meituan.net%2Fmit-x%2Fblog-images-bundle-2019a%2Fbd4b1892.png&valid=true)

strace 申请内容对应的 pmap 地址空间

#### 最后，使用 jstack 去查看对应的线程

因为 strace 命令中已经显示申请内存的线程 ID。直接使用命令`jstack pid`去查看线程栈，找到对应的线程栈（注意 10 进制和 16 进制转换）如下：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fawps-assets.meituan.net%2Fmit-x%2Fblog-images-bundle-2019a%2F00ca5985.png&valid=true)

strace 申请空间的线程栈

这里基本上就可以看出问题来了：MCC（美团统一配置中心）使用了 Reflections 进行扫包，底层使用了 Spring Boot 去加载 JAR。因为解压 JAR 使用 Inflater 类，需要用到堆外内存，然后使用 Btrace 去追踪这个类，栈如下：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fawps-assets.meituan.net%2Fmit-x%2Fblog-images-bundle-2019a%2F6158ede5.png&valid=true)

btrace 追踪栈

然后查看使用 MCC 的地方，发现没有配置扫包路径，默认是扫描所有的包。于是修改代码，配置扫包路径，发布上线后内存问题解决。

### 3. 为什么堆外内存没有释放掉呢？

虽然问题已经解决了，但是有几个疑问：

*   为什么使用旧的框架没有问题？
*   为什么堆外内存没有释放？
*   为什么内存大小都是 64M，JAR 大小不可能这么大，而且都是一样大？
*   为什么 gperftools 最终显示使用的的内存大小是 700M 左右，解压包真的没有使用 malloc 申请内存吗？

带着疑问，笔者直接看了一下 [Spring Boot Loader](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-tools/spring-boot-loader/src/main/java/org/springframework/boot/loader) 那一块的源码。发现 Spring Boot 对 Java JDK 的 InflaterInputStream 进行了包装并且使用了 Inflater，而 Inflater 本身用于解压 JAR 包的需要用到堆外内存。而包装之后的类 ZipInflaterInputStream 没有释放 Inflater 持有的堆外内存。于是笔者以为找到了原因，立马向 Spring Boot 社区反馈了[这个 bug](https://github.com/spring-projects/spring-boot/issues/13935)。但是反馈之后，笔者就发现 Inflater 这个对象本身实现了 finalize 方法，在这个方法中有调用释放堆外内存的逻辑。也就是说 Spring Boot 依赖于 GC 释放堆外内存。

笔者使用 jmap 查看堆内对象时，发现已经基本上没有 Inflater 这个对象了。于是就怀疑 GC 的时候，没有调用 finalize。带着这样的怀疑，笔者把 Inflater 进行包装在 Spring Boot Loader 里面替换成自己包装的 Inflater，在 finalize 进行打点监控，结果 finalize 方法确实被调用了。于是笔者又去看了 Inflater 对应的 C 代码，发现初始化的使用了 malloc 申请内存，end 的时候也调用了 free 去释放内存。

此刻，笔者只能怀疑 free 的时候没有真正释放内存，便把 Spring Boot 包装的 InflaterInputStream 替换成 Java JDK 自带的，发现替换之后，内存问题也得以解决了。

这时，再返过来看 gperftools 的内存分布情况，发现使用 Spring Boot 时，内存使用一直在增加，突然某个点内存使用下降了好多（使用量直接由 3G 降为 700M 左右）。这个点应该就是 GC 引起的，内存应该释放了，但是在操作系统层面并没有看到内存变化，那是不是没有释放到操作系统，被内存分配器持有了呢？

继续探究，发现系统默认的内存分配器（glibc 2.12 版本）和使用 gperftools 内存地址分布差别很明显，2.5G 地址使用 smaps 发现它是属于 Native Stack。内存地址分布如下：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fawps-assets.meituan.net%2Fmit-x%2Fblog-images-bundle-2019a%2F036c6f2d.png&valid=true)

gperftools 显示的内存地址分布

到此，基本上可以确定是内存分配器在捣鬼；搜索了一下 glibc 64M，发现 glibc 从 2.11 开始对每个线程引入内存池（64 位机器大小就是 64M 内存），原文如下：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fawps-assets.meituan.net%2Fmit-x%2Fblog-images-bundle-2019a%2Fd43ef51b.jpg&valid=true)

glib 内存池说明

按照文中所说去修改 MALLOC_ARENA_MAX 环境变量，发现没什么效果。查看 tcmalloc（gperftools 使用的内存分配器）也使用了内存池方式。

为了验证是内存池搞的鬼，笔者就简单写个不带内存池的内存分配器。使用命令`gcc zjbmalloc.c -fPIC -shared -o zjbmalloc.so`生成动态库，然后使用`export LD_PRELOAD=zjbmalloc.so`替换掉 glibc 的内存分配器。其中代码 Demo 如下：

```
#include<sys/mman.h>
#include<stdlib.h>
#include<string.h>
#include<stdio.h>
//作者使用的64位机器，sizeof(size_t)也就是sizeof(long) 
void* malloc ( size_t size ) {
   long* ptr = mmap( 0, size + sizeof(long), PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS, 0, 0 );
   if (ptr == MAP_FAILED) {
  	return NULL;
   }
   *ptr = size;                     // First 8 bytes contain length.
   return (void*)(&ptr[1]);        // Memory that is after length variable
}

void *calloc(size_t n, size_t size) {
 void* ptr = malloc(n * size);
 if (ptr == NULL) {
	return NULL;
 }
 memset(ptr, 0, n * size);
 return ptr;
}
void *realloc(void *ptr, size_t size) {
 if (size == 0) {
	free(ptr);
	return NULL;
 }
 if (ptr == NULL) {
	return malloc(size);
 }
 long *plen = (long*)ptr;
 plen--;                          // Reach top of memory
 long len = *plen;
 if (size <= len) {
	return ptr;
 }
 void* rptr = malloc(size);
 if (rptr == NULL) {
	free(ptr);
	return NULL;
 }
 rptr = memcpy(rptr, ptr, len);
 free(ptr);
 return rptr;
}

void free (void* ptr ) {
   if (ptr == NULL) {
	 return;
   }
   long *plen = (long*)ptr;
   plen--;                          // Reach top of memory
   long len = *plen;               // Read length
   munmap((void*)plen, len + sizeof(long));
}
```

通过在自定义分配器当中埋点可以发现其实程序启动之后应用实际申请的堆外内存始终在 700M-800M 之间，gperftools 监控显示内存使用量也是在 700M-800M 左右。但是从操作系统角度来看进程占用的内存差别很大（这里只是监控堆外内存）。

笔者做了一下测试，使用不同分配器进行不同程度的扫包，占用的内存如下：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fawps-assets.meituan.net%2Fmit-x%2Fblog-images-bundle-2019a%2Fdaa0c8d0.jpg&valid=true)

内存测试对比

为什么自定义的 malloc 申请 800M，最终占用的物理内存在 1.7G 呢？

因为自定义内存分配器采用的是 mmap 分配内存，mmap 分配内存按需向上取整到整数个页，所以存在着巨大的空间浪费。通过监控发现最终申请的页面数目在 536k 个左右，那实际上向系统申请的内存等于 512k * 4k（pagesize） = 2G。 为什么这个数据大于 1.7G 呢？

因为操作系统采取的是延迟分配的方式，通过 mmap 向系统申请内存的时候，系统仅仅返回内存地址并没有分配真实的物理内存。只有在真正使用的时候，系统产生一个缺页中断，然后再分配实际的物理 Page。

## 总结

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fawps-assets.meituan.net%2Fmit-x%2Fblog-images-bundle-2019a%2F3612ea5b.jpg&valid=true)

流程图

整个内存分配的流程如上图所示。MCC 扫包的默认配置是扫描所有的 JAR 包。在扫描包的时候，Spring Boot 不会主动去释放堆外内存，导致在扫描阶段，堆外内存占用量一直持续飙升。当发生 GC 的时候，Spring Boot 依赖于 finalize 机制去释放了堆外内存；但是 glibc 为了性能考虑，并没有真正把内存归返到操作系统，而是留下来放入内存池了，导致应用层以为发生了 “内存泄漏”。所以修改 MCC 的配置路径为特定的 JAR 包，问题解决。笔者在发表这篇文章时，发现 Spring Boot 的最新版本（2.0.5.RELEASE）已经做了修改，在 ZipInflaterInputStream 主动释放了堆外内存不再依赖 GC；所以 Spring Boot 升级到最新版本，这个问题也可以得到解决。

## 参考资料

1.  [GNU C Library (glibc)](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/6.0_release_notes/compiler)
2.  [Native Memory Tracking](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr007.html)
3.  [Spring Boot](https://github.com/spring-projects/spring-boot)
4.  [gperftools](https://github.com/gperftools/gperftools)
5.  [Btrace](https://github.com/btraceio/btrace)

## 作者简介

*   纪兵，2015 年加入美团，目前主要从事酒店 C 端相关的工作。
