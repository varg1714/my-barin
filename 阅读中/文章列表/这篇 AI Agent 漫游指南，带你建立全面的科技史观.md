---
source: https://mp.weixin.qq.com/s/37SlqyDSg0FapEqJwy-0mw
create: 2025-06-10 19:02
read: false
knowledge: false
tags:
  - 大模型
---
### 

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/j3gficicyOvasVeMDmWoZ2zyN8iaSc6XWYj79H3xfgvsqK9TDxOBlcUa6W0EE5KBdxacd2Ql6QBmuhBJKIUS4PSZQ/640?wx_fmt=gif&from=appmsg)

  

作者：kong

以 OpenAI o1 与 DeepSeek R1 为代表的 "类 Agent" 模型、OpenAI DeepResearch 为代表的 “真 Agent” 模型，正在重构 AI Agent 的技术范式。Agentic Workflow 的王座还没坐热，强化学习驱动的端到端 Agent 模型训练已呼啸而来。未来趋势已指明：模型即产品，工程化 Agent 的命运将如何？一起来洞察全新的 Agent 技术范式底下的技术及演进过程，提前看到未来的模样。

欢迎乘坐 Agent 漫游列车，作为 AI Agent 的躬身入局者，我将为你讲解 AI Agent 的前世今生并推演一下未来。

### **时间线**

我们先来回顾一下基于 LLM 的 Agent 发展时间线，LLM 的实质性的起源时间只回溯到 2017 年的注意力机制的提出时间。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy5KibZgTz942zXXRBzQwrO1Hy8vlianomkq7jR2WiajnNkia1ISygPCjruDw/640?wx_fmt=png&from=appmsg)

1.  在 2017 年前，AI 的世界一片混沌，NLP 领域更是停滞在 RNN 和 LSTM 止步不前。
    
2.  《人类群星闪耀时》如果有续集，我认为 2017 年《Attention Is All You Need》的作者应当在列，论文描述的注意力机制——Transformer 架构划破了 AI 世界的第二个长夜，一个崭新的时代光速开启。
    
3.  接下来的标识性事件是 GPT-3 的诞生，代码生成场景，GitHub Copilot 重新定义了代码补全。
    
4.  基于 GPT 3.5 的 ChatGPT 把通过自然语言聊天的形态把大模型带到了普罗大众面前，超越 tiktok 成为增长最快的 app。
    
5.  GPT-4 是首个参数突破万亿的大模型，在 2023 年，GPT-4 的性能无敌，OpenAI 也放慢了继续扩大模型参数的路子，推出插件系统、GPTs 等，当年业界大井喷出大量的 LLM 应用开发框架，CoT 思维链，Agent 理念的祖师爷 ReAct 都在那时候推出，OpenAI 也把工具使用的能力训练进了模型里，推出 FunctionCall，这一年，可谓 AI agent 的白银时代。
    
6.  2024 年，Agent 在水底下快速发展，模型的预训练 Scaling Law 好像失效了，GPT-4 停滞不前，GPT-5 难产，O1 的出现宣告着训练的路径要转向了。
    
7.  2025 年是后预训 Scaling Law 开始生效的时间，蛰伏了两年多的 Agent 得以浮出水面，而模型侧也因为强化学习迎来了第二春：后训练的 Scaling Law。
    
      
    

### **AI Agent 是怎样炼成的**

AI Agent 是大模型应用的一种特定形态，在深入理解什么是 AI Agent 之前，我们先直观理解一下大模型的工作方式：文本补全。

#### **LLM 工作的核心形态：文本补全**

如下图所示，我们给 LLM 发一段文本：“下面我将要讲一个故事。在很久很久以前，有一个”，大模型会收到输入后，它会返回一段文本：“小村庄坐落在群山环换之中。村子里住着。。。（省略数百字）”，然后，结束了。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy5ibXzz6XzDuRaPSbxCMZkKH5kz4hja0H56XIrzohE9bbuiagljR1hibTwg/640?wx_fmt=png&from=appmsg)

这就是大模型一次工作的典型表现，对输入的文本进行补全（Text Completion，这是为什么 LLM 们的接口都是 completion、chat/completion 的原因）。用户输入的部份内容，称之为提示词——Prompt，大模型生成的输出的文本是生成内容——Generated Text。

整个核心形态看似简单，一次输入输出。实际上提示词与生成内容两端分别是两个巨大的领域：提示词工程与模型预训练。

通过提示词，用户可以让大模型实现各种场景的文本生成任务，例如诗歌创作、语言翻译、代码生成、广告文案生成等，而提示词本身的编写方法和质量也会影响大模型生成内容的效果，因此，如何写好提示词是一门综合性的学问。另一方面，提示词是通过自然语言来表达的，所以这也造成了大量的非 AI 科班出身的且非专业开发人员投入到了大模型应用的开发浪潮当中，这个群体形成了提示词工程的阵营，我们看到的大部份 LLM 应用侧的工作都属于该阵营。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy594ZalM9FstjCPnyhULB7tgeFIow95zKQMMTILZNiaictB4FegxhvQswg/640?wx_fmt=png&from=appmsg)

基于以上对 LLM 应用的了解，我们继续往下一站，了解什么是 AI Agent。

#### **什么是 AI Agent**

在业界一度有一个乱象，就是把所有基于大模型的聊天机器人都统称为智能体即 AI Agent。不管你是一个角色扮演的应用，或者通过流程编排出来的一个大模型工作流，还是可以自主决策来去使用工具做任务的真 Agent，这些都统称为 AI agent，但这其实是一个误区和懒惰。现在都说 2025 年是 AI Agent 的元年，我们很有必要去澄清一下 AI Agent 它到底是什么。

AI agent 是基于大模型，具备记忆能力、能够有自主推理和规划工具的使用，从而来解决问题的智能程序。即 AI Agent = 大模型 + 记忆 + 使用工具 + 自主规划。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy5nPS0gkS2ZX2Kc6bn7LzLUibZmwoQ392xpgxZSECvINf7yQEnCYQ4LwA/640?wx_fmt=png&from=appmsg)

**基于大模型**意味着可以通过自然语言去交互，所以聊天是我们使用 AI Agent 最直观感受到的交互方式。

##### **多轮对话与记忆**

**有记忆**能力就意味着他能记得跟你过往跟你聊天和互动的历史，正因为如此，你昨晚和你的 AI 伴侣聊得火热，第二天起来 TA 也不会问你，你是谁，你想干什么？

