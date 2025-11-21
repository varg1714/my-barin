---
source: https://mp.weixin.qq.com/s/lCP1AmJELgleravLCBrewA
create: 2025-10-11 15:27
read: false
knowledge: false
---

FSAC 未来超级架构师

架构师总动员  
实现架构转型，再无中年危机

## 尼恩说在前面

在 45 岁老架构师尼恩的**读者交流群** (50+) 中，最近有小伙伴拿到了一线互联网企业如得物、阿里、滴滴、极兔、有赞、希音、百度、网易、美团的面试资格，遇到很多很重要的面试题：

*   什么是 JIT，JIT 什么优势？什么是类的生命周期七个阶段 ？
*   什么是字节码增强？

最近有小伙伴在面试京东、阿里、希音等大厂，又遇到了相关的面试题。小伙伴没系统化学习，支支吾吾的说了几句，面试官不满意， **挂了**。

针对上面的面试题，接下来尼恩结合互联网上的一些实际案例，大家做一下系统化、体系化的梳理。使得大家内力猛增，可以充分展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer 直提”。

当然，这道面试题，以及参考答案，会收入咱们的《[尼恩 Java 面试宝典 PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》V175 版本，供后面的小伙伴参考，帮助大家进大厂 / 做架构。

最新《尼恩架构笔记》《尼恩高并发三部曲》《尼恩 Java 面试宝典》的 PDF，请关注本公众号【技术自由圈】获取，回复：领电子书

## 1、什么是字节码？

在 Java 之前，跨平台是一个难题。而 Java  跨平台的优势： “一次编译，到处运行” 。

构成 Java 生态跨平台的基石—— Java  字节码。

无论在何种环境下编译，Java 源代码都能生成标准格式的字节码（.class 文件），由 JVM 解释执行。

为啥要从字节码入手。

对开发者而言，了解字节码有助于更深入、直观地理解 Java 语言的底层机制。

例如，通过查看字节码，可以清楚地看到 `volatile` 关键字是如何生效的。

例如，字节码增强技术在 Spring AOP、ORM 框架及热部署等领域应用广泛，掌握其原理对我们大有裨益。

还有，由于 JVM 规范的标准性，任何能生成符合规范的字节码的程序都可以在 JVM 上运行，因此像 Scala、Groovy、Kotlin 等 JVM 语言也有机会扩展 Java 不具备的特性或实现更友好的语法糖。

从字节码的视角学习这些语言，往往能够更深刻地理解其设计思路。

我们通常使用 `javac` 命令将 `.java` 源文件编译为字节码文件

字节码文件由十六进制数值构成，JVM 按一组两个十六进制值（即一个字节）为单位进行读取。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDwkbiazpoYUH73NOic6YBBkldMEWWVTYoIm3NthaiaG3N5VNHxeY9xrsWw/640?from=appmsg&watermark=1#imgIndex=0)

## 2、字节码文件的结构

`.java` 文件经 `javac` 编译后生成 `.class` 文件。

编译后得到的 `ByteCodeDemo.class` 文件以十六进制形式打开后，表现形式如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDY6GPNNY3eS3OGZSSuQcXNHUgtFPhmxl6icZWw1iaps1iaYW9WibkRlqZ7A/640?from=appmsg&watermark=1#imgIndex=1)

JVM 规范要求字节码文件必须按以下十部分顺序构成：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDoa4LxqoP0VlGqt1MtC6RsclPWWiaHxM2LWan02tfyuXTQ7MyXa131SQ/640?from=appmsg&watermark=1#imgIndex=2)

**（1）魔数（Magic Number）**

每个 `.class` 文件的前 4 个字节是固定的魔数值：`0xCAFEBABE`。

JVM 通过该标志，快速判断文件是否为可能有效的字节码文件。

有趣的是，魔数 “CAFEBABE” 由 Java 之父 James Gosling 指定，意为“Cafe Babe”（咖啡宝贝），而 Java 的图标正是一杯咖啡。

**（2）版本号**

魔数之后的 4 个字节是版本号信息：前 2 个字节表示次版本号（Minor Version），后 2 个字节表示主版本号（Major Version）。

例如 `00 00 00 34` 表示次版本号为 0，主版本号为 52（十六进制 0x34），对应 Java 8。

**（3）常量池（Constant Pool）**

版本号之后是常量池，它是字节码文件中的资源仓库，主要存放两大类常量：

*   **字面量**

    如 `final` 修饰的常量值、文本字符串等；

*   **符号引用**

    包括类 / 接口的全限定名、字段名称与描述符、方法名称与描述符等。

常量池的结构分为两部分：

*   **常量池计数器（constant_pool_count）**

    2 个字节，表示常量池中的常量数量（实际数量为 `计数值 - 1`）；

*   **常量池数据区**

    由多个 `cp_info` 表项组成，每个 `cp_info` 对应一个常量。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDpvEtYV4cIIJOiaeBHc3vkYVUulS1Bmfm8DaUcOa4obnlFbf65lmgxuQ/640?from=appmsg&watermark=1#imgIndex=3)

JVM 共定义了 14 种 `cp_info` 类型，其通用结构如下：

```
    u1 tag;          // 常量类型标识
    u2 info_length;  // 后续信息长度（视类型而定）
    u1 bytes[info_length]; // 具体数据内容
}


```

通过 `javap -verbose ByteCodeDemo` 可查看反编译后的常量池内容，更直观易懂

**（4）访问标志（Access Flags）**

常量池后的 2 个字节是访问标志，用于标识类或接口的访问权限及属性，例如是否为 `public`、`final`、`abstract` 等。

JVM 使用位掩码表示多个标志组合，例如 `public final` 类对应的标志值为 `0x0001 | 0x0010 = 0x0011`。

**（5）当前类名**

访问标志后的 2 个字节是一个索引值，指向常量池中该类全限定名的字符串常量。

**（6）父类名称**

当前类名后的 2 个字节也是一个索引值，指向常量池中父类的全限定名字符串。

**（7）接口信息**

父类名称后是接口计数器（2 字节）及接口索引列表，列出所有实现的接口在常量池中的索引。

**（8）字段表（Fields Table）**

字段表用于描述类中声明的字段（类变量和实例变量），不包括方法内的局部变量。结构分为两部分：

*   字段计数器（2 字节）：表示字段个数；
*   字段信息表（field_info）：每个字段的详细信息。

字段信息结构如下：

```
    u2 access_flags;    // 访问标志，如private、static等
    u2 name_index;      // 字段名在常量池中的索引
    u2 descriptor_index;// 字段描述符（如I表示int）索引
    u2 attributes_count;// 属性个数
    attribute_info attributes[attributes_count]; // 属性表
}


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDdmrlLMdEfibAqqtfGx2iawYddcWtDjomZOa48Bm0V0P4vC87Dql40lpQ/640?from=appmsg&watermark=1#imgIndex=4)

案例中 `private int a`，0002 对应为 Private。

通过索引下标在常量池查询 #7 号常量，得到字段名为 “a”，描述符为 “I”（代表 int）。

综上，就可以唯一确定出一个类中声明的变量 `private int a`。

**（9）方法表（Methods Table）**

方法表存储所有方法的信息，也分为两部分：

*   方法计数器（2 字节）：表示方法个数；
*   方法信息表（method_info）：每个方法的详细信息。

方法信息结构如下：

```
    u2 access_flags;     // 访问标志，如public、synchronized等
    u2 name_index;       // 方法名在常量池中的索引
    u2 descriptor_index; // 方法描述符（如()V）索引
    u2 attributes_count; // 属性个数
    attribute_info attributes[attributes_count]; // 属性表
}


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDXZjbmibutZOP7dVWRxhN2093hodwfsq2C8zPsAz2wvLGqYWkC02l4Yw/640?from=appmsg&watermark=1#imgIndex=5)

通过 `javap -verbose` 可查看方法表的详细信息，可以看到主要包括以下三个部分：

*   “Code 区”：源代码对应的 JVM 指令操作码，在进行字节码增强时重点操作的就是 “Code 区” 这一部分。
*   “LineNumberTable”：行号表，将 Code 区的操作码和源代码中的行号对应，Debug 时会起到作用（源代码走一行，需要走多少个 JVM 指令操作码）。
*   “LocalVariableTable”：本地变量表，包含 This 和局部变量，之所以可以在每一个方法内部都可以调用 This，是因为 JVM 将 This 作为每一个方法的第一个参数隐式进行传入。当然，这是针对非 Static 方法而言。

**（10）附加属性表（Additional Attributes）**

字节码文件的最后部分可包含自定义属性信息，如源文件名称、注解信息等。

## 3、字节码指令集

JVM 字节码指令是堆栈导向的，操作码长度为一个字节（0x00~0xFF），最多支持 256 条指令。

Java 虚拟机规范定义了丰富的指令集，包括算术运算、类型转换、对象创建、方法调用等类别。

完整指令集可参考 Oracle 官方文档。

在上图中，Code 区的红色编号 0～17，就是. java 中的方法源代码编译后让 JVM 真正执行的操作码，也就是完整的 add() 方法的实现

## 4、操作数栈与字节码执行

JVM 的指令集是基于栈而不是寄存器，基于栈可以具备很好的跨平台性（因为寄存器指令集往往和硬件挂钩），

但缺点在于，要完成同样的操作，基于栈的实现需要更多指令才能完成（因为栈只是一个 FILO 结构，需要频繁压栈出栈）。

另外，由于栈是在内存实现的，而寄存器是在 CPU 的高速缓存区，相较而言，基于栈的速度要慢很多，这也是为了跨平台性而做出的牺牲。

**这也是为啥 java 比 go 慢的原因之一。**

JVM：基于栈的虚拟机，这个是 java 慢的一个核心原因之一

*   **JVM 字节码**

     是一种中间语言，运行在虚拟机上。

*   它的指令操作主要依赖于**操作数栈（operand stack）**，而不是寄存器。
*   例如，`iadd` 指令会从栈顶弹出两个整数，相加后再压回栈中。
*   优点：指令集简洁，跨平台，容易实现。
*   缺点：性能相对较低，因为频繁的栈操作。

Go 基于寄存器的机器码，这个是 Go 快的一个核心原因之一

*   Go 是**静态编译型语言**，编译器（如 `gc`）会将 Go 代码直接编译成**目标平台的机器码**（如 x86-64、ARM）。
*   这些机器码是**基于寄存器的**，因为现代 CPU 架构（如 x86、ARM）本身就是寄存器架构。
*   Go 编译器在生成代码时，会使用寄存器来存储变量、参数、返回值等，而不是像 JVM 那样用栈。
*   所以，Go 的 “指令集” 其实就是目标平台的机器指令集，是寄存器驱动的。

我们在上文所说的操作码或者操作集合，其实控制的就是这个 JVM 的操作数栈。

为了更直观地感受操作码是如何控制操作数栈的，以及理解常量池、变量表的作用，这里将 add() 方法的对操作数栈的操作制作为 GIF 动图，如下图所示。

GIF 动图中，仅截取了常量池中被引用的部分，以指令 iconst_2 开始到 ireturn 结束，与方法表 Code 区 0~17 的指令一一对应：

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpD5Y9iaicD9CQM5k2WzC6Po6U97NYbtr3xoaro7JZklJygzWPFf9TpMs0A/640?from=appmsg#imgIndex=6)

基于栈的设计使 JVM 具有良好的跨平台性，但执行性能可能不如基于寄存器的架构。

为了缓解性能问题，JVM 引入了 JIT（即时编译）技术，将热点代码编译为本地机器码执行。

理论上，  JIT 最终吐出来的东西一定是 “**基于寄存器的真正机器码**”，因为只有这样才能在物理 CPU 上直接跑。

但 JIT  不是一次性把整个 Java 字节码 “平移” 成机器码，而是先把**基于栈的 byte-code** 当输入，经过一次 “**栈→寄存器**” 的转换，再生成宿主 CPU 的寄存器指令。

下面把流程拆开说清：

**1、输入：基于栈的字节码**例：

```
iload_2
iadd
istore_3


```

语义：把局部变量 1、2 压栈 → 弹出两个 int 相加 → 结果存回局部变量 3。

全程没有寄存器名字，只有深度为 2 的 operand stack。

**2、JIT 编译器（C1/C2/Graal 等）做的核心事情**

a. **字节码解析 + 类型推导** → 得到一个有类型的 “栈机器” 中间表示（IR）。

b. **“栈调度”(stack-to-register allocation)**：把每条 push/pop 映射到虚拟寄存器（SSA 值）。

c. **全局寄存器分配**（图着色、线性扫描等）：把虚拟寄存器再映射到物理寄存器（x86 的 rax/rbx/rsi…）。d. **指令选择**：生成真正的 ADD、MOV、LEA… 指令。

e. **优化 + 重排 + 发射**：最终得到可在 CPU 上直接运行的机器码片段。

**3、输出：基于寄存器的机器码**

上面 4 条字节码可能被 JIT 编译成（x86-64）：

```
add    eax, [rbp+local_2]   ; eax += local_2
mov    [rbp+local_3], eax   ; local_3 = eax


```

已经没有 operand stack 的影子，完全使用寄存器与内存操作数。

JIT 的**输入**是 “基于栈的字节码”，**输出**是 “基于寄存器的本地机器码”；中间经过**把栈语义等价变换成寄存器语义**的编译过程。

## 5、查看字节码的工具

除了使用 `javap`命令反编译字节码外，推荐使用 IntelliJ IDEA 插件 jclasslib 可视化查看字节码细节。

安装后，通过菜单栏的 “View” → “Show Bytecode With jclasslib” 即可查看当前类的字节码信息，包括常量池、方法表、属性表等，界面直观友好。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDsBHynHt89RpAj64RteObAiatemsBx2A9bLyKGmL1QPmDcBfQ48ksjkA/640?from=appmsg&watermark=1#imgIndex=7)

