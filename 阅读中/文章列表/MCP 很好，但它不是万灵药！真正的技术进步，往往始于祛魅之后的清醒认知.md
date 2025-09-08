---
source: https://mp.weixin.qq.com/s/Zs2yF1MovpBD_a12bIVibg
create: 2025-06-10 19:02
read: true
knowledge: true
tags:
  - 大模型
knowledge-date: 2025-09-07
summary: "[[阅读中/阅读总结/MCP 很好，但它不是万灵药！真正的技术进步，往往始于祛魅之后的清醒认知|MCP 很好，但它不是万灵药！真正的技术进步，往往始于祛魅之后的清醒认知]]"
---
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/j3gficicyOvasVeMDmWoZ2zyN8iaSc6XWYj79H3xfgvsqK9TDxOBlcUa6W0EE5KBdxacd2Ql6QBmuhBJKIUS4PSZQ/640?wx_fmt=gif&from=appmsg)

作者：boyang

人人都在聊 MCP，但人们口中的 MCP 往往只是一个拼凑而成的幻影。如今，各大厂商纷纷为它镀金包装，就像硅谷创投圈每隔几年就热炒一次的那个 “改变世界” 的万能工具。然而，当最初的狂热稍退，我们不得不面对更复杂的问题：MCP 真的适用于所有场景吗？它是否被赋予了过高的期待？技术史上从不缺少“神话”，而真正的进步，往往始于祛魅之后的清醒认知。

当下 AI 领域最炙手可热的概念，莫过于 MCP。MCP 指的是 Model Context Protocol（模型上下文协议）。令人意外的是，一个协议系统的热度，甚至盖过了 OpenAI 发布的最新模型，成为行业讨论的焦点。  

随着 Manus 的爆火，全球开发者对 Agent 技术的热情空前高涨。MCP 作为 Agent 工具调用的 “统一协议”，短短两个月内即获得了 OpenAI、Google 等主要 AI 公司的支持，从一个边缘技术规范一跃成为 AI 生态的底层标准。

它的崛起速度之快，堪称 AI 基础设施领域的 “现象级事件”。而开发者社区也涌现出各种 MCP 服务，仿佛它已是 AI 工具调用的 “终极答案”。

然而，当最初的狂热稍退，我们不得不面对更复杂的问题：MCP 真的适用于所有场景吗？它是否被赋予了过高的期待？

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasnhWPhxbgksvficRlCFMObdib4n6ic2uBYp4X9zVricX8WwGUSfI1B3M1JHoyVGxwmtBzm9xy6dibqBvg/640?wx_fmt=png&from=appmsg)

本文将从 MCP 的起源出发，剖析其核心价值与局限性，澄清常见误解，并探讨它的未来发展方向。我们的目的并非否定 MCP 的价值，而是希望回归理性——只有明确它的实际定位和适用边界，才能真正发挥它的潜力。毕竟，技术史上从不缺少 “神话”，而真正的进步，往往始于祛魅之后的清醒认知。

### 一、MCP 的本质：统一的工具调用协议

#### 什么是 MCP？

MCP 是一种开放的技术协议，旨在标准化大型语言模型（LLM）与外部工具和服务的交互方式。你可以把 MCP 理解成像是一个 AI 世界的通用翻译官，让 AI 模型能够与各种各样的外部工具 "对话"。

#### 为什么需要 MCP？

在 MCP 出现之前，**AI 工具调用面临两大痛点：**

第一是接口碎片化：每个 LLM 使用不同的指令格式，每个工具 API 也有独特的数据结构，开发者需要为每个组合编写定制化连接代码；第二是开发低效：这种 "一对一翻译" 模式成本高昂且难以扩展，就像为每个外国客户雇佣专属翻译。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j3gficicyOvasnhWPhxbgksvficRlCFMObdvWfGRsnjN4K6PZnL460fUfztxU3A4CaIyZVVfVspwcza1icT2vfSFLg/640?wx_fmt=other&from=appmsg)