AI agent 要实现记忆能力，简单的做法就是把前序的聊天记录附在提示词里，但很快迎来新的问题，聊天记录多了，很容易就导致模型上下文爆 token 无法继续生成，随后又发展出只取最近 N 次聊天记录、只取与当前问题相关的聊天记录等等手段。

单有记忆能支持人机之间进行连续的多轮对话还不够，因为光说不练的也不能叫做 Agent。

##### **使用工具**

所以 TA 必须得**懂得用工具**。所谓的使用工具，就是去访问各种资源，调度数据接口等。例如，我们常见到的一种 AI 聊天的形态——联网搜索，你可以把它看成一种使用工具的能力，AI 把你的问题和该问题在网络上相关的一些内容加到一起去，让大模型给你生成答案。

话又说回来，能使用工具的就是 Agent 了吗？我们来比较一下元宝联网搜索的自动挡和手动挡。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy5uVYrw1YCkZ3U0d2ibuibAVAlV1hD2licCicwr9NVpDSHeJNsZicdGP1BD1A/640?wx_fmt=png&from=appmsg)

在元宝里面，你只要勾选了联网的手动挡，每次你提问他都会先联网查询再给你回答，而联网的自动挡会先判断你这个问题需不需要更多辅助它解决的信息，需要了再去联网搜索，不需要他就直接回答。同样是使用工具，但手动挡表现出来的是固定的工作模式，而自动挡做法是 AI agent 的模式，它有自己的**自主的规划和反思**过程，这是 AI Agent 的另一个重要的特征。这个容后详述。

###### **Function Call**

回到工具，大模型是怎样使用工具的呢？我们都知道，大模型是一个文本模型，它只能输出文本，所以，实际上所谓的使用工具，只是大模型在文本里说明要使用什么工具，LLM 的应用程序解释这段文本找到使用工具的信息，按照大模型的吩附来执行工具的调用，如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy5fNE65oKJ9lVichZuYCGxgHgicONVw4iaSYFfLUKCOmobkHqkBfa39MRWA/640?wx_fmt=png&from=appmsg)

上图中，我们在给大模型的输入的提示词内容包括：

1.  可用的工具说明，包括工具的功能、接受的参数明细等。
    
2.  工具的调用规范及示例，通过对工具调用的规范进行详细说明，并使用 fewshot 的技术来给大模型学习一些例子。
    
3.  用户问题，最后是附上用户的提问。
    

大模型在回复的时候，会按照提示词中的工具调用规范返回实际的工具使用例子，在上图中是一串 json 格式的配置数据，表达了要调用 search_web 这个工具，参数有 query 和 limit 两个。

后来，这种教大模型如何返回工具使用命令的工作，被 OpenAI 率先预训练到模型里面去了，并把这个功能叫 Function Call，训练到模型去即意味着不需要再通过提示词指导大模型使用工具了，而只需要告知大模型你有什么工具可用即可，在 OpenAI 的接口中，通过 tools 指定可用的工具集。

再后来的事大家都知道了，主流的大模型都先后效仿 openAI 支持了 function call。

###### **MCP**

MCP（Model Context Protocol) 是由 Anthropic(Claude 母公司) 在 2024 年底提出的一种大模型上下文模议，目的是让 Agent 能够更方便地发现和使用来自各处的工具，让 Agent 能做的事情更多。最早的落地场景是在 Cluade 的桌面端中使用，Claude 通过 MCP 协议对用户计算机的文件进行读写和对用户的电脑进行操作。

MCP 随着 AI Agent 的出圈也飞速流行起来，当前已然是一片不 MCP 无 Agent 的态势，国内外大模型厂纷纷下场支持 MCP，MCP 成了事实上的 Agent 工具使用标准。

关于 MCP 与大模型 Function Call 的关系， 经常会被误读，说 MCP 是替代 Function Call 的。**但实际上，Function Call 和 MCP 两者是不同层面的东西，甚至反过来说，是紧密配合的。如果 一个模型不具备 Function Call 或等价的能力，那它就用不了 MCP。**

Function Call 是大模型返回调用工具指令的能力，MCP 是 Agent 在工程侧的程序具体执行调用工具的手段，一个是说，一个是做。

在有 MCP 之前，Agent 收到大模型的 Function Call 指令后通过各种方法去调用外部的各种资源和服务的，如要自己实现读写文件，查数据库，调搜索接口等等，这些方法可以千差万别，开发过程长，成本高。

而 MCP 的出现，统一了工程侧调用工具的规范，它服务的厂商按照 MCP Server 的标准提供服务，Agent 的程序只需要统一使用 call_tool 这个 MCP Client 的功能来执行调用即可，一下子节省了大量的工具适配的工作。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy51r3iaxcia3PsvHCEa6lVylMexPKUjZ1wWicwmR7MDU4heW5cJloFwZGWA/640?wx_fmt=png&from=appmsg)

所以，MCP 不是来代替 Function Call 的，而是帮工程侧调用外部工具提效的。Function Call 是使用工具的基石能力，MCP 打开了 AI Agent 连接世界的大门，两者强强联合，才是提效的真相。

##### **自主规划与反思**

上面说过，只会无差别的使用工具，是不经过事先思考的行为，这种 LLM 应用不能被称之为 AI Agent。 自主规划和反思甚至自我批评，是 AI Agent 模拟人类工作方式的体现，也是 AI Agent 的核心要素。

###### **规划：思维链（CoT）**

思维链（Chain of Thought，简称 CoT；Wei 等人 2022 年提出）已成为提升大模型处理复杂任务性能的事实上的标准提示词技术。人们通过引导模型 "逐步思考"，将任务拆解为多个更小、更简单的子步骤，从而提供模型的输出性能。CoT 不仅将庞大任务转化为可管理的分步流程，在 DeepSeek R1 这类推理模型中，还为理解模型的推理过程提供了透明化的解读路径。

除了思维链，类似的思路还有思维树（Tree of Thoughts， ToT）和思维图（Graph of Thoughts，GoT）。它们都对 CoT 进行了扩展，在特定的应用场景均有显著的提升。但是实际应用中，CoT 是绝对的主流。

###### **反思：ReAct**

反思能力能让 Agent 具备迭代出可用答案的可能性。Agent 通常不止一次调用 LLM 和工具，每一次采取行动调用工具后，都需要经过反思来确定是否做好了，不够好接下来该怎么做。

ReAct（Reasoing Acting， 由 Yao 在 2023 年提出）思考框架，它指导 AI Agent 通过思考、行动、观察的循环来实成任务。Agent 接到任务后的工作流程大致如下：

