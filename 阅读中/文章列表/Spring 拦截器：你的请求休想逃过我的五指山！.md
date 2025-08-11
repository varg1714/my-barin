---
source: https://mp.weixin.qq.com/s/5_rsVC7Oh8amdxC90W7W_A
create: 2025-08-11 14:08
read: false
knowledge: false
---
### 拦截器概述

在 Spring 框架中，拦截器 (Interceptor) 是一种强大的机制，它允许开发者在请求处理的不同阶段插入自定义逻辑。WebApplicationContext 作为 Spring Web 应用的上下文容器，为拦截器的配置和管理提供了基础支持。

拦截器主要作用于以下场景：  
权限验证  
日志记录  
性能监控  
事务管理  
通用行为注入等

### 拦截器与 WebApplicationContext 的关系

WebApplicationContext 是 Spring Web 应用的 IoC 容器扩展，它继承自 ApplicationContext，并添加了 Web 应用特有的功能。拦截器通过 WebApplicationContext 进行注册和管理，成为请求处理管道的一部分。

```
public interface WebApplicationContext extends ApplicationContext {
    String ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE = WebApplicationContext.class.getName() + ".ROOT";
    ServletContext getServletContext();
}
```

### 拦截器类型

#### HandlerInterceptor

最常用的拦截器接口，定义了三个关键方法：

```
public interface HandlerInterceptor {
    default boolean preHandle(HttpServletRequest request, 
                             HttpServletResponse response, 
                             Object handler) throws Exception {
        return true;
    }
    default void postHandle(HttpServletRequest request, 
                          HttpServletResponse response, 
                          Object handler,
                          ModelAndView modelAndView) throws Exception {
    }
    default void afterCompletion(HttpServletRequest request, 
                               HttpServletResponse response, 
                               Object handler, 
                               Exception ex) throws Exception {
    }
}
```

```
Java配置方式
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoggingInterceptor())
                .addPathPatterns("/**")
                .excludePathPatterns("/static/**");
        registry.addInterceptor(new AuthInterceptor())
                .addPathPatterns("/admin/**");
    }
}
```

HandlerInterceptor 的扩展，增加了异步处理的支持。

#### WebRequestInterceptor

与 HandlerInterceptor 类似，但提供了更通用的 WebRequest 抽象，不依赖于 Servlet API。

### 拦截器配置

```
@Component
public class MyInterceptor implements HandlerInterceptor {
    // 实现方法
}
@Configuration
public class InterceptorConfig {
    @Autowired
    private MyInterceptor myInterceptor;
    @Bean
    public WebMvcConfigurer adapter() {
        return new WebMvcConfigurer() {
            @Override
            public void addInterceptors(InterceptorRegistry registry) {
                registry.addInterceptor(myInterceptor);
            }
        };
    }
}
```

```
registry.addInterceptor(new InterceptorA()).order(1);
registry.addInterceptor(new InterceptorB()).order(2)
```

```
public class LoggingInterceptor implements HandlerInterceptor {
    private static final Logger logger = LoggerFactory.getLogger(LoggingInterceptor.class);
    @Override
    public boolean preHandle(HttpServletRequest request, 
                           HttpServletResponse response, 
                           Object handler) {
        long startTime = System.currentTimeMillis();
        request.setAttribute("startTime", startTime);
        logger.info("Request URL: {} : Start Time={}", 
                   request.getRequestURL(), startTime);
        return true;
    }
    @Override
    public void afterCompletion(HttpServletRequest request, 
                               HttpServletResponse response, 
                               Object handler, 
                               Exception ex) {
        long startTime = (Long) request.getAttribute("startTime");
        long endTime = System.currentTimeMillis();
        logger.info("Request URL: {} : End Time={} : Time Taken={}ms", 
                   request.getRequestURL(), endTime, (endTime - startTime));
    }
}
```

### 拦截器执行流程

拦截器在 DispatcherServlet 的处理流程中扮演重要角色：

*   preHandle：在处理器执行前调用
    

返回 true 继续执行

返回 false 中断请求处理

*   postHandle：在处理器执行后，视图渲染前调用
    

可修改 ModelAndView

*   afterCompletion：在完整请求完成后调用
    

适合资源清理

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BWVJb1DCVIDJBGiaicHuocebicQRsnnyAhMV95aRhZKLSg6ffFv7unXEfFF50AYlhzQCes6RicXFj6FnQ/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1)

### 高级拦截器特性

