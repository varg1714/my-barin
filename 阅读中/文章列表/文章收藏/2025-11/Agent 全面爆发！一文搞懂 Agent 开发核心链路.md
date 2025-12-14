---
source: https://mp.weixin.qq.com/s/gSDpv742AcDg7Jn4Ayppyg
create: 2025-11-25 16:43
read: false
knowledge: false
---
![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe94cQiccAo2zibZETiaOnMVLNQAO0Zne2x8KlehRMR8AsOTW90m1pAicBEw5wBJFkQiax8ricKGbKibEKV8gQ/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=0)

![](https://mmbiz.qpic.cn/mmbiz_gif/VY8SELNGe96srmm5CxquJGSP4BbZA8IDLUj8l7F3tzrm8VuILsgUPDciaDLtvQx78DbkrhAqOJicxze5ZUO5ZLNg/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=1)

  

👉目录

1 Agent 介绍

2 Agent 的核心演变和核心模块

3 怎么基于大模型搭建一个 Agent?【开发视角】

4 Agent 如何评估？

当数据自己会思考，当代码拥有自主决策的能力——从 Workflow 到 Agentic AI，一个完全自主的数据分析师，究竟是如何诞生的？

  

两年深耕，无数次架构迭代与实战验证，我们终于摸清了 Agent 开发的核心脉络。本文将带你深入 Agent 的 “大脑” 与“四肢”，从规划、记忆、工具调度，到上下文工程的精妙设计，一步步拆解如何构建一个真正 “会思考、能执行” 的智能体。无论你是好奇者、学习者，还是同行探索者，都可以了解下架构认知与实践心得。

文章略长，但每一段都来自真实项目的沉淀。如果你也相信 “自主智能” 不是空中楼阁，那么，欢迎进入 Agent 的构建现场。

关注腾讯云开发者，一手技术干货提前解锁👇

一个完全自主的 Agentic AI 数据分析师，可能么？可能！

作为 AI Agent 领域的入局者，接触该类产品开发不知不觉已有两年多之久。期间前前后后调研了不少内容，从开发经验总结的角度写下这一篇文章总结记录的同时，希望能给需要的同学们带来一定帮助。

这期间团队也做了不少产品落地的尝试，从一开始的网页端 OlaChat，到辅助写代码的 copilot，再到如今的数据分析 Agent dola，我们终于看到了全自主 AI 数据分析师的影子，腾讯 PCG 大数据平台部的新一代数据分析 AI 助手——Dola 是一款基于 Agentic AI 能力开发的数据分析助手：用户只需要引入个人的数据表，就能得到一枚专属的 AI 分析师。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxPibr8X8hX7vJlk7UaIcCsjgf8qqlZcK1KFLxMic0uj2iat6cCTwviayjCQ/640?wx_fmt=png&from=appmsg#imgIndex=2)

它不仅能够完成日常的取数、跑数等基础任务，还能自主规划并执行复杂场景的数据分析，例如异动归因、画像对比分析、股票基金回测、房价预测等。Dola 可以自行编写 SQL、纠正 SQL 错误、执行查询、使用 Python 进行数据处理与可视化，并最终生成一份完整的分析报告。全程无需编写一行代码，只需通过自然语言对话，你就能拥有一个全自动工作的 “数据小黑工”。目前已经能自己自主完成数据 / 产运同学的部分工作。

这里以 1 个股票回测的例子看看 dola 的效果：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxE2QwCb2PutbwZDoibKyJibu1cibqicLMJagyiaVmEMpXO3YKMyicrtDCC8LA/640?wx_fmt=png&from=appmsg#imgIndex=3)

话不多说，现在我们就开启学习之旅吧——本次文章有点长，涵盖 Agent 基础架构全介绍（规划、记忆、工具调度、上下文工程等等）、如何开发以及如何评估等话题，大家准备好开始阅读啦~

  

  

# 01

  

  

Agent 介绍

 1.1 什么是 Agent？

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxD2dSDme5qx6yF4zfia6xY3iatc3icLBrcLnyGx5NopjS7BOZd4PIlzN2g/640?wx_fmt=png&from=appmsg#imgIndex=4)

Agent（智能体或代理）是人工智能领域中的核心概念，指能够感知环境、自主决策并执行任务以实现特定目标的智能实体。简单来讲，可以理解它是代理你去做一些事情。

Agent 具备四大核心能力：

*   环境感知：通过视觉传感器、语音接口等多模态 "感官" 实时获取环境数据
    
*   智能决策：运用深度学习模型和强化学习算法进行复杂决策
    
*   任务执行：可调用 API 工具库或操控物理设备完成实际工作
    
*   持续进化：具备在线学习和迁移学习能力，实现性能的持续提升
    

 1.2 agent 基础框架

最早大家所熟悉的 AI Agent 图如下：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRx2uZPdt9SKDb11iajr0HEUXu31OeTCoU25a1F0pScNanpVYlDBiaFwicicQ/640?wx_fmt=png&from=appmsg#imgIndex=5)

主要有规划、记忆、工具、执行模块（后两者结合可以理解为工具使用）。而完成这些操作的底层控制中枢，类比人可以理解为是人的大脑，在 AI 领域则是大模型充当了这一角色。

所以归纳得到，AI Agent = 大脑（LLM）+ 记忆 + 工具使用 + 规划。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRx2WAgF2tia4cmGFLSqyHkhmg9T2xT2uS2M5dqJKTaDibajiaec1qQnGymA/640?wx_fmt=png&from=appmsg#imgIndex=6)

大模型目前的发展历程中，聚焦从内容智能到行为智能，从而实现通用人工智能。对话、推理、自主调度 Agent、创新、组织是智能化的五大体现。从最早期的对话机器人形态到如今可借助工具等自主解决问题的 Agents 形态，未来大模型有希望激发创新、组织层面的能力。

 1.3 Agent 的分类？

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxERjXS1ELZd7LHg8ib6YvFu3Q7uUib8gCibXpiahAPPDZhqxA4ic2lau98ibQ/640?wx_fmt=png&from=appmsg#imgIndex=7)

Agent 可以分为四种形态:

1.  Reflection【反思模式】：通过模型自身反思来改进任务的执行，例如 react、self-refine、refine 属于该类
    
2.  Tool use【工具调用】：涉及模型调用外部工具或者库来解决任务
    
3.  Planning【规划模式】：提前计划和组织步骤来提升效率和准确率
    
4.  Multi-agent collaboration【多智能体协作】：涉及多个智能体进行协作来提升任务执行能力，例如 A2A 协议属于解决该类问题的协议
    

1）Reflection【反思模式】

通过模型自身反思来改进任务的执行，例如 react、self-refine、refine 属于该类

*   代表技术：ReAct 框架、Self-Refine 算法
    
*   特点：通过任务执行后的自我反思不断优化决策过程
    
*   应用场景：需要持续优化的复杂决策系统
    

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxjb1kqk1FajCcj45UQdcMTxgJg2j2uk2RPl7MpaXjAnk0zUc1ickHrqw/640?wx_fmt=png&from=appmsg#imgIndex=8)

2）Tool use【工具调用】

涉及模型调用外部工具或者库来解决任务。具体调用什么工具和工具的参数均由模型决定。

*   核心技术：函数调用（Function Calling）、API 集成
    
*   优势：突破大模型固有局限，实现现实世界操作
    
*   典型案例：自动订票系统、智能客服工单处理（一些相对简单的内容）
    

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxqN4EiasiboPZjxHAufX8BzuxQaezQUBoA2aDWy0dg9byY4BNhw6Y3Uicg/640?wx_fmt=png&from=appmsg#imgIndex=9)

3）Planning【规划模式】

提前计划和组织步骤来提升效率和准确率：  

*   关键技术：分层任务网络（HTN）、蒙特卡洛树搜索
    
*   价值：显著提升复杂任务的执行效率和成功率
    
*   应用实例：物流路径规划、生产排程优化
    

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxiaspu6Z6sDL2vIFovRSJXAibKic9l0NBmO2Po735bNib6eywicdW9to5s9w/640?wx_fmt=png&from=appmsg#imgIndex=10)

4）Multi-agent collaboration【多智能体协作】

涉及多个智能体进行协作来提升任务执行能力，例如 A2A 协议属于解决该类问题的协议：

*   前沿协议：A2A 协作框架、联邦学习机制
    
*   突破：实现智能体间的知识共享与协同决策
    
*   典型应用：分布式智能系统、群体机器人控制
    

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxIrAqHAmKHfsfVXFroBL4gVU5OCicEmLMdM5hibD7g71kBTBflEkUNmpg/640?wx_fmt=png&from=appmsg#imgIndex=11)

 1.4 Agent 的开发框架

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxCPziao4NXKjibpfkIhIgoVAoQE3xXj4vJYr7gTz5jbkicaUSK8AIib59Qg/640?wx_fmt=png&from=appmsg#imgIndex=12)

大模型多 Agent 协作技术的发展可以明显划分为三个主要阶段，每个阶段都有其独特的技术特点和代表性成果。上图展示了从 2023 年初至 2025 年，该领域的主要发展脉络和关键技术节点。下面我们将对每个阶段进行详细分析。

早期探索阶段（2023 年初 - 2023 年中）

在大模型多 Agent 协作的早期探索阶段，研究主要集中在概念验证和基础框架构建上。这一时期的特点是研究者们开始意识到单一大模型在处理复杂任务时的局限性，并尝试通过多个 Agent 的协作来突破这些限制。早期的研究更多关注于如何让多个 LLM 驱动的 Agent 进行基本的信息交换和任务分工，协作机制相对简单，主要采用顺序执行或简单的并行处理模式。

这一阶段的代表性工作包括对多 Agent 系统基本架构的定义和初步的协作协议设计。研究者们开始探索如何将传统多 Agent 系统的理论与大语言模型的能力相结合，但在 Agent 间的深度协作、知识共享和动态协调方面仍存在较大挑战。

框架成熟阶段（2023 年中 - 2024 年中）

随着技术的不断发展，2023 年中期到 2024 年中期标志着多 Agent 框架的成熟阶段。这一时期最重要的里程碑是微软 AutoGen 框架的发布，它为多 Agent 协作提供了一个标准化、易用的开发平台。AutoGen 的核心创新在于其 **"可对话 Agent"** 概念，使得不同类型的 Agent 能够通过自然语言进行灵活的交互和协作。

在这一阶段，研究重点从基础概念转向实用性和可扩展性。《Scaling Large Language Model-based Multi-Agent Collaboration》等研究开始关注多 Agent 系统的规模化问题，探讨如何在增加 Agent 数量的同时保持系统效率。MacNet 模型的提出标志着对 Agent 协作网络结构优化的深入思考，引入了动态协作机制和基于注意力的通信协议。

同时，这一阶段也见证了多 Agent 系统在特定领域的深度应用探索。研究者们开始将通用的多 Agent 框架与具体的应用场景相结合，如金融交易、软件开发、内容创作等，形成了领域特化的 Agent 协作模式。

应用深化阶段（2024 年中 - 2025 年）

进入 2024 年中期以后，大模型多 Agent 协作技术进入了应用深化阶段。这一时期的特点是从概念验证转向实际部署，从通用框架转向领域专精。TradingAgents 和 FinTeam 等专门针对金融领域的多 Agent 系统的出现，标志着该技术开始在垂直行业中找到具体的落地场景。

在这一阶段，技术发展呈现出几个明显趋势：首先是 Agent 角色的专业化程度不断提高，每个 Agent 都被赋予特定的专业知识和技能；其次是协作机制的复杂化，从简单的信息传递发展为复杂的工作流编排和决策协商；第三是评估体系的完善，研究者们开始关注如何科学地评估多 Agent 系统的协作效果和整体性能。

最新的研究如《Multi-Agent Collaboration Mechanisms: A Survey of LLMs》和《A Survey on LLM-based Multi-Agent System: Recent Advances and Challenges》反映了该领域的理论体系正在趋于成熟，同时也指出了未来发展的关键方向。

下面给出主流的一些其他框架对比表：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxico1MKiaWcNwkGFicYXcHAz6z9cScjCaQzsfLz5ictoy4x4cicR8DWvGjLA/640?wx_fmt=png&from=appmsg#imgIndex=13)

除了上述还有 qwen-agent， OpenAI Swarm 等。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxgFOVsQ5iboicYDBQ1WPn8LhljMFLAXSd6BAxydS1tHLmzwDZTWrwSv4g/640?wx_fmt=png&from=appmsg#imgIndex=14)

在技术发展的浪潮中，单智能体系统逐渐难以满足日益复杂的效能需求，由此催生了多种多智能体协同解题的架构思路。以下将介绍当前三个主流智能体框架中，如何有效实现多智能体之间的协作与问题求解。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxmATvhXfiaqtTw3uYkMwHKlejzibMAPlBN7K1d7pm4BOJpnibmyxTIqlPg/640?wx_fmt=png&from=appmsg#imgIndex=15)

AutoGen 适合需要高度定制化和灵活对话的场景，其强大的可编程性使其能够处理各种复杂的协作需求，但需要开发者具备较强的技术能力。

LangGraph 适合需要精确流程控制和状态管理的场景，其图结构工作流和状态管理机制使其在处理复杂业务逻辑时具有优势，但学习成本相对较高。

Crew AI 适合需要快速搭建团队协作系统的场景，其高级抽象和简洁接口使其易于上手，但在复杂定制需求方面可能存在局限。

目前市场上已有框架各有优劣，例 langchain 这种框架代码太过于笨重，在实际开发过程中会遇到无法调控需改底层源码的问题，届时可能更加费力不讨好。为了灵活自主调控，我们产品目前是选择自己搭建的 Agent 框架。选择合适的框架需要综合考虑项目需求、团队技术能力、开发周期等多个因素。在实际应用中，也可以考虑混合使用多个框架，发挥各自的优势。

  

  

# 02

  

  

Agent 的核心演变和核心模块

前后跟 Agent 打交道算起来也有 2 年半多了，我们最早的期望就是打造智能数据分析的一站式服务。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxz1sHSX81NV49d1Vpwj74zmblk10kNn8CDzoD4CyuJaN7XYy0J88uug/640?wx_fmt=png&from=appmsg#imgIndex=16)

早前大模型的能力受限，我们自己给 Agent 创建手脚，更多是以 workflow 的形态存在的。最早我们依托 workflow 的产品框架如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxZSF3ZcIx99FRpeIeXEx0m3JZvpn4ylknlupAicFEqQjEvhd0buiaY0ew/640?wx_fmt=png&from=appmsg#imgIndex=17)

核心模块上跟 Agent 没有太大的区别，实际工作中是以规定好的流程进行行动的。

workflow 和 Agent 的区别：workflow 更多是指开发人员预定好工作的步骤流程，系统内按照预定好的步骤进行行动。而 Agent 更多是指自主定义和自主探索。

随着大模型能力的增强，目前它的自主探索能力已经具备，workflow 可能是特定时代特定业务下的特定产物，长期 AGI 下，更多产品还是会以 Agent 形态存在。

 2.1 规划模块

*   https://arxiv.org/pdf/2402.02716 详细可阅读该文章
    
*   A Survey of Large Language Models
    

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxbPJ3t3GKtGfzVbfUbq211yLyXIP2w79GcDjDDl0jfxAtUCaTqo6vhw/640?wx_fmt=png&from=appmsg#imgIndex=18)

什么是规划：

Planning 可以分为：

*   任务分解：把大任务划分成小任务，进而解决复杂问题；代表性的做法是基于 CoT 技术直接让模型进行任务分解。
    
*   反思和提炼：基于已有动作做自我批评和自我反思，从错误中学习优化接下来的动作。这里有代表性的做法是 ReAct，相信熟悉 Agent 的同学不会陌生。
    

ReAct 的核心点：

指导 AI Agent 通过思考、行动、观察的循环来实成任务。Agent 接到任务后的工作流程大致如下：1、思考 (thought)，要解决该问题，下一步需要采取什么行动。2、行动 (action)，大模型输出行动指令，让 Agent 调用外部工具。3、观察 (observation)，把工具执行的结果给大模型进行观察。4、回答 (answer)，如果工具执行的结果已能得到答案，组织语言回答。如果目前得到的信息仍无法作答，进入下一次循环，继续思考使用工具

ReAct 整体其实很像 PDCA 流程，先计划，然后做，check 之后再处理。

规划能让 Agent 在复杂问题上显得更加聪明，类比人类在完成复杂任务的时候可能大多时候也会有个计划大纲。例如写一本厚厚的书，我们需要拟定一个大纲；做 PPT 也是同理，大多时候我们需要写一下脉络再具体完成每个小节。

规划的框架如下图所示，左边是任务的规划者，右边是规划的执行者。简单来做则计划可以不支持做更新，复杂一些则在执行者执行的时候仍可以动态去更新计划（refine 逻辑）。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxDph4lP83Ue0krIKyibIKESXlk114QcJTk1Aa8e9rqtCeOnvBGkv8wkw/640?wx_fmt=png&from=appmsg#imgIndex=19)

源自：A Survey of Large Language Models

目前一些主流产品都使用了规划模式：manus、cursor 等。

计划模块如何实现决定了系统的效率和性能。该模块可以通过模型微调或者上下文工程完成，前者更适配特定业务但缺乏灵活和快速扩展的能力，后者可以快速扩展但缺乏一定的业务适配度，具体使用中可以依据业务场景决策。

 2.2 记忆系统

记忆系统是什么？

记忆系统是指通过特定机制存储、管理和检索信息，以增强模型在长期交互或复杂任务中的上下文连贯性、个性化响应及知识持久化的技术框架。其核心目标是解决大模型因固定上下文窗口限制导致的 “失忆” 问题，并模拟人类记忆的分层与动态更新特性。

为什么需要记忆系统？

*   大模型本身不存在记忆能力——如果开发 Agent 应用，需要外挂记忆系统进行信息的记忆
    
*   大模型本身的上下文阈值是有限的——开发过程中没有办法直接给它所有的对话或者文本内容，超出后早期对话内容会被丢弃，导致多轮对话中出现 **“断片” 或重复提问 **。记忆系统通过分层存储（短期 / 长期记忆）和动态检索（如向量数据库），将历史信息压缩后注入当前上下文，确保对话连贯性。
    

记忆系统的分层架构：

记忆系统通常借鉴人类记忆的三层结构，分为短期、中期和长期记忆：

*   短期记忆（STM）：存储当前对话或任务的即时信息，受限于模型的上下文窗口长度（如 GPT-4 的 2048 tokens）。实现技术：将对话历史直接嵌入提示词中，但容量有限。
    
*   中期记忆（MTM）：整合短期记忆中的主题信息，通过分段分页策略组织（如 MemoryOS 将同一主题的对话归并为 “段”），并基于热度算法（访问频率、时间衰减等）动态更新。
    
*   长期记忆（LPM）：持久化存储用户偏好、角色特征等个性化数据。例如，MemoryOS 的 LPM 模块包含用户画像和智能体特征，通过向量数据库或知识图谱实现长期存储，然后通过 RAG 手段来进行提取。
    

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxlxsjlvWrqD4hEv5H4ORYff14vMKVQN01jRa9icEiatkhWqNn3wSwHBaw/640?wx_fmt=png&from=appmsg#imgIndex=20)

 2.3 工具 / 函数调度

what？——function call 是什么？LLM 通过结构化指令调用外部函数 / API，将自然语言意图转为可执行指令的能力。简单说：Function Call 就是让智能助手能调用外部工具的功能，比如查天气、订外卖、算数学题，让它从 “只会说话” 变成 “会办实事” 的全能帮手！

why？——为什么需要 function call？ 大模型本身的一些缺陷，例如： 1. 知识有时效性缺陷 比如你问 "2025 年 NBA 总冠军是谁？"（假设现在是 2025 年），模型如果没学过 2025 年的新数据，就答不上来。但用 Function Call 调用体育新闻 API，马上能拿到实时结果。 2. 不会做专业操作 模型懂医学知识，但不能直接帮你查医院排班；懂数学公式，但不会用 Excel 算工资表。而 Function Call 能喊医院系统、办公软件来干活，实现 "理论 + 实操" 结合。

Function call 可能的问题：错误调用参数、幻觉生成 API、依赖关系混乱、隐私问题（如乱调用支付软件）。如下面举了一些例子：

1.  如果 “外援” 掉链子，助手也会变笨 比如天气 API 突然崩了，助手查不到天气，只能跟你说：“抱歉，天气信息没拿到...”
    
2.  隐私问题：别让助手乱翻你的东西 如果助手能调用你的支付宝付款，万一被坏人骗了，可能会乱花钱——所以现在很多 Function Call 都需要你 “授权确认” 才能用。
    
3.  复杂任务容易搞砸 比如你让助手 “先订机票，再订酒店，最后租辆车”，如果其中一步出错（比如机票卖完了），助手可能不知道怎么调整，需要你手把手教它。
    

 2.4 额外话题：MCP 是什么？

最简单而言：MCP 提供了工具（资源和 prompt 用的少）的规范化协议。它的核心组件有以下几点：

1.  MCP 主机（Host）：发起请求的应用程序（如 AI 编程助手、IDE 插件）；
    
2.  MCP 客户端（Client）：与服务器保持 1:1 连接的通信模块；
    
3.  MCP 服务器（Server）：运行于本地或远程的轻量级程序，负责访问数据或执行工具（如读取本地文件、调用 API）；提供三种标准类型能力：Resources、Tools、Prompts。
    
4.  资源层：包括本地文件、数据库和远程服务（如云平台 API）。本地资源（Local Resources）：本地计算机中可供 MCP 服务器安全访问的资源，如文件、数据库。远程资源（Remote Resources）：MCP 服务器可以连接到的远程资源，如通过 API 提供的数据。
    

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxIXRJQicricr4iaZ6nJsOVv2wMnu1ZF4hFOINiarPjoCGeGL9n8UPjsrqFA/640?wx_fmt=png&from=appmsg#imgIndex=21)

为什么选择 MCP？

1.  标准化集成：统一接口降低开发复杂度
    
2.  动态扩展与实时交互能力【可随时增减工具】
    
3.  生态兼容性与开发效率提升【目前已成为主流协议】
    

工具侧具体的实现不公开给 HOST 应用系统，互相独立维护。MCP 协议统一，有助于优秀工具的快速集成，在实际开发 HOST 系统的时候就能快速使用到市面上相对较好的工具（而不需要自己再调用 API 维护工具函数或者去找到开源代码重新整合到系统中)。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRx1Va0wyK2uaUF816vXW7ez036GnzrBibAxhAb9cLohald0QVRAnWOTIg/640?wx_fmt=png&from=appmsg#imgIndex=22)