1、 思考 (thought)，要解决该问题，下一步需要采取什么行动。

2、 行动 (action)，大模型输出行动指令，让 Agent 调用外部工具。

3、 观察 (observation)，把工具执行的结果给大模型进行观察。

4.1、回答 (answer)，如果工具执行的结果已能得到答案，组织语言回答。

4.2、如果目前得到的信息仍无法作答，进入下一次循环，继续思考使用工具。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy5s1YBqmQuicaHWuo1xM0EpFAZHY6HaWEWHnrpe0ByiaGj0bibdJxecLTNw/640?wx_fmt=png&from=appmsg)

看起来是不是很像咱们人类的 PDCA（Plan Do Check Act）的翻版？

ReAct 模式是当下 AI Agent 领域事实上的工作模式，包括基于 OpenAI Function Call 实现的 Agent 在内的背后也是同样的工作模式。只不过，使用内置的 Function Call 的方式，不需要额外提供提示词来指导模型行动罢了。

#### 为什么 Agent 不 Work

AI Agent 在大众看到之前已经发展了两年多，直到最近 Manus 的爆火才被出现在大家面前，根本原因是，Agent 的可靠性不足，上限较低。所以一直还摆不上台面，仅在有限的场景迭代和落地。

实现一个 Agent 不难，有开发经验的同学，通过学习在一两天内可以开发出一个可以运行的 Agent，但要做一个可用的 Agent，则还需要大量的工作。

判断一个 Agent 是否可用，主要取决于具体场景的错误容忍度和受众的介入程度。以 AI 编程为例，开发者对 Agent 生成代码的预期是 “规模不大的需求，代码生成还不错，会有问题，但可以通过反复沟通去修正，最终达到相对可接受的结果”。所以，Vibe coding 这个场景火了，大量不懂代码的开发者诞生了。Deep Research 所关注的研报场景同理。

所以，当下大家能看到的生产级别的 Agent，基本上都有这两个特征：复杂度与规模较低、容错水平高。

影响 Agent 在大规模复杂问题上的性能因素是幻觉和记忆管理的挑战。

##### **一定是幻觉**

大模型是一个概率模型，它生成的内容一定的概率是错误的，即我们常说的**幻觉。**

Agent 执行一次任务，通常需要组合多次大模型的调用来完成工作，在总体的结果成功率上比单次的大模型调用会更加低。例如：假设平均单次调成大模型生成内容的正确率在 90%，那 4 次组合调用后，正确率直接下降到 60-70% 。

##### **记忆管理的难**

当前基于大语言模型的 Agent 普遍面临 "记忆困境"，这种困境源于大模型自身的无状态特性与人类认知过程中持续演进的记忆机制之间的本质差异。传统采用简单对话历史堆砌的 "伪记忆" 实现方式，在应对需要长期记忆保持、复杂知识关联和动态经验积累的场景时，暴露出一系列结构性矛盾。

###### **上下文窗口的限制**

当前主流大模型的上下文处理能力受限于固定长度的窗口机制（如 GPT-4 的 32k tokens）。这种物理限制导致对话轮次或任务复杂度超过窗口容量时，必然发生历史信息截断，造成关键记忆丢失；其次，随着上下文长度增加，模型处理效率呈指数级下降。这种矛盾在需要长期任务追踪的场景（如连续多日项目管理）中尤为突出。

大模型厂商不断推出支持更大 size 上下文的模型，截止发稿为止，最大的上下文是 Meta 的 Llama scout 1000 万 token。

###### **超长上下文的注意力有效性衰减**

尽管上下的尺寸越来越大，甚至能塞下全集的哈里波特了，但是超长上下文注意力的准确性又成了另一个问题。

Transformer 架构的自注意力机制虽然赋予了模型强大的上下文关联能力，但其计算复杂度 O(n²) 的特性导致随着上下文长度扩展，有效注意力的分布呈现显著稀释效应。根据 ICLR 2023 的研究成果，在 16k tokens 的上下文长度下，模型对前 20% 输入内容的注意力权重占比超过 65%，而对后 20% 内容的注意力权重不足 8%。这种 "近因偏好" 现象使得早期关键信息容易被后续内容覆盖，导致记忆保持的时序稳定性问题。更严重的是，当处理超长文档（如百页技术手册）时，模型可能陷入 "注意力涣散" 状态，出现关键信息漏读或误读。

Google 的 BigBird 和 DeepSeek 的 NSA(Native Sparse Attention) 都在致力于解决这个问题。

**相关记忆的准召问题**

既然暴力的强塞所有的聊天记录不行，那就换一种思路吧，只取跟当前问题有关联的聊天记录总可以了吧？我们把聊天记录存在向量数据库中，通过向量检查召回关联的内容，实现按需注入历史。

然而，向量数据库的召回也是一个庞大复杂的工程（RAG 中的 R），召回数据的准确与否，直接决定了大模型回答的质量。为了提升准召率，RAG 一路发展到基于知识图谱的 RAG，又到了今天的 Agentic RAG，仍然没有到头。

### **有办法！**

方法总比问题多嘛，既然知道 agent 面临着怎样的挑战，就给出针对性的解决方案吧。为了提升 agent 的性能，业界提出了各种解决方案，总结起来有 3 大类。

● 引入 workflow，使用固化的工作流程来提升确定性，但同时牺牲掉灵活性。

● 在 ReAct 框架的基础上做工程侧的极致优化

● 引入多 agent，效仿人类团队协作，突破单 agent 的极限，发挥群集智慧。

#### **workflow 的第二春**

AI Agent 不稳定？那我们来固化工作流程，让 AI 在必要的时候工作就好？这个解题思路引出了 AI workflow 的技术形态。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy579moicBQhln8bt5C4Ds8glLM7FWiabLCRXnlBuL5vMlrHKWx8oiaxXAmg/640?wx_fmt=png&from=appmsg)

从技术演进视角来看，Workflow 本质上是将低代码开发框架与 LLM 相结合的产物，旧瓶装新酒。其在大模型时代的流行主要源于两个关键因素：首先，当前开发范式已从传统编码转向提示词工程，开发者需要高频迭代提示词而非底层代码；其次，可视化流程编排显著降低了调试门槛，使非技术背景人员也能通过直观界面完成 AI 能力集成。

现有 Workflow 更多是业务逻辑的标准化封装，AI 仅作为模块化组件服务于特定环节。这种架构虽提升了开发效率，但也存在本质局限——既无法实现智能体（Agent）的自主推理能力，也难以支撑复杂场景的端到端智能化。