而 MCP 则采用了一种通用语言格式（JSON - RPC），一次学习就能与所有支持这种协议的工具进行交流。一个通用翻译器，不管什么 LLM，用上它就能使用工具 / 数据了。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j3gficicyOvasnhWPhxbgksvficRlCFMObdVicJRZUEeomMBa4yV4IJLhpW9mWnGZUDKqhm6WZcF1cYP4zCAlsdeRA/640?wx_fmt=other&from=appmsg)

这就是 MCP 的全部功能。

#### MCP 的工作原理

MCP 的技术架构可以简单理解为一个由三个核心部分组成的系统：MCP Host、MCP Client 和 MCP Server。这三部分共同工作，让 AI 模型能够顺畅地与外部世界交流。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j3gficicyOvasnhWPhxbgksvficRlCFMObdDddw3OtiaNicbXZN5vbJEA0Kn1oibny6Yh5UtnSt9jSg79Bp02oicpOG2w/640?wx_fmt=other&from=appmsg)

要准确理解 MCP 的角色，我们可以将其比作现代企业环境中的一个通信系统。

在这个比喻中，用户扮演着企业高管的角色，负责理解用户需求并做出最终决策。，大模型（如 Claude 或 GPT）理解高管的指令，规划任务步骤，判断何时需要使用哪些外部服务，并最终整合信息为高管提供答案。Agent 系统则是真正的私人助理或执行秘书去执行，而 MCP 则像是秘书使用的标准化通信平台或企业服务接入系统，它不做决策，只负责按照秘书的指示，以统一的格式和协议与各种服务提供商交流。

在 MCP 出现之前，AI 与外部工具的交互就像是处在通信标准混乱的时代。每当秘书（Agent）需要联系不同部门或外部供应商时，都必须使用不同的通信设备或软件：给财务部打电话需要座机，联系 IT 部门需要用 Slack，预订会议室要用 Outlook，订餐则需要使用外卖 App。每个系统都有不同的操作界面、不同的数据格式和不同的通信协议，秘书必须熟悉所有这些不同的系统才能高效工作。对开发者而言，这意味着为每个工具单独编写连接代码，既费时又缺乏可扩展性。

MCP 的出现改变了这一局面。它就像是一个统一的企业通信平台，无论秘书需要联系哪个部门或服务商，都可以使用同一个系统，遵循同一套通信协议。

**MCP 的技术架构由三个核心组件构成：**

**MCP Host （执行环境） 就像是企业的办公环境和基础设施。**它提供了高管办公和秘书工作的场所，是一切活动的发生地。在实际应用中，Claude Desktop、Cursor 这类 AI 应用就是典型的 Host，它们提供了用户与 AI 交互的界面和环境，同时也为 Agent 和 MCP Client 提供了运行空间。

**MCP Client （通信枢纽） 像是秘书（Agent）使用的标准化供应商。**它不参与决策，不理解任务本质，只负责按照秘书的指示，以正确的格式和协议与各种服务提供商通信。MCP Client 是一个纯技术组件，处理通信协议、数据格式转换和连接管理等底层问题。

**MCP Server （服务终端） 就像是各个专业部门或外部服务提供商，每一个都负责特定类型的服务。**有的提供数据分析（如财务部），有的提供信息检索（如资料室），还有的提供内容生成（如市场部）。在 MCP 架构中，每个 Server 提供特定类型的功能：工具、资源或提示。

在 MCP 出现之前，当秘书需要完成高管的多项任务时，必须切换使用多种通信工具和系统，熟悉各自不同的操作方式。例如，预订会议室需要登录内部系统 A，获取财报需要使用系统 B，订餐则需要拨打餐厅电话。开发者则需要为每个工具单独编写连接代码，效率低下且难以维护。

在 MCP 之后，秘书只需使用一个统一的通信平台，就能以相同的方式联系所有部门和服务提供商。开发者也只需实现一次 MCP 接口，就能让 AI 系统与所有支持该协议的工具进行交互。

#### MCP 不是 Function Call 的替代，而是基于 Function Call 的工具箱

很多人认为，MCP 是对传统 Function Call 的一种替代。

**而实际上，两者并非替代关系，而是紧密合作的关系。**