掌握字节码结构与指令集，不仅有助于我们深入理解 Java 程序的运行机制，也是学习高级主题如字节码增强、性能优化的重要基础。

类的装载就是将字节码文件加载到 JVM 中的过程

## 1、类的入口点

在深入探讨 JVM 如何加载. class 文件之前，先回顾一下 C 语言的执行过程。

一个简单的 C 程序：

```
int main() {
    printf("Hello, World! \n");
    return 0;
}


```

编辑完保存为 hello.c 文本文件，然后安装 gcc 编译器（GNU C/C++）

```
$ ./a.out
Hello, World!


```

这个过程就是 gcc 编译器将 hello.c 文本文件编译成机器指令集，然后读取到内存直接在计算机的 CPU 运行。

从操作系统层面看的话，就是一个进程的启动到结束的生命周期。

而 Java 的执行方式，和 c 有所不同。

我们编写一个 HelloWorld 类：

```
    public static void main(String[] args) {
        System.out.println("my classLoader is " + HelloWorld.class.getClassLoader());
    }
}


```

需要通过两个步骤执行：

```
$ java HelloWorld              # 执行


```

这里的关键区别在于：执行 Java 程序时，操作系统启动的是 java 命令对应的进程（即 JVM），而 HelloWorld 类只是传递给这个进程的参数。

JVM 会寻找并执行指定类中的 main 方法作为程序入口。

Java 的 main 方法有严格的格式要求，必须是`public static void main(String[] args)`。

我们可以通过实验验证这些要求的必要性：

*   去掉 public 修饰符：JVM 会报错，因为需要访问权限
*   去掉 static 修饰符：JVM 会报错，因为需要能够不创建实例就调用
*   修改返回类型：JVM 会报错，因为需要统一的 void 返回类型
*   更改方法名：JVM 会报错，因为找不到约定的入口方法

这些规则确保了 JVM 能够以统一的方式启动任何 Java 程序。

从底层实现来看，java 命令（由 C++ 编写）通过 JNI（Java Native Interface）调用 Java 的 main 方法。

源码简化后的调用逻辑如下：

```
mainClassName = GetMainClassName(env, jarfile);
mainClass = LoadClass(env, classname);
mainID = (env)->GetStaticMethodID(env, mainClass, "main", "([Ljava/lang/String;)V");
(env)->CallStaticVoidMethod(env, mainClass, mainID, mainArgs);


```

## 2、类的生命周期七个阶段

当我们编写一个 Java 类并运行它时，这个类在 JVM 中会经历完整的生命周期。

理解这个过程对于掌握 JVM 工作原理至关重要。

一个类在 JVM 中的完整生命周期包括 7 个阶段：加载（Loading）、验证（Verification）、准备（Preparation）、解析（Resolution）、初始化（Initialization）、使用（Using）和卸载（Unloading）。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDNvlVgOgtko2dl8Naeia0ucvK6hWJUpHI489Qrx3uuibMFqTTIib0Ef7KQ/640?from=appmsg&watermark=1#imgIndex=8)

前五个阶段（加载到初始化）统称为类加载过程。

下面我们详细分析每个阶段的具体工作。

### 2.1 加载阶段（Loading）

加载是类生命周期的起点，主要任务是找到类的二进制表示并将其加载到内存中。

**核心操作：**

*   根据类的全限定名（如`java.lang.String`）定位 class 文件
*   读取 class 文件的二进制字节流
*   将字节流转换为方法区的运行时数据结构
*   在堆内存中生成一个代表该类的`java.lang.Class`对象，作为访问方法区数据的入口

**技术细节：**

加载阶段由类加载器（ClassLoader）具体执行。JVM 内置了三级类加载器：

*   启动类加载器（Bootstrap ClassLoader）：加载 JRE 核心库
*   扩展类加载器（Extension ClassLoader）：加载扩展库
*   应用类加载器（Application ClassLoader）：加载用户类路径上的类

如果找不到指定的类文件，JVM 会抛出`NoClassDefFoundError`。值得注意的是，加载阶段并不检查 class 文件的语法和格式是否正确，这些检查会在后续阶段进行。

### 2.2 验证阶段（Verification）

验证是连接过程的第一步，确保被加载的类符合 JVM 规范且不会危害虚拟机安全。

这个阶段就像是对进口商品进行海关检查，防止有害内容进入国内。

**验证内容包括：**

**(1) 文件格式验证：检查魔数（CAFEBABE）、版本号等基本信息**

**(2) 元数据验证：对类的元数据进行语义检查，如是否有父类、是否实现所有抽象方法等**

**(3) 字节码验证：通过数据流和控制流分析，验证程序逻辑的合理性**

**(4) 符号引用验证：检查常量池中的符号引用能否正确解析**

**可能抛出的异常：**

*   `VerifyError`

    验证失败

*   `ClassFormatError`

    类格式错误

*   `UnsupportedClassVersionError`

    版本不支持

验证过程中可能需要加载其他相关类，如果发现类层次结构问题（如循环继承），JVM 会抛出`ClassCircularityError`。

### 2.3 准备阶段（Preparation）

准备阶段为类变量分配内存并设置初始值。这些变量指的是被 static 修饰的静态变量，不包括实例变量。

**关键特点：**

*   分配内存：在方法区中为静态变量分配内存空间
*   设置默认值：为静态变量赋予类型默认值（0、false、null 等）
*   不执行代码：此阶段不会执行任何 Java 代码或赋值语句

**示例分析：**

```
public static final int CONST = 456; // 准备阶段后CONST的值就是456


```

对于 final 静态常量，某些 JVM 实现会直接在此阶段赋值，这是因为常量的值在编译期就能确定。

这种差异有时会让开发者感到困惑，特别是从其他语言（如 C#）转来的开发者。

### 2.4 解析阶段（Resolution）

解析阶段将常量池中的符号引用转换为直接引用。这个过程就像是将地址簿中的名称转换为实际的家庭住址。

**解析内容：**

*   类或接口解析：将类名解析为实际类引用
*   字段解析：将字段符号引用解析为具体字段
*   方法解析：将方法符号引用解析为具体方法
*   接口方法解析：将接口方法符号引用解析为具体实现

**技术细节：**

符号引用是一组描述被引用目标的符号，包含在 class 文件的常量池中。直接引用则可以是直接指向目标的指针、相对偏移量或能间接定位到目标的句柄。

解析阶段可以在初始化之后进行，这是 Java 语言 "运行时绑定" 特性的基础。JVM 实现可以根据需要灵活选择解析时机，采用惰性解析策略以提高性能。

### 2.5 初始化阶段（Initialization）

初始化是类加载过程的最后一步，真正开始执行类中定义的 Java 代码。

**触发条件：**

JVM 规范严格规定，只有在类首次被 "主动使用" 时才进行初始化。主动使用包括：

*   创建类的实例
*   访问类的静态变量或静态方法
*   使用反射调用类方法
*   初始化子类会触发父类初始化
*   包含 main 方法的启动类

**初始化内容：**

*   执行静态变量赋值语句
*   执行静态代码块（static{}）
*   执行类构造器`<clinit>()`方法

### 2.6 使用和卸载阶段

初始化完成后，类就进入了使用阶段，可以被正常实例化和调用了。当类不再被需要时，可能进入卸载阶段。

**使用阶段：**

类完全加载后，应用程序可以创建实例、调用方法、访问字段等。这是类生命周期中最长的阶段。

**卸载条件：**

当满足以下条件时，类可以被卸载：

*   该类的所有实例都已被回收
*   加载该类的 ClassLoader 已被回收
*   该类对应的 Class 对象没有被任何地方引用

卸载类的过程由 JVM 的垃圾收集器完成，开发者通常不需要关心这个过程。

### 2.7 惰性加载（Lazy Loading）机制

HotSpot JVM 为了提高性能，采用了许多优化策略。

其中最重要的是惰性加载（Lazy Loading）机制：除非必要，否则不会提前加载和链接类。

**示例说明：**

```
    static {
        System.out.println("A初始化");
    }
}
    static {
        System.out.println("B初始化");
    }
        System.out.println("B的方法被调用");
    }
}
    public static void main(String[] args) {
        A a = null; // 不会触发A的初始化
        System.out.println("A的引用已创建");
    }
}


```

**输出结果：**

```
B初始化
B的方法被调用


```

这种惰性加载策略显著提高了 JVM 的启动性能和内存使用效率。

只有当一个类被真正 "主动使用" 时，JVM 才会执行完整的加载、链接和初始化过程。

## 3、类加载器

在 JVM 执行 main 方法之前，首先需要找到对应的. class 文件并将其加载到内存中。

这个任务由类加载器（ClassLoader）完成。

从操作系统层面看，类加载本质上就是 JVM 进程通过 I/O 操作从磁盘或网络读取. class 文件的二进制数据，然后将其存入内存的特定区域。

如果单纯从技术实现角度考虑，类加载可以用很简单的代码实现：

C 语言版本：

Java 版本：

但如果只是简单地将类文件读入内存，没有良好的组织和管理机制，JVM 将无法高效地使用这些类。

因此，JVM 设计了一套完整的类加载体系结构。

### 3.1 系统内置的类加载器

JVM 提供了三层类加载器，形成了清晰的层次结构：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDq2LpicHlgtLN86sKwGicclaKRvwibQzVyX5BwrVsCwLgrv7wS2AiaxYASA/640?from=appmsg&watermark=1#imgIndex=9)

**1. 启动类加载器（Bootstrap Class Loader）**

*   实现：由 C++ 编写，是 JVM 自身的一部分
*   职责：加载 Java 核心库（如 rt.jar）
*   路径：/jre/lib

```
URL[] urls = sun.misc.Launcher.getBootstrapClassPath().getURLs();
/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/rt.jar
/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/sunrsasig.jar
/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/jsse.jar
/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/jce.jar
/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/charsets.jar
/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/jfr.jar
/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/classes


```

它可以看做是 JVM 自带的，我们再代码层面无法直接获取到启动类加载器的引用，所以不允许直接操作它，如果打印出来就是个 `null`。

举例来说，java.lang.String 是由启动类加载器加载的，所以 String.class.getClassLoader() 就会返回 null

**2. 扩展类加载器（Extension Class Loader）**

*   实现：由 Java 编写，继承自 URLClassLoader
*   职责：加载 Java 扩展库
*   路径：/jre/lib/ext

```
System.out.println(System.getProperty("java.ext.dirs"));
/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/ext:
/Library/Java/Extensions:
/Network/Library/Java/Extensions:
/System/Library/Java/Extensions:/usr/lib/java


```

**3. 应用程序类加载器（Application Class Loader）**

*   实现：sun.misc.Launcher$AppClassLoader
*   职责：加载用户类路径（-classpath 或 - cp 指定的路径）上的类
*   特点：是默认的系统类加载器

看下 Launcher 源码

```
 * Launcher 类是JVM启动应用程序的入口点，负责创建和初始化三层类加载器体系结构
 * 一句话解释：Launcher类是JVM启动应用程序的核心引导类，负责创建层级化的类加载器并设置初始安全环境
 * 由JVM内部实现，不继承java.lang.ClassLoader
 * 负责加载Java核心库，如rt.jar、resources.jar等
 * 在Java代码中无法直接获取其引用，打印结果为null
 *  Bootstrap ClassLoader由JVM内部实现，无Java代码

/
public class Launcher {
    public Launcher() {
        // 创建扩展类加载器
        ClassLoader extcl;
        try {
            extcl = ExtClassLoader.getExtClassLoader();
        } catch (IOException e) {
            throw new InternalError("Could not create extension class loader", e);
        }
        try {
            loader = AppClassLoader.getAppClassLoader(extcl);
        } catch (IOException e) {
            throw new InternalError("Could not create application class loader", e);
        }
    }
}


```

### 3.2 自定义类加载器

除了系统内置的类加载器，JVM 还允许用户自定义类加载器。

通过继承 ClassLoader 类并重写 findClass 方法，我们可以实现各种灵活的类加载策略。

自定义类加载器的典型用途包括：

*   从非标准来源加载类（如网络、加密文件等）
*   实现类的热部署和热替换
*   实现类的隔离和环境分离

首先我们看 ClassLoader 的核心方法 loadClass 和 findClass（简化版）

```
    synchronized (getClassLoadingLock(name)) {
        // 首先检查类是否已加载
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            try {
                if (parent != null) {
                    // 委派给父加载器
                    c = parent.loadClass(name, false);
                } else {
                    // 没有父加载器，使用启动类加载器
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // 父加载器无法完成加载
            }
                // 父加载器无法加载，尝试自己加载
                c = findClass(name);
            }
        }
        return c;
    }
}
 * 查找类的具体实现 - 由子类重写的方法
 * 这是类加载器的自定义加载逻辑的核心方法
 */
protected Class<?> findClass(String name) throws ClassNotFoundException {
    // 默认实现直接抛出异常，强制子类必须重写此方法
    // 子类需要实现从特定来源（文件系统、网络等）加载类字节码的逻辑
    throw new ClassNotFoundException(name);
}


```

通过复写 loadClass 方法，我们甚至可以读取一份加了密的文件，然后在内存里面解密，这样别人反编译你的源码也没用，因为 class 是经过加密的。

下面案例：通过自定义类加载器打破了双亲委派模型，双亲委派后文介绍，演示了同一个类被不同类加载器加载后在 JVM 中形成完全隔离的两个类实例的现象

