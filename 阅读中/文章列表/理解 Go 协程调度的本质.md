---
source: https://mp.weixin.qq.com/s/j9OpuIxXRWa9524oacGCzw
create: 2025-04-17 21:41
read: false
---
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/j3gficicyOvasVeMDmWoZ2zyN8iaSc6XWYj79H3xfgvsqK9TDxOBlcUa6W0EE5KBdxacd2Ql6QBmuhBJKIUS4PSZQ/640?wx_fmt=gif&from=appmsg)

  

文末有福利~

作者：jiayan

golang 的一大特色就是 goroutine，它是支持高并发程序的重要保障；通过 go 关键字我们就能轻易创建大量的轻量级协程，但它和我们认知中的线程有什么区别呢，轻量在哪里，具体是如何进行调度的..... 本文将从涉及到的一些基础知识开始，逐步介绍到 go 协程调度的核心原理，希望你能有所收获~  

### 函数调用栈

#### 进程在内存中的布局

首先回顾下进程的内存布局~ 操作系统把磁盘上的可执行文件加载到内存运行之前，会做很多工作，其中很重要的一件事情就是把可执行文件中的代码，数据放在内存中合适的位置，并分配和初始化程序运行过程中所必须的堆栈，所有准备工作完成后操作系统才会调度程序起来运行。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvatXEp81ac9kdSiaAdzNAZyfACQmib9DrsicGzpzicPLRZmfL8ibYwlHm9NAy2AHokmk64g0iarcjhmsu05Q/640?wx_fmt=png&from=appmsg)

用户程序所使用的内存空间在低地址，内核空间所使用的内存在高地址，需要特别注意的是栈是从高地址往低地址生长。

#### 各区域详解

*   **代码区**，也被称为代码段，这部分内存存放了程序的机器代码。这部分内存通常是只读的，以防止程序意外地修改其自身的指令。
    
*   **数据区**，包括程序的全局变量和静态变量，程序加载完毕后数据区的大小也不会发生改变。
    
*   **堆**，堆是用于动态内存分配的区域，例如 c 语言的`malloc`函数和 go 语言的`new`函数就是在堆上分配内存。堆从低地址向高地址增长。
    
*   **栈**，栈内存是一个连续的内存区域，通常从高地址向低地址增长。每次函数调用都会在栈上分配一个新的栈帧，函数返回时栈帧会被释放。栈帧包含了函数调用的上下文信息，包括函数的参数、局部变量和返回地址。
    

#### 栈详解

##### 栈内存中保存了什么

*   保存函数的局部变量；
    
*   返回函数的返回值；
    
*   向被调用函数传递参数；
    
*   保存函数的返回地址，返回地址是指从被调用函数返回后调用者应该继续执行的指令地址;
    

每个函数在执行过程中都需要使用一块栈内存用来保存上述这些值，我们称这块栈内存为某函数的栈帧 (stack frame)。

##### 与栈密切相关的三个寄存器

AMD64 CPU 中有 3 个与栈密切相关的寄存器：

*   **rsp 寄存器** ，始终指向当前函数调用栈栈顶。
    
*   **rbp 寄存器** ，一般用来指向当前函数栈帧的起始位置，即栈底。
    
*   **ip 寄存器**，保存着下一条将要执行的指令的内存地址。CPU 在执行指令时，会根据 IP 寄存器的值从内存中获取指令从而执行，在大多数情况下，IP 寄存器的值会按顺序递增，以指向下一条指令，这使得程序能够顺序执行。
    

假设现在有如下的一段 go 函数调用链且 ** 当前正在执行函数 C()**：

```
main() { A()}func A() {  ... 声明了一些局部变量...  B(1, 2)  test := 123}func B (a int , b int) {        ... 声明了一些局部变量...  C()}func C (a int, b int , c int) {   test := 123 }
```

则函数 ABC 的栈帧以及 rsp/rbp/ip 寄存器的状态大致如下图所示（注意，栈从高地址向低地址方向生长）：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvatXEp81ac9kdSiaAdzNAZyfAiatR02oRiaznzPTvkD71omEMeYaWsnXsNkJEjGtGDsfyiaqhaicTYqUr8Q/640?wx_fmt=png&from=appmsg)

对于上图，有几点需要说明一下：

*   go 语言中调用函数时，参数和返回值都是存放在调用者的栈帧之中，而不是在被调函数之中；
    
*   目前正在执行 C 函数，且函数调用链为 A()->B()->C()，所以以栈帧为单位来看的话，C 函数的栈帧目前位于栈顶；
    
*   cpu 硬件寄存器 rsp 指向整个栈的栈顶，当然它也指向 C 函数的栈帧的栈顶，而 rbp 寄存器指向的是 C 函数栈帧的起始位置；
    
*   虽然图中 ABC 三个函数的栈帧看起来都差不多大，但事实上在真实的程序中，每个函数的栈帧大小可能都不同，因为不同的函数局部变量的个数以及所占内存的大小都不尽相同；
    
*   有些编译器比如 gcc 会把参数和返回值放在寄存器中而不是栈中，go 语言中函数的参数和返回值都是放在栈上的；
    

随着程序的运行，如果 C、B 两个函数都执行完成并返回到了 A 函数继续执行，则栈状态如下图：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvatXEp81ac9kdSiaAdzNAZyfAAC4ynZF0AMHVDzQTzKnIAu79Uibe1oMichGFlmnyQ4l7vt7ibKXHm0Rbw/640?wx_fmt=png&from=appmsg)

因为 C、B 两个函数都已经执行完成并返回到了 A 函数之中，所以 C、B 两个函数的栈帧就已经被 POP 出栈了，也就是说它们所消耗的栈内存被自动回收了。因为现在正在执行 A 函数，所以寄存器 rbp 和 rsp 指向的是 A 函数的栈中的相应位置。可以看到 cpu 的 rbp, rsp 分别指向了 a 函数的栈底和栈顶，同时 ip 寄存器指向了 A 函数调用完 B 后的下一行代码 **test:= 123** 的地址，接下来 CPU 会根据 IP 寄存器，从代码区的内存中获取下一行代码所对应的汇编指令去执行。

##### 栈溢出

即使是同一个函数，每次调用都会产生一个不同的栈帧，因此对于递归函数，每递归一次都会消耗一定的栈内存，如果递归层数太多就有导致栈溢出的风险，这也是为什么我们在实际的开发过程中应该尽量避免使用递归函数的原因之一，另外一个原因是递归函数执行效率比较低，因为它要反复调用函数，而调用函数有较大的性能开销。

### linux 线程以及线程调度

#### 一段 c 程序

要深入理解 go 的协程调度逻辑，就需要对操作系统线程有个大致的了解，因为 go 的调度系统是建立在操作系统线程之上的，所以我们先来通过 linux 下的 C 语言 demo 入手，我们把这个程序跑在一台**单核 CPU** 的机器上。

C 语言中我们一般使用 pthread 线程库，而使用该线程库创建的用户态线程其实就是 Linux 操作系统内核所支持的线程，它与 go 语言中的工作线程是一样的，这些线程都由 Linux 内核负责管理和调度，然后 go 语言在操作系统线程之上又做了 goroutine，实现了一个二级线程模型。

