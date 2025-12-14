---
source: https://mp.weixin.qq.com/s/jknTjxJt1YHHH9bjMbGlrQ
create: 2025-11-28 16:38
read: false
knowledge: false
---
FSAC 未来超级架构师

架构师总动员  
实现架构转型，再无中年危机

## 尼恩说在前面

在 45 岁老架构师 尼恩的**读者交流群** (50+) 中，最近有小伙伴拿到了一线互联网企业如得物、阿里、滴滴、极兔、有赞、希音、百度、网易、美团、蚂蚁、得物的面试资格，遇到很多很重要的相关面试题：

1.  MyBatis 插件是基于什么机制实现的？
    
2.  MyBatis 插件中的责任链模式是如何实现的？与传统的责任链模式有何不同？
    
3.  请解释 MyBatis 插件机制中,  如何体现 OCP 原则 和 SRP 原则？
    

**MyBatis 底层原理是常见的面试题。**

**最近有小伙伴在面 蚂蚁，又被问到了，可以说是逢面必问。**

小伙伴没有系统的去梳理和总结，所以支支吾吾的说了几句，面试官不满意，面试挂了。

这里，尼恩给大家做一下系统化、体系化的梳理，使得大家内力猛增.

按照尼恩的思路做 暴击，  可以充分展示一下大家雄厚的 “技术肌肉”，让面试官爱到 “不能自已、口水直流”，然后实现”offer 直提”。

最终，机会爆表，实现”offer 自由” 。

当然，这道面试题，以及参考答案，也会收入咱们的 《尼恩 Java 面试宝典 PDF》V175 版本，供后面的小伙伴参考，提升大家的 3 高 架构、设计、开发水平。

《尼恩 架构笔记》《尼恩高并发三部曲》《尼恩 Java 面试宝典》的 PDF，请到文末公号【技术自由圈】获取

本文作者：

*   第一作者   老架构师 肖恩（肖恩  是尼恩团队 高级架构师，负责写此文的第一稿，初稿 ）
    
*   第二作者    老架构师 尼恩    （**45 岁老架构师， 负责  提升此文的 技术高度，让大家有一种  俯视 技术、俯瞰技术、 技术自由  的感觉**）
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WNhcvNOWZnMeNbpLN3wdaNEibBWGGfUiaf7pianz82A2bfJia89wXCKcmJqyHaeZCSbc4nrfEW4lKJ1SA/640?from=appmsg&watermark=1#imgIndex=0)

## 一： MyBatis  插件的应用 场景

MyBatis  插件的核心价值是 **分离通用逻辑与业务逻辑**，解决以下高频场景痛点：

*   **SQL 监控与诊断**：
    

统计 SQL 执行耗时、记录 SQL 语句及参数、检测慢 SQL 并报警；

*   **通用功能封装**：
    

分页逻辑（如 PageHelper 插件）、多租户数据隔离（自动添加租户 ID 条件）、数据权限控制（基于用户角色过滤数据）；

*   **数据安全处理**：
    

敏感字段加密（如密码、手机号存入数据库时加密）、敏感字段解密（查询后自动解密展示）；

*   **日志增强**：
    

定制化 SQL 日志格式（如添加请求 ID、用户 ID 便于链路追踪）；

*   **数据自动填充**：
    

创建时间（createTime）、更新时间（updateTime）、创建人（creator）等公共字段的自动赋值；

*   **SQL 改写**：
    

动态添加查询条件、替换表名（如分表场景）、优化 SQL 语句（如添加索引提示）。

## 二： MyBatis 插件机制 基础使用：  从 0 到 1 入门

MyBatis 插件 不是 Maven 插件，也不是 IDEA 插件，而是 MyBatis **运行时动态代理链**的一环。

MyBatis 插件 是基于 JDK 动态代理实现的一环，嵌入在 MyBatis 核心执行流程中的 **运行时拦截链**。

通过这一机制，开发者可以在不修改源码的前提下，动态地介入 SQL 执行过程，实现诸如性能监控、SQL 改写、数据加密等横切关注点功能。

MyBatis 插件 与 Spring AOP 中的 “环绕通知” 非常相似，但在 MyBatis 的上下文中更加轻量且专注。

### 2.1  Interceptor 定义

MyBatis 插件 官方文档英文叫 **Interceptor**，中文文档常译 “拦截器”，但社区口语也喊 “插件”。

MyBatis 插件 的实现类，必须实现 org.apache.ibatis.plugin.Interceptor 接口。

Interceptor 接口  核心方法包括：

*   intercept(Invocation invocation)：拦截逻辑的核心实现方法；可以在此添加前置处理、后置处理，甚至完全替换原行为。
    
*   plugin(Object target)：生成代理对象（默认调用 Plugin.wrap() 即可）； 完成  JDK 动态代理包装。
    
*   setProperties(Properties properties)：读取插件配置的属性（如分页 大小、阈值、加密密钥）。这个方法, 在插件初始化阶段被调用一次， 做配置解析和初始化工作。
    

### 2.2 MyBatis  插件  “插” 在 哪儿

MyBatis  四大金刚接口的全部**公有方法**均可被拦截：

<table><thead><tr><td><span><strong><span leaf="">接口</span></strong></span></td><td><span><strong><span leaf="">典型可拦截方法</span></strong></span></td><td><span><strong><span leaf="">场景举例</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">Executor</span></span></section></td><td><section><span><span leaf="">update, query, commit, rollback, close</span></span></section></td><td><section><span><span leaf="">慢 SQL、读写分离、多租户</span></span></section></td></tr><tr><td><section><span><span leaf="">StatementHandler</span></span></section></td><td><section><span><span leaf="">prepare, parameterize, batch, update, query</span></span></section></td><td><section><span><span leaf="">分页改写 SQL、加 Hint</span></span></section></td></tr><tr><td><section><span><span leaf="">ParameterHandler</span></span></section></td><td><section><span><span leaf="">setParameters</span></span></section></td><td><section><span><span leaf="">统一加密、脱敏</span></span></section></td></tr><tr><td><section><span><span leaf="">ResultSetHandler</span></span></section></td><td><section><span><span leaf="">handleResultSets, handleOutputParameters</span></span></section></td><td><section><span><span leaf="">解密、字典翻译、驼峰下划线</span></span></section></td></tr></tbody></table>

MyBatis  四大金刚接口  介绍如下：

*   Executor 是 SQL 执行的总调度者，负责事务管理和缓存协调；
    
*   StatementHandler 负责真正与 JDBC Statement 打交道，准备预编译语句；
    
*   ParameterHandler 处理参数绑定，将 Java 对象映射到 PreparedStatement 的占位符；
    
*   ResultSetHandler 则负责结果集的解析与映射。
    

MyBatis  四大金刚接口  贯穿了完整的 SQL 生命周期，因此它们成为插件最理想的切入点。例如：

*   如果 要做通用分页，最佳位置是拦截 Executor.query；
    
*   如果 想统一处理敏感字段加密，则应在 ParameterHandler.setParameters 阶段介入。
    

### 2.3 自定义  LimitPlugin 插件 示例

下面是一个典型的 MyBatis 插件实现，用于在 SQL 准备阶段，  插入日志输出 ：

```
@Intercepts(
  @Signature(type = StatementHandler.class,
             method = "prepare",
             args = {Connection.class, Integer.class}))
public class LimitPlugin implements Interceptor {
  public Object intercept(Invocation inv) throws Throwable {
    System.out.println("→ 准备阶段被拦截");
    return inv.proceed();          // 必须调用，否则 SQL 不会继续
  }
    ......
}


```

在这个例子中， 通过 @Intercepts 注解声明了要拦截的目标：StatementHandler.prepare(Connection, Integer) 方法。

当 MyBatis 执行任何 Mapper 查询时，只要经过该方法，就会触发 LimitPlugin.intercept() 回调。

**上面的 注解 说明**

MyBatis 使用 @Intercepts 和 @Signature 注解来声明插件的拦截点。

@Intercepts 和 @Signature 注解 其匹配机制极为严格，依赖于 **类型、方法名、参数列表三者完全一致**。

```
@Intercepts({
  @Signature(
    type = Executor.class,           // 四大核心接口之一
    method = "query",                // 方法名（区分大小写）
    args = {
      MappedStatement.class, 
      Object.class, 
      RowBounds.class, 
      ResultHandler.class
    } // 参数类型顺序必须完全一致
  )
})


```

常见错误：  如 args 漏写 RowBounds，  启动不报错，但永远匹配不到。

### 自定义 插件 注册：

为了让插件生效，需要将其注册到 MyBatis 的配置体系中。

最常见的方式是在 mybatis-config.xml 中使用  标签：

```
<plugins>
  <plugin interceptor="demo.LimitPlugin"/>
</plugins>


```

运行任意 Mapper，控制台即可看到 “→ 准备阶段被拦截”。

### 2.5 自定义 插件  配置方式大全

插件需注册到 MyBatis 中才能生效，支持 3 种注册方式：

**传统 XML 配置方式**（适用于纯 MyBatis 或早期 Spring 集成项目）：

```
<bean class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="plugins">
    <array><bean class="demo.LimitPlugin"/></array>
  </property>
</bean>


```

这种方式通过 Spring Bean 容器管理插件实例，适合已有 XML 配置体系的老项目迁移。

**Spring Boot 自动配置注册**（推荐用于 Spring Boot 项目）：

```
@Configuration
public class MyBatisConfig {
  @Bean
  public LimitPlugin limitPlugin() {   // 方法名即 beanName
    return new LimitPlugin();
  }
}


```

利用 Spring 的自动装配机制，只需将插件声明为 Bean，MyBatis-Spring-Boot-Starter 会在启动时自动将其加入 Configuration.plugins 列表。

这是目前最简洁、最符合现代开发习惯的方式。

**Java 代码配置注册**（适用于无 XML 的轻量级应用或测试场景）：

```
new SqlSessionFactoryBuilder()
    .build(configuration)
    .getConfiguration()
    .addInterceptor(new LimitPlugin());


```

这种方式完全绕开配置文件，通过编程方式手动添加插件，灵活性最高，常用于单元测试或嵌入式场景。

三种方式本质相同，最终都是将 Interceptor 实例添加到 Configuration 的插件列表中。

选择哪种方式取决于项目的整体架构和技术演进路线。

### 2.6 自定义 插件  生命周期

自定义 插件  生命周期 的生与死：

*   实例化：MyBatis 启动时一次性创建 **单例**，随后一直复用。MyBatis 插件在容器启动阶段完成实例化，且**全局唯一、单例存在**。 无论有多少个 Mapper 或 SQL 执行请求，同一个插件类在整个 JVM 中只会被创建一次。
    
*   销毁：与 SqlSessionFactory 生命周期相同，几乎无需关心。
    

### 2.7 常见的 MyBatis 插件