Function Call 是大型语言模型（LLM）与外部工具或 API 交互的核心机制。它是大模型的一个基础能力，就是识别什么时候要工具，可能需要啥类型的工具的能力。

**而 MCP 则是工具分类的箱子。**因此 MCP 不是要取代 Function Call，而是在 Function Call 基础上，联合 Agent 一起去完成复杂任务。

如果把整个工具调用的流程剖析开来，实际是 "Function Call + Agent + MCP 系统" 的组合。

**用一句话说清楚：**大模型通过 FunctionCall 表达，我要调用什么工具，Agent 遵循指令执行工具的调用，而 MCP 则是提供了一种统一的工具调用规范。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j3gficicyOvasnhWPhxbgksvficRlCFMObdpVfTQod0QSYp3wX2PpnIxfyXYu08BHy007HnNgxlanhUJZBjluaDNA/640?wx_fmt=other&from=appmsg)

**用一个比喻来理解：**老板（用户）要喝咖啡，于是，在办公室（MCP Host）里，办公室主任（大模型）吩咐秘书（Agent）去买一杯美式（Function Call）。秘书（Agent）查了一下供应商名册，发现美式咖啡的供应商已接入了美团或公司统一采购系统（实现了 MCP Server），接着，秘书在采购系统中找到供应商（MCP Client）一键下单。

在过去没有 MCP 时，大模型下发 Function Call，Agent 去执行翻译，直接连接到 API 去调用工具。因此，你得为每个 API 单独设置对应的调用模式，去单独定义工具列表和调用模式，这样 Agent 才知道怎么去翻译。而有了 MCP 后，只是很多 API 都可以直接通过供应商 MCP Client 一键下单了，Agent 省力了。但大模型的 Function Call 没有任何变化。还是 {tool: “买加啡”, "type": "美式"} 这个形式。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j3gficicyOvasnhWPhxbgksvficRlCFMObdnm249Rnhu1HCOykSpDw6lo0YefZdhDRJkAdzH1y5PUJvFPb5Gk1MJA/640?wx_fmt=other&from=appmsg)

不过在过去，有人会把这一整套 Function Call+ Agent +API 的模式叫做一个 Function Calling，所以会引起混淆。

通过区分 Function Call 和 MCP，我们可以清晰地看出，MCP 并不负责决定使用哪个工具，也不进行任务规划或理解用户意图。这些是 Agent 层面的工作。MCP 只是提供了一个统一的工具接口，成为了产业内认可的工具调用标准协议。

### 二、MCP 的开发难题与市场乱局

#### **一宗罪：开发的难题**

今年 2 月以来，AI 开发社区掀起了一场 "MCP 淘金热"。在没有官方应用商店的情况下，短短三个月就有数千个工具自发接入 MCP 协议。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j3gficicyOvasnhWPhxbgksvficRlCFMObdFYFB53t8h0NeibR3ibX0ASqVtY99jYga0hZNZMNdJ6mCEJSDlzoI9ZWA/640?wx_fmt=other&from=appmsg)

这仅仅是一家 MCP Server 集成网站的量就超过一万种。这种野蛮生长让 MCP 迅速成为行业焦点，但也暴露了理想与现实之间的鸿沟——开发者们最初将 MCP 视为 "万能钥匙"，实际使用后却发现它更像是 "专业扳手"，在某些场景下表现出色，在其他场合却显得水土不服。

在 MCP 的所有参与者中，可以分为本地客户端应用、云端客户端应用和 MCP 服务端开发者。本地应用就是类似本地的 AI 助手，云端客户端则是类似于 ChatGPT 的网页版，而 MCP 服务器开发者，则是工具的真正供给方。他们需要把自由的 API 重新做一遍符合 MCP 规则的包装盒接入。在 MCP 刚刚上线的时间内，最开心的是本地客户端应用，而云端客户端应用和 MCP 服务器开发者就不太舒服了。要理解这一点，得回到 MCP 的缘起。

MCP 的起源始于 Anthropic 的 Claude Desktop 应用，它最初是为了调用本地文件和功能而设计的接口协议，深深植根于客户端的需求土壤中。

