---
source: https://mp.weixin.qq.com/s/M-SHqDgMK__dOwxH_bgEGw
create: 2025-11-25 16:32
read: false
knowledge: false
---
![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naJrW53r8unHlzGms6BHD12S7efKOfhqsotWf4L1DiaFiat1r1icyYibVSuntYp8OvBUHXicqrgDHlZqp4g/640?wx_fmt=jpeg&from=appmsg#imgIndex=0)

前两天 Java Agent 生态迎来重大发布，Spring AI 发布 1.1.0 GA 正式版本，紧接着，Spring AI Alibaba 1.1 版本正式发布。1.1 版本是在总结 1.0 版本企业实践基础上发布的新版本，标志着构建企业级、生产就绪的 AI 智能体（Agent）应用进入了一个新的阶段。

*   官方文档：https://java2ai.com/
    
*   Github Repo：https://github.com/alibaba/spring-ai-alibaba
    

Spring AI Alibaba 的核心目标是：

*   提供构建 Agentic 智能体应用最简单的方式，您只需不到 10 行代码就可以启动并运行一个智能体应用。
    
*   同时，对于企业级场景需要的 Multi-agent、Workflow 工作流编排提供强大支持。
    

本文将深入解读 1.1 版本的核心能力，从基础的 `ReactAgent` 构建到复杂的 “上下文工程”，再到强大的多智能体（Multi-agent）协作，帮助您全面了解如何利用该框架构建下一代智能应用。

架构概览：三层设计

Spring AI Alibaba 项目在架构上包含三个清晰的层次：

1.Agent Framework：以 `ReactAgent` 设计理念为核心的 Agent 开发框架，内置了自动上下文工程和 Human In The Loop 等高级能力。

2.Graph：一个更底层级别的工作流和多代理协调框架，是 Agent Framework 的底层运行时基座，用于实现复杂的工作流编排，同时对用户开放 API。

3.Augmented LLM：基于 Spring AI 框架的底层原子抽象，提供了模型、工具、消息、向量存储等构建 LLM 应用的基础。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLlc4VK3D74sm8IcSGNQuZ1Rj19DRMQxBCKT80NWsvEcD0RSW3vJxibAKTVSohjca2ClKjM2tyLDrw/640?wx_fmt=jpeg&from=appmsg#imgIndex=1)

快速开始

社区在链接 [1] 提供了一个简单的 ChatBot 智能体示例，可以执行 Python 脚本、Shell 脚本、查看本地文件等。  

接下来我们尝试通过运行 ChatBot 示例快速体验 Spring AI Alibaba 1.1 版本。

**快速运行一个 ChatBot 示例**

1. 下载示例源码

```
git clone https://github.com/alibaba/spring-ai-alibaba.git
cd examples/chatbot

```

2. 启动 ChatBot 智能体

```
mvn spring-boot:run

```

3. 打开内置的 Agent Chat UI 和智能体聊天（测试智能体）

在示例的 Console 输出栏，可以看到一条 UI 地址 [2] 打印出来：  

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLlc4VK3D74sm8IcSGNQuZ1wicEkCJjEScRCvyib0Q7R7XaTpUuMF0uZnZrzcdY1oJ2DzENhmicKXoWg/640?wx_fmt=jpeg&from=appmsg#imgIndex=2)

点击打开浏览器页面即可与智能体聊天了，可以看到详细的工具调用、推理过程。

![](https://mmbiz.qpic.cn/mmbiz_gif/Z6bicxIx5naLlc4VK3D74sm8IcSGNQuZ1JdBQw0KrTLs8gMialXIgt26DLxvYPbjwzib0bL8Af4RMOvLfII3aibIIQ/640?wx_fmt=gif&from=appmsg#imgIndex=3)

**示例源码解读**

#### 首先，是需要在项目添加如下 POM 依赖：

在项目中添加如下依赖：

```
<dependencies>
  <!-- Spring AI Alibaba Agent Framework -->
  <dependency>
    <groupId>com.alibaba.cloud.ai</groupId>
    <artifactId>spring-ai-alibaba-agent-framework</artifactId>
    <version>1.1.0.0-M5</version>
  </dependency>
  <!-- DashScope ChatModel 支持（如果使用其他模型，请参考文档选择对应的 starter） -->
  <dependency>
    <groupId>com.alibaba.cloud.ai</groupId>
    <artifactId>spring-ai-alibaba-starter-dashscope</artifactId>
    <version>1.1.0.0-M5</version>
  </dependency>
  <!-- 【二选一】OpenAi ChatModel 支持 -->
  <!--
  <dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-openai</artifactId>
    <version>1.1.0.0-M4</version>
  </dependency>
  -->
  <!--【可选依赖】，studio可以默认带来Chat UI界面 -->
  <dependency>
    <groupId>com.alibaba.cloud.ai</groupId>
    <artifactId>spring-ai-alibaba-studio</artifactId>
    <version>1.1.0.0-M5</version>
  </dependency>
</dependencies>

```

#### 创建 Agent

创建一个 ChatBot Agent 只需要如下几行代码（本示例使用 Spring Boot 方式启动）：

```
// ChatModel 等模型通信实例可自动注入
@Bean
public ReactAgent chatbotReactAgent(ChatModel chatModel,
        ToolCallback executeShellCommand,
        ToolCallback executePythonCode,
        ToolCallback viewTextFile) {
    return ReactAgent.builder()
            .name("SAA")
            .model(chatModel)
            .instruction(INSTRUCTION)
            .enableLogging(true)
            .tools(
                    executeShellCommand,
                    executePythonCode,
                    viewTextFile
            )
            .build();
}

```

python 工具定义如下，这里使用 GraalVM Python 库执行脚本，具体 PythonTool 可在 Github 仓库查看源码：

```
@Bean
public ToolCallback executePythonCode(){
    return FunctionToolCallback.builder("execute_python_code", new PythonTool())
            .description(PythonTool.DESCRIPTION)
            .inputType(PythonTool.PythonRequest.class)
            .build();
}

```

核心设计理念

**ReactAgent**

`ReactAgent`是 1.1 版本的核心组件之一，它基于 ReAct（Reasoning + Acting） 范式。这意味着 Agent 不仅仅是调用 LLM，它还可以在一个循环中运行，通过 “思考（Reasoning）” 来分析任务、决定使用哪个 “工具（Acting）”，然后“观察（Observation）” 工具结果，并迭代此过程，直到任务完成。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLlc4VK3D74sm8IcSGNQuZ1ic1HJ7MH6P8BOJyWtkTibz0KVX5yzlOJ4BicicP4CwXwBOuAp7Kms6QFzw/640?wx_fmt=jpeg&from=appmsg#imgIndex=4)

ReactAgent 行为由三个核心组件定义：

1.Model (模型)：Agent 的 “大脑”，即 LLM 推理引擎，如 `DashScopeChatModel`。

2.Tools (工具)：赋予 Agent 执行操作的能力。您可以轻松定义一个工具：

```
// 定义一个搜索工具
publicclassSearchToolimplementsBiFunction<String, ToolContext, String> {
    @Override
    public String apply(String query, ToolContext toolContext){
        return"搜索结果：" + query;
    }
}
// 将其注册为工具回调
ToolCallback searchTool = FunctionToolCallback
    .builder("search", new SearchTool())
    .description("搜索信息的工具")
    .build();

```

3.System Prompt (系统提示)：通过 `systemPrompt` 或 `instruction` 参数，塑造 Agent 的角色和行为方式。

**Graph（Workflow）**

Graph 是 Agent Framework 的底层运行时基座，是一个低级工作流和多智能体编排框架。它通过 State（状态）、Node（节点） 和 Edge（边） 三个核心概念，使开发者能够实现复杂的应用程序编排。

Graph 引擎原生支持 Streaming（流式响应）、Human In the Loop（人工介入）、Memory & Context（记忆与上下文管理）等智能体核心能力。

Graph 提供了声明式的 Agentic API 与底层原子化的 Graph API 两种开发模式，开发者可以根据业务需求选择合适的方式。在复杂场景中，可以将 `ReactAgent` 作为 SubGraph Node 集成到 StateGraph 中，实现 Agent 与自定义 Node 的混合编排，支持顺序执行、并行处理、条件分支等复杂流程控制，为构建企业级多智能体应用提供了强大的底层支撑。

框架内置了多种流程型 Agent，支持不同的多智能体协作模式：

*   SequentialAgent（顺序执行）：按预定义顺序依次执行多个 Agent (A -> B -> C)。每个 Agent 的输出（通过 `outputKey` 指定）会传递给下一个 Agent，适用于需要按步骤顺序处理的任务，如 "写作 -> 评审 -> 发布" 的流程。
    
*   ParallelAgent（并行执行）：将相同的输入同时发送给所有子 Agent 并行处理，然后使用 `MergeStrategy` 合并结果。适用于需要同时执行多个独立任务的场景，如同时生成散文、诗歌和总结等。
    
*   LlmRoutingAgent（智能路由）：使用 LLM 根据用户输入和子 Agent 的 `description`，智能地选择一个最合适的子 Agent 来处理请求。适用于需要根据上下文动态选择专家 Agent 的场景，如根据问题类型自动路由到编程专家或写作专家。
    

上下文工程 (Context Engineering)

构建 Agent 最大的难点是使其可靠。Agent 失败通常不是因为 LLM 能力不足，而是因为没有向 LLM 传递 “正确” 的上下文。

上下文工程（Context Engineering）正是解决这一问题的核心理念：以正确的格式提供正确的信息和工具，使 LLM 能够可靠地完成任务。

Spring AI Alibaba 将上下文分为三类，并提供了精细化的控制机制：

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naLlc4VK3D74sm8IcSGNQuZ19fmxmPuQZ8EYOESfUEOBkMxnA3gaZ1oC5H2OXLgLvK2qoTIoHQxf0g/640?wx_fmt=png&from=appmsg#imgIndex=5)

Spring AI Alibaba 1.1 版本提供了一些常用的默认 Hook 和 Interceptor 实现：

1. 人工介入 (Human-in-the-Loop)

2.Planning（规划）

3. 模型调用限制（Model Call Limit）

4. 工具重试（Tool Retry）

5.LLM Tool Selector（LLM 工具选择器）

6.LLM Tool Emulator（LLM 工具模拟器）

7.Context Editing（上下文编辑）

**人工介入 (Human-in-the-Loop)**

在生产环境中，高风险操作（如删除数据库、发送邮件）需要人工监督。`HumanInTheLoopHook` (HITL) 完美解决了这个问题。

它允许 Agent 暂停执行，等待人工决策（批准、编辑或拒绝）后再继续。

1. 配置 Hook：在 Agent 上配置 `HumanInTheLoopHook`，指定需要审批的工具（如 `execute_sql`）。此功能必须配置 `saver` (检查点)。

```
HumanInTheLoopHook humanReviewHook = HumanInTheLoopHook.builder()
    .approvalOn("execute_sql", ToolConfig.builder().description("SQL执行需要审批").build())
    .build();
ReactAgent agent = ReactAgent.builder()
    .hooks(humanReviewHook)
    .saver(new MemorySaver()) // 必须配置 Saver
    .tools(executeSqlTool)
    .build();

```

2. 触发中断：当 Agent 尝试调用 `execute_sql` 时，执行会暂停。第一次 `invokeAndGetOutput` 调用会返回一个 `InterruptionMetadata` (中断元数据)。

```
String threadId = "user-123";
RunnableConfig config = RunnableConfig.builder().threadId(threadId).build();
Optional<NodeOutput> result = agent.invokeAndGetOutput("删除旧记录", config);
// 此时 result.get() 是 InterruptionMetadata

```

3. 人工决策与恢复：您的应用程序向用户展示中断信息。用户做出决策（例如 “批准”）后，您构建一个包含该决策的反馈，并再次调用 Agent 以恢复执行。

```
// 假设用户批准了
InterruptionMetadata approvalMetadata = buildApprovalFeedback(interruptionMetadata);
RunnableConfig resumeConfig = RunnableConfig.builder()
    .threadId(threadId) // 必须使用相同的 threadId
    .addMetadata(RunnableConfig.HUMAN_FEEDBACK_METADATA_KEY, approvalMetadata)
    .build();
// 第二次调用以恢复执行
Optional<NodeOutput> finalResult = agent.invokeAndGetOutput("", resumeConfig);

```

**消息压缩 (Summarization)**

当对话历史接近 Token 限制时，自动调用 LLM 压缩（总结）旧消息，防止上下文溢出。这是处理长期对话和多轮交互的关键能力，可以确保 Agent 在保持对话上下文的同时，不会因为消息历史过长而超出模型的上下文窗口限制。

适用场景：

*   超出上下文窗口的长期对话；
    
*   具有大量历史记录的多轮对话；
    
*   需要保留完整对话上下文的应用程序；
    

使用示例：

```
import com.alibaba.cloud.ai.graph.agent.hook.summarization.SummarizationHook;
import com.alibaba.cloud.ai.graph.checkpoint.savers.MemorySaver;
// 创建消息压缩 Hook
SummarizationHook summarizationHook = SummarizationHook.builder()
    .model(chatModel)  // 用于生成摘要的 ChatModel（可以使用更便宜的模型）
    .maxTokensBeforeSummary(4000)  // 触发摘要之前的最大 token 数
    .messagesToKeep(20)  // 摘要后保留的最新消息数
    .build();
// 使用
ReactAgent agent = ReactAgent.builder()
    .name("my_agent")
    .model(chatModel)
    .hooks(summarizationHook)
    .saver(new MemorySaver())  // 需要配置 Saver 来持久化状态
    .build();

```

配置选项：

*   model：用于生成摘要的 ChatModel（可以使用更便宜的模型来降低成本）；
    
*   maxTokensBeforeSummary：触发摘要之前的最大 token 数，当消息历史达到此阈值时自动触发压缩；
    
*   messagesToKeep：摘要后保留的最新消息数，确保最近的对话内容完整保留；
    

**Planning（规划）**

在执行工具之前强制执行一个规划步骤，以概述 Agent 将要采取的步骤。这对于需要执行复杂、多步骤任务的 Agent 特别有用，可以提高执行透明度，并便于调试错误。

适用场景：

*   需要执行复杂、多步骤任务的 Agent；
    
*   通过在执行前显示 Agent 的计划来提高透明度；
    
*   通过检查建议的计划来调试错误；
    

```
import com.alibaba.cloud.ai.graph.agent.interceptor.todolist.TodoListInterceptor;
ReactAgent agent = ReactAgent.builder()
    .name("planning_agent")
    .model(chatModel)
    .tools(myTool)
    .interceptors(TodoListInterceptor.builder().build())

```

**模型调用限制（Model Call Limit）**

限制模型调用次数以防止无限循环或过度成本。这是生产环境中成本控制的重要手段。

适用场景：

*   防止失控的 Agent 进行太多 API 调用；
    
*   在生产部署中强制执行成本控制；
    
*   在特定调用预算内测试 Agent 行为；
    

```
import com.alibaba.cloud.ai.graph.agent.hook.modelcalllimit.ModelCallLimitHook;
import com.alibaba.cloud.ai.graph.checkpoint.savers.MemorySaver;
ReactAgent agent = ReactAgent.builder()
    .name("my_agent")
    .model(chatModel)
    .hooks(ModelCallLimitHook.builder().runLimit(5).build())  // 限制模型调用次数为5次
    .saver(new MemorySaver())
    .build();

```

**工具重试（Tool Retry）**

自动重试失败的工具调用，具有可配置的指数退避策略。这对于处理外部 API 调用中的瞬态故障特别有用，可以提高依赖网络的工具的可靠性。

适用场景：

*   处理外部 API 调用中的瞬态故障；
    
*   提高依赖网络的工具的可靠性；
    
*   构建优雅处理临时错误的弹性 Agent；
    

```
import com.alibaba.cloud.ai.graph.agent.interceptor.toolretry.ToolRetryInterceptor;
ReactAgent agent = ReactAgent.builder()
    .name("resilient_agent")
    .model(chatModel)
    .tools(searchTool, databaseTool)
    .interceptors(ToolRetryInterceptor.builder()
        .maxRetries(2)
        .onFailure(ToolRetryInterceptor.OnFailureBehavior.RETURN_MESSAGE)
        .build())
    .build();

```

**LLM Tool Selector（LLM 工具选择器）**

使用一个 LLM 来决定在多个可用工具之间选择哪个工具。当多个工具可以实现相似目标时，可以根据细微的上下文差异进行智能选择。

适用场景：

*   当多个工具可以实现相似目标时；
    
*   需要根据细微的上下文差异进行工具选择；
    
*   动态选择最适合特定输入的工具；
    

```
import com.alibaba.cloud.ai.graph.agent.interceptor.toolselection.ToolSelectionInterceptor;
ReactAgent agent = ReactAgent.builder()
    .name("smart_selector_agent")
    .model(chatModel)
    .tools(tool1, tool2)
    .interceptors(ToolSelectionInterceptor.builder().build())
    .build();

```

**LLM Tool Emulator（LLM 工具模拟器）**

在没有实际执行工具的情况下，使用 LLM 模拟工具的输出。这对于演示、测试和开发阶段特别有用，可以在不产生实际成本或副作用的情况下测试 Agent 逻辑。

适用场景：

*   在演示或测试期间模拟 API；
    
*   在开发过程中为工具提供占位符行为；
    
*   在不产生实际成本或副作用的情况下测试 Agent 逻辑；
    

```
import com.alibaba.cloud.ai.graph.agent.interceptor.toolemulator.ToolEmulatorInterceptor;
ReactAgent agent = ReactAgent.builder()
    .name("emulator_agent")
    .model(chatModel)
    .tools(simulatedTool)
    .interceptors(ToolEmulatorInterceptor.builder().model(chatModel).build())
    .build();

```

**Context Editing（上下文编辑）**

在将上下文发送给 LLM 之前对其进行修改，以注入、删除或修改信息。这是上下文工程的核心能力之一，可以动态调整传递给模型的信息。

适用场景：

*   向 LLM 提供额外的上下文或指令；
    
*   从对话历史中删除不相关或冗余的信息；
    
*   动态修改上下文以引导 Agent 的行为；
    

```
import com.alibaba.cloud.ai.graph.agent.interceptor.contextediting.ContextEditingInterceptor;
ReactAgent agent = ReactAgent.builder()
    .name("context_aware_agent")
    .model(chatModel)
    .interceptors(ContextEditingInterceptor.builder()
        .trigger(120000)      // 当上下文超过120000 tokens时触发
        .clearAtLeast(60000)  // 至少清理60000 tokens
        .build())
    .build();

```

Hooks 与 Interceptors

Hooks (钩子) 和 Interceptors (拦截器) 是实现 “上下文工程” 的核心机制。它们允许您在 Agent 执行的每一步进行监控、修改、控制和强制执行。

*   Hooks (`AgentHook`, `ModelHook`)：在 Agent 或模型生命周期的特定节点（如 `beforeModel`, `afterModel`）_插入_自定义逻辑，如日志记录或消息修剪。
    
*   Interceptors (`ModelInterceptor`, `ToolInterceptor`)：_包裹_模型或工具的调用，允许您拦截和_修改_请求 / 响应，或实现重试、缓存、安全护栏等。
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLlc4VK3D74sm8IcSGNQuZ13ic4mMApokj530nCBB2dVGN6UjpbjFAx16h2r75FPDatArYjPb3G01Q/640?wx_fmt=jpeg&from=appmsg#imgIndex=6)

Hook 与 Interceptor 能力强大，具体使用方式和工作原理请参考官方文档 [3]。  

**示例 1：内容审核 Interceptor**

```
publicclassContentModerationInterceptorextendsModelInterceptor {
    privatestaticfinal List<String> BLOCKED_WORDS =
        List.of("敏感词1", "敏感词2", "敏感词3");
    @Override
    public ModelResponse interceptModel(ModelRequest request, ModelCallHandler handler){
        // 检查输入
        for (Message msg : request.getMessages()) {
            String content = msg.getText().toLowerCase();
            for (String blocked : BLOCKED_WORDS) {
                if (content.contains(blocked)) {
                    return ModelResponse.blocked(
                        "检测到不适当的内容，请修改您的输入"
                    );
                }
            }
        }
        // 执行模型调用
        ModelResponse response = handler.call(request);
        // 检查输出
        String output = response.getContent();
        for (String blocked : BLOCKED_WORDS) {
            if (output.contains(blocked)) {
                // 清理输出
                output = output.replaceAll(blocked, "[已过滤]");
                return response.withContent(output);
            }
        }
        return response;
    }
    @Override
    public String getName(){
        return"ContentModerationInterceptor";
    }
}

```

**示例 2：性能监控 - 使用 Interceptor**

使用 `ModelInterceptor` 和 `ToolInterceptor` 监控模型和工具调用的性能：

```
publicclassModelPerformanceInterceptorextendsModelInterceptor {
    @Override
    public ModelResponse interceptModel(ModelRequest request, ModelCallHandler handler){
        // 请求前记录
        System.out.println("发送请求到模型: " + request.getMessages().size() + " 条消息");
        long startTime = System.currentTimeMillis();
        // 执行实际调用
        ModelResponse response = handler.call(request);
        // 响应后记录
        long duration = System.currentTimeMillis() - startTime;
        System.out.println("模型响应耗时: " + duration + "ms");
        return response;
    }
    @Override
    public String getName(){
        return"ModelPerformanceInterceptor";
    }
}
// 工具调用性能监控
publicclassToolPerformanceInterceptorextendsToolInterceptor {
    @Override
    public ToolCallResponse interceptToolCall(ToolCallRequest request, ToolCallHandler handler){
        String toolName = request.getToolName();
        long startTime = System.currentTimeMillis();
        System.out.println("执行工具: " + toolName);
        try {
            ToolCallResponse response = handler.call(request);
            long duration = System.currentTimeMillis() - startTime;
            System.out.println("工具 " + toolName + " 执行成功 (耗时: " + duration + "ms)");
            return response;
        } catch (Exception e) {
            long duration = System.currentTimeMillis() - startTime;
            System.err.println("工具 " + toolName + " 执行失败 (耗时: " + duration + "ms): " + e.getMessage());
            return ToolCallResponse.of(
                request.getToolCallId(),
                request.getToolName(),
                "工具执行失败: " + e.getMessage()
            );
        }
    }
    @Override
    public String getName(){
        return"ToolPerformanceInterceptor";
    }
}
// 使用示例
ReactAgent agent = ReactAgent.builder()
    .name("monitored_agent")
    .model(chatModel)
    .tools(tools)
    .interceptors(new ModelPerformanceInterceptor())
    .interceptors(new ToolPerformanceInterceptor())
    .build();

```

**示例 3：工具缓存 Interceptor**

```
publicclassToolCacheInterceptorextendsToolInterceptor {
    private Map<String, ToolCallResponse> cache = new ConcurrentHashMap<>();
    privatefinallong ttlMs;
    publicToolCacheInterceptor(long ttlMs){
        this.ttlMs = ttlMs;
    }
    @Override
    public ToolCallResponse interceptToolCall(ToolCallRequest request, ToolCallHandler handler){
        String cacheKey = generateCacheKey(request);
        // 检查缓存
        ToolCallResponse cached = cache.get(cacheKey);
        if (cached != null && !isExpired(cached)) {
            System.out.println("缓存命中: " + request.getToolName());
            return cached;
        }
        // 执行工具
        ToolCallResponse response = handler.call(request);
        // 缓存结果
        cache.put(cacheKey, response);
        return response;
    }
    @Override
    public String getName(){
        return"ToolCacheInterceptor";
    }
    private String generateCacheKey(ToolCallRequest request){
        return request.getToolName() + ":" +
               request.getArguments();
    }
    private boolean isExpired(ToolCallResponse response){
        // 实现 TTL 检查逻辑
        returnfalse;
    }
}

```

Memory 与状态管理

Spring AI Alibaba 将短期记忆作为 Agent 状态的一部分进行管理。

通过将这些存储在 Graph 的状态中，Agent 可以访问给定对话的完整上下文，同时保持不同对话之间的分离。状态使用 checkpointer 持久化到数据库（或内存），以便可以随时恢复线程。短期记忆在调用 Agent 或完成步骤（如工具调用）时更新，并在每个步骤开始时读取状态。

**启用短期记忆**

要启用会话级持久化，您只需在创建 Agent 时指定一个 `checkpointer` (保存器)。

```
import com.alibaba.cloud.ai.graph.checkpoint.savers.MemorySaver;
import com.alibaba.cloud.ai.graph.RunnableConfig;
// 1. 配置内存存储
ReactAgent agent = ReactAgent.builder()
    .name("chat_agent")
    .model(chatModel)
    .saver(new MemorySaver()) //
    .build();
// 2. 使用 thread_id 维护对话上下文
RunnableConfig config = RunnableConfig.builder()
    .threadId("user_123") //
    .build();
agent.call("我叫张三", config);
agent.call("我叫什么名字？", config);  // Agent 会回答: "你叫张三"

```

在生产环境中，您可以轻松换成 `RedisSaver` 或 `MongoSaver` 等持久化存储。

**记忆带来的上下文过长问题**

保留所有对话历史是实现短期记忆最常见的形式。但较长的对话对历史可能会导致大模型 LLM 上下文窗口超限，导致上下文丢失或报错。

即使你在使用的大模型上下文长度足够大，大多数模型在处理较长上下文时的表现仍然很差。因为很多模型会被过时或偏离主题的内容 "分散注意力"。同时，过长的上下文，还会带来响应时间变长、Token 成本增加等问题。

在 Spring AI ALibaba 中，ReactAgent 使用 `messages` 记录和传递上下文，其中包括指令（SystemMessage）和输入（UserMessage）。在 ReactAgent 中，消息（Message）在用户输入和模型响应之间交替，导致消息列表随着时间的推移变得越来越长。由于上下文窗口有限，许多应用程序可以从使用技术来移除或 "忘记" 过时信息中受益，即 “上下文工程”。

**示例一：在工具中读取 / 写入短期记忆**

使用 `ToolContext` 参数在工具中访问短期记忆（状态）。

```
publicclassUserInfoToolimplementsBiFunction<String, ToolContext, String> {
    @Override
    public String apply(String query, ToolContext toolContext){
        // 从上下文中获取用户信息
        RunnableConfig config = (RunnableConfig) toolContext.getContext().get("config");
        String userId = (String) config.metadata("user_id").orElse("");
        if ("user_123".equals(userId)) {
            return"用户是 John Smith";
        } else {
            return"未知用户";
        }
    }
}
// 创建工具
ToolCallback getUserInfoTool = FunctionToolCallback
    .builder("get_user_info", new UserInfoTool())
    .description("查找用户信息")
    .inputType(String.class)
    .build();
// 使用
ReactAgent agent = ReactAgent.builder()
    .name("my_agent")
    .model(chatModel)
    .tools(getUserInfoTool)
    .saver(new MemorySaver())
    .build();
RunnableConfig config = RunnableConfig.builder()
    .threadId("1")
    .addMetadata("user_id", "user_123")
    .build();
AssistantMessage response = agent.call("获取用户信息", config);
System.out.println(response.getText());

```

**示例二：在工具中读取 / 写入长期记忆**

下面的示例展示了如何创建一个工具，让 Agent 能够查询用户信息。

```
// 定义请求和响应记录
record GetMemoryRequest(List<String> namespace, String key){ }
record MemoryResponse(String message, Map<String, Object> value){ }
// 创建获取用户信息的工具
BiFunction<GetMemoryRequest, ToolContext, MemoryResponse> getUserInfoFunction =
        (request, context) -> {
            RunnableConfig runnableConfig = (RunnableConfig) context.getContext().get("config");
            Store store = runnableConfig.store();
            Optional<StoreItem> itemOpt = store.getItem(request.namespace(), request.key());
            if (itemOpt.isPresent()) {
                Map<String, Object> value = itemOpt.get().getValue();
                returnnew MemoryResponse("找到用户信息", value);
            }
            returnnew MemoryResponse("未找到用户", Map.of());
        };
ToolCallback getUserInfoTool = FunctionToolCallback.builder("getUserInfo", getUserInfoFunction)
        .description("查询用户信息")
        .inputType(GetMemoryRequest.class)
        .build();
// 创建Agent
ReactAgent agent = ReactAgent.builder()
        .name("memory_agent")
        .model(chatModel)
        .tools(getUserInfoTool)
        .saver(new MemorySaver())
        .build();
// 创建内存存储
MemoryStore store = new MemoryStore();
// 在Store中放入模拟数据，实际应用中，存储可能是其他流程中生成
mockInsertToStore(store);
// 运行Agent
RunnableConfig config = RunnableConfig.builder()
        .threadId("session_001")
        .addMetadata("user_id", "user_123")
        .store(store)
        .build();
agent.invoke("查询用户信息，namespace=['users'], key='user_123'", config);
System.out.println("工具读取长期记忆示例执行完成");

```

**示例三：使用 ModelHook 管理长期记忆**

下面的示例展示了如何使用 ModelHook 在模型调用前后自动加载和保存长期记忆。

```
// 创建记忆拦截器
ModelHook memoryInterceptor = new ModelHook() {
    @Override
    public String getName() {
        return"memory_interceptor";
    }
    @Override
    public HookPosition[] getHookPositions() {
        returnnew HookPosition[] {HookPosition.BEFORE_MODEL, HookPosition.AFTER_MODEL};
    }
    @Override
    public CompletableFuture<Map<String, Object>> beforeModel(OverAllState state, RunnableConfig config) {
        // 从配置中获取用户ID
        String userId = (String) config.metadata("user_id").orElse(null);
        if (userId == null) {
            return CompletableFuture.completedFuture(Map.of());
        }
        Store store = config.store();
        // 从记忆存储中加载用户画像
        Optional<StoreItem> itemOpt = store.getItem(List.of("user_profiles"), userId);
        if (itemOpt.isPresent()) {
            Map<String, Object> profile = itemOpt.get().getValue();
            // 将用户上下文注入系统消息
            String userContext = String.format(
                    "用户信息：姓名=%s, 年龄=%s, 邮箱=%s, 偏好=%s",
                    profile.get("name"),
                    profile.get("age"),
                    profile.get("email"),
                    profile.get("preferences")
            );
            // 获取消息列表
            List<Message> messages = (List<Message>) state.value("messages").orElse(new ArrayList<>());
            List<Message> newMessages = new ArrayList<>();
            // 查找是否已存在 SystemMessage
            SystemMessage existingSystemMessage = null;
            int systemMessageIndex = -1;
            for (int i = 0; i < messages.size(); i++) {
                Message msg = messages.get(i);
                if (msg instanceof SystemMessage) {
                    existingSystemMessage = (SystemMessage) msg;
                    systemMessageIndex = i;
                    break;
                }
            }
            // 如果找到 SystemMessage，更新它；否则创建新的
            SystemMessage enhancedSystemMessage;
            if (existingSystemMessage != null) {
                // 更新现有的 SystemMessage
                enhancedSystemMessage = new SystemMessage(
                        existingSystemMessage.getText() + "\n\n" + userContext
                );
            }
            else {
                // 创建新的 SystemMessage
                enhancedSystemMessage = new SystemMessage(userContext);
            }
            // 构建新的消息列表
            if (systemMessageIndex >= 0) {
                // 如果找到了 SystemMessage，替换它
                for (int i = 0; i < messages.size(); i++) {
                    if (i == systemMessageIndex) {
                        newMessages.add(enhancedSystemMessage);
                    }
                    else {
                        newMessages.add(messages.get(i));
                    }
                }
            }
            else {
                // 如果没有找到 SystemMessage，在开头添加新的
                newMessages.add(enhancedSystemMessage);
                newMessages.addAll(messages);
            }
            return CompletableFuture.completedFuture(Map.of("messages", newMessages));
        }
        return CompletableFuture.completedFuture(Map.of());
    }
    @Override
    public CompletableFuture<Map<String, Object>> afterModel(OverAllState state, RunnableConfig config) {
        // 可以在这里实现对话后的记忆保存逻辑
        return CompletableFuture.completedFuture(Map.of());
    }
};
// 创建带有记忆拦截器的Agent
ReactAgent agent = ReactAgent.builder()
        .name("memory_agent")
        .model(chatModel)
        .hooks(memoryInterceptor)
        .saver(new MemorySaver())
        .build();
// 创建内存存储
MemoryStore memoryStore = new MemoryStore();
// 模拟数据，预先填充用户画像
Map<String, Object> profileData = new HashMap<>();
profileData.put("name", "王小明");
profileData.put("age", 28);
profileData.put("email", "wang@example.com");
profileData.put("preferences", List.of("喜欢咖啡", "喜欢阅读"));
StoreItem profileItem = StoreItem.of(List.of("user_profiles"), "user_001", profileData);
memoryStore.putItem(profileItem);
RunnableConfig config = RunnableConfig.builder()
        .threadId("session_001")
        .addMetadata("user_id", "user_001")
        .store(memoryStore)
        .build();
// Agent会自动加载用户画像信息
agent.invoke("请介绍一下我的信息。", config);
System.out.println("ModelHook管理长期记忆示例执行完成");

```

多智能体 (Multi-agent)

当单个 Agent 难以处理复杂任务时，多智能体架构允许您将任务分解为多个协同工作的专业化 Agent。Spring AI Alibaba 支持两种核心模式：

**模式一：工具调用 (Agent as a Tool)**

这是一种集中式控制流，一个 “控制器” (Controller) Agent 将其他 “子 Agent” (Sub-agent) 作为工具来调用。

子 Agent 独立执行任务并返回结果，但不直接与用户对话。

```
import com.alibaba.cloud.ai.graph.agent.tool.AgentTool;
// 1. 创建子 Agent (作家)
ReactAgent writerAgent = ReactAgent.builder()
    .name("writer_agent")
    .model(chatModel)
    .description("擅长写作，可以写文章")
    .build();
// 2. 创建主 Agent (控制器)
ReactAgent blogAgent = ReactAgent.builder()
    .name("blog_agent")
    .model(chatModel)
    .instruction("使用写作工具来完成用户的文章创作请求。")
    // 将子 Agent 封装为工具
    .tools(AgentTool.getFunctionToolCallback(writerAgent)) //
    .build();
blogAgent.invoke("帮我写一篇关于西湖的散文");

```

您还可以通过在子 Agent 上设置 `inputType` 和 `outputType`，来精细化控制 Agent 间传递的上下文结构。

**模式二：工作流编排 (Handoffs / Flow)**

这是一种去中心化控制流，控制权从一个 Agent “交接” 给下一个 Agent。框架内置了多种流程型 Agent：

*   SequentialAgent：按预定义顺序依次执行 Agent (A -> B -> C)。每个 Agent 的输出（通过 `outputKey` 指定）会传递给下一个。
    
*   ParallelAgent：将相同的输入同时发送给所有子 Agent 并行处理，然后使用 `MergeStrategy` 合并结果。
    
*   LlmRoutingAgent：使用 LLM 根据用户输入和子 Agent 的 `description`，_智能_地选择一个最合适的子 Agent 来处理请求。
    

#### 示例：LlmRoutingAgent

在路由模式中，使用大语言模型（LLM）动态决定将请求路由到哪个子 Agent。这种模式非常适合需要智能选择不同专家 Agent 的场景。LLM 会根据用户输入和子 Agent 的 `description`，智能地选择最合适的 Agent 来处理请求。

```
import com.alibaba.cloud.ai.graph.agent.flow.agent.LlmRoutingAgent;
// 创建多个专家 Agent
ReactAgent writerAgent = ReactAgent.builder()
    .name("writer_agent")
    .model(chatModel)
    .description("擅长创作各类文章，包括散文、诗歌等文学作品")
    .instruction("你是一个知名的作家，擅长写作和创作。请根据用户的提问进行回答。")
    .outputKey("writer_output")
    .build();
ReactAgent codeAgent = ReactAgent.builder()
    .name("code_agent")
    .model(chatModel)
    .description("专门处理编程相关问题，包括代码编写和调试")
    .instruction("你是一个资深的软件工程师，擅长编写和调试代码。")
    .outputKey("code_output")
    .build();
// 创建路由 Agent
LlmRoutingAgent routingAgent = LlmRoutingAgent.builder()
    .name("content_routing_agent")
    .description("根据用户需求智能路由到合适的专家Agent")
    .model(chatModel)  // 路由需要一个 LLM 来进行智能选择
    .subAgents(List.of(writerAgent, codeAgent))
    .build();
// 使用 - LLM 会自动选择最合适的 Agent
// LLM 会路由到 writerAgent
Optional<OverAllState> result1 = routingAgent.invoke("帮我写一篇关于春天的散文");
// LLM 会路由到 codeAgent
Optional<OverAllState> result2 = routingAgent.invoke("帮我写一个 Java 排序算法");

```

**模式三：Agent as Workflow Node**

在复杂的工作流场景中，可以将 `ReactAgent` 作为 Node 集成到 StateGraph 中，实现更强大的组合能力。Agent 作为 Node 可以利用其推理和工具调用能力，处理需要多步骤推理的任务。

`ReactAgent` 可以通过 `asNode()` 方法转换为可以嵌入到父 Graph 中的 Node：

```
publicclassAgentWorkflowExample {
    public StateGraph buildWorkflowWithAgent(ChatModel chatModel){
        // 创建专门的数据分析 Agent
        ReactAgent analysisAgent = ReactAgent.builder()
            .name("data_analyzer")
            .model(chatModel)
            .instruction("你是一个数据分析专家，负责分析数据并提供洞察")
            .tools(dataAnalysisTool, statisticsTool)
            .build();
        // 创建报告生成 Agent
        ReactAgent reportAgent = ReactAgent.builder()
            .name("report_generator")
            .model(chatModel)
            .instruction("你是一个报告生成专家，负责将分析结果转化为专业报告")
            .tools(formatTool, chartTool)
            .build();
        // 构建包含 Agent 的工作流
        StateGraph workflow = new StateGraph("multi_agent_workflow", keyStrategyFactory);
        // 将 Agent 作为 SubGraph Node 添加
        workflow.addNode("analysis", analysisAgent.asNode(
            true,                     // includeContents: 是否传递父图的消息历史
            false,                    // returnReasoningContents: 是否返回推理过程
            "analysis_result"         // outputKeyToParent: 输出键名
        ));
        workflow.addNode("reporting", reportAgent.asNode(
            true,
            false,
            "final_report"
        ));
        // 定义流程
        workflow.addEdge(StateGraph.START, "analysis");
        workflow.addEdge("analysis", "reporting");
        workflow.addEdge("reporting", StateGraph.END);
        return workflow;
    }
}

```

总结

Spring AI Alibaba 1.1 提供了一个从简单到复杂的完整 Agent 开发框架。它以 `ReactAgent` 为核心，通过先进的 “上下文工程” 理念，利用 Hooks 和 Interceptors 提供了强大的可控性。借助 Memory 和 Human-in-the-Loop 等生产级特性，以及灵活的多智能体编排能力，开发者可以构建出真正可靠、智能、可扩展的企业级 AI 应用。

更多内容请查看官网文档：https://java2ai.com/

[1]https://github.com/alibaba/spring-ai-alibaba/tree/main/examples/chatbot

[2]http://localhost:8080/chatui/index.html

[3]https://java2ai.com/docs/frameworks/agent-framework/tutorials/hooks

Qwen-Image，生图告别文字乱码

针对 AI 绘画文字生成不准确的普遍痛点，本方案搭载业界领先的 Qwen-Image 系列模型，提供精准的图文生成和图像编辑能力，助您轻松创作清晰美观的中英文海报、Logo 与创意图。此外，本方案还支持一键图生视频，为内容创作全面赋能。

点击阅读原文查看详情。