<table><thead><tr><td><span><strong><span leaf="">名称</span></strong></span></td><td><span><strong><span leaf="">作用</span></strong></span></td><td><span><strong><span leaf="">备注</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">MyBatis-Plus 分页插件</span></span></section></td><td><section><span><span leaf="">物理分页</span></span></section></td><td><section><span><span leaf="">基于 Executor.query</span></span></section></td></tr><tr><td><section><span><span leaf="">PageHelper</span></span></section></td><td><section><span><span leaf="">物理分页</span></span></section></td><td><section><span><span leaf="">基于 Executor.query</span></span></section></td></tr><tr><td><section><span><span leaf="">MyBatis-Flex 审计插件</span></span></section></td><td><section><span><span leaf="">自动填充时间、操作人</span></span></section></td><td><section><span><span leaf="">基于 Executor.update</span></span></section></td></tr><tr><td><section><span><span leaf="">shardingsphere-jdbc</span></span></section></td><td><section><span><span leaf="">分库分表</span></span></section></td><td><section><span><span leaf="">包装 Executor</span></span></section></td></tr><tr><td><section><span><span leaf="">mybatis-encrypt</span></span></section></td><td><section><span><span leaf="">字段加解密</span></span></section></td><td><section><span><span leaf="">基于 ParameterHandler+ResultSetHandler</span></span></section></td></tr></tbody></table>

这些插件的背后，其实都是对 Interceptor 接口的巧妙运用。例如：

*   分页插件在 Executor.query 阶段分析 SQL 并注入 LIMIT；
    
*   审计插件在 Executor.update 时检查实体字段并自动赋值；
    
*   加密插件则分别在参数设置和结果映射两个阶段完成加解密闭环。
    

### 注意事项与最佳实践

*   **拦截点选择原则**：按需选择最细粒度的拦截点（如仅需处理参数时，优先拦截 ParameterHandler. setParameters，而非 Executor.query），减少性能损耗；
    
*   **插件执行顺序**：多个插件的执行顺序与注册顺序一致（先注册的先执行），若需调整顺序，可通过配置文件或代码注册顺序调整；
    
*   **无状态设计**：插件应设计为无状态（不存储局部变量），避免多线程环境下的数据安全问题； 避免在插件中维护局部状态，所有数据应通过 invocation.getArgs() 或上下文传递。
    
*   **不吞噬异常**：拦截逻辑中捕获异常后,  需重新抛出，确保业务层能感知异常并处理；
    
*   **避免循环依赖**：多个插件间不可相互依赖，否则会导致调用链死循环。例如，A 插件调用 B，B 又反过来触发 A 的逻辑，极易导致栈溢出或死循环。设计时应保持插件职责单一、边界清晰。
    

## 三：MyBatis 插件 核心原理  ：动态代理 + 责任链

核心工作机制：动态代理 + 责任链

MyBatis 插件的底层依赖两大核心机制，二者协同实现 “拦截 - 传递” 的逻辑：

*   **JDK 动态代理**：MyBatis 插件仅支持 JDK 动态代理（不支持 CGLIB），因为四大核心组件均为接口 ，动态代理需基于接口生成代理对象；
    
*   **责任链模式**：多个插件按顺序形成链式结构，每个插件处理后将调用传递给下一个插件，最终执行原始方法。
    

## 3.1 动态代理 + 责任链 是如何配合的？

传统链表式责任链，通过显式 next 指针串联处理器。

而  MyBatis 插件  不同于传统链表式责任链 ，MyBatis 采用了一种更为巧妙的方式 —— **代理嵌套 + 反射调用**，形成一种 “洋葱模型” 的责任链 结构。

洋葱模型” 的责任链 结构 ：  每个插件并不直接持有下一个插件的引用，而是通过层层包裹代理对象的方式，将控制权逐层传递下去。

### 启动期：责任链 加载阶段：

启动期，责任链的  Interceptor    加载到 list。

一切始于 MyBatis 容器的初始化阶段。当 XMLConfigBuilder 解析`<plugins>` 配置标签时，便拉开了插件链构建的序幕。

此时，每一个`<plugin>` 标签都会触发一次拦截器实例化流程。

MyBatis 不仅会通过无参构造函数创建 Interceptor 实例，还会自动调用其 setProperties() 方法注入配置参数，完成个性化设置。

所有实例化后的拦截器，最终都被统一添加到 Configuration 对象中的 interceptorChain 成员变量里 ——  一个 `ArrayList<Interceptor>`。

```
sequenceDiagram
    participant XMLParser as XMLConfigBuilder
    participant Config as Configuration
    participant IC as InterceptorChain
    participant Interceptor1
    participant Interceptor2
    XMLParser->>Config: 解析<plugins>标签
    Config->>Interceptor1: 实例化拦截器1
    Config->>Interceptor1: 调用setProperties()
    Config->>IC: interceptorChain.addInterceptor(interceptor1)
    Config->>Interceptor2: 实例化拦截器2  
    Config->>Interceptor2: 调用setProperties()
    Config->>IC: interceptorChain.addInterceptor(interceptor2)
    Note over IC: interceptorChain = [interceptor1, interceptor2]


```

InterceptorChain 就加入了所有 的 Interceptor  到自己 的 interceptors   。

```
public class InterceptorChain {
    // 核心：存储所有已注册的拦截器
    private final List<Interceptor> interceptors = new ArrayList<>();
    }


```

四大对象创建入口速查

<table><thead><tr><td><span><strong><span leaf="">对象</span></strong></span></td><td><span><strong><span leaf="">创建点</span></strong></span></td><td><span><strong><span leaf="">立即被 pluginAll</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">Executor</span></span></section></td><td><section><span><span leaf="">Configuration.newExecutor()</span></span></section></td><td><section><span><span leaf="">✔</span></span></section></td></tr><tr><td><section><span><span leaf="">StatementHandler</span></span></section></td><td><section><span><span leaf="">Configuration.newStatementHandler()</span></span></section></td><td><section><span><span leaf="">✔</span></span></section></td></tr><tr><td><section><span><span leaf="">ParameterHandler</span></span></section></td><td><section><span><span leaf="">Configuration.newParameterHandler()</span></span></section></td><td><section><span><span leaf="">✔</span></span></section></td></tr><tr><td><section><span><span leaf="">ResultSetHandler</span></span></section></td><td><section><span><span leaf="">Configuration.newResultSetHandler()</span></span></section></td><td><section><span><span leaf="">✔</span></span></section></td></tr></tbody></table>

### 启动期， 责任链  装配 阶段：

基于 动态代理  ，实现  责任链 的 洋葱模式一层一层的 装配  。

当 MyBatis 开始创建四大核心组件（如 Executor）时，真正的魔法开始了。

在 Configuration.newXXX() 方法中，每创建一个核心对象，都会立即调用 interceptorChain.pluginAll(target) 方法，对该对象进行 “全链路包装”。这个过程就像给一颗洋葱一层层地 裹上外皮。

```
public class Configuration {
    // 1. Executor创建入口：接收事务对象，返回代理后的Executor
    public Executor newExecutor(Transaction transaction) {
    ....
    }
     // 2. StatementHandler创建入口：接收Executor、MappedStatement等上下文
   public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement, 
     ....
    }   
   // 3. ParameterHandler创建入口：接收SQL映射信息和参数
    public ParameterHandler newParameterHandler(MappedStatement mappedStatement,  Object parameterObject, BoundSql boundSql) {
       ....
    }     


```

后续创建四大对象时，统一走 InterceptorChain.pluginAll(target)：

```
public Object pluginAll(Object target) {
  for (Interceptor interceptor : interceptors) {
    target = interceptor.plugin(target); // 一层洋葱皮
  }
  return target;
}


```

尼恩提示：这 上面的那一层包裹很重要。上面的代码使用了  默认 interceptor.plugin(target)  。

interceptor.plugin()    这个方法 是关键一环， 这里 返回 Plugin.wrap(target, this)，  而 wrap 里边使用了 **JDK 动态代理**。

```
public interface Interceptor {
    Object intercept(Invocation invocation) throws Throwable;//核心拦截逻辑 - 必须由插件实现
    /** * 默认的插件包装方法 - 关键默认实现！ 90%的情况下使用这个默认实现即可 /
    default Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }
  }


```

启动期的   责任链 + 动态代理 结合 应用阶段，  代理包装 的过程如下：

```
sequenceDiagram
    participant Config as Configuration
    participant IC as InterceptorChain
    participant I1 as Interceptor1
    participant I2 as Interceptor2
    participant Plugin as Plugin类
    participant Target as 原始对象
    Config->>IC: 调用pluginAll(原始对象)
    IC->>I1: 调用interceptor1.plugin(原始对象)
    I1->>Plugin: Plugin.wrap(原始对象, interceptor1)
    Plugin-->>I1: 返回代理对象proxy1
    I1-->>IC: 返回proxy1
    IC->>I2: 调用interceptor2.plugin(proxy1)
    I2->>Plugin: Plugin.wrap(proxy1, interceptor2)
    Plugin-->>I2: 返回代理对象proxy2
    I2-->>IC: 返回proxy2
    IC-->>Config: 返回最终的proxy2



```

proxy2 的  洋葱样式层层包裹 结构：proxy2(interceptor2) → proxy1(interceptor1) → 原始对象。

proxy2  的 洋葱样式 嵌套结构 示意：

```
// 实际的嵌套结构：
代理对象proxy2 = {
    invocationHandler: Plugin对象2 {
        target: 代理对象proxy1,
        interceptor: 拦截器2
    }
}
代理对象proxy1 = {
    invocationHandler: Plugin对象1 {
        target: 原始对象RealTarget, 
        interceptor: 拦截器1
    }
}


```

`Plugin.wrap()` 源码 暗藏 **JDK 动态代理**。

```
public static Object wrap(Object target, Interceptor interceptor) {
 .....
  Class<?>[] interfaces = getAllInterfaces(type, signatureMap); // ① 只代理“有签名”的接口
  if (interfaces.length > 0) {
    //jdk 动态代理
    return Proxy.newProxyInstance(   // ②  创建代理
        type.getClassLoader(),
        interfaces,  //要实现的接口列表
        new Plugin(target, interceptor, signatureMap));         ② 真代理
  }
  return target;                                                 // ③ 无匹配，原样返回
}


```

`Plugin.wrap()` 方法 ，属于 是创建代理对象的工厂。

① 处 ：取得 出现的接口（如 StatementHandler） 。

② 处：JDK 动态代理要求 **目标类必须实现接口**，因此无法拦截四大接口以外的**私有方法**或 **final 类**。

③ 处：若当前插件对 target 无兴趣，直接返回，减少代理层级。

### 运行时： 责任链执行阶段

责任链执行阶段 ，也就是  方法调用与链式执行 ， 是运行时拦截 。

运行时拦截 & 执行 的过程：

*   **触发目标方法**：
    

业务代码调用 MyBatis 的 SqlSession 执行 SQL 时（如 sqlSession.selectList()），最终会调用四大核心组件的目标方法（如 Executor.query()）；

*   **代理对象拦截**：
    

由于核心组件已被代理，调用会先触发代理对象的 InvocationHandler.invoke() 方法（由 Plugin 类实现）；