**因此对于本地客户端用户来说，MCP 是一场革命，它提供了无限扩充的工具箱，允许用户不断扩展 AI 助手的能力边界。**本文作者 Kongjie 认为，"对于桌面端 Agent 来讲，这个 MCP 用起来其实会很爽很顺，因为是为它量身定做的工具。"

所以本地客户端应用如 Cursor 和 Claude Desktop 在使用 MCP 方面大放异彩，它们利用 MCP 让用户能够根据个人需求动态添加工具，实现了 AI 助手能力的无限扩展。

它解决了本地客户端开发中的一个核心痛点：如何让 AI 应用能够与本地环境和外部工具无缝交互，而无需为每个工具单独开发接口。这种统一的协议极大地降低了整合成本，特别是对于小型创业公司和个人开发者，MCP 提供了在有限资源条件下构建功能丰富 AI 应用的捷径。

然而，当我们将目光转向服务端开发（MCP Server）和云端客户端时，MCP 的光芒开始黯淡。MCP 早期对于云端服务器（remote）采用了一种双链接机制，一条是 SSE 的长链接，单向的从服务端到客户端的推送消息用，还有另一条发消息的 http 常规请求短链接。

这种方法对用户及时反馈和中途干预来讲效果很好，**这种需要服务器处理的环境中创造了一系列工程挑战。**

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j3gficicyOvasnhWPhxbgksvficRlCFMObdgPdT0ROSve0oSZUibniasTrkibJBFYSKiaUrCKFR0wD85Z0ZUf9YMtlZpQ/640?wx_fmt=other&from=appmsg)

MCP Host 和 MCP Server 信息沟通是双向的。首先，对于大型企业服务提供商，实现 MCP 接口意味着额外的工作负担，而这些工作可能并不带来相应的回报。由于这些服务通常已经有成熟的 API 体系，再额外提供 MCP 适配层往往只是增加了维护成本，而非创造实质性价值。事实上，许多企业级应用更倾向于使用封闭、可控的工具调用机制，而非 MCP 倡导的开放式生态系统。

而且对于这些大型企业服务供应商来讲，为了应对高并发调用，MCP 服务往往需要扩展到多服务器架构。此时，MCP 的双连接模型引入了跨机器寻址的复杂性。当长连接建立在一台服务器上，而请求可能被路由到另一台服务器时，就需要额外的广播队列机制来协调这些分散的连接，大大增加了实施难度和维护成本。

其次，在云端应用领域，MCP 也有很大的局限性。云端 AI Agent（服务端 Agent）通常运行在无状态的服务中，任务接受之后它进行处理，结束后便释放资源。在服务端使用 MCP Client 时需要临时创建一个 SSE 链接，发送工具调用请求，从 SSE 获得结果再关闭掉 SSE 链接，实在是属于一种很鸡肋的做法，复杂度增加了，性能也下降了，而这种场景下，理应只需要一个单次的 RPC 请求就解决问题。

**实际上云端应用在用 MCP 时，大多还是用的预设的工具集，而没有用到 MCP 的招牌能力——动态工具发现和灵活加载功能。**

Kongjie 认为，"云端本身的数据交互模式，导致虽然 MCP 希望能达成工具的自由使用，但实际上用不上。还是必须用非常规范的流程去调用特定的写死的工具，丧失了灵活性的初衷。"

值得称赞的是，MCP 团队对用户反馈保持开放态度。在收到服务端开发者的反馈后，MCP 在 3 月 26 号更新了协议，使用 streamable HTTP 的 transport 替代了原来的 SSE transport。这样一来，新的协议既支持仅需要单次调用工具请求的无状态服务场景，也兼容了原来通过 http + SSE 双链接才能满足的实时推送的诉求。

这说明，当下的一些 MCP 的问题，源于其最初目的的限制，但并非不可解。

#### **二宗罪：市场的乱局**

另一个 MCP 面对的问题可能就比较普遍了：现在市面上的 MCP 可用性太低。