### MCP 到底是不是良药？

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxLIKLKp2lq68SZVicDq0PZEJhEAzUEhm11b6nzSyS7V0Mcs4xia2W3dZA/640?wx_fmt=png&from=appmsg#imgIndex=23)

MCP 技术并不全都是好处，能解决部分问题，但也带来了其他的问题。

在实际开发下来，发现 MCP 有好有坏，目前发现的好坏处如下：

好处是：可以快速上下工具；能够快速接入市面上已有的 mcp 应用，不同部门之间可以共建 mcp server 服务。【但仍需特别了解 MCP 工具的内部实现，否则不建议接入到自己产品中，存在安全问题的同时可能较难调教整体效果】

坏处如下：

坑点 1：不好查日志，排查问题费劲

server 端的日志较难排查，无法通过类似一个 trace_id 进行串联定位。目前可以通过访问时间戳找到相关的日志群。

坑点 2：连接中断问题

多了一层服务交互，可能出现的问题更多，如最常遇到的就是连接中断问题。sse 的方式已经被 stream mcp 逐渐替代，但后者也并不保证百分百稳定。多了一层服务，就多了一层不稳定性。

坑点 3：性能问题

缺乏连接池，每次交互都需新建一个连接，当流量上来性能会是一个问题。

坑点 4：出参没有严格标准