*   **方法匹配校验**：
    

Plugin.invoke() 方法会根据插件的 @Intercepts 和 @Signature 注解，校验当前调用的方法是否为需要拦截的方法（通过方法名、参数类型匹配）；

*   **执行插件逻辑**：
    

若方法匹配，创建 Invocation 实例（封装目标对象、目标方法、参数），调用插件的 intercept(invocation) 方法，执行插件的自定义逻辑；

*   **链式传递调用**：
    

插件逻辑中通过 invocation.proceed() 触发下一级调用 —— 若存在多个插件，会调用下一个插件的代理逻辑；若为最后一个插件，则通过反射调用原始组件的目标方法；

*   **结果返回**：
    

原始方法执行完成后，返回结果按 “反向顺序” 经过每个插件的后置处理，最终返回给业务代码。

比如  当业务代码调用 sqlSession.selectList() 等方法时，底层最终会触发 Executor.query() 这样的核心方法。

但由于该对象早已被代理，实际执行的是代理对象的 invoke() 方法 —— 即 Plugin.invoke()。

Plugin.invoke() 伪代码：

```
public class Plugin implements InvocationHandler {
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 1. 检查方法是否需要拦截
        if (matches(method)) {
            // 2. 创建 Invocation 对象，target 是下一层代理或原始对象
            Invocation invocation = new Invocation(target, method, args);
            // 3. 调用拦截器的 intercept 方法
            return interceptor.intercept(invocation);
        }
        // 4. 不拦截，直接调用
        return method.invoke(target, args);
    }
}


```

Plugin.invoke()  调用  自定义 Plugin 的 intercept ，执行插件的逻辑。

```
@Intercepts(@Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class}))
public class MyPlugin implements Interceptor {
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        // 前置处理
        System.out.println("Before method execution");
        // 关键：调用 proceed() 继续执行责任链
        Object result = invocation.proceed();
        // 后置处理
        System.out.println("After method execution");
        return result;
    }
}


```

调用  Invocation.proceed()  继续执行责任链 ：

```
/
 * 方法调用信息的封装类，也是责任链传递的关键载体
 */
public class Invocation {
    // 三个核心字段都是 final，保证不可变性
    private final Object target;     // 目标对象（可能是原始对象或内层代理）
    private final Method method;     // 被调用的方法
    private final Object[] args;     // 方法参数
    /
     * 构造函数
/
    public Invocation(Object target, Method method, Object[] args) {
        this.target = target;
        this.method = method;
        this.args = args;
    }
    /
     * 责任链传递的核心方法 - 关键！
/
    public Object proceed() throws InvocationTargetException, IllegalAccessException {
        // 核心：通过反射调用目标方法
        return method.invoke(target, args);
    }
}


```

method.invoke(target, args);  通过反射调用目标方法 ， 目标对象 可能是原始对象或内层代理。

**(1) 这个`target`是在创建`Invocation`对象时传入的，它可能是原始对象，也可能是内层的代理对象。**

**(2) 当`target`是代理对象时，调用其方法会触发下一个拦截器，从而形成责任链的传递。**

**(3) 责任链的传递是通过代理机制隐式实现的，而不是显式的链表结构。**

因此 **责任链** 是通过 “**代理 包裹 代理**” （洋葱模型责任链 ），  而非链表指针完成。

#### 拦截顺序与洋葱模型责任链

假设注册 1 → 2 → 3，则代理嵌套：  proxy3(proxy2(proxy1(original)))

调用顺序：3.before → 2.before → 1.before → original → 1.after → 2.after → 3.after

**先注册后执行**，与 Servlet Filter 相同。

proxy3(proxy2(proxy1(original)))  调试时的调用栈深度分析

```
// 有3个拦截器时的调用栈：
executor.query()                    // 客户端调用
  → Proxy3.invoke()                 // 最外层代理
    → Interceptor3.intercept()      // 拦截器3逻辑
      → invocation3.proceed()       // 调用 proceed()
        → Proxy2.invoke()           // 内层代理被触发
          → Interceptor2.intercept() // 拦截器2逻辑  
            → invocation2.proceed() // 调用 proceed()
              → Proxy1.invoke()      // 更内层代理
                → Interceptor1.intercept() // 拦截器1逻辑
                  → invocation1.proceed() // 调用 proceed()
                    → original.query() // 最终执行原始方法


```

### 不只是插件机制，更是一种架构思维

回顾整个流程，MyBatis 插件机制的本质，是一场 **静态配置与动态代理的精密合谋**：

*   在启动期，通过解析配置完成拦截器收集与代理链预组装；
    
*   在运行时，借助 JDK 动态代理与责任链模式，实现无侵入式的逻辑织入；
    
*   整个过程无需修改原有代码，却能达到类似 AOP 的效果。
    

更重要的是，这种 “代理嵌套 + 反射传递” 的责任链实现方式，展示了如何在有限的技术约束下（如仅支持接口代理），依然构建出灵活、可扩展的插件生态。

## MyBatis 四大金刚接口的对象创建 与插件应用详解

MyBatis 四大金刚接口 的创建时机和插件应用机制。

### 四大金刚接口

<table><thead><tr><td><span><strong><span leaf="">对象</span></strong></span></td><td><span><strong><span leaf="">创建点</span></strong></span></td><td><span><strong><span leaf="">立即被 pluginAll</span></strong></span></td><td><span><strong><span leaf="">说明</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">Executor</span></span></section></td><td><section><span><span leaf="">Configuration.newExecutor()</span></span></section></td><td><section><span><span leaf="">✔</span></span></section></td><td><section><span><span leaf="">SQL 执行器，第一个被创建和包装</span></span></section></td></tr><tr><td><section><span><span leaf="">StatementHandler</span></span></section></td><td><section><span><span leaf="">Configuration.newStatementHandler()</span></span></section></td><td><section><span><span leaf="">✔</span></span></section></td><td><section><span><span leaf="">SQL 语句处理器</span></span></section></td></tr><tr><td><section><span><span leaf="">ParameterHandler</span></span></section></td><td><section><span><span leaf="">Configuration.newParameterHandler()</span></span></section></td><td><section><span><span leaf="">✔</span></span></section></td><td><section><span><span leaf="">参数处理器</span></span></section></td></tr><tr><td><section><span><span leaf="">ResultSetHandler</span></span></section></td><td><section><span><span leaf="">Configuration.newResultSetHandler()</span></span></section></td><td><section><span><span leaf="">✔</span></span></section></td><td><section><span><span leaf="">结果集处理器</span></span></section></td></tr></tbody></table>

### 四大金刚接口 的 实例 创建流程

### 1. Executor  的 对象 创建过程

当一次数据库操作发起时，MyBatis 首先要做的就是创建一个 Executor 实例，它是整个 SQL 执行流程的调度中枢。

它的创建发生在 Configuration.newExecutor() 方法中，这个方法不仅决定了执行器的具体类型（如简单、批处理或可复用），还承担着两层关键职责：二级缓存封装、 插件链注入。

```
public class Configuration {
    /*
     * Executor 的创建入口
/
    public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
        // 1. 创建基础执行器
        Executor executor;
        if (ExecutorType.BATCH == executorType) {
            executor = new BatchExecutor(this, transaction);
        } else if (ExecutorType.REUSE == executorType) {
            executor = new ReuseExecutor(this, transaction);
        } else {
            executor = new SimpleExecutor(this, transaction);
        }
        // 2. 如果启用二级缓存，包装为CachingExecutor
        if (cacheEnabled) {
            executor = new CachingExecutor(executor);
        }
        // 3. 关键：应用所有插件（责任链包装）
        executor = (Executor) interceptorChain.pluginAll(executor);
        return executor;
    }
}


```

这里有几个值得注意的技术细节：

*   **策略模式的应用**：根据配置选择不同的 Executor 实现，体现了典型的策略模式思想；
    
*   **装饰器模式叠加**：如果启用了二级缓存，会使用 CachingExecutor 对原始执行器进行包装，形成责任链结构；
    
*   **插件链最后介入**：无论是否有缓存，最终都要经过 interceptorChain.pluginAll() 处理，这意味着插件看到的是已经被缓存层包裹后的代理对象。
    

我们在开发插件时必须意识到：**所拦截的对象 ，已经是多层代理后的复合体**，不能假设其是 “纯净” 的原始实现。

### 2. StatementHandler 的 对象  的创建过程

紧随 Executor 之后，MyBatis 开始准备 SQL 语句本身。

此时会调用 Configuration.newStatementHandler() 创建 StatementHandler。

这是真正负责将 Mapper 接口方法映射为 JDBC PreparedStatement 的核心组件。

```
public StatementHandler newStatementHandler(Executor executor, 
                                         MappedStatement mappedStatement,
                                         Object parameterObject, 
                                         RowBounds rowBounds,
                                         ResultHandler resultHandler,
                                         BoundSql boundSql) {
    // 1. 创建路由StatementHandler（根据语句类型选择具体实现）
    StatementHandler statementHandler = new RoutingStatementHandler(executor, mappedStatement, 
                                                                  parameterObject, rowBounds, 
                                                                  resultHandler, boundSql);
    // 2. 应用插件链
    statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);
    return statementHandler;
}


```

这里的 RoutingStatementHandler 是一个路由门面。

它会根据 SQL 类型（SELECT/INSERT/UPDATE/DELETE）动态委派给具体的子类（如 SimpleStatementHandler、PreparedStatementHandler 等）。

但无论具体实现如何变化，**所有 StatementHandler 实例在返回前都会被插件链统一包装**。

这使得 StatementHandler 成为了许多高级功能插件的首选目标，比如：

*   SQL 改写类插件（分库分表、读写分离）
    
*   SQL 安全审计（防 SQL 注入检测）
    
*   自动填充字段（创建时间、更新人等）
    

### 3. ParameterHandler 的 对象 的创建过程

参数处理器 ParameterHandler 负责将 Java 对象中的值设置到预编译语句的占位符中。

虽然它的职责看似单一，但它却是实现数据加密、脱敏、自动赋值等功能的理想位置。

```
public ParameterHandler newParameterHandler(MappedStatement mappedStatement,
                                           Object parameterObject,
                                           BoundSql boundSql) {
    // 1. 创建默认参数处理器
    ParameterHandler parameterHandler = new DefaultParameterHandler(mappedStatement, 
                                                                   parameterObject, boundSql);
    // 2. 应用插件链
    parameterHandler = (ParameterHandler) interceptorChain.pluginAll(parameterHandler);
    return parameterHandler;
}


```

注意，这里创建的是 DefaultParameterHandler，它是唯一实现类，不可替换。

但由于它实现了接口且在创建后立即被插件链处理，因此我们仍然可以通过插件机制对其进行增强。

例如，在金融系统中常见的场景是：某些敏感字段（如身份证号、银行卡号）在入库前需加密存储。

传统做法是在业务层手动加密，容易遗漏；而借助插件机制，可以在 setParameters 被调用前自动识别标注了 @Encrypted 的字段并完成加密。