```
import java.io.FileInputStream;
import static java.lang.System.out;
 * 类加载器隔离演示
 */
public class ClassIsolationPrinciple {    
    public static void main(String[] args) {        
        try {            
            String className = "com.zooncool.example.theory.jvm.ClassIsolationPrinciple$Demo"; // 要加载的类名            
            Class<?> class1 = Demo.class;  // 系统类加载器加载的类            
            Class<?> class2 = new MyClassLoader("target/classes").loadClass(className); // 自定义类加载器加载的类            
            out.println(class1.getName());            
            out.println(class2.getName());            
            out.println(class1.getClassLoader());            
            out.println(class2.getClassLoader());            

            out.println(class1.getDeclaredField("example").get(null));            
            out.println(class2.getDeclaredField("example").get(null));        
        } catch (Exception e) {            
            e.printStackTrace();        
        }    
    }
    public static class Demo {        
        public static int example = 0;    
    }
    public static class MyClassLoader extends ClassLoader{        
        private String classPath;        
        public MyClassLoader(String classPath) {            
            this.classPath = classPath;        
        }        
        @Override        
        public Class<?> loadClass(String name) throws ClassNotFoundException {            
            if(!name.contains("java.lang")){ // 排除Java核心类                
                byte[] data = new byte[0];                
                try {                    
                    data = loadByte(name); // 加载类字节码                
                } catch (Exception e) {                    
                    e.printStackTrace();                
                }                
                return defineClass(name, data, 0, data.length); // 定义类            
            }else{                
                return super.loadClass(name); // 核心类仍用父类加载            
            }        
        }        
        private byte[] loadByte(String name) throws Exception {            
            name = name.replaceAll("92.", "/"); // 转换包名为路径            
            String dir = classPath + "/" + name + ".class";            
            FileInputStream fis = new FileInputStream(dir);            
            int len = fis.available();            
            byte[] data = new byte[len];            
            fis.read(data);            
            fis.close();            
            return data;        
        }    
    }
}


```

执行结果如下，我们可以看到加载到内存方法区的两个类的包名 + 名称是一样的，而对应的类加载器却不一样，而且输出被加载类的值也是不一样的。

```
com.zooncool.example.theory.jvm.ClassIsolationPrinciple2$Demo
com.zooncool.example.theory.jvm.ClassIsolationPrinciple2$Demo
-----------------classLoader name-----------------
sun.misc.Launcher$AppClassLoader@18b4aac2
com.zooncool.example.theory.jvm.ClassIsolationPrinciple2$MyClassLoader@511d50c0
-----------------field value-----------------
1
0


```

### 3.3 双亲委派模型

JVM 类加载器遵循双亲委派模型（Parent Delegation Model），这是类加载机制的核心设计原则。

**工作原理**：

**(1) 当一个类加载器收到加载请求时，首先不会自己尝试加载**

**(2) 而是将这个请求委派给父类加载器处理**

**(3) 只有当父类加载器无法完成加载时，子加载器才会尝试自己加载**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDHhArJxVzgQtnR3dmOQdrMsict7yhr0T8UGce76fUEKehlIZicb3kal2A/640?from=appmsg&watermark=1#imgIndex=10)

```
    A[加载请求] --> B[应用程序类加载器]
    B --> C[扩展类加载器]
    C --> D[启动类加载器]
    D --> E[加载成功?]
    E -- 是 --> F[返回类]
    E -- 否 --> G[扩展类加载器尝试加载]
    G --> H[加载成功?]
    H -- 是 --> F
    H -- 否 --> I[应用程序类加载器尝试加载]
    I --> J[加载成功?]
    J -- 是 --> F
    J -- 否 --> K[ClassNotFoundException]
    style F fill:#e8f5e9
    style K fill:#ffebee


```

ClassLoader 的核心方法 loadClass 和 findClass

*   想遵守双亲委派：只重写 `findClass`方法。`loadClass`方法的委托逻辑会保证父加载器先尝试加载。
*   想打破双亲委派：需要重写 `loadClass`方法，实现自己的加载逻辑，绕过对 `parent.loadClass()`的调用

```
    synchronized (getClassLoadingLock(name)) {
        // 首先检查类是否已加载
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            try {
                if (parent != null) {
                    // 委派给父加载器
                    c = parent.loadClass(name, false);
                } else {
                    // 没有父加载器，使用启动类加载器
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // 父加载器无法完成加载
            }
                // 父加载器无法加载，尝试自己加载
                c = findClass(name);
            }
        }
        return c;
    }
}
 * 查找类的具体实现 - 由子类重写的方法
 * 这是类加载器的自定义加载逻辑的核心方法
 */
protected Class<?> findClass(String name) throws ClassNotFoundException {
    // 默认实现直接抛出异常，强制子类必须重写此方法
    // 子类需要实现从特定来源（文件系统、网络等）加载类字节码的逻辑
    throw new ClassNotFoundException(name);
}


```

**双亲委派模型的优势**：

**(1) 安全性：防止核心 Java 类被篡改，确保 Java 平台的安全性**

**(2) 避免重复加载：同一个类只会被一个加载器加载，防止多次加载同一类**

**(3) 结构清晰：形成了清晰的层次结构和责任链**

**打破双亲委派的情况**：

虽然双亲委派是主要模型，但在某些特定场景下需要打破这个机制：

**(1) SPI（Service Provider Interface）机制：如 JDBC、JNDI 等服务发现机制**

**(2) 热部署和模块化需求：如 OSGi 框架**

**(3) 兼容性考虑：向前兼容早期 Java 版本**

### 3.4 类的唯一标识

在 JVM 的视角中，一个类的全局唯一标识并非仅由其全限定名（如 `java.lang.String`）决定，而是由 **`类加载器实例 + 类的全限定名`** 组成的二元组共同构成。

**实现机制**：

**(1) 缓存检查：当一个类加载器接收到加载请求时，JVM 首先会检查 `<类加载器实例, 全限定名>`这个组合是否已经存在于内存中。**

**(2) 返回已有类：如果找到了完全匹配的条目，JVM 会直接返回缓存中已存在的 Class 对象引用，不会重复加载。**

**(3) 首次加载：如果未找到，则会执行完整的加载过程，并在成功后将该条目注册到缓存中。**

无论是遵循双亲委派还是打破它，最终都需要调用 `ClassLoader.defineClass()`方法将字节数组转换为 JVM 认可的 Class 对象。

在这个最终环节，JVM 会进行关键的校验，其核心逻辑（尤其体现在 `preDefineClass`方法中）可以简化为以下流程：

```
    A[defineClass 开始] --> B[计算唯一标识<br>类加载器实例 + 全限定名]
    B --> C{检查缓存是否存在<br>相同标识的类?}
    C -- 是 --> D[抛出 LinkageError<br>重复的类定义]
    C -- 否 --> E[执行真正的类定义<br>调用本地方法 defineClass1]
    E --> F[将新类注册到缓存]
    F --> G[返回新创建的Class对象]


```

`<类加载器实例, 全限定名>`的唯一标识机制，与 `defineClass`方法中的 `preDefineClass`校验流程共同构成了 JVM 的类隔离与安全防线。

这使得不同的类加载器命名空间相互隔离，从而支持了 OSGi、组件热部署等高级特性，同时从根本上保障了 Java 核心运行时库的安全性与完整性。

看下源码 URLClassLoader.java 443：

```
 * 将字节码数据转换为JVM中的Class对象
 * 此方法封装了类加载的核心过程，确保类定义的唯一性
/
private Class<?> defineClass(String name, Resource res) throws IOException {
    // 解析包名并确保包已定义（包定义影响类的可见性）
    int i = name.lastIndexOf('.');
    URL url = res.getCodeSourceURL();
    if (i != -1) {
        String pkgname = name.substring(0, i);
        Manifest man = res.getManifest();
        definePackageInternal(pkgname, man, url); // 确保包层级结构
    }
    java.nio.ByteBuffer bb = res.getByteBuffer();
    if (bb != null) {
        // 使用ByteBuffer路径
        CodeSource cs = new CodeSource(url, res.getCodeSigners());
        return defineClass(name, bb, cs); // 最终调用native方法
    } else {
        // 使用byte[]路径
        byte[] b = res.getBytes();
        CodeSource cs = new CodeSource(url, res.getCodeSigners());
        return defineClass(name, b, 0, b.length, cs); // 最终调用native方法
    }
}
 * 类加载唯一性关键：类全限定名 + 类加载器实例
 * 相同类名由不同类加载器加载会产生不同的Class对象
/
protected final Class<?> defineClass(String name, byte[] b, int off, int len,
                                   ProtectionDomain protectionDomain) {
    // 预处理：验证类名格式和访问权限
    protectionDomain = preDefineClass(name, protectionDomain);
    // 此操作受JVM的类加载唯一性约束
    Class<?> c = defineClass1(name, b, off, len, protectionDomain, null);
    postDefineClass(c, protectionDomain);
    return c;
}
 * JVM本地方法：实际完成字节码到Class对象的转换
 * 此方法确保 (类名, 类加载器) 元组的唯一性
 */
private native Class<?> defineClass1(String name, byte[] b, int off, int len,
                                   ProtectionDomain pd, String source);


```

## 4、类加载的应用场景

类加载器机制不仅是 JVM 的理论基础，更是实现许多高级特性的关键技术。

通过灵活运用类加载器，开发者可以实现模块化部署、代码热更新、环境隔离等强大功能。

下面我们将深入探讨类加载器最典型的应用场景之一：热部署。

### 4.1 热部署应用

#### 什么是热部署？

热部署（Hot Deployment）是指在应用程序运行过程中，无需重启整个服务即可更新代码变更的技术能力。

与冷部署（需要停止服务再部署）相比，热部署能够显著提升开发效率和系统可用性，特别在大型在线系统中具有重要意义。

真正的热部署是应用容器（如 Tomcat、Jetty）提供的一种能力，不同于开发环境中 IDE 使用的热加载（HotSwap）技术。

热加载依赖于 JVM 的 HPROF 接口，只能更新方法体，而热部署能够实现完整的类替换和系统更新。

#### 技术原理

热部署的核心原理基于 JVM 的类加载器隔离机制。

每个类在 JVM 中的唯一标识由`类加载器实例 + 类全限定名`共同决定。利用这个特性，我们可以通过创建新的类加载器来加载更新后的类，从而实现代码的热更新。

整个热部署过程的关键流程如下：

```
    A[检测到代码变更] --> B[编译生成新的.class文件]
    B --> C[创建新的类加载器]
    C --> D[使用新加载器加载更新类]
    D --> E[创建新类实例替换旧实例]
    E --> F[卸载旧类加载器及其加载的类]
    F --> G[完成热部署]
    style G fill:#e8f5e9


```

#### 实操案例

下面通过一个具体示例展示如何实现基本的热部署功能。

这个示例包含一个简单的类和自定义类加载器。

```
import java.io.FileInputStream;
import static java.lang.System.out;
 * 热部署演示类
 * 展示如何通过自定义类加载器实现代码热更新
/
public class HotDeployDemo {
        try {
            // 定义要加载的类全限定名
            String className = "com.zooncool.example.theory.jvm.HotDeployDemo$BusinessLogic";
            while (true) {
                // 创建新的类加载器实例加载最新编译的类
                Class<?> clazz = new HotDeployClassLoader("target/classes").loadClass(className);
                Object instance = clazz.newInstance();
                clazz.getDeclaredMethod("execute").invoke(instance);
                Thread.sleep(5000);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
     * 业务逻辑类 - 这是需要热更新的类
     * 修改此类代码并重新编译后，热部署机制将加载新版本
/
    public static class BusinessLogic {
        public void execute() {
            // 业务逻辑代码
            // 修改此方法内容后重新编译，观察热部署效果
            System.out.println("Business logic v1.0: " + new java.util.Date());
        }
    }
     * 热部署类加载器
     * 打破双亲委派模型，优先自行加载类
/
    public static class HotDeployClassLoader extends ClassLoader {
        private String classPath;
            this.classPath = classPath;
        }
         * 重写loadClass方法，打破双亲委派模型
         * 对于非Java核心类，直接自行加载而不委托给父加载器
/
        @Override
        public Class<?> loadClass(String name) throws ClassNotFoundException {
            // 排除Java核心类，这些必须由父加载器加载
            if (!name.startsWith("java.")) {
                try {
                    // 读取类字节码
                    byte[] bytecode = loadClassBytes(name);
                    // 将字节数组转换为Class对象
                    return defineClass(name, bytecode, 0, bytecode.length);
                } catch (Exception e) {
                    // 加载失败则委托给父加载器
                    return super.loadClass(name);
                }
            } else {
                // Java核心类仍然遵循双亲委派
                return super.loadClass(name);
            }
        }
         * 从文件系统加载类字节码
         * @param name 类的全限定名
         * @return 类的字节码数据
         */
        private byte[] loadClassBytes(String name) throws Exception {
            // 将包名转换为文件路径
            String filePath = name.replace('.', '/') + ".class";
            String fullPath = classPath + "/" + filePath;
            try (FileInputStream fis = new FileInputStream(fullPath)) {
                int length = fis.available();
                byte[] data = new byte[length];
                fis.read(data);
                return data;
            }
        }
    }
}


```

**关键代码解释：**

**(1) 自定义类加载器：`HotDeployClassLoader`通过重写 `loadClass`方法打破了双亲委派模型，实现了自行加载类的功能。**

**(2) 类加载逻辑： 对于非 Java 核心类（不以 "java." 开头的类），直接读取字节码文件并调用 `defineClass`方法定义类对于 Java 核心类，仍然委托给父加载器加载，确保系统稳定性**

**(3) 热部署循环：主程序通过循环不断加载最新版本的类，每次迭代都创建新的类加载器实例，确保能够加载到重新编译后的类。**

启动后修改，比如修改版本 1.0 到 2.0 可以看到修改后热部署效果

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDROI6gg5X7Mnd9Tbib8bHUtuBoWm2j6Cx7BGIj5RYFWa740FB11ShaVA/640?from=appmsg&watermark=1#imgIndex=11)

#### 上下文隔离与状态保留

在实际应用中，直接替换类加载器可能导致状态丢失。

下面示例展示如何实现更高级的热部署，保留应用状态：

通过双轨制类加载策略实现隔离与状态保留：

