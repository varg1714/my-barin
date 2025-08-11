#知识大纲 #Spring

## 1. 基础概念

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
- Intercepter 和 Filter 的区别
    ![1.png|100*100](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYmYLtVQiatSUtz0EscuvicdGoUsSGsZzmzc5Z15682LkzU16biciaRYldBOltE3GZtWUrCh4SOGn8dgicA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)
    
    <table><thead><tr><th><section><span leaf=""><span textstyle="">特性</span></span></section></th><th><section><span leaf=""><span textstyle="">拦截器 (Interceptor)</span></span></section></th><th><section><span leaf=""><span textstyle="">过滤器 (Filter)</span></span></section></th></tr></thead><tbody><tr><td><section><span leaf=""><span textstyle="">容器依赖</span></span></section></td><td><section><span leaf=""><span textstyle="">Spring 容器</span></span></section></td><td><section><span leaf=""><span textstyle="">Servlet 容器</span></span></section></td></tr><tr><td><section><span leaf=""><span textstyle="">作用范围</span></span></section></td><td><section><span leaf=""><span textstyle="">Spring MVC 处理的请求</span></span></section></td><td><section><span leaf=""><span textstyle="">所有进入容器的请求</span></span></section></td></tr><tr><td><section><span leaf=""><span textstyle="">访问对象</span></span></section></td><td><section><span leaf=""><span textstyle="">可以访问 Handler 和方法信息</span></span></section></td><td><section><span leaf=""><span textstyle="">只能访问 ServletRequest/Response</span></span></section></td></tr><tr><td><section><span leaf=""><span textstyle="">执行时机</span></span></section></td><td><section><span leaf=""><span textstyle="">在 DispatcherServlet 内部执行</span></span></section></td><td><section><span leaf=""><span textstyle="">在 DispatcherServlet 外部执行</span></span></section></td></tr><tr><td><section><span leaf=""><span textstyle="">依赖注入</span></span></section></td><td><section><span leaf=""><span textstyle="">支持</span></span></section></td><td><section><span leaf=""><span textstyle="">不支持</span></span></section></td></tr></tbody></table>
- Spring 的默认跨域配置
    - 实现 WebMvcConfigurer.addCorsMappings 方法来进行的 CORS 配置，最后会在 Spring 的 Interceptor 或 Handler 中生效
    - 注入 CorsFilter 的方式会让 CORS 验证在 Filter 中生效
    - 引入 Spring Security 后，需要调用 HttpSecurity.cors 方法以保证 CorsFilter 会在身份验证相关的 Filter 之前执行
    - HttpSecurity.cors + WebMvcConfigurer.addCorsMappings 是一种相对低效的方式，会导致跨域请求分别在 Filter 和 Interceptor 层各经历一次 CORS 验证
    - HttpSecurity.cors + 注册 CorsFilter 与 HttpSecurity.cors + 注册 CorsConfigurationSource 在运行的时候是等效的
    - 在 Spring 中，没有通过 CORS 验证的请求会得到状态码为 403 的响应

<think>undefined 用户提到了代码中的 Java 语法不是 Java 8 的，让我检查一下原文中的代码示例。

我看到原文中有 Kotlin 的代码语法，比如：

```kotlin
@RestController
class HelloController {
    @GetMapping("hello")
    fun hello(): String {
        return "Hello, CORS!"
    }
}
```

还有：

```kotlin
@Configuration
@EnableWebMvc
class MvcConfig: WebMvcConfigurer {
    override fun addCorsMappings(registry: CorsRegistry) {
        registry.addMapping("/hello")
                .allowedOrigins(" http://localhost:8080")
    }
}
```

这些确实是 Kotlin 语法，不是 Java。我需要将这些代码转换为 Java 8 的语法。</think>根据[[Spring Boot 中的跨域，为什么加了 Spring Security 就失效了呢？]]这篇文章，我为你梳理出以下核心内容：

## 2. 使用相关

### 2.1. 跨域配置

引入 Spring Security 后，原本配置好的跨域 CORS 会失效，预检请求（preflight request）会返回 401 未授权错误。

#### 2.1.1. CORS 基础概念

**CORS（Cross-Origin Resource Sharing）**：跨域资源共享
- **域**：由 protocol、host、port 三部分组成
- **资源**：URL 对应的任何内容（图片、字体、HTML、JSON 等）
- **同源策略**：浏览器安全机制，防止 XSS 攻击