### 4. ResultSetHandler 的 对象 的创建过程

最后一个被创建的是 ResultSetHandler，它负责将 JDBC 的 ResultSet 映射为开发者期望的 POJO 或集合。

对于想要干预查询结果的场景（如字段脱敏、空值补全、性能监控），这是一个极其重要的拦截点。

```
public ResultSetHandler newResultSetHandler(Executor executor,
                                           MappedStatement mappedStatement,
                                           RowBounds rowBounds,
                                           ParameterHandler parameterHandler,
                                           ResultHandler resultHandler,
                                           BoundSql boundSql) {
    // 1. 创建默认结果集处理器
    ResultSetHandler resultSetHandler = new DefaultResultSetHandler(executor, mappedStatement,
                                                                   parameterHandler, resultHandler,
                                                                   rowBounds, boundSql);
    // 2. 应用插件链
    resultSetHandler = (ResultSetHandler) interceptorChain.pluginAll(resultSetHandler);
    return resultSetHandler;
}


```

由于结果映射往往涉及复杂反射和嵌套处理，直接在业务代码中做后置处理效率低且难以复用。

而通过插件方式拦截 handleResultSets 方法，则可以在不改动任何 Mapper 接口的前提下，全局统一实现：

*   敏感字段脱敏（手机号中间四位变星号）
    
*   时间字段格式化（Date → LocalDateTime）
    
*   分页总数自动注入
    

### 2.5 可拦截方法完整列表

了解了四大金刚的对象创建流程后，下一步就是明确它们各自暴露了哪些可供插件拦截的方法。

这些方法构成了插件的能力边界。

这些方法 也是面试官常问的一个问题 “你知道 MyBatis 哪些方法可以被拦截吗？” 的标准答案。

<table><thead><tr><td><span><strong><span leaf="">组件</span></strong></span></td><td><span><strong><span leaf="">可拦截方法</span></strong></span></td></tr></thead><tbody><tr><td><section><span><strong><span leaf="">Executor</span></strong></span></section></td><td><section><span><span leaf="">update, query, queryCursor, flushStatements, commit, rollback, getTransaction, close, isClosed</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">StatementHandler</span></strong></span></section></td><td><section><span><span leaf="">prepare, parameterize, batch, update, query, getBoundSql, getParameterHandler</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">ParameterHandler</span></strong></span></section></td><td><section><span><span leaf="">setParameters, getParameterObject</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">ResultSetHandler</span></strong></span></section></td><td><section><span><span leaf="">handleResultSets, handleCursorResultSets, handleOutputParameters</span></span></section></td></tr></tbody></table>

这些方法的选择并非随机，而是围绕 SQL 生命周期的关键节点展开：

*   Executor.query：适合做整体耗时统计、慢 SQL 记录；
    
*   StatementHandler.prepare：适合修改 SQL 文本、添加额外条件；
    
*   ParameterHandler.setParameters：适合参数加密、日志打印；
    
*   ResultSetHandler.handleResultSets：适合结果脱敏、异常兜底。
    

## MyBatis 插件核心支撑组件源码解析

### 核心支撑 组件

除了四大核心组件（Executor、StatementHandler 等），插件机制的核心支撑组件包括：

<table><thead><tr><td><span><strong><span leaf="">组件名称</span></strong></span></td><td><span><strong><span leaf="">全类名</span></strong></span></td><td><span><strong><span leaf="">核心职责</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">Interceptor</span></span></section></td><td><section><span><span leaf="">org.apache.ibatis.plugin.Interceptor</span></span></section></td><td><section><span><span leaf="">插件核心接口，定义拦截逻辑（intercept）、代理生成（plugin）等方法</span></span></section></td></tr><tr><td><section><span><span leaf="">InterceptorChain</span></span></section></td><td><section><span><span leaf="">org.apache.ibatis.plugin.InterceptorChain</span></span></section></td><td><section><span><span leaf="">插件链管理器，存储所有注册的插件，负责生成代理链（pluginAll 方法）</span></span></section></td></tr><tr><td><section><span><span leaf="">Plugin</span></span></section></td><td><section><span><span leaf="">org.apache.ibatis.plugin.Plugin</span></span></section></td><td><section><span><span leaf="">代理实现类，实现 InvocationHandler，负责匹配拦截方法并触发插件逻辑</span></span></section></td></tr><tr><td><section><span><span leaf="">Invocation</span></span></section></td><td><section><span><span leaf="">org.apache.ibatis.plugin.Invocation</span></span></section></td><td><section><span><span leaf="">调用信息封装类，封装目标对象、目标方法、方法参数，提供 proceed() 方法</span></span></section></td></tr><tr><td><section><span><span leaf="">Intercepts</span></span></section></td><td><section><span><span leaf="">org.apache.ibatis.plugin.Intercepts</span></span></section></td><td><section><span><span leaf="">注解，用于声明插件要拦截的组件和方法（包含多个 @Signature 注解）</span></span></section></td></tr><tr><td><section><span><span leaf="">Signature</span></span></section></td><td><section><span><span leaf="">org.apache.ibatis.plugin.Signature</span></span></section></td><td><section><span><span leaf="">注解，精确指定拦截的组件类型（type）、方法名（method）、参数类型（args）</span></span></section></td></tr></tbody></table>

### 【1】Interceptor：插件核心接口（定义扩展规范）

`Interceptor`是所有 MyBatis 插件的顶层接口，定义了插件必须实现的核心方法，是插件机制的 “扩展规范”。

Interceptor 接口是所有自定义插件必须实现的顶层契约，它定义了三个核心能力：**拦截逻辑、代理生成、属性配置**。

可以把它类比为 Spring AOP 中的 Advice，只不过它是基于 JDK 动态代理实现的轻量级方案。

```
package org.apache.ibatis.plugin;
import java.util.Properties;
/
 插件核心接口：所有自定义插件必须实现此接口
/
public interface Interceptor {
   /
    * 1. 核心拦截逻辑：插件的业务逻辑入口
    * @param invocation 封装了目标对象、目标方法、方法参数的调用信息
    * @return 目标方法的执行结果（可被插件修改后返回）
    * @throws Throwable 允许抛出异常，由上层业务处理
/
   Object intercept(Invocation invocation) throws Throwable;
   /
    * 2. 生成代理对象：决定是否为目标对象创建代理（默认委托Plugin.wrap()实现）
    * @param target 被拦截的目标对象（如Executor、StatementHandler实例）
    * @return 代理对象（若无需拦截则返回原始target）
/
   default Object plugin(Object target) {
       // 委托Plugin工具类创建代理，避免每个插件重复实现代理逻辑
       return Plugin.wrap(target, this);
   }
   /*
    * 3. 配置属性：读取插件的配置参数（如XML中<property>标签配置的属性）
    * @param properties 配置属性集合（key为属性名，value为属性值）
/
   default void setProperties(Properties properties) {
       // 默认空实现：插件若需配置属性，可重写此方法
   }
}


```

**源码解读**：

*   `intercept`：必须重写，是插件的核心业务逻辑所在（如 SQL 耗时统计、数据加密），通过`invocation.proceed()`触发后续调用；
    
*   `plugin`：默认方法，通过`Plugin.wrap()`标准化代理生成逻辑，避免插件重复开发动态代理代码；
    
*   `setProperties`：默认空实现，插件若需外部配置（如慢 SQL 阈值、加密密钥），可重写此方法读取属性。
    

尼恩提示：plugin() 和 setProperties() 都是默认方法，大多数情况下 只需要关注 intercept() 即可 。：

### 【2】InterceptorChain：插件链管理器（组织插件执行顺序）

如果说 Interceptor 是个体，那么 InterceptorChain 就是组织。

`InterceptorChain`是插件的 “容器与调度器”，负责存储所有注册的插件，并在核心组件创建时生成代理链，保证插件按注册顺序执行。

它是一个简单的 List 容器，但却承担着至关重要的职责：**管理所有注册的插件，并在对象创建时依次调用其 plugin() 方法，形成层层嵌套的代理链**。

```
package org.apache.ibatis.plugin;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
/
 插件链管理器：管理所有注册的插件，生成代理链
/
public class InterceptorChain {
   // 存储所有注册的插件（线程安全：仅启动时添加，运行时只读）
   private final List<Interceptor> interceptors = new ArrayList<>();
   /
    * 核心方法：为目标对象生成代理链（插件依次包装目标对象）
    * @param target 被拦截的核心组件（如Executor、StatementHandler）
    * @return 最终的代理对象（多层代理：插件1→插件2→原始target）
/
   public Object pluginAll(Object target) {
       // 遍历所有插件，依次为目标对象创建代理（注册顺序=执行顺序）
       for (Interceptor interceptor : interceptors) {
           // 调用每个插件的plugin()方法，生成新的代理对象
           target = interceptor.plugin(target);
       }
       return target;
   }
   /
    * 添加插件到链中（MyBatis启动时解析配置后调用）
    * @param interceptor 插件实例
/
   public void addInterceptor(Interceptor interceptor) {
       interceptors.add(interceptor);
   }
   /*
    * 获取所有插件（只读，避免外部修改插件列表）
    * @return 不可修改的插件列表
/
   public List<Interceptor> getInterceptors() {
       return Collections.unmodifiableList(interceptors);
   }
}


```

**源码解读**：

*   `pluginAll`：核心逻辑，通过循环调用每个插件的`plugin()`方法，将目标对象层层包装为代理（如注册 2 个插件：`target → 插件2代理 → 插件1代理`）；
    
*   `addInterceptor`：MyBatis 启动时，从配置中解析插件后调用此方法添加到列表；
    
*   `getInterceptors`：返回不可修改的列表，防止运行时插件列表被意外篡改，保证线程安全。
    

这个 pluginAll() 方法看似简单，实则暗藏玄机：

*   **执行顺序 = 注册顺序**：先注册的插件会被后注册的插件 “包在外面”，也就是说，调用时会**最先触发**；
    
*   **链式包装**：每次 plugin() 返回的都是一个新的代理对象，作为下一个插件的输入目标；
    
*   **只读保障**：插件列表一旦初始化完成就不再修改，确保运行时线程安全。
    

### 【3】Plugin：代理实现类（动态代理的 “执行者”）

`Plugin`是 MyBatis 动态代理的核心实现类，实现`InvocationHandler`接口，负责：

1、校验方法是否需要拦截；

2、触发插件的`intercept`逻辑；

3、调用原始方法或下一个代理。

Plugin 类 实现了 InvocationHandler 接口，是 JDK 动态代理的实际处理器。

Plugin 类是整个插件机制中最核心的技术实现， 可以把它看作是 MyBatis 自研的 “AOP 代理生成器”。