目前出参就是基础的 JSON Schema，为保证效果，工具最好是需要统一出参的，但 MCP 并没有提供该类的约束，需开发者自己建立和遵循。

坑点 5：效果调教的时候需要同步更新多个服务

开发过程中大多时候需同步更新 server 和 host 端的代码。

如果产品设计上永不考虑快速接入市面上的 MCP 应用并且 MCP server 与 Host 端是一方开发的情况下，其实可以不使用 MCP 框架。

MCP 开发中可能的误区

1.  MCP 出现之前是不是没有办法调度工具？不是的。它算是一个通用管理协议，把一个个工具按照一定的规范装在一个量身定做的箱子里。之前也可以进行调度，调度工具关键的点在于大模型识别出使用什么工具以及工具相对应的参数。langchain 等在 MCP 之前就已经支持了工具调度的能力。
    
2.  MCP 可以代替 function call 么？不行，他们是一起工作的关系。Function Call 是大模型返回调用工具指令的能力，MCP 是 Agent 在工程侧的程序具体执行调用工具的手段，一个是指挥者，一个是执行者。
    
3.  MCP 能减少调用的 token 数么？不行，它只是整合了工具，但实际还是需要把工具参数都发给大模型。整体 tokens 数消耗相比之前的方式是没有本质区别的，整体依赖于系统的实现。
    