**预检请求（Preflight Request）**：
- 非简单请求会先发起一次 OPTIONS 请求
- 携带三个关键 headers：Origin、Access-Control-Request-Method、Access-Control-Request-Headers
- 验证失败返回 4xx，成功返回 200 并带上 CORS 响应头

#### 2.1.2. Spring Boot 中的三种 CORS 配置方式

1. @CrossOrigin 注解

    ```java
    @RestController
    public class HelloController {
        @GetMapping("hello")
        @CrossOrigin(origins = {"http://localhost:8080"})
        public String hello() {
            return "Hello, CORS!";
        }
    }
    ```

2. WebMvcConfigurer.addCorsMappings 方法

    ```java
    @Configuration
    @EnableWebMvc
    public class MvcConfig implements WebMvcConfigurer {
        @Override
        public void addCorsMappings(CorsRegistry registry) {
            registry.addMapping("/hello")
                    .allowedOrigins("http://localhost:8080");
        }
    }
    ```

3. 注入 CorsFilter

    ```java
    @Configuration
    public class CORSConfiguration {
        @Bean
        public CorsFilter corsFilter() {
            CorsConfiguration configuration = new CorsConfiguration();
            configuration.setAllowedOrigins(Arrays.asList("http://localhost:8080"));
            UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
            source.registerCorsConfiguration("/hello", configuration);
            return new CorsFilter(source);
        }
    }
    ```

#### 2.1.3. 与 Spring Security 的冲突

引入 Spring Security 后需要额外配置：

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.cors(); // 关键配置
    }
}
```

或者注入 CorsConfigurationSource：

```java
@Bean
public CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration configuration = new CorsConfiguration();
    configuration.setAllowedOrigins(Arrays.asList("http://localhost:8080"));
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/hello", configuration);
    return source;
}
```

#### 2.1.4. 实现原理分析

1.  Filter 与 Interceptor 的执行顺序
    Filter → DispatcherServlet → Interceptor → Handler
2.  为什么 Spring Security 会导致跨域失效？
    1. 预检请求不携带认证信息
    2. Spring Security 通过 Filter 进行身份验证
    3. Interceptor 在 DispatcherServlet 之后执行
    4. Spring Security 的 Filter 优先级比普通 CorsFilter 高
3. HttpSecurity.cors() 的解决方案
    - 在 Spring Context 中查找 CorsFilter 或 CorsConfigurationSource
    - 将找到的 CorsFilter 添加到 Spring Security 的过滤器链中
    - CorsFilter 在过滤器链中排第 6 位，在所有安全验证 Filter 之前执行

#### 2.1.5. 配置方式对比与建议

| 配置方式 | 生效层级 | 性能 | 推荐度 |
|---------|---------|------|--------|
| @CrossOrigin + HttpSecurity.cors() | Interceptor/Handler | 一般 | ⭐⭐⭐ |
| WebMvcConfigurer + HttpSecurity.cors() | Filter + Interceptor | 较低（重复验证） | ⭐⭐ |
| CorsFilter + HttpSecurity.cors() | Filter | 高 | ⭐⭐⭐⭐⭐ |
| CorsConfigurationSource + HttpSecurity.cors() | Filter | 高 | ⭐⭐⭐⭐⭐ |

##### 2.1.5.1. 最佳实践总结

1. 引入 Spring Security 后必须调用 `HttpSecurity.cors()`
2. 推荐使用 CorsFilter 或 CorsConfigurationSource 方式，避免重复验证
3. CORS 验证失败会返回 403 Forbidden 状态码
4. 预检请求会被专门的 PreFlightHandler 处理
5. 实际请求通过 CorsInterceptor 进行验证

##### 2.1.5.2. 完整配置示例

```java
// 推荐方式一：CorsConfigurationSource + HttpSecurity.cors()
@Configuration
public class CorsConfig {
    
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOriginPatterns(Arrays.asList("*"));
        configuration.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE", "OPTIONS"));
        configuration.setAllowedHeaders(Arrays.asList("*"));
        configuration.setAllowCredentials(true);
        configuration.setMaxAge(3600L);
        
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
}

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.cors()
            .and()
            .csrf().disable()
            .authorizeRequests()
            .anyRequest().authenticated();
    }
}
```