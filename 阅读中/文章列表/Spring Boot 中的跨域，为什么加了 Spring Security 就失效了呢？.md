---
source: https://mp.weixin.qq.com/s/ATzkEaRe9GlPVnM_W0Wt3g?poc_token=HM2ImWij0LuxaU5IJ1t1u9OXSu3IGEOufTnQ6xkU
create: 2025-08-11 14:08
read: false
knowledge: false
---
![](https://mmbiz.qpic.cn/mmbiz_jpg/GvtDGKK4uYlxaXicHAcv5XXhpA0Vo918DllviaVV8fUW7OmwbibozhPssvqcMrKdhBIvYmc1Xz4T5eteKHRruTRcA/640?wx_fmt=jpeg)

本文地址：sf.gg/a/1190000019485883

松哥原创的 Spring Boot 视频教程已经杀青，感兴趣的小伙伴戳这里 -->[Spring Boot+Vue + 微人事视频教程](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

* * *

关于 Spring Boot 中的跨域问题，松哥之前写过一篇文章：[Spring Boot 中三种跨域场景总结](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247488989&idx=1&sn=00881de1a77c2e4027ee948164644485&scene=21#wechat_redirect)，不过当时只是写了用法，没有讲原理，其实原理也不难，最近刚好看到一篇文章很不错，和大家分享一下。

松哥稍微总结一下文章内容：

1.  Spring Boot 中处理跨域有三种方式，@CrossOrigin 注解、WebMvcConfigurer.addCorsMappings 以及 CorsFilter。
    
2.  前两种本质上通过 CorsInterceptor 触发调用，后者是一个普通的 Filter。
    
3.  Filter 先于 DispatcherServlet 执行，DispatcherServlet 先于 Interceptor 执行。
    
4.  引入 Spring Security 后，Spring Security 过滤器会拦截下跨域的预检请求，此时只要放对 CorsFilter 的位置使其先于 Spring Security 过滤器执行，就能解决问题。
    

以下是正文：

[作为一个后端开发，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们经常遇到的一个问题就是需要配置 CORS，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[好让我们的前端能够访问到我们的 API，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[并且不让其他人访问。而在 Spring 中，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们见过很多种 CORS 的配置，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[很多资料都只是告诉我们可以这样配置、可以那样配置，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[但是这些配置有什么区别？](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

## 1.CORS 是什么

[首先我们要明确，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[CORS 是什么，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[以及规范是如何要求的。这里只是梳理一下流程。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

[CORS 全称是 Cross-Origin Resource Sharing，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[直译过来就是跨域资源共享。要理解这个概念就需要知道域、资源和同源策略这三个概念。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

*   [域，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[指的是一个站点，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[由 protocal、host 和 port 三部分组成，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[其中 host 可以是域名，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[也可以是 ip ；port 如果没有指明，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[则是使用 protocal 的默认端口.](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
*   [资源，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[是指一个 URL 对应的内容，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[可以是一张图片、一种字体、一段 HTML 代码、一份 JSON 数据等等任何形式的任何内容。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
*   [同源策略，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[指的是为了防止 XSS，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[浏览器、客户端应该仅请求与当前页面来自同一个域的资源，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[请求其他域的资源需要通过验证。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    

[了解了这三个概念，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们就能理解为什么有 CORS 规范了：从站点 A 请求站点 B 的资源的时候，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[由于浏览器的同源策略的影响，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[这样的跨域请求将被禁止发送；为了让跨域请求能够正常发送，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们需要一套机制在不破坏同源策略的安全性的情况下、允许跨域请求正常发送，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[这样的机制就是 CORS。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

## 2. 预检请求

[在 CORS 中，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[定义了一种预检请求，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[即 preflight request，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[当实际请求不是一个 简单请求 时，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[会发起一次预检请求。预检请求是针对实际请求的 URL 发起一次 OPTIONS 请求，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[并带上下面三个 headers ：](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

*   [Origin：值为当前页面所在的域，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[用于告诉服务器当前请求的域。如果没有这个 header，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)服务器将不会进行 CORS 验证。
    
*   Access-Control-Request-Method：值为实际请求将会使用的方法。
    
*   Access-Control-Request-Headers：值为实际请求将会使用的 header 集合。
    

[如果服务器端 CORS 验证失败，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[则会返回客户端错误，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[即 4xx 的状态码。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

[否则，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[将会请求成功，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[返回 200 的状态码，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[并带上下面这些 headers：](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

*   [Access-Control-Allow-Origin：允许请求的域，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[多数情况下，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)就是预检请求中的 Origin 的值。
    
*   [Access-Control-Allow-Credentials：一个布尔值，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)表示服务器是否允许使用 cookies。
    
*   Access-Control-Expose-Headers：实际请求中可以出现在响应中的 headers 集合。
    
*   [Access-Control-Max-Age：预检请求返回的规则可以被缓存的最长时间，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[超过这个时间，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)需要再次发起预检请求。
    
*   [Access-Control-Allow-Methods：实际请求中可以使用到的方法集合浏览器会根据预检请求的响应，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)来决定是否发起实际请求。
    

### 2.1 小结

[到这里，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect) [我们就知道了跨域请求会经历的故事：](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

1.  [访问另一个域的资源。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
2.  [有可能会发起一次预检请求（非简单请求，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[或超过了 Max-Age）。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
3.  [发起实际请求。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    

[接下来，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们看看在 Spring 中，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们是如何让 CORS 机制在我们的应用中生效的。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

## 3. 三种配置的方式

[Spring 提供了多种配置 CORS 的方式，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[有的方式针对单个 API，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[有的方式可以针对整个应用；有的方式在一些情况下是等效的，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[而在另一些情况下却又出现不同。我们这里例举几种典型的方式来看看应该如何配置。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

[假设我们有一个 API：](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

```
@RestControllerclass HelloController {    @GetMapping("hello")    fun hello(): String {        return "Hello, CORS!"    }}
```

### 3.1 @CrossOrigin 注解

[使用 @CorssOrigin 注解需要引入 Spring Web 的依赖，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[该注解可以作用于方法或者类，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[可以针对这个方法或类对应的一个或多个 API 配置 CORS 规则：](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

```
@RestControllerclass HelloController {    @GetMapping("hello")    @CrossOrigin(origins = ["http://localhost:8080"])    fun hello(): String {        return "Hello, CORS!"    }}
```

### 3.2 实现 WebMvcConfigurer.addCorsMappings 方法

[WebMvcConfigurer 是一个接口，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[它同样来自于 Spring Web。我们可以通过实现它的 addCorsMappings 方法来针对全局 API 配置 CORS 规则：](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

```
@Configuration@EnableWebMvcclass MvcConfig: WebMvcConfigurer {    override fun addCorsMappings(registry: CorsRegistry) {        registry.addMapping("/hello")                .allowedOrigins("http://localhost:8080")    }}
```

### 3.3 注入 CorsFilter

[CorsFilter 同样来自于 Spring Web，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[但是实现 WebMvcConfigurer.addCorsMappings 方法并不会使用到这个类，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[具体原因我们后面来分析。我们可以通过注入一个 CorsFilter 来使用它：](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

```
@Configurationclass CORSConfiguration {    @Bean    fun corsFilter(): CorsFilter {        val configuration = CorsConfiguration()        configuration.allowedOrigins = listOf("http://localhost:8080")        val source = UrlBasedCorsConfigurationSource()        source.registerCorsConfiguration("/hello", configuration)        return CorsFilter(source)    }}
```

[注入 CorsFilter 不止这一种方式，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们还可以通过注入一个 FilterRegistrationBean 来实现，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[这里就不给例子了。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

[在仅仅引入 Spring Web 的情况下，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[实现 WebMvcConfigurer.addCorsMappings 方法和注入 CorsFilter 这两种方式可以达到同样的效果，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[二选一即可。它们的区别会在引入 Spring Security 之后会展现出来，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们后面再来分析。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

## 4.Spring Security 中的配置

[在引入了 Spring Security 之后，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们会发现前面的方法都不能正确的配置 CORS，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[每次 preflight request 都会得到一个 401 的状态码，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[表示请求没有被授权。这时，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们需要增加一点配置才能让 CORS 正常工作：](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

```
@Configurationclass SecurityConfig : WebSecurityConfigurerAdapter() {    override fun configure(http: HttpSecurity?) {        http?.cors()    }}
```

[或者，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[干脆不实现 WebMvcConfigurer.addCorsMappings 方法或者注入 CorsFilter ，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[而是注入一个 CorsConfigurationSource ，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[同样能与上面的代码配合，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[正确的配置 CORS：](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

```
@Beanfun corsConfigurationSource(): CorsConfigurationSource {    val configuration = CorsConfiguration()    configuration.allowedOrigins = listOf("http://localhost:8080")    val source = UrlBasedCorsConfigurationSource()    source.registerCorsConfiguration("/hello", configuration)    return source}
```

[到此，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们已经看过了几种典型的例子了，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们接下来看看 Spring 到底是如何实现 CORS 验证的。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

## 5. 这些配置有什么区别

[我们会主要分析实现 WebMvcConfigurer.addCorsMappings 方法和调用 HttpSecurity.cors 方法这两种方式是如何实现 CORS 的，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[但在进行之前，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们要先复习一下 Filter 与 Interceptor 的概念。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

### 5.1 Filter 与 Interceptor

![](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYmYLtVQiatSUtz0EscuvicdGoUsSGsZzmzc5Z15682LkzU16biciaRYldBOltE3GZtWUrCh4SOGn8dgicA/640?wx_fmt=png)

[上图很形象的说明了 Filter 与 Interceptor 的区别，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[一个作用在 DispatcherServlet 调用前，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[一个作用在调用后。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

[但实际上，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[它们本身并没有任何关系，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[是完全独立的概念。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

[Filter 由 Servlet 标准定义，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[要求 Filter 需要在 Servlet 被调用之前调用，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[作用顾名思义，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[就是用来过滤请求。在 Spring Web 应用中，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[DispatcherServlet 就是唯一的 Servlet 实现。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

[Interceptor 由 Spring 自己定义，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[由 DispatcherServlet 调用，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[可以定义在 Handler 调用前后的行为。这里的 Handler ，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[在多数情况下，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[就是我们的 Controller 中对应的方法。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

[对于 Filter 和 Interceptor 的复习就到这里，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们只需要知道它们会在什么时候被调用到，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[就能理解后面的内容了。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

### 5.2 WebMvcConfigurer.addCorsMappings 方法做了什么

[我们从 WebMvcConfigurer.addCorsMappings 方法的参数开始，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[先看看 CORS 配置是如何保存到 Spring 上下文中的，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[然后在了解一下 Spring 是如何使用的它们。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

#### 5.2.1 注入 CORS 配置

##### 5.2.1.1 CorsRegistry 和 CorsRegistration

[WebMvcConfigurer.addCorsMappings 方法的参数 CorsRegistry 用于注册 CORS 配置，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[它的源码如下：](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

```
public class CorsRegistry {    private final List<CorsRegistration> registrations = new ArrayList<>();    public CorsRegistration addMapping(String pathPattern) {        CorsRegistration registration = new CorsRegistration(pathPattern);        this.registrations.add(registration);        return registration;    }    protected Map<String, CorsConfiguration> getCorsConfigurations() {        Map<String, CorsConfiguration> configs = new LinkedHashMap<>(this.registrations.size());        for (CorsRegistration registration : this.registrations) {            configs.put(registration.getPathPattern(), registration.getCorsConfiguration());        }        return configs;    }}
```

[我们发现这个类仅仅有两个方法：](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

*   [addMapping 接收一个 pathPattern，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[创建一个 CorsRegistration 实例，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[保存到列表后将其返回。在我们的代码中，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[这里的 pathPattern 就是 /hello。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
*   [getCorsConfigurations 方法将保存的 CORS 规则转换成 Map 后返回。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    

[CorsRegistration 这个类，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[同样很简单，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们看看它的部分源码：](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

```
public class CorsRegistration {    private final String pathPattern;    private final CorsConfiguration config;    public CorsRegistration(String pathPattern) {        this.pathPattern = pathPattern;        this.config = new CorsConfiguration().applyPermitDefaultValues();    }    public CorsRegistration allowedOrigins(String... origins) {        this.config.setAllowedOrigins(Arrays.asList(origins));        return this;    }}
```

[不难发现，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[这个类仅仅保存了一个 pathPattern 字符串和 CorsConfiguration，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[很好理解，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[它保存的是一个 pathPattern 对应的 CORS 规则。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

[在它的构造函数中，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[调用的 CorsConfiguration.applyPermitDefaultValues 方法则用于配置默认的 CORS 规则：](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

*   [allowedOrigins 默认为所有域。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
*   [allowedMethods 默认为 GET 、HEAD 和 POST。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
*   [allowedHeaders 默认为所有。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
*   [maxAge 默认为 30 分钟。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
*   [exposedHeaders 默认为 null，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[也就是不暴露任何 header。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
*   [credentials 默认为 null。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    

[创建 CorsRegistration 后，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们可以通过它的 allowedOrigins、allowedMethods 等方法修改它的 CorsConfiguration，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[覆盖掉上面的默认值。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

[现在，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们已经通过 WebMvcConfigurer.addCorsMappings 方法配置好 CorsRegistry 了，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[接下来看看这些配置会在什么地方被注入到 Spring 上下文中。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

##### 5.2.1.2 WebMvcConfigurationSupport

[CorsRegistry.getCorsConfigurations 方法，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[会被 WebMvcConfigurationSupport.getConfigurations 方法调用，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)这个方法如下：

```
protected final Map<String, CorsConfiguration> getCorsConfigurations() {    if (this.corsConfigurations == null) {        CorsRegistry registry = new CorsRegistry();        addCorsMappings(registry);        this.corsConfigurations = registry.getCorsConfigurations();    }    return this.corsConfigurations;}
```

[addCorsMappings(registry) 调用的是自己的方法，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[由子类 DelegatingWebMvcConfiguration 通过委托的方式调用到 WebMvcConfigurer.addCorsMappings 方法，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们的配置也由此被读取到。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

[getCorsConfigurations 是一个 protected 方法，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[是为了在扩展该类时，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[仍然能够直接获取到 CORS 配置。而这个方法在这个类里被四个地方调用到，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[这四个调用的地方，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[都是为了注册一个 HandlerMapping 到 Spring 容器中。每一个地方都会调用 mapping.setCorsConfigurations 方法来接收 CORS 配置，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[而这个 setCorsConfigurations 方法，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[则由 AbstractHandlerMapping 提供，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[CorsConfigurations 也被保存在这个抽象类中。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

[到此，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们的 CORS 配置借由 AbstractHandlerMapping 被注入到了多个 HandlerMapping 中，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[而这些 HandlerMapping 以 Spring 组件的形式被注册到了 Spring 容器中，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[当请求来临时，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[将会被调用。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

#### 5.2.2 获取 CORS 配置

[还记得前面关于 Filter 和 Interceptor 那张图吗？当请求来到 Spring Web 时，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[一定会到达 DispatcherServlet 这个唯一的 Servlet。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

[在 DispatcherServlet.doDispatch 方法中，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[会调用所有 HandlerMapping.getHandler 方法。好巧不巧，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[这个方法又是由 AbstractHandlerMapping 实现的：](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

```
@Override@Nullablepublic final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {    // 省略代码    if (CorsUtils.isCorsRequest(request)) {        CorsConfiguration globalConfig = this.corsConfigurationSource.getCorsConfiguration(request);        CorsConfiguration handlerConfig = getCorsConfiguration(handler, request);        CorsConfiguration config = (globalConfig != null ? globalConfig.combine(handlerConfig) : handlerConfig);        executionChain = getCorsHandlerExecutionChain(request, executionChain, config);    }    return executionChain;}
```

[在这个方法中，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[关于 CORS 的部分都在这个 if 中。我们来看看最后这个 getCorsHandlerExecutionChain 做了什么：](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

```
protected HandlerExecutionChain getCorsHandlerExecutionChain(HttpServletRequest request,        HandlerExecutionChain chain, @Nullable CorsConfiguration config) {    if (CorsUtils.isPreFlightRequest(request)) {        HandlerInterceptor[] interceptors = chain.getInterceptors();        chain = new HandlerExecutionChain(new PreFlightHandler(config), interceptors);    }    else {        chain.addInterceptor(new CorsInterceptor(config));    }    return chain;}
```

[可以看到：](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

*   [针对 preflight request，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[由于不会有对应的 Handler 来处理，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[所以这里就创建了一个 PreFlightHandler 来作为这次请求的 handler。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
*   [对于其他的跨域请求，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[因为会有对应的 handler，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[所以就在 handlerExecutionChain 中加入一个 CorsInterceptor 来进行 CORS 验证](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    

[这里的 PreFlightHandler 和 CorsInterceptor 都是 AbstractHandlerMapping 的内部类，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[实现几乎一致，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[区别仅仅在于一个是 HttpRequestHandler，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[一个是 HandlerInterceptor；它们对 CORS 规则的验证都交由 CorsProcessor 接口完成，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[这里采用了默认实现 DefaultCorsProcessor。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

[DefaultCorsProcessor 则是依照 CORS 标准来实现，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[并在验证失败的时候打印 debug 日志并拒绝请求。我们只需要关注一下标准中没有定义的验证失败时的状态码：](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

```
protected void rejectRequest(ServerHttpResponse response) throws IOException {    response.setStatusCode(HttpStatus.FORBIDDEN);    response.getBody().write("Invalid CORS request".getBytes(StandardCharsets.UTF_8));}
```

[CORS 验证失败时调用这个方法，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[并设置状态码为 403。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

#### 5.2.3 小结

[通过对源码的研究，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们发现实现 WebMvcConfigurer.addCorsMappings 方法的方式配置 CORS，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[会在 Interceptor 或者 Handler 层进行 CORS 验证。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

### 5.3 HtttpSecurity.cors 方法做了什么

[在研究这个方法的行为之前，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们先来回想一下，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们调用这个方法解决的是什么问题。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

[前面我们通过某种方式配置好 CORS 后，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[引入 Spring Security，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[CORS 就失效了，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[直到调用这个方法后，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[CORS 规则才重新生效。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

[下面这些原因，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[导致了 preflight request 无法通过身份验证，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[从而导致 CORS 失效：](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

1.  [preflight request 不会携带认证信息。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
2.  [Spring Security 通过 Filter 来进行身份验证。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
3.  [Interceptor 和 HttpRequestHanlder 在 DispatcherServlet 之后被调用。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
4.  [Spring Security 中的 Filter 优先级比我们注入的 CorsFilter 优先级高。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    

[接下来我们就来看看 HttpSecurity.cors 方法是如何解决这个问题的。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

#### 5.3.1 CorsConfigurer 如何配置 CORS 规则

[HttpSecurity.cors 方法中其实只有一行代码：](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

```
public CorsConfigurer<HttpSecurity> cors() throws Exception {    return getOrApply(new CorsConfigurer<>());}
```

[这里调用的 getOrApply 方法会将 SecurityConfigurerAdapter 的子类实例加入到它的父类 AbstractConfiguredSecurityBuilder 维护的一个 Map 中，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[然后一个个的调用 configure 方法。所以，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们来关注一下 CorsConfigurer.configure 方法就好了。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

```
@Overridepublic void configure(H http) throws Exception {    ApplicationContext context = http.getSharedObject(ApplicationContext.class);    CorsFilter corsFilter = getCorsFilter(context);    if (corsFilter == null) {        throw new IllegalStateException(                "Please configure either a " + CORS_FILTER_BEAN_NAME + " bean or a "                        + CORS_CONFIGURATION_SOURCE_BEAN_NAME + "bean.");    }    http.addFilter(corsFilter);}
```

[这段代码很好理解，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[就是在当前的 Spring Context 中找到一个 CorsFilter，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[然后将它加入到 http 对象的 filters 中。由上面的 HttpSecurity.cors 方法可知，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[这里的 http 对象实际类型就是 HttpSecurity。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

#### 5.3.2 getCorsFilter 方法做了什么

[也许你会好奇，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[HttpSecurity 要如何保证 CorsFilter 一定在 Spring Security 的 Filters 之前调用。但是在研究这个之前，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们先来看看同样重要的 getCorsFilter 方法，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[这里可以解答我们前面的一些疑问。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

```
private CorsFilter getCorsFilter(ApplicationContext context) {    if (this.configurationSource != null) {        return new CorsFilter(this.configurationSource);    }    boolean containsCorsFilter = context            .containsBeanDefinition(CORS_FILTER_BEAN_NAME);    if (containsCorsFilter) {        return context.getBean(CORS_FILTER_BEAN_NAME, CorsFilter.class);    }    boolean containsCorsSource = context            .containsBean(CORS_CONFIGURATION_SOURCE_BEAN_NAME);    if (containsCorsSource) {        CorsConfigurationSource configurationSource = context.getBean(                CORS_CONFIGURATION_SOURCE_BEAN_NAME, CorsConfigurationSource.class);        return new CorsFilter(configurationSource);    }    boolean mvcPresent = ClassUtils.isPresent(HANDLER_MAPPING_INTROSPECTOR,            context.getClassLoader());    if (mvcPresent) {        return MvcCorsFilter.getMvcCorsFilter(context);    }    return null;}
```

[这是 CorsConfigurer 寻找 CorsFilter 的全部逻辑，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们用人话来说就是：](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

1.  [CorsConfigurer 自己是否有配置 CorsConfigurationSource，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[如果有的话，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[就用它创建一个 CorsFilter。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
2.  [在当前的上下文中，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[是否存在一个名为 corsFilter 的实例，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[如果有的话，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[就把他当作一个 CorsFilter 来用。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
3.  [在当前的上下文中，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[是否存在一个名为 corsConfigurationSource 的 CorsConfigurationSource 实例，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[如果有的话，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[就用它创建一个 CorsFilter。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
4.  [在当前上下文的类加载器中，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[是否存在类 HandlerMappingIntrospector，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[如果有的话，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[则通过 MvcCorsFilter 这个内部类创建一个 CorsFilter。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
5.  [如果没有找到，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[那就返回一个 null，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[调用的地方最后会抛出异常，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[阻止 Spring 初始化。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    

[上面的第 2、3、4 步能解答我们前面的配置为什么生效，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[以及它们的区别。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

[注册 CorsFilter 的方式，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[这个 Filter 最终会被直接注册到 Servlet container 中被使用到。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

[注册 CorsConfigurationSource 的方式，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[会用这个 source 创建一个 CorsFiltet 然后注册到 Servlet container 中被使用到。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

[而第四步的情况比较复杂。HandlerMappingIntrospector 是 Spring Web 提供的一个类，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[实现了 CorsConfigurationSource 接口，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[所以在 MvcCorsFilter 中，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[它被直接用于创建 CorsFilter。它实现的 getCorsConfiguration 方法，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[会经历：](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

1.  [遍历 HandlerMapping。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
2.  [调用 getHandler 方法得到 HandlerExecutionChain。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
3.  [从中找到 CorsConfigurationSource 的实例。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
4.  [调用这个实例的 getCorsConfiguration 方法，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[返回得到的 CorsConfiguration。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    

[所以得到的 CorsConfigurationSource 实例，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[实际上就是前面讲到的 CorsInterceptor 或者 PreFlightHandler。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

[所以第四步实际上匹配的是实现 WebMvcConfigurer.addCorsMappings 方法的方式。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

[由于在 CorsFilter 中每次处理请求时都会调用 CorsConfigurationSource.getCorsConfiguration 方法，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[而 DispatcherServlet 中也会每次调用 HandlerMapping.getHandler 方法，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[再加上这时的 HandlerExecutionChain 中还有 CorsInterceptor，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[所以使用这个方式相对于其他方式，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[做了很多重复的工作。所以 WebMvcConfigurer.addCorsMappings + HttpSecurity.cors 的方式降低了我们代码的效率，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[也许微乎其微，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[但能避免的情况下，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[还是不要使用。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

#### 5.3.3 HttpSecurity 中的 filters 属性

[在 CorsConfigurer.configure 方法中调用的 HttpSecurity.addFilter 方法，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[由它的父类 HttpSecurityBuilder 声明，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[并约定了很多 Filter 的顺序。然而 CorsFilter 并不在其中。不过在 Spring Security 中，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[目前还只有 HttpSecurity 这一个实现，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[所以我们来看看这里的代码实现就知道 CorsFilter 会排在什么地方了。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

```
public HttpSecurity addFilter(Filter filter) {    Class<? extends Filter> filterClass = filter.getClass();    if (!comparator.isRegistered(filterClass)) {        throw new IllegalArgumentException("...");    }    this.filters.add(filter);    return this;}
```

[我们可以看到，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[Filter 会被直接加到 List 中，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[而不是按照一定的顺序来加入的。但同时，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们也发现了一个 comparator 对象，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[并且只有被注册到了该类的 Filter 才能被加入到 filters 属性中。这个 comparator 又是用来做什么的呢？](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

[在 Spring Security 创建过程中，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[会调用到 HttpSeciryt.performBuild 方法，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[在这里我们可以看到 filters 和 comparator 是如何被使用到的。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

```
protected DefaultSecurityFilterChain performBuild() throws Exception {    Collections.sort(filters, comparator);    return new DefaultSecurityFilterChain(requestMatcher, filters);}
```

[可以看到，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[Spring Security 使用了这个 comparator 在获取 SecurityFilterChain 的时候来保证 filters 的顺序，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[所以，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[研究这个 comparator 就能知道在 SecurityFilterChain 中的那些 Filter 的顺序是如何的了。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

[这个 comparator 的类型是 FilterComparator ，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[从名字就能看出来是专用于 Filter 比较的类，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[它的实现也并不神秘，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[从构造函数就能猜到是如何实现的：](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

```
FilterComparator() {    Step order = new Step(INITIAL_ORDER, ORDER_STEP);    put(ChannelProcessingFilter.class, order.next());    put(ConcurrentSessionFilter.class, order.next());    put(WebAsyncManagerIntegrationFilter.class, order.next());    put(SecurityContextPersistenceFilter.class, order.next());    put(HeaderWriterFilter.class, order.next());    put(CorsFilter.class, order.next());  // 省略代码}
```

[可以看到 CorsFilter 排在了第六位，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[在所有的 Security Filter 之前，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[由此便解决了 preflight request 没有携带认证信息的问题。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

#### 5.3.4 小结

[引入 Spring Security 之后，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们的 CORS 验证实际上是依然运行着的，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[只是因为 preflight request 不会携带认证信息，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[所以无法通过身份验证。使用 HttpSecurity.cors 方法会帮助我们在当前的 Spring Context 中找到或创建一个 CorsFilter 并安排在身份验证的 Filter 之前，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[以保证能对 preflight request 正确处理。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

## 6. 总结

[研究了 Spring 中 CORS 的代码，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[我们了解到了这样一些知识：](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)

*   [实现 WebMvcConfigurer.addCorsMappings 方法来进行的 CORS 配置，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[最后会在 Spring 的 Interceptor 或 Handler 中生效。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
*   [注入 CorsFilter 的方式会让 CORS 验证在 Filter 中生效。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
*   [引入 Spring Security 后，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[需要调用 HttpSecurity.cors 方法以保证 CorsFilter 会在身份验证相关的 Filter 之前执行。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
*   [HttpSecurity.cors + WebMvcConfigurer.addCorsMappings 是一种相对低效的方式，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[会导致跨域请求分别在 Filter 和 Interceptor 层各经历一次 CORS 验证。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
*   [HttpSecurity.cors + 注册 CorsFilter 与 HttpSecurity.cors + 注册 CorsConfigurationSource 在运行的时候是等效的。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    
*   [在 Spring 中，](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)[没有通过 CORS 验证的请求会得到状态码为 403 的响应。](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247489641&idx=1&sn=4cee9122a2fa2677bdc71abf5c7e8c00&scene=21#wechat_redirect)
    

今日干货

![](https://mmbiz.qpic.cn/mmbiz_jpg/GvtDGKK4uYlgtbREKPZQnH2H9BeAv8OrkqXU7R13XZGIc7TuOI3j2P5cU2Kgs7YmdAoK5Ve8kfjCd5r6B7WF4g/640?wx_fmt=jpeg)

刚刚发表

查看:66666 回复: 666

公众号后台回复 ssm，免费获取松哥纯手敲的 SSM 框架学习干货。