**(1) 状态保留：`AppState`由系统类加载器加载，作为静态变量持久化存储**

**(2) 热更新隔离：`BusinessLogic`由自定义加载器加载，每次循环重新加载新版本**

**(3) 状态共享：通过反射将同一状态对象注入新旧业务实例**

**(4) 类型安全：特殊处理确保状态类始终由系统加载器加载**

```
    A[系统类加载器] --> B[加载AppState]
    A --> C[持久化状态]
    D[自定义加载器] --> E[加载BusinessLogic]
    D -->|每次循环| F[创建新版本业务实例]
    C -->|反射注入| F


```

核心原理：**状态与业务分离加载 + 跨版本状态注入**，实现热更新时不丢失应用状态。

```
 * 高级热部署示例 - 保持应用状态
 * 通过分离业务逻辑和状态数据实现无缝热更新
/
public class AdvancedHotDeploy {
    // 状态数据由系统类加载器加载，热部署时保持不变
    private static AppState state = new AppState();
        try {
            while (true) {
                // 使用自定义加载器加载业务逻辑
                Class<?> logicClass = new HotDeployClassLoader("target/classes")
                        .loadClass("com.zooncool.example.theory.jvm.AdvancedHotDeploy$BusinessLogic");
                Object logicInstance = logicClass.newInstance();
                logicClass.getDeclaredField("state").set(logicInstance, state);
                logicClass.getDeclaredMethod("execute").invoke(logicInstance);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
     * 应用状态 - 由系统类加载器加载
     * 热部署过程中保持状态不变
/
    public static class AppState {
        private int requestCount = 0;
        private String userSession = "default_session";
            requestCount++;
        }
        // 省略getter和setter
            return requestCount;
        }
            this.requestCount = requestCount;
        }
            return userSession;
        }
            this.userSession = userSession;
        }
    }
     * 业务逻辑 - 由自定义加载器加载
     * 可热更新部分
/
    public static class BusinessLogic {
        // 注入状态对象
        public AppState state;
            state.incrementCount();
            System.out.println("Request count: " + state.getRequestCount() + ", Session: " + state.getUserSession());
        }
            return state;
        }
            this.state = state;
        }
    }
     * 热部署类加载器
     * 打破双亲委派模型，优先自行加载类
/
    public static class HotDeployClassLoader extends ClassLoader {
        private String classPath;
            this.classPath = classPath;
        }
         * 重写loadClass方法，打破双亲委派模型
         * 对于非Java核心类，直接自行加载而不委托给父加载器
/
        @Override
        public Class<?> loadClass(String name) throws ClassNotFoundException {
            // 让AppState始终由系统类加载器加载
            if (name.equals("com.zooncool.example.theory.jvm.AdvancedHotDeploy$AppState")) {
                return super.loadClass(name); // 委托给父加载器
            }
                try {
                    byte[] bytecode = loadClassBytes(name);
                    return defineClass(name, bytecode, 0, bytecode.length);
                } catch (Exception e) {
                    return super.loadClass(name);
                }
            } else {
                return super.loadClass(name);
            }
        }
         * 从文件系统加载类字节码
         * @param name 类的全限定名
         * @return 类的字节码数据
/
        private byte[] loadClassBytes(String name) throws Exception {
            // 将包名转换为文件路径
            String filePath = name.replace('.', '/') + ".class";
            String fullPath = classPath + "/" + filePath;
            try (FileInputStream fis = new FileInputStream(fullPath)) {
                int length = fis.available();
                byte[] data = new byte[length];
                fis.read(data);
                return data;
            }
        }
    }
}


```

此案例只是演示类加载的应用场景，在实际生产环境中，热部署通常通过以下方式实现：

**(1) OSGi 框架：提供了完整的模块化系统和热部署能力，每个模块（Bundle）有独立的类加载器**

**(2) 应用服务器热部署：Tomcat、Jetty 等容器提供了热部署功能，通过监控 WAR 文件变化自动重新加载应用**

**(3) Spring Boot DevTools：开发阶段的热部署工具，通过监控类路径变化自动重启应用**

### 4.2 类隔离应用

#### 为什么需要类隔离？

在复杂的 Java 应用中，类冲突是常见痛点：

**(1) 版本冲突：不同模块依赖同一类库的不同版本**

**(2) 安全隔离：核心框架与应用代码需要隔离**

**(3) 环境隔离：测试环境与生产环境类库共存**

传统解决方案如 Maven 依赖排除或 jar 重命名（使用 jarjar 工具）在小型项目中有效，但在分布式系统中捉襟见肘。类隔离技术通过类加载器实现逻辑隔离，成为解决这些问题的银弹。

#### 类隔离的核心原理

类隔离基于 JVM 的黄金法则：

**<类加载器, 类全限定名> 二元组决定类的唯一身份**

```
    A[类加载请求] --> B{是否已加载}
    B -->|是| C[返回缓存类]
    B -->|否| D[创建新类加载器]
    D --> E[加载类字节码]
    E --> F[定义新类]
    F --> G[缓存<加载器+类名, 类>]
    G --> H[返回新类]
    style H fill:#2196F3,stroke:#0D47A1


```

下面案例通过自定义类加载器打破了双亲委派模型，演示了**类隔离机制**：

**(1) 相同类名，不同身份：同一个`User`类被系统加载器和自定义加载器分别加载，在 JVM 中被视为两个完全独立的类**

**(2) 静态字段隔离：修改系统加载的`User.version`值为 2，自定义加载的`User.version`仍保持 1**

**(3) 核心原理：类的唯一标识由`<类加载器实例 + 类全限定名>`决定，不同加载器加载的相同类名在 JVM 中互不干扰**

这展示了如何通过类加载器实现版本隔离，解决依赖冲突问题。

```
    public static void main(String[] args) throws Exception {
        // 系统类加载器加载的类
        Class<?> sysClass = User.class;
        String className = "com.zooncool.example.theory.jvm.ClassIsolationDemo$User";
        Class<?> customClass = new CustomLoader("target/classes").loadClass(className);
                sysClass.getName().equals(customClass.getName())); // true
                (sysClass.getClassLoader() != customClass.getClassLoader())); // true
        User.version = 2;
        System.out.println("系统类: " + sysClass.getField("version").get(null)); // 2
        System.out.println("自定义类: " + customClass.getField("version").get(null)); // 1
    }
        public static int version = 1;
    }
    static class CustomLoader extends ClassLoader {
        private String classPath;
            this.classPath = classPath;
        }
         * 重写loadClass方法，打破双亲委派模型
         * 对于非Java核心类，直接自行加载而不委托给父加载器
/
        @Override
        public Class<?> loadClass(String name) throws ClassNotFoundException {
            // 排除Java核心类，这些必须由父加载器加载
            if (!name.startsWith("java.")) {
                try {
                    // 读取类字节码
                    byte[] bytecode = loadClassBytes(name);
                    // 将字节数组转换为Class对象
                    return defineClass(name, bytecode, 0, bytecode.length);
                } catch (Exception e) {
                    // 加载失败则委托给父加载器
                    return super.loadClass(name);
                }
            } else {
                // Java核心类仍然遵循双亲委派
                return super.loadClass(name);
            }
        }
         * 从文件系统加载类字节码
         * @param name 类的全限定名
         * @return 类的字节码数据
/
        private byte[] loadClassBytes(String name) throws Exception {
            // 将包名转换为文件路径
            String filePath = name.replace('.', '/') + ".class";
            String fullPath = classPath + "/" + filePath;
            try (FileInputStream fis = new FileInputStream(fullPath)) {
                int length = fis.available();
                byte[] data = new byte[length];
                fis.read(data);
                return data;
            }
        }
}


```

执行结果：

```
类加载器不同: true
字段值隔离: 
系统类: 2
自定义类: 1


```

#### 容器级隔离（Tomcat 案例）

在传统 Java 应用中，类加载遵循**双亲委派模型**，所有类都由同一个类加载器体系加载。但在 Web 容器中，这种模式会遇到严重问题：

**1）类库版本冲突**

*   应用 A 需要 `commons-lang 2.4`
*   应用 B 需要 `commons-lang 3.0`
*   如果使用同一类加载器，无法同时加载两个版本

**2）类覆盖和安全问题**

*   应用可能包含与容器相同全限定名的类（如`Servlet API`）
*   恶意应用可能通过替换核心类破坏容器稳定性

**3）资源隔离需求**

*   每个应用应该有独立的静态资源访问范围
*   应用间应该无法直接访问彼此的类

Tomcat 通过精细的类加载器层次实现隔离：

```
    Boot[Bootstrap类加载器] -->|父| Common[Common类加载器]
    Common -->|父| Catalina[Catalina类加载器]
    Common -->|父| Shared[Shared类加载器]
    Shared -->|父| WebApp1[WebApp1类加载器]
    Shared -->|父| WebApp2[WebApp2类加载器]
    style Common fill:#4CAF50
    style Catalina fill:#2196F3
    style Shared fill:#FF9800
    style WebApp1 fill:#FF9800
    style WebApp2 fill:#FF9800


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDRaSPic7EnjTFOWVibbzGXnTicp4wBu86T8ZOlRKM6eibIKH0gtACtHShUg/640?from=appmsg&watermark=1#imgIndex=12)

关键源码实现（Tomcat 9.x）：

```
public final class Bootstrap {
    public static void main(String[] args) {
        // 初始化类加载器层级
        initClassLoaders();
        Thread.currentThread().setContextClassLoader(catalinaLoader);
        Class<?> startupClass = catalinaLoader.loadClass("org.apache.catalina.startup.Catalina");
        Object startupInstance = startupClass.getConstructor().newInstance();
    }
        try {
            // 创建公共类加载器（加载$CATALINA_HOME/lib下的JAR）
            commonLoader = createClassLoader("common", null);
            catalinaLoader = createClassLoader("server", commonLoader);
            sharedLoader = createClassLoader("shared", commonLoader);
        } catch (Throwable t) {
            handleThrowable(t);
            t.printStackTrace();
            System.exit(1);
        }
    }
        // 从catalina.properties读取类路径配置
        String value = properties.getProperty(name + ".loader");
        return ClassLoaderFactory.createClassLoader(repositories, parent);
    }
}


```

隔离效果：

*   **Web 应用隔离**

    ：每个 WebApp 有独立类加载器，互不影响

*   **核心容器保护**

    ：Catalina 加载器与 Web 应用加载器隔离

*   **资源共享**

    ：Common 加载器存放公共库，避免重复加载

#### 模块级隔离（OSGi 案例）

OSGi 实现更细粒度的模块隔离：

```
    A[Bundle A] -->|导入| B[Package X v1]
    C[Bundle B] -->|导入| D[Package X v2]
    E[OSGi框架] -->|解析| F[解决依赖冲突]
    style C fill:#2196F3
    style B fill:#4CAF50
    style D fill:#FF9800


```

关键特性：

*   每个 Bundle（模块）有独立类加载器
*   动态解析依赖关系
*   支持模块热插拔
*   版本化依赖管理

## 1、运行时数据区

JVM 内存从逻辑上划分为多个独立的数据区域，每个区域承担不同的职责：

```
    A[JVM内存结构] --> B[线程私有区]
    A --> C[线程共享区]
    B --> B2[虚拟机栈]
    B --> B3[本地方法栈]
    C --> C2[方法区]
    C --> C3[运行时常量池]
    style C fill:#f3e5f5


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpD4e9mDHvY8iceV9VWHSD5O5t4KDNczDSJ4JylqV5ViblA8ZewoQJanYSg/640?from=appmsg&watermark=1#imgIndex=13)

**程序计数器（Program Counter Register）**

*   作用：记录当前线程执行的字节码行号指示器
*   特性：线程私有，唯一不会出现 OutOfMemoryError 的区域
*   场景：线程切换后能恢复到正确的执行位置

**虚拟机栈（VM Stack）**

*   功能：存储栈帧（局部变量表、操作数栈、动态链接、方法出口）
*   配置：-Xss 设置栈容量（默认 1MB）
*   异常：StackOverflowError（栈深度溢出）、OutOfMemoryError（扩展失败）

**本地方法栈（Native Method Stack）**

*   用途：为 Native 方法服务
*   实现：HotSpot 将虚拟机栈与本地方法栈合并

**堆内存（Heap）**

*   特性：所有对象实例和数组的分配区域
*   配置：-Xmx（最大堆）、-Xms（初始堆）
*   管理：垃圾回收的主要战场

**方法区（Method Area）**

*   存储：类型信息、常量、静态变量、JIT 代码缓存
*   演进：JDK8 前为 PermGen，JDK8 + 为 Metaspace

## 2、堆内存

堆内存采用分代设计，针对不同生命周期的对象采用不同的管理策略：

```
    A[Java堆] --> B[新生代]
    A --> C[老年代]
    B --> B2[Survivor0]
    B --> B3[Survivor1]

    style B2 fill:#f3e5f5
    style B3 fill:#e8f5e9
    style C fill:#e1f5fe


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDyOP1hTYO8ZmXQ78H6lyPr4IWcfemHKZbJyJVKaJTrqhibmSn94ia7hww/640?from=appmsg&watermark=1#imgIndex=14)

**新生代（Young Generation）**

*   **Eden 区**

    ：新对象诞生地（80% 新生代空间）

*   **Survivor 区**

    ：幸存对象暂存区（S0、S1 各占 10%）

**老年代（Tenured Generation）**

*   对象特征：长期存活对象、大对象直接分配
*   回收策略：Major GC/Full GC
*   配置参数：-XX:MaxTenuringThreshold=15（晋升阈值）

## 3、非堆内存

```
    A[JVM内存] --> B[堆内存 Heap]
    A --> C[非堆内存 Non-Heap]
    C --> C2[代码缓存 Code Cache]
    C --> C3[线程栈 Thread Stack]
    C --> C4[直接内存 Direct Memory]
    C --> C5[GC工作内存]
    C --> C6[本地方法栈]
    B --> B2[老年代 Old]
    style B fill:#f3e5f5