简单来说，workflow 本身不是 AI Agent，但基于 workflow 实现的功能可又作为 Agent 的工具，作为 Agent 的有机组成部份。

#### **Beyond ReAct****Agent**

之前说过 ReAct Agent 是当下主流 Agent 的思考与行动框架，但 ReAct 本身也有着很多的缺点：

1.  走一步看一步，缺乏全盘规划。每次的思考与决策需要依赖上一次工具的输出结果。
    
2.  串行调度工具，每次工具调用都跟随着一次 LLM 的调用，没能灵活高效的对工具的调度进行优化。
    
3.  所有工具的执行结果，都会追加到大模型的上下文中供观察使用，经过多次的工具调用来回后，很容易就触发上下文限制，任务以失败告终。
    

针对这些缺点，业界的优化方式也是五花八门，以下举一些代表性的例子：

##### **plan and execute**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy5l0taXSONMPaPo7xeDtt4yEn1jQo2ES5Qy3KB40QgyNOtHdDHEqHJRg/640?wx_fmt=png&from=appmsg)

该思路主要受到 Plan-and-Solve 论文和 Baby-AGI 项目的启发，其核心工作流程包含三个阶段：

● 规划阶段 ：首先生成一个全盘的多步骤的详细行动计划

● 执行阶段 ：按顺序执行每个计划步骤，返回结果

● 重规划阶段：根据执行结果动态调整计划或返回

这种模式引入了全盘规划，且子任务的执行分拆到 Single-Task Agent 上执行，避免了 Token 在同一个 LLM 会话上下文中堆积，降低爆 Token 的可能性。

manus 的 Agent 显然是借鉴了这种 Agent，先生成任务的清单，再对着清单逐个执行，但似乎并没有看到 manus 有重新规划这个步骤。

##### **ReWoo**

ReWOO（ Reasoning WithOut Observation ）是一种创新的增强语言模型（ALM）框架，旨在通过 模块化设计 显著提升多步推理任务的效率与性能。传统 ALM（如 ReAct）依赖交替的 “推理 - 工具调用 - 观察” 流程，导致大量上下文重复输入和计算资源浪费。ReWOO 突破性地将任务分解为三个独立模块：

● Planner（规划器） ：基于大型语言模型（LLM）的推理能力，预先生成任务蓝图，规划多步推理路径（如调用工具的顺序与逻辑），无需等待工具实时反馈。

● Worker（执行器） ：根据蓝图并行调用外部工具（如搜索引擎、计算器、数据库），高效收集证据。

● Solver（求解器） ：综合规划与证据生成最终答案，具备纠错与总结能力。

ReWOO 最显著的特点是拥有一个独立的 Solver（求解器）模块，专门负责综合规划结果和工具执行证据，生成最终答案。在 worker 的执行过程中， ReWOO 不去观察（Observation）工具返回的结果，可以减少 token 的使用及调用 LLM 的次数。

ReWOO 与 Plan and Execute 相比有两个差异：

● worker 的任务执行更多是工具执行，不需要额外的 LLM 来驱动。

● 没有重新规划的过程。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy5fBUibia7dCEm9N6gsMlrbOr2cd1ZrpoDVyyPNhBia7uFK5exaQ9tQKAgg/640?wx_fmt=png&from=appmsg)

##### **LLm Compiler**

LLMCompiler 专为优化大语言模型（LLM）的多工具协作效率而设计的框架。针对传统方法（如 ReAct）因顺序执行函数调用导致的延迟高、成本大、准确率受限等问题，LLMCompiler 创新性地引入编译器式任务编排，通过并行化与动态规划显著提升 LLM 在复杂任务中的表现。

其核心架构：

● 智能规划器（Planner）：将用户查询解析为带依赖关系的任务 DAG，识别可并行执行的函数调用（如并行的网络搜索与数学计算）。

● 动态调度器（Task Fetching Unit）：实时替换占位变量、分发独立任务，最大化并行资源利用率。

● 异步执行器（Executor）：通过工具 API 并发执行任务，支持自定义工具（如搜索引擎、计算器、API 代理）。

LLMCompiler 同样是提前做 DAG 规划，它通过任务依赖关系来对任务进行并行调度，还可以根据结果进行重新规则。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy5qthcsdrdMxGvuaXqia0w7bbGGQ6vX9N6fVMtqfwfrd49KibRyxAMIibAQ/640?wx_fmt=png&from=appmsg)

#### **多 Agent**

人类社会有一句话 “独行快，众行远”，指的是如果要走得更远，需要团队合作。在 Agent 的世界，单个 Agent 在简单任务方面的表达已经不错，但复杂的以及上规模的任务中的表现却乏善可陈。于是我们不由得去向人类的协同方式学习，让 Agent 组成团队，复刻人类的协同方式，看是否能够提升性能。

##### **多 Agent 的形态**

根据多 Agent 的应用场景，我把多 Agent 的产品形态分为社会协同模拟型与任务导向型 。

###### **社会协同模拟型**

类如 “斯坦福小镇” 这一种 agent 社会化实验性的形态，称为社会协同模型型，这类产品不设定具体的任务让 Agent 来实现，而是提供了一个开放性的运行环境，让 Agent 自发地去协同和产生可能的“化学反应”，用于对 Agent 社会化协同的学习与研究。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy5FZ1gvZq6xKGMPumJk8ThbOeV5s1nNdVIg4tsic7edqCDRu2O6nGFSxQ/640?wx_fmt=png&from=appmsg)

###### **任务导向型**

另一种多 agent 的形态是目的性很明确的，有清晰的目标和标准的操作流程（SOP)，典型的代表如软件开发过程、较大篇幅的内容（如论文、小说）等的创作。

MetaGPT 是此类型多 Agent 的代表框架，它通过拆解软件开发的标准流程，为每个过程设定不同的角色来完成对应的任务，最终实现一个软件的开完任务。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy5ljbibYEFJiaUVGUAIB5WhByibev9WndLEAuxPZuyPIh8gvdYns2RCibmUg/640?wx_fmt=png&from=appmsg)

##### **开发框架**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy5aAI7micqTNicA4GtqJibrGl2FNfnMXxbtheibiaqc3gsicWfvF5dKq9t4RIg/640?wx_fmt=png&from=appmsg)

1.  MetaGPT：基于多智能体协作的软件开发框架，通过模拟软件公司角色分工（产品经理 / 工程师等），将标准操作程序（SOP）编码为智能体协作流程，支持从需求分析到代码生成的全生命周期自动化开发，尤其擅长结构化输出文档与代码。
    