```
package org.apache.ibatis.plugin;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.HashMap;
import java.util.HashSet;
import java.util.Map;
import java.util.Set;
/
 动态代理实现类：负责拦截方法调用，触发插件逻辑
/
public class Plugin implements InvocationHandler {
   // 原始目标对象（如Executor实例，非代理对象）
   private final Object target;
   // 对应的插件实例（当前代理关联的Interceptor）
   private final Interceptor interceptor;
   // 插件要拦截的方法映射（key：组件接口类型，value：该接口下需拦截的方法集合）
   private final Map<Class<?>, Set<Method>> signatureMap;
   /
    * 私有构造：仅通过wrap()方法创建实例（避免外部随意创建）
/
   private Plugin(Object target, Interceptor interceptor, Map<Class<?>, Set<Method>> signatureMap) {
       this.target = target;
       this.interceptor = interceptor;
       this.signatureMap = signatureMap;
   }
   /
    * 静态工厂方法：创建代理对象（插件生成代理的统一入口）
    * @param target 目标对象
    * @param interceptor 插件实例
    * @return 代理对象（若无需拦截则返回原始target）
/
   public static Object wrap(Object target, Interceptor interceptor) {
       // 1. 解析插件的@Intercepts和@Signature注解，获取需拦截的方法映射
       Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);
       // 2. 获取目标对象的所有接口（JDK动态代理需基于接口）
       Class<?> type = target.getClass();
       Class<?>[] interfaces = getAllInterfaces(type, signatureMap);
       // 3. 若存在需拦截的接口，生成代理；否则返回原始对象
       if (interfaces.length > 0) {
           return Proxy.newProxyInstance(
               type.getClassLoader(),  // 目标对象的类加载器
               interfaces,             // 目标对象实现的接口（需拦截的接口）
               new Plugin(target, interceptor, signatureMap)  // 代理处理器
           );
       }
       return target;
   }
   /
    * 代理核心逻辑：方法调用时触发（JDK动态代理的核心方法）
    */
   @Override
   public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
       try {
           // 1. 获取当前方法所属接口需拦截的方法集合
           Set<Method> methods = signatureMap.get(method.getDeclaringClass());
           // 2. 若当前方法需拦截，触发插件的intercept逻辑
           if (methods != null && methods.contains(method)) {
               // 封装调用信息为Invocation，传递给插件
               return interceptor.intercept(new Invocation(target, method, args));
           }
           // 3. 若无需拦截，直接调用原始目标对象的方法
           return method.invoke(target, args);
       } catch (Exception e) {
           // unwrapThrowable：解包异常（如InvocationTargetException），返回真实异常
           throw ExceptionUtil.unwrapThrowable(e);
       }
   }
   /
    * 解析插件的@Intercepts注解，生成“接口→方法集合”的映射
/
   private static Map<Class<?>, Set<Method>> getSignatureMap(Interceptor interceptor) {
       // 获取插件类上的@Intercepts注解
       Intercepts interceptsAnnotation = interceptor.getClass().getAnnotation(Intercepts.class);
       // 若未加@Intercepts注解，抛出异常（插件必须声明拦截规则）
       if (interceptsAnnotation == null) {
           throw new PluginException("No @Intercepts annotation was found in interceptor " + interceptor.getClass().getName());
       }
       // 获取@Intercepts注解中的@Signature数组（每个@Signature对应一个拦截规则）
       Signature[] sigs = interceptsAnnotation.value();
       Map<Class<?>, Set<Method>> signatureMap = new HashMap<>();
       for (Signature sig : sigs) {
           // 获取@Signature中指定的组件接口（如Executor.class）
           Set<Method> methods = signatureMap.computeIfAbsent(sig.type(), k -> new HashSet<>());
           try {
               // 根据方法名和参数类型，获取接口中的目标方法
               Method method = sig.type().getMethod(sig.method(), sig.args());
               methods.add(method);
           } catch (NoSuchMethodException e) {
               throw new PluginException("Could not find method on " + sig.type() + " named " + sig.method() + ". Cause: " + e, e);
           }
       }
       return signatureMap;
   }
   /
    * 筛选目标对象实现的接口中，需要被拦截的接口
/
   private static Class<?>[] getAllInterfaces(Class<?> type, Map<Class<?>, Set<Method>> signatureMap) {
       Set<Class<?>> interfaces = new HashSet<>();
       // 遍历目标对象的所有接口（包括父类实现的接口）
       while (type != null) {
           for (Class<?> c : type.getInterfaces()) {
               // 若接口在signatureMap中（即插件需拦截此接口），加入结果集
               if (signatureMap.containsKey(c)) {
                   interfaces.add(c);
               }
           }
           type = type.getSuperclass();
       }
       // 转为数组返回（JDK动态代理需传入接口数组）
       return interfaces.toArray(new Class<?>[0]);
   }
}


```

**源码解读**：

**(1) `wrap`：静态工厂方法，是代理生成的入口，分三步：解析注解→筛选接口→生成代理；**

**(2) `invoke`：代理的核心触发逻辑，先校验方法是否在拦截列表中，是则执行插件`intercept`，否则调用原始方法；**

**(3) `getSignatureMap`：解析`@Intercepts`和`@Signature`注解，将拦截规则转为 “接口→方法” 映射，避免每次调用重复解析；**

**(4) `getAllInterfaces`：筛选目标对象中需拦截的接口，确保代理仅针对插件关注的接口生成，减少不必要的代理开销。**

这段代码体现了三个精巧设计：

**(1) 注解驱动配置：通过 @Intercepts + @Signature 声明式指定拦截点，替代繁琐的 XML 配置；**

**(2) 精确匹配机制：args 参数用于区分重载方法，避免误拦截；**

**(3) 最小代理原则：只对接口中有声明拦截的方法生成代理，减少性能损耗。**

### 【4】Invocation：调用信息封装类（传递调用上下文）

`Invocation`是插件调用的 “上下文载体”，封装了目标对象、目标方法、方法参数，提供`proceed()`方法触发后续调用，解耦插件与原始方法的直接依赖。

Invocation 是插件开发中最常用的辅助类，它封装了目标对象、方法和参数，提供了一个干净的调用上下文。

```
package org.apache.ibatis.plugin;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
/
 调用信息封装类：封装目标方法的调用上下文
/
public class Invocation {
   // 原始目标对象（可能是下一个插件的代理，或最终的核心组件实例）
   private final Object target;
   // 被调用的目标方法
   private final Method method;
   // 目标方法的参数数组
   private final Object[] args;
   /
    * 构造方法：封装调用信息
/
   public Invocation(Object target, Method method, Object[] args) {
       this.target = target;
       this.method = method;
       this.args = args;
   }
   // Getter方法：供插件获取调用信息（如修改参数、获取方法名）
   public Object getTarget() {
       return target;
   }
   public Method getMethod() {
       return method;
   }
   public Object[] getArgs() {
       return args;
   }
   /
    * 核心方法：触发后续调用（下一个插件或原始方法）
    * @return 目标方法的执行结果
    * @throws InvocationTargetException 方法调用异常
    * @throws IllegalAccessException 权限异常
/
   public Object proceed() throws InvocationTargetException, IllegalAccessException {
       // 反射调用目标对象的方法（若target是代理，则触发下一个插件的invoke逻辑）
       return method.invoke(target, args);
   }
}


```

**源码解读**：

*   封装性：将调用相关的三个核心要素（target、method、args）封装，插件无需直接处理反射细节；
    
*   `proceed()`：是调用链传递的关键，插件执行前置逻辑后调用此方法，触发下一个插件或原始方法，实现 “链式执行”；
    
*   可扩展性：插件可通过 Getter 方法修改参数（如`getArgs()[0] = newParam`）或获取方法信息（如`getMethod().getName()`），实现灵活的逻辑干预。
    

它的最大价值体现在 proceed() 方法上：**它是责任链向下传递的开关**。插件可以在调用 proceed() 前后插入逻辑，实现环绕通知的效果。

举个例子，统计 SQL 执行时间的经典写法：

```
long start = System.currentTimeMillis();
Object result = invocation.proceed(); // 触发后续调用
long cost = System.currentTimeMillis() - start;
log.info("SQL executed in {} ms", cost);
return result;


```

此外，你还可通过 getArgs() 修改参数、通过 getMethod() 获取方法名做路由分发，灵活性极高。

### 【5】Intercepts 与 Signature：注解（声明拦截规则）

`Intercepts`和`Signature`是 “注解式配置”，用于声明插件的拦截规则（拦截哪个组件、哪个方法），替代 XML 配置，简化插件开发。

二者均为元注解，仅用于标记和传递配置信息。

#### 5.1 Intercepts 注解（插件的拦截规则容器）

```
package org.apache.ibatis.plugin;
import java.lang.annotation.;
/
 插件拦截规则容器：标注在插件类上，包含多个@Signature注解
*/
@Documented
@Retention(RetentionPolicy.RUNTIME)  // 运行时保留，供Plugin类解析
@Target(ElementType.TYPE)            // 仅可标注在类上（插件类）
public @interface Intercepts {
   // 多个@Signature注解（一个插件可拦截多个方法）
   Signature[] value();
}


```

#### 5.2 Signature 注解（单个拦截规则）

```
package org.apache.ibatis.plugin;
import java.lang.annotation.;
/
 单个拦截规则：指定拦截的组件、方法、参数
*/
@Documented
@Retention(RetentionPolicy.RUNTIME)  // 运行时保留，供Plugin类解析
@Target({})  // 无目标类型：仅可在@Intercepts中使用
public @interface Signature {
   // 拦截的组件接口类型（如Executor.class、StatementHandler.class）
   Class<?> type();
   // 拦截的方法名（如"query"、"update"、"setParameters"）
   String method();
   // 拦截方法的参数类型数组（用于精确匹配方法，避免方法重载冲突）
   Class<?>[] args();
}


```

**源码解读**：

*   元数据作用：二者均为运行时保留的注解，仅用于传递拦截规则，无业务逻辑，核心解析逻辑在`Plugin.getSignatureMap()`中；
    
*   精确匹配：`args`参数是关键，可解决方法重载问题（如`Executor.query`有多个重载方法，通过参数类型区分）；
    
*   灵活性：一个`@Intercepts`可包含多个`@Signature`，支持一个插件拦截多个组件或多个方法（如同时拦截`Executor.query`和`Executor.update`）。
    

## mybatis 的 3 大不足

MyBatis 在实际项目中仍存在一些反模式和潜在风险，稍有不慎就会引发线上故障。

这些往往是面试官压轴提问的方向：“你觉得 MyBatis 插件有什么缺点？”

#### 1. 金字塔陷阱：层层修改 SQL，最终面目全非

当多个插件依次修改 SQL 时（如分页插件改 LIMIT，租户插件加 tenant_id，安全插件过滤敏感词），最终生成的 SQL 与原始 XML 完全不符，调试时极难追踪。

规避方案：统一 SQL 改写入口，或记录每一步的变更日志。

#### 2. 循环增强：插件内部调用 Mapper 导致递归爆栈

常见错误是在插件的 intercept() 中再次调用 sqlSession.selectList()，结果触发新一轮拦截，形成无限递归，最终 StackOverflowError。

规避方案：

*   插件内部禁止调用 Mapper；
    
