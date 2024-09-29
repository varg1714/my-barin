# 1. Spring 介绍

![](https://r2.129870.xyz/img/20220407230008.png)

## 1.1. Spring 各个模块功能

![](https://r2.129870.xyz/img/20220407230544.png)
![](https://r2.129870.xyz/img/20220407230559.png)
![](https://r2.129870.xyz/img/20220407230613.png)
![](https://r2.129870.xyz/img/20220407230625.png)
![](https://r2.129870.xyz/img/20220407230639.png)
![](https://r2.129870.xyz/img/20220407230710.png)

## 1.2. Spring 各个模块的关系

![](https://r2.129870.xyz/img/20220407230726.png)

# 2. 300 代码行手写 Spring

Spring 的执行流程:
![](https://r2.129870.xyz/img/20220410200400.png)

## 2.1. 30 个类手写 Spring

### 2.1.1. IOC 核心组件

![](https://r2.129870.xyz/img/20220416222047.png)

1. Application 是 IOC 与 DISpring 的核心。它负责初始化 Spring 的 bean 并对其管理进行 DI 操作。
2. BeanDefinitionReader 负责从配置中加载 Spring 的 Bean 生命，将其封装为 BeanDefinition 对象。
3. BeanDefinition 对象是初始的 Bean 声明，当对 Bean 进行了代理等操作后，会生成相关的代理类，该代理类就是 Spring 运行过程中实际运行的类。该类由 BeanWrapper 类管理。

### 2.1.2. MVC 核心组件

![](https://r2.129870.xyz/img/20220416222701.png)

1. HandlerMapping 负责将 URL 与对应的处理方法做映射。
2. HandlerAdapter 负责将 http 参数动态解析到 Java 方法中，同时返回 ModelAndView 对象，该对象是视图部分逻辑的封装。
3. 由 ViewResolver 负责解析视图，将其呈现到页面上。

### 2.1.3. AOP 核心组件

![](https://r2.129870.xyz/img/20220416223042.png)

1. AdvisiedSupport 组件负责读取 AOP 配置，将其处理为 Advice 代理对象。
2. JdkDynamicAopProxy 组件负责实现动态代理部分，生成代理对象交由 Spring 管理。
