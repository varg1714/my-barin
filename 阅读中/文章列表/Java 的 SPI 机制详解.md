---
source: https://mp.weixin.qq.com/s/jDwv_UQsaUSOlEDffXn3lg
create: 2025-04-17 21:40
read: false
---
# **1.SPI 简介**

SPI（Service Provicer Interface）是 Java 语言提供的一种接口发现机制，用来实现接口和接口实现的解耦。简单来说，就是系统只需要定义接口规范以及可以发现接口实现的机制，而不需要实现接口。

SPI 机制在 Java 中应用广泛。例如：JDBC 中的数据库连接驱动使用 SPI 机制，只定义了数据库连接接口的规范，而具体实现由各大数据库厂商实现，不同数据库的实现不同，我们常用的 mysql 的驱动也实现了其接口规范，通过这种方式，JDBC 数据库连接可以适配不同的数据库。

SPI 机制在各种框架中也有应用，例如：springboot 的自动装配中查找 spring.factories 文件的步骤就是应用了 SPI 机制；dubbo 也对 Java 的 SPI 机制进行扩展，实现了自己的 SPI 机制。

# **2.SPI 入门案例**

## 2.1. 创建工程

我们刚才在介绍中说过了，SPI 机制需要定义接口规范，这里我们以一个简单的接口案例来说明。

首先我们需要创建四个工程：

•spi-interface，这里定义 SPI 的接口类：Person

•spi-impl1，这里定义接口的第一个实现类：Teacher

•spi-impl2，这里定义接口的第二个实现类：Student

•spi-test，这里通过 SPI 机制加载所有实现类进行测试

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXEhqHAb2ZmfT0pDBcTAjVdBLLzicgwia6yvJCOw7McIUGxXqlVd6Yiap22FWLIoS3S4QzRMficywsUvg/640?wx_fmt=png&from=appmsg&wxfrom=13&tp=wxpic)

 2.2. 创建 SPI 接口规范

接口如下所示：

```
package com.jd.spi;
public interface Person {
     String favorite();
}
```

```
package com.jd.spi;

public class Teacher implements Person {

    public String favorite() {
        return "老师喜欢给学生上课";
    }
}
```

## 2.3. 创建实现类 1 项目

### 2.3.1. 创建接口

接口如下所示：

```
package com.jd.spi;

public class Student implements Person {
    public String favorite() {
        return "学生喜欢努力学习";
    }
}
```

### 2.3.2. 创建 spi 配置文件

如下图所示，在项目的 resources 文件夹下创建两个文件夹 META-INF/services，然后在文件夹下面创建名称为 com.jd.spi.Person 的文件，其文件的内容为当前项目的接口实现类 com.jd.spi.Teacher。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXEhqHAb2ZmfT0pDBcTAjVdVYtox52JZeIceJrgWQHiarjQgOBF3ADjELLpIPQgOYYhEuaDRgCDkHA/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

 2.4. 创建实现类 2 项目

### 2.4.1. 创建实现类 2

接口如下所示：

```
<dependencies>
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>spi-interface</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>spi-impl1</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>spi-impl2</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
```

```
package com.jd.spi;

import java.util.Iterator;
import java.util.ServiceLoader;

public class SPITest {

    public static void main(String[] args) {
        ServiceLoader<Person> loader = ServiceLoader.load(Person.class);
        for(Iterator<Person> it = loader.iterator(); it.hasNext();){
            Person person = it.next();
            System.out.println(person.favorite());;
        }
    }
}
```

### 2.4.2. 创建 spi 配置文件

如下图所示，在项目的 resources 文件夹下创建两个文件夹 META-INF/services，然后在文件夹下面创建名称为 com.jd.spi.Person 的文件，其文件的内容为当前项目的接口实现类 com.jd.spi.Student。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXEhqHAb2ZmfT0pDBcTAjVdxibJfYiapf847BxE443AsplPicgZMicp5tgW2Hos4LCeQWibB9XDeCiabQXQ/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

 2.5. 创建测试项目

### 2.5.1. 引入 3 个 maven 依赖

这里需要引入接口定义项目和两个接口实现项目。

如下所示：

