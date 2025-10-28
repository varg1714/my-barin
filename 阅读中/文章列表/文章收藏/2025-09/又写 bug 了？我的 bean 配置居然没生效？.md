---
source: https://mp.weixin.qq.com/s/rWorrPuxZaneGsjoGMe-Pw
create: 2024-07-31 10:52
read: true
knowledge: true
knowledge-date: 2025-10-27
tags:
  - Spring
  - 框架使用
summary: "[[Spring 中不同依赖注入的区别]]"
---

阿里妹导读

最近在某个项目的开发过程中，遇到了一个 bean 注入不生效的问题，本文主要针对该问题进行展开，欢迎大家共同探讨。

一、背景

该项目涉及到两个应用，其中一个应用 A 需要给另一个应用 B 打一个胖客户端（Fat Client），该胖客户端的代码在两个应用中都有调用。胖客户端内需要配置一个 tair bean，但是该 bean 配置在 A 中生效，但是在 B 中却没有生效。

![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicXVHCbAgl692Siap9rCib6ztl1SjssonsliaVCxiao5kbqw7BM8ibxkhklmK0zBJp8gTUCzhENcm5eZeA/640?wx_fmt=png&from=appmsg)

```java
@Configuration
@Slf4j(topic = "config")
public class XxxTairConfig {
    @Value("${spring.tmg.xxx.tair.username:默认值}")
    private String username;
    @Value("${spring.tmg.xxx.tair.namespace:默认值}")
    private Integer namespace;
    @Bean(initMethod = "init")
    public TairManager tmgXxxTairManager() {
        MultiClusterTairManager tairManager = new MultiClusterTairManager();
        tairManager.setUserName(username);
        tairManager.setDynamicConfig(true);
        return tairManager;
    }
    @Bean
    public TairAccessor tmgXxxTairAccessor(@Qualifier("tmgXxxTairManager") TairManager tmgXXXTairManager) {
        TairAccessorImpl tairAccessor = new TairAccessorImpl();
        tairAccessor.setTairManager(tmgXxxTairManager);
        tairAccessor.setNamespace(namespace);
        return tairAccessor;
    }
}
```

还记得 Spring 是怎么配置 bean 的吗？在 Spring 中总体来看可以通过三种方式来配置对象:

*   使用 XML 文件配置
    
*   使用注解来配置
    
*   使用 JavaConfig 来配置

**注解自动装配**

@Resource 和 @Autowired

@Autowired: 用于构造器、方法、参数或字段上，表明需要自动注入一个 Bean。Spring 会自动装配匹配的 Bean。

@Qualifier: 与 @Autowired 一起使用时，指定要注入的 Bean 的名称，以避免与其他 Bean 的混淆。

@Resource: 来自 JDK，类似于 @Autowired，但默认是按名称装配，也可以混合使用。

@Inject: 来自 javax.inject 包，类似于 @Autowired，属于 JSR-330 标准的一部分。

@Resource 和 @Autowired 注解有什么区别呢？

@Autowired 是 Spring 框架提供的注解，主要用于根据类型自动装配依赖项。

行为和特性：

1.  按类型装配：默认情况下，@Autowired 按类型自动装配 Bean。
    
2.  可选依赖：如果你的依赖是可选的，可以使用 required=false 设置：
    
3.  构造器、方法或字段：可以用在构造器，属性字段或 Setter 方法上。
    
4.  结合 @Qualifier：可以和 @Qualifier 结合使用以实现按名称装配。
    
5.  作为 Spring 特有的注解，它更深度地集成在 Spring 的生态系统中，更适合与其他 Spring 注解一起使用。

@Resource 是 JDK 提供的注解，属于 Java 依赖注入规范（JSR-250）的一部分。

行为和特性：

1.  按名称装配：默认情况下，@Resource 按名称装配。如果没有匹配到名称，再按类型装配。
    
2.  不支持 required 属性：与 @Autowired 不同，@Resource 不支持 required 属性。
    
3.  可以用于字段和 Setter 方法：虽然也可以用于构造器，但不常见。通常用在字段或 Setter 方法上。
    
4.  由于是 Java EE 规范的一部分，它可以与其他 Java EE 注解（如 @PostConstruct 和 @PreDestroy）更好地配合使用。

其他注解

*   @Component: 标注一个类为 Spring 管理的组件。类似的注解还有：
    
*   @Service: 表示服务层组件。
    
*   @Repository: 表示 DAO（数据访问层）组件。
    
*   @Controller: 表示 Spring MVC 控制器组件。
    
*   @Primary: 当一个类型有多个 Bean 时，在不使用 @Qualifier 的情况下，Spring 会优先选择标注了 @Primary 的 Bean。
    
*   @Scope: 用于指定 Bean 的作用域，如 singleton、prototype。

**Java 配置类**

那么怎么通过 java 代码来配置 bean 呢？

*   编写一个 java 类，使用 @Configuration 修饰该类
    
*   被 @Configuration 修饰的类就是配置类