4.  是不是有了 MCP，Agent 应用就能轻而易举搭建了？不是。MCP 只是把工具管理和获取调用这里规范了，但 Agent 应用关键点还有：LLM 调用（含 function call、prompt 工程），记忆系统（RAG，上下文管理），思考和计划系统等。如果想要做好一个 Agent 系统应用，还需要仔细打磨前面提到的模块。
    

  

  

# 03

  

  

怎么基于大模型搭建一个 Agent?【开发视角】

业务大模型的训练和优化等不是此文重点，此处省略。重点关注怎么基于大模型来搭建 Agent 系统，让整体 Agent 产品的效果更好。

 3.1 什么影响了 Agent 的效果？

从前文介绍，可以知道 Agent 由几个重点的模块组成，那么这几个模块的组合方式【Agent 框架设计】以及每个模块的效果决定了最终的效果。

*   Agent 框架设计：对应 Agent 分类里面的内容。是否多 Agent 设计要依据产品实际场景来考虑，并非任何场景都适配多 Agent。多 Agent 本质还是可以理解为上下文工程的演化，让单一 Agent 单独去处理自己部分的上下文内容 + 其他 Agent 精简的上下文。
    
*   上下文工程：曾经大家更熟悉的名词可能是 prompt 工程，但上下文工程含义更广泛，包含了记忆系统提取出来内容的处理（即简单理解为多轮对话）等的处理，并非简单写一个提示词。
    

 3.2 如何构建良好的上下文工程？