```
public final class ServiceLoader<S>
    implements Iterable<S> {

    private static final String PREFIX = "META-INF/services/";

    // The class or interface representing the service being loaded
    private final Class<S> service;

    // The class loader used to locate, load, and instantiate providers
    private final ClassLoader loader;

    // The access control context taken when the ServiceLoader is created
    private final AccessControlContext acc;

    // Cached providers, in instantiation order
    private LinkedHashMap<String,S> providers = new LinkedHashMap<>();

    // The current lazy-lookup iterator
    private LazyIterator lookupIterator;
```

```
public Iterator<S> iterator() {
        return new Iterator<S>() {

            Iterator<Map.Entry<String,S>> knownProviders
                = providers.entrySet().iterator();

            public boolean hasNext() {
                if (knownProviders.hasNext())
                    return true;
                return lookupIterator.hasNext();
            }

            public S next() {
                if (knownProviders.hasNext())
                    return knownProviders.next().getValue();
                return lookupIterator.next();
            }

            public void remove() {
                throw new UnsupportedOperationException();
            }

        };
    }
```

### 2.5.2. 创建测试类

如下所示：

```
public S next() {
            if (acc == null) {
                return nextService();
            } else {
                PrivilegedAction<S> action = new PrivilegedAction<S>() {
                    public S run() { return nextService(); }
                };
                return AccessController.doPrivileged(action, acc);
            }
        }
```

```
private S nextService() {
            if (!hasNextService())
                throw new NoSuchElementException();
            String cn = nextName;
            nextName = null;
            Class<?> c = null;
            try {
                c = Class.forName(cn, false, loader);
            } catch (ClassNotFoundException x) {
                fail(service,
                     "Provider " + cn + " not found");
            }
            if (!service.isAssignableFrom(c)) {
                fail(service,
                     "Provider " + cn  + " not a subtype");
            }
            try {
                S p = service.cast(c.newInstance());
                providers.put(cn, p);
                return p;
            } catch (Throwable x) {
                fail(service,
                     "Provider " + cn + " could not be instantiated",
                     x);
            }
            throw new Error();          // This cannot happen
        }
```

运行测试类，其结果如下所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXEhqHAb2ZmfT0pDBcTAjVde36pW6240JZ8qH1qZ8cHJTefEBtTAqarcRYetuF2WxN62pH1zYYUIg/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

我们发现，Java 的 SPI 机制获取了所有 Person 类的实现类，并执行其对应的 favorite 方法。

# **3.SPI 机制的原理**

## 3.1.ServiceLoader 的核心属性

其核心机制就是 ServiceLoader 类的 load 方法，下面我们将从源码来分析其原理。

首先我们先看下 ServiceLoader 的核心属性：

```
private boolean hasNextService() {
            if (nextName != null) {
                return true;
            }
            if (configs == null) {
                try {
                    String fullName = PREFIX + service.getName();
                    if (loader == null)
                        configs = ClassLoader.getSystemResources(fullName);
                    else
                        configs = loader.getResources(fullName);
                } catch (IOException x) {
                    fail(service, "Error locating configuration files", x);
                }
            }
            while ((pending == null) || !pending.hasNext()) {
                if (!configs.hasMoreElements()) {
                    return false;
                }
                pending = parse(service, configs.nextElement());
            }
            nextName = pending.next();
            return true;
        }
```

这个 PREFIX 属性、providers 属性和 lookupIterator 属性将在后续的代码中使用到，我们发现 PREFIX 属性就是示例中说的 META-INF/services 路径。

## 3.2.ServiceLoader 的遍历器

示例中，我们会获取 serviceLoader 的遍历器 iterator，其方法如下所示：

```
public Iterator<S> iterator() {
        return new Iterator<S>() {
            Iterator<Map.Entry<String,S>> knownProviders
                = providers.entrySet().iterator();
            public boolean hasNext() {
                if (knownProviders.hasNext())
                    return true;
                return lookupIterator.hasNext();
            }
            public S next() {
                if (knownProviders.hasNext())
                    return knownProviders.next().getValue();
                return lookupIterator.next();
            }
            public void remove() {
                throw new UnsupportedOperationException();
            }
        };
    }
```

然后需要执行遍历器的 next 方法获取元素，其 next 方法执行的是`lookupIterator.next()`。

接下来我们来看下 lookupIterator 的 next 方法：