当前 MCP 市场正经历着典型的技术炒作周期。就像当年 App Store 刚上线时的混乱景象，如今数千个 MCP 工具中，真正具备实用价值的不足二成。腾讯开发者 victor 的观察一针见血：“很多 MCP，实际上都没有真的做 MCP。”

victor 表示，他试用了 300 多个 MCP 项目，但**约有 80% 的 MCP 服务器存在严重问题**，从简单的配置错误到完全无法使用的情况不等。这些工具有的是因急于上线而未经充分测试，有的则是开发者实验性质的产物，从未打算投入实际使用。

**而更严重的问题是，其中大部分 MCP 可能市场根本就不需要**。如 Kong 所言："真正有用的，能够继续解决问题的那些工具本来在当它还是一个 API 的时候，别人也去用了。然后现在大家发现 MCP 有市场，就做了一个简单封装，但它本来没有什么特别的用。"

比如说现在 MCP 提供的搜索服务可能就有数十家之多。而在《Evaluation Report on MCP Servers》论文中，作者对此类服务做了一些评测，发现水平差异非常大。其中像 Exa Search 又不准，时间又差。如果有最好的 Bing Web Search，谁会用它呢？

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j3gficicyOvasnhWPhxbgksvficRlCFMObdCSzO7aqRptOia9AxBfsxS1KWjMbWJG1oSR5kkAYeCgNK88yHsV4VQcQ/640?wx_fmt=other&from=appmsg)

**而且，现在 MCP 也缺乏评价体系**。这导致 Agent 无法依靠可靠的指标来选择最适合的工具，只能通过反复尝试或基于模糊的描述做出决策。这种低效的选择过程不仅浪费计算资源，还延长了任务完成时间，降低了用户体验。

据 Kong 表示，“如果选了多个 MCP 服务，其中重名，描述相似的工具太多，Agent 不知道选哪个，也没有排名。只能一个个试，很浪费 token，也效率很低。”

**因此，那些最成功的 AI 应用往往采取了相反的路径**——它们不是提供更多的工具，而是提供更精准的工具。比如 Manus，在 MCP 已经存在的情况下，其实根本没有采用这一协议，而是选择内建应用。因为它只有几十种工具，为了提高调用准确度和稳定性，不如自己去单独写。

而 Cursor 这样的代码编辑器已经内置了最常用的开发功能，使得大多数外部 MCP 工具显得多余。在作者使用 Cursor 时，虽然开启了几十个 MCP，但真正能用到的凤毛麟角。

**但 MCP 市场当前的混乱状态并非失败的标志，而是任何新兴技术生态系统必经的成长阶段。**

从历史经验来看，这种初期的过度膨胀往往会通过市场自然选择机制逐渐收敛，留下真正有价值的部分。就像互联网泡沫之后留下了改变世界的电子商务和社交媒体一样，MCP 热潮过后可能会形成一个更加精简但更有实质价值的工具生态系统。

这个过程中，至少 Anthropic 团队也能从善如流，像 victor 建议的那样 “在协议规范上，要求接入协议的服务能保证参数和工具的质量。” 这样才能让 MCP 的生态变得真正“可用”。

### **三、MCP 很好，但它不是万灵药**

以上两种 MCP 体验的批评，确实来自于 MCP 自身的限制和短板。也是其能力可完成的范围。但在当前，还有一类对 MCP 的批评，明显是由于人们对它赋予了不太恰当的期待。

在近期一篇大火的 MCP 批判文章中，**作者认为，MCP 就是一个残缺的协议，因为它没有规定大语言模型和 MCP 的交互模式。**

很多人期待 MCP 能够一劳永逸的自动改善 AI 系统的决策能力，或者提升任务规划的效率。但这种诉求实际上是在错误地将工具与工匠混为一谈。

因此，**当前对 MCP 的批评存在一个根本性的认知错位——人们期待一个通信协议能完成智能系统的全部工作**。这就像指责 USB 接口不能自动修图，或是埋怨 5G 标准不会编写代码。MCP 本质上只是一套 "工具插座" 的标准规范，它的使命是确保插头能顺利插入，而非决定用哪个电器、怎么使用电器。

