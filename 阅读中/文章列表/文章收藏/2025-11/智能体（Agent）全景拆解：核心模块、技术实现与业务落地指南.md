---
source: https://mp.weixin.qq.com/s/h0LuAvtqGug-0DBy9iVK8g
create: 2025-11-23 22:29
read: false
knowledge: false
---
# 1 什么是 Agent

**智能体（Agent）** 是指能够**感知环境**、**进行决策**并**采取行动**以实现某种目标的系统。

现在我们把 Agent 想象成一个人，这样便于我们更好的理解，比如说我们走在路上，看到对面有辆车开过来了，这个时候我们本能的会去躲避，那整个过程是怎么发生的呢？

看到对面有车开过来，这是通过眼睛看到信息，通过神经传递给大脑，大脑调动之前的记忆发现如果这个时候不躲开，就有可能会被车撞，然后驱动着脚走到旁边去。

上面的过程就是感知路面环境、大脑决策最后采取行动的过程。基于上面的过程我们可以看出，Agent 会包含以下核心模块。

# 2 Agent 核心模块

见下图，不同的机构或团队对 Agent 的核心模块划分不同，但都包含了**感知**、**记忆**、**规划**和**行动**四个模块。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PfMGv3PxR7icm1MYzVh0Qls6ib7f2TGx4XHpf62GjY3bF3YFgUHxaww1YnIpYeVzX3V6iczmrT4QEprwnPmlsFQiaA/640?wx_fmt=png&from=appmsg#imgIndex=0)

## 2.1 LLM（大语言模型）

先简单说下大语言模型，它是智能体的核心，相当于人的大脑，背后的神经网络设计也是基于人类大脑的神经系统，尤其是神经元（Neuron）之间的连接与信号传递方式。

关于大语言模型就不再详细说了，大家基本每天都在用。然后还是想着重说下深度学习，因为它与传统的软件研发流程不一样。就像 AI 大神卡帕西（_斯坦福读书时的导师是李飞飞, OpenAI 的创始成员之一，后面被马斯克亲自招募，出任特斯拉的 AI 总监，之后又回到 OpenAI，去年，他正式宣布离开 OpenAI_）最近的一个演讲中，提到的一个概念，叫 **“软件 3.0”**。

*   **软件 1.0**，是我们最熟悉的传统软件研发流程，指的是由人类程序员一行行编写代码的方式，我们之前的研发流程都属于软件 1.0。
    
*   **软件 2.0**，是随着深度学习的兴起而出现的。它的核心已经不再是人工编写的代码，而是通过训练来控制神经网络的权重。这个阶段，很多事情不再依赖程序员去手动写逻辑，而是依靠数据驱动模型进行 “学习”，而且泛化能力也很好。
    
*   **软件 3.0**，就是我们现在通过 cursor、trae、winself 等 AI 开发工具用自然语言进行编程，它大大降低了软件创造的门槛。
    

但仔细想想我们好像直接从 1.0 跳到了 3.0，并没有经过 2.0，但实际上软件 2.0 的应用其实还是蛮多的，尤其是一些垂直行业，如医疗健康，让机器学习看片子，帮助医生看病；如农业领域，能够预测作物产量和优化种植策略；还有教育行业，能够个性化学习路径和提高学习效率等等。

所以我觉得大家有兴趣可以了解了解。推荐两本书：**《深度学习革命》、《深入浅出神经网络与深度学习》**。

## 2.2 Perception（感知）

即智能体对环境的感知，一个人可以通过眼睛、耳朵来感知环境，机器人可以通过雷达和摄像头感知环境，软件中主要是基于我们输入的上下文数据，可以是**文本、图片、视频、语音、文件**等等。

## 2.3 Memory（记忆）

大模型其实是没有记忆的，只能通过外部技术（如 RAG、ES）模拟记忆效果，以增强任务表现和交互连贯性。

分为**短期记忆（Short-term）** 和 **长期记忆（Long-term）**。个人认为记忆对于大模型来说非常重要，因为随着对模型的不断使用，如果模型厂商如果把这些使用的记录全部变成记忆，那模型就会越来越懂我们，使用起来也会越来越顺手，而不像现在，每次在提示词上要还很多时间，而且换个窗口后又得重来一次。

#### 2.3.1 短期记忆依赖模型上下文的 token 限制