关于上下文工程这一块，汇总了 manus 的经验以及自己开发过程中遇到的一些问题。

核心点 1. 围绕 KV-Cache 优化设计

*   关键指标：KV 缓存命中率直接影响延迟和成本（如 Claude Sonnet 缓存 token 成本降低 10 倍）。
    
*   优化策略：
    

*   稳定提示前缀：避免动态内容（如秒级时间戳）破坏缓存。
    

*   追加式上下文：禁止修改历史动作 / 观察，确保序列化确定性（如 JSON 键顺序稳定）。
    

*   显式缓存断点：对不支持自动缓存的框架，手动标记断点位置。
    

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxBwrSNsicDEibcoFXic14ibVDmSUoiaH0O549ChhhDdusMLviclOwvibIsfydA/640?wx_fmt=png&from=appmsg#imgIndex=24)

核心点 2. 动态约束行为选择（非移除）

*   问题：工具数量爆炸导致模型动作选择混乱。
    
*   解决方案： 
    

*   Logits 掩码：通过屏蔽非法动作 token（如 browser_* 前缀工具组）约束选择。
    

*   状态机管理：根据上下文预填充响应模式（Auto/Required/Specified），不修改工具定义。
    

具体，

*   Auto – 模型可以选择是否调用函数。实现方式：仅预填充回复前缀：<|im_start|>assistant
    
*   Required – 模型必须调用函数，但具体调用哪个函数不受限制。实现方式：预填充到工具调用标记：<|im_start|>assistant<tool_call>
    