#### 拦截器顺序控制

可以通过 order 属性控制多个拦截器的执行顺序：

```
public class AuthInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request,
                            HttpServletResponse response,
                            Object handler) throws Exception {
        HttpSession session = request.getSession();
        if (session.getAttribute("user") == null) {
            response.sendRedirect("/login");
            return false;
        }
        return true;
    }
}
```

```
路径匹配模式
```

支持 Ant 风格的路径模式：

1. 匹配一个字符

2. 匹配零个或多个字符

3. 匹配零个或多个目录

#### 异步请求处理

对于异步请求，afterConcurrentHandlingStarted 方法会被调用而不是 postHandle 和 afterCompletion。

### 拦截器与过滤器的区别

<table><thead><tr><th><section><span leaf=""><span textstyle="">特性</span></span></section></th><th><section><span leaf=""><span textstyle="">拦截器 (Interceptor)</span></span></section></th><th><section><span leaf=""><span textstyle="">过滤器 (Filter)</span></span></section></th></tr></thead><tbody><tr><td><section><span leaf=""><span textstyle="">容器依赖</span></span></section></td><td><section><span leaf=""><span textstyle="">Spring 容器</span></span></section></td><td><section><span leaf=""><span textstyle="">Servlet 容器</span></span></section></td></tr><tr><td><section><span leaf=""><span textstyle="">作用范围</span></span></section></td><td><section><span leaf=""><span textstyle="">Spring MVC 处理的请求</span></span></section></td><td><section><span leaf=""><span textstyle="">所有进入容器的请求</span></span></section></td></tr><tr><td><section><span leaf=""><span textstyle="">访问对象</span></span></section></td><td><section><span leaf=""><span textstyle="">可以访问 Handler 和方法信息</span></span></section></td><td><section><span leaf=""><span textstyle="">只能访问 ServletRequest/Response</span></span></section></td></tr><tr><td><section><span leaf=""><span textstyle="">执行时机</span></span></section></td><td><section><span leaf=""><span textstyle="">在 DispatcherServlet 内部执行</span></span></section></td><td><section><span leaf=""><span textstyle="">在 DispatcherServlet 外部执行</span></span></section></td></tr><tr><td><section><span leaf=""><span textstyle="">依赖注入</span></span></section></td><td><section><span leaf=""><span textstyle="">支持</span></span></section></td><td><section><span leaf=""><span textstyle="">不支持</span></span></section></td></tr></tbody></table>

### 实际应用示例

#### 日志拦截器

```
public class LoggingInterceptor implements HandlerInterceptor {
    private static final Logger logger = LoggerFactory.getLogger(LoggingInterceptor.class);
    @Override
    public boolean preHandle(HttpServletRequest request, 
                           HttpServletResponse response, 
                           Object handler) {
        long startTime = System.currentTimeMillis();
        request.setAttribute("startTime", startTime);
        logger.info("Request URL: {} : Start Time={}", 
                   request.getRequestURL(), startTime);
        return true;
    }
    @Override
    public void afterCompletion(HttpServletRequest request, 
                               HttpServletResponse response, 
                               Object handler, 
                               Exception ex) {
        long startTime = (Long) request.getAttribute("startTime");
        long endTime = System.currentTimeMillis();
        logger.info("Request URL: {} : End Time={} : Time Taken={}ms", 
                   request.getRequestURL(), endTime, (endTime - startTime));
    }
}
```

#### 认证拦截器

```
public class AuthInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request,
                            HttpServletResponse response,
                            Object handler) throws Exception {
        HttpSession session = request.getSession();
        if (session.getAttribute("user") == null) {
            response.sendRedirect("/login");
            return false;
        }
        return true;
    }
}
```

### DEMO 实测效果

被拦截  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BWVJb1DCVIDJBGiaicHuocebicD6vM0Q3ib9ictVmn3lhv96MKuT2srFEq2LZwiahica9OWmhoeNDFcFFIbw/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1)

未被拦截  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BWVJb1DCVIDJBGiaicHuocebiccibic7899E1y7bqhDsUL1ic8AEvZs8awarBGotiaIhic3ObXmAV2vsWBuWA/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXGicp40bMAicmX9DpEDjMlfPJT23acLpRzmuyiaguHv0VlmVDyEFGwd36gZYRShzhv0EPleicHyvk7KA/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&randomid=ivk0xplu&tp=wxpic)

扫一扫，加入技术交流群