2.  AutoGen：微软推出的多智能体对话框架，支持定制化代理角色与自然语言交互，通过模块化设计简化复杂任务编排，可无缝集成 LLM 和工具链，其核心优势在于实现人机混合协作与自动化工作流，特别适合需动态决策的场景。
    
3.  CrewAI：开源协作型智能体框架，强调角色扮演与团队化任务管理，支持自定义代理角色、任务委派及流程控制（顺序 / 层级模式），提供工具集成与知识沉淀机制，适合构建需要明确分工的多代理协作系统（如市场分析 / 项目管理）。
    
4.  Swarm：OpenAI 实验性轻量级框架，聚焦智能体间的动态任务交接（Handoffs），通过函数调用实现执行权转移，保持高度可控性与透明性，与 Chat Completions API 深度整合，适合需细粒度控制的小规模多代理交互场景。
    

当然，langchain 和 langgraph 这类框架同样是可以用于搭建多 agent 的，没把它们列在上面仅仅是因为这两个框架它的普适性更广，不是专为多 agent 而专门提供的。

##### **协同架构**

langgraph 把多 Agent 的协同架构做了一下汇总，除了自定义架构，大致有以下几种类型：

1.  Network（网状），网状架构允许每个 Agent 间互相通讯，该架构的自由度高，但可控性差，适用于社会协同模拟型的 Agent 形态。
    
2.  supervisor（监督者），该架构有一个管理者 Agent，其他所有 Agent 之间不能直接沟通，只能与管理者 Agent 进行沟通。这种架构适用于对任务导向型的多 Agent 形态，可控性较高，但管理者 Agent 的智能程度会成为整个多 Agent 网络的瓶颈。
    

a. supervisor 的结构看起来还跟单 Agent 的结构很相似，实际上，把非管理者 Agent 看成一个个工具的话，它就等同于一个单 Agent，即图中的 supervisor(as tools) 的结构。

b. 所以，多 Agent 并不神秘，你在以前做单 Agent 的时候极有可能就已经实现过 as tools 这种 supervisor 架构的多 Agent 应用了。上面 "plan and execute" 中描述的形态也可以视为一种多 Agent。

1.  Hierarchial（层级监督者），层级监督者是由多个监督者网络进行堆叠而成的，如果把监督者网络看成一个小组由一个组长带领多个组员，那层级监督者网络则更大的的组织，例如是一个中心，甚至是部门，业务线等。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy59UArwuqxaFpDroK5NF9qCRSDXLSMHiaF1zKicRSlsFko6sgdcG10k7yQ/640?wx_fmt=png&from=appmsg)

##### **Agentic Workflow**

agentic workflow 最早由吴恩达提出。简而言之，它的目标是解决复杂任务，通过分解任务、多角色 Agent 协同、迭代改进的手段来实现。它有以下四大机制：

● 工具调用 (Tool Use)

● 多 Agent 协作（Multi-agent）

● 规划能力（Planning）

● 反思机制（Reflection）

光看上面的描述，定义是相当的模糊的，我们拿上文中出现过的 LLM 应用和 Agent 来对比一下，以便进一步理解 agentic workflow。

###### **与 “plan and execute“ agent 的区别**

上面讲的 Plan and Execute 形态的 Agent 看起来就具备” 分解任务”、 “子任务执行 Agent”、“迭代改进” 等等环节，其中子任务执行 Agent 是一个通用的执行者，负责遍历任务并执行。

而 Agentic workflow 对任务执行的要求是由不同角色的 Agent 来执行不同性质的任务，哪个角色应该执行什么任务。

所以，如果把 plan and execute 模式升级一下，定义多个特定职能的 Agent 作为子任务的执行者，有针对性的选择任务来执行，可以得到近似 agentic workflow 的效果。

###### **与 workflow + LLM 的区别**

它和 “workflow 的第二春” 中说的 workflow + LLM 又有什么区别呢？从几个维度来对比：

**1). 动态规划能力**

Agentic Workflow：通过 AI Agent 的推理能力动态分解复杂任务（任务分解模式），并根据环境反馈调整执行路径。

Workflow + LLM：LLM 仅作为静态模块嵌入预定义流程。

**2). 自我迭代优化**

Agentic Workflow：引入反思模式（Reflection），通过执行结果评估和策略校准形成闭环。

Workflow + LLM：缺乏反馈循环，输出质量依赖单次提示效果，无法自我优化。

**3). 执行主体性质**

Agentic Workflow：以 AI Agent 为核心，具备长期记忆（如向量数据库存储用户画像）和工具调用权限（如 API、搜索引擎），形成类人认知架构。

Workflow + LLM：LLM 作为流程中的 “工具人”，仅处理特定环节（如文本生成），无自主决策权。

**4). 任务协作模式**

Agentic Workflow：支持多 Agent 协同（如数据分析 Agent 与优惠优化 Agent 联动），通过信息传递形成集体智能。

Workflow + LLM：流程由人工预先编排，各模块独立运行，缺乏动态协作。

**5). 小结**

Agentic Workflow 是由 AI Agent 集体动态生成并可随机变动的协作流程，而 workflow + LLM 中的 workflow 是一种由开发者定义的静态工作流。

###### **示例分析**

下图所描述的是一个通过 CrewAI 实现的多 agent 智能化的客户优惠推荐系统。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy5w5RtZhaQNtpGSrvEKDJcYPNF9k0QN0wF1PSia0wPEyU1LaOtScwx3jg/640?wx_fmt=png&from=appmsg)

蓝色部份是定义了一种工作流程及每个节点的任务：

1.  提取购买记录：基于用户 ID 和时间范围查询数据。
    
2.  匹配最优优惠：通过 SQL 连接（JOIN）购买记录与优惠表，按折扣排序。
    
3.  生成通知文案：整合优惠信息，添加表情符号，生成吸引人的消息。
    

绿色部份是定义了三种不同职能的 Agent：

1.  购买历史分析 Agent：编写 SQL 查询客户购买记录。
    
2.  优惠管理 Agent：结合购买历史与优惠表，筛选最优折扣。
    
3.  创意文案 Agent：生成个性化优惠通知。
    

工作流程：CrewAI 框架协调 Agent 们执行任务，输出最终优惠通知。

CrewAI 在任务的调度模式上有两种，一种顺序执行 (sequential)，一种是层级模式 (hierarchical)，后者由一个管理者 LLM 来动态调度执行。