*   Specified – 模型必须从特定子集中调用函数。实现方式：预填充到函数名开头：<|im_start|>assistant<tool_call>{"name":「browser\_
    

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxS9aMQhDPib15gbuPzA3bYbM5xQyv0iaeSFJK5uZs1tXgnKFpBibxSmUNA/640?wx_fmt=png&from=appmsg#imgIndex=25)

核心点 3. 文件系统作为扩展上下文

*   挑战：128K 上下文窗口仍不足，且长文本导致性能下降、成本高。
    
*   创新设计：
    

*   外化存储：文件系统作为无限容量的持久化记忆，支持按需读写。
    

*   可逆压缩：仅保留引用（如 URL / 路径），内容可随时还原，避免信息丢失。
    

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxyCtDn85INABCMsvB2jvqrsNQK0lAeiaCLwuPd7oJl084caQFhLSXH5Q/640?wx_fmt=png&from=appmsg#imgIndex=26)

核心点 4. 注意力操控：复述目标

*   现象：Manus 创建 todo.md 并动态更新，勾选已完成项。
    
*   原理： 
    

*   抗遗忘：将长期目标 “背诵” 到上下文末尾，强化模型近期注意力。
    

*   无架构改动：通过自然语言重定向模型焦点。
    

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxhaQRtX1jXkfBUib84EtcGJc3LcCnTSBI2el00E4iaW41JoyrdREBrOdw/640?wx_fmt=png&from=appmsg#imgIndex=27)

核心点 5. 保留错误以促进学习

*   反模式：掩盖错误（重试 / 重置）会剥夺模型学习机会。
    
*   关键实践： 
    

*   失败即证据：保留错误动作及环境反馈，修正模型内部信念。
    

*   智能体标志：错误恢复能力是真实智能行为的核心指标。
    

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxcKWu7mxDgibOszmMgJYEWeGY4Pka6ssYEJ0ibsS0Rhau9ibycM0ibdkkcw/640?wx_fmt=png&from=appmsg#imgIndex=28)

核心点 6. 警惕 Few-Shot 陷阱

*   问题：过多相似示例导致模型陷入固定模式（如重复简历审阅动作）。
    
*   解法： 
    

*   注入多样性：通过模板变体、表述扰动打破单调性。
    

*   动态调整：避免上下文过度同质化。
    

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxB7tsqsbke6n6oicia9D1iciaB1Nf0lNfhPRPmfHwayoowqOK4WNJIMPkbQ/640?wx_fmt=png&from=appmsg#imgIndex=29)

核心点 7. 提示词内容一致性

*   问题：系统专用术语（如内部 API 名称、业务逻辑标签）未被模型正确理解，导致动作偏差。
    
*   关键实践： 
    

*   术语对齐：在提示词、工具描述、环境反馈中强制统一术语表述（如 “用户画像” 不可同时称“客户画像”）。
    

*   动态解释：对关键术语插入简短定义（例：<数据湖> 指跨部门原始数据存储库，非结构化优先）。
    

*   错误溯源：若模型误用术语，在反馈中显式纠正并记录术语表。
    

核心点 8. 动态提示词

*   问题：静态提示词无法适应环境状态变化（如任务阶段切换、权限变更），导致无效动作。
    
*   解法： 
    

*   状态感知注入：根据实时环境变量（如 current_step=3）动态插入提示片段（例：当前需聚合前两步结果，禁止调用数据采集工具）。
    

*   优先级分层：将提示词分为基础规则（不变）+ 场景规则（动态），后者通过函数实时生成。
    

*   历史敏感度：检测到重复失败时，自动追加注意：此前因忽略时间约束失败，本次需校验截止时间。
    

核心点 9. 工具能力边界显式化

*   反模式：模型因工具文档模糊而尝试不可能动作（如用 PDF 解析工具处理图像）。
    
*   解法： 
    

*   能力 - 限制对仗描述：每个工具说明需包含支持输入类型和明确失败条件（例：仅支持 UTF-8 文本，二进制输入返回 ERROR_12）。
    

*   运行时校验：在动作执行前，系统先检查参数是否符合工具约束，快速拦截无效请求。
    

核心点 10. 环境反馈的颗粒度控制

*   问题：过于简略的反馈（如仅返回错误代码）导致模型无法定位问题，过于冗长的反馈则引入噪声。
    
*   解法： 
    

*   分层反馈机制： 初级反馈：即时返回可行动错误类型（如参数缺失：缺少 user_id 字段）。深度反馈：当模型请求 / debug 时提供堆栈追踪或数据流快照。
    

*   错误分类强化：将系统错误映射为模型可理解的类别（如权限类 / 数据格式类），并关联修复建议。
    

核心点 11. 长期记忆与短期上下文的平衡

*   反模式：过度依赖当前会话上下文，忽略历史经验积累。
    
*   关键实践： 
    

*   记忆分级： 短期：保留最近 N 轮动作 - 反馈对，用于即时纠偏。长期：将高频错误模式存入知识库，触发相似场景时主动提醒（如历史记录显示，该 API 在时区为 UTC+8 时易超时）。
    

*   记忆更新策略：对已验证的修正知识（如 X 字段需加密传输），通过人工审核后固化到系统约束。
    

核心点 12. 人类干预的黄金分割点

*   问题：过早人工介入会抑制自主学习，过晚则导致代价高昂的错误。
    
*   策略： 
    

*   三级熔断机制： 自动修复：对已知错误（如字段格式不符）调用预设修正流程。
    

*   半自动干预：当连续 3 次相似失败时，请求人类确认动作。
    

*   强制暂停：检测到高风险动作（如删除操作）且置信度 < 80% 时冻结任务。
    

*   干预记录分析：统计人工介入原因，反向优化自动化规则。
    

尝试点 13. 多智能体竞争验证

*   创新实践： 
    

*   影子模式：部署双模型并行执行任务，对比结果差异并交叉验证（如 Agent A 生成 SQL 查询 vs Agent B 生成 NoSQL 查询）。
    

*   对抗辩论：当结果不一致时，强制双方基于环境反馈进行逻辑辩论，系统根据辩论质量选择最优解。
    

```
# 动态提示词生成伪代码示例
def generate_dynamic_prompt(base_prompt, state):
    # 注入状态感知片段
    if state["step"] == 2 and state["last_error"] == "timeout":
        base_prompt += "\\\\n[紧急] 上一步因超时失败，本次请将timeout参数设为5000ms"
    # 添加长期记忆提醒
    if "api_fail_history" in state:
        base_prompt += f"\\\\n[记忆] 该API近7天失败率{state['api_fail_history']}%，建议备用方案"
    return base_prompt

```

 3.3 记忆系统如何构建？

我们可以以一个最简对话系统为例，结合短期、中期、长期记忆进行分层处理：

*   短期记忆：聚焦当前对话的最近信息，直接将最新消息体输入模型。这类似大脑对当下信息的清晰即时记录，确保模型能响应最新对话内容。
    
*   中期记忆：由于模型存在阈值容量限制，需对当前对话中的关键信息进行摘要处理。这如同大脑在记忆容量有限时，自动将知识压缩为核心要点（非逐字记忆，而是保留关键内容），让模型抓住对话的核心逻辑。
    
*   长期记忆：通过 RAG（检索增强生成）等技术，在用户提问时调用外部知识库，提取已沉淀的长期记忆内容。这一机制类比大脑调用过往经验与知识储备，为对话提供更丰富的背景信息支撑。
    

由于大模型的阈值始终有限，消息体在一定的对话回复轮次之后会超出大模型的上下文窗口限制。最简单的做法直接如基于时间衰减（近期对话优先保留）或重要性排序（关键信息优先），避免记忆冗余。下标给出一些常见的方法，更复杂的系统可以研究下 memoryOS 等架构。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxOhozPDiasrcNsShZNge8qjLibSgKQfHD2pD4u5ApwxGxxP9B8q8ZUIfw/640?wx_fmt=png&from=appmsg#imgIndex=30)

 3.4 如何搭建基于 function call 的大模型 Agent 应用？

核心逻辑：大模型负责 “理解问题 + 安排任务”，外部工具负责 “具体执行”，最后模型把结果包装成你能听懂的话。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxuO8VvjMFRws0Fxzwb3vwgcicJqzDPeYm5mzRsPq8bNh0iajrwhZfTsvQ/640?wx_fmt=png&from=appmsg#imgIndex=31)

用户发起请求

（User→应用系统）：用户向应用系统提交自然语言问题（如 "查询上海天气"）。

提示词编排（应用系统→LLM）：应用系统将问题转化为结构化提示词，调用大模型并请求决策。

工具调用判断（LLM 决策分支）：

无需工具：LLM 直接生成文本回复（如回答常识问题）。

需工具调用：LLM 输出工具名称及参数（如 WeatherAPI: {location: "上海"}）。

安全审批（应用系统→Host 系统）：工具调用需经权限校验与风险审核，通过后触发执行确认。

函数执行（Host 系统）：Host 系统按参数调用目标工具（如天气 API），获取原始结果（如 JSON 数据）。

结果整合（逆向传递链）：工具结果经 Host 系统→应用系统→LLM 逐层返回，LLM 将数据转化为自然语言（如 "上海今日多云，28°C"）。

最终响应（应用系统→User）：应用系统将 LLM 生成的友好回复返回给用户，流程终止。

怎么实现函数调度的功能？

整体其实就是提示词来激发模型选择工具的能力，目前开发过程中主要有两种实现方式：一种是直接使用大模型 API 提供方提供的利用 tools 和 messages 的组织方式（其实背后他们自己也是通过 prompt 来组建的一套能力，不过相比自己开发一套提示词工程而言，他们更了解底层模型如何做的，在通用化场景下的适配效果可能比自己研究要好很多。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxrjuNYdRJegDtKAftZpISfHQnQ9Bdeaby5rymVcAje6bDSRXQmxEkDw/640?wx_fmt=png&from=appmsg#imgIndex=32)

引阿里云官网提供的函数调用的流程图解

1）使用官方提供的服务 + openai 接口访问方式：快速使用推荐！

简版代码如下，但提前需准备好 tools 的定义（见下文 tools 的样式小节）和 messages 数组（见下文 messages 该如何组织部分，后续推文会有更详细的介绍）。

```
from openai import OpenAI
import os
client = OpenAI(
    # 若没有配置环境变量，请用提供商提供的api_key="sk-xxx",
    api_key=os.getenv("API_KEY"),
    base_url="BASE_URL",
)
def function_calling():
    completion = client.chat.completions.create(
        model="qwen-plus",  # 此处以qwen-plus为例
        messages=messages,
        tools=tools # 这里tools的结构体可以见下文tools的样式
    )
    print("返回对象：")
    print(completion.choices[0].message.model_dump_json())
    print("\\\\n")
    return completion
print("正在发起function calling...")
completion = function_calling()

```

2）prompt + 系统自实现：** 完全小白不建议用这个哈，相对自适配能力更强，但很考验对底层模型的理解。

上图提供了函数调用流程的图解，实际自己用 prompt 开发流程与上图一致。用户提问的时候将工具内容和问题组织为一定的提示词给到大模型，然后写一段拆解函数拆解大模型返回内容里面携带的工具调用信息和其他文本内容。

 3.5 如何搭建基于 MCP 协议的大模型 Agent 应用？

相比仅基于 function call 的应用而言，其实就是引入了 mcp 来管理工具资源，除此之外其他的都是一样的。下面给出了工作流程图解。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRx1Va0wyK2uaUF816vXW7ez036GnzrBibAxhAb9cLohald0QVRAnWOTIg/640?wx_fmt=png&from=appmsg#imgIndex=33)

基于 MCP 协议的大模型 Agent 应用的工作流程图解。流程始于用户发送请求（步骤 1），Host 系统将问题及工具列表传递给 MCP-Client（步骤 2），随后 LLM 生成工具调用参数（步骤 3）并经审批确认（步骤 4-5）。MCP-Server Tool 执行具体操作（步骤 6-7）后，结果通过多级传递（步骤 8-10）最终由 LLM 生成响应（步骤 11）返回给用户（步骤 12）。整个架构突出了 MCP 协议在协调用户、主机系统、大语言模型和工具服务之间的核心枢纽作用，体现了模块化分工与安全审批机制的设计理

1) User→Host 系统 用户向 HOST 系统发送一个需要处理的问题（自然语言)

