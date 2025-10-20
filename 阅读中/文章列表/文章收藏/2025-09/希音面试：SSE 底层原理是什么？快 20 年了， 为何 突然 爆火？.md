---
source: https://mp.weixin.qq.com/s/vct-YamYF2XFi4VatVRWMw
create: 2025-09-11 21:13
read: true
knowledge: true
knowledge-date: 2025-10-09
tags:
  - 计算机原理
  - 网络
summary: "[[SSE 原理]]"
---
FSAC 未来超级架构师

架构师总动员  
实现架构转型，再无中年危机

## 尼恩说在前面

在 45 岁老架构师 尼恩的**读者交流群** (50+) 中，通过 **Java+AI 双驱架构** 帮助很多小伙伴拿到了一线企业如 字节、得物、阿里、滴滴、极兔、有赞、希音、百度、网易、美团、蚂蚁、得物的面试资格，遇到很多很重要的面试题：

什么是 SSE？SSE 为何突然爆火？

SSE 与 WEBSocket 如何选型？

最近有小伙伴在**面试希音、滴滴、阿里等**，都到了这个的面试题。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WM9jp4Ufo3GuDFRCziaY8F7T8Cnk0Ds6BaJuss4Rdt7E8uBm9X9TCicEtdgRygaGoLOyPU83PRwDl0Q/640?from=appmsg&watermark=1#imgIndex=0)

小伙伴是按照尼恩的套路是作答的，拿到了  阿里、希音 offer 。

小伙伴是二本，而且空挡一年了， 能拿到  阿里、希音 offer  ，他也觉得尼恩的回答套路 太牛逼了。

这里尼恩把这个 sse 的介绍体系， 展示给大家，帮助大家进大厂，拿高薪。

sse 现在很火，建议大家收藏起来，多看几遍

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/xlgvgPaib7WM9jp4Ufo3GuDFRCziaY8F7TgLuh46C9cic9lHANQ9wvDgtqhFJwDicpNjIzibpibDS1GBvIib0Zv2UgsTQ/640?from=appmsg&watermark=1#imgIndex=1)

 一、什么是 SSE？SSE 为何突然爆火？

 - 1.1、什么是 SSE？

 - 1.2  SSE (Server-Sent Events) 诞生背景

 - 1.3 SSE   发展历程

 - 1.4、SSE 的主要特点

- 二：SSE 出来 20 年才一夜 爆火 ，为什么？

 - 2.1 什么是 “打字机” 式逐 token 输出？

 - 2.2 为什么 SSE 是这种模式的 “天作之合”？

- 三、SSE 的工作原理

 - 3.1 工作机制的流程图

 - 3.2、SSE 与其他通信方式对比

 - 3.3 SSE 的适用场景

- 四、sse 客户端 API 详解

 - 4.1、认识 浏览器  EventSource 对象

 - 4.2、基本使用方法

 - 4.3、自定义事件

- 五、SSE 服务器实现：数据格式与规则

 - 5.1 HTTP 头信息要求

 - 5.2 数据传输格式

 - 5.3 核心字段说明

 - 5.4 服务器发送流程

- 六、SSE 实战案例：用 Spring Boot 搭建实时通信系统

 - 6.1、案例整体架构

 - 6.2、服务端实现

 - 6.3、客户端实现（HTML 页面）

 - 6.4、运行与测试

 - 6.5、 服务端 关键技术点

- 七：SSE 与 WEBSocket 如何选型？

 - 7.1 sse 与 WEBSocket 全面对比

 - 7.2  如何选择？决策指南

 - 7.3  WebSocket+SSE 混合架构

 - 7.4  chat2ai 是选择 sse 协议还是 WEBSocket

# 一、什么是 SSE？SSE 为何突然爆火？

你有没有想过，为什么 ChatGPT 的回答能逐字逐句地 “流” 出来？

这一切的背后，都离不开一项关键技术——**SSE（Server-Sent Events）**！

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WM9jp4Ufo3GuDFRCziaY8F7TfYSpbvGfzVW8anMgDWqDgw45wvVbWROOKSfkeQ1QsXpN9VlFGrg0KQ/640?from=appmsg&watermark=1#imgIndex=2)

## 1.1、什么是 SSE？

SSE（Server-Sent Events）是一种基于 HTTP 协议的**服务器推送技术**，允许服务端主动向客户端发送数据流。

**SSE  可以被理解为 HTTP 的一个扩展或一种特定用法。它不是一个全新的、独立的协议，而是构建在标准 HTTP/1.1 协议之上的技术。**

SSE 就像是服务器打开了一个 “单向数据管道”，服务器通过 HTTP 扩展 可以持续不断地流向浏览器，无需客户端反复发起请求。

其实很简单的：  SSE = HTTP 扩展字段 + Keepalive 长连接。

SSE 提供了一种简单、可靠的方式来实现服务器向客户端的实时数据推送。它非常适合通知、实时数据更新、日志流和类似 ChatGPT 的逐字输出场景。如果你只需要单向通信，SSE 往往是比 WebSocket 更简单、更轻量的选择。

SSE 适用于服务器主动向客户端推送数据的场景，如实时通知、动态更新等。

所以，目前 几乎所有主流浏览器都原生支持 SSE。

## 1.2  SSE (Server-Sent Events) 诞生背景

### 短轮询、长轮询、Flash 、 WebSocket

在 SSE 技术出现之前，Web 应用要实现服务器向客户端的实时数据推送，主要依赖以下几种技术，但它们都存在明显的缺陷：

1、 **短轮询 (Polling)**:

*   **原理**：用短连接请求数据。客户端以固定的时间间隔（例如每秒一次）频繁地向服务器发送请求，询问是否有新数据。
    
*   **缺点**：大量请求可能是无效的（无新数据），浪费服务器和带宽资源，实时性差。
    

**短轮询 的流程图**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WM9jp4Ufo3GuDFRCziaY8F7ThzN55xOeqfWdAgaZsAsBQbZvBtvjOM1XjuSiblGjicXma1OHCQXTHCOg/640?from=appmsg&watermark=1#imgIndex=3)

2、 **长轮询 (Long Polling)**:

*   **原理**：使用长连接请求数据。 客户端发送一个请求，服务器会保持这个连接打开（长连接），直到有新数据可用或超时。一旦客户端收到响应，会立即发起下一个请求。
    
*   **缺点**：虽然减少了无效请求，但每个连接仍然需要客户端发起，服务器需要维护大量挂起的连接，实现复杂。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WM9jp4Ufo3GuDFRCziaY8F7TtKHXK3LCcriaqpqjwjapN42dFVqPHhyico2DrY1IwmDLHFuHJcmKwFBQ/640?from=appmsg&watermark=1#imgIndex=4)

长轮询 (Long Polling) 突破：减少无效请求，但服务器需维护挂起连接

3、 **基于 Flash 的解决方案**:

*   **原理**：利用 Adobe Flash 插件提供的 Socket 功能实现全双工通信。
    
*   **缺点**：依赖浏览器插件，在移动端（如 iPhone）不受支持，且随着技术的发展（Flash 被淘汰）已走向消亡。基于 Flash 方法都非原生支持，效率低下或依赖外部插件。
    

4、 **基于 WebSocket 的解决方案**:

*   **原理**：在客户端与服务器之间建立一条全双工的 TCP 长连接，双方可随时互相推送数据。
    
*   **缺点**： 需要一次额外的协议升级握手（`Upgrade: websocket`），对 CDN、防火墙、代理服务器的兼容性不如普通 HTTP； 双向通信能力在 “服务器→客户端单向推送” 场景下显得过度设计，增加心跳、重连、帧解析等复杂度；早期浏览器支持不一（IE ≤ 9 无原生实现），需要 Polyfill 或 Flash 降级方案。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WM9jp4Ufo3GuDFRCziaY8F7TQlTZqaRIJGzYv54ornKU0ianCg4LMSel7RzoZw65Q9LF3zjjNM7MoMQ/640?from=appmsg&watermark=1#imgIndex=5)

WebSocket：全双工通道 革命性：摆脱 HTTP 束缚，实现真 实时交互

WebSocket 并不是 Web 领域 的通讯协议，属于复杂度高  二进制通讯协议。

### SSE 诞生的核心背景

因此，Web 领域迫切需要一种**标准化的、高效的、由浏览器原生支持的**服务器到客户端的单向通信机制。这就是 SSE 诞生的核心背景。

**核心需求：**

*   **简单**：易于服务器和客户端实现。
    
*   **高效**：基于 HTTP/HTTPS，避免不必要的请求开销。
    
*   **标准**：成为 W3C 标准，得到浏览器原生支持。
    
*   **自动重连**：内置连接失败后自动重试的机制。
    

SSE：真正的服务器推送

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WM9jp4Ufo3GuDFRCziaY8F7T0F2QuiaIXtud352wjJ5lbNibGa4xSfucVSeIeOR5objG6jOAbFIYmPGQ/640?from=appmsg&watermark=1#imgIndex=6)

## 1.3 SSE   发展历程

SSE 的发展是 Web 标准化进程和实时通信需求共同推动的结果。

下图概述了其关键发展节点：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WM9jp4Ufo3GuDFRCziaY8F7TBfVHAUcuthXUlibkDdkxFEVd9ypiaeOa8KWmk3vHpib3r6icxF2WUFiczBA/640?from=appmsg&watermark=1#imgIndex=7)