窃以为 hierarchical 模式才是真正意义上的 agentic workflow，因为工作流是动态的，可通过反思机制进行实时调整的，是由管理者 LLM 来自主决定的。而顺序执行的模式，和 workflow + LLM 的模型没有本质的区别。

##### **Why Do Multi-Agent LLM Systems Fail?**

多 Agent 看起来很美，但在实际的落地过程却也有一地鸡毛的时候，加州大学伯克利分校等机构经过研究发表的《Why Do Multi-agent LLM Systems Fail》的论文指出了多 Agent 架构失败的原因：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy5rpFsHzeiaEB5Ds4cCiaPJ0jFXpibKPrRwibSAme6K9rYjzT0jI9XFavwJw/640?wx_fmt=png&from=appmsg)

1.  系统设计与规范问题（占 37.2%）
    

核心问题：架构设计缺陷、角色定义模糊、对话流程管理不当。

● 违反任务规范：智能体未遵循任务约束

● 角色越权：智能体超出职责范围（如 CPO 擅自定义产品愿景）。

● 步骤重复：冗余步骤导致效率低下。

● 对话历史丢失：上下文截断引发逻辑断裂。

● 终止条件不明确：无法判断任务何时完成。

1.  智能体间协作错位（占 31.4%）
    

核心问题：沟通机制低效、信息共享不足、协作流程失控。

● 对话重置：意外重启对话导致进展丢失。

● 信息隐瞒：关键数据未共享（如手机代理未告知 API 格式要求）。

● 任务偏离：讨论偏离核心目标（如 32% 的任务因跑题失败）。

● 推理 - 行动不匹配：逻辑推理与执行行为矛盾。

1.  任务验证与终止问题（占 31.4%）
    

核心问题：验证机制缺失或低效、过早终止任务。

● 过早终止：未完成必要步骤即结束（如棋类游戏未验证规则）。

● 验证不完整：仅检查表面问题（如代码编译通过但功能错误）。

● 错误验证：验证逻辑存在缺陷（如接受非法棋步输入）。

从智能体间协作错位中可以看到，多 agent 不仅复刻了人类协同的形态，还把人与人沟通的坏毛病也学习了，会隐瞒，跑题和知行不一。

### **中场战事，推理 “类 Agent“的崛起**

上面工程侧为了 Agent 输出更好的性能，想尽了办法极致压榨。模型侧也没闲着，也一直在探寻着新的 Scaling Law。

OpenAI 推出了推理模型 O1，它的工作方式是在输出内容前先进行一次内部思考 (推理)，然后再基于思考的结论来组织回答。这种分段式的生成像极了 agent 的工作方式，所以，我对 O1 的第一反应是 openAI 搞了个推理的 agent？大模型 Scaling Law 到头了，改搞工程 agent 了？后来看到技术实现才得知 O1 是强化学习的产物，O1 仍然是一个模型，但它像 agent 一样工作的模式以致我在后来把它们称为 "类 agent" 模型。

#### **犹抱琵琶半遮脸的 O1**

O1 刚出来的时候，推理的过程是完全不可见的，一个 Loading 转了几分钟看不到里面发生了什么。OpenAI 是这样解释原因的：

1.  技术权衡：思维链的忠实性和可读性是监控模型推理过程的前提，但若在思维链上加入政策合规性或用户偏好的训练，会破坏其有效性。因此，OpenAI 选择不向用户展示原始思维链，以避免潜在的干扰。
    
2.  竞争优势：隐藏推理细节可保护核心技术不被竞争对手模仿，尤其是在模型逻辑推理能力显著超越同行的背景下。
    
3.  用户体验优化：原始思维链可能包含冗长且复杂的中间步骤，直接展示会影响交互效率。OpenAI 转而提供模型生成的思维链摘要，以更简洁的方式呈现推理结果。
    

#### **掀桌子的 DeepSeek R1**

DeepSeek 是配得上伟大这样的赞誉的。

DeepSeek R1 以更高的性能、低一个数量级的成本、开源的方式打脸了 O1，掀翻了桌子。R1 发布即公开了推理过程思维链的全部内容。DeepSeek 成了真正的 “OpenAI”。

DeepSeek 公开了 R1 的训练技术细节：

● R1-Zero 版本完全摒弃监督微调，通过多目标强化学习（创新的 GRPO 算法）整合准确性、推理速度与资源消耗指标。其中 GRPO 算法可以降低对标注数据的依赖，大大降低了训练成本。

● 但由于 R1-Zero 存在思维链的可读性问题，在 R1 的正式版的训练时，分拆成了两次的 SFT+RL 的步骤：

○ 加入了一些冷启动数据（思维链内容）对 V3 进行有监督微调，再强化学习得到较好的思维链可读效果；

○ 基于上一个 Checkpoint 模型生成 60 万条思维链内容再加上 20 万条生成的的示例数据进行监督微调，最后通过强化学习进行对齐得到 R1。

过程如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy5EYiawu251CXFS4UdPNYmj1Uib6fppO8WH1SeoVXYRc8aMicibXtXXGJNTQ/640?wx_fmt=png&from=appmsg)

#### **强化学习是后训练的 Scaling Law**

如果抛开思维链的可读性不谈，R1-Zero 已经是一个高性能的推理模型，在 Zero 的训练细节上我们看到只需要强化学习就够了。R1-Zero 向我们传递了一个最重要的信息：有针对性的强化学习训练的效果可能优于单纯增加大模型参数量做预训练的效果，这也是 OpenAI O1 背后的秘密。OpenAI 看起来已经放弃了更大规模参数预训练模型的路子，而全面转向了后训练 + 强化学习，强化学习是新的 Scaling Law。

强化学习，它不算是一种新技术了，它原理是通过生成结果对模型进行的奖励和惩罚反馈，让模型在无数次的生成和反馈中调整和优化并找到最有效的工作方式，而不需要教模型怎么做。

O1 首先验证了新的训练路径，R1 把全部的细节公诸于众，一时间，强化学习训练成了大模型厂商们的 Next。Claude sonnet 3.7 跟上了节奏推出推理版，并针对复杂的代码问题进行了强化学习，在生成代码方面性能较 sonnet 3.5 有显著提升；openAI 推出的 DeepResearch 就是基于 O3 端到端训练的 Agent 模型。

## **产品的 R1“后遗症 “**

DeepSeek R1 在 2025 年的春节期间爆火出圈，成了国民级的 AI 应用。R1 的交互简单朴素，先是输出一大段思考过程，再生成最终的答案，输出推理的过程让用户避免了漫长的等待，在正式答案出来之前，阅读一下推理过程也是一件有意思的事。