2) Host 系统→MCP-Client Host 系统将用户原始问题（自然语言）和可调用工具列表（如 API、数据库接口等）打包发送至 MCP-Client，触发工具规划流程。

3) MCP-Client→LLM MCP-Client 请求 LLM 分析用户问题，生成结构化工具调用方案（包括工具选择、输入参数等），例如解析 "查询北京天气" 为调用 WeatherAPI 的参数 {location: "北京"}。

4-5) MCP-Client→Host 系统（审批闭环） MCP-Client 提交工具调用申请至 Host 系统，触发安全审批（如权限校验 / 风险过滤），通过后返回确认执行指令，确保操作合规性。

6) Host 系统→MCP-Server Tool Host 系统向 MCP-Server Tool 发送工具执行命令（如 REST API 调用指令），携带 LLM 生成的标准化参数。

7) MCP-Server Tool 执行 工具服务执行具体操作（如调用天气 API、数据库查询等），生成原始执行结果（如 JSON 格式的天气数据）。

8-10) 结果逆向传递链 MCP-Server Tool→Host 系统→MCP-Client→LLM，逐层返回工具执行结果，数据经校验和格式化传递，保持上下文一致性。

11) LLM 生成最终响应 LLM 将工具返回的原始数据转化为用户友好的自然语言（如 "北京今日晴，25°C"），并补充逻辑推理或多工具结果融合。

12) Host 系统→User Host 系统将 LLM 生成的最终响应返回给用户，完成闭环。流程中 MCP 协议通过标准化接口和审批机制，确保多主体间安全、高效的协同。

 3.6 如何搭建多 Agent 应用？

多 Agent 可以当做是上下文工程的增强。具体本文不详细叙述，引 claude 的经验贴：https://www.anthropic.com/engineering/built-multi-agent-research-system 

  

  

# 04

  

  

Agent 如何评估？

关于这个问题目前就调研来看，没有比较完善的评估方式。调研的 bechmark 跟大家分享下，调研角度上涵盖数据分析场景与多轮对话 / 工具交互领域的代表性 Benchmark 数据集：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95y4F4O80UVRyQkVhqeYSRxH8YsFCf8hpNKOpDhYAiaIibXg4l7mItPkOX0gy51ic1SepUwyfvc0VU8g/640?wx_fmt=png&from=appmsg#imgIndex=34)

复杂的文本 + 图（非纯答案）场景，若追求严格准确性，仍需人工介入。

参考文献

[1] 2023. AgentBench: Evaluating LLMs as Agents. arXiv preprint arXiv:2308.03688. https://arxiv.org/abs/2308.03688 