```

JVM 的非堆内存主要包括以下几个核心区域：

**1）元空间（Metaspace）**

**存储内容**：类元数据、方法信息、字段信息、常量池、字节码等

**特点**：

*   使用本地内存（Native Memory）
*   自动扩展，无固定上限
*   类卸载时释放对应元数据

**2） 代码缓存（Code Cache）**

**存储内容**：JIT 编译后的本地机器码

```
    // 热点方法会被JIT编译并存入代码缓存
    public void hotMethod() {
        for (int i = 0; i < 1000000; i++) {
            // 频繁执行的代码会被编译优化
            Math.sin(i);
        }
    }
}


```

**配置参数**：

*   `-XX:ReservedCodeCacheSize=240m`
    
    - 最大代码缓存大小
    
*   `-XX:InitialCodeCacheSize=32m`
    
    - 初始大小

**3）线程栈（Thread Stack）**

**存储内容**：线程执行状态、局部变量、方法调用栈帧

```
    public void recursiveCall(int depth) {
        int localVar = depth * 10; // 存储在栈帧中
        if (depth > 0) {
            recursiveCall(depth - 1); // 每次调用创建新栈帧
        }
    }
}


```

**配置**：`-Xss1m`（每个线程栈大小）

**4）直接内存（Direct Memory）**

**存储内容**：NIO Buffer、MMAP 文件映射等

```
    public void useDirectBuffer() {
        // 分配直接内存，不经过Java堆
        ByteBuffer directBuffer = ByteBuffer.allocateDirect(1024 * 1024); // 1MB
        FileChannel channel = FileChannel.open(Paths.get("test.dat"));
        channel.read(directBuffer);
    }
}


```

**配置**：`-XX:MaxDirectMemorySize=256m`

**5） JVM 内部数据结构**

**包括**：

*   垃圾收集器工作内存（Card Table、Remembered Set 等）
*   JIT 编译器工作内存
*   符号表（Symbol Table）
*   字符串常量池（JDK7 + 在堆中）

## 4、运行时栈帧结构

下图是 JVM 实例执行方法时的内存布局。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDibF03Ng9Y319blaFKnkTKdspeIHYhvQU7JiasE4d2nKttp2DGIZRQTnw/640?from=appmsg&watermark=1#imgIndex=15)

每个方法调用都会在虚拟机栈中创建一个栈帧（Stack Frame），用于存储方法的执行状态和数据。

**栈帧的核心组成部分**：

```
    A[栈帧结构] --> B[局部变量表]
    A --> C[操作数栈]
    A --> D[动态链接]
    A --> E[方法返回地址]
    B --> B2[局部变量]





```

**1）局部变量表（Local Variables Table）**

存储方法参数和方法内定义的局部变量，以变量槽（Slot）为基本单位。

```
    public void method(int param1, String param2) {
        // 局部变量表包含：
        // Slot 0: this引用
        // Slot 1: param1 (int)
        // Slot 2: param2 (String引用)
        double localVar2 = 20.5; // Slot 4-5（double占用2个Slot）
    }
}


```

**2）操作数栈（Operand Stack）**

用于存储计算过程中的中间结果，是字节码指令操作的工作区。

```
    public int calculate(int a, int b) {
        // 字节码操作过程：
        // iload_1  // 将a压入操作数栈
        // iload_2  // 将b压入操作数栈
        // iadd     // 弹出两个值相加，结果压回栈顶
        // ireturn  // 返回栈顶结果
        return a + b;
    }
}


```

**3）动态链接（Dynamic Linking）**

每个栈帧都包含一个指向运行时常量池中该栈帧所属方法的引用，以便在运行时动态确定实际调用的方法。

**4）方法返回地址（Return Address）**

存储方法正常完成或异常中断后需要返回的位置信息。

垃圾回收（Garbage Collection，GC）是 JVM 自动管理内存的核心机制，它负责自动识别和回收程序中不再使用的对象所占用的内存空间。

与 C/C++ 等需要手动管理内存的语言不同，Java 通过 GC 机制解放了程序员，避免了常见的内存泄漏和悬空指针问题。

**GC 的核心任务**：

*   自动识别内存中的垃圾对象（不再被引用的对象）
*   回收垃圾对象占用的内存空间
*   整理内存碎片，提高内存利用率

```
    A[GC开始] --> B[标记可达对象]
    B --> C[清除不可达对象]
    C --> D[压缩内存空间]
    D --> E[GC完成]
    style E fill:#e8f5e9


```

## 1、垃圾对象的识别算法

垃圾回收的第一步是准确识别哪些对象是 "垃圾"。JVM 主要使用两种算法进行判断：

```
    A[垃圾对象识别] --> B[引用计数算法]
    A --> C[根搜索算法]
    B --> B2[无法解决循环引用]
    B --> B3[Python/COM使用]
    C --> C2[解决循环引用问题]
    C --> C3[Java/.NET使用]


```

### 1.1 引用计数算法（Reference Counting）

**核心原理**：为每个对象维护一个引用计数器，当被引用时计数器加 1，引用失效时减 1。当计数器为 0 时，表示对象不再被使用。

引用计数器的实现很简单，对于一个对象 A，只要有任何一个对象引用了 A，则 A 的引用计数器就加 1，当引用失效时，引用计数器就减 1。只要对象 A 的引用计数器的值为 0，则对象 A 就不可能再被使用。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDSRoM4CC76Cuaicao3LiccichUIy4nwp3UAsaVUibR2OWj2JGeaHRicv3pxA/640?from=appmsg&watermark=1#imgIndex=16)

**缺点**：

*   无法解决循环引用问题
*   每次引用操作都需要更新计数器，性能开销大
*   实现复杂，需要处理并发计数的一致性

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDicSCWMCZXkk5uVlibk4ZDmDhWu583gS3MQuAVAM3Bs2BA4aDBshlJR9g/640?from=appmsg&watermark=1#imgIndex=17)

### 1.2 根搜索算法（GC Roots Tracing）

**核心原理**：通过一系列称为 "GC Roots" 的对象作为起点，从这些节点开始向下搜索，如果某个对象到 GC Roots 没有任何引用链相连，则证明此对象是不可达的，可以被回收。

**Java 中的 GC Roots 包括**：

*   虚拟机栈中引用的对象（局部变量）
*   方法区中类静态属性引用的对象
*   方法区中常量引用的对象
*   本地方法栈中 JNI 引用的对象

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDKhuL0BpIUE5YgB0MISLzkvdVhNGBUJkp91arTtUDB1yrkSgAj9KSWQ/640?from=appmsg&watermark=1#imgIndex=18)

## 2、垃圾回收算法

现代 JVM 主要采用三种基础垃圾回收算法，每种算法都有其特定的适用场景。

### 2.1 标记 - 清除算法（Mark-Sweep）

**算法流程**：

**(1) 标记阶段：从 GC Roots 开始，遍历所有可达对象并进行标记**

**(2) 清除阶段：回收所有未被标记的对象**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpD3Fjjticna78YIGqU5qdtPb2BIIF8upd62G7jDMNhQTtJBjj3aaxes1A/640?from=appmsg&watermark=1#imgIndex=19)

**优点**：实现简单，不需要移动对象

**缺点**：产生内存碎片，影响大对象分配

### 2.2 复制算法（Copying）

**算法流程**：

**(1) 将内存分为大小相等的两块（From 和 To 空间）**

**(2) 每次只使用其中一块（From 空间）**

**(3) GC 时将存活对象复制到另一块（To 空间）**

**(4) 清空已使用的内存空间**

**(5) 交换两个空间的角色**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpD8QrSAYHmb4x0OGEM6nTVVQJ4GNgOpLBicYqibvVGGTWBUFUL3ImmV85g/640?from=appmsg&watermark=1#imgIndex=20)

**优点**：没有内存碎片，分配高效

**缺点**：内存利用率只有 50%

### 2.3 标记 - 整理算法（Mark-Compact）

**算法流程**：

**(1) 标记阶段：与标记 - 清除相同，标记所有可达对象**

**(2) 整理阶段：将所有存活对象向内存一端移动**

**(3) 清理阶段：清理边界以外的内存**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDLdz9TujTsNbNj7LhUkbzdZqxNrn46Zh6AGGEIUjbnSTTqfz1G3Lhqw/640?from=appmsg&watermark=1#imgIndex=21)

**优点**：没有碎片，内存利用率高

**缺点**：需要移动对象，成本较高

## 3、GC 分代思想

现代垃圾收集器普遍采用分代收集策略，其核心思想是根据对象存活时间的不同，将堆内存划分为新生代和老年代两个主要区域，并采用不同的回收算法提升效率。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDjZ52BpyyXzZicxRydm0YNCAoEtFuvYzWP5o4ChyhXUD1eOW3p9CTxaQ/640?from=appmsg&watermark=1#imgIndex=22)

**新生代：**专门存放生命周期短的对象，内部划分为 Eden 区和两个 Survivor 区。新对象首先分配在 Eden 区，当空间不足时触发 Minor GC。存活的对象会转移到 Survivor 区，每经历一次 GC 年龄增加 1 岁，当超过阈值（默认 15 次）后晋升到老年代。

**老年代：**存放长期存活的对象。由于对象存活率高，通常采用标记 - 清除或标记 - 整理算法进行 Full GC。

此外，Java 8 用元空间取代了持久代，将类元数据存储到本地内存，避免了之前持久代容易内存溢出的问题。

## 4、内存分配策略

JVM 通过几条核心规则来管理对象的内存分配，确保内存使用的高效性。

**对象优先在 Eden 区分配**

大多数新创建的对象会在新生代的 Eden 区进行分配。当 Eden 区空间不足时，会触发 Minor GC 进行垃圾回收。

**大对象直接进入老年代**

通过 `-XX:PretenureSizeThreshold`参数可设置大对象阈值。超过该大小的对象将直接在老年代分配，避免在新生代中产生大量内存复制。

**长期存活对象进入老年代**

每个对象都有年龄计数器，每经历一次 Minor GC 且存活则年龄加 1。当年龄超过阈值（`-XX:MaxTenuringThreshold`，默认 15）时，对象晋升到老年代。

**动态年龄判定**

如果某年龄及以上对象总大小超过 Survivor 区容量一半，也就是说此时 S0 或者 S1 区即将容纳不了存活的新生代对象，则这些对象直接晋升老年代，无需达到最大年龄阈值。

**空间分配担保**

进行 Minor GC 前，JVM 会检查老年代剩余空间：

*   若大于新生代对象总大小，直接进行 Minor GC
*   否则检查是否允许担保失败（-XX:-HandlePromotionFailure），如果允许，根据历史晋升平均值决定是否冒险进行 Minor GC
*   若担保失败，会出现`promotion failed`错误，则先进行 Full GC

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpD7ao3gj3np0fpG7vqKzicQwWS64cINfUEZThHNFwIVVaIzJfYvTveNPg/640?from=appmsg&watermark=1#imgIndex=23)

**Survivor 区空间不足**

当 Minor GC 后存活对象过多，Survivor 区无法容纳时，多余对象将通过分配担保机制直接进入老年代。

## 5、常见垃圾回收器

垃圾回收器主要以下几种：

*   串行垃圾回收器（Serial Garbage Collector）：gc 单线程内存回收、会暂停所有用户线程
*   并行垃圾回收器（Parallel Garbage Collector）：收集是指多个 GC 线程并行工作，但此时用户线程是暂停的
*   并发标记扫描垃圾回收器（CMS Garbage Collector）：GC 线程和用户线程并发并发执行
*   G1 垃圾回收器（G1 Garbage Collector）：通过分区（Region） 和可预测的停顿模型，在吞吐量与延迟之间取得平衡的垃圾收集器
*   ZGC：基于着色指针和读屏障技术，实现全阶段并发，追求亚毫秒级极致低停顿的垃圾收集器

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDdJPTK2w1Q0Je0FtKxtMuibia0QJ5A7ByUHtUPO8IMq3LTsMv424QmibvQ/640?from=appmsg&watermark=1#imgIndex=24)

上图是 HotSpot 里的收集器，中间的横线表示分代，有连线表示可以组合使用。

常见垃圾回收器详细内容参考如下：

*   [美团面试：CMS 原理什么？漏标 + 多标 + 浮动垃圾 如何解决？90% 的程序员都答错了！](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247505019&idx=1&sn=9905bb2caad9178a330783a9b90fbdef&scene=21#wechat_redirect)
*   [美团面试：G1 垃圾回收 底层原理是什么？说说你的调优过程？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247503490&idx=1&sn=fe8dcd5a67b7bd7b1d5bebecd21a3086&scene=21#wechat_redirect)
*   [京东面试： 垃圾回收器 CMS、G1 区别是什么？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247505279&idx=1&sn=c80bb324794677953fed9c86958b93d3&scene=21#wechat_redirect)
*   [阿里面试：如何选 GC？ZGC 底层原理是什么？染色指针、转发表 是什么 ？90% 的程序员都答错了！](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247505032&idx=1&sn=24bffe9fdff44d710021109fcdc7a9aa&scene=21#wechat_redirect)
*   [大厂（转转、携程、京东）都用分代 ZGC，卡顿降低 20 倍，吞吐量提升 4 倍。分代 ZGC 这么牛？底层原理是什么？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247505075&idx=1&sn=dbf6f4ab6cd846f62f287cd2fe9fe33d&scene=21#wechat_redirect)

Java 程序的执行过程是一个从高级语言到机器指令的转换过程。

执行引擎（Execution Engine）是 JVM 的核心组件，负责将加载到内存中的字节码转换为机器指令并执行。

执行引擎（Execution Engine） 的设计直接影响着 Java 程序的运行效率和性能表现。

执行引擎在 JVM 架构中扮演着 "CPU" 的角色，它直接负责字节码的解释和执行。

与物理 CPU 不同的是，JVM 执行引擎需要处理平台无关的字节码，并将其转换为特定平台的本地机器指令。

## 1、解释执行和编译执行

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpD8m1ms7XfSlY2fJ2xrgc2Ub8vicpL6N3iaOssZRtSQNKMicZ4FiaoJf4rhQ/640?from=appmsg&watermark=1#imgIndex=25)

JVM 采用解释执行和编译执行相结合的混合模式，兼顾启动速度和长期运行性能。

```
    A[Java源代码] --> B[编译器编译]
    B --> C[字节码文件.class]
    C --> D[类加载器加载]
    D --> E[执行引擎处理]
    F -->|首次执行| G[解释器逐行解释执行]
    F -->|热点代码| H[JIT编译器编译]
    H --> J[生成本地代码]
    J --> K[本地代码执行]
    K --> L
    style H fill:#e8f5e9


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDF7skiakJpEqm4iaiayMkHFyvxdED1wBdYduXE3k7Zh1uibqG2KibDDsu8MQ/640?from=appmsg&watermark=1#imgIndex=26)