*   使用 ThreadLocal 标记当前是否已在拦截流程中；
    
*   或通过 Ordered 接口显式控制执行顺序。
    

#### 3. 热部署失效：DevTools 重启后插件重复注册

Spring Boot DevTools 重启时，SqlSessionFactory 是新实例，但如果插件被注册到了 static 列表中，旧实例未清理，就会导致同一插件被多次加载，出现重复拦截。

规避方案：避免使用 static 集合存储插件；推荐通过 Spring 容器管理插件生命周期。

### mybatis 使用的雷区

MyBatis 保证 **一个 SqlSessionFactory 对应一套 Interceptor 单例**，因此

*   禁止在插件里存 **请求级** 变量 → 用 ThreadLocal 或 Invocation 临时传参。
    
*   禁止开线程池 → 生命周期与 MyBatis 不一致易内存泄漏。
    

## MyBatis  核心架构思想

MyBatis 插件机制的 “动态代理 + 责任链” 组合，并非单纯的技术堆砌，而是对**开闭原则、单一职责、最小侵入性**等经典架构原则的深度落地，形成了 “灵活扩展与核心稳定共存” 的设计哲学。

### 一：OCP 开闭原则：扩展开放，修改关闭

**思想内涵**：

允许在不修改框架核心源码的前提下，通过扩展机制增强功能，是插件机制的核心设计准则。

**落地细节**：

*   核心组件（Executor、StatementHandler 等）以接口形式定义，避免因实现类修改影响全局；
    
*   插件通过 Interceptor 接口接入，无需修改 MyBatis 核心流程（如 SQL 执行链路）；
    

示例：

新增 “SQL 加密” 功能时，只需实现 Interceptor 接口定义插件，无需改动 Executor 或 StatementHandler 的原始实现，符合 “不修改核心代码” 的开闭原则。

#### 从源码看 OCP 如何落地

Configuration 创建对象时统一：

```
executor = (Executor) interceptorChain.pluginAll(executor);


```

此后无论新增多少插件，Configuration 源码**永不改动**，符合 **Open-Close**。

如何理解 这里的 “开放” 与 “封闭”?

*   **开放**：需求变化时，允许通过**新增代码**（继承、实现新类、插件等）来扩展功能。
    
*   **封闭**：扩展时**尽量不改动**已有源码或二进制，避免引入回归缺陷
    

在 MyBatis 中，四大核心对象（Executor、StatementHandler…）的创建逻辑被固化在 `Configuration` 里，但框架通过 `InterceptorChain.pluginAll()` 给它们套上一层 **JDK 动态代理**：

```
public Object pluginAll(Object target) {
  for (Interceptor interceptor : interceptors) {
    target = interceptor.plugin(target); // 层层包裹，不改动原类
  }
  return target;
}


```

当需要增加 “分页 / 加密 / 多租户” 等横切功能时：

*   **只新增**一个实现 `Interceptor` 的插件类；
    
*   **无需修改**`Configuration`、`DefaultSqlSessionFactory` 等任何核心源码；
    

插件通过注解声明签名，框架在运行时**动态代理**织入，既扩展了能力，又保证了旧代码零触碰——这就是 OCP 的经典实践

OCP 要求 “**不改旧代码，只写新代码**”；

MyBatis 用 “动态代理 + 责任链” 真正做到：**“想加功能？写个插件就行， core 文件一眼不看。”**

### 二： SRP 单一职责原则：职责分离，各司其职

如果说 OCP 是插件机制的顶层设计目标，那么 **单一职责原则（SRP）** 就是保障这一目标能够稳健落地的基础架构理念。

**思想内涵**：每个组件仅负责单一功能，避免职责耦合导致的维护复杂度。

**落地细节**：

*   **动态代理（Plugin 类）**：仅负责 “代理生成 + 方法拦截校验”，不参与业务逻辑；
    
*   **责任链（InterceptorChain 类）**：仅负责 “插件管理 + 代理链装配”，不干预插件具体逻辑；
    
*   **插件（Interceptor 实现类）**：仅关注自身横切逻辑（如日志、监控），无需感知其他插件存在；
    

试想一下：如果一个类既要做代理生成，又要管理插件列表，还得执行具体的拦截逻辑，那它的职责就会高度耦合，一旦改动牵一发而动全身。

MyBatis 显然没有走这条路，而是将整个插件体系拆解为三个清晰的角色，各自专注自己的领域。

*   首先是 Plugin 类，它的唯一任务就是**生成 JDK 动态代理对象**，并在 invoke() 方法中判断当前调用的方法是否匹配插件声明的签名。如果是，则交由 Interceptor.intercept() 处理；否则直接放行，调用原始方法。
    
*   其次是 InterceptorChain，它只关心两件事：维护注册的插件列表，以及按顺序执行 plugin() 方法完成代理链的装配。它并不知道也不关心每个插件到底做了什么业务逻辑。
    
*   最后是用户自定义的 Interceptor 实现类，它专注于横切关注点本身——比如记录日志、统计耗时、多租户字段注入等。它不需要感知其他插件的存在，也不参与代理机制的构建。
    

这种 “各司其职” 的分工方式，使得整个系统具备了良好的模块化结构。每一个组件都可以独立测试、独立替换，也为后续的调试和性能优化提供了便利。

### 三：最小侵入性原则：减少对核心流程的干扰

**思想内涵**：扩展功能时，尽量减少对框架核心流程的依赖与干扰，保持核心逻辑简洁。

**落地细节**：

*   代理生成仅针对 “插件声明需要拦截的接口”（Plugin.getAllInterfaces() 筛选接口），避免对无关接口生成代理；
    
*   插件未匹配拦截规则时，直接调用原始方法（Plugin.invoke() 中的 method.invoke(target, args)），无额外性能损耗；
    

示例：若插件仅拦截 Executor.query()，则 Plugin 仅为 Executor 接口生成代理，不会影响 StatementHandler 等其他组件，体现 “按需代理” 的最小侵入。

尽管动态代理是一种强大的增强手段，但如果使用不当，也可能带来额外的性能开销。

比如，若对所有对象的所有方法都生成代理，即使没有实际拦截逻辑，也会导致反射调用增多、内存占用上升。

为此，MyBatis 提出了 **最小侵入性原则** ——即扩展功能应尽可能减少对核心流程的干扰，确保在未命中拦截规则时，性能损耗趋近于零。

这一点在 Plugin 类的设计中体现得淋漓尽致。它并不是盲目地为所有接口生成代理，而是通过 getAllInterfaces() 方法精确筛选出那些**被插件明确声明需要拦截的接口类型**。

举个例子：如果你的插件只标注了要拦截 Executor.query() 方法，那么框架只会针对 Executor 接口生成代理，而 StatementHandler、ParameterHandler 等其他组件则完全不受影响。

更进一步，在 Plugin.invoke() 方法中，对于不匹配的方法调用，会直接通过 method.invoke(target, args) 转发到原始目标对象，没有任何中间处理或条件判断。

这意味着：

*   没有冗余的反射开销；
    
*   没有多余的对象包装；
    
*   更没有全局性的性能退化。
    

只有当方法签名真正匹配时，才会进入 intercept() 回调。

这种 “**按需代理、精准拦截**”的策略，既保证了功能的灵活性，又最大限度地降低了副作用，完美诠释了 “扩展不应成为负担” 的设计理念。

### 四：DIP 依赖倒置原则：面向接口编程

**思想内涵**：依赖抽象接口而非具体实现，降低组件间耦合，提升扩展性。

**落地细节**：

*   四大核心组件均以接口形式存在（如 Executor、StatementHandler），插件针对接口拦截，而非具体实现类；
    
*   动态代理基于接口生成（JDK 动态代理特性），确保插件与核心组件的实现解耦；
    

示例：无论 Executor 的实现类是 BaseExecutor 还是 CachingExecutor，插件均可通过拦截 Executor 接口生效，无需关注具体实现，符合 “面向接口编程”。

### 五、微内核架构

微内核架构：核心聚焦，能力外置

**思想内涵**

在系统从单体走向模块化、从封闭走向可扩展的过程中，**微内核架构**逐渐成为高可维护性框架设计的核心范式之一。它也被称为 “插件化架构”，其本质是一种**职责分离的极致体现**——将最稳定、最基础的功能抽象为一个轻量级的 “内核”，而把所有可能变化或定制化的功能剥离出去，作为“插件” 动态加载。

这种架构的核心理念是：**内核只做最必要的事，其余都交给生态去扩展**。具体来说，微内核仅保留系统运行所必需的基础流程控制和组件管理能力，比如请求调度、生命周期管理、通信协议等；而像日志记录、权限校验、性能监控这类非功能性需求，则全部通过标准化接口以插件形式挂载进来。

**落地细节（结合 MyBatis 插件机制）**

MyBatis 的微内核架构在插件机制中体现得淋漓尽致，其核心与插件的职责边界划分极为清晰：

**微内核核心职责**：仅聚焦 “SQL-Mapper 映射” 这一核心目标，具体包括：

1、解析 Mapper 接口与 XML 映射文件，建立 SQL 与方法的绑定关系；

2、管理四大核心组件（Executor、StatementHandler 等）的生命周期，确保 SQL 执行的基础链路（参数处理→SQL 执行→结果映射）正常运行；

3、提供插件接入的标准化接口（如 Interceptor、Invocation），但不介入插件的具体业务逻辑。

**插件外置能力**：所有非核心的增强功能均通过插件实现，且插件完全独立于内核，例如：

1、性能监控（SQL 耗时统计）、日志记录（SQL 语句与参数打印）；

2、数据安全（敏感字段加密 / 解密）、通用业务（多租户数据隔离、分页逻辑）；

3、这些功能既不嵌入内核源码，也不依赖内核的内部实现细节，仅通过内核提供的接口接入。

**具体体现（源码与案例）**

从 Configuration 类（MyBatis 的核心配置载体）可见，内核仅维护插件的 “管理入口”（interceptorChain），不关心插件的具体逻辑：

```
public class Configuration {
    // 内核仅持有插件链的引用，负责加载和装配，不参与具体执行
    protected final InterceptorChain interceptorChain = new InterceptorChain();
    public Executor newExecutor(Transaction transaction) {
        Executor executor = new SimpleExecutor(this, transaction);
        // 将executor交给插件链进行代理包装，内核不做任何判断
        return (Executor) interceptorChain.pluginAll(executor);
    }
}


```

案例：若需新增 “SQL 语法检查” 功能，无需修改 MyBatis 内核（如 Executor 或 StatementHandler 的源码），只需开发一个拦截 StatementHandler.prepare() 方法的插件，在方法执行前校验 SQL 语法。插件移除时，仅需删除配置，对内核无任何影响，完美体现 “能力外置” 的微内核思想。

### 六、接口隔离 + 最小知识：聚焦必要，屏蔽复杂

**思想内涵**

