---
source: https://mp.weixin.qq.com/s/EcDCKN4-movoU2JgIqZSXg
create: 2025-10-16 20:55
read: false
knowledge: false
---
![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naKic9HtALLuaAa0Mbuegx8V0kMrGjwLamHgLUllIBdc5RuJAqUHIL58cspempH4zFIjyRyKJn1vZQA/640?wx_fmt=jpeg&from=appmsg#imgIndex=0)

前言：直面误解，回归工程

虽然之前对于 MCP 有过一次调研，但是最近上手在做一些 MCP 的工程实践的过程中，确实发现还是有很多误解。我翻阅了许多技术文章，以及协同沟通的时候发现，往往大家天然的将 MCP 简单地视为一种 “更高级” 或“可跨模型”的 Function Calling。这种认知偏差，不仅掩盖了 MCP 作为一套软件工程协议的真正价值，更可能导致在技术选型和系统架构设计上的严重误判。

当一个集成了 20 个 OpenAPI 工具的 MCP 应用，其单次请求的 Token 消耗轻易突破 60K，导致成本和延迟飙升时；当生产环境中的模型因为一个微小的 Prompt 变动而出现大面积的工具调用格式错误时，我们必须扪心自问：我们是否真的理解了 MCP 的本质？

于是提笔，希望将本文作为一篇写给 AI 工程师的硬核 “辟谣” 指南，旨在彻底澄清这一混淆。并提出并严谨论证一个核心观点：MCP 本质上是一套模型无关的、用于构建可互操作 AI 应用的工程协议，其核心组件 Server/Client 与 LLM 的智能决策并无直接关联。

我们将以架构师的视角，遵循 “假设 - 验证” 的逻辑链条，通过以下路径展开论证：

1. 架构分析：从官方文档出发，将普遍的 “CS 架构” 误解纠正为 “CHS 三组件” 的本质，并精确界定各组件的工程职责。

2. SDK 检验：深入 MCP 官方 SDK 的 Server 与 Client 实现，用代码事实证明其作为纯粹 RPC 中间件的模型无关性。

3. Host 解剖：以开源项目 CherryStudio 为例，精准定位 Host 组件中构建 Prompt 和调用 LLM 的 “犯罪现场”，找到 AI 能力的真正归属。

4. 重回概念：重新回到概念，清晰切割 MCP（基础设施协议）与 Function Calling（模型决策能力）的边界，并通过伪代码比较两者的流行实现。

最终，回归工程的第一性原理，围绕 “效果和成本”，重新审视 MCP 在 AI 应用开发中的真实角色。

第一章：剖析 MCP 核心架构：从 CS 误解到 CHS 本质

**1.1 普遍误解：为何 MCP 架构常被错认为 CS？**

对于任何初次接触 Model Context Protocol（MCP）的工程师来说，一个普遍的认知陷阱源于其官方文档中的核心架构图。这张图直观上呈现了一个客户端与服务端进行交互的经典形态，极易让人先入为主地将其归类为传统的 Client-Server（CS）架构。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naLTaj6dickraLHIwn2sibGBKwofFCZccj6yCahibgM11uJbxfnEACXCzibNVXoXDaO7wJibwqWrqPp2FCg/640?wx_fmt=png&from=appmsg#imgIndex=1)

参考 1《MCP 架构》

然而，这种基于视觉相似性的判断，是导致后续一系列概念混淆的根源。它不仅是一个微不足道的命名偏差，更从根本上模糊了 MCP 设计的核心思想。官方规范开宗明义地指出：

"The Model Context Protocol (MCP) follows a client-host-server architecture."

这一定义明确地将 Host 引入，构成了理解 MCP 的关键。将一个三组件（Client-Host-Server, CHS）系统误读为两组件（CS）系统，会直接导致对协议职责、数据流向和 AI 能力归属的严重误判，为后续的系统设计与问题排查埋下隐患。

**1.2 CHS 三组件职责详解：Host、 Client 与 Server**

要彻底破除 CS 误解，就必须用精确的工程语言，对 CHS 三组件的职责边界进行严格界定。它们在 MCP 生态中扮演着截然不同且相互解耦的角色。

*   Host：AI 智能的唯一承载者
    Host 是整个系统中唯一与大型语言模型（LLM）直接交互的组件。其核心任务包括：管理完整的对话上下文、动态构建和拼接 Prompt、解析 LLM 的响应、根据 AI 决策生成与 MCP Server 交互的指令。简而言之，一个 MCP 应用的 “智能水平” 完全由其 Host 的实现质量决定。
    

*   Server：确定性能力的执行器
    Server 是一个标准的网络服务，它向外界声明并提供一组具有确定性的、可供远程调用的能力（Capabilities）。这些能力可以是工具调用、文件操作等（全量枚举源码里有）。Server 接收标准化的请求，执行相应的能力，并返回确定的结果。它不包含任何 AI 逻辑，其行为是可预测且可靠的。值得注意的是，工具调用只是其众多规划能力中的一种，尽管是目前最常用的一种。
    

*   Client：无状态的协议中间件
      Client 的角色最为纯粹，它是一个位于 Host 和 Server 之间的协议客户端和状态管理器。它严格实现了 MCP 的通信协议，负责处理协议握手、会话管理、心跳维持以及将 Host 的意图（如 “调用某个工具”）转换为符合 MCP 规范的 JSON-RPC 请求并发送给 Server。它不关心业务逻辑，也不理解 AI 意图，仅作为连接 Host 与 Server 的标准化通信管道。
    

通过上述定义，我们可以清晰地看到，将 Host 与 Client 混为一谈是问题的核心。社区中流传的许多图表之所以引起误解，正是因为它们将承载 AI 逻辑的 Host 与负责协议通信的 Client 绘制成了一个模糊的整体，掩盖了 MCP 架构设计的精髓。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naLTaj6dickraLHIwn2sibGBKwdnaaXht1sh1DQYHGX3uRI9bWPjic4gKkV8GTwIjlicZVYKhcjiceNYvaQ/640?wx_fmt=png&from=appmsg#imgIndex=2)

图中容易引发 Client 发起了大模型调用的误会

引用来自：https://blog.dailydoseofds.com/p/function-calling-and-mcp-for-llms

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naLTaj6dickraLHIwn2sibGBKwZcSaqdqN06FsXPVlg2viaNhvjX1mJA7ia3WibfbFA3CEuq44y4sSV0MRA/640?wx_fmt=png&from=appmsg#imgIndex=3)

图中容易引发 Client 发起了大模型调用的误会

https://modelcontextprotocol.io/specification/2025-06-18/server/tools

第二章：深入 SDK 源码：Server 与 Client 的模型无关性证明

**2.1 分析对象：MCP 官方 NodeJS SDK**

我们现在将从架构理论转向对源代码的法证式检验。本章的分析对象是 MCP 官方的 NodeJS 实现：@modelcontextprotocol/sdk。我们的目标非常明确：通过剖析其核心组件 Server 和 Client 的实现，从代码层面提供无可辩驳的证据，证明它们是纯粹的、与模型无关的工程中间件。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naLTaj6dickraLHIwn2sibGBKwiaOGBqMzWWXnD6z5J2yLXlLrwbfUb0SNq922KG2znicDPyDibnmezvib4Q/640?wx_fmt=png&from=appmsg#imgIndex=4)

《MCP SDK 的调用时序图》，仅展示 tool 部分，参考 1

**2.2 Server 生命周期剖析：从实例化到协议握手**

一个 MCP Server 的生命周期，从实例化到处理第一个请求，其每一步都清晰地暴露了它的非 AI 本质。整个过程是一个标准的网络服务启动与协议协商流程，不涉及任何 LLM 逻辑。

步骤一：实例化阶段：声明能力，注册内部处理器在服务器代码中执行 new Server(...) 时，Server 的构造函数立即执行了两个关键的、与 AI 无关的设置：

*   能力声明：它记录下传入的 capabilities 对象（例如 {tools: {} }），即在服务启动前就确定了 “我能提供什么服务”。
    

*   处理器注册：它立即为内部方法 "initialize" 注册了一个请求句柄（RequestHandler）。
    

```ts
// @modelcontextprotocol/sdk/dist/esm/server/index.js 第32-40行
constructor(_serverInfo, options) {
  super(options); 
  this._serverInfo = _serverInfo;
  this._capabilities = options?.capabilities || {}; // 传入能力"initialize","tools/list","tools/call"...
  this._instructions = options?.instructions;
  this.setRequestHandler(InitializeRequestSchema, (request) => this._oninitialize(request));
  this.setNotificationHandler(...);
}
//DEMO
new Server(
  { name: 'MiaodaoServer', version: '1.0.0' },
  { capabilities: { tools: {} } }
)
```

步骤二：连接与握手阶段：纯粹的协议交互当客户端连接并发送第一个请求（必然是 {"method": "initialize", ...}）时，Server 的执行流是一个典型的 RPC 过程：

*   传输层 onmessage 事件被触发。
    
*   _onrequest 方法根据请求中的 method 字段（"initialize"）查找到在构造函数中预注册的 _oninitialize 处理器。
    
*   执行_oninitialize。此函数的逻辑是一个纯粹的协议握手：
    

```ts
// @modelcontextprotocol/sdk/dist/esm/server/index.js 第138-150行
async _oninitialize(request) {
  const requestedVersion = request.params.protocolVersion;
  this._clientCapabilities = request.params.capabilities;
  this._clientVersion = request.params.clientInfo;
  
  return {
    protocolVersion: SUPPORTED_PROTOCOL_VERSIONS.includes(requestedVersion)
      ? requestedVersion
      : LATEST_PROTOCOL_VERSION,
    capabilities: this.getCapabilities(),// 返回能力清单:"initialize","tools/list","tools/call"...
    serverInfo: this._serverInfo,
    ...(this._instructions && { instructions: this._instructions }),
  };
}
```

在这个至关重要的初始化阶段，Server 只关心协议版本和能力清单的元数据交换。它完全不感知、不处理任何自然语言或 AI 意图。

**2.3 请求处理流程：一个确定性的 JSON-RPC 分发器**

初始化握手完成后，Server 便进入等待服务调用的状态。其处理后续请求（如 tools/call）的机制，进一步印证了它作为一个非智能分发器的本质。

决定性证据在于每一个 MCP 服务器对于 setRequestHandler 的实现。开发者通过此方法，将一个请求类型（由其方法名和参数结构）与一个确定性的本地函数绑定。

```ts
// @modelcontextprotocol/sdk/dist/esm/shared/protocol.js 第325-329行
setRequestHandler(requestSchema, handler) {
  const method = requestSchema.shape.method.value;
  // 5. 调用能力验证函数
  this.assertRequestHandlerCapability(method);
  this._requestHandlers.set(method, (request, extra) => {
    return Promise.resolve(handler(requestSchema.parse(request), extra));
  });
}

// @modelcontextprotocol/sdk/dist/esm/types.js 第816-822行
exportconst CallToolRequestSchema = RequestSchema.extend({
    method: z.literal("tools/call"),
    params: BaseRequestParamsSchema.extend({
        name: z.string(),                              // 工具名称
        arguments: z.optional(z.record(z.unknown())),  // 工具参数
    }),
});

// 用户实例化一个Server后绑定一个tools/call
server.setRequestHandler(
  CallToolRequestSchema, // 包含请求方法名和参数结构的请求类型
  async (request) => {
    return handleToolCall(request.params.name, request.params.arguments);
  }
);
```

这段代码的含义是：凡是符合 CallToolRequestSchema 规范的请求，一律交由开发者预先编写好的 handleToolCall 函数来处理。

当一个 tools/call 请求到达时，Server 内部的_onrequest 方法执行一个机械的 “查表 - 分发” 操作：

1. 从请求 JSON 中提取 method 字段，即 "tools/call"。

2. 在一个内部哈希表 (this._requestHandlers) 中查找与该 method 关联的处理器，即上面注册的 async (request) => { ... } 匿名函数。

3. 执行该函数，并将 request 作为参数传入。

在整个 “接收 - 校验 - 分发 - 执行” 的循环中，Server 不进行任何推理或决策。它只是一个忠实的、类型安全的 RPC 路由器。它不知道 tools/call 的业务含义，更不知道这个调用指令是由 LLM 在远端的 Host 中生成的。它的世界里只有结构化的 JSON、方法名和预先注册的函数指针。

**2.4 结论：Server 与 Client 作为纯粹的工程中间件**

基于对 SDK 生命周期和请求处理流程的法证式分析，我们可以得出结论：

*   MCP Server 是一个实现了 MCP 协议的、模型无关的 RPC 服务端。它的核心职责是：在启动时声明能力、通过 initialize 握手交换元数据、并将符合规范的 JSON-RPC 请求分发给预定义的确定性函数执行。
    
*   MCP Client（基于协议对称性）扮演着完全对等的角色：一个 RPC 客户端。它的职责是将 Host 的指令封装成标准化的 JSON-RPC 请求，发送给 Server，并接收响应。
    

它们共同构成了 MCP 协议的通信层。它们是连接 AI 智能（Host）与确定性能力（Server Tools）之间的标准化管道，而不是管道中流淌的 “智能” 本身。误以为它们与 LLM 有直接关联，就如同误以为 HTTP 协议本身能够理解网页内容一样，是一种根本性的概念混淆。

第三章：Host 实战解剖：在 CherryStudio 中定位 LLM 交互

在理论和源码分析之后，本章我们将进入实战，通过解剖一个非常流行的开源项目来定位 LLM 交互的精确位置。这个案例将最终证明：在 MCP 体系中，所有与 AI 智能相关的复杂工作，均由 Host 组件承担。

**3.1 案例背景：CherryStudio 中的 Host 拆分实现**

我们的分析对象是 CherryStudio，一个基于 Electron 构建的 AI 桌面应用。由于 Electron 特殊的双进程架构（主进程 + 渲染进程），CherryStudio 对 MCP Host 的实现进行了一次职责拆分，观察起来变得更加清晰：

*   主进程：运行在 Node.js 环境中，负责系统级操作和网络通信。其中 MCPService.ts 封装了 Client 能力和一部分 Host 调用职责，主要作为与外部 MCP Server 通信的代理。
    
*   渲染进程：运行在内置浏览器环境中，负责 UI 渲染和用户交互。其中 ApiService.ts 承担了 Host 的核心 AI 职责，包括 Prompt 构建和 LLM 调用。
    

这种拆分清晰地隔离了协议通信与 AI 逻辑，让我们能更精确地追踪数据流和决策过程。

**3.2 主进程：作为 MCP 通信代理的 MCPService.ts**

首先分析位于主进程的 src/main/services/MCPService.ts 我们发现它的职责高度聚焦于协议通信管理：

连接管理：通过 initClient 方法，负责创建和管理与外部 MCP Server 的 Client 连接实例。

请求转发：它暴露了如 listTools 和 callTool 等方法，但其内部实现仅仅是将来自渲染进程的 IPC 请求，转换为对 @modelcontextprotocol/sdk 中 Client 实例的方法调用。

```ts
// src/main/services/MCPService.ts  第562-583行
// 举例：listTools实现
private async listToolsImpl(server: MCPServer): Promise<MCPTool[]> {
  const client = await this.initClient(server)
  try {
    const { tools } = await client.listTools() // 创建Client调用listTools
    const serverTools: MCPTool[] = []
    tools.map((tool: any) => {
      const serverTool: MCPTool = {..格式转换...}
      serverTools.push(serverTool)
    })
    return serverTools
  }
}
```

显而易见，MCPService.ts 并没有与任何 LLM 直接交互。它扮演的是一个无状态的协议代理角色，是 Host 伸向主进程、用以操作 MCP Client 的手臂，而非 Host 的大脑。

**3.3 渲染进程：构建 Prompt 并调用 LLM 的 ApiService.ts**

本章的核心发现位于渲染进程的 src/renderer/src/services/ApiService.ts。 这里是所有 AI 智能的汇聚点，是 Host 大脑的真正所在。当用户提交一条消息后，ApiService.ts 中的 fetchChatCompletion 函数 编排了一个复杂的 AI 工作流：

1. 工具发现：通过 IPC 调用主进程的 MCPService.ts，获取当前可用 MCP Server 提供的工具列表（window.api.mcp.listTools()）。

2. 系统提示词构建：调用 buildSystemPromptWithTools 函数，将获取到的工具定义（包括名称、描述、参数 Schema）格式化后，注入到一个庞大的 System Prompt 中。这个 Prompt 指导 LLM 如何理解和使用这些工具。

3. 大模型调用：这是决定性的证据。代码最终调用了 AI.completionsForTrace，这是一个封装了对底层 AI Provider（如 OpenAI、Anthropic）API 请求的函数。

```ts
// src/renderer/src/services/ApiService.ts (fetchChatCompletion 函数核心逻辑)

// 发现MCP 服务器中提供的Tools
const mcpTools = await window.api.mcp.listTools(activeMcpServer);

// 把Tools的函数名、用途、参数解析一起构建一个系统提示词
const systemPrompt = buildSystemPromptWithTools(mcpTools);
const messages = [ { role: 'system', content: systemPrompt }, ...conversation ];

// 异步调用大语言模型
const completionsParams: CompletionsParams = {
  messages: messages,
  // ... other parameters
};
await AI.completionsForTrace(completionsParams, requestOptions);
```

代码明确显示，ApiService.ts 负责了将 MCP 工具信息 “翻译” 成 LLM 能理解的 Prompt，并将用户的请求连同这份 “工具使用说明书”（即 Prompt）一起发送给 LLM 进行决策。LLM 返回的工具调用意图，再由该服务进一步处理，并通过 IPC 调用 MCPService.ts 来实际执行。

**3.4 最终定位：Host 是承载 AI 智能的唯一组件**

通过对 CherryStudio 的实战解剖，我们得出了与前两章完全一致的结论：

*   MCP Server 提供了确定性的能力。
    
*   MCP Client 负责标准化的通信。
    
*   MCP Host（在 CherryStudio 中由 ApiService.ts 和 MCPService.ts 共同实现）承担了全部的 AI 智能工作：它发现工具，它构建 Prompt，它调用 LLM，它解析结果。
    

一个 MCP 应用的最终效果——无论是工具调用的准确性，还是对话的流畅性——本质上取决于其 Host 的实现水平，尤其是 Prompt 工程的质量和对 LLM 能力的驾驭。MCP 协议本身只提供了一个稳固的工程舞台，而 Host 才是舞台上真正的 “主角”。

第四章：概念辨析：MCP 与 Function Calling 的层级与关系

通过前三章的分析，我们已经从工程角度证明了 MCP 的核心是模型无关的协议。现在，我们需要在概念层面，将其与另一个广受欢迎的技术 Function Calling 进行清晰的切割。作为同样可以通过大模型调用外部工具的方法，因为 MCP 出现时间更晚且支持范围更广，将 MCP 视为 “更高级的 Function Calling” 是一个更大的误解，这是一种将不同层级概念混为一谈的错误认知。

**4.1 角色定位：基础设施协议 vs. 模型决策能力**

要理解二者的关系，首先必须明确它们在 AI 应用工作流中所处的不同层级和扮演的根本角色。

*   Function Calling：是一种内嵌于 LLM 中的模型决策能力。它解决了 “决定做什么” 的问题。当模型具备 Function Calling 能力时，它能根据对话上下文，自行决定是否需要调用外部工具，并以结构化的格式（如 JSON）返回调用意图（函数名和参数）。这是 LLM 自身推理能力的延伸。
    
*   MCP：是一套定义 “如何调用” 的基础设施协议。它解决了工具的标准化、发现、安全和互操作性问题。MCP 将外部工具抽象为标准化的、可通过网络访问的服务。它就像软件工程中的 REST API 或 gRPC，为 AI 应用提供了一个统一、解耦的工具调用框架。
    

因此，Function Calling 是 AI 的 “大脑”，而 MCP 是连接大脑与外部世界的 “神经系统”。它们并非替代关系，而是协作关系，共同构成一个完整的 “感知 - 决策 - 行动” 循环。

**4.2 工程实现对比：从伪代码看解耦优势**

通过并列对比两种实现方式的伪代码，我们可以直观地看到 MCP 带来的工程价值。假设我们的任务是实现一个查询云服务 ECS 规格的工具。

还是 Python 的语法写起来舒服，各位看官切换一下语言，反正是伪代码随意一点。

场景一：传统的 Function Calling 的 Python 伪代码实现（Function Calling 意图识别 + 自行实现函数调用）

```ts
## Step1.调用大模型意图识别
messages = [{"content": "用户问句，帮我查一下杭州4C8G的ECS规格？", "role": "user"}]
assistant_output = call_llm(messages, tools=TOOLS) #注意：这里直接传递了数据结构，不再需要文本拼接

## Step2.读取意图识别结果，并且本地调用工具
if assistant_output['tool_calls']['function']['name'] == 'describe_instance_types':
    tool_info = {"name": "describe_instance_types", "role": "tool"}
    arguments = json.loads(assistant_output['tool_calls']['function']['arguments'])
    param_1 = arguments['param_1']
    param_2 = arguments['param_2']
    # 调用工具
    tool_info['content'] = describe_instance_types(param_1, param_2)
elif assistant_output['tool_calls']['function']['name'] == 'describe_regions':
    tool_info = {"name": "describe_regions", "role": "tool"}
    # 调用工具
    tool_info['content'] = describe_regions()

## Step3.返回结果
messages.append(tool_info)
final_assistant_output = call_llm(messages)

## 工具定义
TOOLS = [{"type": "function","function": {"name": "describe_regions","description": "查询地域","parameters": {},}},
         {"type": "function","function": {"name": "describe_instance_types","description": "查询实例规格表",
                                          "parameters": {"properties": {"param_1": { "类型、描述、举例" },"param_2": { "类型、描述、举例" },}}}}]
## 工具实现
def describe_regions():
    ...
## 工具实现
def describe_instance_types(param_1, param_2):
    ...
```

在这种模式下，工具的定义、实现和调用逻辑全部硬编码在 Host 应用中。这导致了高度的耦合：每增加或修改一个工具，都需要修改并重新部署整个应用。

场景二：基于 MCP 的 Python 伪代码实现（Prompt 模式意图识别 + MCP Tools Call）

```ts
## Step0.初始化MCP客户端
client = Client()
await client.connect("...")
tools = await client.list_tools()

## Step1.调用大模型意图识别
messages = [
    {"role": "system", "content": build_mcp_tool_prompt(tools)},#注意：这里是文本拼接了Prompt
    {"role": "user", "content": "帮我查一下杭州4C8G的ECS规格？"}
]
assistant_response = await call_llm(messages)

## Step2.解析意图识别结果，并且远程调用工具
tool_call = parse_tool_use(assistant_response)
result = await client.call_tool(name=tool_call["name"],arguments=tool_call["arguments"])

## Step3.结果输出
messages.append({"result": result})
final_response = await call_llm(messages)

## 工具定义 TOOLS 不再需要，client.list_tools()获得
## 工具实现 def describe_regions()不再需要，client.call_tool()调用

## 新增：通用意图识别提示词构建
def build_mcp_tool_prompt(tools)
    tool_descriptions = ""
    for tool in tools:
        tool_descriptions = tool_descriptions + f"<tool>{tool['name']}</tool>\n"
    return f"""
你是一个智能助手，可以使用以下工具来帮助用户解决问题。可用MCP工具列表：
{tool_descriptions}
请使用以下XML格式调用工具：
<tool_use><name>{{工具名称}}</name><arguments>{{JSON格式参数}}</arguments></tool_use>
示例：
<tool_use><name>DescribeInstanceTypes</name>
<arguments>{{"param_1": "cn-hangzhou", "param_2": "4C8G"}}</arguments></tool_use>"""

## 新增：解析大模型输出获得工具和入参
def parse_tool_use(response):
    tool_use_regex = re.compile(r"<tool_use>\s*<name>(.*?)</name>\s*<arguments>(.*?)</arguments>\s*</tool_use>",re.DOTALL)
    match = tool_use_regex.search(response)
    tool_name, arguments_str = match.groups()
    return {"name": tool_name, "arguments": arguments_str}
```

在 MCP 模式下，Host 应用转变为一个纯粹的 AI 决策和调度中心。它不关心工具如何实现，只关心如何通过标准化协议 mcp_client.call_tool() 来调用它们。这种彻底的解耦带来了巨大的工程优势：

*   标准化与互操作性：任何实现了 MCP Server 的工具，都可以被 Host“即插即用”，无需定制开发。
    
*   独立演进：工具（MCP Server）和 AI 应用（MCP Host）可以独立开发、部署和扩展。
    
*   可维护性：Host 的代码变得更加简洁，只关注核心的 AI 逻辑。
    

**4.3 协作而非替代：Host 中融合 MCP 与 Function Calling 的策略**

一个成熟的 Host 实现，会根据模型的具体能力，智能地选择使用 Function Calling 或基于 Prompt 的指令来与 MCP Server 交互。CherryStudio 项目就是这种融合策略的绝佳范例。

在其 Host 实现中，存在一个双重意图识别机制：

1. 能力检测：首先通过 isFunctionCallingModel 函数判断当前配置的 LLM 是否原生支持 Function Calling。

```ts
// src/config/models.ts  第356-359行
// 确认模型是不是支持Function Calling
export function isFunctionCallingModel(model?: Model): boolean {
  // Logic to check provider, model name, and user settings...
  if (['openai', 'anthropic', 'deepseek'].includes(model.provider)) {
    returntrue;
  }
  return FUNCTION_CALLING_REGEX.test(model.id);
}
```

2. 策略切换：

*   如果模型支持 Function Calling：Host 会将从 MCP Server 获取的工具列表，通过 mcpToolsToOpenAIChatTools 函数转换成符合 OpenAI 规范的 tools 参数，直接利用模型原生的 Function Calling 能力进行决策。
    
*   如果模型不支持：Host 会回退到 “Prompt Engineering” 模式，将工具定义拼接成 XML 格式的文本，注入 System Prompt 中，并依赖 parseToolUse 函数从 LLM 返回的纯文本中用正则表达式提取 <tool_use> 标签，来解析工具调用意图。
    

这个设计证明了：Function Calling 和基于 Prompt 的指令生成，都是 Host 用来驱动 MCP 协议的 “上层决策方法”。MCP 作为底层基础设施，对上层决策方式保持中立。因此，MCP 不仅不是 Function Calling 的替代品，反而可以与 Function Calling（或其他意图识别技术）无缝协作，各司其职。

第五章：实践出真知：决定 MCP 应用效果的三大关键因素

至此，“采用 MCP 就等于获得优异 AI 应用效果” 的观点显然已不成立。MCP 本身作为一套工程协议，只提供了 “可能性”，并未保证 “有效性”。一个高性能的 MCP 应用，其最终表现取决于 Host 如何协同三大关键技术因素：工具（Tools）的质量、提示词（Prompt）的精度，以及大模型（LLM）自身的核心能力。这三者共同构成了一个系统的性能边界。

**5.1 因素一：工具的质量与原子性**

工具是 AI 行动的触手，其设计质量直接决定了行动的有效范围和可靠性。

*   原子性：工具应遵循单一职责原则。一个 “管理云主机” 的万能工具，其决策复杂度远高于一组原子化的工具（如 create_instance, get_instance_status, stop_instance）。原子化使 LLM 能更灵活地组合工具以完成复杂任务。
    
*   接口定义的精确性：工具的 inputSchema 是 LLM 生成参数的唯一依据。一个模糊的描述如 "description": "pod name"，远不如一个精确的定义有效："description": "The name of the Kubernetes pod, must follow RFC 1123 DNS label format"。后者能极大提升参数生成的准确率。
    
*   可靠性与幂等性：工具后端必须高可用。对于任何写操作，接口应设计为幂等的，允许 Host 在网络超时等不确定情况下安全重试，而不会产生重复创建资源的副作用。
    

**5.2 因素二：提示词工程的精确性**

在依赖 Prompt 进行意图识别时，Prompt 是 Host 传递给 LLM 的 “作战指令”，其工程质量是 AI 决策水平的核心变量。

*   系统提示词的指令约束力：System Prompt 必须对 LLM 的行为进行强力约束，包括明确的角色定义（“你是一个 k8s 运维助手”）、不可逾越的规则（“禁止在生产环境执行删除操作前进行二次确认”），以及清晰无歧义的工具调用格式规约（如 XML 或 JSON 示例）。
    
*   工具描述的信息熵 ：工具的 description 字段是 LLM 选择工具的核心依据。描述得太宽泛（如 "Handles files"）会导致滥用；太狭隘或充满技术术语（如 "Executes a POST to /v1/data-proc"）则可能导致模型无法将其与用户意图关联。好的描述应在用户意图和技术实现间找到平衡。
    

**5.3 因素三：大模型自身的核心能力**

LLM 是系统的 “智力核心”，其基础能力决定了应用效果的理论上限。

*   推理与规划能力：面对一个需要多步工具调用的复杂任务（如 “为新功能 feat-x 创建一个 CI/CD 流水线”），模型的规划能力至关重要。它需要能生成一个逻辑自洽的工具调用序列：create_git_branch -> build_docker_image -> deploy_helm_chart。
    
*   指令遵循与格式生成能力：这是衡量模型 “可靠性” 的核心指标。一个频繁“越狱”、不遵循指定 XML/JSON 格式输出的模型，会给 Host 的解析层带来巨大的工程负担和不确定性。
    
*   参数生成准确性：即便选对工具，能否从用户模糊的对话（如 “查一下 web 主服务的日志”）中准确提取并生成符合 Schema 的参数（如 {"pod_name":"webapp-main-7f9c8d...","namespace":"production"}），是决定工具调用成败的 “最后一公里”，直接依赖模型对上下文和 Schema 的理解能力。
    

第六章：现实的权衡：MCP 方案的固有挑战与成本

在认可 MCP 作为工程框架的价值之后，我们必须以现实主义的视角审视其引入的固有挑战与成本。任何架构决策都是一种权衡，MCP 也不例外。尤其是在 Host 依赖 Prompt 而非原生 Function Calling 来实现意图识别时，其代价在两个维度上尤为突出：Token 成本和意图识别的稳定性。

**6.1 挑战一：持续增长的 Token 成本与性能开销**

基于 Prompt 的 MCP 方案在 Token 消耗上存在显著的 “先天不足”，这直接转化为高昂的 API 调用成本和更长的响应延迟。其成本结构主要由两部分构成：

1. 固定的 “启动成本”：庞大的 System Prompt 为了让一个通用 LLM 能够理解并使用外部工具，Host 必须在每次请求的 System Prompt 中注入一个庞大的 “操作手册”。这包括：

*   指令与规则定义：详细的角色定位和行为约束。
    
*   格式与示例规约：冗长的工具调用格式（如 XML）定义和 few-shot 示例。
    
*   完整的工具清单序列化：将所有可用工具的名称、描述、以及参数的 JSON Schema 全部序列化为文本。
    

当工具集规模扩大时，这部分成本会急剧膨胀。以一个集成了 19 个工具（阿里云 ECS 的 OpenAPI）的 MCP Server 为例，在一次典型的调用中，仅 System Prompt 部分就可能消耗高达 60,000 个 Token。这笔巨大的固定开销在每一轮对话中都会产生。

```
<tool>
  <name>miaodao-Ecs_20140526_DescribeAvailableResource</name>
  <description>简介:
查询可用区的资源库存状态。您可以在某一可用区创建实例（RunInstances）或者修改实例规格（ModifyInstanceSpec）时查询该可用区的资源库存状态。
详细描述:
参数`DestinationResource`的取值有不同的逻辑与要求。在下列的顺序列表中，顺序越低的取值需要设置更多的参数，不支持通过低顺序的取值筛选高顺序的资源类别。
- 取值顺序：`Zone > IoOptimized > InstanceType = Network = ddh > SystemDisk > DataDisk`
- 取值示例：
    - 若参数`DestinationResource`取值为`DataDisk`：
         - `ResourceType`取值为`disk`表示查询与ECS实例规格无关的数据盘类型，可以不传入参数`InstanceType`。
        - `ResourceType`取值为`instance`表示查询待挂载至ECS实例的数据盘类型，由于实例规格对数据盘有限制，所以需要同时指定`InstanceType`与参数`DataDiskCategory  `。
    - 若参数`DestinationResource`取值为`SystemDisk`，`ResourceType`取值为`instance`，由于ECS实例规格对系统盘存在限制，则必须要传入参数`InstanceType`。
    - 若参数`DestinationResource`取值为`InstanceType`，建议传入参数`IoOptimized`和`InstanceType`。
    - 查询指定地域下所有可用区的ecs.g5.large库存供应情况：`RegionId=cn-hangzhou &DestinationResource=InstanceType &IoOptimized=optimized &InstanceType=ecs.g5.large`。
    - 查询指定地域下有ecs.g5.large库存供应的可用区列表：`RegionId=cn-hangzhou &DestinationResource=Zone &IoOptimized=optimized &InstanceType=ecs.g5.large`。
<details>
<summary>查询杭州地域供应实例规格为ecs.g5.large的可用区列表。</summary>
```
"RegionId": "cn-hangzhou",
"DestinationResource": "Zone"，
"InstanceType": "ecs.g5.large"
```
</details>
<details>
<summary>查询杭州地域、所有可用区下的实例类型为ecs.g5.large的库存。</summary>
```
"RegionId": "cn-hangzhou",
"DestinationResource": "InstanceType"，
"InstanceType": "ecs.g5.large"
```
</details>
<details>
<summary>【只购买数据盘】查询杭州地域、可用区b下的数据盘类型为cloud_efficiency的库存。</summary>
```
"RegionId": "cn-hangzhou",
"ZoneId": "cn-hangzhou-b",
"ResourceType": "disk",
"DestinationResource": "DataDisk"
```
</details>
<details>
<summary>【购买ECS实例和系统盘】查询杭州地域、可用区b下的实例类型为ecs.g7.large、系统盘类型为cloud_essd的库存。</summary>
```
"RegionId": "cn-hangzhou",
"ZoneId": "cn-hangzhou-b",
"ResourceType": "instance",
"InstanceType": "ecs.g7.large",
"DestinationResource": "SystemDisk",
"SystemDiskCategory": "cloud_essd"
```
</details>
额外请求参数描述:

，用户描述的地域信息需要同时传递给 x_mcp_region_id 参数。</description>
  <arguments>
    {"type":"object","properties":{"DedicatedHostId":{"type":"string","description":"专有宿主机ID。\n\nexample value:dh-bp165p6xk2tlw61e","example":"dh-bp165p6xk2tlw61e****","exampleSetFlag":true},"IoOptimized":{"type":"string","description":"是否为I/O优化实例。取值范围： \n         \n- none：非I/O优化实例。\n- optimized：I/O优化实例。\n\n\n默认值：optimized。\n\nexample value:optimized","example":"optimized","exampleSetFlag":true},"ZoneId":{"type":"string","description":"可用区ID。\n\n默认值：无。返回该地域（`RegionId`）下所有可用区符合查询条件的资源。\n\nexample value:cn-hangzhou-e","example":"cn-hangzhou-e","exampleSetFlag":true},"InstanceChargeType":{"type":"string","description":"资源的计费方式。更多信息，请参见计费概述。取值范围： \n       \n- PrePaid：包年包月。  \n- PostPaid：按量付费。\n\n默认值：PostPaid。\n\nexample value:PrePaid","example":"PrePaid","exampleSetFlag":true},"DestinationResource":{"type":"string","description":"要查询的资源类型。取值范围： \n         \n- Zone：可用区。\n- IoOptimized：I/O优化。\n- InstanceType：实例规格。\n- Network：网络类型。\n- ddh：专有宿主机。\n- SystemDisk：系统盘。\n- DataDisk：数据盘。\n\n>当DestinationResource取值为`SystemDisk`时，由于系统盘受实例规格限制，此时必须传入InstanceType。\n\n参数DestinationResource的取值方式请参见本文中的接口说明。\n\nexample value:InstanceType","example":"InstanceType","exampleSetFlag":true},"Memory":{"type":"number","description":"实例规格的内存大小，单位为GiB。取值参见实例规格族。\n\n当DestinationResource取值为InstanceType时，Memory才为有效参数。\n\nexample value:8.0","format":"float","example":"8.0","exampleSetFlag":true},"SpotDuration":{"type":"integer","description":"抢占式实例的保留时长，单位为小时。 默认值：1。取值范围：\n- 1：创建后阿里云会保证实例运行1小时不会被自动释放；超过1小时后，系统会自动比较出价与市场价格、检查资源库存，来决定实例的持有和回收。\n- 0：创建后，阿里云不保证实例运行1小时，系统会自动比较出价与市场价格、检查资源库存，来决定实例的持有和回收。\n\n实例回收前5分钟阿里云会通过ECS系统事件向您发送通知。抢占式实例按秒计费，建议您结合具体任务执行耗时来选择合适的保留时长。\n\n> 当`InstanceChargeType`取值为`PostPaid`，并且`SpotStrategy`值为`SpotWithPriceLimit`或`SpotAsPriceGo`时该参数生效。\n\nexample value:1","format":"int32","example":"1","exampleSetFlag":true},"ResourceType":{"type":"string","description":"资源类型。取值范围：\n\n- instance：ECS实例。\n- disk：云盘。\n- reservedinstance：预留实例券。\n- ddh：专有宿主机。\n\nexample value:instance","example":"instance","exampleSetFlag":true},"SystemDiskCategory":{"type":"string","description":"系统盘类型。取值范围： \n         \n- cloud：普通云盘。\n- cloud_efficiency：高效云盘。\n- cloud_ssd：SSD云盘。\n- ephemeral_ssd：本地SSD盘。\n- cloud_essd：ESSD云盘。\n- cloud_auto：ESSD AutoPL云盘。\n<props=\"china\">\n- cloud_essd_entry：ESSD Entry云盘。\n</props>\n\n默认值：cloud_efficiency。\n\n> 参数ResourceType取值为instance、DestinationResource取值为DataDisk时，参数SystemDiskCategory是必选参数。如果未传递参数值，则以默认值生效。\n\nexample value:cloud_ssd","example":"cloud_ssd","exampleSetFlag":true},"NetworkCategory":{"type":"string","description":"网络类型。取值范围： \n        \n- vpc：专有网络。\n- classic：经典网络。\n\nexample value:vpc","example":"vpc","exampleSetFlag":true},"Cores":{"type":"integer","description":"实例规格的vCPU内核数目。取值参见实例规格族。\n\n当DestinationResource取值为InstanceType时，Cores才为有效参数。\n\nexample value:2","format":"int32","example":"2","exampleSetFlag":true},"Scope":{"type":"string","description":"预留实例券的范围。取值范围：\n         \n- Region：地域级别。\n- Zone：可用区级别。\n\nexample value:Region","example":"Region","exampleSetFlag":true},"RegionId":{"type":"string","description":"目标地域ID。您可以调用DescribeRegions查看最新的阿里云地域列表。\n\nexample value:cn-hangzhou","example":"cn-hangzhou","exampleSetFlag":true},"x_mcp_region_id":{"title":"X Mcp Region Id","type":"string","description":"To modify the parameters of the operation region, you need to identify the user's behavior and determine whether the operation needs to be performed in a specific region. The supported range is:\\n`cn-qingdao`,`cn-beijing`,`cn-zhangjiakou`,`cn-zhengzhou-jva`,`cn-huhehaote`,`cn-wulanchabu`,`cn-hangzhou`,`cn-shanghai`,`cn-nanjing`,`cn-fuzhou`,`cn-shenzhen`,`cn-heyuan`,`cn-guangzhou`,`cn-chengdu`,`cn-wuhan-lr`,`cn-hongkong`,`ap-northeast-1`,`ap-northeast-2`,`ap-southeast-1`,`ap-southeast-2`,`ap-southeast-3`,`ap-southeast-5`,`ap-southeast-6`,`us-east-1`,`us-west-1`,`eu-west-1`,`eu-central-1`,`ap-south-1`,`me-east-1`,`cn-hangzhou-finance`,`cn-shanghai-finance-1`,`cn-shenzhen-finance-1`,`ap-southeast-7`,`cn-beijing-finance-1`,`me-central-1`,`cn-heyuan-acdr-1`,`na-south-1`,`us-southeast-1`","exampleSetFlag":false,"anyOf":[{"type":"string","exampleSetFlag":false},{"type":"null","exampleSetFlag":false}]},"InstanceType":{"type":"string","description":"实例规格。更多信息，请参见实例规格族，您也可以调用DescribeInstanceTypes接口获得最新的规格表。\n\n参数InstanceType的取值方式请参见本文开头的接口说明。\n\nexample value:ecs.g5.large","example":"ecs.g5.large","exampleSetFlag":true},"DataDiskCategory":{"type":"string","description":"数据盘类型。取值范围： \n         \n- cloud：普通云盘。\n- cloud_efficiency：高效云盘。\n- cloud_ssd：SSD云盘。\n- ephemeral_ssd：本地SSD盘。\n- cloud_essd：ESSD云盘。\n- cloud_auto：ESSD AutoPL云盘。\n<props=\"china\">\n- cloud_essd_entry：ESSD Entry云盘。\n</props>\n\nexample value:cloud_ssd","example":"cloud_ssd","exampleSetFlag":true},"SpotStrategy":{"type":"string","description":"按量付费实例的竞价策略。取值范围： \n         \n- NoSpot：正常按量付费实例。\n- SpotWithPriceLimit：设置上限价格的抢占式实例。\n- SpotAsPriceGo：系统自动出价，最高按量付费价格。\n\n默认值：NoSpot。\n\n当参数`InstanceChargeType`取值为`PostPaid`时，参数`SpotStrategy`才有效。\n\nexample value:NoSpot","example":"NoSpot","exampleSetFlag":true}},"required":["RegionId","DestinationResource"],"additionalProperties":false,"$defs":{}}
  </arguments>
</tool>
```

《某一个 ECS 的 API 接口说明被拼接到 SYSTEM Prompt 中的样子》

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naLTaj6dickraLHIwn2sibGBKwibeNgq7f2fRpSZTTibOVho3lkhEaomRHv3iamIFGyuuAADAfduPgaQFKQ/640?wx_fmt=png&from=appmsg#imgIndex=5)

《一个 19 个 ECS 的常用 API 用于 ECS 实例的创建的 MCP 服务的调用量监控》

2. 滚雪球式的 “滚动成本”：上下文累积在多轮对话中，为了维持状态，每一次新的 API 请求都必须包含完整的历史上下文。这包括：

*   之前的用户提问和模型回答。
    
*   模型生成的工具调用 XML 块。
    
*   工具执行后返回的结果（可能是冗长的 JSON 或文本）。
    

这种机制导致请求的 payload 随着对话的进行而线性增长，形成 “滚雪球” 效应。一轮包含多次工具调用的复杂对话，其最终的 Token 消耗总量可能达到数十万甚至更高，这对成本和系统延迟（TTFT）都构成了严峻的挑战。

**6.2 挑战二：依赖模型能力的意图识别稳定性问题**

当 Host 将意图识别的重任完全交给 LLM 对 Prompt 的 “自由发挥” 时，系统的可靠性便直接与模型的 “指令遵循能力” 挂钩，这引入了诸多工程上的不确定性。

1. 格式污染与解析脆弱性  

整个机制建立在一个脆弱的假设之上：模型能 100% 稳定地生成符合预设格式（如 XML）的文本。然而在实践中，格式错误是高发问题，迫使开发者必须编写大量防御性的解析和清洗代码。常见故障模式包括：

*   标签损坏：生成未闭合或错误的 XML 标签，如 <argumens>。
    
*   JSON 参数畸形：在 <arguments> 块内生成不符合 JSON 规范的字符串（如引号缺失、尾部逗号），导致反序列化失败。
    
*   内容混杂：在标准 XML 块前后夹杂 “好的，我将为您调用工具...” 等无关对话，增加了提取目标的复杂度。
    

2. 参数幻觉  

与原生 Function Calling 相比，基于 Prompt 的参数生成更容易出现 “幻觉”。模型完全依赖其对工具 description 的文本理解来构建参数。这导致了两种典型风险：

*   参数名幻觉：编造一个 inputSchema 中不存在的参数名。
    
*   参数值幻觉：从用户对话中错误地提取或捏造一个值，并填入参数中。
    

例如，当用户说 “帮我查下北京的 ECS 规格”，如果不对参数进行描述，模型可能会错误地生成 {"city": "北京市", "district": "庞各庄"}，而工具实际只接受 {"region": "cn-beijing"}。这种细微的偏差是导致工具调用失败的常见根源。

3. 对提示词工程的过度敏感性

系统的表现与 Prompt 的微观设计高度耦合。对 System Prompt 的任何微小调整，都可能引起模型行为的蝴蝶效应，导致原有的稳定表现衰退。这使得 Prompt 的维护成本极高，每一次迭代都需要进行大量的回归测试，以确保关键路径的稳定性不受影响。

选择基于 Prompt 的 MCP 实现，本质上是用更高的灵活性和模型普适性，换取了更高的运营成本和更低的可预测性。这要求团队在系统设计之初就必须对这些成本和风险有清醒的认识，并构建相应的监控、告警和容错机制来应对这种固有的不确定性。

结论：回归工程本质——MCP 的真正价值与未来展望

经过从架构、源码到实践的层层剖析，回归本文的核心论点：MCP 的光环，并非源于某种神秘的 AI“黑科技”，而是植根于其卓越的软件工程设计原则。

我们已经证明，MCP 本身并不产生智能。它既不是更聪明的 Function Calling，更不是解决 AI Agent 所有难题的银弹。相反，它是一套严谨、模型无关的软件工程框架，其核心价值在于为构建一个复杂、异构、可互操作的 AI 工具生态系统，提供了坚实的工程底座。

MCP 的真正贡献，在于它通过 “关注点分离” 这一经典的工程思想，实现了不确定的 AI 智能（Host）与确定的能力执行（Server）之间的彻底解耦。这一设计带来了三个优势：

*   标准化：它为 “AI 工具” 提供了统一的接口规范，如同 Web 世界的 HTTP 协议，让任何能力都能被标准化地接入和消费。
    
*   解耦：它将 AI 应用的 “大脑”（Host）与 “手脚”（Server）分离开来，使得二者可以独立开发、部署、迭代和扩展，极大地提升了系统的可维护性和演进效率。
    
*   互操作性：它打破了模型、厂商与工具之间的壁垒，为构建一个跨平台、跨语言的统一 AI Agent 生态铺平了道路。
    

这种工程层面的价值，使得开发者能够从繁琐的、非标的底层通信和工具适配工作中解放出来，将全部精力聚焦于真正决定应用成败的核心——提升 Host 的 “智能”：优化提示词工程、选择更强的模型、设计更复杂的任务规划与调度逻辑。

随着 AI Agent 从单体应用走向分布式、协作式的 “Agent 网络”，MCP 所倡导的标准化与互操作性将变得愈发重要。它不仅是当前构建健壮 AI 应用的务实选择，更是未来实现“万物皆可为 AI 之工具” 这一宏大愿景的关键。对于每一位致力于将 AI 从 “玩具” 推向 “工业级生产力” 的工程师而言，深刻理解并善用 MCP 的工程本质，或许正是我们在这场技术变革中，最强大的武器。

以上。

参考文档：

参考 1：MCP 架构：https://modelcontextprotocol.io/specification/2025-06-18/architecture

参考 2：FunctionCalling 与 MCP：https://blog.dailydoseofds.com/p/function-calling-and-mcp-for-llms

参考 3：CherryStudio：https://github.com/CherryHQ/cherry-studio

**轻松实现客服数据智能分析与高效存储**

针对大模型应用开发中存在的环境搭建复杂、数据库集成困难等问题，本方案基于阿里云 DMS 原生托管 Dify 工作空间，深度集成云数据库与阿里云百炼（简称 “百炼”）大模型服务，快速搭建开箱即用的客服对话数据质检服务，显著降低数据库 + AI 应用的开发门槛。

点击阅读原文查看详情。