让我们对图中的关键阶段进行详细解读：

**1、 诞生背景（2006 年以前）**

*   Web 早期只有 “请求 - 响应” 范式，实时需求（股票、IM、行情）只能靠轮询或长轮询，延迟高、浪费资源。
    
*   Comet（长连接 iframe、jsonp、xhr-streaming 等 Hack 方案）出现，但实现复杂、浏览器兼容性差、占用连接数高。
    
*   业界急需一种 “浏览器原生、基于 HTTP、单向服务器推送” 的轻量机制。
    

**2、 概念提出与标准化 (约 2006-2009 年)**

*   SSE 的概念最初作为 **HTML5** 标准的一部分被提出，由 **WHATWG** (Web Hypertext Application Technology Working Group) 和 **W3C** (World Wide Web Consortium) 共同推动。
    
*   其设计思想是定义一个简单的、基于 HTTP 的协议，允许服务器通过一个长连接持续地向客户端发送文本流。
    
*   2006 年，Opera 9 在浏览器里率先实现名为 Server-Sent Events 的实验 API，用 DOM 事件把服务器推送的文本块喂给页面。
    
*   同期 WHATWG HTML5 草案开始收录相关章节，定义了 text/event-stream MIME 类型及 “event: / data:” 行协议。
    
*   后来，它从庞大的 HTML5 规范中分离出来，成为了一个独立的 W3C 标准文档。
    
*   2008 年：SSE 被正式写入 HTML5 草案，随后进入 W3C 标准流程。
    

**3、 浏览器支持与推广 (约 2010-2015 年)**

*   **2011 年左右**，主流浏览器（如 Firefox、Chrome、Safari、Opera）开始陆续支持 SSE API。 Firefox 6、Chrome 6、Safari 5、Opera 11.5 陆续完成原生实现；IE 系列缺席（直到 Edge 79 才补票）。
    
*   **关键的障碍**：**Internet Explorer (包括 IE 11)** 始终没有支持 SSE API。这在一定程度上限制了其早期的广泛应用，开发者通常需要为此准备降级方案（如回落到长轮询）。
    
*   随着 Chrome、Firefox 等现代浏览器的市场份额不断上升，以及移动端浏览器对 SSE 的良好支持，SSE 逐渐成为开发实时 Web 应用的可信选择。
    
*   2014 年 10 月：HTML5 成为 W3C Recommendation，SSE 作为官方子模块锁定最终语法，浏览器阵营格局定型。
    

**4. 正式推荐与成熟 (2015 年至 2022)**

*   2015-2020 年，WebSocket 与 WebRTC 占据实时通信话题中心，SSE 主要在企业内部仪表盘、日志 tail 等低频场景默默使用。
    
*   SSE 由于有 “单向文本流 + 自动重连 + 轻量”  特性，**所以没有被 WebSocket 与 WebRTC  踩死**， 使其在 IoT 设备、移动端 WebView 中仍保有一席之地。
    
*   **2015 年**，W3C 发布了 **Server-Sent Events** 的正式推荐标准，标志着该技术的成熟和稳定。
    
*   在此期间，前端生态框架（如 React、Vue.js）和后端语言（如 Node.js、Python、Java）都提供了对 SSE 的良好支持，出现了大量易用的库和示例。
    

**5、 大模型时代的爆发（2022 至今）**

*   虽然 **WebSocket** 提供了全双工通信能力，但 SSE 因其简单的 API、基于 HTTP 带来的良好兼容性（如无需担心代理或防火墙问题）、以及自动重连等特性，在**只需要服务器向客户端推送数据**的场景中（如新闻推送、实时行情、状态更新、AI 处理进度流式输出等）成为了更简单、更合适的选择。
    
*   ChatGPT、Claude 等生成式 AI 需要 “打字机” 式逐 token 输出，SSE 天然契合：
    
*   基于 HTTP/1.1 无需升级协议，CDN 缓存友好；
    
*   浏览器 EventSource API 一行代码即可接入；
    
*   文本流可直接承载 JSON Lines 或 markdown 片段。
    
*   2022 年底起，OpenAI、Anthropic、Google Bard 均把 text/event-stream 作为官方流式回答协议，社区库（FastAPI SSE-Star、Spring WebFlux、Node sse.js、Go gin-sse）迎来二次繁荣。
    

## 1.4、SSE 的主要特点

<table><thead><tr><td><span><strong><span leaf="">特性</span></strong></span></td><td><span><strong><span leaf="">说明</span></strong></span></td></tr></thead><tbody><tr><td><section><span><strong><span leaf="">单向通信</span></strong></span></section></td><td><section><span><span leaf="">仅支持服务器向客户端发送数据</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">基于 HTTP</span></strong></span></section></td><td><section><span><span leaf="">无需升级协议或使用额外端口，兼容现有网络设施</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">自动重连</span></strong></span></section></td><td><section><span><span leaf="">浏览器在连接断开后可自动重新建立连接</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">轻量易用</span></strong></span></section></td><td><section><span><span leaf="">浏览器原生支持，API 简洁易懂</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">文本流支持</span></strong></span></section></td><td><section><span><span leaf="">默认支持 UTF-8 文本，二进制数据需编码后传输</span></span></section></td></tr></tbody></table>

SSE 和 WebSocket 都能建立浏览器与服务器的长期通信，但区别很明显：

*   **SSE** 是单向推送  不是双向推送， 而且是 http 协议的一个扩展协议， 使用简单、自动重连，适合文本类实时推送。
    
*   **WebSocket** 是双向通信，不是 http 协议的一个扩展协议，WebSocket  更灵活，但实现相对复杂。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WM9jp4Ufo3GuDFRCziaY8F7TcODna3GmvKelFuEkIpENVI1GDH9BP2aZ2KnaOYIeBWTGYbVr3GJSYg/640?from=appmsg&watermark=1#imgIndex=8)

**流程解读：**

1、**连接初始化**：客户端使用特定的 `Content-Type: text/event-stream` 向服务器发起一个普通的 HTTP GET 请求。服务器确认并保持连接开放。

2、**数据推送**：服务器通过保持打开的连接，以纯文本格式（遵循 `data: ...`、`event: ...` 等规范）持续发送数据块。每个消息以两个换行符 `\n\n` 结束。

3、**连接容错**：如果连接因网络问题中断，SSE 客户端内置的机制会自动尝试重新建立连接，极大地提高了应用的鲁棒性。

4、**客户端处理**：浏览器端的 `EventSource` API 会解析收到的数据流，触发相应的事件（如 `onmessage` 或自定义事件），让开发者能够处理推送来的数据

SSE 的诞生是 Web 开发对**简单、高效、标准化**的服务器推送技术需求的直接结果。它有效地替代了笨拙的轮询技术，在与 WebSocket 的竞争中，找到了自身在**单向数据流**场景下的独特定位。

其发展历程经历了从概念提出、浏览器支持到成为正式标准的完整路径。尽管曾受限于 IE，但在现代浏览器中已成为一项稳定、可靠且被广泛采用的技术。如今，在实时通知、金融仪表盘、实时日志跟踪和大型语言模型（LLM）的流式响应输出等场景中，SSE 都是首选的解决方案。

# 二：SSE 出来 20 年才一夜 爆火 ，为什么？

**SSE 最近站到聚光灯下，几乎可以说最大的推手就是当前 AI 应用（尤其是 ChatGPT 等大型语言模型）的爆发式增长。**

SSE  之所以成为 AI 应用的 “标配”，是因为 SSE 与  AI 所需的 “打字机” 输出模式  是 **天作之合**。

## 2.1 什么是 “打字机” 式逐 token 输出？

“打字机” 式 逐 token 输出是一种**流式传输**方式，它模拟了人类打字或思考的过程。

服务器不是等待 LLM 生成整个答案 后一次性发送给 用户，而是 流式输出， **每生成一个 “词元”（token，可以粗略理解为一个词或一个字），就立刻发送这个 “词元”**。

下面举一个例子，对比 一下  传统方式（非流式）和 “打字机” （流式）式 的过程。

**传统方式（非流式）过程如下：**

1、你提问：“请写一首关于春天的诗”。

2、服务器端的 AI 开始思考、生成，整个过程你需要等待（可能好几秒甚至更久）。

3、AI 生成完整的诗歌：“春风拂面绿意浓，百花争艳映晴空...”。

4、服务器将**整首诗**作为一个完整的 JSON 对象 `{ "content": "春风拂面绿意浓，百花争艳映晴空..." }` 发送给客户端。

5、客户端一次性收到全部内容并渲染出来。

**“打字机”（流式）过程如下：**

1、你提问：“请写一首关于春天的诗”。

2、服务器端的 AI 生成第一个 token “春”，**立刻**通过 SSE 发送 `data: “春”`。

3、客户端收到 “春” 并显示出来。

4、AI 生成第二个 token “风”，**立刻**发送 `data: “风”`。

5、客户端在 “春” 后面追加“风”，形成“春风”。

6、后续 token “拂”、“面”、“绿”、“意”、“浓”... 依次迅速发送和追加。

7、你看到的效果就是**文字一个接一个地 “打” 在屏幕上**，就像有人在远端为你实时打字一样。

**“打字机”（流式） 模式的巨大优势：**

1、**极低的感知延迟**：用户几乎在提问后瞬间就能看到第一个字开始输出，无需经历漫长的等待白屏期，体验流畅自然。