*   **接口隔离原则（Interface Segregation Principle）**：客户端不应依赖其不需要的接口，应将庞大的接口拆分为多个细小、专用的接口，让客户端仅依赖自身所需的接口，减少不必要的依赖耦合。
    
*   **最小知识原则（Law of Demeter）**：一个对象应尽可能少地了解其他对象，仅与 “直接朋友”（如直接依赖的对象、方法参数中的对象）交互，避免暴露过多内部细节，降低系统复杂度。
    

两者结合，核心目标是**让插件在开发和使用过程中，仅关注 “必要的信息”，屏蔽框架内部的复杂细节**，降低插件开发的学习成本和维护难度。

**落地细节（结合 MyBatis 插件机制）**

MyBatis 插件机制通过 “接口设计” 和 “上下文封装”，严格遵循这两个原则：

*   **接口隔离：插件仅依赖四大核心接口，不依赖内部细节接口**：
    

1、插件开发时，仅需针对 Executor、StatementHandler、ParameterHandler、ResultSetHandler 这四大核心接口设计拦截逻辑，无需关注 MyBatis 内部的 SqlSource（SQL 源解析）、BoundSql（绑定 SQL 与参数）、MappedStatement（映射语句配置）等细节接口；

2、即使插件需要获取 SQL 信息（如修改 SQL 语句），也无需直接依赖 BoundSql 的创建逻辑，可通过 StatementHandler.getBoundSql() 方法间接获取，避免与内部细节强耦合。

*   **最小知识：通 过 Invocation 封装上下文，屏蔽反射细节** ：
    

1、插件执行拦截逻辑时，无需直接处理 “反射调用目标方法” 的复杂细节（如获取方法参数、处理异常），Invocation 类已封装目标对象、目标方法、方法参数，并提供 proceed() 方法简化调用；

2、插件开发者无需了解 “代理链如何嵌套”“下一个插件是谁” 等细节，只需调用 invocation.proceed() 即可触发后续流程，符合 “最小知识” 中 “仅与直接朋友交互” 的要求。

**具体体现（源码与案例）**

**案例一：分页插件的实现**

假设我们要实现一个通用分页功能，希望在不修改原有 SQL 的前提下,  自动加上 LIMIT offset, size 条件。

传统做法可能是封装一个工具类，在 DAO 层手动拼接 LIMIT。

但这种方式无法做到全局统一，且容易遗漏。

而在 MyBatis 插件体系中， 可以轻松拦截 StatementHandler.prepare() 方法，动态修改 SQL：

```
public Object intercept(Invocation invocation) throws Throwable {
    StatementHandler statementHandler = (StatementHandler) invocation.getTarget();
    BoundSql boundSql = statementHandler.getBoundSql();
    String originalSql = boundSql.getSql();
    // 添加分页条件
    String pageSql = originalSql + " LIMIT 0, 10";
    // 使用反射修改 BoundSql 中的 sql 字段
    Field sqlField = BoundSql.class.getDeclaredField("sql");
    sqlField.setAccessible(true);
    sqlField.set(boundSql, pageSql);
    return invocation.proceed();
}


```

请注意：在整个过程中，插件**仅通过 StatementHandler 接口获取 BoundSql**，并未直接依赖 SqlSource 或 XMLLanguageDriver 等解析组件。即便未来 MyBatis 更换了 SQL 解析引擎，只要 getBoundSql() 返回格式不变，插件依然可用。

这就是接口隔离的价值：**依赖抽象，而非实现**。

**案例二：性能监控插件的简化开发**

再来看一个更常见的需求：统计每条 SQL 的执行耗时。

如果没有良好的上下文封装，开发者可能需要手动保存开始时间、捕获异常、确保 finally 块中计算耗时…… 稍有疏忽就会导致内存泄漏或统计不准。

但在 MyBatis 中，这一切都被 Invocation 简化了：

```
public Object intercept(Invocation invocation) throws Throwable {
    long start = System.currentTimeMillis();
    try {
        // 只需一行 proceed()，无需关心底层反射细节
        return invocation.proceed();
    } finally {
        long cost = System.currentTimeMillis() - start;
        System.out.println("SQL 执行耗时：" + cost + "ms");
    }
}


```

甚至连异常都不用手动抛出——proceed() 方法内部已经做了封装。插件只需要专注自己的业务逻辑：记录时间差。

这种 “傻瓜式” 的开发体验，正是优秀框架的设计功力所在：**把复杂留给自己，把简单留给用户**。

### 七、约定优于配置 思想

**（1） 思想内涵**

在现代框架设计中，**“约定优于配置”** 已成为提升开发效率的核心设计范式之一。

尤其对于中高级开发者而言，理解这一思想不仅有助于快速掌握主流框架的工作机制，还能在面试中展现出对架构演进逻辑的深刻认知。

所谓 “约定优于配置”，本质上是一种通过**预设合理默认行为来减少显式配置**的设计哲学。

它的核心理念是：**框架提前定义一套通用且高频使用的规则（即 “约定”），只要开发者遵循这套规则，就可以零配置或极简声明完成功能实现；只有当实际需求偏离默认路径时，才需要通过配置进行定制化覆盖**。

举个典型的场景：在一个新项目中接入分页组件，如果每个 DAO 方法都要手动配置拦截目标、方法名、参数类型等信息，那无疑会带来巨大的维护成本和出错概率。

而如果框架规定 “所有名为 query 的方法都自动可被分页插件拦截”，开发者只需命名规范即可生效——这就体现了“约定” 的价值。

这种范式最直接的好处在于：**显著降低开发者的心智负担和编码成本**。

无需反复查阅文档配置标签，也不必编写大量 XML 或 Java Config 代码，让工程师能将注意力集中在真正的业务逻辑上，而不是陷在繁琐的基础设施配置里。这也是为什么 Spring Boot、MyBatis、Ruby on Rails 等框架都将 “约定优于配置” 作为核心设计理念的关键原因。

**（2）落地细节（结合 MyBatis 插件机制）**

以 MyBatis 的插件机制为例，其设计正是该范式的典型应用，尤其体现在**如何用注解替代复杂 XML 配置**这一点上。

在早期 ORM 框架中，扩展点往往依赖 XML 文件进行声明式配置，比如 Hibernate 的事件监听器、iBATIS 时代的拦截器注册等，都需要在配置文件中显式写出类名、方法签名甚至参数类型，既冗长又容易出错。

而 MyBatis 则另辟蹊径，引入了基于注解的拦截规则解析机制，实现了 “**注解即配置**” 的轻量级约定模型。

具体来说，MyBatis 框架默认约定：**只要一个类实现了 Interceptor 接口，并使用 @Intercepts 和 @Signature 注解正确标注拦截目标，就能自动被识别并加载为有效插件，无需任何 XML 注册或额外元数据配置**。

这意味着开发者不再需要关心 “在哪里写配置”“怎么写标签” 这类问题，只需要关注两个关键点：

*   我要拦截哪个组件？（如 Executor、StatementHandler）
    
*   拦截它的哪个方法？传参是什么？
    

而这正是 @Signature 注解所封装的内容。例如，设置 type = Executor.class, method = "query"，就相当于告诉框架：“请帮我拦截执行器的查询方法”。

整个过程完全符合 “**遵循约定 → 零配置生效**” 的设计逻辑。

**（3）具体体现（源码与案例）**

假设我们要开发一个简单的 **SQL 执行日志插件**，用于记录每次数据库操作的耗时和 SQL 语句。

按照传统思路，可能会想到去修改 XML 配置文件，添加一堆和标签。

但在 MyBatis 中，这一切都可以简化为一段注解声明：

```
@Intercepts({
    @Signature(
        type = Executor.class,
        method = "query",
        args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class}
    ),
    @Signature(
        type = Executor.class,
        method = "update",
        args = {MappedStatement.class, Object.class}
    )
})
public class LogPlugin implements Interceptor {
    // 插件逻辑：打印SQL、统计耗时等
}


```

看到这里你会发现：**整个插件定义过程中没有任何 XML 配置，也没有 Spring Bean 注册或其他初始化代码**。

只要这个类被成功扫描到并加入插件链，MyBatis 就会自动解析注解内容，构建出完整的拦截规则映射表。

那么这个 “自动解析” 到底是怎么发生的呢？

答案藏在 Plugin.getSignatureMap() 方法中。这是插件机制的核心入口之一，负责从拦截器类上提取出所有应拦截的方法签名。

我们来看关键源码片段：

```
private static Map<Class<?>, Set<Method>> getSignatureMap(Interceptor interceptor) {
    // 框架默认解析 @Intercepts 注解 —— 这就是“约定”的体现
    Intercepts interceptsAnnotation = interceptor.getClass().getAnnotation(Intercepts.class);
    if (interceptsAnnotation == null) {
        throw new PluginException("No @Intercepts annotation was found...");
    }
    // 解析注解中的每一个 @Signature，生成 Class -> Method 的映射
    Signature[] sigs = interceptsAnnotation.value();
    // 后续逻辑：构建拦截规则集合...
}


```

这段代码清晰地揭示了 “约定” 的执行流程：

**(1) 第一步：检查是否存在 @Intercepts 注解 —— 如果没有，直接抛异常，说明这不是一个合法插件；**

**(2) 第二步：读取其中的所有 @Signature 规则 —— 自动提取 type、method、args 字段；**

**(3) 第三步：生成内部映射结构 —— 用于后续动态代理时判断是否需要拦截。**

整个过程完全自动化，开发者无需干预解析逻辑，甚至连 “注册解析器” 这样的概念都不需要了解。

这就是 “**框架替你做了合理的默认选择**” 的最佳诠释。

反观如果没有这套约定会怎样？

我们可以设想一种 “无约定” 的极端情况：必须在 mybatis-config.xml 中手动配置每一个拦截规则，就像下面这样：

```
<plugins>
    <plugin class="com.example.LogPlugin">
        <signature type="Executor" method="query" 
                   args="MappedStatement,Object,RowBounds,ResultHandler"/>
        <signature type="Executor" method="update" 
                   args="MappedStatement,Object"/>
    </plugin>
</plugins>


```

虽然语法上看似清晰，但问题也很明显：

*   配置分散，难以维护；
    
*   参数列表需手动拼接字符串，极易出错；
    
*   新增一个拦截点就得改一次 XML，违反开闭原则；
    
*   IDE 无法校验方法是否存在、参数是否匹配，调试成本高。
    

相比之下，注解方式不仅语法简洁，更重要的是具备**编译期检查能力**：IDE 能提示错误、自动补全、跳转定义，极大提升了开发体验和稳定性。

### 八、 动态代理与责任链的协同设计

在深入剖析 MyBatis 插件机制时，我们常听到 “基于动态代理 + 责任链模式” 的说法。

但这只是表象——真正精妙之处在于，MyBatis 并非简单地将两种设计模式拼接在一起。

MyBatis 是通过**阶段化协同**的方式，把 “动态代理” 和“责任链”有机融合：**启动期完成责任链的装配（即代理链构建），运行期实现责任链的执行（即调用传递）**。