**1）解释器（Interpreter）：**

解释器逐条读取字节码指令，逐条解释并执行。这种方式启动速度快，但执行效率相对较低。

**工作特点**：

*   每次执行都需要重新解释字节码
*   无需等待编译时间，立即执行
*   适合执行频率不高的代码

**2）JIT 编译器（Just-In-Time Compiler）**

JIT 编译器将热点代码（频繁执行的代码）编译为本地机器码，并缓存起来供后续直接执行。

**工作流程**：

**(1) 监控方法执行频率**

**(2) 识别热点代码（HotSpot）**

**(3) 将字节码编译为优化后的本地代码**

**(4) 替换原有的解释执行路径**

```
    // 频繁调用的方法会成为JIT编译的候选
    public void hotMethod() {
        for (int i = 0; i < 100000; i++) {
            // 循环内的代码可能被JIT深度优化
            processData(i);
        }
    }
        // 方法内容
    }
}


```

HotSpot VM 是 Oracle JDK 默认的虚拟机，其名称来源于就是它能够识别和优化 "热点" 代码的能力。

**热点检测机制**：

*   方法调用计数器：统计方法被调用的次数
*   回边计数器：统计循环体执行的次数
*   默认阈值：Client 模式 1500 次，Server 模式 10000 次

## 2、编译优化技术

### 2.1 早期编译：javac 编译器

javac 编译器将 Java 源代码编译为字节码文件，在这个过程中会进行一些基础优化。

**主要的编译过程**：

**(1) 词法、语法分析**

**(2) 填充符号表**

**(3) 注解处理**

**(4) 语义分析与字节码生成**

```
public class SyntaxSugar {
    public void demo() {
        // 源代码中的语法糖
        Integer integer = 100; // 自动装箱：Integer.valueOf(100)
        int primitive = integer; // 自动拆箱：integer.intValue()
        Integer integer2 = Integer.valueOf(100);
        int primitive2 = integer2.intValue();
    }
}


```

### 2.2 晚期编译：JIT 深度优化

JIT 编译器在运行时进行深度优化，生成高度优化的本地代码。

#### 分层编译策略

现代 JVM 采用分层编译策略，平衡启动性能和运行期性能：

```
java -server -jar application.jar
java -client -jar application.jar
java -XX:+TieredCompilation -jar application.jar
java -XX:CompileThreshold=5000 -jar application.jar
java -XX:ReservedCodeCacheSize=256m -jar application.jar


```

#### JIT 优化技术

**方法内联（Method Inlining）**

方法内联是 JIT 最重要的优化技术之一，它通过将小方法调用替换为方法体本身来消除方法调用开销。

方法内联（Method Inlining）将目标方法的代码 “复制” 并“粘贴”到调用它的地方，从而完全消除方法调用的开销

```
    public void processData() {
        for (int i = 0; i < 100000; i++) {
            // JIT会评估是否内联helperMethod
            int result = helperMethod(i);
            // ... 使用result
        }
    }
    // 1. 方法体较小（默认小于35字节）
    // 2. 调用频率高
    // 3. 非虚方法（static/private/final/构造函数）
    private int helperMethod(int value) {
        // 简单操作更容易被内联
        return value * 2 + 1;
    }
    // 1. 方法体过大（可通过-XX:MaxInlineSize调整）
    // 2. 虚方法（需要运行时确定具体实现）
    public virtual void largeMethod() {
        // 复杂逻辑，方法体过大
        // 不会被内联
    }
}


```

**内联配置参数**：

```
-XX:MaxInlineSize=35        # 最大内联方法字节数
-XX:FreqInlineSize=325      # 频繁调用方法的内联大小限制
-XX:MaxInlineLevel=15       # 最大内联嵌套深度
-XX:+PrintInlining


```

**逃逸分析（Escape Analysis）**

逃逸分析确定对象的作用域范围，对于未逃逸出方法的对象可以进行栈上分配、标量替换等优化。

```
    public void testEscape() {
        // 情况1：无逃逸 - 对象完全在方法内部使用
        NoEscapeObject obj1 = new NoEscapeObject();
        obj1.value = computeValue();
        System.out.println(obj1.value);
        // obj1没有逃逸，可能进行栈上分配或标量替换
        EscapeObject obj2 = createObject();
        // obj2逃逸到调用者，不能在栈上分配
        synchronized (this) {
            ThreadEscapeObject obj3 = new ThreadEscapeObject();
            new Thread(() -> {
                obj3.value = 100; // 线程逃逸
            }).start();
        }
    }
        return new EscapeObject(); // 方法逃逸
    }
    static class NoEscapeObject {
        int value;
    }
    static class EscapeObject {
        int value;
    }
    static class ThreadEscapeObject {
        int value;
    }
}


```

**基于逃逸分析的优化**：

**(1) 栈上分配（Stack Allocation）：对于未逃逸对象，直接在栈帧中分配内存，避免堆分配和 GC 开销**

**(2) 标量替换（Scalar Replacement）：将对象分解为基本类型字段，消除对象头开销**

**(3) 锁消除（Lock Elision）：对于线程局部的未逃逸对象，移除不必要的同步操作**

**循环优化（Loop Optimization）**

JIT 编译器对循环结构进行深度优化，显著提升数值计算和数组处理性能。

```
    public void optimizedLoops(int[] array) {
        // 1. 循环展开（Loop Unrolling）
        // 原始循环：
        for (int i = 0; i < array.length; i++) {
            array[i] = array[i] * 2;
        }
        for (int i = 0; i < array.length; i += 4) {
            array[i] = array[i] * 2;
            array[i+1] = array[i+1] * 2;
            array[i+2] = array[i+2] * 2;
            array[i+3] = array[i+3] * 2;
        }
        for (int i = 0; i < array.length; i++) {
            // JIT可能消除数组范围检查，因为i已经在安全范围内
            array[i] = processElement(array[i]);
        }
        // 处理循环开头和结尾的特殊情况
    }
        return value + 1;
    }
    public void vectorizedLoop(float[] a, float[] b, float[] c) {
        // JIT可能使用SIMD指令并行处理多个数据元素
        for (int i = 0; i < a.length; i++) {
            c[i] = a[i] + b[i];
        }
    }
}


```

## 1、字节码增强概述

字节码增强技术是一类直接对已编译的字节码进行修改或动态生成全新字节码文件的技术。

这种技术使得开发者能够在不同阶段干预程序的执行行为，而无需修改源代码。

字节码增强主要分为两大类：**静态增强**（编译期或类加载前）和**动态增强**（运行时），如下图所示：

```
    A[字节码增强技术] --> B[静态增强]
    A --> C[动态增强]
    B --> B2[类加载期增强]
    C --> C2[运行时重转换]
    B2 --> B21[ASM<br>Javassist<br>Byte Buddy]
    C2 --> C32[Agent<br>动态Attach]


```

字节码增强主要应用于以下场景：

*   AOP（面向切面编程）实现
*   热部署与热修复
*   性能监控和数据采集
*   动态代理
*   Mock 测试

## 2、静态字节码增强

静态字节码增强是 Java 生态中一项强大的技术，它在**编译期或类加载前**对字节码进行修改，为应用程序注入额外功能而不需要改动源代码。

### 2.1 ASM：底层字节码操作框架

ASM 是一个轻量级但功能强大的 Java 字节码操作框架，它可以直接生成或转换编译后的 Java 类。

ASM 被广泛应用于许多知名项目，如 CGLIB、Spring、Groovy 等。

ASM 属于静态增强发生在类加载之前，包括编译期和类加载期两个阶段。

```
    participant Src as 源代码
    participant Compiler as 编译器
    participant Enhancer as 字节码增强器
    participant JVM as JVM
    Compiler->>Enhancer: 生成原始字节码
    Enhancer->>Enhancer: 分析并修改字节码
    Enhancer->>JVM: 增强后的字节码
    JVM->>JVM: 加载并执行增强类


```

#### ASM 核心 API 设计

ASM 提供了两套 API 来处理字节码：Core API 和 Tree API，可以类比 XML 解析的 SAX 方式和 DOM 方式，它们分别对应不同的使用场景和编程模式。

**Core API** 采用基于访问者的设计模式，类似于 SAX 解析 XML 的方式，以流式处理字节码，内存效率高但编程难度较大：

Core API 使用的是**访问者模式**，访问者模式主要用于修改或操作一些数据结构比较稳定的数据，当然字节码文件的结构是由 JVM 固定的，所以很适合利用访问者模式对字节码文件进行修改。在 Core API 中有以下几个关键类：

*   ClassReader：用于读取已经编译好的. class 文件。
*   ClassWriter：用于重新构建编译后的类，如修改类名、属性以及方法，也可以生成新的类的字节码文件。
*   各种 Visitor 类：如上所述，CoreAPI 根据字节码从上到下依次处理，对于字节码文件中不同的区域有不同的 Visitor，比如用于访问方法的 MethodVisitor、用于访问类变量的 FieldVisitor、用于访问注解的 AnnotationVisitor 等。为了实现 AOP，重点要使用的是 MethodVisitor。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDdJovcpXcMZMNQgbY9iaH96tqLBDNgpo3eId6ZGhDL3DEMDffdyAIexw/640?from=appmsg&watermark=1#imgIndex=27)

```
import org.objectweb.asm.ClassReader;
import org.objectweb.asm.ClassWriter;
import org.objectweb.asm.ClassVisitor;
import org.objectweb.asm.MethodVisitor;
import org.objectweb.asm.Opcodes;
    public static void main(String[] args) {
        // ClassReader: 读取字节码
        ClassReader reader = new ClassReader("java.lang.String");
        ClassWriter writer = new ClassWriter(ClassWriter.COMPUTE_MAXS);
        ClassVisitor visitor = new CustomClassVisitor(writer);
        reader.accept(visitor, 0);
        byte[] transformedClass = writer.toByteArray();
    }
}


```

**Tree API** 则采用基于树状结构的设计，类似于 DOM 解析 XML，将整个类结构加载到内存中，编程更简单但消耗更多内存：

```
import org.objectweb.asm.tree.ClassNode;
import org.objectweb.asm.tree.MethodNode;
    public static void main(String[] args) {
        // 将类解析为ClassNode
        ClassNode classNode = new ClassNode();
        ClassReader reader = new ClassReader("java.lang.String");
        reader.accept(classNode, 0);
        for (MethodNode method : classNode.methods) {
            if ("toString".equals(method.name)) {
                // 修改方法逻辑
            }
        }
        ClassWriter writer = new ClassWriter(ClassWriter.COMPUTE_MAXS);
        classNode.accept(writer);
        byte[] transformedClass = writer.toByteArray();
    }
}


```

一般情况下选型 ASM 就是为了追求极致性能，基本是都是采用 Core API 方式

#### 使用 ASM 实现简单 AOP 案例

下面我们通过一个完整示例展示如何使用 ASM 实现方法级别的 AOP 增强。首先定义需要增强的基础类：

```
public class Base {
    public void process(){
        System.out.println("process");
    }
}


```

为了利用 ASM 实现 AOP，需要定义两个类：

*   MyClassVisitor 类，用于对字节码的 Visit 以及修改；
*   Generator 类，在这个类中定义 ClassReader 和 ClassWriter，其中的逻辑是，classReader 读取字节码，然后交给 MyClassVisitor 类处理，处理完成后由 ClassWriter 写字节码并将旧的字节码替换掉。

Generator 类较简单，我们先看一下它的实现，如下所示，然后重点解释 MyClassVisitor 类。

```
import org.objectweb.asm.ClassVisitor;
import org.objectweb.asm.ClassWriter;
import java.io.File;
import java.io.FileOutputStream;
    public static void main(String[] args) throws Exception {
        // 1. 创建ClassReader读取目标类的字节码
        // "meituan/bytecode/asm/Base"是要修改的类的全限定名（内部格式）
        ClassReader classReader = new ClassReader("meituan/bytecode/asm/Base");
        // COMPUTE_MAXS参数让ASM自动计算最大栈大小和局部变量表大小
        ClassWriter classWriter = new ClassWriter(ClassWriter.COMPUTE_MAXS);
        // MyClassVisitor负责实际的字节码修改逻辑
        ClassVisitor classVisitor = new MyClassVisitor(classWriter);
        // SKIP_DEBUG参数表示跳过调试信息（行号、局部变量名等）
        classReader.accept(classVisitor, ClassReader.SKIP_DEBUG);
        byte[] data = classWriter.toByteArray();
        File f = new File("operation-server/target/classes/meituan/bytecode/asm/Base.class");
        try (FileOutputStream fout = new FileOutputStream(f)) {
            fout.write(data);
        }
    }
}


```

