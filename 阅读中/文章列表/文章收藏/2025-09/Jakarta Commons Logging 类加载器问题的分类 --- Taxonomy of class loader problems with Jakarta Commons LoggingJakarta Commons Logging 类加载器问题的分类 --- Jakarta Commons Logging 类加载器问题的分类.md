---
source: https://articles.qos.ch/classloader.html
create: 2025-09-22 16:32
read: true
knowledge: true
knowledge-date: 2025-09-30
tags:
  - Java
summary: "[[Jakarta Commons Logging 类加载器问题的分类 --- Taxonomy of class loader problems with Jakarta Commons LoggingJakarta Commons Logging 类加载器问题的分类 --- Jakarta Commons Logging 类加载器问题的分类]]"
---
## Types of problems 问题类型

Class loading problems encountered when using JCL fall into three main categories:  
使用 JCL 时遇到的类加载问题主要有三类：

*   **Type-I:** A `java.lang.NoClassDefFoundError` thrown when a class is inaccessible from a parent class loader even if the said class is available to a child class loader  
    **I 型：** A 当某个类无法从父类加载器访问时抛出 `java.lang.NoClassDefFoundError` ，即使该类可供子类加载器访问
*   **Type-II:** Assignment incompatibility of two classes loaded by distinct class loaders, even in case where the two classes are bit-wise identical.  
    **类型 II：** 两个类的分配不兼容 由不同的类加载器加载，即使两个 类按位相同。
*   **Type-III:** Holding references to a given class loader will prevent the resources loaded by that class loader from being garbage collected.  
    **类型 III：** 保存对给定类加载器的引用 将阻止该类加载器加载的资源 正在被垃圾回收。

At least on a superficial level, Type-I problems are relatively easy to understand whereas Type-II problems are a bit harder to grasp at first glance. Consequently, we will start by illustrating problems of Type-I. Given their different nature, Type-III problems will be discussed only briefly towards the end of this document.  
至少从表面上看，第一类问题相对容易理解，而第二类问题乍一看则略显困难。因此，我们将首先阐述第一类问题。鉴于两者的性质不同，本文结尾处将简要讨论第三类问题。

## Class loader trees 类加载器树

### Parent-first delegation model  
父母优先委托模型

As you probably know, a class loader usually has a reference to its parent. It follows that class loaders can be arranged into trees of class loaders. In order to retrieve a resource, a class loader will normally delegate to its _parent first_. The javadoc documentation of the `Classloader` class states:  
您可能知道，类加载器通常引用其父类加载器。因此，类加载器可以排列成类加载器树。为了获取资源，类加载器通常会_首先委托给其父类加载器_ 。Classloader `Classloader` 的 javadoc 文档 状态：

The `ClassLoader` class uses a delegation model to search for classes and resources. Each instance of `ClassLoader` has an associated parent class loader. When called upon to find a class or resource, a `ClassLoader` instance will delegate the search for the class or resource to its parent class loader before attempting to find the class or resource itself. The virtual machine's built-in class loader, called the bootstrap class loader, does not itself have a parent but may serve as the parent of a `ClassLoader` instance.  
`ClassLoader` 类使用委托模型来搜索类和资源。每个 `ClassLoader` 实例都有一个关联的父类。 类加载器。当需要查找类或资源时， `ClassLoader` 实例会先将类或资源的搜索委托给其父类加载器，然后再尝试自行查找。虚拟机的内置类加载器（称为引导类加载器）本身没有父类加载器，但可以作为 `ClassLoader` 实例的父类加载器。

Thus, parent-first delegation is the default and recommended delegation model in Java.  
因此，父级优先委托是 Java 中默认且推荐的委托模型。

### Child-first delegation model  
儿童优先委托模式

However, under certain circumstances, a child class loader may prefer to first attempt to load resources on its own, and only later to delegate to its parent. We refer to this kind of class loaders as following the _child-first_ delegation model. The Java Servlet Specification Version 2.4 states the following:  
然而，在某些情况下，子类加载器可能倾向于先尝试自己加载资源，然后再委托给其父类加载器。我们将这种类加载器称为遵循 _“子优先_委托”原则的加载器。 模型。Java Servlet 规范 2.4 版规定 下列的：

**SRV.9.7.2 Web Application Classloader  
SRV.9.7.2 Web 应用程序类加载器**

The classloader that a container uses to load a servlet in a WAR must allow the developer to load any resources contained in library JARs within the WAR following normal J2SE semantics using `getResource`. It must not allow the WAR to override J2SE or Java servlet API classes. It is further recommended that the loader not allow servlets in the WAR access to the web container's implementation classes. _It is recommended also that the application class loader be implemented so that classes and resources packaged within the WAR are loaded in preference to classes and resources residing in container-wide library JARs._  
容器用于加载 WAR 中 servlet 的类加载器必须允许开发人员使用 `getResource` 按照常规 J2SE 语义加载 WAR 中库 JAR 中包含的任何资源。它不得允许 WAR 覆盖 J2SE 或 Java servlet API 类。此外，建议加载器不允许 WAR 中的 servlet 访问 Web 容器的实现类。 _此外，建议应用程序类加载器的实现方式是，优先加载 WAR 中打包的类和资源，而不是加载容器级库 JAR 中的类和资源。_

Note that the Servlet Specification only recommends child-first delegation order. In practice, not all Servlet Containers implement the child-first delegation model. Both delegation models are fairly common in practice.  
请注意，Servlet 规范仅推荐使用 child-first 委托顺序。实际上，并非所有 Servlet 容器都实现了 child-first 委托模型。这两种委托模型在实践中都相当常见。

## Simple class loader implementations for both models  
两种模型的简单类加载器实现

