#知识大纲 #Spring

# 1. Spring 大纲

![](https://r2.129870.xyz/img/20220504152622.png)

- BeanFactory、FactoryBean、ApplicationContext 区别
	1. ClassPathXmlApplicaitonContext
	2. FileSystemXmlApplicationContext
	3. XmlWebApplicationContext
- Spring 框架中用到的设计模式
	- 简单工厂模式
	- 工厂方法模式
	- 代理模式
	- 模版模式
	- 责任链模式
	- 观察者模式
	- 迭代器模式
	- 适配器模式
- Spring 中 Bean 注入的方式
	- Setter 方法注入
	- 构造器注入
	- 静态工厂注入
	- 实例工厂
- Bean 的作用域
	- Singleton
	- Prototype
	- Request
	- Session
	- GlobalSession
- SpringMVC 请求的处理
	1. 请求到达 DIspatcherServler
	2. 转发到 HandlerMapping
	3. 找到 HandlerAdapter
	4. Handler 执行返回 ModelAndView
	5. 到达 ViewResolver
	6. 解析返回 View 对象
	7. 视图渲染
- AOP 通知类型
	- Before
	- After
	- Afterreturning
	- Afterthroing
	- Around
- SpringBean 的生命周期
	1. 实例化 bean
	2. 依赖注入
	3. 处理 Aware 接口 (BeanNameAware、BeanFactoryAware、ApplicationContextAware)
	4. BeanPostProcessor-postpostProcessBeforeInitialization(before)
	5. InitializingBean
	6. Init-method
	7. BeanPostProcessor-postpostProcessBeforeInitialization (after)
	8. Bean 的使用
	9. DIsposableBean
	10. Destory-method
- Spring 循环依赖的解决，三级缓存
	1. SingleObjects
	2. EarlySingleObjects
	3. SingleFactories
- 事务的传播级别
	1. Required
	2. RequiredNew
	3. Nested(子事务异常，父事务不会滚。父事务异常，子事务回滚)
	4. Support
	5. NotSupported
	6. Mandatory
	7. Never
- 事务三要素
	1. 执行侧：数据源
	2. 管理侧：事务管理器
	3. 配置侧：事务应用与属性配置