R1 的产品交互也瞬间成为了教科书级别的范例。它的两阶段输出的形态正快速统一 Agent 们的输出行为。

##### **R1 前 Agent 输出招式**

Agent 不像 LLM，能快速地开始输出答案，Agent 通常有一系列的中间工作步骤，到最后一步才会输出给用户的答案，而这中间会有颇长的一段等待时间，为了缓解用户在等待过程的焦虑和优化等待体现，Agent 们都很努力在尝试把中间过程也通过各种方式输出给用户：

例如 ChatGPT 是这样的：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy5cKNshKKPkV54mWTTSr3XoqTjPficT0esaDWcCBe1kj8JxNhjicwCWRnA/640?wx_fmt=png&from=appmsg)

dify 是这样的：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy5IumRqhmqSV4pgxqZABtA5fYP0nicnN1N33hCGa6lcc4pSWgEhyFia25w/640?wx_fmt=png&from=appmsg)

我们的 FoT Agent 是这样的：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy5ecdIEOBh5ZFZGakW1P8nPiaSUicynFXAw971o0ZgDyXYNm5vbxRibAS1w/640?wx_fmt=png&from=appmsg)

然而，这些努力并没有什么作用，Agent 的用户们对这些输出的中间过程并不买单，抱怨看不懂，出结果又慢。

##### **R1 后的统一 “深度思考”**

R1 出来后，Agent 产品们除了在模型层面光速接入 DeepSeek 之外，在产品交互也是象素级的致敬着 R1。例如，我们的媒资助手 Agent 是一个基于 DeepSeek V3 的 ReAct Agent，它把 ReAct 每一步思考（Thought）的过程组装起来，伪装成深度思考的过程，看起来毫无违和感：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy5Tpic6YKYMFkNyzzhbHSO7W2M7aciax5qicSs56Ga7w0cNPRZuHLxBZ7lQ/640?wx_fmt=png&from=appmsg)

还有微信读书的 AI 问书、微信输入法的问 AI，底层的架构是基于小 size 的 QWen 模型做了 SFT 的 Agent + Deepseek R1 做最终解读，而在交互层，也是把 Agent 的工作过程和 R1 的思考融合呈现到深度思考的内容里了：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy53smWhKka6rhGF59IswBNiad9n0iaJoHYbuhHSAhwFMHMyjkeQjP2HsGg/640?wx_fmt=png&from=appmsg)

不再有花哨的 loading 和中间步骤的结构化呈现过程，只剩下朴实无华的 “深度思考” 样式的过程文本，也貌似让原来挑剔无比的用户满意了，感谢伟大的 DeepSeek！端的是一个大道至简，大巧不工啊哈哈。

### **下半场：模型即产品与 Agent 社会化协同**

我把 OpenAI 的 Deep Research 问世看作 AI Agent 下半场开始的标记性事件。Agent 正式进入模型内化的新阶段。沿着中场战事的推理 “类 Agent” 模型同样的进化路子，Deep Research 基于 O3 通过端到端的强化学习得到了一个 "真. Agent" 模型。

#### **模型即产品**

Deep Research 这个 "真. Agent" 有两个特点：

1.  **端到端训练**，就是它的训练是全链路的，对于做研报这个场景，从拿到问题、使用网络搜索工具、多轮验证重做到最终输出完整的研报的整个链路都在训练范围内。它不再像过去只让模型针对问题只做一次的文本输出。
    
2.  **Agent 模型**，对，Deep Research 的工作形式是一个 Agent，但技术上它是以一个模型出现的。在此之前，我们基于常规的 LLM 也可以做 Deep Research 这类型的工作，那就是写代码开发一个 Agent（大家可以看到现在有很多开源版的 Deep Research），这需要在工程侧来发力。但现在，OpenAI 的 Deep Research 告诉大家，原来工程上要做的事情现在不需要了，我们只需要通过强化学习就可以让模型本身掌握原来要用工程来控制的工作方式，同时还能达到更高的质量。即，工程复杂度没了，效果还更好了。
    

对比一下 O1 和 Deep Research:

● O1 推理模通过强化训练 “推理” 能力，推理能力得到了质的飞跃

● Deep Research 通过强化训练 “做研报” 的过程（包括使用搜索工具）和质量得到了一个做高质量研报的 Agent。

嗯，AI Agent 下半场的玩法变了：你想要什么样的 Agent，通过强化学习训练一个 Agent 模型，而不一定要通过编写工程代码来实现它，而这个 Agent 模型就是一个产品。这就是最近流行起来的一个说法：模型即产品。说的是，未来针对场景化的产品需求，可以基于大模型通过强化学习对场景进行训练，最终交付一个 Agent 模型作为产品，不再区分什么模型层，应用层，而是**模应一体**了。就在前两周，OpenAI 的 O3 也正式发布，O3 表现出来的则是一个比 Deep Research 更通用的 Agent 模型。这进一步指明了 Agent 模型化、模应一体化的道路。

#### **工程化 Agent 的生存空间**

如果 AI Agent 的下半场是面向场景的端到端 Agent 模型的战场，那原来通过工程化手段做的 Agent 是否还有生存空间呢？答案是确定的，在接下来的一段时间内（至少两年），三种形态的 Agent 会持续共存：

1.  纯工程 Agent，即由提示词工程加代码实现 Agent，在产品的 MVP 阶段用于快速验证产品，或产品流量不大，对 Token 成本不敏感的场景，适合用这种方式落地。它的实现门槛低，包括技术实现和成本都一样，甚至通过当下流行的可视化 Agent 搭建平台，不用写代码就可以快速搭建起来。
    
2.  SFT Agent，指针对 Agent 的行为（包括但不限规划和反思能力等）进行了有监督微调——目的是让指令跟随相对更稳定、节省提示词成本。实际上，节省提示词成本是做 SFT Agent 的最大的动机，相比起提示词 token 成本的下降，微调带来的指令跟随稳定性的提升可能没那么显著，这也是吴恩达一直说绝大多数 Agent 应用都能通过提示词来解决的原因。所以，SFT Agent 较为适用于大流量但工具需要支持动态添加的场景。
    
3.  端到端 Agent 模型，即针对垂直场景，通过端到端强化学习进行后训练的模型。它适用于大流量且需求明确垂直的场景。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy5ljPunOg4eb0V1y3GkV2mmmKuameia5r7dktbmSibTdxDXO3RFQm4iceLA/640?wx_fmt=png&from=appmsg)