In order to write small test cases reproducing class loader problems, we need one class loader implementation for the parent-first delegation model and another implementation for the child-first model. Fortunately, a class loader for the parent-first delegation model is readily available. Not only does [java.net.URLClassLoader](http://java.sun.com/j2se/1.4.2/docs/api/java/net/URLClassLoader.html) implement the parent-first model, it is also very convenient to use.  
为了编写小型测试用例来重现类加载器 问题，我们需要一个类加载器实现 父母优先委托模型和另一种实现 child-first 模型。幸运的是， 父级优先委托模型很容易实现。不仅 [java.net.URLClassLoader](http://java.sun.com/j2se/1.4.2/docs/api/java/net/URLClassLoader.html) 实现父母优先模式，也非常方便 使用。

As far as I know, there is no class loader implementing the child-first model that ships with the JDK. However, it does not take much to write our own.  
据我所知，JDK 中没有实现 child-first 模型的类加载器。不过，我们自己写一个也并不费力。

```java
/**
 * An almost trivial no fuss implementation of a class loader 
 * following the child-first delegation model.
 * 
 * @author Ceki Gülcü
 */
public class ChildFirstClassLoader extends URLClassLoader {

  public ChildFirstClassLoader(URL[] urls) {
    super(urls);
  }

  public ChildFirstClassLoader(URL[] urls, ClassLoader parent) {
    super(urls, parent);
  }

  public void addURL(URL url) {
    super.addURL(url);
  }
  
  public Class loadClass(String name) throws ClassNotFoundException {
  	return loadClass(name, false);
  }

  /**
   * We override the parent-first behavior established by 
   * java.lang.Classloader.
   * 
   * The implementation is surprisingly straightforward.
   */
  protected Class loadClass(String name, boolean resolve)
    throws ClassNotFoundException {
  	
    // First, check if the class has already been loaded
    Class c = findLoadedClass(name);

    // if not loaded, search the local (child) resources
    if (c == null) {
    	try {
        c = findClass(name);
    	} catch(ClassNotFoundException cnfe) {
    	  // ignore
    	}
    }

    // if we could not find it, delegate to parent
    // Note that we don't attempt to catch any ClassNotFoundException
    if (c == null) {
      if (getParent() != null) {
        c = getParent().loadClass(name);
      } else {
        c = getSystemClassLoader().loadClass(name);
      }
    }

    if (resolve) {
      resolveClass(c);
    }

    return c;
  }
}


```

In a nutshell, [`ChildFirstClassLoader`](https://articles.qos.ch/delegation/src/java/ch/qos/ChildFirstClassLoader.java) extends `java.net.URLClassLoader` and overrides its `loadClass()` method so that parent delegation occurs only _after_ an unsuccessful attempt to load a class locally.  
简而言之， [`ChildFirstClassLoader`](https://articles.qos.ch/delegation/src/java/ch/qos/ChildFirstClassLoader.java) 扩展 `java.net.URLClassLoader` 并覆盖其 `loadClass()` 方法，以便仅在本地加载类的尝试失败_后_才会发生父委托。

Now that we have the class loader implementations, we also need one or two classes to load and play with. We will play with various versions of the `Box` interface shown next.  
现在我们有了类加载器的实现，我们还需要加载一两个类来测试一下。接下来我们将测试一下 `Box` 接口的各种版本。

```java
package box;

/*
 * Box is a small two method interface. We don't really care about 
 * what implementations of the Box interface do, as long as they
 * implement its two methods.
 * 
 * @author Ceki Gülcü
 */
public interface Box {

  /**
  * Returns the version number of the implementing class.
  */
  public int get();

  /**
   * Perform some operation. We don't really care what it is.
   */
  public void doOp();
  
}
  

```

Here is a quasi-trivial implementation of `Box`.  
这是 `Box` 的一个简单实现。

```java
package box;

/**
 * A no brainer implementation of Box returning a number. This
 * number is set at compilation time by Ant. 
 *  
 * @author Ceki Gülcü
 */
public class BoxImpl implements Box  {

  /**
   * Create an instance of BoxImpl and print it.
   */
  static public void main(String[] args) {
    Box box = new BoxImpl();
    System.out.println("BoxImpl version is "+box.get());
  }

  /**
   * The appropriate Ant task replaces @V@ with an integer constant.
   */
  public int get() {
    return @V@;
  }

  /**
   * Print this instance as a string.
   */
  public void doOp() {
  	System.out.println(this);
  }
  
  /**
   * Returns this instance's string representation.
   */
  public String toString() {
   return "BoxImpl("+get()+")"; 
  }
}
  

```

In the [accompanying distribution](https://articles.qos.ch/delegation/cl-problems.zip) to this document, you will find 3 jar files. The file _boxAPI.jar_ includes _Box.class_ and nothing else. The file _box0.jar_ includes a version of the class `BoxImpl` which returns zero (0) in its `get()` method. The file _box1.jar_ includes a version of the class `BoxImpl` which returns one (1) in its `get()` method.  
在本文档的[随附分发包](https://articles.qos.ch/delegation/cl-problems.zip)中，您将找到 3 个 jar 文件 _。boxAPI.jar_ 文件包含 _Box.class_ ，没有其他内容。文件 _box0.jar_ 包含 `BoxImpl` 类的一个版本，该类的 `get()` 方法返回零 (0)。文件 _box1.jar_ 包含该类的一个版本 `BoxImpl` 在 `get()` 中返回 1 方法。

Executing the command 执行命令

java -cp boxAPI.jar;box0.jar box.BoxImpl

yields 产量

Box version is 0 盒装版本为0

Whereas executing the command  
执行命令

java -cp boxAPI.jar;box1.jar box.BoxImpl

yields 产量

Box version is 1 盒装版本为1

This shows that the jar files _box0.jar_ and _box1.jar_ contain implementations marked with different version numbers.  
这表明 jar 文件 _box0.jar_ 和 _box1.jar_ 包含标有不同标记的实现 版本号。

The java applications [`ChildFirstTest`](https://articles.qos.ch/delegation/src/java/ch/qos/test/ChildFirstTest.java) and [`ParentFirstTest`](https://articles.qos.ch/delegation/src/java/ch/qos/test/ParentFirstTest.java) included in the distribution demonstrate that `ChildFirstClassLoader` and `URLClassLoader` respectively implement the child-first and parent-first delegation mechanisms.  
Java 应用程序 [`ChildFirstTest`](https://articles.qos.ch/delegation/src/java/ch/qos/test/ChildFirstTest.java) 和 [`ParentFirstTest`](https://articles.qos.ch/delegation/src/java/ch/qos/test/ParentFirstTest.java) 包括在分布中表明 `ChildFirstClassLoader` 和 `URLClassLoader` 分别实现 child-first 和 parent-first 委托 机制。

They can be run as follows:  
它们可以按如下方式运行：

java -cp classes;boxAPI.jar;box0.jar ch.qos.test.ChildFirstTest  
java -cp 类；boxAPI.jar；box0.jar ch.qos.test.ChildFirstTest

and respectively as 分别为

java -cp classes;boxAPI.jar;box0.jar ch.qos.test.ParentFirstTest  
java -cp 类；boxAPI.jar；box0.jar ch.qos.test.ParentFirstTest

## JCL in parent-first class loader trees  
父级优先类加载器树中的 JCL

In parent-first class loader trees JCL suffers from problems of Type-I. To understand the causes, several facts must be carefully reviewed.  
在父类优先的类加载器树中，JCL 会遭遇 I 类问题。要理解其原因，必须仔细回顾几个事实。

JCL relies on dynamic discovery to determine the user's preferred logging API. In theory, log4j, `java.util.logging` and Avalon logkit are supported. In order to locate classes and resources, JCL's discovery mechanism relies _exclusively_ on the thread context class loader which is by definition the class loader returned by the [`getContextClassLoader()`](http://java.sun.com/j2se/1.4.2/docs/api/java/lang/Thread.html#getContextClassLoader()) method for the current thread. As long as the thread context class loader (TCCL) is not explicitly defined, it defaults to the system class loader, that is the class loader used to load the application.  
JCL 依靠动态发现来确定用户首选的日志 API。理论上，支持 log4j、 `java.util.logging` 和 Avalon logkit。为了定位类和资源，JCL 的发现机制_完全_依赖于线程上下文类加载器，根据定义，该加载器是由 [`getContextClassLoader()`](http://java.sun.com/j2se/1.4.2/docs/api/java/lang/Thread.html#getContextClassLoader()) 返回的类加载器。 当前线程的方法。只要线程上下文类 加载器（TCCL）没有明确定义，它默认为系统 类加载器，即用来加载应用程序的类加载器。

In the following three examples, the system class loader will have access to _boxAPI.jar_, _commons-logging.jar_ and the _classes/_ directory. An instance of `URLClassLoader` (parent-first delegation), will have the system class loader as its parent. It will also have access to _box1.jar_, _commons-logging.jar_ and _log4j.jar_.  
在以下三个示例中，系统类加载器将有权访问 _boxAPI.jar_ 、 _commons-logging.jar_ 和 _classes/_ 目录。 `URLClassLoader` （父级优先委托），将具有 系统类加载器作为其父类加载器。它还可以访问 _box1.jar_ 、 _commons-logging.jar_ 和 _log4j.jar_ 。

### Example-1 示例 1

This first example, [ParentFirstTestJCL0](https://articles.qos.ch/delegation/src/java/ch/qos/test/ParentFirstTestJCL0.java), will not explicitly set the TCCL. Thus, the TCCL will default to the system class loader. _Please note that this example does **not** illustrate a bug in JCL but rather serves as a warm up for the rest of the examples._  
第一个例子 [ParentFirstTestJCL0](https://articles.qos.ch/delegation/src/java/ch/qos/test/ParentFirstTestJCL0.java) 不会显式设置 TCCL。因此，TCCL 将默认使用系统类加载器。 _请注意，此示例**并非为了**演示 JCL 中的错误，而是为其余示例热身。_

```java
public class ParentFirstTestJCL0 {
  
  public static void main(String[] args) throws Exception {
  
    URLClassLoader childClassLoader =
      new URLClassLoader(
        new URL[] {
          new URL("file:box1.jar"), 
          new URL("file:lib/commons-logging.jar"),
          new URL("file:lib/log4j.jar")
        });

    Log log = LogFactory.getLog("logger.name.not.important.here");
    log.info("a message");

    Class boxClass = childClassLoader.loadClass("box.BoxImplWithJCL");
    Box box = (Box) boxClass.newInstance();
    System.out.println(box);
    box.doOp();

  }
}
  

```

The `ParentFirstTestJCL0` application creates a child class loader of type `URLClassLoader` and adds _box1.jar_, _commons-logging.jar_ and _log4j.jar_ to it. In the next two lines, a JCL `Log` instance is retrieved by `LogFactory` and used to log a message. The child class loader is then used to create an instance of [`box.BoxImplWithJCL`](https://articles.qos.ch/delegation/src/java/box/BoxImplWithJCL.java). We then call the `doOp` method of the newly created `BoxImplWithJCL` instance. The `doOp()` method in `BoxImplWithJCL` invokes the [`LogFactory.getLog`](http://commons.apache.org/logging/apidocs/org/apache/commons/logging/LogFactory.html#getLog(java.lang.String)) method to retrieve a [`Log`](http://commons.apache.org/logging/apidocs/org/apache/commons/logging/Log.html) instance and then uses it to log a simple message.  
`ParentFirstTestJCL0` 应用程序创建一个 `URLClassLoader` 类型的子类加载器，并添加 _box1.jar_ 、 _commons-logging.jar_ 和 _log4j.jar_ 到它。在接下来的两行中，一个 JCL `Log` 实例由 `LogFactory` 检索 并用于记录消息。然后使用子类加载器 创建 [`box.BoxImplWithJCL`](https://articles.qos.ch/delegation/src/java/box/BoxImplWithJCL.java) 的实例。然后我们调用新创建的实例的 `doOp` 方法 `BoxImplWithJCL` 实例。BoxImplWithJCL 中的 `doOp()` 方法调用 `BoxImplWithJCL` [`LogFactory.getLog`](http://commons.apache.org/logging/apidocs/org/apache/commons/logging/LogFactory.html#getLog(java.lang.String)) 来检索 [`Log`](http://commons.apache.org/logging/apidocs/org/apache/commons/logging/Log.html) 实例，然后使用它来记录一条简单消息。

The next diagram illustrates this configuration more graphically.  
下图更形象地说明了此配置。

[![](https://articles.qos.ch/images/cl-example-1.gif)](https://articles.qos.ch/images/cl-example-1.gif)

Running the command: 运行命令：

java -cp boxAPI.jar;classes;lib/commons-logging.jar ch.qos.test.ParentFirstTestJCL0  
java -cp boxAPI.jar;类;lib/commons-logging.jar ch.qos.test.ParentFirstTestJCL0

yields 产量

```
Feb 6, 2005 3:44:45 PM ch.qos.test.ParentFirstTestJCL0 main
INFO: a message
Feb 6, 2005 3:44:45 PM box.BoxImplWithJCL doOp
INFO: hello  

```

Wait a minute, that is output from java.util.logging, not log4j! How did that happen?  
等一下，这是 java.util.logging 的输出，而不是 log4j！这是怎么回事？

Although we could load `box.BoxImplWithJCL` using the child class loader (only it had access to _box1.jar_), JCL ignored _log4j.jar_ in the child class loader because the thread context class loader was not set. JCL then discovered `java.util.logging` which provides a default configuration directing messages of level INFO and above to the console.  
虽然我们可以使用子类加载器加载 `box.BoxImplWithJCL` （只有它可以访问 _box1.jar_ ），但 JCL 忽略了子类加载器中的 _log4j.jar_ ，因为 线程上下文类加载器未设置。JCL 随后发现 `java.util.logging` 提供了默认 配置将 INFO 级别及以上消息定向到 安慰。

In the next example, we will explicitly set the thread context class loader (TCCL) and subsequently run into a whole new set of problems.  
在下一个示例中，我们将明确设置线程上下文类加载器（TCCL），随后遇到一系列全新的问题。

### Example-2 示例 2

The [`ParentFirstTestJCL1`](https://articles.qos.ch/delegation/src/java/ch/qos/test/ParentFirstTestJCL1.java) application is similar to `ParentFirstTestJCL0` except that it explicitly sets the child class loader as the TCCL. It then loads a new instance of `BoxImplWithJCL` and attempts to invoke the `doOp` method on that instance.  
[`ParentFirstTestJCL1`](https://articles.qos.ch/delegation/src/java/ch/qos/test/ParentFirstTestJCL1.java) 该应用程序与 `ParentFirstTestJCL0` 类似，只是它明确将子类加载器设置为 TCCL。然后，它加载一个 `BoxImplWithJCL` 的新实例，并尝试在该实例上调用 `doOp` 方法。

```java
package ch.qos;

public class ParentFirstTestJCL1 {

  public static void main(String[] args) throws Exception {
  
    URLClassLoader childClassLoader = new URLClassLoader(new URL[] {
        new URL("file:box1.jar"),
        new URL("file:lib/commons-logging.jar"),
        new URL("file:lib/log4j.jar")});
  		
    Thread.currentThread().setContextClassLoader(childClassLoader);
  		
    Class boxClass = childClassLoader.loadClass("box.BoxImplWithJCL");
    Box box = (Box) boxClass.newInstance();
    box.doOp();
  }
}

```

The class loader configuration for Example-2 is illustrated in the figure below.  
示例 2 的类加载器配置如下图所示。

[![](https://articles.qos.ch/images/cl-example-2.gif)](https://articles.qos.ch/images/cl-example-2.gif)

Running the following command  
运行以下命令

java -cp boxAPI.jar;classes;lib/commons-logging.jar ch.qos.test.ParentFirstTestJCL1  
java -cp boxAPI.jar;类;lib/commons-logging.jar ch.qos.test.ParentFirstTestJCL1

yields: 产量：

```
Exception in thread "main" org.apache.commons.logging.LogConfigurationException: \
org.apache.commons.logging.LogConfigurationException: 
  No suitable Log constructor [Ljava.lang.Class;@12b66 for org.apache.commons.logging.impl.Log4JLogger 
Caused by java.lang.NoClassDefFoundError: org/apache/log4j/Category) 
Caused by org .apache.commons.logging.LogConfigurationException: 
   No suitable Log constructor [Ljava.lang.Class;@12b6651 for org.apache.commons.logging.impl.Log4JLogger 
(Caused by java.lang.NoClassDefFoundError: org/apache/log4j/Category))
        at org.apache.commons.logging.impl.LogFactoryImpl.newInstance(LogFactoryImpl.java:543)
        at org.apache.commons.logging.impl.LogFactoryImpl.getInstance(LogFactoryImpl.java:235)
        at org.apache.commons.logging.LogFactory.getLog(LogFactory.java:370)
        at box.BoxImplWithJCL.doOp(BoxImplWithJCL.java:23)
        at ch.qos.test.ParentFirstTestJCL1.main(ParentFirstTestJCL1.java:44)
Caused by: org.apache.commons.logging.LogConfigurationException: 
  No suitable Log constructor [Ljava.lang.Class;@12b6651 for org.apache.commons.logging.impl.Log4J
Logger (Caused by java.lang.NoClassDefFoundError: org/apache/log4j/Category)
        at org.apache.commons.logging.impl.LogFactoryImpl.getLogConstructor(LogFactoryImpl.java:413)
        at org.apache.commons.logging.impl.LogFactoryImpl.newInstance(LogFactoryImpl.java:529)
        ... 4 more
Caused by: java.lang.NoClassDefFoundError: org/apache/log4j/Category
        at java.lang.Class.getDeclaredConstructors0(Native Method)
        at java.lang.Class.privateGetDeclaredConstructors(Class.java:1590)
        at java.lang.Class.getConstructor0(Class.java:1762)
        at java.lang.Class.getConstructor(Class.java:1002)
        at org.apache.commons.logging.impl.LogFactoryImpl.getLogConstructor(LogFactoryImpl.java:410)
        ... 5 more


```

What went wrong?  哪里出了问题？

The `LogConfigurationException` is thrown when we attempt to log from the `doOp()` method in [`box.BoxImplWithJCL`](https://articles.qos.ch/delegation/src/java/box/BoxImplWithJCL.java). The `BoxImplWithJCL` class is loaded and instantiated by the child class loader. The child class loader delegates the loading of the `LogFactory` class to its parent. In order to return a Log instance, at some point LogFactory invokes its `getLogConstructor()` method. The `LogConfigurationException` is thrown from within this method.  
当我们尝试从 [`box.BoxImplWithJCL`](https://articles.qos.ch/delegation/src/java/box/BoxImplWithJCL.java) 中的 `doOp()` 方法进行日志记录时，会抛出 `LogConfigurationException` 。 `BoxImplWithJCL` 类由子类加载器加载并实例化。子类加载器将 `LogFactory` 类的加载委托给其父类加载器。为了返回 一个 Log 实例，在某个时刻 LogFactory 调用它的 `getLogConstructor()` 方法。 此方法内部抛出了 `LogConfigurationException` 。

On line 368 of [_LogFactoryImpl.java_](https://articles.qos.ch/delegation/jcl-src/impl/LogFactoryImpl.java), `getLogConstructor()` invokes `getLogClassName()`. Given that the classes `org.apache.log4j.Logger` and `org.apache.commons.logging.impl.Log4JLogger` are visible through the TCCL (defined to be the child class loader), the `getLogClassName()` method returns the string value "org.apache.commons.logging.impl.Log4JLogger". On lines 373 through 398, the `Log4jLogger` class is loaded into memory by the parent class loader. The TCCL, i.e. the child class loader in this example, delegates to its parent, the system class loader, which is able to find JCL's `Log4jLogger` class. This copy of `Log4jLogger` is compared against a copy of the `org.apache.commons.logging.Log` class which was also loaded by the same class loader. The comparison succeeds and we proceed to the next step.  
在 [_LogFactoryImpl.java_](https://articles.qos.ch/delegation/jcl-src/impl/LogFactoryImpl.java) 的第 368 行， `getLogConstructor()` 调用 `getLogClassName()` 。假设类 `org.apache.log4j.Logger` 和 `org.apache.commons.logging.impl.Log4JLogger` 可见 通过 TCCL（定义为子类加载器）， `getLogClassName()` 方法返回字符串值“org.apache.commons.logging.impl.Log4JLogger”。在第 373 行到第 398 行， `Log4jLogger` 类由父类加载器加载到内存中。TCCL（本例中为子类加载器）将加载委托给其父类加载器，后者能够找到 JCL 的 `Log4jLogger` 类。 `Log4jLogger` 与 `org.apache.commons.logging.Log` 类，也是 由同一个类加载器加载。比较成功，我们 继续下一步。

JCL loads logging API implementation classes using the TCCL but hands down the work of obtaining the constructor method to the current class loader. This discrepancy is at the heart of JCL's class loader problems of Type-I.  
JCL 使用 TCCL 加载日志记录 API 实现类，但将获取构造函数方法的工作交给了当前类加载器。这种差异是 JCL 类加载器 I 类问题的核心。

On line 410, the `Class.getConstructor()` method is invoked on the `Log4jLogger` class -- loaded by the parent class loader. This in turn triggers the loading of log4j's `Logger` class. This is done using the class loader that loaded the `Log4jLogger` class, i.e. the parent class loader, which cannot see `log4j.jar`, hence the root `java.lang.NoClassDefFoundError` exception about `org.apache.log4j.Category` being undefined. In other words, JCL loads logging API implementation classes using the TCCL but hands down the work of obtaining the constructor method for the implementation class to the current class loader. This discrepancy is at the heart of JCL's class loader problems of Type-I.  
在第 410 行，在 `Log4jLogger` 类上调用 `Class.getConstructor()` 方法——由 父类加载器。这反过来又触发了 log4j 的加载 `Logger` 类。这是使用加载 `Log4jLogger` 类的类加载器（即父类加载器）完成的，父类加载器无法看到 `log4j.jar` ，因此根 `java.lang.NoClassDefFoundError` 例外情况 `org.apache.log4j.Category` 未定义。在其他 换句话说，JCL 使用 TCCL 加载日志 API 实现类 但毫无疑问，获得构造函数方法的工作 实现类到当前类加载器。这种差异 是 JCL 的 I 类加载器问题的核心。

### Example-2B: Excluding `Log4jLogger` from _commons-logging.jar_  
示例 2B：从中排除 `Log4jLogger` _commons-logging.jar_

The file _commons-logging-api.jar_ which ships with the JCL distribution is identical to _commons-logging.jar_ except that the former does not include the `org.apache.commons.logging.impl.Log4JLogger` and `org.apache.commons.logging.impl.AvalonLogger` classes.  
JCL 发行版附带的 _commons-logging-api.jar_ 文件与 _commons-logging.jar_ 相同，除了 前者不包括 `org.apache.commons.logging.impl.Log4JLogger` 和 `org.apache.commons.logging.impl.AvalonLogger` 个类。

Interestingly enough running `ParentFirstTestJCL1` with _commons-logging-api.jar_ on the class path gives better results. The class loader configuration for Example-2B is illustrated in the figure below.  
有趣的是，运行 `ParentFirstTestJCL1` 在类路径上添加 _commons-logging-api.jar_ 可以获得更好的效果。示例 2B 的类加载器配置如下图所示。

[![](https://articles.qos.ch/images/cl-example-2B.gif)](https://articles.qos.ch/images/cl-example-2B.gif)

Running `ParentFirstTestJCL1` with _commons-logging-api.jar_ on the class path as:  
使用以下方式运行 `ParentFirstTestJCL1` _commons-logging-api.jar_ 在类路径上为：

java -cp boxAPI.jar;classes;lib/commons-logging-api.jar ch.qos.test.ParentFirstTestJCL1  
java -cp boxAPI.jar;类;lib/commons-logging-api.jar ch.qos.test.ParentFirstTestJCL1

will load log4j classes without errors. Indeed, the `Log4JLogger` class will be loaded by the child class loader instead of the parent which cannot see `Log4JLogger`. The child class loader will also be able to load log4j classes.  
将加载 log4j 类而不会出现错误。事实上， `Log4JLogger` 类将由子类加载 加载器而不是无法看到的父级 `Log4JLogger` 。子类加载器也将能够加载 log4j 类。

Unfortunately, this approach has the obvious inconvenience of preventing the use of of log4j (or any other API residing outside the JDK) by classes loaded by the parent class loader, even if log4j classes are visible to the parent class loader.  
不幸的是，这种方法明显不方便，因为它会阻止父类加载器加载的类使用 log4j（或 JDK 之外的任何其他 API），即使 log4j 类对于父类加载器来说是可见的。

Placing _commons-logging-api.jar_ in the parent class loader is safe but precludes the use of log4j by classes directly loaded by the parent class loader. We will come back to this point when discussing child-first class loader trees. As shown in Example-2, placing _commons-logging.jar_ in the parent class loader is unsafe when log4j classes are visible to a child class loader, also defined to be the TCCL.  
将 _commons-logging-api.jar_ 放在父类加载器中是安全的，但这会阻止由父类加载器直接加载的类使用 log4j。在讨论子类优先加载器树时，我们会再次讨论这一点。如示例 2 所示，将 _commons-logging.jar_ 放在父类中 当 log4j 类对子类可见时，加载器是不安全的 加载器，也定义为 TCCL。

In simple English, JCL's discovery mechanism can safely bridge only for `java.util.logging`, other logging APIs residing outside the JDK cannot be discovered in a safe manner in most configurations.  
简单来说，JCL 的发现机制只能安全地桥接 `java.util.logging` ，在大多数配置中无法以安全的方式发现 JDK 之外的其他日志记录 API。

### Example-3 示例 3

The [`ParentFirstTestJCL2`](https://articles.qos.ch/delegation/src/java/ch/qos/test/ParentFirstTestJCL2.java) application is very similar to `ParentFirstTestJCL1`. It illustrates that logging through JCL will fail as soon as the thread context class loader is set to the child class loader.  
[`ParentFirstTestJCL2`](https://articles.qos.ch/delegation/src/java/ch/qos/test/ParentFirstTestJCL2.java) 应用程序与 `ParentFirstTestJCL1` 非常相似。它 说明一旦线程 上下文类加载器设置为子类加载器。

```
NoClassDefFoundError

```

Running 跑步

java -cp boxAPI.jar;classes;lib/commons-logging.jar ch.qos.test.ParentFirstTestJCL2  
java -cp boxAPI.jar;类;lib/commons-logging.jar ch.qos.test.ParentFirstTestJCL2

will yield the same error message as in the previous example.  
将产生与前面的示例相同的错误消息。

## JCL in child-first class loader trees  
子类优先加载器树中的 JCL

In child-first class loader trees, JCL suffers from problems of both Type-I and Type-II. We will start this section with an example reproducing a Type-II problem.  
在子类优先的类加载器树中，JCL 会同时遭遇 I 类和 II 类问题。本节我们将以一个重现 II 类问题的示例开始。

### Example-4 示例 4

The next example, [ChildFirstTestJCL0](https://articles.qos.ch/delegation/src/java/ch/qos/test/ChildFirstTestJCL0.java), demonstrates that when both a parent and child class loader have direct access to _commons-logging.jar_ and the TCCL is set to point to the child class loader, the first call to JCL's `LogFactory.getLog()` method from a class directly loaded by the parent class loader will throw an exception.  
下一个示例 [ChildFirstTestJCL0](https://articles.qos.ch/delegation/src/java/ch/qos/test/ChildFirstTestJCL0.java) 演示了当父类加载器和子类加载器都可以直接访问 _commons-logging.jar_ 并且 TCCL 设置为 指向子类加载器，第一次调用 JCL 的 直接加载的类中的 `LogFactory.getLog()` 方法 由父类加载器加载时会抛出异常。

```java
package ch.qos.test;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

import ch.qos.ChildFirstClassLoader;

public class ChildFirstTestJCL0 {
  public static void main(String[] args) throws Exception {
    ChildFirstClassLoader child = new ChildFirstClassLoader( 
        new URL[] { new URL("file:lib/commons-logging.jar") });

    Thread.currentThread().setContextClassLoader(child);

    // JCL will throw an exception as soon as LogFactory is called after the 
    // TCCL is set to a child class loader, and both the parent and child
    // have direct access to commons-logging.jar
    Log log = LogFactory.getLog("logger.name.not.important.here");
  }
} 

```

The class loader configuration for Example-4 is illustrated in the figure below.  
示例 4 的类加载器配置如下图所示。

[![](https://articles.qos.ch/images/cl-example-4.gif)](https://articles.qos.ch/images/cl-example-4.gif)

Running application [ChildFirstTestJCL0](https://articles.qos.ch/delegation/src/java/ch/qos/test/ChildFirstTestJCL0.java) with the following command line  
运行应用程序 [ChildFirstTestJCL0](https://articles.qos.ch/delegation/src/java/ch/qos/test/ChildFirstTestJCL0.java) 使用以下命令行

java -cp classes;lib/commons-logging.jar ch.qos.test.ChildFirstTestJCL0  
java -cp 类；lib/commons-logging.jar ch.qos.test.ChildFirstTestJCL0

will result in the following exception  
将导致以下异常

```
Exception in thread "main" org.apache.commons.logging.LogConfigurationException: \
  org.apache.commons.logging.LogConfigurationException: \
    org.apache.commons.logging.LogConfigurationException: Invalid class loader hierarchy.  \
  You have more than one version of 'org.apache.commons.logging.Log' visible, which is not allowed. 
(Caused by org.apache.commons.logging.LogConfigurationException: Invalid class loader hierarchy.  
 You have more than one version of 'org.apache.commons.logging.Log' visible, which is not allowed.) 
(Caused by org.apache.commons.logging.LogConfigurationException: \
org.apache.commons.logging.LogConfigurationException: Invalidclass loader hierarchy.  \
You have more than one version of 'org.apache.commons.logging.Log' visible, which is not allowed. 
(Caused by org.apache.commons.logging.LogConfigurationException: Invalid class loader hierarchy.  \
You have more than one version of 'org.apache.commons.logging.Log' visible, which is not allowed.))
        at org.apache.commons.logging.impl.LogFactoryImpl.newInstance(LogFactoryImpl.java:543)
        at org.apache.commons.logging.impl.LogFactoryImpl.getInstance(LogFactoryImpl.java:235)
        at org.apache.commons.logging.LogFactory.getLog(LogFactory.java:370)
        at ch.qos.test.ChildFirstTestJCL0.main(ChildFirstTestJCL0.java:43)
Caused by: org.apache.commons.logging.LogConfigurationException: \
org.apache.commons.logging.LogConfigurationException: Invalid class loader hierarchy.  \
You have more than one version of 'org.apache.commons.logging.Log' visible, which is not allowed. 
(Caused by org.apache.commons.logging.LogConfigurationException: Invalid class loader hierarchy. \
You have more than one version of 'org.apache.commons.logging.Log' visible, which is not allowed.)
        at org.apache.commons.logging.impl.LogFactoryImpl.getLogConstructor(LogFactoryImpl.java:397)
        at org.apache.commons.logging.impl.LogFactoryImpl.newInstance(LogFactoryImpl.java:529)
        ... 3 more
Caused by: org.apache.commons.logging.LogConfigurationException: Invalid class loader hierarchy.  \
You have more than one version of 'org.apache.commons.logging.Log' visible, which is not allowed.
        at org.apache.commons.logging.impl.LogFactoryImpl.getLogConstructor(LogFactoryImpl.java:385)
        ... 4 more
  

```

The exception is thrown because the `Log` implementation loaded by the TCCL, i.e. the child class loader, is not the same as the `Log` interface in use within `LogFactory` (loaded by the parent class loader). According to [Section 4.3.2](http://java.sun.com/docs/books/jls/second_edition/html/typesValues.doc.html#97058) of the Java Language Specification, two classes loaded by different class loaders are considered to be distinct and hence incompatible.  
抛出异常是因为 `Log` 由 TCCL 加载的实现，即子类加载器，是 与使用的 `Log` 接口不同 `LogFactory` （由父类加载器加载）。根据 Java 语言规范[第 4.3.2 节](http://java.sun.com/docs/books/jls/second_edition/html/typesValues.doc.html#97058) ，由父类加载器加载的两个类 不同的类加载器被认为是不同的，因此 不相容。

### Example-4B 示例 4B

In the previous example, `LogFactory` and `Log` classes were loaded by the parent loader while its implementation was loaded by the TCCL. Example-4B demonstrates that if the TCCL follows child-first delegation, then in case the `LogFactory` and `Log` classes are loaded by the TCCL, the `Log` implementation will be compatible since it will be loaded by the same class loader.  
在前面的例子中， `LogFactory` 和 `Log` 类由父加载器加载，而其 实现已由 TCCL 加载。示例 4B 演示了 如果 TCCL 遵循儿童优先授权，那么如果 `LogFactory` 和 `Log` 类由 TCCL 加载， `Log` 实现将兼容，因为它将由同一个类加载器加载。

```java
package ch.qos.test;

import box.Box;
import ch.qos.ChildFirstClassLoader;

import java.net.URL;

/**
 * Usage:
 *
 *   java -cp classes;lib/commons-logging.jar ch.qos.test.ChildFirstTestJCL1
 *
 * @author Ceki Gülcü
 */
public class ChildFirstTestJCL1 {
  public static void main(String[] args) throws Exception {
    ChildFirstClassLoader child = new ChildFirstClassLoader( 
        new URL[] { 
            new URL("file:lib/commons-logging.jar"),
            new URL("file:box1.jar")});

    Thread.currentThread().setContextClassLoader(child);

    Class boxClass = child.loadClass("box.BoxImplWithJCL");
    Box box = (Box) boxClass.newInstance();
    box.doOp();
   }
}
  

```

Running 跑步

java -cp boxAPI.jar;classes;lib/commons-logging.jar ch.qos.test.ChildFirstTestJCL1  
java -cp boxAPI.jar;类;lib/commons-logging.jar ch.qos.test.ChildFirstTestJCL1

will output a log message using java.util.logging.  
将使用 java.util.logging 输出日志消息。

### Example-5 示例 5

Previously in the fourth example, we have seen that having a copy of _commons-logging.jar_ in both the parent and child class loaders will cause problems as soon as a class loaded by the parent attempts to use JCL. In this fifth example, called [ChildFirstTestJCL2](https://articles.qos.ch/delegation/src/java/ch/qos/test/ChildFirstTestJCL2.java), we will remove _commons-logging.jar_ from the child class loader and hand it a copy of _log4j.jar_ instead. It demonstrates that when the parent class loader can see _commons-logging.jar_ and the child class loader (also set to be the TCCL) can see _log4j.jar_, then as in the second and third examples, JCL will throw a `LogConfigurationException` because it will not be able to find the log4j classes.  
在之前的第四个示例中，我们已经看到，如果在父类加载器和子类加载器中都保留 _commons-logging.jar_ 的副本，那么当父类加载器加载的类尝试使用 JCL 时，就会出现问题。在这个名为 [ChildFirstTestJCL2](https://articles.qos.ch/delegation/src/java/ch/qos/test/ChildFirstTestJCL2.java) 的第五个示例中，我们将从子类加载器中删除 _commons-logging.jar_ ，并改为使用 _log4j.jar_ 的副本。它 演示了当父类加载器可以看到 _commons-logging.jar_ 和子类加载器（也设置为 TCCL）可以看到 _log4j.jar_ ，然后像第二个和 第三个例子，JCL 将抛出一个 `LogConfigurationException` ，因为它无法 找到 log4j 类。

```java
package ch.qos.test;

import java.net.URL;
import box.Box;

import ch.qos.ChildFirstClassLoader;

/**
 * Usage:
 *
 *   java -cp classes;boxAPI.jar;lib/commons-logging.jar ch.qos.test.ChildFirstTestJCL1
 * 
 * @author Ceki Gülcü
 */
public class ChildFirstTestJCL2 {

  public static void main(String[] args) throws Exception {
    
    ChildFirstClassLoader child = new ChildFirstClassLoader(new URL[] {
        new URL("file:box1.jar"), 
        new URL("file:lib/log4j.jar") });

    Thread.currentThread().setContextClassLoader(child);
    
    Class boxClass = child.loadClass("box.BoxImplWithJCL");
    Box box = (Box) boxClass.newInstance();
    box.doOp();
  }
}

```

The class loader configuration for Example-5 is illustrated in the figure below.  
示例 5 的类加载器配置如下图所示。

[![](https://articles.qos.ch/images/cl-example-5.gif)](https://articles.qos.ch/images/cl-example-5.gif)

Running the command: 运行命令：

java -cp classes;boxAPI.jar;lib/commons-logging.jar ch.qos.test.ChildFirstTestJCL2  
java -cp 类；boxAPI.jar；lib/commons-logging.jar ch.qos.test.ChildFirstTestJCL2

will cause the following exception to be thrown.  
将导致抛出以下异常。

```
Exception in thread "main" org.apache.commons.logging.LogConfigurationException: \
org.apache.commons.logging.LogConfigurationException: \ 
No suitable Log constructor [Ljava.lang.Class;@4a5ab2 for org.apache.commons.logging.impl.Log4JLogger\
(Caused by java.lang.NoClassDefFoundError: org/apache/log4j/Category) 
(Caused by org.apache.commons.logging.LogConfigurationException: \
No suitable Log constructor [Ljava.lang.Class;@4a5ab2 for org.apache.commons.logging.impl.Log4JLogger 
(Caused by java.lang.NoClassDefFoundError: org/apache/log4j/Category))
        at org.apache.commons.logging.impl.LogFactoryImpl.newInstance(LogFactoryImpl.java:543)
        at org.apache.commons.logging.impl.LogFactoryImpl.getInstance(LogFactoryImpl.java:235)
        at org.apache.commons.logging.LogFactory.getLog(LogFactory.java:370)
        at box.BoxImplWithJCL.doOp(BoxImplWithJCL.java:23)
        at ch.qos.test.ChildFirstTestJCL2.main(ChildFirstTestJCL1.java:42)
Caused by: org.apache.commons.logging.LogConfigurationException: \ 
 No suitable Log constructor [Ljava.lang.Class;@4a5ab2 for org.apache.commons.logging.impl.Log4JLogger
(Caused by java.lang.NoClassDefFoundError: org/apache/log4j/Category)
        at org.apache.commons.logging.impl.LogFactoryImpl.getLogConstructor(LogFactoryImpl.java:413)
        at org.apache.commons.logging.impl.LogFactoryImpl.newInstance(LogFactoryImpl.java:529)
        ... 4 more
Caused by: java.lang.NoClassDefFoundError: org/apache/log4j/Category
        at java.lang.Class.getDeclaredConstructors0(Native Method)
        at java.lang.Class.privateGetDeclaredConstructors(Class.java:1590)
        at java.lang.Class.getConstructor0(Class.java:1762)
        at java.lang.Class.getConstructor(Class.java:1002)
        at org.apache.commons.logging.impl.LogFactoryImpl.getLogConstructor(LogFactoryImpl.java:410)
        ... 5 more
  

```

The causes for this exception are very similar to those discussed in the second example.  
导致此异常的原因与第二个示例中讨论的原因非常相似。

### Example-6 示例 6

The [ChildFirstTestJCL3](https://articles.qos.ch/delegation/src/java/ch/qos/test/ChildFirstTestJCL3.java) application is very similar to the previous example. It illustrates how logging by any class loaded by the parent class loader using JCL will fail as soon as the thread context class loader is set to the child class loader.  
[ChildFirstTestJCL3](https://articles.qos.ch/delegation/src/java/ch/qos/test/ChildFirstTestJCL3.java) 应用程序与前面的示例非常相似。它说明了 如何使用 JCL 对父类加载器加载的任何类进行日志记录 一旦线程上下文类加载器设置为 子类加载器。

### Example-7 示例 7

Suppose you are using Tomcat and wanted to use log4j in your web-application. In Tomcat, each web-application has its own class loader following the child-first delegation model. Suppose you add _log4j.jar_ in your web-application's war file and add _commons-logging.api_ in Tomcat's _common/lib/_ directory.. Well, in that case, you'd be reenacting the set up described in Example-5 and Example-6 and you'd be getting the same exceptions.  
假设您正在使用 Tomcat 并希望在您的 web 应用程序。在 Tomcat 中，每个 web 应用程序都有自己的类 加载器遵循 child-first 委托模型。假设你添加 在您的 Web 应用程序的 war 文件中_引入 log4j.jar_ 并添加 Tomcat 的 _common/lib/ 中_的 _commons-logging.api_ 目录。好吧，在这种情况下，你会重新设置 示例 5 和示例 6 中描述的方法，你会得到相同的结果 例外。

For the sceptical about the fidelity of this simulation, the [distribution](https://articles.qos.ch/delegation/cl-problems.zip) accompanying this document contains a small but complete web-application called "Hello" which can reproduce the problem in Tomcat.  
对于怀疑该模拟保真度的人来说，伴随该模拟的[分布](https://articles.qos.ch/delegation/cl-problems.zip) 文档包含一个小而完整的网络应用程序，名为 “Hello”，可以在 Tomcat 中重现该问题。

## Holding references to class loaders  
保存对类加载器的引用

In order to track the various logging implementations and their wrappers, JCL internally manages a hash table keyed by class loader. This causes problems of Type-III. Indeed, by holding strong references to class loaders, JCL prevents application containers from garbage collecting recycled web-applications. This limitation will be addressed in JCL version 1.0.5 by replacing the regular hash table with a [WeakHashTable](http://commons.apache.org/commons/logging/optional/apidocs/org/apache/commons/logging/impl/WeakHashtable.html).  
为了跟踪各种日志记录实现及其包装器，JCL 内部管理一个以类加载器为键的哈希表。这会导致 III 类问题。事实上，通过持有对类加载器的强引用，JCL 会阻止应用程序容器对已回收的 Web 应用程序进行垃圾回收。JCL 1.0.5 版本将通过使用 [Wea​​kHashTable](http://commons.apache.org/commons/logging/optional/apidocs/org/apache/commons/logging/impl/WeakHashtable.html) 替换常规哈希表来解决此限制。

## Compared to static binding, e.g. [SLF4J](http://www.slf4j.org/)  
与静态绑定相比，例如 [SLF4J](http://www.slf4j.org/)

Until recently JCL was the only solution that provided an abstraction of various logging APIs. With the benefit of hindsight, the log4j project has developed an alternative solution called "Simple Logging Facade for Java" or SLF4J in short.  
直到最近，JCL 才成为唯一提供各种日志 API 抽象的解决方案。事后看来，log4j 项目开发了一种替代解决方案，称为“Java 简单日志外观”（Simple Logging Facade for Java），简称 SLF4J。

SLF4J supports multiple implementations, namely, NOP, Simple, log4j, JDK 1.4 logging and logback. SLF4J distributions ship with four jar files _slf4j-nop.jar_, _slf4j-simple.jar_, _slf4j-log4j12.jar_and _slf4j-jdk14.jar_. Each of these jar files is hardwired _at compile time_ to use just one implementation, that is NOP, Simple, log4j12 and JDK 1.4 logging, respectively. [Logback](http://logback.qos.ch/) distributions ship with _logback-classic.jar_ which implements the SLF4J API natively.  
SLF4J 支持多种实现，包括 NOP、Simple、log4j、JDK 1.4 logging 和 logback。SLF4J 发行版附带四个 jar 文件 _：slf4j-nop.jar_ 、 _slf4j-simple.jar_ 、 _slf4j-log4j12.jar_ 和 _slf4j-jdk14.jar_ 。这些 jar 文件_在编译时_都硬编码为只使用一种实现，分别是 NOP、Simple、log4j12 和 JDK 1.4 日志[记录](http://logback.qos.ch/) 。Logback 发行版附带 _logback-classic.jar_ ， 本机实现 SLF4J API。

Unless the _slf4j-xxx.jar_ or _logback-classic.jar_ file is corrupt, static linking can never cause bugs of Type-I, Type-II or Type-III for the simple reason that all API bindings are done at compile time with all the required files bundled in the _slf4j-xxx.jar_ or _logback-classic.jar_ file.  
除非 _slf4j-xxx.jar_ 或 _logback-classic.jar_ 文件损坏，静态链接永远不会导致 I 类错误， 类型 II 或类型 III，原因很简单，所有 API 绑定都是 在编译时完成，所有必需的文件都捆绑在 _slf4j-xxx.jar_ 或 _logback-classic.jar_ 文件。

You can verify this for yourself with the help of [`BoxImplWithSLF4J`](https://articles.qos.ch/delegation/src/java/box/BoxImplWithSLF4J.java) and related test cases found in the distribution. In a nutshell, SLF4J will never behave unexpectedly or crash your application.  
您可以借助 [`BoxImplWithSLF4J`](https://articles.qos.ch/delegation/src/java/box/BoxImplWithSLF4J.java) 和相关测试用例来验证这一点 分布。简而言之，SLF4J 永远不会出现意外行为或 导致应用程序崩溃。

Given the severity of the problems with JCL, we recommend that you consider switching to [SLF4J](http://www.slf4j.org/) which provides better functionality with none of the bugs. In the short term however, you can substantially alleviate some of the pain of using JCL in conjunction with log4j, if you _exactly_ adhere to the following scenario.  
鉴于 JCL 问题的严重性，我们建议您考虑切换到 [SLF4J](http://www.slf4j.org/) ，它提供了更好的功能，并且没有任何 bug。不过，如果您_严格_遵循以下方案，短期内可以显著减轻 JCL 与 log4j 结合使用带来的一些痛苦。

## In all servers, 在所有服务器中，

**Make sure that you are using JCL version 1.0.4 or later.** Earlier versions of JCL pose problems beyond those discussed in this document.  
**确保您使用的是 JCL 版本 1.0.4 或更高版本。** 早期版本的 JCL 存在的问题超出了 本文件。

### For Tomcat 5.0.27 and later,  
对于 Tomcat 5.0.27 及更高版本，

1.  Make sure you are using JCL version 1.0.4 or later. Tomcat version 5.0.x ships with an outdated version of JCL.  
    确保您使用的是 JCL 1.0.4 或更高版本。Tomcat 5.0.x 版本附带的 JCL 版本已过时。
2.  Leave the file _TOMCAT_HOME/bin/commons-logging-api.jar_ as is, i.e. do not remove it.  
    留下文件 _TOMCAT_HOME/bin/commons-logging-api.jar_ 保持原样，即执行 不删除它。
3.  Place the files _commons-logging.jar_ and _log4j.jar_ in the directory _TOMCAT_HOME/common/lib/_.  
    将文件 _commons-logging.jar_ 和 目录中的 _log4j.jar_ _TOMCAT_HOME/common/lib/_ 。
4.  Do **not** include any other copies of _commons-logging.jar_ and _log4j.jar_ in your web-applications' _WEB-INF/lib/_ directory.  
    **不**包括任何其他副本 _commons-logging.jar_ 和 _log4j.jar_ 位于您的 web 应用程序的 _WEB-INF/lib/_ 目录中。
5.  Do **not** set the system properties `org.apache.commons.logging.LogFactory` or `org.apache.commons.logging.Log`.  
    **不要**设置系统属性 `org.apache.commons.logging.LogFactory` 或 `org.apache.commons.logging.Log` .

### For Resin 2.0.x 对于 Resin 2.0.x

1.  Make sure you are using JCL version 1.0.4 or later.  
    确保您使用的是 JCL 版本 1.0.4 或更高版本。
2.  Place the files _commons-logging.jar_ and _log4j.jar_ in the directory _RESIN_HOME/lib/_.  
    将文件 _commons-logging.jar_ 和 目录中的 _log4j.jar_ _RESIN_HOME/lib/_ 。
3.  Do **not** include any other copies of _commons-logging.jar_ and _log4j.jar_ in your web-applications' _WEB-INF/lib/_ directory.  
    **不**包括任何其他副本 _commons-logging.jar_ 和 _log4j.jar_ 位于您的 web 应用程序的 _WEB-INF/lib/_ 目录中。
4.  Do **not** set the system properties `org.apache.commons.logging.LogFactory` or `org.apache.commons.logging.Log`.  
    **不要**设置系统属性 `org.apache.commons.logging.LogFactory` 或 `org.apache.commons.logging.Log` .

### Jetty 码头

I am happy to report that future versions of Jetty are expected to drop JCL in favor of SLF4J. However, for older versions of Jetty, the following instructions should get you going.  
我很高兴地告诉大家，Jetty 的未来版本预计将放弃 JCL，转而使用 SLF4J。不过，对于旧版本的 Jetty，以下说明应该可以帮助你入门。

1.  Make sure you are using JCL version 1.0.4 or later.  
    确保您使用的是 JCL 版本 1.0.4 或更高版本。
2.  Place the files _commons-logging.jar_ and _log4j.jar_ in the directory _JETTY_HOME/ext/_.  
    将文件 _commons-logging.jar_ 和 目录中的 _log4j.jar_ _JETTY_HOME/ext/_ 。
3.  Do **not** include any other copies of _commons-logging.jar_ and _log4j.jar_ in your web-applications' _WEB-INF/lib/_ directory.  
    **不**包括任何其他副本 _commons-logging.jar_ 和 _log4j.jar_ 位于您的 web 应用程序的 _WEB-INF/lib/_ 目录中。
4.  Set the system property `org.apache.commons.logging.LogFactory` to `org.apache.commons.logging.impl.LogFactoryImpl`. This can be accomplished by starting Jetty as follows on the command line:  
    设置系统属性 `org.apache.commons.logging.LogFactory` 至 `org.apache.commons.logging.impl.LogFactoryImpl` 。这 可以通过在命令行中按如下方式启动 Jetty 来完成：
    
    ```
    java -Dorg.apache.commons.logging.LogFactory=org.apache.commons.logging.impl.LogFactoryImpl \ 
      -jar start.jar
    
    ```
    
5.  Do **not** set the `org.apache.commons.logging.Log` system property.  
    **不要**设置 `org.apache.commons.logging.Log` 系统属性。

As demonstrated above, JCL's discovery mechanism invents new and original ways of shooting yourself in the foot. For example, with JCL you can shoot yourself in the foot while aiming at the sky. Thanks to JCL you can be hit by lightning in the middle of the desert when it's not raining. If your computing life is too dull and trouble is what you are looking for, then JCL is the way to go.  
如上所述，JCL 的发现机制发明了一些新颖且原创的搬起石头砸自己脚的方法。例如，使用 JCL，你可以一边瞄准天空，一边搬起石头砸自己的脚。多亏了 JCL，即使没有下雨，你也可能在沙漠中被闪电击中。如果你觉得计算生活太枯燥，而你正在寻找麻烦，那么 JCL 就是你的选择。

In summary, statically bound discovery provides better functionality, with none of the painful bugs associated with JCL's dynamic discovery. In order for JCL to _reliably_ discover a logging API, this API must be located at the same level or higher in the class loader tree. Thus, in practical terms, the only logging API _safely_ bridged by JCL is the `java.util.logging` API. When compared with a statically bound bridging mechanism, for example as implemented by [SLF4J](http://www.slf4j.org/), JCL's dynamic discovery mechanism has strictly zero added value. In summary, statically bound discovery provides better functionality, with none of the painful bugs associated with JCL's dynamic discovery.  
总而言之，静态绑定发现提供了更好的功能，并且没有 JCL 动态发现中那些令人头疼的 bug。 为了使 JCL 能够_可靠地_发现日志 API，该 API 必须位于类加载器树中的同一级别或更高级别。因此，实际上，JCL 唯一_安全_桥接的日志 API 是 `java.util.logging` API。与静态绑定桥接机制（例如由 [SLF4J](http://www.slf4j.org/) 实现的机制）相比，JCL 的动态 发现机制完全没有附加值。总而言之， 静态绑定发现提供了更好的功能， 不存在与 JCL 动态相关的痛苦错误 发现。

### Decrypting the broader phenomenon  
解密更广泛的现象

As many Apache components rely on JCL, many developers inside and outside Apache are fooled into thinking that using JCL is safe and innocuous. Such developers assume that since a large number of projects already use JCL, it cannot suffer from any serious bugs because such bugs would have been previously reported and presumably corrected. As Eric S. Raymond [put it](http://www.catb.org/~esr/writings/cathedral-bazaar/cathedral-bazaar/ar01s04.html), "given enough eyeballs, all bugs are shallow."  
由于许多 Apache 组件依赖于 JCL，Apache 内部和外部的许多开发人员都被误导，认为使用 JCL 是安全无害的。这些开发人员认为，既然大量项目已经在使用 JCL，它就不会存在任何严重的 bug，因为这些 bug 之前应该已经报告过，而且应该已经修复了。正如 Eric S. Raymond [所说](http://www.catb.org/~esr/writings/cathedral-bazaar/cathedral-bazaar/ar01s04.html) ：“只要足够多的关注，所有 bug 都是浅显的。”

There is no doubt that as the number of eyeballs increases, the probability of catching bugs increases as well. However, we implicitly equate _eyeballs_ with _users_ of software. Unfortunately, the vast majority of users of any given software do not look at the software they are using, at least not in a serious and meaningful way. In reality the improvement in catching bugs is only logarithmic with the number of users.  
毫无疑问，随着眼球数量的增加，发现漏洞的概率也会增加。然而，我们隐含地将_眼球_等同_于_ 软件。不幸的是，任何给定软件的绝大多数用户 软件不要看他们正在使用的软件，至少不要 以一种严肃而有意义的方式。实际上，提高捕捉 错误数量仅与用户数量呈对数关系。

The fact that many people suffer the effects of a bug does not mean they will report it, let alone research it.  
事实上，许多人遭受漏洞的影响并不意味着他们会报告它，更不用说研究它了。

The nature of the bug also plays a crucial role in its identification. Humans are good at hunting down bugs that they can see. However, humans are not mentally equipped to identify class loader bugs which depend on factors external to the code being examined. Even an experienced and competent developer can stare at a class loader bug for hours and still not realize that it is there. It takes the precision of a JVM and a lot of patience to hunt down class loader bugs which are likely to remain elusive for the foreseeable future.  
错误的性质在识别过程中也起着至关重要的作用。人类擅长追踪他们能看见的错误。然而，人类的思维能力不足以识别依赖于被检查代码外部因素的类加载器错误。即使是经验丰富且能力出众的开发人员，也可能盯着一个类加载器错误几个小时，却仍然没有意识到它的存在。追踪类加载器错误需要 JVM 的精确度和极大的耐心，而这些错误在可预见的未来很可能难以发现。