MyClassVisitor 继承自 ClassVisitor，用于对字节码的观察。

它还包含一个内部类 MyMethodVisitor，继承自 MethodVisitor 用于对类内方法的观察，整体代码如下：

```
import org.objectweb.asm.MethodVisitor;
import org.objectweb.asm.Opcodes;
 * 自定义类访问器，用于增强类的方法字节码
/
public class MyClassVisitor extends ClassVisitor implements Opcodes {
     * 构造函数
     * @param cv 委托的ClassVisitor
/
    public MyClassVisitor(ClassVisitor cv) {
        // 使用ASM5版本，并传入委托的ClassVisitor
        super(ASM5, cv);
    }
     * 访问类的基本信息
     * @param version 类版本
     * @param access 访问修饰符
     * @param name 类名
     * @param signature 泛型签名
     * @param superName 父类名
     * @param interfaces 实现的接口
/
    @Override
    public void visit(int version, int access, String name, String signature,
                      String superName, String[] interfaces) {
        // 将类信息传递给下一个访问器（不做修改）
        cv.visit(version, access, name, signature, superName, interfaces);
    }
     * 访问类中的方法
     * @param access 方法访问修饰符
     * @param name 方法名
     * @param desc 方法描述符
     * @param signature 方法泛型签名
     * @param exceptions 方法抛出的异常
     * @return 方法访问器
/
    @Override
    public MethodVisitor visitMethod(int access, String name, String desc, 
                                    String signature, String[] exceptions) {
        // 获取原始方法访问器
        MethodVisitor mv = cv.visitMethod(access, name, desc, signature, exceptions);
        //Base类中有两个方法：无参构造以及process方法，这里不增强构造方法
        if (!name.equals("<init>") && mv != null) {
            // 为每个非构造方法创建自定义方法访问器
            mv = new MyMethodVisitor(mv);
        }
        return mv;
    }
     * 自定义方法访问器，实现字节码增强
/
    class MyMethodVisitor extends MethodVisitor implements Opcodes {
        /
         * 构造函数
         * @param mv 委托的MethodVisitor
/
        public MyMethodVisitor(MethodVisitor mv) {
            super(Opcodes.ASM5, mv);
        }
         * 访问方法代码开头
         * 在此处插入"start"打印语句
/
        @Override
        public void visitCode() {
            super.visitCode(); // 调用父类方法
            mv.visitFieldInsn(GETSTATIC, "java/lang/System", "out", "Ljava/io/PrintStream;");
            mv.visitLdcInsn("start");
            mv.visitMethodInsn(INVOKEVIRTUAL, "java/io/PrintStream", "println", "(Ljava/lang/String;)V", false);
        }
         * 访问方法指令
         * @param opcode 操作码
/
        @Override
        public void visitInsn(int opcode) {
            // 检查是否为返回指令或抛出异常指令
            if ((opcode >= Opcodes.IRETURN && opcode <= Opcodes.RETURN)
                    || opcode == Opcodes.ATHROW) {
                // 在返回前插入"end"打印语句
                mv.visitFieldInsn(GETSTATIC, "java/lang/System", "out", "Ljava/io/PrintStream;");
                mv.visitLdcInsn("end");
                mv.visitMethodInsn(INVOKEVIRTUAL, "java/io/PrintStream", "println", "(Ljava/lang/String;)V", false);
            }
            // 继续处理原始指令
            mv.visitInsn(opcode);
        }
    }
}


```

利用这个类就可以实现对字节码的修改。关键步骤：

*   首先通过 MyClassVisitor 类中的 visitMethod 方法，判断当前字节码读到哪一个方法了。跳过构造方法 "" 后，将需要被增强的方法交给内部类 MyMethodVisitor 来进行处理。
*   接下来，进入内部类 MyMethodVisitor 中的 visitCode 方法，它会在 ASM 开始访问某一个方法的 Code 区时被调用，重写 visitCode 方法，将 AOP 中的前置逻辑就放在这里。
*   MyMethodVisitor 继续读取字节码指令，每当 ASM 访问到无参数指令时，都会调用 MyMethodVisitor 中的 visitInsn 方法。我们判断了当前指令是否为无参数的 “return” 指令，如果是就在它的前面添加一些指令，也就是将 AOP 的后置逻辑放在该方法中。
*   综上，重写 MyMethodVisitor 中的两个方法，就可以实现 AOP 了，而重写方法时就需要用 ASM 的写法，手动写入或者修改字节码。通过调用 methodVisitor 的 visitXXXXInsn()方法就可以实现字节码的插入，XXXX 对应相应的操作码助记符类型，比如 mv.visitLdcInsn("end")对应的操作码就是 ldc "end"，即将字符串 “end” 压入栈。

完成这两个 Visitor 类后，运行 Generator 中的 main 方法完成对 Base 类的字节码增强，增强后的结果可以在编译后的 Target 文件夹中找到 Base.class 文件进行查看，可以看到反编译后的代码已经改变了（如图 18 左侧所示）。然后写一个测试类 MyTest，在其中 new Base()，并调用 base.process() 方法，可以看到下图右侧所示的 AOP 实现效果：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDtevbtthycDI2OQlJXZaOrAbRq41MpJt8reemPJkZoA8hYKpD4bjUww/640?from=appmsg&watermark=1#imgIndex=28)

#### ASM 开发工具支持

利用 ASM 手写字节码时，需要利用一系列 visitXXXXInsn() 方法来写对应的助记符，所以需要先将每一行源代码转化为一个个的助记符，然后通过 ASM 的语法转换为 visitXXXXInsn() 这种写法。

第一步将源码转化为助记符就已经够麻烦了，不熟悉字节码操作集合的话，**需要我们将代码编译后再反编译**，才能得到源代码对应的助记符。第二步利用 ASM 写字节码时，如何传参也很令人头疼。ASM 社区也知道这两个问题，所以提供了工具 ASM ByteCode Outline。

安装后，右键选择 “Show Bytecode Outline”，在新标签页中选择“ASMified” 这个 tab，如图所示，就可以看到这个类中的代码对应的 ASM 写法了。图中上下两个红框分别对应 AOP 中的前置逻辑于后置逻辑，将这两块直接复制到 Visitor 中的 visitMethod()以及 visitInsn()方法中，就可以了。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDqUzUeCtia6Sz3kNtjYD4w7b5qPwbs8OlFNP7cFIKIBr3kSB4It5iaOPQ/640?from=appmsg&watermark=1#imgIndex=29)

### 2.2 Javassist：源码级字节码操作

Javassist 提供了更高级的 API，允许开发者以 Java 源代码的形式操作字节码，无需深入了解 JVM 指令集。

Javassist 的核心类包括：

*   **ClassPool**

    Ct 对象的容器，负责管理 CtClass 实例

*   **CtClass**

    编译时类信息，代表一个类文件

*   **CtMethod**

    类中的方法

*   **CtField**

    类中的字段

看一个简单的案例：

```
    public static void main(String[] args) throws Exception {
        // 获取默认ClassPool
        ClassPool pool = ClassPool.getDefault();
        CtClass ctClass = pool.get("com.example.BaseService");
        CtMethod method = ctClass.getDeclaredMethod("process");
        method.insertBefore("System.out.println(\"Javassist: 方法开始\");");
        method.insertAfter("System.out.println(\"Javassist: 方法结束\");");
        ctClass.writeFile("target/classes");
        Class<?> clazz = ctClass.toClass();
        BaseService service = (BaseService) clazz.newInstance();
        service.process();
    }
}


```

## 2、动态字节码增强

### 2.1 动态增强的核心挑战

上一小节两个案例都是静态字节码增强，也就是不涉及 JVM 运行时的类重载

在实际生产环境中，我们经常需要在 JVM 运行时对已加载的类进行增强，而不是在类加载前。

这种需求常见于线上诊断、热修复和性能监控等场景。

然而，JVM 的类加载机制对此有着严格的限制。

比如对上面 Javassist 的 Demo 中 main() 方法的第一行添加 Base b=new Base()，我们可以清晰看到这个限制：

```
        // 1. 创建原始Base实例,加上会报错
        Base b = new Base();
        b.process(); // 调用原始方法
        ClassPool cp = ClassPool.getDefault(); // 获取类池实例
        CtClass cc = cp.get("meituan.bytecode.asm.Base"); // 获取目标类的CtClass表示
        CtMethod m = cc.getDeclaredMethod("process"); // 获取process方法
        m.insertBefore("{ System.out.println(\"start\"); }"); // 在方法开始前插入代码
        m.insertAfter("{ System.out.println(\"end\"); }"); // 在方法结束后插入代码
        Class c = cc.toClass(); // 将CtClass转换为Java Class对象
        cc.writeFile("/Users/zen/projects"); // 将增强后的字节码写入文件系统
        Base h = (Base) c.newInstance(); // 创建增强后的类实例
        h.process(); // 调用增强后的方法
    }
}


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDdcicGb1LfhhKMKP6tOiaU7DGibjpPAZ3iadxgBKYkAT3BzMQzIgZPr6OLQ/640?from=appmsg&watermark=1#imgIndex=30)

这个错误揭示了 JVM 的核心安全机制：**同一个类加载器对同一个类全限定名只能定义一次**。

这种设计保证了运行时环境的稳定性，但也给动态增强带来了挑战。

显然，如果只能在类加载前对类进行强化，那字节码增强技术的使用场景就变得很窄了。我们期望的效果是：在一个持续运行并已经加载了所有类的 JVM 中，还能利用字节码增强技术对其中的类行为做替换并重新加载。

为了模拟这种情况，我们将 Base 类做改写，在其中编写 main 方法，每五秒调用一次 process() 方法，在 process() 方法中输出一行 “process”。

我们的目的就是，在 JVM 运行中的时候，将 process()方法做替换，在其前后分别打印 “start” 和“end”。也就是在运行中时，每五秒打印的内容由 "process" 变为打印 "start process end"。

那如何解决 JVM 不允许运行时重加载类信息的问题呢？为了达到这个目的，我们接下来一一介绍需要借助的 Java 的 instrument 类库。

```
    public static void main(String[] args) {
        String name = ManagementFactory.getRuntimeMXBean().getName();
        String s = name.split("@")[0];
        //打印当前Pid
        System.out.println("pid:"+s);
        while (true) {
            try {
                Thread.sleep(5000L);
            } catch (Exception e) {
                break;
            }
            process();
        }
    }
        System.out.println("process");
    }
}


```

### 2.2 Instrumentation API

为了突破这个限制，需要使用 java 的 instrument 库，Instrument 是 JVM 提供的一个可以修改已加载类的类库，专门为 Java 语言编写的插桩服务提供支持。它需要依赖 JVMTI 的 Attach API 机制实现，JVMTI 这一部分，在后面进行介绍。

在 JDK 1.6 以前，Instrument 只能在 JVM 刚启动开始加载类时生效，而在 JDK 1.6 之后，Instrument 支持了在运行时对类定义的修改。

要使用 Instrument 的类修改功能，我们需要实现它提供的 ClassFileTransformer 接口，定义一个类文件转换器。接口中的 transform() 方法会在类文件被加载时调用，而在 Transform 方法里，我们可以利用上文中的 ASM 或 Javassist 对传入的字节码进行改写或替换，生成新的字节码数组后返回。

我们定义一个实现了 ClassFileTransformer 接口的类 TestTransformer，依然在其中利用 Javassist 对 Base 类中的 process()方法进行增强，在前后分别打印 “start” 和“end”，代码如下：

```
 * 自定义类文件转换器，用于在类加载时修改字节码
 * 实现ClassFileTransformer接口，可在Java Agent中使用
/
public class TestTransformer implements ClassFileTransformer {
     * 类文件转换方法，在类加载时被JVM调用
/
    @Override
    public byte[] transform(ClassLoader loader, String className, 
                           Class<?> classBeingRedefined, 
                           ProtectionDomain protectionDomain, 
                           byte[] classfileBuffer) {
        // 打印正在转换的类名（注意：className使用JVM内部格式）
        System.out.println("Transforming " + className);
            // 1. 获取Javassist类池实例
            ClassPool cp = ClassPool.getDefault();
            CtClass cc = cp.get("meituan.bytecode.jvmti.Base");
            CtMethod m = cc.getDeclaredMethod("process");
            m.insertBefore("{ System.out.println(\"start\"); }");
            m.insertAfter("{ System.out.println(\"end\"); }");
            return cc.toBytecode();
        } catch (Exception e) {
            // 7. 打印异常信息（在实际应用中应考虑更完善的错误处理）
            e.printStackTrace();
        }
        return null;
    }
}


```

### 2.2 Java Agent 技术

动态字节码增强，还需要需要使用 java agent 技术

Java Agent 是 Java 平台提供的一种强大机制，它利用 JVM 提供的 `java.lang.instrument`API 来**拦截和修改类的字节码**，允许开发人员在 JVM 运行时**动态修改和增强已加载的类**，而无需修改源代码。

这项技术是许多高级工具和框架的基础，包括 APM（应用性能监控）、热部署工具、诊断工具等。

Java Agent 的工作流程可以通过以下序列图来理解：

```
    participant A as 应用程序
    participant J as JVM
    participant I as Instrumentation API
    participant T as Transformer
    participant B as 字节码操作库(ASM/Javassist)
    A->>J: java -javaagent:agent.jar -jar app.jar
    J->>I: 初始化Instrumentation
    I->>T: 调用premain方法
    T->>B: 注册字节码转换器
    Note over J, B: 类加载过程
    J->>T: 请求字节码转换
    T->>B: 修改字节码
    B->>J: 返回修改后字节码
    J->>A: 加载增强后的类
    A->>J: 程序正常运行
    J->>I: 通过Attach API加载Agent
    I->>T: 调用agentmain方法
    T->>B: 注册转换器并重转换类
    B->>J: 返回修改后字节码
    J->>A: 使用增强后的类


