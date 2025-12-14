---
source:
  - https://pdai.tech/md/spring/spring-x-framework-aop-source-3.html
create: 2025-11-28
read: true
knowledge: true
knowledge-date: 2025-11-28
tags:
  - Spring
  - 框架原理
summary: "[[Spring AOP 实现原理详解之 AOP 代理的创建]]"
---

> 我们在前文中已经介绍了 SpringAOP 的切面实现和创建动态代理的过程，那么动态代理是如何工作的呢？本文主要介绍 Cglib 动态代理的案例和 SpringAOP 实现的原理。@pdai

- [Spring进阶 - Spring AOP实现原理详解之Cglib代理实现](#spring%E8%BF%9B%E9%98%B6---spring-aop%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86%E8%AF%A6%E8%A7%A3%E4%B9%8Bcglib%E4%BB%A3%E7%90%86%E5%AE%9E%E7%8E%B0)
    - [引入](#%E5%BC%95%E5%85%A5)
        - [动态代理要解决什么问题？](#%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86%E8%A6%81%E8%A7%A3%E5%86%B3%E4%BB%80%E4%B9%88%E9%97%AE%E9%A2%98)
            - [什么是代理？](#%E4%BB%80%E4%B9%88%E6%98%AF%E4%BB%A3%E7%90%86)
            - [什么是动态代理？](#%E4%BB%80%E4%B9%88%E6%98%AF%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86)
        - [什么是Cglib? SpringAOP和Cglib是什么关系？](#%E4%BB%80%E4%B9%88%E6%98%AFcglib-springaop%E5%92%8Ccglib%E6%98%AF%E4%BB%80%E4%B9%88%E5%85%B3%E7%B3%BB)
    - [Cglib代理的案例](#cglib%E4%BB%A3%E7%90%86%E7%9A%84%E6%A1%88%E4%BE%8B)
        - [pom包依赖](#pom%E5%8C%85%E4%BE%9D%E8%B5%96)
        - [定义实体](#%E5%AE%9A%E4%B9%89%E5%AE%9E%E4%BD%93)
        - [被代理的类](#%E8%A2%AB%E4%BB%A3%E7%90%86%E7%9A%84%E7%B1%BB)
        - [cglib代理](#cglib%E4%BB%A3%E7%90%86)
        - [使用代理](#%E4%BD%BF%E7%94%A8%E4%BB%A3%E7%90%86)
        - [简单测试](#%E7%AE%80%E5%8D%95%E6%B5%8B%E8%AF%95)
    - [Cglib代理的流程](#cglib%E4%BB%A3%E7%90%86%E7%9A%84%E6%B5%81%E7%A8%8B)
    - [SpringAOP中Cglib代理的实现](#springaop%E4%B8%ADcglib%E4%BB%A3%E7%90%86%E7%9A%84%E5%AE%9E%E7%8E%B0)
    - [示例源码](#%E7%A4%BA%E4%BE%8B%E6%BA%90%E7%A0%81)

## 引入

> 我们在前文中已经介绍了 SpringAOP 的切面实现和创建动态代理的过程，那么动态代理是如何工作的呢？本文主要介绍 Cglib 动态代理的案例和 SpringAOP 实现的原理。

要了解动态代理是如何工作的，首先需要了解

- 什么是代理模式？
- 什么是动态代理？
- 什么是 Cglib？
- SpringAOP 和 Cglib 是什么关系？

### 动态代理要解决什么问题？

#### 什么是代理？

**代理模式**(Proxy pattern): 为另一个对象提供一个替身或占位符以控制对这个对象的访问

![](https://pdai.tech/images/pics/a6c20f60-5eba-427d-9413-352ada4b40fe.png)

举个简单的例子：

我(client)如果要买(doOperation)房，可以找中介(proxy)买房，中介直接和卖方(target)买房。中介和卖方都实现买卖(doOperation)的操作。中介就是代理(proxy)。

#### 什么是动态代理？

> 动态代理就是，在程序运行期，创建目标对象的代理对象，并对目标对象中的方法进行功能性增强的一种技术。

在生成代理对象的过程中，目标对象不变，代理对象中的方法是目标对象方法的增强方法。可以理解为运行期间，对象中方法的动态拦截，在拦截方法的前后执行功能操作。

![](https://pdai.tech/images/spring/springframework/spring-springframework-aop-61.png)

### 什么是 Cglib? SpringAOP 和 Cglib 是什么关系？

> Cglib 是一个强大的、高性能的代码生成包，它广泛被许多 AOP 框架使用，为他们提供方法的拦截。

![](https://pdai.tech/images/spring/springframework/spring-springframework-aop-62.png)

- 最底层是字节码，字节码相关的知识请参考 [JVM基础 - 类字节码详解](https://pdai.tech/md/java/jvm/java-jvm-class.html)
- ASM 是操作字节码的工具
- cglib 基于 ASM 字节码工具操作字节码（即动态生成代理，对方法进行增强）
- SpringAOP 基于 cglib 进行封装，实现 cglib 方式的动态代理

## Cglib 代理的案例

> 这里我们写一个使用 cglib 的简单例子。@pdai

### pom 包依赖

引入 cglib 的依赖包

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>tech-pdai-spring-demos</artifactId>
        <groupId>tech.pdai</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>007-spring-framework-demo-aop-proxy-cglib</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- https://mvnrepository.com/artifact/cglib/cglib -->
        <dependency>
            <groupId>cglib</groupId>
            <artifactId>cglib</artifactId>
            <version>3.3.0</version>
        </dependency>
    </dependencies>

</project>
```

### 定义实体

User

```java
package tech.pdai.springframework.entity;

/**
 * @author pdai
 */
public class User {

    /**
     * user's name.
     */
    private String name;

    /**
     * user's age.
     */
    private int age;

    /**
     * init.
     *
     * @param name name
     * @param age  age
     */
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

### 被代理的类

即目标类, 对被代理的类中的方法进行增强

```java
package tech.pdai.springframework.service;

import java.util.Collections;
import java.util.List;

import tech.pdai.springframework.entity.User;

/**
 * @author pdai
 */
public class UserServiceImpl {

    /**
     * find user list.
     *
     * @return user list
     */
    public List<User> findUserList() {
        return Collections.singletonList(new User("pdai", 18));
    }

    /**
     * add user
     */
    public void addUser() {
        // do something
    }

}
```

### cglib 代理

cglib 代理类，需要实现 MethodInterceptor 接口，并指定代理目标类 target

```java
package tech.pdai.springframework.proxy;

import java.lang.reflect.Method;

import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

/**
 * This class is for proxy demo.
 *
 * @author pdai
 */
public class UserLogProxy implements MethodInterceptor {

    /**
     * 业务类对象，供代理方法中进行真正的业务方法调用
     */
    private Object target;

    public Object getUserLogProxy(Object target) {
        //给业务对象赋值
        this.target = target;
        //创建加强器，用来创建动态代理类
        Enhancer enhancer = new Enhancer();
        //为加强器指定要代理的业务类（即：为下面生成的代理类指定父类）
        enhancer.setSuperclass(this.target.getClass());
        //设置回调：对于代理类上所有方法的调用，都会调用CallBack，而Callback则需要实现intercept()方法进行拦
        enhancer.setCallback(this);
        // 创建动态代理类对象并返回
        return enhancer.create();
    }

    // 实现回调方法
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        // log - before method
        System.out.println("[before] execute method: " + method.getName());

        // call method
        Object result = proxy.invokeSuper(obj, args);

        // log - after method
        System.out.println("[after] execute method: " + method.getName() + ", return value: " + result);
        return null;
    }
}
```

### 使用代理

启动类中指定代理目标并执行。

```java
package tech.pdai.springframework;

import tech.pdai.springframework.proxy.UserLogProxy;
import tech.pdai.springframework.service.UserServiceImpl;

/**
 * Cglib proxy demo.
 *
 * @author pdai
 */
public class ProxyDemo {

    /**
     * main interface.
     *
     * @param args args
     */
    public static void main(String[] args) {
        // proxy
        UserServiceImpl userService = (UserServiceImpl) new UserLogProxy().getUserLogProxy(new UserServiceImpl());

        // call methods
        userService.findUserList();
        userService.addUser();
    }
}
```

### 简单测试

我们启动上述类 main 函数，执行的结果如下：

```java
[before] execute method: findUserList
[after] execute method: findUserList, return value: [User{name='pdai', age=18}]
[before] execute method: addUser
[after] execute method: addUser, return value: null
```

## Cglib 代理的流程

我们把上述 Demo 的主要流程画出来，你便能很快理解

![](https://pdai.tech/images/spring/springframework/spring-springframework-aop-63.png)

更多细节：

- 在上图中，我们可以通过在 Enhancer 中配置更多的参数来控制代理的行为，比如如果只希望增强这个类中的一个方法（而不是所有方法），那就增加 callbackFilter 来对目标类中方法进行过滤；Enhancer 可以有更多的参数类配置其行为，不过我们在学习上述主要的流程就够了。
- final 方法为什么不能被代理？很显然 final 方法没法被子类覆盖，当然不能代理了。
- Mockito 为什么不能 mock 静态方法？因为 mockito 也是基于 cglib 动态代理来实现的，static 方法也不能被子类覆盖，所以显然不能 mock。但 PowerMock 可以 mock 静态方法，因为它直接在 bytecode 上工作，更多可以看[Mockito单元测试](https://pdai.tech/md/develop/ut/dev-ut-x-mockito.html)。（pdai: 通了没？是不是 so easy...）

## SpringAOP 中 Cglib 代理的实现

> SpringAOP 封装了 cglib，通过其进行动态代理的创建。

我们看下 CglibAopProxy 的 getProxy 方法

```java
@Override
public Object getProxy() {
  return getProxy(null);
}

@Override
public Object getProxy(@Nullable ClassLoader classLoader) {
  if (logger.isTraceEnabled()) {
    logger.trace("Creating CGLIB proxy: " + this.advised.getTargetSource());
  }

  try {
    Class<?> rootClass = this.advised.getTargetClass();
    Assert.state(rootClass != null, "Target class must be available for creating a CGLIB proxy");

    // 上面流程图中的目标类
    Class<?> proxySuperClass = rootClass;
    if (rootClass.getName().contains(ClassUtils.CGLIB_CLASS_SEPARATOR)) {
      proxySuperClass = rootClass.getSuperclass();
      Class<?>[] additionalInterfaces = rootClass.getInterfaces();
      for (Class<?> additionalInterface : additionalInterfaces) {
        this.advised.addInterface(additionalInterface);
      }
    }

    // Validate the class, writing log messages as necessary.
    validateClassIfNecessary(proxySuperClass, classLoader);

    // 重点看这里，就是上图的enhancer，设置各种参数来构建
    Enhancer enhancer = createEnhancer();
    if (classLoader != null) {
      enhancer.setClassLoader(classLoader);
      if (classLoader instanceof SmartClassLoader &&
          ((SmartClassLoader) classLoader).isClassReloadable(proxySuperClass)) {
        enhancer.setUseCache(false);
      }
    }
    enhancer.setSuperclass(proxySuperClass);
    enhancer.setInterfaces(AopProxyUtils.completeProxiedInterfaces(this.advised));
    enhancer.setNamingPolicy(SpringNamingPolicy.INSTANCE);
    enhancer.setStrategy(new ClassLoaderAwareGeneratorStrategy(classLoader));

    // 设置callback回调接口，即方法的增强点
    Callback[] callbacks = getCallbacks(rootClass);
    Class<?>[] types = new Class<?>[callbacks.length];
    for (int x = 0; x < types.length; x++) {
      types[x] = callbacks[x].getClass();
    }
    // 上节说到的filter
    enhancer.setCallbackFilter(new ProxyCallbackFilter(
        this.advised.getConfigurationOnlyCopy(), this.fixedInterceptorMap, this.fixedInterceptorOffset));
    enhancer.setCallbackTypes(types);

    // 重点：创建proxy和其实例
    return createProxyClassAndInstance(enhancer, callbacks);
  }
  catch (CodeGenerationException | IllegalArgumentException ex) {
    throw new AopConfigException("Could not generate CGLIB subclass of " + this.advised.getTargetClass() +
        ": Common causes of this problem include using a final class or a non-visible class",
        ex);
  }
  catch (Throwable ex) {
    // TargetSource.getTarget() failed
    throw new AopConfigException("Unexpected AOP exception", ex);
  }
}
```

获取 callback 的方法如下，提几个理解的要点吧，具体读者在学习的时候建议把我的例子跑一下，然后打一个断点进行理解。

- `rootClass`: 即目标代理类
- `advised`: 包含上文中我们获取到的 advisor 增强器的集合
- `exposeProxy`: 在 xml 配置文件中配置的，背景就是如果在事务 A 中使用了代理，事务 A 调用了目标类的的方法 a，在方法 a 中又调用目标类的方法 b，方法 a，b 同时都是要被增强的方法，如果不配置 exposeProxy 属性，方法 b 的增强将会失效，如果配置 exposeProxy，方法 b 在方法 a 的执行中也会被增强了
- `DynamicAdvisedInterceptor`: 拦截器将 advised(包含上文中我们获取到的 advisor 增强器)构建配置的 AOP 的 callback（第一个 callback)
- `targetInterceptor`: xml 配置的 optimize 属性使用的（第二个 callback)
- 最后连同其它 5 个默认的 Interceptor 返回作为 cglib 的拦截器链，之后通过 CallbackFilter 的 accpet 方法返回的索引从这个集合中返回对应的拦截增强器执行增强操作。

```java
private Callback[] getCallbacks(Class<?> rootClass) throws Exception {
  // Parameters used for optimization choices...
  boolean exposeProxy = this.advised.isExposeProxy();
  boolean isFrozen = this.advised.isFrozen();
  boolean isStatic = this.advised.getTargetSource().isStatic();

  // Choose an "aop" interceptor (used for AOP calls).
  Callback aopInterceptor = new DynamicAdvisedInterceptor(this.advised);

  // Choose a "straight to target" interceptor. (used for calls that are
  // unadvised but can return this). May be required to expose the proxy.
  Callback targetInterceptor;
  if (exposeProxy) {
    targetInterceptor = (isStatic ?
        new StaticUnadvisedExposedInterceptor(this.advised.getTargetSource().getTarget()) :
        new DynamicUnadvisedExposedInterceptor(this.advised.getTargetSource()));
  }
  else {
    targetInterceptor = (isStatic ?
        new StaticUnadvisedInterceptor(this.advised.getTargetSource().getTarget()) :
        new DynamicUnadvisedInterceptor(this.advised.getTargetSource()));
  }

  // Choose a "direct to target" dispatcher (used for
  // unadvised calls to static targets that cannot return this).
  Callback targetDispatcher = (isStatic ?
      new StaticDispatcher(this.advised.getTargetSource().getTarget()) : new SerializableNoOp());

  Callback[] mainCallbacks = new Callback[] {
      aopInterceptor,  // 
      targetInterceptor,  // invoke target without considering advice, if optimized
      new SerializableNoOp(),  // no override for methods mapped to this
      targetDispatcher, this.advisedDispatcher,
      new EqualsInterceptor(this.advised),
      new HashCodeInterceptor(this.advised)
  };

  Callback[] callbacks;

  // If the target is a static one and the advice chain is frozen,
  // then we can make some optimizations by sending the AOP calls
  // direct to the target using the fixed chain for that method.
  if (isStatic && isFrozen) {
    Method[] methods = rootClass.getMethods();
    Callback[] fixedCallbacks = new Callback[methods.length];
    this.fixedInterceptorMap = CollectionUtils.newHashMap(methods.length);

    // TODO: small memory optimization here (can skip creation for methods with no advice)
    for (int x = 0; x < methods.length; x++) {
      Method method = methods[x];
      List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, rootClass);
      fixedCallbacks[x] = new FixedChainStaticTargetInterceptor(
          chain, this.advised.getTargetSource().getTarget(), this.advised.getTargetClass());
      this.fixedInterceptorMap.put(method, x);
    }

    // Now copy both the callbacks from mainCallbacks
    // and fixedCallbacks into the callbacks array.
    callbacks = new Callback[mainCallbacks.length + fixedCallbacks.length];
    System.arraycopy(mainCallbacks, 0, callbacks, 0, mainCallbacks.length);
    System.arraycopy(fixedCallbacks, 0, callbacks, mainCallbacks.length, fixedCallbacks.length);
    this.fixedInterceptorOffset = mainCallbacks.length;
  }
  else {
    callbacks = mainCallbacks;
  }
  return callbacks;
}
```

可以结合调试，方便理解

![](https://pdai.tech/images/spring/springframework/spring-springframework-aop-64.png)

## 示例源码

https://github.com/realpdai/tech-pdai-spring-demos