2、**提供了 “正在进行” 的反馈**：看着文字逐个出现，给人一种模型正在为你 “思考” 和“创作”的生动感，而不是在“沉默中宕机”。

3、**更高效地利用时间**：用户可以在前半句还在输出时，就开始阅读和理解，节省了总体的认知时间。

## 2.2 为什么 SSE 是这种模式的 “天作之合”？

这正是 SSE 的设计初衷和核心优势所在，它与 AI 流式输出的需求完美匹配：

1、**单向通信的完美匹配**：

AI 的文本生成过程本质上是**服务器到客户端的单向数据推送**。

客户端只需要接收，不需要在生成过程中频繁地发送请求。

SSE 的 “服务器推送” 模型正是为此而生，而 WebSocket 的双向能力在这里是多余的。

2、**基于 HTTP/HTTPS，简单且兼容**：

SSE 使用标准的 HTTP 协议，这意味着 **SSE 易于实现和调试**：任何后端框架和前端语言都能轻松处理。在浏览器中调试时，你可以在 “网络” 选项卡中直接看到以文本流形式传输的事件，非常直观。

SSE 使用标准的 HTTP 协议，这还意味着 **容易绕过网络障碍**：公司防火墙和代理通常对 HTTP/HTTPS 放行，而可能会阻拦陌生的 WebSocket 协议。这使得 SSE 的部署兼容性极好。

3、**内置的自动重连机制**：

网络连接并不完全可靠。

如果用户在接收很长的回答时网络波动，连接中断，SSE 客户端会自动尝试重新连接。

这对于长时间流的应用至关重要，提供了天然的鲁棒性。

4、**轻量级的文本协议**：

AI 流式输出传输的就是文本（UTF-8 编码）。SSE 的协议 `data: ...\n\n` 就是为传输文本片段而设计的，极其高效和简单。

WebSocket 虽然也能传文本，但其协议设计还考虑了二进制帧、掩码等更复杂的情况，对于纯文本流来说显得有些 “重”。

5、**原生浏览器 API**：

现代浏览器都原生支持 `EventSource` API，开发者无需引入额外的第三方库，即可轻松实现接收流式数据，减少了依赖和打包体积。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WM9jp4Ufo3GuDFRCziaY8F7TXDZvgS4bH2slsH3xia0ywfyOTZVibgeNSRC43l1dlxylEYVuYvtYkxeQ/640?from=appmsg&watermark=1#imgIndex=9)

所以，**SSE 站到聚光灯下的原因正是：**

**AI 应用需要 “打字机” 式的逐 token 输出体验，而 SSE 作为一种基于 HTTP 的、简单的、单向的服务器推送技术，是实现这种体验最自然、最高效、最可靠的技术选择。**

它就像是为这个场景量身定做的工具，没有多余的功能，只有恰到好处的设计。

因此，当 ChatGPT 等应用席卷全球时，其背后默默无闻的 SSE 技术也终于从幕后走到了台前，被广大开发者所重新认识和重视。

# 三、SSE 的工作原理

## 3.1 工作机制的流程图

SSE 通过一个持久的 HTTP 连接实现服务器到客户端的单向数据流。

以下是其工作机制的流程图：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WM9jp4Ufo3GuDFRCziaY8F7TYldI3ogYyMIHmwDvOGibcp6alHdhAmMiamE0Ymj9yWUdNL6bhWQ1J4qA/640?from=appmsg&watermark=1#imgIndex=10)

**关键步骤解析：**

**1、浏览器发起一个 HTTP 请求，Header 中包含：**

```
```   Accept: text/event-stream
```

**2、服务器响应类型必须为：**

```
```   Content-Type: text/event-stream   Cache-Control: no-cache   Connection: keep-alive
```

**3、服务器发送事件格式（每个事件以两个换行符结束）：**

```
```   event: message   data: {"time": "2023-10-05T12:00:00", "value": "New update!"}   id: 12345   retry: 5000   \n\n
```

**4、浏览器通过 `EventSource`API 接收并处理事件。**

**5、服务器发送 一个特殊 “结束” 事件，可以结束传输。**

比如，服务器发送一个如 `event: end` 的消息，可以结束传输。

客户端预先监听这个自定义的 `end` 事件，一旦收到，就知道传输结束，并可以**选择主动关闭 EventSource 连接**。

**6、若连接中断，浏览器会根据 `retry`字段自动重连。**

如果没有收到  特殊 “结束” 事件， 浏览器 可以自动重连。

## 3.2、SSE 与其他通信方式对比

不同通信技术各有适用场景，我们用表格清晰对比：

<table><thead><tr><td><span><strong><span leaf="">技术</span></strong></span></td><td><span><strong><span leaf="">通信方向</span></strong></span></td><td><span><strong><span leaf="">基于协议</span></strong></span></td><td><span><strong><span leaf="">复杂度</span></strong></span></td><td><span><strong><span leaf="">适用场景</span></strong></span></td><td><span><strong><span leaf="">浏览器原生支持</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">轮询（Polling）</span></span></section></td><td><section><span><span leaf="">客户端→服务器</span></span></section></td><td><section><span><span leaf="">HTTP</span></span></section></td><td><section><span><span leaf="">简单</span></span></section></td><td><section><span><span leaf="">数据更新频率低（如定时查邮件）</span></span></section></td><td><section><span><span leaf="">全部</span></span></section></td></tr><tr><td><section><span><span leaf="">长轮询（Long Polling）</span></span></section></td><td><section><span><span leaf="">客户端→服务器</span></span></section></td><td><section><span><span leaf="">HTTP</span></span></section></td><td><section><span><span leaf="">中等</span></span></section></td><td><section><span><span leaf="">低频但需实时（如即时消息提醒）</span></span></section></td><td><section><span><span leaf="">全部</span></span></section></td></tr><tr><td><section><span><span leaf="">SSE</span></span></section></td><td><section><span><span leaf="">服务器→客户端</span></span></section></td><td><section><span><span leaf="">HTTP</span></span></section></td><td><section><span><span leaf="">简单</span></span></section></td><td><section><span><span leaf="">持续单向推送（如实时日志、股价、chartGPT）</span></span></section></td><td><section><span><span leaf="">（除 IE）</span></span></section></td></tr><tr><td><section><span><span leaf="">WebSocket</span></span></section></td><td><section><span><span leaf="">双向</span></span></section></td><td><section><span><span leaf="">自定义协议</span></span></section></td><td><section><span><span leaf="">复杂</span></span></section></td><td><section><span><span leaf="">互动性强（如在线游戏、视频聊天）</span></span></section></td><td><section><span><span leaf="">全部</span></span></section></td></tr></tbody></table>

简单说：

*   只需服务器 "说话" 选 SSE
    
*   需要双方 "对话" 选 WebSocket
    
*   偶尔查一次数据选轮询 / 长轮询
    

## 3.3 SSE 的适用场景

**1、ChatGPT 式逐字输出 (“打字机” 式逐 词元 token 输出)**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WM9jp4Ufo3GuDFRCziaY8F7TAFIibHVLHJguAjT8ibicYhQmEVtQc5qvsU0uIqJfaXuwKrXibwM125849Q/640?from=appmsg&watermark=1#imgIndex=11)

**2、实时通知系统**

*   新订单提醒
    
*   用户消息推送
    
*   审核状态更新
    

**3、实时数据看板**

*   股票行情
    
*   设备监控数据
    
*   实时日志流
    

# 四、sse 客户端 API 详解

SSE 的客户端实现非常简单，浏览器原生提供了`EventSource`对象来处理与服务器的 SSE 连接。下面我们详细介绍它的使用方法和核心特性。

## 4.1、认识 浏览器  EventSource 对象

### 浏览器兼容性检测

在使用 SSE 前，首先需要确认当前浏览器是否支持`EventSource`（除 IE/Edge 外，几乎所有现代浏览器都支持）。检测方法如下：

```
// 检查浏览器是否支持SSEif ('EventSource' in window) {  // 支持SSE，可正常使用  console.log('浏览器支持SSE');} else {  // 不支持SSE，需降级处理  console.log('浏览器不支持SSE');}
```

### 创建连接

使用`EventSource`创建与服务器的连接非常简单，只需传入服务器的 SSE 接口地址：

```
// 建立与服务器的SSE连接// url为服务器提供的SSE接口地址（可同域或跨域）var source = new EventSource(url);
```

如果需要跨域请求并携带 Cookie，可通过第二个参数配置：

```
// 跨域请求时，允许携带Cookievar source = new EventSource(url, {   withCredentials: true  // 默认为false，设为true表示跨域请求携带Cookie});
```

### 连接状态（readyState）

`EventSource`实例的`readyState`属性用于表示当前连接状态，只读且有三个可能值：

<table><thead><tr><td><span><strong><span leaf="">值</span></strong></span></td><td><span><strong><span leaf="">常量对应</span></strong></span></td><td><span><strong><span leaf="">含义说明</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">0</span></span></section></td><td><section><span><span leaf="">EventSource.CONNECTING</span></span></section></td><td><section><span><span leaf="">连接未建立，或断线后正在重连</span></span></section></td></tr><tr><td><section><span><span leaf="">1</span></span></section></td><td><section><span><span leaf="">EventSource.OPEN</span></span></section></td><td><section><span><span leaf="">连接已建立，可正常接收服务器推送的数据</span></span></section></td></tr><tr><td><section><span><span leaf="">2</span></span></section></td><td><section><span><span leaf="">EventSource.CLOSED</span></span></section></td><td><section><span><span leaf="">连接已关闭，且不会自动重连</span></span></section></td></tr></tbody></table>

可以通过该属性判断当前连接状态，例如：

```
if (source.readyState === EventSource.OPEN) {  console.log('SSE连接已正常建立');}
```

## 4.2、基本使用方法

`EventSource`通过事件机制处理连接过程中的各种状态和接收的数据，核心事件包括`open`、`message`、`error`。

下面用流程图展示 SSE 客户端的完整使用流程：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WM9jp4Ufo3GuDFRCziaY8F7T9N6EDWfYdK3qPWKW9moAcXhgA94CBUBcX7X6dzCnWHSM2wxuxicFWMQ/640?from=appmsg&watermark=1#imgIndex=12)

### 连接建立：open 事件

当客户端与服务器成功建立 SSE 连接时，会触发`open`事件：

```
// 方式1：使用onopen属性source.onopen = function (event) {  console.log('SSE连接已建立');  // 可在此处做连接成功后的初始化操作，如更新UI状态};// 方式2：使用addEventListener（推荐，可添加多个回调）source.addEventListener('open', function (event) {  console.log('SSE连接已建立（监听方式）');}, false);
```

### 接收数据：message 事件

当客户端收到服务器推送的数据时，会触发`message`事件（默认事件，处理未指定类型的消息）：

```
// 方式1：使用onmessage属性source.onmessage = function (event) {  // event.data为服务器推送的文本数据  var data = event.data;  console.log('收到数据：', data);  // 可在此处处理数据，如更新页面内容};// 方式2：使用addEventListenersource.addEventListener('message', function (event) {  var data = event.data;  console.log('收到数据（监听方式）：', data);}, false);
```

注意：`event.data`始终是字符串类型，如果服务器发送的是 JSON 数据，需要用`JSON.parse(data)`转换。

### 连接错误：error 事件

当连接发生错误（如网络中断、服务器出错）时，会触发`error`事件：

```
// 方式1：使用onerror属性source.onerror = function (event) {  // 可根据readyState判断错误类型  if (source.readyState === EventSource.CONNECTING) {    console.log('连接出错，正在尝试重连...');  } else {    console.log('连接已关闭，无法重连');  }};// 方式2：使用addEventListenersource.addEventListener('error', function (event) {  // 错误处理逻辑}, false);
```

### 关闭连接：close() 方法

如果需要主动关闭 SSE 连接（关闭后不会自动重连），可调用`close()`方法：

```
// 主动关闭SSE连接source.close();console.log('SSE连接已手动关闭');
```

## 4.3、自定义事件

默认情况下，服务器推送的消息会触发`message`事件。

但实际开发中，我们可能需要区分不同类型的消息（如 "新订单通知" 和 "系统公告"），这时就可以使用**自定义事件**。

客户端通过`addEventListener`监听自定义事件名，例如监听`order`事件：

```
// 监听名为"order"的自定义事件source.addEventListener('order', function (event) {  var orderData = event.data;  console.log('收到新订单：', orderData);  // 处理订单相关逻辑}, false);// 再监听一个名为"notice"的自定义事件source.addEventListener('notice', function (event) {  var noticeData = event.data;  console.log('收到系统公告：', noticeData);  // 处理公告相关逻辑}, false);
```

注意：自定义事件不会触发`message`事件，只会被对应的`addEventListener`捕获。

上面代码中，浏览器对 SSE 的`foo``notice`事件进行监听。如何实现服务器发送`foo``notice`事件，请看下文。

# 五、SSE 服务器实现：数据格式与规则

服务器要实现 SSE，核心是按照特定格式向客户端发送数据。

下面详细介绍服务器端的实现规范。

## 5.1 HTTP 头信息要求

服务器向客户端发送 SSE 数据时，必须设置以下 HTTP 响应头，否则客户端无法正确识别为事件流：

```
Content-Type: text/event-stream  // 必须，指定为事件流类型Cache-Control: no-cache          // 必须，禁止缓存，确保数据实时性Connection: keep-alive           // 必须，保持长连接
```

这三个头信息是 SSE 的基础，缺少任何一个都可能导致连接失败或数据异常。

## 5.2 数据传输格式

服务器发送的每条消息（message）由多行组成，每行格式为`[字段]: 值\n`（字段名后必须跟冒号和空格，结尾用换行符`\n`）。

多条消息之间用`\n\n`（两个换行符）分隔。

此外，以`:`开头的行是注释（服务器可定期发送注释保持连接）。

**基本格式示例**

```
: 这是一条注释（客户端会忽略）\ndata: 这是第一条消息\n\ndata: 这是第二条消息的第一行\ndata: 这是第二条消息的第二行\n\n
```

注意：换行符必须是`\n`（Unix 格式），`\r\n`可能导致客户端解析错误。

## 5.3 核心字段说明

SSE 消息支持四个核心字段，分别用于不同场景：

### 1. data 字段：消息内容

`data`字段用于携带实际的消息内容，是最常用的字段。

*   单行数据：
    

```
``` data: Hello, SSE!\n\n  // 单行数据，以\n\n结束
```