```
public S next() {
            if (acc == null) {
                return nextService();
            } else {
                PrivilegedAction<S> action = new PrivilegedAction<S>() {
                    public S run() { return nextService(); }
                };
                return AccessController.doPrivileged(action, acc);
            }
        }
```

其执行的是`nextService`方法，如下所示：

```
private S nextService() {
            if (!hasNextService())
                throw new NoSuchElementException();
            String cn = nextName;
            nextName = null;
            Class<?> c = null;
            try {
                c = Class.forName(cn, false, loader);
            } catch (ClassNotFoundException x) {
                fail(service,
                     "Provider " + cn + " not found");
            }
            if (!service.isAssignableFrom(c)) {
                fail(service,
                     "Provider " + cn  + " not a subtype");
            }
            try {
                S p = service.cast(c.newInstance());
                providers.put(cn, p);
                return p;
            } catch (Throwable x) {
                fail(service,
                     "Provider " + cn + " could not be instantiated",
                     x);
            }
            throw new Error();          // This cannot happen
        }
```

`nextService`方法首先执行`hasNextService`方法，如下所示：

```
private boolean hasNextService() {
            if (nextName != null) {
                return true;
            }
            if (configs == null) {
                try {
                    String fullName = PREFIX + service.getName();
                    if (loader == null)
                        configs = ClassLoader.getSystemResources(fullName);
                    else
                        configs = loader.getResources(fullName);
                } catch (IOException x) {
                    fail(service, "Error locating configuration files", x);
                }
            }
            while ((pending == null) || !pending.hasNext()) {
                if (!configs.hasMoreElements()) {
                    return false;
                }
                pending = parse(service, configs.nextElement());
            }
            nextName = pending.next();
            return true;
        }
```

这个方法会执行`String fullName = PREFIX + service.getName()`，而 PREFIX 就是我们前面刚才说的非常重要的属性，其值为`META-INF/services/`，`service`就是接口类，其最终的 fullName 指的就是`META-INF/services`文件夹下的名称为`com.jd.spi.Person`的文件。

接着会执行`configs = loader.getResources(fullName)`方法，这个方法这里不做详细描述，其主要功能就是获取类路径下所有相对路径为 fullName 的所有文件的 URL 对象。

然后会执行`pending = parse(service, configs.nextElement())`方法，这个方法这里也不详细描述，其主要功能是读取文件，将文件内容变成字符串，然后`nextName`就被赋值为当前文件的内容，即**实现类的接口全限定名**。

因此，执行`hasNextService()`方法后，nextName 被赋值为一个实现类的全限定名。

我们继续看上面的`nextService()`方法，其最终会执行`c = Class.forName(cn, false, loader)`方法，这个方法很明显就是通过反射实例化一个对象。通过一系列操作，最终返回了对应实现类的对象。

## 3.3. 流程总结

我们将其总结为以下几个步骤：

1. 创建 ServiceLoader 对象

2. 创建迭代器 lookupIterator

3. 通过迭代器的 hasNextService 方法读取类路径下`META-INF/services`目录的所有名称为接口全限定名的文件，将其内容存入 configs 对象中

4. 从 configs 对象中获取实现类的全限定名，然后通过反射实例化对象

从上述流程，我们也可以总结实现 SPI 的几点重要信息：

1. 实现工程必须在类路径下的`META-INF/services`目录下创建接口全限定名的文件，其文件内容必须是接口实现类的全限定名

2. 实现类必须有一个无参构造方法，因为 SPI 默认是使用无参构造方法实例化对象的

# **4. 总结**

本文首先概述了 Java 的 SPI 机制，随后阐述了其基本使用方法，最后深入探讨了其实现原理。SPI 在 Java 语言体系中具有广泛应用，能够有效地实现系统解耦，众多框架基于此机制进行了拓展和优化，从而实现了更为强大的 SPI 机制。掌握 SPI 的使用技巧可以帮助我们设计出更为灵活的系统，而深入理解其原理则有助于提升我们的技术水平。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXGicp40bMAicmX9DpEDjMlfPJT23acLpRzmuyiaguHv0VlmVDyEFGwd36gZYRShzhv0EPleicHyvk7KA/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=wxpic)

  

扫一扫，加入技术交流群