以本次用到的 bean 配置为例：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="
  http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
  http://www.springframework.org/schema/context
  http://www.springframework.org/schema/context/spring-context.xsd">

  <bean id="tmgXxxTairManager" class="com.taobao.tair.impl.mc.MultiClusterTairManager" init-method="init">
    <property name="userName">
      <value>tair用户名参数</value>
    </property>
    <property name="timeout">
      <value>50</value>
    </property>
  </bean>
  <bean id="tmgXxxTairAccessor" class="com.alibaba.tmallg.gcommon.tair.TairAccessorImpl">
    <property name="tairManager" ref="tmgXxxTairManager"/>
    <property name="namespace" value="tair namespace参数"/>
  </bean>

</beans>
```

**xml 配置**

还是以本次用到的 bean 配置为例：

```java
@Configuration
@Slf4j(topic = "config")
public class XxxTairConfig {
    @Value("${spring.tmg.xxx.tair.username:默认值}")
    private String username;
    @Value("${spring.tmg.xxx.tair.namespace:默认值}")
    private Integer namespace;
    @Bean(initMethod = "init")
    public TairManager tmgXxxTairManager() {
        MultiClusterTairManager tairManager = new MultiClusterTairManager();
        tairManager.setUserName(username);
        tairManager.setDynamicConfig(true);
        return tairManager;
    }
    @Bean
    public TairAccessor tmgXxxTairAccessor(TairManager tmgXxxTairManager) {
        TairAccessorImpl tairAccessor = new TairAccessorImpl();
        tairAccessor.setTairManager(tmgXxxTairManager);
        tairAccessor.setNamespace(namespace);
        return tairAccessor;
    }
}
```

为什么我的 bean 配置没生效？

最早，在胖客户端中，是通过 Java 配置类配置的 tair 的 bean。最早的代码如下：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="
  http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
  http://www.springframework.org/schema/context
  http://www.springframework.org/schema/context/spring-context.xsd">

  <bean id="tmgXxxTairManager" class="com.taobao.tair.impl.mc.MultiClusterTairManager" init-method="init">
    <property name="userName">
      <value>tair用户名参数</value>
    </property>
    <property name="timeout">
      <value>50</value>
    </property>
  </bean>
  <bean id="tmgXxxTairAccessor" class="com.alibaba.tmallg.gcommon.tair.TairAccessorImpl">
    <property name="tairManager" ref="tmgXxxTairManager"/>
    <property name="namespace" value="tair namespace参数"/>
  </bean>

</beans>
```

在应用 A 中，该配置没有问题，tair 也可以正常查询。

但是部署到应用 B 后，奇怪的问题出现了：同样的 key，应用 A 可以查到，tair 控制台也可以查到，但是应用 B 死活查不出来。

![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicXVHCbAgl692Siap9rCib6ztHt3VDtTpIicGJD1I0gqj4URCRiax9mFMvfuP35c24ziaibXKOzia0AzxOMA/640?wx_fmt=png&from=appmsg)

四、排查

**排查过程**

难道是因为 username 的 @Value 默认值没生效？因为 debug 可以看到 namespace 属性，符合预期，但是 username 属性看不到，我最早怀疑是 username 没配置成功。但 namespace 是生效的，理论上不应该部分不生效呀？抱着试试看的态度，遂尝试将 username 直接写死成目标值。重新部署后，依然不行，看来不是 @Value 的锅。

中间我尝试将 java 配置 bean 的方式，改成用 xml 配置，这样是生效的，可以解决问题。

但是为什么 java 配置 bean 的方式不生效呢？

还原后，继续 debug，感觉依旧有些摸不着头脑。后来请团队同学帮忙看了下，一开始也觉得奇怪，tair 各个单元都是有数据的，和单元化也没关系，后来点开 tairManager 属性值看到 mdbcomm，问了句，我们的 tair 用的是 mdb 吗？最后打开控制台一看，是 ldb……

![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicXVHCbAgl692Siap9rCib6ztnc9oczKVy6amLmHW9icVSJ38iaV9tTBYNkcibEB43jS6cvB6iaR46s9vsg/640?wx_fmt=png&from=appmsg)

**Spring 依赖注入优先级**

原来是 TairManager 注入的不是我们在配置类中配置的 tmgXxxTairManager？！

可是 tmgXxxTairManager 也是一个 bean，不应该也是唯一的吗，难道这里不是按照 bean 名字注入的吗

那这里依赖注入的优先级是什么呢？

@Resource

在使用 @Resource 注解进行依赖注入时，优先级规则如下：

**明确指定名称：**

*   如果通过 @Resource(name="beanName") 明确指定了 Bean 的名称，那么 Spring 会首先按照名称匹配进行注入。
    
*   在这种情况下，@Primary 注解不会影响注入结果。

**按字段或属性名称匹配：**

*   如果没有通过 name 属性指定 Bean 的名称，Spring 会尝试按照字段或属性的名称进行匹配。
    
*   在这种情况下，@Primary 注解也不会影响注入结果。

**按类型匹配：**

*   如果按名称匹配失败（包括明确指定名称和按字段名称匹配都没有找到合适的 Bean），Spring 会按类型匹配。
    