*   多行数据（适合 JSON 等复杂结构）：
    

```
```  data: {\n               // 第一行以\n结束  data: "name": "张三",\n  // 第二行以\n结束  data: "age": 20\n        // 第三行以\n结束  data: }\n\n              // 最后一行以\n\n结束
```

客户端接收后，`event.data`会自动拼接为完整字符串：`{"name": "张三","age": 20}`

### 2. event 字段：指定事件类型

`event`字段用于指定消息的事件类型，客户端可通过对应事件名监听（即 3.3 节的自定义事件）。

服务器发送：

```
event: order\n           // 指定事件类型为orderdata: 新订单ID：12345\n   // 消息内容\n                       // 消息结束（\n\n简化为单独一行）
```

客户端监听：

```
source.addEventListener('order', function(event) {  console.log(event.data);  // 输出：新订单ID：12345});
```

### 3. id 字段：消息标识

`id`字段用于给消息设置唯一标识，客户端会自动记录最后一条消息的`id`（存于`source.lastEventId`）。

**核心作用**：当连接断线重连时，客户端会在请求头中携带`Last-Event-ID: [最后收到的id]`，服务器可根据该 ID 恢复数据传输（避免重复或丢失）。

服务器发送：

```
id: msg1001\n            // 消息标识data: 这是第1001条消息\n\n
```

客户端重连时的请求头：

```
Last-Event-ID: msg1001  // 自动携带最后收到的id
```

### 4. retry 字段：重连间隔

`retry`字段用于指定客户端断线后的重连间隔（单位：毫秒），默认重连间隔约为 3 秒。

服务器发送：

```
retry: 5000\n            // 告诉客户端，断线后5秒再重连data: 重连间隔已设置为5秒\n\n
```

### 5. 服务器保持连接示例

服务器可以定期发送注释行，保持连接活跃：

```
: 这是保持连接活动的注释行\n: 服务器时间 2023-10-05T12:00:00\n
```

## 5.4 服务器发送流程

服务器发送 SSE 数据的完整流程如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WM9jp4Ufo3GuDFRCziaY8F7TfpRpZdjz3nr3gID0JlfgwmgBl3biaHRRdmDWiaJsghroYIkfQ1hI9EJg/640?from=appmsg&watermark=1#imgIndex=13)

下面是一个包含多种字段的服务器发送示例，模拟一个实时通知系统：

```
: 服务器开始发送消息（注释）\nid: 1001\nevent: notice\ndata: 系统将在10分钟后维护\n\nid: 1002\nevent: order\ndata: {"orderId": "20230501", "status": "paid"}\n\nretry: 10000\nid: 1003\ndata: 重连间隔已调整为10秒\n\n: 这是保持连接活动的注释行\n: 服务器时间 2023-10-05T12:00:00\n
```

客户端接收后：

*   `notice`事件会捕获到 "系统将在 10 分钟后维护"
    
*   `order`事件会捕获到订单 JSON 数据
    
*   重连间隔被设置为 10 秒
    
*   最后收到的消息 ID 是 1003（断线重连时会携带）
    

通过以上规范，服务器就能轻松实现 SSE 功能，向客户端实时推送数据。

相比 WebSocket，SSE 的服务器实现更简单，无需处理复杂的协议握手，只需按格式发送文本数据即可。

# 六、SSE 实战案例：用 Spring Boot 搭建实时通信系统

接下来， 通过一个完整案例  手把手教你用 Spring Boot 实现 SSE 功能。

这个案例包含服务端（后端）和客户端（前端）代码， 可以直接运行体验服务器主动推送数据的效果。

## 6.1、案例整体架构

我们要实现的系统包含三个核心部分：

*   后端服务：基于 Spring Boot，提供 SSE 连接接口、消息广播接口和任务进度推送接口
    
*   前端页面：一个简单的 HTML 页面，通过`EventSource`与后端建立 SSE 连接
    
*   交互流程：客户端连接后，可接收服务器主动推送的连接状态、广播消息和任务进度
    

整体架构流程图：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WM9jp4Ufo3GuDFRCziaY8F7TVhQqMUnPyOwoFEnrY7PEagz8pQtKUURsz0RrhPuLCqv0HMaibIzENwQ/640?from=appmsg&watermark=1#imgIndex=14)

## 6.2、服务端实现

### 6.2.1 准备依赖

首先创建 Spring Boot 项目，在`pom.xml`中添加以下依赖（用于 web 开发和页面渲染）：

```
<dependencies>    <!-- Spring Web：提供SSE相关类和HTTP服务 -->    <dependency>        <groupId>org.springframework.boot</groupId>        <artifactId>spring-boot-starter-web</artifactId>    </dependency>    <!-- Thymeleaf：用于渲染前端页面 -->    <dependency>        <groupId>org.springframework.boot</groupId>        <artifactId>spring-boot-starter-thymeleaf</artifactId>    </dependency></dependencies>
```

这些依赖是基础：`spring-boot-starter-web`提供了 SSE 核心类`SseEmitter`，`spring-boot-starter-thymeleaf`用于将 HTML 页面返回给浏览器。

### 6.2.2 编写 SSE 核心控制器

创建`SseController`，这是服务端处理 SSE 连接和消息推送的核心类：

```
package com.example.sse.controller;import org.springframework.http.MediaType;import org.springframework.web.bind.annotation.GetMapping;import org.springframework.web.bind.annotation.RequestParam;import org.springframework.web.bind.annotation.RestController;import org.springframework.web.servlet.mvc.method.annotation.SseEmitter;import java.io.IOException;import java.util.concurrent.CopyOnWriteArrayList;import java.util.concurrent.ExecutorService;import java.util.concurrent.Executors;@RestControllerpublic class SseController {    // 存储所有活跃的SSE连接（线程安全的列表）    // CopyOnWriteArrayList适合读多写少场景，避免并发问题    private final CopyOnWriteArrayList<SseEmitter> emitters = new CopyOnWriteArrayList<>();    // 线程池：用于异步发送事件，避免阻塞主线程    private final ExecutorService executor = Executors.newCachedThreadPool();    /     * 客户端订阅SSE的接口     * 客户端通过访问该接口建立长连接，接收服务器推送的事件     */    @GetMapping(value = "/sse/subscribe", produces = MediaType.TEXT_EVENT_STREAM_VALUE)    public SseEmitter subscribe() {        // 创建SseEmitter实例，设置超时时间为无限（默认30秒会超时，这里设为Long.MAX_VALUE避免自动断开）        SseEmitter emitter = new SseEmitter(Long.MAX_VALUE);        // 将新连接加入活跃列表（后续推送消息时会遍历这个列表）        emitters.add(emitter);        // 设置连接完成/超时的回调：从活跃列表中移除该连接，释放资源        emitter.onCompletion(() -> emitters.remove(emitter)); // 连接正常关闭时        emitter.onTimeout(() -> emitters.remove(emitter));     // 连接超时关闭时        // 发送初始连接成功消息（给客户端的"欢迎消息"）        try {            emitter.send(SseEmitter.event()                    .name("CONNECTED")  // 事件名称：客户端可通过"CONNECTED"事件监听                    .data("You are successfully connected to SSE server!")  // 消息内容                    .reconnectTime(5000)); // 告诉客户端：如果断开连接，5秒后重连        } catch (IOException e) {            // 发送失败时，标记连接异常结束            emitter.completeWithError(e);        }        return emitter; // 将emitter返回给客户端，保持连接    }    /     * 广播消息接口：向所有已连接的客户端推送消息     * 可通过浏览器访问 http://localhost:8080/sse/broadcast?message=xxx 触发/    @GetMapping("/sse/broadcast")    public String broadcastMessage(@RequestParam String message) {        // 用线程池异步执行广播，避免阻塞当前请求        executor.execute(() -> {            // 遍历所有活跃连接，逐个发送消息            for (SseEmitter emitter : emitters) {                try {                    emitter.send(SseEmitter.event()                            .name("BROADCAST")  // 事件名称：客户端监听"BROADCAST"事件                            .data(message)      // 广播的消息内容                            .id(String.valueOf(System.currentTimeMillis()))); // 消息ID（用于重连时定位）                } catch (IOException e) {                    // 发送失败（可能客户端已断开），从列表中移除并标记连接结束                    emitters.remove(emitter);                    emitter.completeWithError(e);                }            }        });        return "Broadcast message: " + message; // 给调用者的响应    }    /     * 模拟长时间任务：向客户端推送实时进度     * 适合文件上传、数据处理等需要实时反馈进度的场景/    @GetMapping("/sse/start-task")    public String startTask() {        // 异步执行任务，避免阻塞当前请求        executor.execute(() -> {            try {                // 模拟任务进度：从0%到100%，每次增加10%                for (int i = 0; i <= 100; i += 10) {                    Thread.sleep(1000); // 休眠1秒，模拟处理耗时                    // 向所有客户端推送当前进度                    for (SseEmitter emitter : emitters) {                        try {                            emitter.send(SseEmitter.event()                                    .name("PROGRESS")  // 事件名称：客户端监听"PROGRESS"事件                                    .data(i + "% completed")  // 进度数据                                    .id("task-progress")); // 固定ID，标识这是任务进度消息                        } catch (IOException e) {                            // 发送失败，移除连接                            emitters.remove(emitter);                        }                    }                    // 任务完成时，发送结束消息                    if (i == 100) {                        for (SseEmitter emitter : emitters) {                            try {                                emitter.send(SseEmitter.event()                                        .name("COMPLETE")  // 事件名称：客户端监听"COMPLETE"事件                                        .data("Task completed successfully!"));                            } catch (IOException e) {                                emitters.remove(emitter);                            }                        }                    }                }            } catch (InterruptedException e) {                // 任务被中断时，恢复线程中断状态并退出                Thread.currentThread().interrupt();                break;            }        });        return "Task started!"; // 告诉调用者任务已启动    }}
```

#### 核心代码说明：

*   `SseEmitter`：Spring 提供的 SSE 核心类，每个实例对应一个客户端连接
    
*   `emitters`列表：管理所有活跃连接，方便广播消息（类似 "客户端注册表"）
    
*   `executor`线程池：异步处理消息发送，避免阻塞主线程（如果同步发送，一个客户端卡住会影响所有用户）
    
*   事件发送：通过`emitter.send(SseEmitter.event())`构建消息，可指定事件名、数据、ID 和重连时间
    

### 6.2.3 编写页面控制器

创建`PageController`，用于将前端页面返回给浏览器：

```
package com.example.sse.controller;import org.springframework.stereotype.Controller;import org.springframework.web.bind.annotation.GetMapping;@Controller // 注意这里用@Controller而非@RestController，用于返回页面public class PageController {    /*     * 访问根路径时，返回SSE客户端页面/    @GetMapping("/")    public String index() {        // 返回src/main/resources/templates目录下的sse-client.html        return "sse-client";    }}
```

## 6.3、客户端实现（HTML 页面）

在`src/main/resources/templates`目录下创建`sse-client.html`，这是用户交互的前端页面：

```
Server-Sent Events (SSE) Client            Connect to SSE                Disconnect                 Send Broadcast             Start Task                 Messages:
```

#### 客户端核心逻辑：

*   `eventSource`：`EventSource`实例，是客户端与服务端 SSE 连接的 "桥梁"
    
*   事件监听：通过`addEventListener`监听服务端定义的事件（`CONNECTED`/`BROADCAST`等）
    
*   自动重连：当连接断开时，`EventSource`会自动重试（无需手动写重连逻辑）
    
*   交互函数：`connectSSE`/`disconnectSSE`等函数对应页面按钮，实现用户操作
    

## 6.4、运行与测试

### 6.4.1 启动步骤

1、 确保 Spring Boot 项目配置正确（默认端口 8080，无需额外配置）2、 启动 Spring Boot 应用（运行带有`main`方法的启动类）3、 打开浏览器，访问 `http://localhost:8080/`，看到客户端页面

### 6.4.2 功能测试

**1、建立连接**：点击 "Connect to SSE" 按钮，页面会显示 "连接成功" 的消息（服务端通过`CONNECTED`事件推送）

**2、发送广播**：点击 "Send Broadcast" 按钮，输入任意消息（如 "Hello SSE"），页面会显示广播消息（服务端向所有连接的客户端推送）

**3、启动任务**：点击 "Start Task" 按钮，页面会每秒收到一条进度消息（从 0% 到 100%），最后显示 "任务完成"

**4、断开连接**：点击 "Disconnect" 按钮，连接关闭，不再接收消息

### 4.3 测试流程图

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WM9jp4Ufo3GuDFRCziaY8F7TzJTnV0eUNB5Toh2oPFibJDsr3kJbbYGejDwNNWahvaJaez9ic8hLHr3A/640?from=appmsg&watermark=1#imgIndex=15)

## 6.5、 服务端 关键技术点

**1、SseEmitter 的作用**：Spring 封装的 SSE 工具类，简化了 "保持连接 + 发送事件" 的实现，无需手动处理 HTTP 流格式。

**2、连接管理**：用`CopyOnWriteArrayList`存储活跃连接，确保线程安全；通过`onCompletion`/`onTimeout`回调清理无效连接，避免内存泄漏。

**3、异步处理**：必须用线程池（`ExecutorService`）异步发送消息，否则会阻塞主线程，导致新请求无法处理。

**4、事件设计**：通过`name`区分不同类型的事件（如`PROGRESS`/`BROADCAST`），客户端按需监听，逻辑更清晰。

**5、自动重连**：SSE 客户端（`EventSource`）内置重连机制，网络恢复后会自动重新连接，无需额外代码。

通过这个案例，你可以清晰看到 SSE 的优势：实现简单（几行代码就能建立实时连接）、无需额外协议（基于 HTTP）、自带重连机制。如果你的场景只需要服务器单向推送数据（如实时通知、进度更新），SSE 会是比 WebSocket 更轻量的选择。

# 七：SSE 与 WEBSocket 如何选型？

接下来，尼恩介绍一下 SSE 与 WEBSocket 如何选型？

## 7.1 sse 与 WEBSocket 全面对比

来对 Server-Sent Events (SSE) 和 WebSocket 进行一场全面、深入的对比。

<table><thead><tr><td><span><strong><span leaf="">特性</span></strong></span></td><td><span><strong><span leaf="">Server-Sent Events (SSE)</span></strong></span></td><td><span><strong><span leaf="">WebSocket</span></strong></span></td></tr></thead><tbody><tr><td><section><span><strong><span leaf="">通信方向</span></strong></span></section></td><td><section><span><strong><span leaf="">单向</span></strong><span leaf="">&nbsp;(服务器 -&gt; 客户端)</span></span></section></td><td><section><span><strong><span leaf="">全双工</span></strong><span leaf="">&nbsp;(服务器 &lt;-&gt; 客户端)</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">协议基础</span></strong></span></section></td><td><section><span><strong><span leaf="">HTTP</span></strong></span></section></td><td><section><span><strong><span leaf="">独立的 WS/WSS 协议</span></strong><span leaf="">&nbsp;(基于 HTTP 升级)</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">协议开销</span></strong></span></section></td><td><section><span><span leaf="">非常低 (轻量级文本数据)</span></span></section></td><td><section><span><span leaf="">极低 (每个消息帧头部开销仅 2-10 字节)</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">数据格式</span></strong></span></section></td><td><section><span><strong><span leaf="">文本</span></strong><span leaf="">&nbsp;(通常为 UTF-8)，支持结构化事件 ID、类型和数据</span></span></section></td><td><section><span><strong><span leaf="">文本</span></strong><span leaf="">&nbsp;或&nbsp;</span><strong><span leaf="">二进制</span></strong></span></section></td></tr><tr><td><section><span><strong><span leaf="">自动重连</span></strong></span></section></td><td><section><span><strong><span leaf="">原生支持</span></strong></span></section></td><td><section><span><span leaf="">需要手动实现</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">兼容性</span></strong></span></section></td><td><section><span><strong><span leaf="">良好</span></strong><span leaf="">&nbsp;(除 IE 外的主流浏览器)</span></span></section></td><td><section><span><strong><span leaf="">优秀</span></strong><span leaf="">&nbsp;(包括 IE 10+)</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">安全性</span></strong></span></section></td><td><section><span><span leaf="">基于 HTTPS 的安全性与 HTTP 相同</span></span></section></td><td><section><span><span leaf="">基于 WSS 的安全性与 HTTPS 相同</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">实现复杂度</span></strong></span></section></td><td><section><span><strong><span leaf="">非常简单</span></strong><span leaf="">&nbsp;(客户端和服务器端都易于实现)</span></span></section></td><td><section><span><strong><span leaf="">相对复杂</span></strong></span></section></td></tr></tbody></table>

为了更直观地理解两者的工作模式差异，请看下面的序列图：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WM9jp4Ufo3GuDFRCziaY8F7T88GuYP0YIvbUaiad7T8mJ8XfHM5iaR8FGknYKerAI0nVuIfaZxNSq3fQ/640?from=appmsg&watermark=1#imgIndex=16)

### 7.1.1. 协议与连接建立

**SSE 协议与连接建立**：

*   基于纯粹的 **HTTP**。客户端发起一个普通的 HTTP GET 请求，并携带特殊的头 `Accept: text/event-stream`。
    
*   服务器响应后，**保持这个 TCP 连接打开**，并开始发送数据流 ，直到遇到结束标记。
    
*   这是一种长连接的 HTTP 用法。
    

**WebSocket 协议与连接建立**：

*   基于独立的 **WebSocket 协议**。连接始于一个特殊的 HTTP 请求，即 **“协议升级” 请求**（`Connection: Upgrade`, `Upgrade: websocket`）。
    
*   服务器响应 `HTTP 101 Switching Protocols` 后，**最初的 HTTP 连接被替换为 WebSocket 连接**，此后通信不再遵循 HTTP 协议，而是在其之上建立一个全双工的通道。
    

### 7.1.2. 数据流与通信模式

*   **SSE**：**单向通信**。设计初衷就是让服务器能够主动、高效地向客户端推送数据。• 数据是**文本流**，格式简单且可读性强。每条消息可以附带一个事件类型（`event:`）和一个 ID（`id:`）。• 客户端使用 `EventSource` API 监听来自服务器的事件。
    
*   **WebSocket：全双工通信**。在连接建立后，客户端和服务器处于完全平等的地位，可以随时、任意地相互发送消息。 另外，WS 协议  支持**文本和二进制数据**，灵活性极高，非常适合需要频繁双向交互的场景（如游戏、协作编辑）。
    

### 7.1.3. 能力与特性

*   **SSE**：**内置自动重连机制**。如果连接断开，`EventSource` 对象会自动尝试重新连接，并在重连后自动发送上一个收到的事件 ID，服务器据此可判断错过了哪些消息，实现数据恢复。SSE 有出色的浏览器支持**。所有现代浏览器（Chrome, Firefox, Safari, Edge）都原生支持，**Internet Explorer （古代浏览器）完全不支持**（通常需要 polyfill 或降级方案）。**
    
*   WebSocket**：**无自动重连**。连接断开后，需要开发者手动编写重连逻辑和状态恢复逻辑。**WS 协议比 SSE 协议有更广泛的浏览器支持，包括 Internet Explorer 10+。
    

### 7.1.4. 开发与集成

*   **SSE**：**开发与集成 非常简单**。服务器端几乎不需要特殊的库，任何能输出 HTTP 流的后端语言都可以实现。客户端 API 也非常直观。与现有 **HTTP 认证**、**CORS** 机制完全兼容，处理方式与普通 HTTP 请求一致。
    
*   **WebSocket**：**开发与集成 相对复杂**。服务器端需要支持 WebSocket 协议的库（如 `ws` for Node.js, `Socket.IO` 等）。客户端需要处理连接状态、心跳包等。 虽然升级握手是 HTTP，但后续通信是独立协议，因此一些复杂的网络环境（如某些代理服务器）可能会带来问题。
    

## 7.2  如何选择？决策指南

选择的关键在于**应用场景和核心需求**。

### 7.2.1 选择 SSE 的场景：

*   **服务器到客户端的单向数据流**。
    
*   **简单和快速实现**是关键因素。
    
*   需要**自动错误恢复（重连）**。
    
*   数据传输格式是文本（如 JSON），且不需要二进制。
    

**SSE** 典型的应用场景包括：

*   **实时新闻推送**、体育比分更新。
    
*   **金融报价行情**（如股票价格变动）。
    
*   **社交媒体动态更新**（如 Twitter 时间线）。
    
*   **服务器日志流**监控。
    
*   **AI 处理进度**或结果的流式输出。
    

### 7.2.2 选择 WebSocket 场景：

*   **真正的实时双向通信**，客户端和服务器都需要频繁地发送消息。
    
*   需要传输**二进制数据**（如视频、音频、图像碎片）。
    
*   构建**交互性极强的应用**，其中低延迟至关重要。
    

**WebSocket** 典型的应用场景包括：

*   **在线聊天应用**（如 Slack, Discord）。
    
*   **多人在线游戏**。
    
*   **协同编辑工具**（如 Google Docs）。
    
*   **实时仪表盘和控制面板**（需要双向控制）。
    

<table><thead><tr><td><span><strong><span leaf="">特性</span></strong></span></td><td><span><strong><span leaf="">SSE</span></strong></span></td><td><span><strong><span leaf="">WebSocket</span></strong></span></td><td><span><strong><span leaf="">胜出方</span></strong></span></td></tr></thead><tbody><tr><td><section><span><strong><span leaf="">双向通信</span></strong></span></section></td><td><section><span><span leaf="">❌</span></span></section></td><td><section><span><span leaf="">✅</span></span></section></td><td><section><span><span leaf="">WebSocket</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">协议简单性</span></strong></span></section></td><td><section><span><span leaf="">✅</span></span></section></td><td><section><span><span leaf="">❌</span></span></section></td><td><section><span><span leaf="">SSE</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">自动重连</span></strong></span></section></td><td><section><span><span leaf="">✅</span></span></section></td><td><section><span><span leaf="">❌</span></span></section></td><td><section><span><span leaf="">SSE</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">二进制支持</span></strong></span></section></td><td><section><span><span leaf="">❌</span></span></section></td><td><section><span><span leaf="">✅</span></span></section></td><td><section><span><span leaf="">WebSocket</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">浏览器支持</span></strong></span></section></td><td><section><span><span leaf="">良好 (无 IE)</span></span></section></td><td><section><span><span leaf="">优秀</span></span></section></td><td><section><span><span leaf="">WebSocket (略优)</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">网络兼容性</span></strong></span></section></td><td><section><span><span leaf="">✅ (穿透性好)</span></span></section></td><td><section><span><span leaf="">⚠️ (可能被代理阻挡)</span></span></section></td><td><section><span><span leaf="">SSE</span></span></section></td></tr></tbody></table>

简单来说：

*   **SSE 是 “智能电台”**：你只管听，电台（服务器）会持续且可靠地向你广播节目。断了信号还会自己重搜频道。
    
*   **WebSocket 是 “电话线”**：你和对方可以随时、自由地对话，说任何形式的内容（文字或暗语），但电话线断了你得自己重拨。
    

## 7.3  WebSocket+SSE 混合架构

尼恩辅导了大量的顶级互联网 通讯案例，一般来说大型应用场景，强网用 WebSocket、弱网适合使用 SSE ，这就是 WebSocket+SSE 混合架构。

强网用 WebSocket、弱网自动降级到 SSE 的混合架构， 核心在于**网络质量动态评估**和**双通道无缝切换**。

WebSocket+SSE 混合架构 具体实现方案如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WM9jp4Ufo3GuDFRCziaY8F7TKJ3X5pHP8oI6MCJfADibzXCEjUxmnoqOkP9BibaTMcYxibNkbJuKOyOqw/640?from=appmsg&watermark=1#imgIndex=17)

### 1、核心模块：网络质量探针（客户端） 实现

```
class NetworkProbe {  // 关键指标  static RTT_THRESHOLD = 300 // RTT超过300ms视为弱网  static PACKET_LOSS_THRESHOLD = 0.2 // 丢包率>20%触发降级  // 网络状态检测  async check() {    const { rtt, packetLoss } = await this._measure()    return {      isWeak: rtt > NetworkProbe.RTT_THRESHOLD ||               packetLoss > NetworkProbe.PACKET_LOSS_THRESHOLD    }  }  // 实际测量方法  _measure() {    return new Promise(resolve => {      const start = Date.now()      fetch('/ping', { cache: 'no-store' })        .then(() => {          const rtt = Date.now() - start          resolve({ rtt, packetLoss: 0 })        })        .catch(() => resolve({ rtt: Infinity, packetLoss: 1 }))    })  }}
```

### 2、核心模块：双协议连接管理器（客户端）

```
class HybridConnection {  constructor() {    this.currentProtocol = null    this.ws = null    this.sse = null    this.messageQueue = [] // 消息缓冲队列  }  // 智能连接初始化  async connect() {    const { isWeak } = await new NetworkProbe().check()    this.currentProtocol = isWeak ? 'sse' : 'ws'    if (this.currentProtocol === 'ws') {      this._initWebSocket()    } else {      this._initSSE()    }  }  // WebSocket初始化  _initWebSocket() {    this.ws = new WebSocket('wss://api.example.com')    this.ws.onmessage = this._handleMessage    // 发送缓冲队列消息    this.messageQueue.forEach(msg => this.ws.send(msg))    this.messageQueue = []  }  // SSE初始化  _initSSE() {    this.sse = new EventSource('https://api.example.com/sse')    this.sse.onmessage = this._handleMessage  }  // 统一消息处理  _handleMessage = (event) => {    const data = event.data || event    // 业务逻辑处理...  }  // 发送消息（自动选择协议）  send(data) {    if (this.currentProtocol === 'ws' && this.ws?.readyState === 1) {      this.ws.send(JSON.stringify(data))    } else if (this.currentProtocol === 'sse') {      // SSE需通过独立HTTP请求发送      fetch('/send', { method: 'POST', body: JSON.stringify(data) })    } else {      // 协议切换中暂存消息      this.messageQueue.push(JSON.stringify(data))    }  }  // 协议切换（核心！）  async switchProtocol() {    const { isWeak } = await new NetworkProbe().check()    // 无需切换    if (isWeak && this.currentProtocol === 'sse') return    if (!isWeak && this.currentProtocol === 'ws') return    // 执行切换    if (isWeak) {      this.ws?.close()      this._initSSE()      this.currentProtocol = 'sse'    } else {      this.sse?.close()      this._initWebSocket()      this.currentProtocol = 'ws'    }  }}
```

## 7.4  chat2ai 是选择 sse 协议还是 WEBSocket

**直接答案：**

*   **对于绝大多数 chat2ai 应用  优先选择 SSE (Server-Sent Events)。**
    
*   复杂的  chat2ai 应用  优先选择 WEBSocket。
    

但这并非绝对，我们需要根据具体的功能需求来决定。下面我将为你进行详细的分析和推理。

### 7.4 .1 核心决策分析

AI 聊天应用的核心交互是：

1、**客户端**发送一条消息（一个问题）。

2、**服务器**接收后，调用大语言模型（LLM）API。

3、**服务器**将模型**流式**返回的答案（逐词或逐句）实时推送给客户端。

4、**客户端**实时渲染这个流式的答案，营造出 “打字机” 效果。

这个过程的关键在于**第 3 步**，即**服务器向客户端的单向数据推送**。这正是 SSE 的绝对主场。

### 7.4 .2 为什么 SSE 是更优的选择？

以下流程图清晰地展示了基于不同技术方案的聊天交互过程，其中突出了 SSE 方案的巨大优势：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WM9jp4Ufo3GuDFRCziaY8F7T5kBgsYnhsicTS0bgfLVkqWjLdwZhPZpVNBDaUMibcaW7d6DVXZkss18A/640?from=appmsg&watermark=1#imgIndex=18)

正如上图所示，SSE 方案在实现上更加直接和高效，因为它基于 HTTP，并且专门为服务器到客户端的单向数据流设计。

此外，SSE 还带来了以下巨大优势：

1、**开发复杂度极低**：   - **后端**：你不需要引入任何复杂的 WebSocket 库（如 `ws`, `Socket.IO`）。你只需要建立一个普通的 HTTP 路由（如 `POST /chat` 用于发送消息，`GET /chat/stream` 用于接收流），并在控制器中输出 `text/event-stream` 格式的响应流。

*   **前端**：使用浏览器原生的 `EventSource` API 即可轻松监听数据流，几行代码就能实现。无需实例化和管理 WebSocket 连接对象。
    

2、**出色的兼容性与可维护性**：

*   SSE 基于 HTTP，这意味着它更容易通过公司防火墙、代理，与现有的认证系统（如 Cookie、JWT）、CORS 策略协同工作，几乎不会遇到奇怪的网络问题。
    
*   在浏览器 “网络” 选项卡中，SSE 的流清晰可见，易于调试。每个消息都是可读的文本，调试体验非常好。
    

3、**内置的自动重连与断点续传机制**：

*   这是 SSE 的 “杀手级特性”。网络连接不稳定是移动端的常见问题。如果用户在接收一个很长答案的过程中网络中断，SSE 会在网络恢复后**自动重新连接**。
    
*   更强大的是，SSE 协议支持发送最后一个消息的 ID。服务器可以识别出这个 ID，并判断客户端错过了哪些数据，从而**从断点处继续发送**，而不是重新开始生成整个回答。这既节省了昂贵的 API 调用费用，也提升了用户体验。**这在 WebSocket 中需要手动实现所有逻辑，非常复杂。**
    

### 7.4.3  WebSocket 的适用场景

虽然 SSE 是主流选择，但在 **chat2ai** 应用变得非常复杂时，WebSocket 可能会成为更好的选择。

**在以下情况下， 应该考虑使用 WebSocket：**

1、**需要极高频的双向通信**：不仅仅是用户提问 ->AI 回答。例如：

*   **实时协作编辑**：多个用户同时编辑一份由 AI 生成的文档，每个人的输入都需要实时同步给其他所有人。
    
*   **AI 多人游戏**：基于 AI 生成剧情和环境的实时互动游戏，玩家的每一个动作都需要实时影响虚拟世界。
    

2、**当需要传输二进制数据的时候**：

*   有的聊天应用不仅支持文本，还支持**实时语音对话**（客户端录音发送二进制音频流，服务器返回 AI 语音二进制流）。WebSocket 对二进制数据的支持是天生的。
    

3、**你需要非常精确的控制心跳和连接状态**：   - WebSocket 允许 手动发送 Ping/Pong 帧来检测连接活性，虽然复杂，但给了开发人员最大的控制权。

### 7.4.4 传输协议选型 结论与建议

1、**起步和绝大多数情况：从 SSE 开始。**

这是最直接、最高效、最能给你带来稳定体验的选择。

使用 sse 遇到的技术挑战会更少，开发速度更快。

ChatGPT、Claude 等绝大多数顶级应用都使用 SSE 不是没有道理的。

2、**未来如果需要扩展：采用混合架构。**

如果应用未来需要加入上述 WebSocket 的适用功能（如实时语音），完全可以**同时使用两种协议**：

*   使用 **SSE** 专门处理 **AI 文本答案的流式推送**。
    
*   使用 **WebSocket** 专门处理 **实时语音、实时协作**等真正的双向通信功能。
    
*   或者 强弱结合，自动切换。
    

**因此，对于  chat2ai 的传输协议选型答案是：优先选择 SSE。**

<table><thead><tr><td><span><strong><span leaf="">特性</span></strong></span></td><td><span><strong><span leaf="">SSE</span></strong></span></td><td><span><strong><span leaf="">WebSocket</span></strong></span></td><td><span><strong><span leaf="">对 chat2ai 的启示</span></strong></span></td></tr></thead><tbody><tr><td><section><span><strong><span leaf="">主要方向</span></strong></span></section></td><td><section><span><strong><span leaf="">服务器 -&gt; 客户端</span></strong></span></section></td><td><section><span><strong><span leaf="">客户端 &lt;-&gt; 服务器</span></strong></span></section></td><td><section><span><strong><span leaf="">AI 回答是单向流，SSE 更匹配</span></strong></span></section></td></tr><tr><td><section><span><strong><span leaf="">协议</span></strong></span></section></td><td><section><span><span leaf="">HTTP</span></span></section></td><td><section><span><span leaf="">WS (独立协议)</span></span></section></td><td><section><span><strong><span leaf="">SSE 更简单，兼容性更好</span></strong></span></section></td></tr><tr><td><section><span><strong><span leaf="">自动重连</span></strong></span></section></td><td><section><span><strong><span leaf="">原生支持</span></strong></span></section></td><td><section><span><span leaf="">需手动实现</span></span></section></td><td><section><span><strong><span leaf="">SSE 提供巨大优势，提升用户体验</span></strong></span></section></td></tr><tr><td><section><span><strong><span leaf="">数据格式</span></strong></span></section></td><td><section><span><span leaf="">文本</span></span></section></td><td><section><span><span leaf="">文本 &amp;&nbsp;</span><strong><span leaf="">二进制</span></strong></span></section></td><td><section><span><span leaf="">纯文本 AI 回答，SSE 足够</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">复杂度</span></strong></span></section></td><td><section><span><strong><span leaf="">低</span></strong></span></section></td><td><section><span><span leaf="">高</span></span></section></td><td><section><span><strong><span leaf="">SSE 开发更快，更易维护</span></strong></span></section></td></tr></tbody></table>

## 说在最后：有问题找老架构取经‍

只要按照上面的 尼恩团队梳理的 方案去作答， 你的答案不是 100 分，而是 120 分。  面试官一定是 心满意足， 五体投地。

按照尼恩的梳理，进行 深度回答，可以充分展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer 直提”。

在面试之前，建议大家系统化的刷一波 5000 页《[尼恩 Java 面试宝典 PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》，里边有大量的大厂真题、面试难题、架构难题。

很多小伙伴刷完后， 吊打面试官， 大厂横着走。

在刷题过程中，如果有啥问题，大家可以来 找 40 岁老架构师尼恩交流。

另外，如果没有面试机会， 可以找尼恩来改简历、做帮扶。前段时间，**[刚指导一个小伙 暴涨 200%（涨 2 倍），29 岁 / 7 年 / 双非一本 ， 从 13K 一次涨到 37K ，逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486433&idx=1&sn=0f630e8fc7620b3bf0e054827b48d91e&scene=21#wechat_redirect)。**

狠狠卷，实现 “offer 自由” 很容易的， 前段时间一个武汉的跟着尼恩卷了 2 年的小伙伴， 在极度严寒 / 痛苦被裁的环境下， offer 拿到手软， 实现真正的 “offer 自由” 。

## 惊天大逆袭： 通过  Java+AI  实现弯道超车，  完成转架构 

 [会 AI 的程序员，工资暴涨 50%！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486827&idx=1&sn=db3f268c6752ddb074a98c7b13a84011&scene=21#wechat_redirect)

  

[极速上岸： 被裁 后， 8 天 拿下 京东，狠涨 一倍 年薪 48W， 小伙伴 就是 做对了一件事](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486913&idx=1&sn=edd6774e9d17c39327e8dcb95fdd68d9&scene=21#wechat_redirect)

  

[外包 + 二本 可以进 美团： 26 岁小 2 本 一步登天， 进了顶奢大厂（ 美团） ， 太爽了](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486904&idx=1&sn=e7c878f99ed101ebfbd1d80ec0e07401&scene=21#wechat_redirect)

  

[暴涨 150%，4 年 CRUD 一步登天， 进  ‘宇宙厂’， 26 岁 小伙 6 个月 大逆袭](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486898&idx=1&sn=c1f7b50eb13ad990a19f099d9e608231&scene=21#wechat_redirect)

  

[Java+Al 大逆袭 1： 34 岁无路可走，一个月翻盘，拿 3 个架构 offer，靠  Java+Al   逆天改命！！！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486848&idx=1&sn=ff43d058271b0801f84aba3f3d957855&scene=21#wechat_redirect)

  

java+AI 大逆袭 2：[：3 年 程序媛 被裁， 25W-》40W 上岸， 逆涨 60%。 Java+AI 太神了， 架构小白 2 个月逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486858&idx=1&sn=9cd122c18d166433b43d0952d3a7a7a8&scene=21#wechat_redirect)

  

  

[Java+AI 逆袭 ： 36 岁 / 失业 7 个月 / 彻底绝望 。狠卷 3 个月 Java+AI ，终于逆风翻盘，顺利 上岸](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486870&idx=1&sn=990a540a155cdb8254ebce6c75816882&scene=21#wechat_redirect)

  

[Java+AI 逆袭 ： 闲了一年，41 岁 / 失业 12 个月 / 彻底绝望 。狠卷 2 个月 Java+AI ，终于逆风翻盘](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486880&idx=1&sn=a37033157c766ac5ae7c9195fe776693&scene=21#wechat_redirect)

## 冲大厂 案例：  全网顶尖、高薪案例， 进大厂拿高薪， 实现薪酬腾飞、人生逆袭 

[涨一倍：从 30 万 涨 60 万，3 年经验小伙 冲大厂成功，逆天了 ！！！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486818&idx=1&sn=ec5ff9c809133d903bdecddcd25c440e&scene=21#wechat_redirect)

[阿里 + 美团 offer：25 岁 屡战屡败 绝望至极。找尼恩转架构升级，1 个月拿到阿里 + 美团 offer，逆天改命年薪 50W](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486794&idx=1&sn=fc1a1f843b0a91b61f7124ac0c2b51e2&scene=21#wechat_redirect)

  

[阿里 offer：6 年一本 不想 混小厂了。狠卷 1 年  拿到 得物 + 阿里 offer ， 彻底上岸 ，逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486782&idx=1&sn=52ed05742c8afe7abf3b50405e78c853&scene=21#wechat_redirect)

## 大龄逆袭的案例： 大龄被裁，快速上岸的，远离没有 offer 的焦虑、恐慌 

[47 岁超级大龄，被裁员后 找尼恩辅导收  2 个 offer，一个 40 多 W。 35 岁之后，只要 技术好，还是有饭吃，关键是找对方向，找对路子](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486323&idx=1&sn=a088584f97b657add9a058c149b72067&scene=21#wechat_redirect)

[大龄不难：39 岁 / 15 年老码农，15 天时间 40W 上岸，管一个 team，不用去 铁人三项了！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486310&idx=1&sn=505e479da54cf1365afd4d39bddc18bd&scene=21#wechat_redirect)

## 草根逆袭， 100W 年薪 天花板 案例。 他们 如何 实现薪酬腾飞、人生逆袭？ 

[专科生 100 年薪 ：35 岁专科 草根逆袭，2 线城市年薪 100W 逆天改命， 从 超低起点 塔基（8W）--》塔腰 -》塔尖（100W）](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486860&idx=1&sn=940405ff775e014bac52ba0636ea69e7&scene=21#wechat_redirect)

# **[年薪 100W 的底层逻辑：](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486315&idx=1&sn=52cc0a640c1499115f5a45b82097d857&chksm=97b57ae3a0c2f3f502293e84cde4bc892c45d06dee3579471a522ab497fb53e32640fb00c339&scene=21#wechat_redirect)** [大厂被裁，他们两个，如何实现年薪百万？](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486315&idx=1&sn=52cc0a640c1499115f5a45b82097d857&chksm=97b57ae3a0c2f3f502293e84cde4bc892c45d06dee3579471a522ab497fb53e32640fb00c339&scene=21#wechat_redirect) 

**[年薪 100W](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)****[：](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)**[40](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)[岁小伙，被裁 6 个月，猛卷 3 月，100W 逆袭 ，秘诀：升级首席架构 / 总架构](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)

[最新的 100W 案例：环境太糟，如何升 P8 级，年入 100W？](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486369&idx=1&sn=0b32e8907dbcf26eaf33ea3f37ddd099&chksm=97b57a29a0c2f33f468d29dafe739204543873efb459f324871c3c68a3ecc178b1a9ab08a2fa&scene=21#wechat_redirect)

**职业救助站**

实现职业转型，极速上岸

![](https://mmbiz.qpic.cn/mmbiz_jpg/9qY2Gt3PBs3icE6WcM4nBOdRRcczOW4wkNDgCMZhBfqeet9suwGHfgQduP33YYqqZLOppecMI72Pv5ibNkPeIk6g/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=19)

关注**职业救助站**公众号，获取每天职业干货  
助您实现**职业转型、职业升级、极速上岸**  
---------------------------------

**技术自由圈**

实现架构转型，再无中年危机

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/xlgvgPaib7WMtyFcJx45oIeWL1d6AENcdoAvuTPuq46E9n8pPe4SHXoSicuEVvhibHRSIzmEBPlC7POcDaqD6ID7w/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=20)

关注**技术自由圈**公众号，获取每天技术千货  
一起成为牛逼的**未来超级架构师**

**几十篇架构笔记、5000 页面试宝典、20 个技术圣经  
请加尼恩个人微信 免费拿走**

**暗号，请在 公众号后台 发送消息：领电子书**

如有收获，请点击底部的 " 在看 "和" 赞 "，谢谢