```

Java Agent 的两种加载方式：

#### 启动时 Agent（Premain 模式）

**实现方式：**

```
import java.lang.instrument.Instrumentation;
    /*
     * JVM 启动时加载Agent的入口方法
     * @param agentArgs 从命令行传递的参数
     * @param inst JVM 自动传入的Instrumentation实例
/
    public static void premain(String agentArgs, Instrumentation inst) {
        System.out.println("Premain Agent启动，参数: " + agentArgs);
        inst.addTransformer(new MyClassTransformer());
        inst.addTransformer(new MonitoringTransformer(), true);
    }
    public static void premain(String agentArgs) {
        System.out.println("Premain Agent简化版启动");
    }
}


```

**MANIFEST.MF 配置：**

```
Premain-Class: com.example.PreMainAgent
Can-Redefine-Classes: true
Can-Retransform-Classes: true


```

**使用方式：**

#### 运行时 Agent（Agentmain 模式）

**实现方式：**

```
import java.lang.instrument.Instrumentation;
import java.lang.instrument.UnmodifiableClassException;
    /*
     * 运行时加载Agent的入口方法
     * @param agentArgs 传递的参数
     * @param inst Instrumentation实例
/
    public static void agentmain(String agentArgs, Instrumentation inst) {
        System.out.println("动态Agent加载成功");
            // 添加转换器并指定可重转换
            inst.addTransformer(new DynamicTransformer(), true);
            Class<?>[] classes = inst.getAllLoadedClasses();
            for (Class<?> clazz : classes) {
                // 找到目标类并重新转换
                if (clazz.getName().equals("com.example.TargetClass")) {
                    inst.retransformClasses(clazz);
                    System.out.println("成功重转换类: " + clazz.getName());
                    break;
                }
            }
        } catch (UnmodifiableClassException e) {
            System.err.println("类重转换失败: " + e.getMessage());
        }
    }
}


```

**MANIFEST.MF 配置：**

```
Agent-Class: com.example.DynamicAgent
Can-Redefine-Classes: true
Can-Retransform-Classes: true


```

#### 案例实操

演示示例使用运行时 Agent，在真正的运行时，通过 Java Agent 机制将转换器注入到运行的 JVM 中。整个流程如下所示：

```
    participant A as 外部程序(Attacher)
    participant V as 目标JVM虚拟机
    participant I as Instrumentation
    participant T as Transformer
    participant C as 目标类(Base)
    A->>V: 2. 加载Agent jar包
    V->>I: 3. 初始化Instrumentation
    I->>T: 4. 注册ClassFileTransformer
    I->>C: 5. 触发类重定义(retransformClasses)
    C->>T: 6. 请求字节码转换
    T->>C: 7. 返回增强后的字节码
    I->>V: 8. 重新定义类
    V->>A: 9. 返回操作结果
    A->>V: 10. 断开连接


```

这个 Java Agent 能够在 JVM 运行时动态修改已加载的`Base`类的字节码，通过注册自定义转换器实现方法级别的功能增强，无需重启应用即可实现热更新。

```
 * Java Agent类，用于在运行时动态修改已加载的类
 * 通过Instrumentation API实现字节码增强
/
public class TestAgent {
     * Agent运行时入口方法，通过Attach API动态加载时调用
     * 
/
    public static void agentmain(String args, Instrumentation inst) {
        // 1. 注册自定义的类转换器，该转换器使用Javassist进行字节码操作
        // 第二个参数true表示此转换器支持类的重转换（retransformation）
        inst.addTransformer(new TestTransformer(), true);
            // 2. 触发Base类的重转换过程，JVM会调用所有已注册的转换器
            // 对Base类进行字节码修改，实现运行时增强
            inst.retransformClasses(Base.class);
            System.out.println("Agent Load Done.");
        } catch (Exception e) {
            // 4. 异常处理：打印加载失败信息
            // 在生产环境中应记录详细日志而非简单输出
            System.out.println("agent load failed!");
        }
    }
}


```

然后使用 maven 打包 Java Agent

```
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.2.0</version>
                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                        </manifest>
                        <manifestEntries>
                            <Agent-Class>meituan.bytecode.jvmti.TestAgent</Agent-Class>
                            <Can-Redefine-Classes>true</Can-Redefine-Classes>
                            <Can-Retransform-Classes>true</Can-Retransform-Classes>
                            <Boot-Class-Path>javassist-3.28.0-GA.jar</Boot-Class-Path>
                        </manifestEntries>
                    </archive>
                </configuration>
            </plugin>


```

### JVMTI 技术

#### 什么是 JPDA，JVMTI

要实现动态字节码增强，实现运行时 Agent，还需要 JVMTI。

JVMTI 是 JPDA 的一部分，下面简单介绍下

JPDA（Java Platform Debugger Architecture）。如果 JVM 启动时开启了 JPDA，那么类是允许被重新加载的。在这种情况下，已被加载的旧版本类信息可以被卸载，然后重新加载新版本的类。正如 JDPA 名称中的 Debugger，JDPA 其实是一套用于调试 Java 程序的标准，任何 JDK 都必须实现该标准。

JPDA 定义了一整套完整的体系，它将调试体系分为三部分，并规定了三者之间的通信接口。

三部分由低到高分别是 Java 虚拟机工具接口（JVMTI），Java 调试协议（JDWP）以及 Java 调试接口（JDI），三者之间的关系如下图所示

现在回到正题，运行时 Agent，借助 JVMTI 的就是一部分能力，帮助动态重载类信息。

JVM TI（JVM TOOL INTERFACE，JVM 工具接口）是 JVM 提供的一套对 JVM 进行操作的工具接口。通过 JVMTI 可以实现对 JVM 的多种操作，然后通过接口注册各种事件勾子。

在 JVM 事件触发时，同时触发预定义的勾子，以实现对各个 JVM 事件的响应，事件包括类文件加载、异常产生与捕获、线程启动和结束、进入和退出临界区、成员变量修改、GC 开始和结束、方法调用进入和退出、临界区竞争与等待、VM 启动与退出等等。

#### 案例实操

运行时 Agent，通常使用 Attach API（JVMTI） 提供 JVM 进程间通信的能力，比如说我们为了让另外一个 JVM 进程把线上服务的线程 Dump 出来，会运行 jstack 或 jmap 的进程，并传递 pid 的参数，告诉它要对哪个进程进行线程 Dump，这就是 Attach API 做的事情。

在下面，我们将通过 Attach API 的 loadAgent() 方法，将打包好的 Agent jar 包动态 Attach 到目标 JVM 上。具体实现起来的步骤如下：

实操案例我们使用 Attach API 动态加载 Agent，将我们上面打包好的 jar 包 Attach 到指定的 JVM pid 上，代码如下

```
    public static void main(String[] args) throws AttachNotSupportedException, IOException, AgentLoadException, AgentInitializationException {
        // 传入目标 JVM pid
        VirtualMachine vm = VirtualMachine.attach("10960");
        vm.loadAgent("target/jvm-demo2-1.0-SNAPSHOT.jar");
    }
}


```

注意需要依赖 tools.jar

```
        <dependency>
            <groupId>com.sun</groupId>
            <artifactId>tools</artifactId>
            <version>${java.version}</version>
            <scope>system</scope>
            <systemPath>${java.home}/../lib/tools.jar</systemPath>
        </dependency>


```

上面 agent.jar，由于在 MANIFEST.MF 中指定了 Agent-Class，所以在 Attach 后，目标 JVM 在运行时会走到 TestAgent 类中定义的 agentmain()方法，而在这个方法中，我们利用 Instrumentation，将指定类的字节码通过定义的类转化器 TestTransformer 做了 Base 类的字节码替换（通过 javassist），并完成了类的重新加载。由此，我们达成了 “在 JVM 运行时，改变类的字节码并重新载入类信息” 的目的。

以下为运行时重新载入类的效果：先运行 Base 中的 main() 方法，启动一个 JVM，可以在控制台看到每隔五秒输出一次 "process"。接着执行 Attacher 中的 main() 方法，并将上一个 JVM 的 pid 传入。此时回到上一个 main() 方法的控制台，可以看到现在每隔五秒输出 "process" 前后会分别输出 "start" 和 "end"，也就是说完成了运行时的字节码增强，并重新载入了这个类。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WMcQtQfSg5jNicySwCIwcrpDWWeMTcLndcaNIKfu3RRfBXLOoN5t34z1ibaW8E60nhrpzPWvr0kl45w/640?from=appmsg&watermark=1#imgIndex=31)

动态字节码增强技术为 Java 应用提供了强大的运行时诊断和修复能力，是构建 APM、热修复系统等技术的基础。掌握这一技术，能够帮助开发者更好地理解和优化 Java 应用的运行时行为。 

## 说在最后：有问题找老架构取经‍

按照此文的套路去回答，一定会 **吊打面试官，让面试官爱到 “不能自已、口水直流”**，然后实现”offer 直提”。

在面试之前，建议大家系统化的刷一波 5000 页《[尼恩 Java 面试宝典 PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》，里边有大量的大厂真题、面试难题、架构难题。

很多小伙伴刷完后，吊打面试官，大厂横着走。

在刷题过程中，如果有啥问题，大家可以来找 40 岁老架构师尼恩交流。

另外，如果没有面试机会，可以找尼恩来改简历、做帮扶。前段时间，**[指导一个小伙 暴涨 200% ，29 岁 / 7 年 / 双非一本  逆天改命。](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486433&idx=1&sn=0f630e8fc7620b3bf0e054827b48d91e&scene=21#wechat_redirect)**

跟着尼恩狠狠卷，实现 “offer 自由” 很容易的。

很多跟着尼恩 **卷硬核技术的小伙伴， offer 拿到手软**，实现真正的 “offer 自由” 。

## Java+AI  弯道超车： 跟着尼恩卷最新技术，占据技术领先地位，没有危机 

 [会 AI 的程序员，工资暴涨 50%！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486827&idx=1&sn=db3f268c6752ddb074a98c7b13a84011&scene=21#wechat_redirect)

[28 岁 / 6 年 / 被裁 1 年，收 3 大厂 offer ， 成 大厂 皇后 。2 本学历 51W 年薪，惊天 逆涨，涨薪 2 倍](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486987&idx=1&sn=ff977f450dd242446f228d3a6585e258&scene=21#wechat_redirect)，大厂皇后

[涨薪传奇： 18k->38K , 单月暴 20K，32 岁小伙伴 2 个月时间年薪 翻 1.5 倍 ，一步登天 + 逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486960&idx=1&sn=f57253a448694c32e834207381c42284&scene=21#wechat_redirect)

[低学历 传奇：29 岁 6 年专套本，受够了外包，狠卷 3 个月逆袭大厂 涨 1 倍， 逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486945&idx=1&sn=f6ff6231ccf3585624161c2ef1f50fdf&scene=21#wechat_redirect)

[极速上岸： 被裁 后， 8 天 拿下 京东，狠涨 一倍 年薪 48W， 小伙伴 就是 做对了一件事](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486913&idx=1&sn=edd6774e9d17c39327e8dcb95fdd68d9&scene=21#wechat_redirect)

[外包 + 二本 进 美团： 26 岁小 2 本 一步登天， 进了顶奢大厂（ 美团） ， 太爽了](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486904&idx=1&sn=e7c878f99ed101ebfbd1d80ec0e07401&scene=21#wechat_redirect)

[超牛的 Java+Al 双栖架构： 34 岁无路可走，一个月翻盘，拿 3 个架构 offer，靠  Java+Al   逆天改命！！！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486848&idx=1&sn=ff43d058271b0801f84aba3f3d957855&scene=21#wechat_redirect)

java+AI 逆袭 2：[：3 年 程序媛 被裁， 25W-》40W 上岸， 逆涨 60%。 Java+AI 太神了， 架构小白 2 个月逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486858&idx=1&sn=9cd122c18d166433b43d0952d3a7a7a8&scene=21#wechat_redirect)

[Java+AI 逆袭 3 ： 36 岁 / 失业 7 个月 / 彻底绝望 。狠卷 3 个月 Java+AI ，终于逆风翻盘，顺利 上岸](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486870&idx=1&sn=990a540a155cdb8254ebce6c75816882&scene=21#wechat_redirect)

[Java+AI 逆袭 ： 闲了一年，41 岁 / 失业 12 个月 / 彻底绝望 。狠卷 2 个月 Java+AI ，终于逆风翻盘](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486880&idx=1&sn=a37033157c766ac5ae7c9195fe776693&scene=21#wechat_redirect)

[Java+AI 逆袭 5：1 个月大涨 2.5W，37 岁 脱坑外包， 入了正编，GO+AI 要逆天了](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486885&idx=1&sn=4e26fbb093f45d437dedf14ea9b9e6c5&scene=21#wechat_redirect)

**职业救助站**

实现职业转型，极速上岸

![](https://mmbiz.qpic.cn/mmbiz_jpg/9qY2Gt3PBs3icE6WcM4nBOdRRcczOW4wkNDgCMZhBfqeet9suwGHfgQduP33YYqqZLOppecMI72Pv5ibNkPeIk6g/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=32)

关注**职业救助站**公众号，获取每天职业干货  
助您实现**职业转型、职业升级、极速上岸**  
---------------------------------

**技术自由圈**

实现架构转型，再无中年危机

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/xlgvgPaib7WMtyFcJx45oIeWL1d6AENcdoAvuTPuq46E9n8pPe4SHXoSicuEVvhibHRSIzmEBPlC7POcDaqD6ID7w/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=33)

关注**技术自由圈**公众号，获取每天技术千货  
一起成为牛逼的**未来超级架构师**

**几十篇架构笔记、5000 页面试宝典、20 个技术圣经  
请加尼恩个人微信免费拿走**

**暗号，请在公众号后台发送消息：领电子书**

如有收获，请点击底部的 " 在看 "和" 赞 "，谢谢