*   在这种情况下，如果存在多个同类型的 Bean，则 @Primary 注解会起作用，标记为 @Primary 的 Bean 将被优先注入。

@Autowired

**按类型匹配：**

*   Spring 首先通过类型匹配找到所有符合要求的候选 Bean。如果只有一个候选 Bean，那么该 Bean 会被注入。

**按名称匹配结合 @Qualifier：**

*   如果有多个同类型的 Bean，可以使用 @Qualifier 注解来指定具体的 Bean。
    
*   @Qualifier 的值必须与一个候选 Bean 的名称匹配，匹配成功的 Bean 会被注入。

**使用 @Primary：**

*   如果仍存在多个符合要求的 Bean，并且其中一个 Bean 标记了 @Primary，Spring 会优先选择标记了 @Primary 的 Bean 进行注入。

**按名称匹配字段或属性名称：**

*   在没有使用 @Qualifier 时，如果存在多个候选 Bean，Spring 会尝试通过字段或属性名称进行匹配。
    
*   如果找到名称匹配的 Bean，则该 Bean 会被注入。

**NoUniqueBeanDefinitionException：**

*   如果存在多个候选 Bean，但没有使用 @Qualifier 指定名称，且没有标记 @Primary，会抛出 NoUniqueBeanDefinitionException，表明有多个 Bean 类型匹配但无法确定注入哪个。
*   @Bean 方法中的参数。

Spring 框架在处理 @Bean 方法中的参数时，默认的行为与 @Autowired 注解的工作方式是一致的。

例如本文中涉及到的 tmgXxxTairManager 参数，注入的并不一定是上面定义的 tmgXxxTairManager bean。

```java
@Configuration
@Slf4j(topic = "config")
public class XxxTairConfig {

    @Value("${spring.tmg.xxx.tair.username:默认值}")
    private String username;

    @Value("${spring.tmg.xxx.tair.namespace:默认值}")
    private Integer namespace;

    @Bean(initMethod = "init")
    public TairManager tmgXxxTairManager() {
        MultiClusterTairManager tairManager = new MultiClusterTairManager();
        tairManager.setUserName(username);
        tairManager.setDynamicConfig(true);
        return tairManager;
    }

    @Bean
    public TairAccessor tmgXxxTairAccessor(@Qualifier("tmgXxxTairManager") TairManager tmgXxxTairManager) {
        TairAccessorImpl tairAccessor = new TairAccessorImpl();
        tairAccessor.setTairManager(tmgXxxTairManager);
        tairAccessor.setNamespace(namespace);
        return tairAccessor;
    }
}
```

五、解决方案

**xml 配置 bean**

在使用 XML 配置 bean 时，ref 元素通常是用来引用其他已经定义的 Bean，并且是通过 Bean 的 id 来进行引用和注入的。这种方法使得在 XML 配置的 Spring 应用程序中可以灵活地管理和注入依赖。

这也解释了为什么中间改成用 xml 配置是没问题的。

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="
  http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
  http://www.springframework.org/schema/context
  http://www.springframework.org/schema/context/spring-context.xsd">
  <bean init-method="init">
    <property name="userName">
      <value>tair用户名参数</value>
    </property>
    <property name="timeout">
      <value>50</value>
    </property>
  </bean>
  <bean>
    <property name="tairManager" ref="tmgXxxTairManager"/>
    <property name="namespace" value="tair namespace参数"/>
  </bean>
</beans>
```

 **java 配置 bean**

通过 @Qualifier 指定 bean，避免受 @Primary 等因素的影响。

```java
@Configuration
@Slf4j(topic = "config")
public class XxxTairConfig {
    @Value("${spring.tmg.xxx.tair.username:默认值}")
    private String username;
    @Value("${spring.tmg.xxx.tair.namespace:默认值}")
    private Integer namespace;
    @Bean(initMethod = "init")
    public TairManager tmgXxxTairManager() {
        MultiClusterTairManager tairManager = new MultiClusterTairManager();
        tairManager.setUserName(username);
        tairManager.setDynamicConfig(true);
        return tairManager;
    }
    @Bean
    public TairAccessor tmgXxxTairAccessor(@Qualifier("tmgXxxTairManager") TairManager tmgXxxTairManager) {
        TairAccessorImpl tairAccessor = new TairAccessorImpl();
        tairAccessor.setTairManager(tmgXxxTairManager);
        tairAccessor.setNamespace(namespace);
        return tairAccessor;
    }
}
```

六、结语

在使用配置类配置 bean 时，@Bean 方法的参数，或者用 @Autowired 配置 bean 时，最好使用 @Qualifier 指定注入的 bean，避免注入的 bean 不符合预期。@Resource 则通常不存在这种烦恼。

```
团队介绍
```

我们是天猫国际前台技术团队，致力于通过技术能力解决人、货、场之间的高效匹配问题，持续为消费者打造优秀的进口商品购物体验。我们始终关注用户的真实需求和反馈，不断探索和应用新技术，和各个团队紧密合作，为用户提供更加智能、便捷的购物体验，将天猫国际打造成为全球消费者信赖的跨境购物平台。