参考代码如下， 在每次调用大模型的时候，需要将工具返回的信息、人的反馈信息等都塞到 messages 中，然后去调用大模型接口。

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line
interface ChatMessage {
  role: 'user' | 'assistant' | 'system' ｜ 'tool';
  content: string;
  tool_calls?: xxx;
  call_id?: string;
}
class Memory {
  private messages: ChatMessage[] = [];
  constructor() {
    this.messages = [];
  }
  addMessage(message: ChatMessage) {
    this.messages.push(message);
  }
  getMessages() {
    return this.messages;
  }
}

```

#### 2.3.2 长期记忆依赖于外部组件（如 RAG， elasticsearch）

可以通过文件或数据库来做存储介质，可以搜索、更新, 长期记忆又可以继续分，如用户画像记忆，情景记忆，事实记忆等。

*   接下来讲下在蛋糕烘培助手案例中是如何处理记忆的，实时的对话信息作为工作记忆，存在内存中，到达一定阈值后，由大模型来提取关键信息来作为短期记忆。然后上半年商机挖掘项目能够分析出客户画像，将这部分信息作为长期记忆存储起来，后续的对话，每次调用大模型的时候会进行召回并更新短期记忆的召回次数。这些短期记忆会进行周期性的评估（包括召回次数和重要性），重要的会合并进核心记忆，不重要的进行遗忘。
    
*   整体会分三层：工作记忆、短期记忆、 长期记忆
    
    ![](https://mmbiz.qpic.cn/sz_mmbiz_png/PfMGv3PxR7icm1MYzVh0Qls6ib7f2TGx4XzDzru6AjiblGxUAM7KyPbUUufvJSoWE2UHyC8R1Le7MbxAVbOUYibaLA/640?wx_fmt=png&from=appmsg#imgIndex=1)
    
*   每次询问通过向量检索更新短期记忆召回次数
    
*   短期记忆由大模型来进行提取为边缘记忆，并设置重要性评分
    
    ![](https://mmbiz.qpic.cn/sz_mmbiz_png/PfMGv3PxR7icm1MYzVh0Qls6ib7f2TGx4XAAO5GMMzatNwq0Kicxw65v28pDVAabzibgLkAhBNCpKORNmhqYsyYibmw/640?wx_fmt=png&from=appmsg#imgIndex=2)
    

## 2.4 Action（行动）

行动很好理解，LLM 相当于我们的大脑，但是光有大脑，没有手脚也做不成事，所以 Agent 就需要借助外部的工具进行行动。

我们需要把我们的工具（包括名称、描述和需要的参数）告诉 LLM，当 LLM 来决定是否用工具，以及什么时机用工具。

介绍下案例中使用的商品查询工具，即用户通过一个问题或图片就能找到商家后台商品库的商品

*   商品的关键词查找处理：
    

*   将商家后台的商品图片和标题通过多模态模型提取出关键信息；
    
*   大模型会提取出关键信息，然后让工具执行，在执行过程中将关键词和商家后台商品的关键信息项进行相似度检索，从而找出匹配的；
    

*   图片的查找处理：
    

*   将商家后台的商品图片向量化；
    
*   模型提取出用户想要的蛋糕商品图片链接，然后和商家后台商品图片进行相似度匹配；
    

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line
// 定义tools
const tools = [
    {
        "type": "function",
        "function": {
          "name": "query_goods",
          "description": "查询蛋糕商品信息",
          "parameters": {
            "type": "object",
            "properties": {
              "keywords": {
                "type": "array",
                "items": {
                  "type": "string",
                  "description": "蛋糕商品关键信息项",
                },
                "description": "蛋糕商品关键信息，包含适用场合、目标人群、风格、口味、主题、颜色等",
              },
              "image_url": {
                "type": "string",
                "description": "蛋糕商品图片链接",
              },
            }
          },
       }
    },
    ...
];

```

## 2.5 Planning（规划）

规划这部分相对会复杂点，接下来主要将常见的两种规划模式，分别是 **ReAct** 和 **Plan-and-Resolve**，除了这两种模式还有**多 Agent 模式**等。

#### 2.5.1 ReAct（Reason + Action 推理 - 行动 - 反馈）

通过不断的**推理、行动和反馈**来最终完成用户的目标。