Agent 才刚刚进入大众的视野，在技术和生态侧，随着 MCP 和 A2A 等协议的成熟及智能体生态的发展，Agent 的进化会进一步加速，有更多的可能性在等待着我们。

#### **Agent 的社会化协同**

以及 A2A 为代表的 Agent 间协同协议拉开了 Agent 社会化协同的大幕。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasD6InoF8Vp9V2ibI5B7fIy5PqW6txZicibMPSe8iaPkAYxvoiaur5eMt3NcufvyFygPu45rBLvf5brt9A/640?wx_fmt=png&from=appmsg)

之前我们提的多 agent 和 agentic workflo 中的 agent 们的通讯，就如果我们在一个小团队里面紧密协同那样。而 Google 提出的 A2A 协议，把 Agent 之间的协同范围一下子提升到了全球的范围，它为每个 Agent 派发了身份证（AgentCard），在经过认识、握手后（鉴权），Agent 们可以进行沟通和协作。

展开想象一下：

● 每个人都配套一个人个的 Agent，用于代表你跟 Agent 的世界来交互，这个场景就很好玩了，跟朋友们约出去玩？让咱们的 Agent 们先商量一下，给我们一个方案；

● 买机票？我也不需要直接用某程的平台，只需要交代我的专属 Agent，它自动发现和跟服务商的 Agent（机构 Agent）来沟通并支付就 OK 了。

● 你看，一个赛博数字世界就这么展开了。

我愿把这种场面称之为 Agent 的社会化协同，它将最大程度上复刻人类社会的形同范式，Agent 间需要有验证机制，能互相加好友，具备支付能力，能主动发起任务等等。技术上，这将有模型技术之外的海量的 agent 社会基础平台等着被搭建。包括 Agent 通讯的安全、信用、支付体系等等。

### **致亲爱的乘客**

#### **做 AI 的领导者**

AI 正在对全行业进行无差别的颠覆，所有人都面临着工作方式的升级。不是说有全新职业的出现，而是大部份职业都会被要求原地升级 + AI。

**我们每个人都会从个人劳动者转变成 AI 领导者，我们要提升自己的 AI 领导力。**

过去，我们通过个人的专业能力来交付工作成果，个人要亲自去执行具体的任务。

现在到不远的未来，是我们带着 AI 一起工作并完成目标，我们作为 AI 的领导者，需要对 AI 团队进行目标设定，对 AI 协作过程进行管理和干预，对 AI 最终产出进行验收。

虽然执行性的工具会逐渐交给 AI，但这并不意味着对个人的专业能力不作要求了。相反，它对我们的专业能力要求更高了，因为我们需要以内行人的角度来验收 AI 给我们产出的东西，减少的只是我们做具体任务的时间。

因为 AI，未来可能每个行业都可能呈现出两头重，中间轻的形成。以软件开发这个岗位来做一下推演。

Vibe Coding 这个词相信大家已有所耳闻，现在越来越多完全没有编程经验的人（暂称为小白）通过 Cursor 这类 AI 编程工具摇身变成了开发者，这类开发者自己动手解决长尾的、相对简单的个性化的需求，中低端的开发者的工作将会由小白们 + AI 来接管。但是大规模，严肃的生产型应用，小白 + AI 也是无法掌控的，这个场景需要更专业的工程师，甚至是架构师 + AI 来支撑，AI 一定是必备的了。可见，小白和架构师就是两头，初中级的工程师如果想要继续留在这个行业，是需要进一步提升自己的专业能力和 AI 领导力的。

所以：全面拥抱 AI 吧，以最快的速度。

#### **我们的征程是星辰大海**

当瓦特改良的蒸汽机轰鸣着撕裂中世纪余晖，当珍妮纺织机的梭子编织出工业文明的经纬，旧时代的质疑声总如潮水般涌来——1830 年马车上挥舞的皮鞭在对抗铁路钢轨，1910 年马车夫的咒骂声淹没在福特 T 型车的鸣笛中。历史总在证明：人类对变革的恐惧，终将被创新者的勇气锻造成进步的阶梯。

站在 AI 浪潮席卷全球的临界点，我们目睹着更宏大的技术跃迁。AlphaGo 落子的清脆声响彻人类智慧圣殿，ChatGPT 的字符洪流重塑知识生产边界，波士顿动力的机械骨骼正在突破生物运动的极限。如同十九世纪纺织女工面对蒸汽机时的惶恐，今日的焦虑不过是文明跃迁时的引力震荡。

但请记住：马车消亡时，人类获得了驾驭钢铁的速度；纺车停转时，世界收获了机械纺织的精度；而当 AI 接管程式化劳动，我们终将解锁更珍贵的创造力密码。凯恩斯曾在汽车取代马车的年代预言："我们终将学会游泳，在技术的海洋里。" 数据显示，自动驾驶技术每年将挽救全球 130 万条生命；AI 医疗系统已能诊断出人类医生难以察觉的早期病症。

那些被技术重塑的行业，正在诞生更璀璨的新职业星辰：提示词工程师构筑人机对话的巴别塔，AI 伦理师守护智能时代的道德罗盘，元宇宙建筑师在数字空间重构文明形态。正如马车夫转型为汽车司机，纺织女工成为流水线技师，每一次技术革命都在创造更高阶的人类价值。

不必困在技术性失业的叙事茧房，人类真正的对手从来不是机器，而是固守成规的思维惯性。当 NASA 用 AI 分析亿万光年外的星云数据，当脑机接口帮助渐冻症患者重获交流能力，我们分明看见：智能革命正在拓展人类探索的边疆。

心存焦虑时，请回望文明长河中的灯塔——蒸汽机没有埋葬人类，电力没有禁锢光明，互联网更未终结真实。我们的征途注定是星辰大海，在算法与神经元共舞的新纪元，唯有保持认知的流动性，在持续迭代中铸造不可替代性，方能在技术洪流中锚定航向。正如航海者从不诅咒潮汐，真正的勇者会将 AI 化作驶向星海的方舟。（by Deepseek）

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/j3gficicyOvasVeMDmWoZ2zyN8iaSc6XWYjZ7Hx6Udjjk2BGLzC9ahJq7ibxDd1RGA0c9NYZc1husEsvb3tY4FcWPQ/640?wx_fmt=gif&from=appmsg)

  

  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasVeMDmWoZ2zyN8iaSc6XWYj5q5PQEOc5ibURPb03vnRibrxC3UR8xzdyATfiawTYRV2vJvBnAIcE1FeQ/640?wx_fmt=png&from=appmsg)