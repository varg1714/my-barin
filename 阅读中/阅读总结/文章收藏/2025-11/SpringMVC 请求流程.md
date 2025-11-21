---
source:
  - "[[阅读中/文章列表/文章收藏/2025-11/Spring基础 - SpringMVC请求流程和案例]]"
create: 2025-11-12
---
## 1. 核心概念：什么是 MVC 与 Spring MVC？

1. **MVC 模式**
    MVC 是一种软件设计规范，其核心思想是**解耦**，将应用程序分为三个核心部分：
    * **Model (模型)**: 负责处理应用程序的数据和业务逻辑。
    * **View (视图)**: 负责数据显示，将模型数据呈现给用户。
    * **Controller (控制器)**: 负责处理用户交互，接收用户请求，调用模型进行处理，并指定要呈现的视图。

    ![MVC 模式图](https://pdai.tech/images/spring/springframework/spring-springframework-mvc-4.png)

2. **Spring MVC**
    Spring MVC 是 Spring 框架在原生技术（如 IOC、AOP）基础上，遵循 MVC 规范推出的一个**请求驱动类型**的轻量级 Web 框架。它的目标是简化 Java Web 开发，通过将 Web 层进行职责解耦，实现清晰、灵活、易于测试的 Web 应用程序。

## 2. Spring MVC 请求处理全流程

Spring MVC 的核心是一个前端控制器 `DispatcherServlet`，它负责接收所有请求，并将其分发给不同的组件进行处理。

![Spring MVC 请求流程图](https://pdai.tech/images/spring/springframework/spring-springframework-mvc-5.png)

**核心流程步骤详解：**

1. **用户请求 -> `DispatcherServlet`**
    所有请求首先到达前端控制器 `DispatcherServlet`。它不直接处理请求，而是作为中央枢纽，负责协调和委派。

2. **`DispatcherServlet` -> `HandlerMapping` (处理器映射器)**
    `DispatcherServlet` 查询一个或多个 `HandlerMapping`，找到与当前请求（URL、HTTP 方法等）匹配的处理器（`Handler`）。这个 `Handler` 通常是一个 Controller 方法。`HandlerMapping` 会将找到的 `Handler` 连同相关的拦截器（`HandlerInterceptor`）一起封装成一个 `HandlerExecutionChain` 对象返回。

3. **`DispatcherServlet` -> `HandlerAdapter` (处理器适配器)**
    `DispatcherServlet` 拿到 `Handler` 后，并不会直接执行它。而是选择一个能够处理该类型 `Handler` 的 `HandlerAdapter`。这是实现框架灵活性的关键，我们将在下一节深入探讨。

4. **`HandlerAdapter` -> 调用 `Handler`**
    `HandlerAdapter` 会以一种统一的方式去**执行**真正的 `Handler`（Controller 方法）。它负责处理方法参数的解析、数据绑定、方法调用等复杂工作，然后将 `Handler` 的返回值封装成一个 `ModelAndView` 对象。

5. **`Handler` -> `ModelAndView` -> `DispatcherServlet`**
    `HandlerAdapter` 将包含模型数据（Model）和逻辑视图名（View Name）的 `ModelAndView` 对象返回给 `DispatcherServlet`。

6. **`DispatcherServlet` -> `ViewResolver` (视图解析器)**
    `DispatcherServlet` 将 `ModelAndView` 中的逻辑视图名交给 `ViewResolver`。`ViewResolver` 负责将逻辑视图名解析成一个具体的 `View` 对象（例如一个 `JstlView` 对象，对应一个 JSP 文件）。

7. **`View` 渲染 -> 响应用户**
    `View` 对象使用 `ModelAndView` 中的模型数据进行渲染（例如，将数据显示在 JSP 页面上），最终生成 HTML 响应，由 `DispatcherServlet` 返回给客户端。

## 3. 核心组件深度解析：处理器适配器 (HandlerAdapter)

这是理解 Spring MVC 灵活性的关键。

**1. 为什么需要 `HandlerAdapter`？**

`DispatcherServlet`（项目经理）需要一种**统一的方式**来调用各种不同类型的 `Handler`（专家）。但 `Handler` 的写法多种多样：

* 有实现特定接口的（如 `Controller`）。
* 有使用注解的普通方法（如 `@RequestMapping`）。
* 有函数式编程的 `HandlerFunction`。

`HandlerAdapter`（专家的私人助理）的作用就是**适配这些不同类型的 `Handler`**，让 `DispatcherServlet` 可以用同一种方式（调用 `adapter.handle(...)`）来执行它们，而无需关心其内部实现细节。

**2. `HandlerAdapter` 的具体实现与适配目标**

下表总结了 Spring MVC 中主要的 `HandlerAdapter` 及其功能：

| HandlerAdapter | | 适配的 Handler 类型 | | 核心适配工作与场景 |
| :--- | :--- | :--- | :--- | :--- |
| **`RequestMappingHandlerAdapter`** | | 被 `@RequestMapping` 注解的方法 (`HandlerMethod`) | | **现代主流 (注解驱动)**。它完成两项关键适配：<br>1. **参数适配**：将 HTTP 请求内容适配成方法需要的各种 Java 对象（`@RequestParam`, `@RequestBody` 等）。<br>2. **返回值适配**：将方法的各种返回类型（`String`, `ResponseEntity`, POJO 等）适配成统一的 `ModelAndView`。 |
| **`HandlerFunctionAdapter`** | | `HandlerFunction` | | **现代主流 (函数式编程)**。用于支持 Spring 5 引入的函数式端点，将 `ServerRequest` 传递给处理器，并处理返回的 `ServerResponse`。 |
| **`SimpleControllerHandlerAdapter`** | | 实现 `Controller` 接口的类 | | **早期 Spring MVC**。适配实现了 `Controller` 接口、拥有固定 `handleRequest` 方法的处理器。 |
| **`HttpRequestHandlerAdapter`** | | 实现 `HttpRequestHandler` 接口的类 | | **特定场景**。适配那些不需要视图解析、直接操作 `HttpServletResponse` 的处理器（如文件下载）。 |
| **`SimpleServletHandlerAdapter`** | | 实现 `javax.servlet.Servlet` 接口的类 | | **遗留系统集成**。允许将一个标准 Servlet 作为处理器集成到 Spring MVC 流程中，非常罕见。 |

## 4. 其他重要组件

除了上述核心流程中的组件，Spring MVC 还依赖其他组件来完成更丰富的功能：

* **`MultipartResolver`**: 用于处理文件上传请求（`multipart/form-data`）。它会将请求包装成 `MultipartHttpServletRequest`，方便后续获取文件。
    ![MultipartResolver](https://pdai.tech/images/spring/springframework/spring-springframework-mvc-7.png)
* **`LocaleResolver`**: 用于解析客户端的区域信息，以支持国际化（i18n）。
    ![LocaleResolver](https://pdai.tech/images/spring/springframework/spring-springframework-mvc-6.png)
* **`ThemeResolver`**: 用于解析应用的主题（如 CSS 和图片），以支持视觉风格的切换。
    ![ThemeResolver](https://pdai.tech/images/spring/springframework/spring-springframework-mvc-9.png)

## 5. Spring MVC 案例实践总结

原始笔记中的案例完美地展示了上述理论的实际应用。

1. **项目配置 (`pom.xml`)**:
    * 核心依赖是 `spring-webmvc`，它会自动引入 Spring Core 等相关包。

2. **Web 入口配置 (`web.xml`)**:
    * 配置 `DispatcherServlet` 作为所有请求的入口 (`<url-pattern>/</url-pattern>`)。
    * 通过 `<init-param>` 的 `contextConfigLocation` 指定 Spring MVC 配置文件的位置（`classpath:springmvc.xml`）。

3. **Spring MVC 配置 (`springmvc.xml`)**:
    * `<context:component-scan>`: 扫描指定的包（如 `tech.pdai.springframework.springmvc`），自动注册带有 `@Controller`, `@Service` 等注解的 Bean。
    * `<mvc:annotation-driven/>`: **非常关键！** 它会隐式地注册 `RequestMappingHandlerAdapter`、`HandlerMapping` 以及一系列支持注解功能（如 JSON 转换、数据校验）的组件。
    * **`InternalResourceViewResolver`**: 配置视图解析器。示例中配置了前缀 `/WEB-INF/views/` 和后缀 `.jsp`，这意味着当 Controller 返回逻辑视图名 "userList" 时，它会被解析为物理路径 `/WEB-INF/views/userList.jsp`。

4. **代码实现 (Controller, Service, DAO)**:
    * **`UserController`**: 使用 `@Controller` 声明，并用 `@RequestMapping("/user")` 将 `list` 方法映射到 `/user` 请求。方法返回一个 `ModelAndView` 对象，其中包含了要传递给视图的数据 (`userList`) 和逻辑视图名 (`userList`)。
    * **`UserServiceImpl` / `UserDaoImpl`**: 展示了分层结构，Controller 依赖 Service，Service 依赖 DAO。

5. **视图 (`userList.jsp`)**:
    * 使用 JSTL 的 `<c:forEach>` 标签遍历从 Controller 传递过来的 `userList` 模型数据，并将其渲染成一个 HTML 表格。

通过这个案例，您可以清晰地看到一个请求是如何从 `web.xml` 进入 `DispatcherServlet`，再由 `springmvc.xml` 的配置驱动，最终由 Controller 处理并渲染到 JSP 页面的完整过程。