蛋糕烘培助手的流程如下图：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PfMGv3PxR7icm1MYzVh0Qls6ib7f2TGx4XJUrzL4bsvgASffw4xy5ibCjtbBEbkVFCnpdo0qj9evegYnh1FjDaR7Q/640?wx_fmt=png&from=appmsg#imgIndex=3)

#### 2.5.2 Plan-and-Solve

**将任务分解成子目标并分布执行**，这个和 ReAct 的区别是：Plan-and-Solve 是先通过大模型将任务拆解，然后再分布执行，和 ReAct 是一步一步的去执行，两种模式适用的场景不一样。

比如说旅游规划的 agent，就可以用 Plan-and-Solve 模式，先生成计划（如要了解天气、要了解住宿等），然后再解决计划， 最终整合让大模型完成回答， 当然中间也可以加反思，对计划进行反思等。

下面是基于 Plan-and-Solve 的流程图：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PfMGv3PxR7icm1MYzVh0Qls6ib7f2TGx4Xsgql0J7ygoEKdwPyiamIm3kvwtIqQjicWVa1r5LXgGdq4zV7LGGyzZXA/640?wx_fmt=png&from=appmsg#imgIndex=4)

# 3 Agent 评估报告

评估报告是 Agent 开发过程中不可或缺的环节，包括**响应时间、完成度、工具使用情况、后续优化**等，它能够系统性地收集和分析 Agent 的运行数据, 为后续的优化和迭代提供科学依据。

所以在 agent 的设计环节都需要将这些数据都记录下来。

# 4 Agent 成本分析

在做 AI 应用的过程中，我们经常会被问到一个问题，做个东西的成本怎么样？

AI 相关应用与传统的应用之间的计费方式完全不一样了，所以 Agent 中每次的模型调用，所以针对 Agent 需要做成本分析。

基于成本分析可以进行价值回顾和 agent 的优化。

# 5 Agent vs Workflow

用一句话来概括它们之间的区别就是：**Agent 是你只需要告诉我你要什么，不需要告诉我怎么做，而 workflow 就是你要按照我编排好的路径来执行**。一个是基于大模型来规划，一个是基于人来规划。

如何选择？

*   **选 Workflow**
    

*   数据处理管道
    
*   文档审批流程
    

*   任务流程相对固定，步骤可预定义
    
*   需要严格的执行顺序和质量控制
    
*   要求高可靠性和可重复性
    
*   涉及多个系统集成，需要标准化流程
    
*   合规要求严格，需要审计追踪
    
*   如：数据处理管道、文档审批流程
    

*   **选 Agent**
    

*   智能客服助手
    
*   个人 AI 助理
    
*   复杂问题诊断
    

*   任务需要动态决策和灵活应对
    
*   环境不确定性高，需要自主判断
    
*   要求个性化和上下文理解
    
*   需要创造性解决问题
    
*   用户交互频繁，需要对话式体验
    
*   如：智能客服助手、个人 AI 助理、复杂问题诊断
    

# 6 FM Agent 与 RL Agent

以上讲的都是 FM Agent，是基于大型**预训练模型（Foundation Model） 的智能体**，比如基于 GPT-4、Claude、Gemini 等大模型构建的智能体。

核心能力是利用大语言模型（LLM）的理解、推理、规划和生成能力来感知世界、做决策、执行任务。

除了 FM Agent，还有 RL Agent，它是基于**强化学习（Reinforcement Learning） 原理构建的 Agent**。

它与环境交互，根据奖励信号学习最优策略，以最大化长期累计奖励。如 DeepMind 的 AlphaGo Zero，在围棋游戏中击败了世界顶级选手，就是一个 RL Agent。

# 7 最后

今天的分享只是掀开了 Agent 世界的一角。它的深度与潜力远不止于此，新的思路、框架与落地案例正层出不穷。希望这份经验与尝试，能为大家点亮一条更高效、更有趣的道路。

回过头来看，不论是 LLM、Workflow 还是 Agent，终究只是工具。只有当它们与真实业务场景发生化学反应，切实解决痛点，创造价值时，技术的光芒才会真正被点亮。愿我们带着好奇与敬畏，把技术的锋利，打磨成改变世界的钥匙。

文末福利

有赞研发团队前端、后端、客户端、测试、AI 开发、运维开发、数据开发等多个岗位招聘中，看机会的伙伴可以将简历发送至 zhuzhuyun@youzan.com。