[2] InfoQuest: Evaluating Multi-Turn Dialogue Agents for Open-Ended Information Seeking. arXiv preprint arXiv:2502.12257. https://arxiv.org/abs/2502.12257 

[3] Wang, X., et al. (2023). MINT: A New Benchmark Tailored for LLMs' Multi-Turn Interactions. Blog post. https://xwang.dev/blog/2023/mint/ 

[4] ToolBench: An Instruction-tuning Dataset for Tool Use. Papers With Code. https://paperswithcode.com/dataset/toolbench 

[5] GTA: A Benchmark for General Tool Agents. arXiv preprint arXiv:2407.08713. https://arxiv.org/abs/2407.08713 

[6] ToolDial: Multi-turn Dialogue Generation Method for Tool-Augmented Language Models. arXiv preprint arXiv:2503.00564. https://arxiv.org/abs/2503.00564v1 

[7] AgentBoard: An Analytical Evaluation Board of Multi-turn LLM Agents. NeurIPS 2024 Poster. https://proceedings.neurips.cc/paper_files/paper/2024/file/877b40688e330a0e2a3fc24084208dfa-Paper-Datasets_and_Benchmarks_Track.pdf 

[8] WorkBench: a Benchmark Dataset for Agents in a Realistic Workplace Setting. arXiv preprint arXiv:2405.00823. https://arxiv.org/abs/2405.00823 

[9] DataSciBench: An LLM Agent Benchmark for Data Science. arXiv preprint arXiv:2502.13897. https://arxiv.org/abs/2502.13897 

[10] A Survey of Large Language Models

[11] understanding the planning of LLM agents: A survey. https://arxiv.org/pdf/2402.02716 

[12] MemoryOS

[13] AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation

[14] LangChain: Building applications with LLMs through composability [15] 【上下文工程】Mei, L., Yao, J., Ge, Y., Wang, Y., Bi, B., Cai, Y., ... & Liu, S. (2025). A Survey of Context Engineering for Large Language Models. arXiv preprint arXiv:2507.13334. https://arxiv.org/abs/2507.13334 

[16] LangChain 官方文档. https://python.langchain.com/docs/get_started/introduction 

[17] AutoGen GitHub 仓库. https://github.com/microsoft/autogen 

[18] MetaGPT GitHub 仓库. https://github.com/geekan/MetaGPT 

[19] CrewAI GitHub 仓库. https://github.com/crewAIInc/crewAI 

[20] BabyAGI GitHub 仓库. https://github.com/babyagi/babyagi 

[21] LlamaIndex 官方文档. https://www.llamaindex.ai/ 

[22] Semantic Kernel GitHub 仓库. https://github.com/microsoft/semantic-kernel 

[23] Auto-GPT GitHub 仓库. https://github.com/Significant-Gravitas/Auto-GPT 

[24] AgentGPT 官方网站. https://agentgpt.reworkd.ai/ 

[25] Langroid GitHub 仓库. https://github.com/langroid/langroid 

[26] Haystack 官方文档. https://haystack.deepset.ai/ 

-End-

原创作者｜谢苑珍

感谢你读到这里，不如关注一下？👇

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe951ia9iadG3cGPp3OjMQBY8jUDyMQB9NRlcpN0NbibgksMBfHCS5aeo3P2y0RInfFicPmeIqibvgic9wBxA/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=11)

📢📢来抢开发者限席名额！点击下方图片直达👇

![](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe97xxBnxOESLsT50U66W2y6IhGsLUSzWYw0IxYo5l5rn59xtcodquyPzYE7KGQcAmZe8D89eHx9cyg/640?wx_fmt=jpeg&from=appmsg#imgIndex=36)

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe975eiakGydXqTICibuXvLhyqN5sicc7ia7Cvb8nJGK2gjavrfIIYr5oicm20W8hFPvUdSm8UTzzWiaFco9Q/640?wx_fmt=png#imgIndex=37)

你对本文内容有哪些看法？同意、反对、困惑的地方是？欢迎留言，我们将邀请作者针对性回复你的评论，同时选一条优质评论送出腾讯云定制文件袋套装 1 个（见下图）。11 月 25 日中午 12 点开奖。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe96Ad6VYX3tia1sGJkFMibI6902he72w3I4NqAf7H4Qx1zKv1zA4hGdpxicibSono28YAsjFbSalxRADBg/640?wx_fmt=png#imgIndex=38)

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe979Bb4KNoEWxibDp8V9LPhyjmg15G7AJUBPjic4zgPw1IDPaOHDQqDNbBsWOSBqtgpeC2dvoO9EdZBQ/640?wx_fmt=png#imgIndex=39)

[

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95V5JIRxicdvMKeRpZAe5Zqp9S9pJaavWEicakR1ZsjeXzEoJog9BNFQS9icLraHVh0uCSADJFYA1poA/640?wx_fmt=png#imgIndex=40)

](https://mp.weixin.qq.com/s?__biz=MzI2NDU4OTExOQ==&mid=2247692226&idx=1&sn=9abde171b9b77326eba39da4e9d0f3ca&scene=21#wechat_redirect)

[

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe951ia9iadG3cGPp3OjMQBY8jUBwh4MBSTjwD7NIF5KIOXLbPI3quyt8wEcdkylgcGREcAM8uOd7eiatA/640?wx_fmt=png#imgIndex=41)

](https://mp.weixin.qq.com/s?__biz=MzI2NDU4OTExOQ==&mid=2247691682&idx=1&sn=8eac13cc848ee0779e1a4ab85142d87a&scene=21#wechat_redirect)

[

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe951ia9iadG3cGPp3OjMQBY8jUVU4nJ5icTPibsgNT17xNWyHEmhjJ1wT2tPlwC3VRFCjKAQdxwvic0xZTQ/640?wx_fmt=png#imgIndex=42)

](https://mp.weixin.qq.com/s?__biz=MzI2NDU4OTExOQ==&mid=2247691610&idx=1&sn=99c2ab7608d2ed7edc1e1aa029dd08ed&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95pIHzoPYoZUNPtqXgYG2leyAEPyBgtFj1bicKH2q8vBHl26kibm7XraVgicePtlYEiat23Y5uV7lcAIA/640?wx_fmt=png#imgIndex=43)