这种分阶段的设计，精准解决了插件系统中最核心的三大痛点：**执行顺序如何确定？调用流程如何传递？扩展逻辑如何保持灵活解耦？**

接下来，我们就从这两个关键阶段入手，层层剥开 MyBatis 插件背后的架构智慧。

#### 1. 启动期：责任链的静态装配 —— 用代理嵌套替代传统链表

大多数开发者理解的责任链模式，往往是通过 next 指针串联多个处理器，形成一条显式的调用链条。

但 MyBatis 并未采用这种方式，而是在框架初始化阶段，利用**循环代理包装**的技术手段，把插件列表转化为一个层层包裹的代理结构。

这不仅避免了维护链表指针的复杂性，还让整个责任链的构建过程变得简洁且可预测。

具体来看，这一装配过程可分为三个关键步骤：

首先，在配置解析阶段，XMLConfigBuilder 会读取  标签，逐一实例化用户定义的 Interceptor 实现类，并将其注册到全局的 InterceptorChain 中，存入一个有序的 interceptors 列表。

这个列表的存在，保证了插件的注册顺序是明确且可控的。

紧接着，在创建四大核心组件（如 Executor、StatementHandler 等）时，MyBatis 会调用 InterceptorChain.pluginAll(target) 方法，遍历所有已注册的插件。

每一轮都会通过 Plugin.wrap(target, interceptor) 对当前目标对象进行代理封装。

尼恩提示，这里的 target 并不总是原始对象——随着循环推进，它可能是前一个插件生成的代理实例。

于是，最终形成的结构是一个典型的 “代理套娃”：Proxy3(Proxy2(Proxy1(OriginalExecutor)))。

这种嵌套式代理结构，本质上就是一条隐式的责任链。

相比于传统链表需要手动管理 next 指针，MyBatis 借助 JDK 动态代理的机制，天然实现了调用的逐层穿透。

更重要的是，**插件的执行顺序完全由注册顺序决定**，无需额外配置或依赖注入，逻辑清晰直观，极大降低了使用成本和出错概率。

#### 2. 运行期：代理链的动态调用 —— 以 Invocation 驱动链式传递

一旦代理链构建完毕，真正的魔法便在每一次 SQL 执行中上演。

此时，最外层代理的 invoke() 方法将成为整个拦截流程的入口。

整个执行过程不再是简单的单点拦截，而是一场精心编排的 “洋葱式” 穿透调用。

当业务代码调用 executor.query(...) 时，实际触发的是最外层代理对象（比如 Proxy3）的 Plugin.invoke() 方法。这里的第一步是方法匹配：Plugin 会根据预先缓存的 signatureMap 判断当前调用的方法是否属于该插件的关注范围。

如果命中，则封装一个 Invocation 对象，交由对应插件的 intercept() 方法处理；否则直接放行，进入下一层。

Invocation 是整个调用链的核心载体，它封装了目标对象、被调用方法以及参数数组，使得插件无需关心底层反射细节，只需专注于业务增强逻辑。

而在插件内部，若希望继续向下传递调用，就必须显式调用 invocation.proceed() —— 这个方法就像一把钥匙，控制着是否进入内层代理或最终的目标方法。

一旦 proceed() 被调用，反射机制便会触发 target 上的同名方法。而这个 target 可能仍是某个代理实例，于是流程再次进入其 invoke() 方法，重复上述判断与拦截过程，直到抵达最内层的原始对象，执行真正的 SQL 查询逻辑。

待核心方法执行完成后，结果并不会立刻返回给调用方，而是沿着原路反向回溯：从最内层插件开始，依次执行各层的后置处理逻辑（例如修改结果集、记录耗时、打印日志等），最终回到最外层代理，将最终结果交付出去。

这种 “进栈 - 执行 - 出栈” 的调用模型，带来了极高的灵活性。尤其重要的是，proceed() 的存在赋予了插件中断调用的能力——例如权限校验类插件可在失败时直接抛异常或返回空值，从而阻断后续流程，实现类似 “过滤器” 的短路行为。

#### 3. 关键方法论：洋葱模型 —— 多层增强的理想范式

所谓 “洋葱模型”，是指整个代理链如同一颗洋葱，每一层插件都包裹在核心业务之外。

调用时，先由外向内逐层执行前置逻辑；到达中心后执行原始方法；再由内向外逐层执行后置逻辑。这种对称结构，完美支持了对同一方法的多维度、分层次的功能增强。

举个例子，假设我们注册了三个插件：

*   最外层是日志插件（Plugin3），负责记录请求入口与出口；
    
*   中间是监控插件（Plugin2），用于统计 SQL 执行时间；
    
*   最内层是数据脱敏插件（Plugin1），在结果返回前对敏感字段进行掩码处理。
    

那么在一次查询过程中，前置逻辑的执行顺序为：

Plugin3.before() → Plugin2.before() → Plugin1.before()

接着执行原始 query() 方法获取结果；

随后后置逻辑按相反顺序执行：

Plugin1.after() → Plugin2.after() → Plugin3.after()

你会发现，每个插件都能完整掌控 “进入” 和“离开”两个时机，彼此独立又协同工作。

你可以随时添加、移除或调整插件顺序，而不会影响其他模块的正常运行。这种松耦合、高内聚的特性，正是 “插件化” 架构追求的理想状态。

更进一步地说，洋葱模型的本质是一种**环绕式增强（around advice）**，它比前置通知（before）或后置通知（after）更具表达力。MyBatis 正是借助代理链与 Invocation 的组合，实现了轻量级但功能完整的环绕拦截能力，而无需引入复杂的 AOP 框架。

综上所述，MyBatis 插件系统的强大，并不在于用了多少高深的技术术语，而在于它用最朴素的 JDK 动态代理，结合责任链的思想，构造出了一套**启动期装配、运行期执行、结构清晰、易于扩展**的插件机制。

对于开发者而言，理解这套机制不仅能帮助你在面试中脱颖而出，更能让你在实际项目中自信地编写高性能、低侵入的拦截逻辑——无论是做 SQL 审计、性能监控，还是实现租户隔离、数据加密，都能游刃有余。

#### 4：架构思想与方法论的本质

MyBatis 插件机制的本质，是通过 “动态代理 + 责任链” 的协同设计，将**横切关注点（如日志、监控、安全）与核心业务逻辑（SQL 执行）分离**，同时通过 “接口化定义、阶段化装配、链式执行” 的方法论，实现 “扩展灵活、核心稳定” 的架构目标。

其设计的精妙之处在于：

**(1) 不引入复杂的容器或配置，仅通过简单的组件（InterceptorChain、Plugin、Invocation）实现插件生态；**

**(1) 动态代理与责任链并非孤立存在，而是通过 “代理嵌套” 和 “proceed() 传递” 深度融合，形成高效的扩展机制；**

**(1) 始终围绕 “最小侵入性” 和 “单一职责”，确保核心流程简洁，同时为开发者提供足够灵活的扩展能力。**

这种架构思想不仅适用于 MyBatis 插件，也为其他框架的扩展机制设计提供了参考（如 Spring AOP 的 “代理 + 切面”、Servlet Filter 的 “责任链”），是 “经典设计原则落地实践” 的优秀案例。

## MyBatis  核心架构思想  大总结

MyBatis 插件机制是经典架构思想落地的典范。

其核心通过 “动态代理 + 责任链” 的协同设计，将开闭原则、单一职责、最小侵入性等理念深度整合。

框架以微内核架构聚焦 SQL 映射核心流程，而将日志、分页、加密等横切关注点通过标准化 Interceptor 接口外置为插件，实现了核心稳定与扩展灵活的平衡。

借助 “约定优于配置” 的注解声明与运行时代理链的洋葱模型，插件可无侵入地织入增强逻辑。

这种设计不仅保证了框架的高内聚、低耦合，更提供了高度可扩展的生态基础，成为中大型项目基础层建设的优秀参考范式。

## 遇到问题，找老架构师取经

借助此文的问题 套路 ，大家可以 **暴击面试官**，**保证 offer 直接到手，还有可能会 涨薪  100%-200%。**

后面，尼恩 java 面试宝典回录成视频， 给大家打造一套进大厂的塔尖视频。

在面试之前，建议大家系统化的刷一波 5000 页《[尼恩 Java 面试宝典 PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》，里边有大量的大厂真题、面试难题、架构难题。

很多小伙伴刷完后， 吊打面试官， 大厂横着走。

在刷题过程中，如果有啥问题，大家可以来 找 40 岁老架构师尼恩交流。

另外，如果没有面试机会，**可以找尼恩来改简历、做帮扶。**

遇到职业难题，找老架构取经， 可以省去太多的折腾，省去太多的弯路。

尼恩指导了大量的小伙伴上岸，前段时间，**[刚指导 32 岁 高中生，冲大厂成功。特批 成为 架构师，年薪 50W，逆天改命 ！！！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486848&idx=1&sn=ff43d058271b0801f84aba3f3d957855&scene=21#wechat_redirect)。**

狠狠卷，实现 “offer 自由” 很容易的， 前段时间一个武汉的跟着尼恩卷了 2 年的小伙伴， 在极度严寒 / 痛苦被裁的环境下， offer 拿到手软， 实现真正的 “offer 自由” 。

[会 AI 的程序员，工资暴涨 50%！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486827&idx=1&sn=db3f268c6752ddb074a98c7b13a84011&scene=21#wechat_redirect)

## 跟着 45 岁老架构 尼恩  狠狠卷 硬核技术 ， 占据  领先地位， 实现 逆天改命

 [会 AI 的程序员，工资暴涨 50%！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486827&idx=1&sn=db3f268c6752ddb074a98c7b13a84011&scene=21#wechat_redirect)

  

[奇迹 :  一年 涨 2 倍，   年薪 60W 梦想实现  。 接下来，开启 40 岁之前的 年薪  200W 梦想](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247487014&idx=1&sn=5d1359a7adc7c9e46e6374597bb03138&scene=21#wechat_redirect)

  

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

![](https://mmbiz.qpic.cn/mmbiz_jpg/9qY2Gt3PBs3icE6WcM4nBOdRRcczOW4wkNDgCMZhBfqeet9suwGHfgQduP33YYqqZLOppecMI72Pv5ibNkPeIk6g/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=1)

关注**职业救助站**公众号，获取每天职业干货  
助您实现**职业转型、职业升级、极速上岸**  
---------------------------------

**技术自由圈**

实现架构转型，再无中年危机

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/xlgvgPaib7WMtyFcJx45oIeWL1d6AENcdoAvuTPuq46E9n8pPe4SHXoSicuEVvhibHRSIzmEBPlC7POcDaqD6ID7w/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=2)

关注**技术自由圈**公众号，获取每天技术千货  
一起成为牛逼的**未来超级架构师**

**几十篇架构笔记、5000 页面试宝典、20 个技术圣经  
请加尼恩个人微信 免费拿走**

**暗号，请在 公众号后台 发送消息：领电子书**

如有收获，请点击底部的 " 在看 "和" 赞 "，谢谢