```
#include <stdio.h>#include <unistd.h>#include <pthread.h>#define N (1000 * 1000 * 1000)volatile int g = 0;void *start(void *arg){        int i;        for (i = 0; i < N; i++) {                g++;        }        return NULL;}int main(int argc, char *argv[]){        pthread_t tid;        // 使用pthread_create函数创建一个新线程执行start函数        pthread_create(&tid, NULL, start, NULL);        for (;;) {                usleep(1000 * 100 * 5);                printf("loop g: %d\n", g);                if (g == N) {                        break;                }        }        pthread_join(tid, NULL); // 等待子线程结束运行        return 0;}```c$./threadloop g: 98938361loop g: 198264794loop g: 297862478loop g: 396750048loop g: 489684941loop g: 584723988loop g: 679293257loop g: 777715939loop g: 876083765loop g: 974378774loop g: 1000000000
```

该程序运行起来之后将会有 2 个线程，一个是操作系统把程序加载起来运行时创建的主线程，另一个是主线程调用 pthread_create 创建的 start 子线程，主线程在创建完子线程之后每隔 500 毫秒打印一下全局变量 g 的值直到 g 等于 10 亿，而 start 线程启动后就开始执行一个 10 亿次的对 g 自增加 1 的循环，这两个线程同时并发运行在系统中，**操作系统负责对它们进行调度**，我们无法精确预知某个线程在什么时候会运行。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvatXEp81ac9kdSiaAdzNAZyfAl3wAGUplajnD5w2cRdV2D58QiaxMjvN0oibX2EIeKib16khibFNxugtYGg/640?wx_fmt=png&from=appmsg)

#### 操作系统在调度线程时会做哪些事情

1.  **选择线程**：操作系统调度器会根据特定的调度算法（如优先级调度、轮转调度、最短作业优先等）选择下一个要执行的线程。
    
2.  **上下文切换**：操作系统会保存当前正在运行的线程的状态（这被称为上下文），然后加载被选中的线程的上下文。上下文包括了线程的程序计数器、寄存器的值等。
    
3.  **线程切换**：操作系统会将 CPU 的控制权交给被选中的线程，该线程会从它上次停止的地方开始执行。
    
4.  **回到原线程**：当被选中的线程的执行时间片用完或者被阻塞时，操作系统会再次保存它的上下文，然后选择另一个线程执行。这个过程会不断重复。
    

#### 具体需要保存哪些寄存器呢

*   通用寄存器，线程运行时很可能会用到，保存当前线程的一些工作变量。
    
*   ip 寄存器（指令指针寄存器）（程序运行起来后低地址有一部分专门用来存放代码数据，IP 寄存器通常指向这一区域，指明下一条要运行的代码地址）。
    
*   栈寄存器 RBP（指向当前栈的栈底）和 RSP（当前栈的栈顶），2 个栈寄存器确定了线程执行时需要使用的栈内存。所以恢复 CPU 寄存器的值就相当于改变了 CPU 下一条需要执行的指令，同时也切换了函数调用栈。
    

#### 线程调度的核心是什么

操作系统对线程的调度可以简单的理解为**内核调度器对不同线程所使用的寄存器和栈的切换**。

### goroutine 调度器

#### 调度模型

##### 传统线程模型的问题

###### 调度

上面讲到了线程是操作系统级别的调度单位，通常由操作系统内核管理。切上下文切换的开销通常在微秒级别，且频繁的上下文切换会显著影响性能。

###### 资源消耗

每个线程都有自己的堆栈和线程局部存储（Thread Local Storage），这会消耗更多的内存资源。创建和销毁线程的开销也相对较大。在 **Linux** 中，默认的线程栈大小通常为 8 MB。

###### 调度策略

线程调度通常由操作系统内核使用复杂的调度算法（如轮转调度、优先级调度等）来管理。调度器需要考虑多个线程的优先级、状态、资源占用等因素，调度过程相对复杂。

##### goroutine 有多轻量

而相对的，用户态的 goroutine 则轻量得多：

1.  goroutine 是用户态线程，其创建和切换都在用户代码中完成而无需进入操作系统内核，所以其开销要远远小于系统线程的创建和切换；
    
2.  goroutine 启动时默认栈大小只有 2k，这在多数情况下已经够用了，即使不够用，goroutine 的栈也会自动扩大，同时，如果栈太大了过于浪费它还能自动收缩，这样既没有栈溢出的风险，也不会造成栈内存空间的大量浪费。
    

正是因为 go 语言中实现了如此轻量级的线程，才使得我们在 Go 程序中，可以轻易的创建成千上万甚至上百万的 goroutine 出来并发的执行任务而不用太担心性能和内存等问题。

##### go 调度器的简化模型

goroutine 建立在操作系统线程基础之上，它与操作系统线程之间实现了一个多对多 (M:N) 的两级线程模型 这里的 M:N 是指 M 个 goroutine 运行在 N 个操作系统线程之上，内核负责对这 N 个操作系统线程进行调度，而这 N 个系统线程又负责对这 M 个 goroutine 进行调度和运行。

所谓的**对 goroutine 的调度，是指程序代码按照一定的算法在适当的时候挑选出合适的 goroutine 并放到 CPU 上去运行的过程**，这些负责对 goroutine 进行调度的程序代码我们称之为 **goroutine 调度器**。用极度简化了的伪代码来描述 goroutine 调度器的工作流程大概是下面这个样子：

```
// 程序启动时的初始化代码......for i := 0; i < N; i++ { // 创建N个操作系统线程执行schedule函数     create_os_thread(schedule) // 创建一个操作系统线程执行schedule函数}//schedule函数实现调度逻辑func schedule() {    for { //调度循环          // 根据某种算法从M个goroutine中找出一个需要运行的goroutine          g := find_a_runnable_goroutine_from_M_goroutines()          run_g(g) // CPU运行该goroutine，直到需要调度其它goroutine才返回          save_status_of_g(g) // 保存goroutine的状态，主要是寄存器的值      }}
```

这段伪代码表达的意思是，程序运行起来之后创建了 N 个由内核调度的操作系统线程（为了方便描述，我们称这些系统线程为**工作线程**）去执行 shedule 函数，而 schedule 函数在一个**调度循环**中反复从 M 个 goroutine 中挑选出一个需要运行的 goroutine 并跳转到该 goroutine 去运行，直到需要调度其它 goroutine 时才返回到 schedule 函数中通过 save_status_of_g 保存刚刚正在运行的 goroutine 的状态然后再次去寻找下一个 goroutine。

##### GM 模型

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/j3gficicyOvatXEp81ac9kdSiaAdzNAZyfAduWEyjBTwd2EpEmT2Q4lodTRVsTZf1zzP1xlF0t1QhwX55YFgtKIFQ/640?wx_fmt=gif&from=appmsg)

在 Go 1.1 版本之前，其实用的就是`GM`模型。GM 模型的调度逻辑和上面讲到的简化版模型非常类似，是一种**多对多**的模型，go 程序底层使用了多个操作系统线程，同时在 go 语言层面实现了语言级的轻量级协程 goroutine（对操作系统来说是透明的，操作系统只知道切换线程并且执行线程上的代码)，每个操作系统线程都会不断的去全局队列中获取 goroutine 来执行。

*   **goroutine (G)**: goroutine 是 go 语言中的轻量级线程。它们由 go 运行时管理，创建和销毁的开销相对较小。用户可以通过 `go` 关键字轻松地启动一个新的 goroutine。
    
*   **操作系统线程 (M)**: M 代表操作系统线程，go 运行时使用这些线程来执行 goroutine。每个 M 线程可以在操作系统的线程池中运行，负责执行 goroutine 的代码。
    

##### GM 模型的缺点

1️⃣ 有一个全局队列带来了一个问题，因为从队列中获取 goroutine 必须要加锁，导致**锁的争用非常频繁**。尤其是在大量 goroutine 被调度的情况下，对性能的影响也会非常明显。

2️⃣ 每个线程在运行时都可能会遇到需要进行系统调用的情况。早期 GM 模型中每个 M 都关联了内存缓存（mcache）和其他的缓存（栈空间），但实际上只有正在运行的 go 代码的 M 才需要 mcache（阻塞在系统调用的 M 不需要 mcache）。运行 go 代码的 M 和系统调用阻塞的 M 比例大概在 1:100，这就导致了大量的资源消耗（每个 mcache 会占用到 2M）。

3️⃣ 造成延迟和额外的系统负载。比如当 G 中包含创建新协程的时候，M 创建了 G’，为了继续执行 G，需要把 G’交给 M’执行，也造成了很差的局部性，因为 G’和 G 是相关的，最好放在 M 上执行，而不是其他 M'。

##### GMP 模型

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvatXEp81ac9kdSiaAdzNAZyfAzz7ib2atwEN8g7g2d5X4Ua8icGB4PM4G9L6zAmagWjADqbMO1ImRiaVQQ/640?wx_fmt=png&from=appmsg)

基于**没有什么是加一个中间层不能解决的**思路，`golang`在原有的`GM`模型的基础上加入了一个调度器`P`，可以简单理解为是在`G`和`M`中间加了个中间层。于是就有了现在的`GMP`模型里。

*   `P` 的加入，还带来了一个**本地协程队列**，跟前面提到的**全局队列**类似，也是用于存放`G`，想要获取等待运行的`G`，会**优先**从本地队列里拿，访问本地队列**无需加锁**。而全局协程队列依然是存在的，但是功能被弱化，不到**万不得已**是不会去全局队列里拿`G`的。
    
*   `GM`模型里 M 想要运行`G`，直接去全局队列里拿就行了；`GMP`模型里，`M`想要运行`G`，就得先获取`P`，然后从 `P` 的本地队列获取 `G`。
    
*   新建 `G` 时，新`G`会优先加入到 `P` 的本地队列；如果本地队列满了，则会把本地队列中一半的 `G` 移动到全局队列。`P` 的本地队列为空时，就从全局队列里去取。
    
    ![](https://mmbiz.qpic.cn/sz_mmbiz_gif/j3gficicyOvatXEp81ac9kdSiaAdzNAZyfAGO1wAP0c3tsRyr7QXWpOJPTEHzFwZQ4KrQziakwSqfc3SqjqicAhCiaBw/640?wx_fmt=gif&from=appmsg)
    
*   新建 `G` 时，新`G`会优先加入到 `P` 的本地队列；如果本地队列满了，则会把本地队列中一半的 `G` 移动到全局队列。
    
*   `P` 的本地队列为空时，就从全局队列里去取。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvatXEp81ac9kdSiaAdzNAZyfAxRrV0RVMtJDeeCCECzSUv5zIVNE80wnBs96KDzcYzaY5fAnCsZE4Fg/640?wx_fmt=png&from=appmsg)

*   如果全局队列为空时，`M` 会从其他 `P` 的本地队列**偷（stealing）一半 G** 放到自己 `P` 的本地队列。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/j3gficicyOvatXEp81ac9kdSiaAdzNAZyfAJ9QzRL7LdHEicydlUula5n6uYZ6ZYzZZRgG9XI5GxXFNQibrnZw0NckQ/640?wx_fmt=gif&from=appmsg)

*   `M` 运行 `G`，`G` 执行之后，`M` 会从 `P` 获取下一个 `G`，不断重复下去。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/j3gficicyOvatXEp81ac9kdSiaAdzNAZyfAGaDVqKMEIt0sscXmZrsgv74QmanPOahHetSeDibeVxUtdekcntMQnZQ/640?wx_fmt=gif&from=appmsg)

##### 为什么要有 P

这时候就有会疑惑了，如果是想实现本地队列、Work Stealing 算法，那**为什么不直接在 M 上加呢，M 也照样可以实现类似的功能**。为什么又再加多一个 P 组件？结合 M（系统线程） 的定位来看，若这么做，有以下问题。

*   一般来讲，M 的数量都会多于 P。像在 Go 中，M 的数量最大限制是 10000，P 的默认数量的 CPU 核数。另外由于 M 的属性，也就是如果存在系统阻塞调用，阻塞了 M，又不够用的情况下，M 会不断增加。
    
*   M 不断增加的话，如果本地队列挂载在 M 上，那就意味着本地队列也会随之增加。这显然是不合理的，因为本地队列的管理会变得复杂，且 Work Stealing 性能会大幅度下降。
    
*   M 被系统调用阻塞后，我们是期望把他既有未执行的任务分配给其他继续运行的，而不是一阻塞就导致全部停止
    

#### 调度器数据结构概述

系统线程对 goroutine 的调度与内核对系统线程的调度原理是一样的，**实质都是通过保存和修改 CPU 寄存器的值来达到切换线程 / goroutine 的目的**。因此，为了实现对 goroutine 的调度，需要引入一个数据结构来保存 CPU 寄存器的值以及 goroutine 的其它一些状态信息，在 go 语言调度器源代码中，这个数据结构是一个名叫 **g 的结构体**，它保存了 goroutine 的所有信息，该结构体的每一个实例对象都代表了一个 goroutine。调度器代码可以通过 g 对象来对 goroutine 进行调度。

*   `当goroutine被调离CPU时，调度器代码负责把CPU寄存器的值保存在g对象的成员变量之中`
    
*   `当goroutine被调度起来运行时，调度器代码又负责把g对象的成员变量所保存的寄存器的值恢复到CPU的寄存器`
    

前面我们所讲的 G，M，P，在源码中均有与之对应的数据结构。

##### 重要的结构体

G M P 结构体定义于 src/runtime/runtime2.go。

###### g 结构体

```
type g struct {     // 记录该goroutine使用的栈，当前 goroutine 的栈内存范围 [stack.lo, stack.hi)    stack       stack     // 下面两个成员用于栈溢出检查，实现栈的自动伸缩，抢占调度也会用到stackguard0      stackguard0 uintptr       _panic       *_panic      _defer       *_defer      // 此goroutine正在被哪个工作线程执行    m            *m           // 存储 goroutine 的调度相关的数据    sched        gobuf      // schedlink字段指向全局运行队列中的下一个g， // 所有位于全局运行队列中的g形成一个链表 schedlink      guintptr     // 不涉及本篇内容的字段已剔除 ...}
```

下面看看 gobuf 结构体，主要在调度器保存或者恢复上下文的时候用到：

```
type gobuf struct {    // 栈指针，对应上文讲到的RSP寄存器的值    sp   uintptr    // 程序计数器，对应上文讲到的RIP寄存器    pc   uintptr    // 记录当前这个gobuf对象属于哪个goroutine    g    guintptr     // 系统调用的返回值    ret  sys.Uintreg // 保存CPU的rbp寄存器的值  bp   uintptr    ...}
```

stack 结构体主要用来记录 goroutine 所使用的栈的信息，包括栈顶和栈底位置：

```
// Stack describes a Go execution stack.// The bounds of the stack are exactly [lo, hi),// with no implicit data structures on either side.//用于记录goroutine使用的栈的起始和结束位置type stack struct {      lo uintptr    // 栈顶，指向内存低地址    hi uintptr    // 栈底，指向内存高地址}
```

在执行过程中，G 可能处于以下几种状态：

```
const (    //  刚刚被分配并且还没有被初始化    _Gidle = iota // 0     // 没有执行代码，没有栈的所有权，存储在运行队列中    _Grunnable // 1     // 可以执行代码，拥有栈的所有权，被赋予了内核线程 M 和处理器 P    _Grunning // 2     // 正在执行系统调用，拥有栈的所有权，没有执行用户代码，    // 被赋予了内核线程 M 但是不在运行队列上    _Gsyscall // 3     // 由于运行时而被阻塞，没有执行用户代码并且不在运行队列上，    // 但是可能存在于 Channel 的等待队列上    _Gwaiting // 4      // 表示当前goroutine没有被使用，没有执行代码，可能有分配的栈    _Gdead // 6      // 栈正在被拷贝，没有执行代码，不在运行队列上    _Gcopystack // 8     // 由于抢占而被阻塞，没有执行用户代码并且不在运行队列上，等待唤醒    _Gpreempted // 9     // GC 正在扫描栈空间，没有执行代码，可以与其他状态同时存在    _Gscan          = 0x1000     ...)
```

上面的状态看起来很多，但是实际上只需要关注下面几种就好了：

*   等待中：_ Gwaiting、_Gsyscall 和 _Gpreempted，这几个状态表示 G 没有在执行；
    
*   可运行：_Grunnable，表示 G 已经准备就绪，可以在线程运行;
    
*   运行中：_Grunning，表示 G 正在运行；
    

###### m 结构体

```
type m struct {    // g0主要用来记录工作线程使用的栈信息，在执行调度代码时需要使用这个栈 // 执行用户goroutine代码时，使用用户goroutine自己的栈，调度时会发生栈的切换    g0      *g                    // 线程本地存储 thread-local，通过TLS实现m结构体对象与工作线程之间的绑定，下文会详细介绍    tls           [6]uintptr   // thread-local storage (for x86 extern register)    // 当前运行的G    curg          *g       // current running goroutine    // 正在运行代码的P    p             puintptr // attached p for executing go code (nil if not executing go code)    nextp         puintptr    // 之前使用的P    oldp          puintptr    // 记录所有工作线程的一个链表    alllink       *m // on allm    schedlink     muintptr    // Linux平台thread的值就是操作系统线程ID    thread        uintptr // thread handle    freelink      *m      // on sched.freem    ...}
```

###### p 结构体

调度器中的处理器 P 是线程 M 和 G 的中间层，用于调度 G 在 M 上执行。

```
type p struct {    id          int32    // p 的状态    status      uint32      // 调度器调用会+1    schedtick   uint32     // incremented on every scheduler call    // 系统调用会+1    syscalltick uint32     // incremented on every system call    // 对应关联的 M    m           muintptr        mcache      *mcache    pcache      pageCache     // defer 结构池    deferpool    [5][]*_defer      deferpoolbuf [5][32]*_defer      // 可运行的 goroutine 队列，可无锁访问    runqhead uint32    runqtail uint32    runq     [256]guintptr    // 缓存可立即执行的 G    runnext guintptr     // 可用的 G 列表，G 状态等于 Gdead     gFree struct {        gList        n int32    }    ...}
```

P 的几个状态：

```
const (     // 表示P没有运行用户代码或者调度器     _Pidle = iota     // 被线程 M 持有，并且正在执行用户代码或者调度器    _Prunning     // 没有执行用户代码，当前线程陷入系统调用    _Psyscall    // 被线程 M 持有，当前处理器由于垃圾回收 STW 被停止    _Pgcstop     // 当前处理器已经不被使用    _Pdead)
```

###### schedt 结构体

```
type schedt struct {    ... // 锁，从全局队列获取G时需要使用到    lock mutex     // 空闲的 M 列表    midle        muintptr      // 空闲的 M 列表数量    nmidle       int32          // 下一个被创建的 M 的 id    mnext        int64      // 能拥有的最大数量的 M      maxmcount    int32        // 由空闲的p结构体对象组成的链表    pidle      puintptr // idle p's    // 空闲 p 数量    npidle     uint32    // 全局 runnable G 队列    runq     gQueue    runqsize int32       // 有效 dead G 的全局缓存.    gFree struct {        lock    mutex        stack   gList // Gs with stacks        noStack gList // Gs without stacks        n       int32    }     // sudog 结构的集中缓存    sudoglock  mutex    sudogcache *sudog     // defer 结构的池    deferlock mutex    deferpool [5]*_defer     ...}
```

###### 重要的全局变量

```
allgs     []*g     // 保存所有的gallm       *m    // 所有的m构成的一个链表，包括下面的m0allp       []*p    // 保存所有的p，len(allp) == gomaxprocsncpu             int32   // 系统中cpu核的数量，程序启动时由runtime代码初始化gomaxprocs int32   // p的最大值，默认等于ncpu，但可以通过GOMAXPROCS修改sched      schedt     // 调度器结构体对象，记录了调度器的工作状态m0  m       // 代表进程的主线程g0   g        // m0的g0，也就是m0.g0 = &g0
```

在程序初始化时，这些全变量都会被初始化为 0 值，指针会被初始化为 nil 指针，切片初始化为 nil 切片，int 被初始化为数字 0，结构体的所有成员变量按其本类型初始化为其类型的 0 值。所以程序刚启动时 allgs，allm 和 allp 都不包含任何 g,m 和 p。

##### 线程执行的代码是如何找到属于自己的那个 m 结构体实例对象的呢

前面我们说 GMP 模型中每个工作线程都有一个 m 结构体对象与之对应，但并未详细说明它们之间是如何对应起来的~ 如果只有一个工作线程，那么就只会有一个 m 结构体对象，问题就很简单，定义一个全局的 m 结构体变量就行了。可是我们有多个工作线程和多个 m 需要一一对应，这里就需要用到线程的本地存储了。

###### **线程本地存储（TLS）**

TLS 是一种机制，允许每个线程有自己的独立数据副本。这意味着多个线程可以同时运行而不会相互干扰，因为每个线程都可以访问自己的数据副本。

###### 寄存器中 **`fs` 段的作用 **

在 Linux 系统中，`fs` 段可以用于存储线程的 TLS 数据，通常通过 `fs` 段寄存器来访问。

###### go 语言中的使用

*   在 Go 语言的运行时（runtime）中，`fs` 段被用来存储与每个 goroutine 相关的线程局部数据。Go 的 GMP 模型（goroutine, M, P）中，`m` 结构体的 `tls` 字段通常会被设置为当前线程的 `fs` 段，以便快速访问线程局部存储。
    
*   通过将 `fs` 段与 `m` 结构体的 `tls` 字段关联，Go 可以高效地管理和访问每个 goroutine 的特定数据。
    

具体到 goroutine 调度器代码，**每个工作线程在刚刚被创建出来进入调度循环之前就利用线程本地存储机制为该工作线程实现了一个指向 m 结构体实例对象的私有全局变量**，这样在之后的代码中就使用该全局变量来访问自己的 m 结构体对象以及与 m 相关联的 p 和 g 对象。

有了上述数据结构以及工作线程与数据结构之间的映射机制，我们可以再丰富下前面讲到的初始调度模型：

```
// 程序启动时的初始化代码......for i := 0; i < N; i++ { // 创建N个操作系统线程执行schedule函数     create_os_thread(schedule) // 创建一个操作系统线程执行schedule函数}// 定义一个线程私有全局变量，注意它是一个指向m结构体对象的指针// ThreadLocal用来定义线程私有全局变量ThreadLocal self *m//schedule函数实现调度逻辑func schedule() {    // 创建和初始化m结构体对象，并赋值给私有全局变量self    self = initm()      for { //调度循环          if (self.p.runqueue is empty) {                 // 根据某种算法从全局运行队列中找出一个需要运行的goroutine                 g := find_a_runnable_goroutine_from_global_runqueue()           } else {                 // 根据某种算法从私有的局部运行队列中找出一个需要运行的goroutine                 g := find_a_runnable_goroutine_from_local_runqueue()           }          run_g(g) // CPU运行该goroutine，直到需要调度其它goroutine才返回          save_status_of_g(g) // 保存goroutine的状态，主要是寄存器的值     }}
```

仅仅从上面这个伪代码来看，我们完全不需要线程私有全局变量，只需在 schedule 函数中定义一个局部变量就行了。但真实的调度代码错综复杂，不光是这个 schedule 函数会需要访问 m，其它很多地方还需要访问它，所以需要使用全局变量来方便其它地方对 m 的以及与 m 相关的 g 和 p 的访问。

### 从 main 函数启动开始分析

下面我们通过一个简单的 go 程序入手分析 调度器的初始化，go routine 的创建与退出，工作线程的调度循环以及 goroutine 的切换。

```
package mainimport "fmt"func main() {    fmt.Println("Hello World!")}
```

#### 程序入口

linux amd64 系统的启动函数是在 asm_amd64.s 的 runtime·rt0_go 函数中。当然，不同的平台有不同的程序入口。rt0_go 函数完成了 go 程序启动时的所有初始化工作，因此这个函数比较长，也比较繁杂，但这里我们只关注与调度器相关的一些初始化，下面我们分段来看：

```
TEXT runtime·rt0_go(SB),NOSPLIT|NOFRAME|TOPFRAME,$0    // copy arguments forward on an even stack    MOVQ   DI, AX    // 这行代码将寄存器 `DI` 的值（通常是命令行参数的数量，即 `argc`）复制到寄存器 `AX` 中。`AX` 现在存储了程序的参数个数    MOVQ   SI, BX    // 这行代码将寄存器 `SI` 的值（通常是指向命令行参数字符串数组的指针，即 `argv`）复制到寄存器 `BX` 中。`BX` 现在存储了指向命令行参数的指针。    SUBQ   $(5*8), SP    // 这行代码从栈指针 `SP` 中减去 `40` 字节（`5*8`），为局部变量和函数参数分配空间。这里的 `5` 可能表示 3 个参数和 2 个自动变量（局部变量）。这行代码的目的是在栈上为这些变量留出空间。    ANDQ   $~15, SP // 这行代码将栈指针 `SP` 对齐到 16 字节的边界。确保栈在函数调用时是对齐的    MOVQ   AX, 24(SP) // 这行代码将 `AX` 中的值（即 `argc`）存储到栈上相对于 `SP` 的偏移量 +`24` 的位置。    MOVQ   BX, 32(SP) // - 这行代码将 `BX` 中的值（即 `argv`）存储到栈上相对于 `SP` 的偏移量 +`32` 的位置。
```

上面的第 4 条指令用于调整栈顶寄存器的值使其按 16 字节对齐，也就是让栈顶寄存器 SP 指向的内存的地址为 16 的倍数，最后两条指令把 argc 和 argv 搬到新的位置。

#### 初始化 g0

继续看后面的代码，下面开始初始化全局变量 g0，前面我们说过，g0 的主要作用是提供一个栈供 runtime 代码执行，因此这里主要对 g0 的几个与栈有关的成员进行了初始化，从这里可以看出 g0 的栈大约有 64K。

```
从系统线程的栈中划分出一部分作为g0的栈,然后初始化g0的栈信息和stackgard MOVQ $runtime·g0(SB), DI  // 把g0的地址放入寄存器DI LEAQ (-64*1024)(SP), BX   // 设置寄存器BX的值为 SP（主线程栈栈顶指针） - 64k 的位置 MOVQ BX, g_stackguard0(DI) // g0.stackguard0 = BX ， 也就是设置g0.stackguard0 指向主线程栈的栈顶-64k的位置 MOVQ BX, g_stackguard1(DI) //g0.stackguard1 = SP - 64k MOVQ BX, (g_stack+stack_lo)(DI) //g0.stack_lo = SP - 64k MOVQ SP, (g_stack+stack_hi)(DI) //g0.stack_lo = SP
```

运行完上面这几行指令后 g0 与栈之间的关系如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvatXEp81ac9kdSiaAdzNAZyfA3HUibPKd3WN78gyY7n6gnCmzGvOA4byY2RjgrXaXgyCbWds8GNDiagFg/640?wx_fmt=png&from=appmsg)

### 主线程与 m0 绑定

设置好 g0 栈之后，我们跳过 CPU 型号检查以及 cgo 初始化相关的代码，接着分析如何把 m 数据结构 和 线程绑定在一起，原因在上面已描述过：每个线程需要能快速找到自己所属的 m 结构体。

```
LEAQ runtime·m0+m_tls(SB), DI    // DI = &m0.tls，取m0的tls成员的地址到DI寄存器 CALL runtime·settls(SB)  // 调用settls设置线程本地存储，settls函数的参数在DI寄存器中 // store through it, to make sure it works // 验证settls是否可以正常工作，如果有问题则abort退出程序 get_tls(BX)  //获取fs段基地址并放入BX寄存器，其实就是m0.tls[1]的地址，get_tls的代码由编译器生成 MOVQ $0x123, g(BX) //把整型常量0x123设置到线程本地存储中 MOVQ runtime·m0+m_tls(SB), AX  //获取m.tls结构体的地址到AX寄存器中 CMPQ AX, $0x123 // 判断m.tls[0]的值是否等于123，是的话说明tls工作正常 JEQ 2(PC) CALL runtime·abort(SB)
```

设置 tls 的函数 runtime·settls(SB) 位于源码 `src/runtime/sys_linux_amd64.s` 处，主要内容就是通过一个系统调用将 fs 段基址设置成 m.tls[1] 的地址，而 fs 段基址又可以通过 CPU 里的寄存器 fs 来获取。这段代码运行后，工作线程代码就可以通过 CPU 的 fs 寄存器来找到 m.tls。

#### m0 和 g0 绑定

```
ok: // set the per-goroutine and per-mach "registers" get_tls(BX) //获取fs段基址到BX寄存器 LEAQ runtime·g0(SB), CX //CX = g0的地址 MOVQ CX, g(BX)  //把g0的地址保存在线程本地存储里面，也就是m0.tls[0]=&g0 LEAQ runtime·m0(SB), AX //AX = m0的地址 // 下面把m0和g0关联起来m0->g0 = g0，g0->m = m0 // save m->g0 = g0  // save m->g0 = g0 MOVQ CX, m_g0(AX) //m0.g0 = g0 // save m0 to g0->m MOVQ AX, g_m(CX) //g0.m = m0
```

上面的代码首先把 g0 的地址放入主线程的线程本地存储中，然后把 m0 和 g0 绑定在一起，这样，之后在主线程中**通过 get_tls 可以获取到 g0**，通过 g0 的 m 成员又可以找到 m0，于是这里就实现了 m0 和 g0 与主线程之间的关联。

从这里还可以看到，保存在主线程本地存储中的值是 g0 的地址，也就是说工作线程的私有全局变量其实是一个指向 g 的指针而不是指向 m 的指针，目前这个指针指向 g0，表示代码正运行在 g0 栈。此时，主线程，m0，g0 以及 g0 的栈之间的关系如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvatXEp81ac9kdSiaAdzNAZyfAF7iaiaLMy79fx5kKkCdEibuCP7vBicstAxLkxWibeb00Llu3Bfsm4lSsSXw/640?wx_fmt=png&from=appmsg)

#### 初始化 m0

```
CALL runtime·check(SB) MOVL 24(SP), AX  // copy argc MOVL AX, 0(SP) MOVQ 32(SP), AX  // copy argv MOVQ AX, 8(SP) CALL runtime·args(SB) CALL runtime·osinit(SB)  // 调度器初始化 CALL runtime·schedinit(SB) // 新建一个 goroutine，该 goroutine 绑定 runtime.main MOVQ $runtime·mainPC(SB), AX  // entry PUSHQ AX CALL runtime·newproc(SB) POPQ AX // 启动M，开始调度循环，运行刚刚创建的goroutine CALL runtime·mstart(SB) // 上面的mstart永远不应该返回的，如果返回了，一定是代码逻辑有问题，直接abort CALL runtime·abort(SB) // mstart should never return RET
```

上面的 CALL 方法中：

*   schedinit 进行各种运行时组件初始化工作，这包括我们的调度器与内存分配器、回收器的初始化；
    
*   newproc 负责根据主 G 入口地址创建可被运行时调度的执行单元；
    
*   mstart 开始启动调度器的调度循环；
    

#### 调度器初始化（schedinit）

```
func schedinit() {// raceinit must be the first call to race detector.// In particular, it must be done before mallocinit below calls racemapshadow.       //getg函数在源代码中没有对应的定义，由编译器插入类似下面两行代码    //get_tls(CX)    //MOVQ g(CX), BX; BX存器里面现在放的是当前g结构体对象的地址    _g_ := getg() // _g_ = &g0    ......    //设置最多启动10000个操作系统线程，也是最多10000个M    sched.maxmcount = 10000    ......       mcommoninit(_g_.m) //初始化m0，因为从前面的代码我们知道g0->m = &m0    ......    sched.lastpoll = uint64(nanotime())    procs := ncpu  //系统中有多少核，就创建和初始化多少个p结构体对象    if n, ok := atoi32(gogetenv("GOMAXPROCS")); ok && n > 0 {        procs = n //如果环境变量指定了GOMAXPROCS，则创建指定数量的p    }    if procresize(procs) != nil {//创建和初始化全局变量allp        throw("unknown runnable goroutine during bootstrap")    }    ......}
```

前面我们已经看到，g0 的地址已经被设置到了线程本地存储之中，schedinit 通过 getg 函数（getg 函数是编译器实现的，我们在源代码中是找不到其定义的）从线程本地存储中获取当前正在运行的 g，这里获取出来的是 g0，然后调用 mcommoninit 函数对 m0(g0.m) 进行必要的初始化，对 m0 初始化完成之后调用 procresize 初始化系统需要用到的 p 结构体对象，p 就是 processor 的意思，**它的数量决定了最多可以有多少个 goroutine 同时并行运行**。

schedinit 函数除了初始化 m0 和 p，还设置了全局变量 sched 的 maxmcount 成员为 10000，限制最多可以创建 10000 个操作系统线程出来工作。

##### M0 初始化

```
func mcommoninit(mp *m, id int64) {    _g_ := getg()    ...    lock(&sched.lock)    // 如果传入id小于0，那么id则从mReserveID获取，初次从mReserveID获取id为0    if id >= 0 {        mp.id = id    } else {        mp.id = mReserveID()    }    //random初始化，用于窃取 G    mp.fastrand[0] = uint32(int64Hash(uint64(mp.id), fastrandseed))    mp.fastrand[1] = uint32(int64Hash(uint64(cputicks()), ^fastrandseed))    if mp.fastrand[0]|mp.fastrand[1] == 0 {        mp.fastrand[1] = 1    }    // 创建用于信号处理的gsignal，只是简单的从堆上分配一个g结构体对象,然后把栈设置好就返回了    mpreinit(mp)    if mp.gsignal != nil {        mp.gsignal.stackguard1 = mp.gsignal.stack.lo + _StackGuard    }    // 把 M 挂入全局链表allm之中    mp.alllink = allm    ...}
```

这里传入的 id 是 - 1，初次调用会将 id 设置为 0，这里并未对 m0 做什么关于调度相关的初始化，所以**可以简单的认为这个函数只是把 m0 放入全局链表 allm 之中就返回了**。

m0 完成基本的初始化后，继续调用 procresize 创建和初始化 p 结构体对象，在这个函数里面会创建指定个数（根据 cpu 核数或环境变量确定）的 p 结构体对象放在全变量 allp 里, 并把 m0 和 allp[0] 绑定在一起，因此当这个函数执行完成之后就有。

```
m0.p = allp[0]allp[0].m = &m0
```

**到这里 m0, g0, 和 m 需要的 p 完全关联在一起了**

##### P 初始化

由于用户代码运行过程中也支持通过 GOMAXPROCS() 函数调用 procresize 来重新创建和初始化 p 结构体对象，而在运行过程中再动态的调整 p 牵涉到的问题比较多，所以这个函数的处理比较复杂，这里只保留了初始化时会执行的代码。

```
func procresize(nprocs int32) *p {    old := gomaxprocs //系统初始化时 gomaxprocs = 0    ......    // Grow allp if necessary.    if nprocs > int32(len(allp)) { //初始化时 len(allp) == 0        // Synchronize with retake, which could be running        // concurrently since it doesn't run on a P.        lock(&allpLock)        if nprocs <= int32(cap(allp)) {            allp = allp[:nprocs]        } else {    //初始化时进入此分支，创建allp 切片            nallp := make([]*p, nprocs)            // Copy everything up to allp's cap so we            // never lose old allocated Ps.            copy(nallp, allp[:cap(allp)])            allp = nallp        }        unlock(&allpLock)    }    // initialize new P's    //循环创建nprocs个p并完成基本初始化    for i := int32(0); i < nprocs; i++ {        pp := allp[i]        if pp == nil {            pp = new(p)//调用内存分配器从堆上分配一个struct p            pp.id = i            pp.status = _Pgcstop            ......            atomicstorep(unsafe.Pointer(&allp[i]), unsafe.Pointer(pp))        }        ......    }    ......    _g_ := getg()  // _g_ = g0    if _g_.m.p != 0 && _g_.m.p.ptr().id < nprocs {//初始化时m0->p还未初始化，所以不会执行这个分支        // continue to use the current P        _g_.m.p.ptr().status = _Prunning        _g_.m.p.ptr().mcache.prepareForSweep()    } else {  //初始化时执行这个分支        // release the current P and acquire allp[0]        if _g_.m.p != 0 {//初始化时这里不执行            _g_.m.p.ptr().m = 0        }        _g_.m.p = 0        _g_.m.mcache = nil        p := allp[0]        p.m = 0        p.status = _Pidle        acquirep(p) //把p和m0关联起来，其实是这两个strct的成员相互赋值        if trace.enabled {            traceGoStart()        }    }       //下面这个for 循环把所有空闲的p放入空闲链表    var runnablePs *p    for i := nprocs - 1; i >= 0; i-- {        p := allp[i]        if _g_.m.p.ptr() == p {//allp[0]跟m0关联了，所以是不能放任            continue        }        p.status = _Pidle        if runqempty(p) {//初始化时除了allp[0]其它p全部执行这个分支，放入空闲链表            pidleput(p)        } else {            ......        }    }    ......       return runnablePs}
```

这里总结一下这个函数的主要流程：

1.  使用 make([]*p, nprocs) 初始化全局变量 allp，即 allp = make([]*p, nprocs)
    
2.  循环创建并初始化 nprocs 个 p 结构体对象并依次保存在 allp 切片之中
    
3.  把 m0 和 allp[0] 绑定在一起，即 m0.p = allp[0], allp[0].m = m0
    
4.  把除了 allp[0] 之外的所有 p 放入到全局变量 sched 的 pidle 空闲队列之中
    

下面我们用图来总结下整个调度器各部分的组成：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvatXEp81ac9kdSiaAdzNAZyfAP7uDcD3be1bbKHG0tq4wUibYwQBdJQiaC4Gry0eJdjQibOfjgibAsTpRTQ/640?wx_fmt=png&from=appmsg)

#### goroutine 的创建（newproc）

经过上文介绍我们介绍 m0 初始化中有说到，初始化过程中会新建一个 goroutine，该 goroutine 绑定 runtime.main，而 runtime.main 实际上最后会走到我们实现的 main 函数上。**新建 goroutine 的操作就是通过 newproc() 调用来实现的**。

newproc 函数用于创建新的 goroutine，它有两个参数，先说第二个参数 fn，新创建出来的 goroutine 将从 fn 这个函数开始执行，而这个 fn 函数可能也会有参数，newproc 的第一个参数正是 fn 函数的参数以字节为单位的大小。比如有如下 go 代码片段：

```
func start(a, b, c int64) {    ......}func main() {    go start(1, 2, 3)}
```

编译器在编译上面的 go 语句时，就会把其替换为对 newproc 函数的调用，编译后的代码逻辑上等同于下面的伪代码。

```
func main() {    push 0x3    push 0x2    push 0x1    runtime.newproc(24, start)}
```

可以看到编译器会帮我们把三个参数 1，2，3 分别压栈作为 start 函数的参数，然后再调用 newproc 函数。我们会注意到 newproc 函数本身还需要两个参数，第一个是 24，表示 start 函数需要 24 个字节大小的参数 为什么需要传递 start 函数的参数大小给到 newproc 函数呢? 这里是因为新建 goroutine 会在堆上创建一个全新的栈，需要**把 start 需要用到的参数先从当前 goroutine 的栈上拷贝到新的 goroutine 的栈上**之后才能让其开始执行，而 newproc 函数本身并不知道需要拷贝多少数据到新创建的 goroutine 的栈上去，所以需要用参数的方式指定拷贝多少数据。

newproc 函数是对 newproc1 的一个包装，这里最重要的准备工作有两个，一个是获取 fn 函数第一个参数的地址（代码中的 argp），另一个是使用 systemstack 函数切换到 g0 栈，当然，对于我们这个初始化场景来说现在本来就在 g0 栈，所以不需要切换，然而这个函数是通用的，在用户的 goroutine 中也会创建 goroutine，这时就需要进行栈的切换。

```
// Create a new g running fn with siz bytes of arguments.// Put it on the queue of g's waiting to run.// The compiler turns a go statement into a call to this.// Cannot split the stack because it assumes that the arguments// are available sequentially after &fn; they would not be// copied if a stack split occurred.//go:nosplitfunc newproc(siz int32, fn *funcval) {    //函数调用参数入栈顺序是从右向左，而且栈是从高地址向低地址增长的    //注意：argp指向fn函数的第一个参数，而不是newproc函数的参数    //参数fn在栈上的地址+8的位置存放的是fn函数的第一个参数    argp := add(unsafe.Pointer(&fn), sys.PtrSize)    gp := getg()  //获取正在运行的g，初始化时是m0.g0       //getcallerpc()返回一个地址，也就是调用newproc时由call指令压栈的函数返回地址，    //对于我们现在这个场景来说，pc就是CALLruntime·newproc(SB)指令后面的POPQ AX这条指令的地址    pc := getcallerpc()       //systemstack的作用是切换到g0栈执行作为参数的函数    //我们这个场景现在本身就在g0栈，因此什么也不做，直接调用作为参数的函数    systemstack(func() {        newproc1(fn, (*uint8)(argp), siz, gp, pc)    })}
```

newproc1 函数的第一个参数 fn 是新创建的 goroutine 需要执行的函数，注意这个 fn 的类型是 funcval 结构体类型，其定义如下：

```
type funcval struct {    fn uintptr    // variable-size, fn-specific data here}
```

第二个参数 argp 是 fn 函数的第一个参数的地址，第三个参数是 fn 函数的参数以字节为单位的大小，后面两个参数我们不用关心。这里需要注意的是，newproc1 是在 g0 的栈上执行的。

```
// Create a new g running fn with narg bytes of arguments starting// at argp. callerpc is the address of the go statement that created// this. The new g is put on the queue of g's waiting to run.func newproc1(fn *funcval, argp *uint8, narg int32, callergp *g, callerpc uintptr) {    //因为已经切换到g0栈，所以无论什么场景都有 _g_ = g0，当然这个g0是指当前工作线程的g0    //对于我们这个场景来说，当前工作线程是主线程，所以这里的g0 = m0.g0    _g_ := getg()    ......    _p_ := _g_.m.p.ptr() //初始化时_p_ = g0.m.p，从前面的分析可以知道其实就是allp[0]    newg := gfget(_p_) //从p的本地缓冲里获取一个没有使用的g，初始化时没有，返回nil    if newg == nil {         //new一个g结构体对象，然后从堆上为其分配栈，并设置g的stack成员和两个stackgard成员        newg = malg(_StackMin)        casgstatus(newg, _Gidle, _Gdead) //初始化g的状态为_Gdead         //放入全局变量allgs切片中        allgadd(newg) // publishes with a g->status of Gdead so GC scanner doesn't look at uninitialized stack.    }       ......       //调整g的栈顶置针，无需关注    totalSize := 4*sys.RegSize + uintptr(siz) + sys.MinFrameSize // extra space in case of reads slightly beyond frame    totalSize += -totalSize & (sys.SpAlign - 1)                  // align to spAlign    sp := newg.stack.hi - totalSize    spArg := sp    //......       if narg > 0 {         //把参数从执行newproc函数的栈（初始化时是g0栈）拷贝到新g的栈        memmove(unsafe.Pointer(spArg), unsafe.Pointer(argp), uintptr(narg))        // ......    }
```

这段代码主要从堆上分配一个 g 结构体对象并为这个 newg 分配一个大小为 2048 字节的栈，并设置好 newg 的 stack 成员，然后把 newg 需要执行的函数的参数从执行 newproc 函数的栈（初始化时是 g0 栈）拷贝到 newg 的栈，完成这些事情之后 newg 的状态如下图所示：这里需要注意的是，**新创建出来的 g 都是在堆上分配内存的**，主要有以下几个原因:

1.  **生命周期**：goroutine 的生命周期可能会超过创建它的函数的生命周期。如果在栈上分配 goroutine，那么当创建 goroutine 的函数返回时，goroutine 可能还在运行，这将导致栈被回收，从而引发错误。而在堆上分配 goroutine 可以避免这个问题，因为堆上的内存只有在没有任何引用指向它时才会被垃圾回收。
    
2.  **大小**：goroutine 的栈大小是动态的，它可以根据需要进行扩展和收缩。如果在栈上分配 goroutine，那么每次 goroutine 的栈大小改变时，都需要移动整个 goroutine 的内存，这将导致大量的性能开销。而在堆上分配 goroutine 可以避免这个问题，因为堆上的内存可以动态地进行扩展和收缩。
    
3.  **并发**：在 go 语言中，可以同时运行多个 goroutine。如果在栈上分配 goroutine，那么每个线程的栈大小都需要足够大，以容纳所有的 goroutine，这将导致大量的内存浪费。而在堆上分配 goroutine 可以避免这个问题，因为堆是所有线程共享的，因此可以更有效地利用内存。因此，出于生命周期、大小和并发等考虑，GMP 模型中新创建的 goroutine 都是在堆上分配内存的。下面让我们用一个图示来总结下目前的 GMP 模型中各个部分的情况。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvatXEp81ac9kdSiaAdzNAZyfAPh3TKgiblPDW1fOH1iazleUo0DUn7SuFOwY7vfw4GuriawfoeziaARro1w/640?wx_fmt=png&from=appmsg)

继续分析 newproc1()：

```
//把newg.sched结构体成员的所有成员设置为0    memclrNoHeapPointers(unsafe.Pointer(&newg.sched), unsafe.Sizeof(newg.sched))       //设置newg的sched成员，调度器需要依靠这些字段才能把goroutine调度到CPU上运行。    newg.sched.sp = sp  //newg的栈顶    newg.stktopsp = sp    //newg.sched.pc表示当newg被调度起来运行时从这个地址开始执行指令    //把pc设置成了goexit这个函数偏移1（sys.PCQuantum等于1）的位置，    //至于为什么要这么做需要等到分析完gostartcallfn函数才知道    newg.sched.pc = funcPC(goexit) + sys.PCQuantum // +PCQuantum so that previous instruction is in same function    newg.sched.g = guintptr(unsafe.Pointer(newg))    gostartcallfn(&newg.sched, fn) //调整sched成员和newg的栈// adjust Gobuf as if it executed a call to fn// and then did an immediate gosave.func gostartcallfn(gobuf *gobuf, fv *funcval) {    var fn unsafe.Pointer    if fv != nil {        fn = unsafe.Pointer(fv.fn) //fn: goroutine的入口地址，初始化时对应的是runtime.main    } else {        fn = unsafe.Pointer(funcPC(nilfunc))    }    gostartcall(gobuf, fn, unsafe.Pointer(fv))}// adjust Gobuf as if it executed a call to fn with context ctxt// and then did an immediate gosave.func gostartcall(buf *gobuf, fn, ctxt unsafe.Pointer) {    sp := buf.sp //newg的栈顶，目前newg栈上只有fn函数的参数，sp指向的是fn的第一参数    if sys.RegSize > sys.PtrSize {        sp -= sys.PtrSize        *(*uintptr)(unsafe.Pointer(sp)) = 0    }    sp -= sys.PtrSize //为返回地址预留空间，    //这里在伪装fn是被goexit函数调用的，使得fn执行完后返回到goexit继续执行，从而完成清理工作    *(*uintptr)(unsafe.Pointer(sp)) = buf.pc //在栈上放入goexit+1的地址    buf.sp = sp //重新设置newg的栈顶寄存器    //这里才真正让newg的ip寄存器指向fn函数，注意，这里只是在设置newg的一些信息，newg还未执行，    //等到newg被调度起来运行时，调度器会把buf.pc放入cpu的IP寄存器，    //从而使newg得以在cpu上真正的运行起来    buf.pc = uintptr(fn)    buf.ctxt = ctxt}
```

gostartcall 函数的主要作用有两个：

1.  调整 newg 的栈空间，把 goexit 函数的第二条指令的地址入栈，伪造成 goexit 函数调用了 fn，从而使 fn 执行完成后执行 ret 指令时返回到 goexit 继续执行完成最后的清理工作；
    
2.  **重新设置 newg.buf.pc 为需要执行的函数的地址，即 fn，我们这个场景为 runtime.main 函数的地址，如果是在运行中 go aa() 启动的协程，那么 newg.buf.pc 会为 aa() 函数的地址**。
    

```
//newg真正从哪里开始执行并不依赖于这个成员，而是sched.pc    newg.startpc = fn.fn      ......       //设置g的状态为_Grunnable，表示这个g代表的goroutine可以运行了    casgstatus(newg, _Gdead, _Grunnable)    ......       //把newg放入_p_的运行队列，初始化的时候一定是p的本地运行队列，其它时候可能因为本地队列满了而放入全局队列    runqput(_p_, newg, true)    ......}
```

这时 newg 也就是 main goroutine 的状态如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvatXEp81ac9kdSiaAdzNAZyfAicia4DXrDGpkwN31w0H4aPeu4MQRYQrnL4v6VyzkCtQH6nPhV2gDxImw/640?wx_fmt=png&from=appmsg)

可以看到 newproc 执行完毕时，p, m0, g0, newg, allp 的内存均已分配好且它们之间的关系也通过指针挂钩上，我们留意下 newg 的堆栈中目前栈顶是 goexit+1 这个位置的返回地址；目前 newg 尚未被调度起来运行，只是刚加入 p 的本地队列，后续该 newg 被 cpu 调度到时，cpu sp 寄存器会指向栈顶，在 newg 自身的逻辑开始执行后，newg 的栈内存会被不断使用，SP 寄存器不断移动来指示栈的内存的增长与回收，当所有逻辑执行完时，sp 寄存器重新指回这个 goexit+1 这个位置。最后弹出该位置的值作为 cpu rip 寄存器的值，从而去执行 runtime.goexit1(SB) 这个命令，继续进行调度循环（这个后面会讲到）。

下面我们总结下 newproc 做了什么事情:

1.  在堆上给新 goroutine 分配一段内存作为栈空间，设置堆栈信息到新 goroutine 对应的 g 结构体上，核心是设置 gobuf.pc 指向要执行的代码段，待调度到该 g 时，会将保存的 pc 值设置到 cpu 的 RIP 寄存器上从而去执行该 goroutine 对应的代码。
    
2.  把传递给 goroutine 的参数从当前栈拷贝到新 goroutine 所在的栈上。
    
3.  把 g 加入到 p 的本地队列等待调度，如果本地队列满了会加入到全局队列（程序刚启动时只会加入到 p 的本地队列）。
    

#### 调度循环的启动（mstart）

前面我们完成了 `newproc` 函数，接下来是 runtime·rt0_go 中的最后一步，启动调度循环，即 mstart 函数。

```
func mstart0() {    gp := getg()    ...    // Initialize stack guard so that we can start calling regular    // Go code.    gp.stackguard0 = gp.stack.lo + stackGuard    gp.stackguard1 = gp.stackguard0     mstart1()    // Exit this thread.    if GOOS == "windows" || GOOS == "solaris" || GOOS == "plan9" || GOOS == "darwin" || GOOS == "aix" {        // Window, Solaris, Darwin, AIX and Plan 9 always system-allocate        // the stack, but put it in _g_.stack before mstart,        // so the logic above hasn't set osStack yet.        osStack = true    }}
```

`mstart` 函数设置了 stackguard0 和 stackguard1 字段后，就直接调用 mstart1() 函数，由于只分析 main goroutine 的启动，这里省略部分无关的代码：

```
func mstart1() {    // 启动过程时 _g_ = m0.g0    _g_ := getg() //getcallerpc()获取mstart1执行完的返回地址    //getcallersp()获取调用mstart1时的栈顶地址    save(getcallerpc(), getcallersp())    ...    // 进入调度循环。永不返回    schedule()}
```

save 函数非常重要，它主要做了这两个操作

```
gp.sched.pc = getcallerpc()
gp.sched.sp = getcallersp()
```

将 mstart 调用 mstart1 的返回地址以及当时的栈顶指针保存到 g0 的 sched 结构中，保存好后，我们可以看到现在的指针指向情况（注意红线部分） 。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvatXEp81ac9kdSiaAdzNAZyfAQ0SDt3Qkhp9EX2yJdHWYEcyzt4r39yGTffbYuokt7crhiahccFKRQ8Q/640?wx_fmt=png&from=appmsg)

这里设置好后，g0 对象的 sp 值就不会变化了，一直指向 mstart 函数的栈顶，后续每次切换回 g0 时，都会从 g0 对象的 sp 值中恢复寄存器 SP，从而切换到 g0 栈。

继续分析代码，先看下 schedule() 函数的逻辑，这是 GMP 模型调度逻辑的核心，每次调度 goroutine 都是从它开始的：

```
// One round of scheduler: find a runnable goroutine and execute it.// Never returns.func schedule() {    _g_ := getg()  //_g_ = 每个工作线程m对应的g0，初始化时是m0的g0    //......    var gp *g      //......        if gp == nil {        // Check the global runnable queue once in a while to ensure fairness.        // Otherwise two goroutines can completely occupy the local runqueue        // by constantly respawning each other.        //为了保证调度的公平性，每进行61次调度就需要优先从全局运行队列中获取goroutine，        //因为如果只调度本地队列中的g，那么全局运行队列中的goroutine将得不到运行        if _g_.m.p.ptr().schedtick%61 == 0 && sched.runqsize > 0 {            lock(&sched.lock) //所有工作线程都能访问全局运行队列，所以需要加锁            gp = globrunqget(_g_.m.p.ptr(), 1) //从全局运行队列中获取1个goroutine            unlock(&sched.lock)        }    }    if gp == nil {        //从与m关联的p的本地运行队列中获取goroutine        gp, inheritTime = runqget(_g_.m.p.ptr())        if gp != nil && _g_.m.spinning {            throw("schedule: spinning with local work")        }    }    if gp == nil {        //如果从本地运行队列和全局运行队列都没有找到需要运行的goroutine，        //则调用findrunnable函数从其它工作线程的运行队列中偷取，如果偷取不到，则当前工作线程进入睡眠，        //直到获取到需要运行的goroutine之后findrunnable函数才会返回。      gp, inheritTime = findrunnable() // blocks until work is available    }    //跟启动无关的代码.....    //当前运行的是runtime的代码，函数调用栈使用的是g0的栈空间    //调用execte切换到gp的代码和栈空间去运行    execute(gp, inheritTime)  }
```

schedule 函数优先从 p 本地队列获取 goroutine，获取不到时会去全局运行队列中加锁获取 goroutine，在我们的场景中，前面的启动流程已经创建好第一个 goroutine 并放入了当前工作线程的本地运行队列（即 runtime.main 对应的 goroutine）。获取到 g 后，会调用 execute 去切换到 g，具体的切换逻辑继续看下 execute 函数。

```
func execute(gp *g, inheritTime bool) {    _g_ := getg() //g0    //设置待运行g的状态为_Grunning    casgstatus(gp, _Grunnable, _Grunning)      //......        //把g和m关联起来    _g_.m.curg = gp     gp.m = _g_.m    //......    //gogo完成从g0到gp真正的切换    gogo(&gp.sched)}
```

这里的重点是 gogo 函数，真正完成了 g0 到 g 的切换，**切换的实质就是 CPU 寄存器以及函数调用栈的切换：**

```
TEXT runtime·gogo(SB), NOSPLIT, $16-8    //buf = &gp.sched    MOVQ  buf+0(FP), BX   # BX = buf      //gobuf->g --> dx register    MOVQ  gobuf_g(BX), DX  # DX = gp.sched.g      //下面这行代码没有实质作用，检查gp.sched.g是否是nil，如果是nil进程会crash死掉    MOVQ  0(DX), CX   # make sure g != nil      get_tls(CX)       //把要运行的g的指针放入线程本地存储，这样后面的代码就可以通过线程本地存储    //获取到当前正在执行的goroutine的g结构体对象，从而找到与之关联的m和p    MOVQ  DX, g(CX)      //把CPU的SP寄存器设置为sched.sp，完成了栈的切换(画重点！)    MOVQ  gobuf_sp(BX), SP  # restore SP      //下面三条同样是恢复调度上下文到CPU相关寄存器    MOVQ  gobuf_ret(BX), AX    MOVQ  gobuf_ctxt(BX), DX    MOVQ  gobuf_bp(BX), BP      //清空sched的值，因为我们已把相关值放入CPU对应的寄存器了，不再需要，这样做可以少gc的工作量    MOVQ  $0, gobuf_sp(BX)  # clear to help garbage collector    MOVQ  $0, gobuf_ret(BX)    MOVQ  $0, gobuf_ctxt(BX)    MOVQ  $0, gobuf_bp(BX)      //把sched.pc值放入BX寄存器    MOVQ  gobuf_pc(BX), BX      //JMP把BX寄存器的包含的地址值放入CPU的IP寄存器，于是，CPU跳转到该地址继续执行指令，    JMP BX
```

这个函数，其实就只做了两件事：

*   把 gp.sched 的成员恢复到 CPU 的寄存器完成状态以及栈的切换；
    
*   跳转到 gp.sched.pc 所指的指令地址（runtime.main）处执行。
    
    最后我们再总结一下程序开始运行后从 g0 栈切换到 main goroutine 栈的流程
    
    保存 g0 的调度信息，主要是保存 CPU 栈顶寄存器 SP 到 g0.sched.sp 成员之中；  
    调用 schedule 函数寻找需要运行的 goroutine，我们这个场景找到的是 main goroutine;  
    调用 gogo 函数首先从 g0 栈切换到 main goroutine 的栈，然后从 main goroutine 的 g 结构体对象之中取出 sched.pc 的值并使用 JMP 指令跳转到该地址去执行；
    

### go 的调度循环是什么

上文我们分析了 main goroutine 的启动，main 的 goroutine 和非 main 得 goroutine 稍微会有一点差别，主要在于 main goutine 对应的 runtime.main 函数，执行完毕后会直接在汇编代码中执行 exit 从而退出程序，而非 main goroutine 在执行完对应的逻辑后，会进入调度循环，不断找到下一个 goroutine 来执行。假设我们在代码中使用 go aa() 启动了一个协程，从 aa() 被开始调度到 aa 运行完后退出，是沿着这个路径来执行的。

```
schedule()->execute()->gogo()->aa()->goexit()->goexit1()->mcall()->goexit0()->schedule()
```

可以看出，一轮调度是从调用 schedule 函数开始的，然后经过一系列代码的执行到最后又再次通过调用 schedule 函数来进行新一轮的调度，从一轮调度到新一轮调度的这一过程我们称之为一个**调度循环**，这里说的调度循环是指某一个工作线程的调度循环，而同一个 Go 程序中可能存在多个工作线程，每个工作线程都有自己的调度循环，也就是说每个工作线程都在进行着自己的调度循环。

调度循环的细节这里就不再分析，本文就只介绍到协程调度的核心原理，相信看完本文你已经有所收获~ 最后让我们用一个图来了解下调度循环的大体流程:

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvatXEp81ac9kdSiaAdzNAZyfAGnPzYZwmsiaibwPDE8J8t6IFLQoNp2iaQAxLbTHN9V7aQNuFAc57WTs0Q/640?wx_fmt=png&from=appmsg)

**Reference**

1.  [详解 Go 语言调度循环源码实现](https://www.luozhiyun.com/archives/448)
    
2.  [go 语言调度器源代码情景分析之 go 汇编语言](https://mp.weixin.qq.com/s?__biz=MzU1OTg5NDkzOA==&mid=2247483704&idx=1&sn=a5e9f1fd2c0de42d5710afbb4553d411&scene=21#wechat_redirect)
    
3.  [go 语言调度器源分析之调度器初始化](https://mp.weixin.qq.com/s?__biz=MzU1OTg5NDkzOA==&mid=2247483769&idx=1&sn=3d77609a293d87e64639afc8d2219e1c&scene=21#wechat_redirect)
    
4.  [go 语言调度器源分析之创建 main goroutine](https://mp.weixin.qq.com/s?__biz=MzU1OTg5NDkzOA==&mid=2247483776&idx=1&sn=a74df39487b4ff22073e97eeb59d48ed&scene=21#wechat_redirect)
    
5.  [动图图解！GMP 模型里为什么要有 P](https://cloud.tencent.com/developer/article/1819618)
    
6.  [GMP 模型，为什么要有 P？](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247487503&idx=1&sn=bfc20f81a1c6059ca489733b31a2c63c&scene=21#wechat_redirect)
    
7.  [Scalable Go Scheduler Design Doc (可伸缩 Go 调度器设计)](https://docs.google.com/document/d/1TTj4T2JO42uD5ID9e89oa0sLKhJYD0Y_kqxDv3I3XMw/edit?tab=t.0)
    

[  
](https://docs.google.com/document/d/1TTj4T2JO42uD5ID9e89oa0sLKhJYD0Y_kqxDv3I3XMw/edit?tab=t.0)

[  
](https://docs.google.com/document/d/1TTj4T2JO42uD5ID9e89oa0sLKhJYD0Y_kqxDv3I3XMw/edit?tab=t.0)

**🎁 互动福利 🎁**

**评论区留言你使用 GO 语言的心得**

****抽取 3 位粉丝各送出一本书****

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvauE3G2kQExphdgialW9bd9ziarm7Jibp89WMjIvYzLMiaxz9IO0or5wxt07QYM24I8gvMzuUcjVX6LHww/640?wx_fmt=png&from=appmsg)

本书基于在读者之间广为传阅的同名开源电子书《Go 语言设计与实现》，是难得一见的 Go 语言进阶图书。 书中结合近 200 幅生动的全彩图片，配上详尽的文字剖析与精选源代码段，为读者奉上了异彩纷呈、系统完善的 Go 语言解读。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasVeMDmWoZ2zyN8iaSc6XWYj5q5PQEOc5ibURPb03vnRibrxC3UR8xzdyATfiawTYRV2vJvBnAIcE1FeQ/640?wx_fmt=png&from=appmsg)