Agent 调用工具的效果受到诸多因素制约——工具选择能力、任务规划能力、上下文理解能力——而这些能力的培养与提升，都不在 MCP 的职责范围内。**MCP 只承诺提供统一的工具接口，而不承诺这些工具将被如何选择和组合。**

在技术演进的长河中，我们总是倾向于寻找一劳永逸的解决方案，一种可以解决所有问题的银弹。但如同软件工程中著名的 "没有银弹" 论断，AI 工具调用领域同样不存在万能的解决方案。一个真正强大的 AI 系统需要多个精心设计的组件协同工作：大语言模型提供基础理解和生成能力，Agent 框架负责任务规划和执行逻辑，而 MCP 则专注于提供统一的工具调用接口。

它展示了良好的协议设计哲学——关注一个问题并解决好它，而非尝试解决所有问题。正是这种克制，使得 MCP 能够在客户端工具集成领域取得实质性进展。

因此，不管是阿里的 Qwen，还是百度的 “心响”，字节的扣子空间，都拥抱了 MCP 协议，也试图在内部建立一个更高效的 MCP 生态空间。

虽然都有部署，但他们还是有比较明确的区别的，并非纯粹 “拿来主义”。

百度试图从 C 端切入， “心响”（Kokone）利用 MCP 协议整合多种 AI 模型和外部工具，主要面向手机端，目前仍只支持安卓系统，意图将自身产品打入用户的日常场景体验之中，培养用户习惯。

而字节跳动的扣子空间集成超过 60 款 MCP 扩展插件，主要入口为网页端，还推出了支持 MCP 的 AI 原生 IDE——Trae，主要瞄准工作场景和生产力场景。

阿里在支付宝等产品中集成了 MCP 协议，实现了 AI 工具的一键调用，4 月 29 日开源 Qwen3 系列模型也支持 MCP 协议，增强其 Agent 能力。

腾讯云开发者发布了 AI 开发套件，支持 MCP 插件托管服务；腾讯云大模型知识引擎支持用户调用 MCP 插件；腾讯云代码助手 CodeBuddy 推出的 Craft 软件开发智能体，可兼容 MCP 开放生态。另外，腾讯地图、腾讯云存储也发布了自己的 MCP SERVER。

但正如编程范式从汇编语言进化到面向对象一样，AI 工具使用也可能从直接操作单一工具进化到与专业化 Agent 的协作。

在这种新范式下，MCP 可能只是底层基础设施的一部分，而不是用户或开发者直接面对的界面。更全面的方案可能需要结合像 Agent to Agents（A2A）这样的架构，通过抽象层次提高任务规划和工具选择的效率。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j3gficicyOvasnhWPhxbgksvficRlCFMObdJDAdoI9ibpiag4JIktDL6Y8pLC6tpibYBpY2ZYzq0qYkrtRL5YdJ9j6CQ/640?wx_fmt=other&from=appmsg)

**当我们将 MCP 放回 "协议" 的本位，反而能看清其推动行业标准化的真实力量——这或许才是技术演进中最珍贵的 "祛魅时刻"。**

**如果你想深入了解 AI，欢迎扫码加入 ima 知识库「AI 能量站」知识科普基础论文及解读。本知识库收录顶刊、权威会议论文、头部机构技术报告、高校出版社书籍等 A 方面的知识科普，以及行业权威专家解读，全面解锁 AI 的奥秘。**

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j3gficicyOvasnhWPhxbgksvficRlCFMObdG8WmQguMrlHWLhuJ2VJicqg2DhH9oQEDPjFVEaW6Vmdbnh2q2GUib6Fw/640?wx_fmt=other&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/j3gficicyOvasVeMDmWoZ2zyN8iaSc6XWYjZ7Hx6Udjjk2BGLzC9ahJq7ibxDd1RGA0c9NYZc1husEsvb3tY4FcWPQ/640?wx_fmt=gif&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasVeMDmWoZ2zyN8iaSc6XWYj5q5PQEOc5ibURPb03vnRibrxC3UR8xzdyATfiawTYRV2vJvBnAIcE1FeQ/640?wx_fmt=png&from=appmsg)