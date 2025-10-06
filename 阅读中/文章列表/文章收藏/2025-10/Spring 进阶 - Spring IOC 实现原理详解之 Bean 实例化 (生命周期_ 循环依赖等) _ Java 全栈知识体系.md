---
source: https://pdai.tech/md/spring/spring-x-framework-ioc-source-3.html
create: 2025-10-03 14:12
read: false
knowledge: false
---

上文，我们看了 IOC 设计要点和设计结构；以及 Spring 如何实现将资源配置（以 xml 配置为例）通过加载，解析，生成 BeanDefination 并注册到 IoC 容器中的；容器中存放的是 Bean 的定义即 BeanDefinition 放到 beanDefinitionMap 中，本质上是一个 `ConcurrentHashMap<String, Object>`；并且 BeanDefinition 接口中包含了这个类的 Class 信息以及是否是单例等。那么如何从 BeanDefinition 中实例化 Bean 对象呢，这是本文主要研究的内容？@pdai

*   [Spring 进阶 - Spring IOC 实现原理详解之 Bean 实例化 (生命周期, 循环依赖等)](#spring%E8%BF%9B%E9%98%B6--spring-ioc%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86%E8%AF%A6%E8%A7%A3%E4%B9%8Bbean%E5%AE%9E%E4%BE%8B%E5%8C%96%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96%E7%AD%89)
    *   [引入](#%E5%BC%95%E5%85%A5)
    *   [BeanFactory 中 getBean 的主体思路](#beanfactory%E4%B8%ADgetbean%E7%9A%84%E4%B8%BB%E4%BD%93%E6%80%9D%E8%B7%AF)
        *   [初步的思考](#%E5%88%9D%E6%AD%A5%E7%9A%84%E6%80%9D%E8%80%83)
        *   [Spring 中 getBean 的主体思路](#spring%E4%B8%ADgetbean%E7%9A%84%E4%B8%BB%E4%BD%93%E6%80%9D%E8%B7%AF)
    *   [重点：Spring 如何解决循环依赖问题](#%E9%87%8D%E7%82%B9spring%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96%E9%97%AE%E9%A2%98)
        *   [Spring 单例模式下的属性依赖](#spring%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F%E4%B8%8B%E7%9A%84%E5%B1%9E%E6%80%A7%E4%BE%9D%E8%B5%96)
        *   [Spring 为何不能解决非单例属性之外的循环依赖？](#spring%E4%B8%BA%E4%BD%95%E4%B8%8D%E8%83%BD%E8%A7%A3%E5%86%B3%E9%9D%9E%E5%8D%95%E4%BE%8B%E5%B1%9E%E6%80%A7%E4%B9%8B%E5%A4%96%E7%9A%84%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96)
            *   [Spring 为什么不能解决构造器的循环依赖？](#spring%E4%B8%BA%E4%BB%80%E4%B9%88%E4%B8%8D%E8%83%BD%E8%A7%A3%E5%86%B3%E6%9E%84%E9%80%A0%E5%99%A8%E7%9A%84%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96)
            *   [Spring 为什么不能解决 prototype 作用域循环依赖？](#spring%E4%B8%BA%E4%BB%80%E4%B9%88%E4%B8%8D%E8%83%BD%E8%A7%A3%E5%86%B3prototype%E4%BD%9C%E7%94%A8%E5%9F%9F%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96)
            *   [Spring 为什么不能解决多例的循环依赖？](#spring%E4%B8%BA%E4%BB%80%E4%B9%88%E4%B8%8D%E8%83%BD%E8%A7%A3%E5%86%B3%E5%A4%9A%E4%BE%8B%E7%9A%84%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96)
        *   [那么其它循环依赖如何解决？](#%E9%82%A3%E4%B9%88%E5%85%B6%E5%AE%83%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3)
    *   [重点：Spring 中 Bean 的生命周期](#%E9%87%8D%E7%82%B9spring%E4%B8%ADbean%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F)
        *   [Spring Bean 生命周期流程](#spring-bean%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E6%B5%81%E7%A8%8B)
        *   [Spring Bean 生命周期案例](#spring-bean%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E6%A1%88%E4%BE%8B)
        *   [Spring Bean 生命周期源码](#spring-bean%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E6%BA%90%E7%A0%81)
    *   [参考文章](#%E5%8F%82%E8%80%83%E6%96%87%E7%AB%A0)

## [#](#引入) 引入

上文，我们看了 IOC 设计要点和设计结构；以及 Spring 如何实现将资源配置（以 xml 配置为例）通过加载，解析，生成 BeanDefination 并注册到 IoC 容器中的；容器中存放的是 Bean 的定义即 BeanDefinition 放到 beanDefinitionMap 中，本质上是一个 `ConcurrentHashMap<String, Object>`；并且 BeanDefinition 接口中包含了这个类的 Class 信息以及是否是单例等。那么如何从 BeanDefinition 中实例化 Bean 对象呢？

本文主要研究如何从 IOC 容器已有的 BeanDefinition 信息，实例化出 Bean 对象；这里还会包括三块重点内容：

*   BeanFactory 中 getBean 的主体思路
*   Spring 如何解决循环依赖问题
*   Spring 中 Bean 的生命周期

![](https://pdai.tech/images/spring/springframework/spring-framework-ioc-source-74.png)

## [#](#beanfactory中getbean的主体思路) BeanFactory 中 getBean 的主体思路

上文中我们知道 BeanFactory 定义了 Bean 容器的规范，其中包含根据 bean 的名字, Class 类型和参数等来得到 bean 实例。

```java
Object getBean(String name) throws BeansException;    
Object getBean(String name, Class requiredType) throws BeansException;    
Object getBean(String name, Object... args) throws BeansException;
<T> T getBean(Class<T> requiredType) throws BeansException;
<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;


```

### [#](#初步的思考) 初步的思考

上文我们已经分析了 IoC 初始化的流程，最终的将 Bean 的定义即 BeanDefinition 放到 beanDefinitionMap 中，本质上是一个 `ConcurrentHashMap<String, Object>`；并且 BeanDefinition 接口中包含了这个类的 Class 信息以及是否是单例等；

![](https://pdai.tech/images/spring/springframework/spring-framework-ioc-source-100.png)

这样我们初步有了实现 `Object getBean(String name)` 这个方法的思路：

*   从 beanDefinitionMap 通过 beanName 获得 BeanDefinition
*   从 BeanDefinition 中获得 beanClassName
*   通过反射初始化 beanClassName 的实例 instance
    *   构造函数从 BeanDefinition 的 getConstructorArgumentValues() 方法获取
    *   属性值从 BeanDefinition 的 getPropertyValues() 方法获取
*   返回 beanName 的实例 instance

由于 BeanDefinition 还有单例的信息，如果是无参构造函数的实例还可以放在一个缓存中，这样下次获取这个单例的实例时只需要从缓存中获取，如果获取不到再通过上述步骤获取。

（PS：如上只是我们初步的思路，而 Spring 还需要考虑各种设计上的问题，比如 beanDefinition 中其它定义，循环依赖等；所以我们来看下 Spring 是如何是如何实现的）

### [#](#spring中getbean的主体思路) Spring 中 getBean 的主体思路

BeanFactory 实现 getBean 方法在 AbstractBeanFactory 中，这个方法重载都是调用 doGetBean 方法进行实现的：

```java
public Object getBean(String name) throws BeansException {
  return doGetBean(name, null, null, false);
}
public <T> T getBean(String name, Class<T> requiredType) throws BeansException {
  return doGetBean(name, requiredType, null, false);
}
public Object getBean(String name, Object... args) throws BeansException {
  return doGetBean(name, null, args, false);
}
public <T> T getBean(String name, @Nullable Class<T> requiredType, @Nullable Object... args)
    throws BeansException {
  return doGetBean(name, requiredType, args, false);
}


```

我们来看下 doGetBean 方法 (这个方法很长，我们主要看它的整体思路和设计要点）：

```java
protected <T> T doGetBean(
			String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly)
			throws BeansException {

  
  String beanName = transformedBeanName(name);
  Object beanInstance;

  
  Object sharedInstance = getSingleton(beanName);
  if (sharedInstance != null && args == null) {
    
    beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, null);
  }

  else {
    
    if (isPrototypeCurrentlyInCreation(beanName)) {
      throw new BeanCurrentlyInCreationException(beanName);
    }

    
    BeanFactory parentBeanFactory = getParentBeanFactory();
    if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
      
      String nameToLookup = originalBeanName(name);
      if (parentBeanFactory instanceof AbstractBeanFactory) {
        return ((AbstractBeanFactory) parentBeanFactory).doGetBean(
            nameToLookup, requiredType, args, typeCheckOnly);
      }
      else if (args != null) {
        
        return (T) parentBeanFactory.getBean(nameToLookup, args);
      }
      else if (requiredType != null) {
        
        return parentBeanFactory.getBean(nameToLookup, requiredType);
      }
      else {
        return (T) parentBeanFactory.getBean(nameToLookup);
      }
    }

    if (!typeCheckOnly) {
      
      markBeanAsCreated(beanName);
    }

    StartupStep beanCreation = this.applicationStartup.start("spring.beans.instantiate")
        .tag("beanName", name);
    try {
      if (requiredType != null) {
        beanCreation.tag("beanType", requiredType::toString);
      }
      RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
      checkMergedBeanDefinition(mbd, beanName, args);

      
      String[] dependsOn = mbd.getDependsOn();
      if (dependsOn != null) {
        for (String dep : dependsOn) {
          if (isDependent(beanName, dep)) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                "Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
          }
          registerDependentBean(dep, beanName);
          try {
            getBean(dep); 
          }
          catch (NoSuchBeanDefinitionException ex) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                "'" + beanName + "' depends on missing bean '" + dep + "'", ex);
          }
        }
      }

      
      if (mbd.isSingleton()) {
        sharedInstance = getSingleton(beanName, () -> {
          try {
            
            return createBean(beanName, mbd, args);
          }
          catch (BeansException ex) {
            
            
            
            destroySingleton(beanName);
            throw ex;
          }
        });
        beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
      }
      
      else if (mbd.isPrototype()) {
        
        Object prototypeInstance = null;
        try {
          beforePrototypeCreation(beanName);
          prototypeInstance = createBean(beanName, mbd, args);
        }
        finally {
          afterPrototypeCreation(beanName);
        }
        beanInstance = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
      }
      
      else {
        String scopeName = mbd.getScope();
        if (!StringUtils.hasLength(scopeName)) {
          throw new IllegalStateException("No scope name defined for bean ´" + beanName + "'");
        }
        Scope scope = this.scopes.get(scopeName);
        if (scope == null) {
          throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
        }
        try {
          Object scopedInstance = scope.get(beanName, () -> {
            beforePrototypeCreation(beanName);
            try {
              return createBean(beanName, mbd, args);
            }
            finally {
              afterPrototypeCreation(beanName);
            }
          });
          beanInstance = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
        }
        catch (IllegalStateException ex) {
          throw new ScopeNotActiveException(beanName, scopeName, ex);
        }
      }
    }
    catch (BeansException ex) {
      beanCreation.tag("exception", ex.getClass().toString());
      beanCreation.tag("message", String.valueOf(ex.getMessage()));
      cleanupAfterBeanCreationFailure(beanName);
      throw ex;
    }
    finally {
      beanCreation.end();
    }
  }

  return adaptBeanInstance(name, beanInstance, requiredType);
}


```

这段代码很长，主要看我加中文注释的方法即可。

*   解析 bean 的真正 name，如果 bean 是工厂类，name 前缀会加 &，需要去掉
*   无参单例先从缓存中尝试获取
*   如果 bean 实例还在创建中，则直接抛出异常
*   如果 bean definition 存在于父的 bean 工厂中，委派给父 Bean 工厂获取
*   标记这个 beanName 的实例正在创建
*   确保它的依赖也被初始化
*   真正创建
    *   单例时
    *   原型时
    *   根据 bean 的 scope 创建

## [#](#重点-spring如何解决循环依赖问题) 重点：Spring 如何解决循环依赖问题

首先我们需要说明，Spring 只是解决了单例模式下属性依赖的循环问题；Spring 为了解决单例的循环依赖问题，使用了三级缓存。

### [#](#spring单例模式下的属性依赖) Spring 单例模式下的属性依赖

先来看下这三级缓存

```java
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);
 

private final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);


private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>(16);



```

*   **第一层缓存（singletonObjects）**：单例对象缓存池，已经实例化并且属性赋值，这里的对象是**成熟对象**；
*   **第二层缓存（earlySingletonObjects）**：单例对象缓存池，已经实例化但尚未属性赋值，这里的对象是**半成品对象**；
*   **第三层缓存（singletonFactories）**: 单例工厂的缓存

如下是获取单例中

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
  
  Object singletonObject = this.singletonObjects.get(beanName);
  
  if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
    synchronized (this.singletonObjects) {
        singletonObject = this.earlySingletonObjects.get(beanName);
        if (singletonObject == null && allowEarlyReference) {
          ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
          if (singletonFactory != null) {
            
              singletonObject = singletonFactory.getObject();
              
              this.earlySingletonObjects.put(beanName, singletonObject);
              this.singletonFactories.remove(beanName);
          }
        }
    }
  }
  return (singletonObject != NULL_OBJECT ? singletonObject : null);
}


```

补充一些方法和参数

*   `isSingletonCurrentlyInCreation()`：判断当前单例 bean 是否正在建立中，也就是没有初始化完成 (好比 A 的构造器依赖了 B 对象因此得先去建立 B 对象，或则在 A 的 populateBean 过程当中依赖了 B 对象，得先去建立 B 对象，这时的 A 就是处于建立中的状态。)
*   `allowEarlyReference` ：是否容许从 singletonFactories 中经过 getObject 拿到对象

分析 getSingleton() 的整个过程，Spring 首先从一级缓存 singletonObjects 中获取。若是获取不到，而且对象正在建立中，就再从二级缓存 earlySingletonObjects 中获取。若是仍是获取不到且容许 singletonFactories 经过 getObject() 获取，就从三级缓存 singletonFactory.getObject()(三级缓存) 获取，若是获取到了则从三级缓存移动到了二级缓存。

从上面三级缓存的分析，咱们能够知道，Spring 解决循环依赖的诀窍就在于 singletonFactories 这个三级 cache。这个 cache 的类型是 ObjectFactory，定义以下：

```java
public interface ObjectFactory<T> {
    T getObject() throws BeansException;
}


```

在 bean 建立过程当中，有两处比较重要的匿名内部类实现了该接口。一处是 Spring 利用其建立 bean 的时候，另外一处就是:

```java
addSingletonFactory(beanName, new ObjectFactory<Object>() {
   @Override   public Object getObject() throws BeansException {
      return getEarlyBeanReference(beanName, mbd, bean);
   }});


```

此处就是解决循环依赖的关键，这段代码发生在 createBeanInstance 以后，也就是说单例对象此时已经被建立出来的。这个对象已经被生产出来了，虽然还不完美（尚未进行初始化的第二步和第三步），可是已经能被人认出来了（根据对象引用能定位到堆中的对象），因此 Spring 此时将这个对象提早曝光出来让你们认识，让你们使用。

好比 “A 对象 setter 依赖 B 对象，B 对象 setter 依赖 A 对象”，A 首先完成了初始化的第一步，而且将本身提早曝光到 singletonFactories 中，此时进行初始化的第二步，发现本身依赖对象 B，此时就尝试去 get(B)，发现 B 尚未被 create，因此走 create 流程，B 在初始化第一步的时候发现本身依赖了对象 A，因而尝试 get(A)，尝试一级缓存 singletonObjects(确定没有，由于 A 还没初始化彻底)，尝试二级缓存 earlySingletonObjects（也没有），尝试三级缓存 singletonFactories，因为 A 经过 ObjectFactory 将本身提早曝光了，因此 B 可以经过 ObjectFactory.getObject 拿到 A 对象 (半成品)，B 拿到 A 对象后顺利完成了初始化阶段一、二、三，彻底初始化以后将本身放入到一级缓存 singletonObjects 中。此时返回 A 中，A 此时能拿到 B 的对象顺利完成本身的初始化阶段二、三，最终 A 也完成了初始化，进去了一级缓存 singletonObjects 中，并且更加幸运的是，因为 B 拿到了 A 的对象引用，因此 B 如今 hold 住的 A 对象完成了初始化。

### [#](#spring为何不能解决非单例属性之外的循环依赖) Spring 为何不能解决非单例属性之外的循环依赖？

通过以下几个问题，辅助我们进一步理解。

#### [#](#spring为什么不能解决构造器的循环依赖) Spring 为什么不能解决构造器的循环依赖？

构造器注入形成的循环依赖： 也就是 beanB 需要在 beanA 的构造函数中完成初始化，beanA 也需要在 beanB 的构造函数中完成初始化，这种情况的结果就是两个 bean 都不能完成初始化，循环依赖难以解决。

Spring 解决循环依赖主要是依赖三级缓存，但是的**在调用构造方法之前还未将其放入三级缓存之中**，因此后续的依赖调用构造方法的时候并不能从三级缓存中获取到依赖的 Bean，因此不能解决。

#### [#](#spring为什么不能解决prototype作用域循环依赖) Spring 为什么不能解决 prototype 作用域循环依赖？

这种循环依赖同样无法解决，因为 spring 不会缓存‘prototype’作用域的 bean，而 spring 中循环依赖的解决正是通过缓存来实现的。

#### [#](#spring为什么不能解决多例的循环依赖) Spring 为什么不能解决多例的循环依赖？

多实例 Bean 是每次调用一次 getBean 都会执行一次构造方法并且给属性赋值，根本没有三级缓存，因此不能解决循环依赖。

### [#](#那么其它循环依赖如何解决) 那么其它循环依赖如何解决？

那么实际开发中，类似的依赖是如何解决？

*   **生成代理对象产生的循环依赖**

这类循环依赖问题解决方法很多，主要有：

1.  使用 @Lazy 注解，延迟加载
2.  使用 @DependsOn 注解，指定加载先后关系
3.  修改文件名称，改变循环依赖类的加载顺序

*   **使用 @DependsOn 产生的循环依赖**

这类循环依赖问题要找到 @DependsOn 注解循环依赖的地方，迫使它不循环依赖就可以解决问题。

*   **多例循环依赖**

这类循环依赖问题可以通过把 bean 改成单例的解决。

*   **构造器循环依赖**

这类循环依赖问题可以通过使用 @Lazy 注解解决。

## [#](#重点-spring中bean的生命周期) 重点：Spring 中 Bean 的生命周期

Spring 只帮我们管理单例模式 Bean 的**完整**生命周期，对于 prototype 的 bean ，Spring 在创建好交给使用者之后则不会再管理后续的生命周期。

Spring 容器可以管理 singleton 作用域 Bean 的生命周期，在此作用域下，Spring 能够精确地知道该 Bean 何时被创建，何时初始化完成，以及何时被销毁。

而对于 prototype 作用域的 Bean，Spring 只负责创建，当容器创建了 Bean 的实例后，Bean 的实例就交给客户端代码管理，Spring 容器将不再跟踪其生命周期。每次客户端请求 prototype 作用域的 Bean 时，Spring 容器都会创建一个新的实例，并且不会管那些被配置成 prototype 作用域的 Bean 的生命周期。

了解 Spring 生命周期的意义就在于，**可以利用 Bean 在其存活期间的指定时刻完成一些相关操作**。这种时刻可能有很多，但一般情况下，会在 Bean 被初始化后和被销毁前执行一些相关操作。

### [#](#spring-bean生命周期流程) Spring Bean 生命周期流程

在 Spring 中，Bean 的生命周期是一个很复杂的执行过程，我们可以利用 Spring 提供的方法定制 Bean 的创建过程。

**Spring 容器中 Bean 的生命周期流程**

![](https://pdai.tech/images/spring/springframework/spring-framework-ioc-source-102.png)

*   如果 BeanFactoryPostProcessor 和 Bean 关联, 则调用 postProcessBeanFactory 方法.(即首**先尝试从 Bean 工厂中获取 Bean**)
*   如果 InstantiationAwareBeanPostProcessor 和 Bean 关联，则调用 postProcessBeforeInstantiation 方法
*   根据配置情况调用 Bean 构造方法**实例化 Bean**。
*   利用依赖注入完成 Bean 中所有**属性值的配置注入**。
*   如果 InstantiationAwareBeanPostProcessor 和 Bean 关联，则调用 postProcessAfterInstantiation 方法和 postProcessProperties
*   **调用 xxxAware 接口** (上图只是给了几个例子)
    *   **第一类 Aware 接口**
        *   如果 Bean 实现了 BeanNameAware 接口，则 Spring 调用 Bean 的 setBeanName() 方法传入当前 Bean 的 id 值。
        *   如果 Bean 实现了 BeanClassLoaderAware 接口，则 Spring 调用 setBeanClassLoader() 方法传入 classLoader 的引用。
        *   如果 Bean 实现了 BeanFactoryAware 接口，则 Spring 调用 setBeanFactory() 方法传入当前工厂实例的引用。
    *   **第二类 Aware 接口**
        *   如果 Bean 实现了 EnvironmentAware 接口，则 Spring 调用 setEnvironment() 方法传入当前 Environment 实例的引用。
        *   如果 Bean 实现了 EmbeddedValueResolverAware 接口，则 Spring 调用 setEmbeddedValueResolver() 方法传入当前 StringValueResolver 实例的引用。
        *   如果 Bean 实现了 ApplicationContextAware 接口，则 Spring 调用 setApplicationContext() 方法传入当前 ApplicationContext 实例的引用。
        *   ...
*   如果 BeanPostProcessor 和 Bean 关联，则 Spring 将调用该接口的预初始化方法 postProcessBeforeInitialzation() 对 Bean 进行加工操作，此处非常重要，Spring 的 AOP 就是利用它实现的。
*   如果 Bean 实现了 InitializingBean 接口，则 Spring 将调用 afterPropertiesSet() 方法。(或者有执行 @PostConstruct 注解的方法)
*   如果在配置文件中通过 **init-method** 属性指定了初始化方法，则调用该初始化方法。
*   如果 BeanPostProcessor 和 Bean 关联，则 Spring 将调用该接口的初始化方法 postProcessAfterInitialization()。此时，Bean 已经可以被应用系统使用了。
*   如果在 `<bean>` 中指定了该 Bean 的作用范围为 scope="singleton"，则将该 Bean 放入 Spring IoC 的缓存池中，将触发 Spring 对该 Bean 的生命周期管理；如果在 `<bean>` 中指定了该 Bean 的作用范围为 scope="prototype"，则将该 Bean 交给调用者，调用者管理该 Bean 的生命周期，Spring 不再管理该 Bean。
*   如果 Bean 实现了 DisposableBean 接口，则 Spring 会调用 destory() 方法将 Spring 中的 Bean 销毁；(或者有执行 @PreDestroy 注解的方法)
*   如果在配置文件中通过 **destory-method** 属性指定了 Bean 的销毁方法，则 Spring 将调用该方法对 Bean 进行销毁。

**Bean 的完整生命周期经历了各种方法调用，这些方法可以划分为以下几类**：(结合上图，需要有如下顶层思维)

*   **Bean 自身的方法**： 这个包括了 Bean 本身调用的方法和通过配置文件中`<bean>`的 init-method 和 destroy-method 指定的方法
*   **Bean 级生命周期接口方法**： 这个包括了 BeanNameAware、BeanFactoryAware、ApplicationContextAware；当然也包括 InitializingBean 和 DiposableBean 这些接口的方法（可以被 @PostConstruct 和 @PreDestroy 注解替代)
*   **容器级生命周期接口方法**： 这个包括了 InstantiationAwareBeanPostProcessor 和 BeanPostProcessor 这两个接口实现，一般称它们的实现类为 “后处理器”。
*   **工厂后处理器接口方法**： 这个包括了 AspectJWeavingEnabler, ConfigurationClassPostProcessor, CustomAutowireConfigurer 等等非常有用的工厂后处理器接口的方法。工厂后处理器也是容器级的。在应用上下文装配配置文件之后立即调用。

### [#](#spring-bean生命周期案例) Spring Bean 生命周期案例

我们通过一个例子来验证上面的整个流程

定义 Bean（这里是 User）, 并让它实现 BeanNameAware, BeanFactoryAware, ApplicationContextAware 接口和 InitializingBean, DisposableBean 接口：

```java
package tech.pdai.springframework.entity;

import lombok.ToString;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.BeanFactory;
import org.springframework.beans.factory.BeanFactoryAware;
import org.springframework.beans.factory.BeanNameAware;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;


@Slf4j
@ToString
public class User implements BeanFactoryAware, BeanNameAware, ApplicationContextAware,
        InitializingBean, DisposableBean {
    
    private String name;

    
    private int age;

    
    private BeanFactory beanFactory;

    
    private ApplicationContext applicationContext;

    
    private String beanName;

    public User() {
        log.info("execute User#new User()");
    }

    public void setName(String name) {
        log.info("execute User#setName({})", name);
        this.name = name;
    }

    public void setAge(int age) {
        log.info("execute User#setAge({})", age);
        this.age = age;
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        log.info("execute BeanFactoryAware#setBeanFactory");
        this.beanFactory = beanFactory;
    }

    @Override
    public void setBeanName(String s) {
        log.info("execute BeanNameAware#setBeanName");
        this.beanName = s;
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        log.info("execute ApplicationContextAware#setApplicationContext");
        this.applicationContext = applicationContext;
    }

    @Override
    public void destroy() throws Exception {
        log.info("execute DisposableBean#destroy");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        log.info("execute InitializingBean#afterPropertiesSet");
    }


    public void doInit() {
        log.info("execute User#doInit");
    }

    public void doDestroy() {
        log.info("execute User#doDestroy");
    }

}


```

定义 BeanFactoryPostProcessor 的实现类

```java
@Slf4j
@Component
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
        log.info("execute BeanFactoryPostProcessor#postProcessBeanFactory");
    }
}


```

定义 InstantiationAwareBeanPostProcessor 的实现类

```java
@Slf4j
@Component
public class MyInstantiationAwareBeanPostProcessor implements InstantiationAwareBeanPostProcessor {
    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        log.info("execute InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation for {}", beanName);
        return InstantiationAwareBeanPostProcessor.super.postProcessBeforeInstantiation(beanClass, beanName);
    }

    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        log.info("execute InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation for {}", beanName);
        return InstantiationAwareBeanPostProcessor.super.postProcessAfterInstantiation(bean, beanName);
    }

    @Override
    public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) throws BeansException {
        log.info("execute InstantiationAwareBeanPostProcessor#postProcessProperties for {}", beanName);
        return InstantiationAwareBeanPostProcessor.super.postProcessProperties(pvs, bean, beanName);
    }
}


```

定义 BeanPostProcessor 的实现类

```java
@Slf4j
@Component
public class MyBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        log.info("execute BeanPostProcessor#postProcessBeforeInitialization for {}", beanName);
        return BeanPostProcessor.super.postProcessBeforeInitialization(bean, beanName);
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        log.info("execute BeanPostProcessor#postProcessAfterInitialization for {}", beanName);
        return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
    }
}


```

通过 Java 配置方式初始化 Bean

```java
@Configuration
public class BeansConfig {

    @Bean(name = "user", initMethod = "doInit", destroyMethod = "doDestroy")
    public User create() {
        User user = new User();
        user.setName("pdai");
        user.setAge(18);
        return user;
    }
}


```

测试的主方法

```java
@Slf4j
public class App {

    
    public static void main(String[] args) {
        log.info("Init application context");
        
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(
                "tech.pdai.springframework");

        
        User user = (User) context.getBean("user");

        
        log.info(user.toString());

        log.info("Shutdown application context");
        context.registerShutdownHook();
    }
}


```

输出结果（剔除无关输出）：

```java
12:44:42.547 [main] INFO tech.pdai.springframework.App - Init application context
...
12:44:43.134 [main] INFO tech.pdai.springframework.processor.MyBeanFactoryPostProcessor - execute BeanFactoryPostProcessor#postProcessBeanFactory
...
12:44:43.216 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'user'
12:44:43.216 [main] INFO tech.pdai.springframework.processor.MyInstantiationAwareBeanPostProcessor - execute InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation for user
12:44:43.236 [main] INFO tech.pdai.springframework.entity.User - execute User#new User()
12:44:43.237 [main] INFO tech.pdai.springframework.entity.User - execute User#setName(pdai)
12:44:43.237 [main] INFO tech.pdai.springframework.entity.User - execute User#setAge(18)
12:44:43.237 [main] INFO tech.pdai.springframework.processor.MyInstantiationAwareBeanPostProcessor - execute InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation for user
12:44:43.237 [main] INFO tech.pdai.springframework.processor.MyInstantiationAwareBeanPostProcessor - execute InstantiationAwareBeanPostProcessor#postProcessProperties for user
12:44:43.242 [main] INFO tech.pdai.springframework.entity.User - execute BeanNameAware#setBeanName
12:44:43.242 [main] INFO tech.pdai.springframework.entity.User - execute BeanFactoryAware#setBeanFactory
12:44:43.242 [main] INFO tech.pdai.springframework.entity.User - execute ApplicationContextAware#setApplicationContext
12:44:43.242 [main] INFO tech.pdai.springframework.processor.MyBeanPostProcessor - execute BeanPostProcessor#postProcessBeforeInitialization for user
12:44:43.242 [main] INFO tech.pdai.springframework.entity.User - execute InitializingBean#afterPropertiesSet
12:44:43.243 [main] INFO tech.pdai.springframework.entity.User - execute User#doInit
12:44:43.243 [main] INFO tech.pdai.springframework.processor.MyBeanPostProcessor - execute BeanPostProcessor#postProcessAfterInitialization for user
12:44:43.270 [main] INFO tech.pdai.springframework.App - User(name=pdai, age=18)
12:44:43.270 [main] INFO tech.pdai.springframework.App - Shutdown application context
12:44:43.276 [SpringContextShutdownHook] INFO tech.pdai.springframework.entity.User - execute DisposableBean#destroy
12:44:43.276 [SpringContextShutdownHook] INFO tech.pdai.springframework.entity.User - execute User#doDestroy


```

### [#](#spring-bean生命周期源码) Spring Bean 生命周期源码

## [#](#参考文章) 参考文章

https://juejin.cn/post/6844903843596107790

https://www.zhihu.com/question/438247718/answer/1730527725