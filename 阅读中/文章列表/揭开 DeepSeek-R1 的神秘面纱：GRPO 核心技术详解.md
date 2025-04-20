---
source: https://mp.weixin.qq.com/s/jlduUciOX00mTpcvy-rX2g
create: 2025-04-17 21:41
read: false
tags:
  - 大模型
---
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/j3gficicyOvasVeMDmWoZ2zyN8iaSc6XWYj79H3xfgvsqK9TDxOBlcUa6W0EE5KBdxacd2Ql6QBmuhBJKIUS4PSZQ/640?wx_fmt=gif&from=appmsg)

相信各位做 LLM 的朋友春节期间都已经被 DeepSeek-R1 给刷屏了，如果有阅读过 DeepSeek-R1 技术报告的朋友肯定都被其中的一个核心技术 **GRPO** 给吸引了，那么今天就来给大家过一过强化学习的瘾，仔细解读一波。

作者：ethan

### GRPO 技术背景

GRPO 技术其实不是在 DeepSeek-R1 中提出，早在 DeepSeek 去年 2 月发的一篇论文《DeepSeekMath: Pushing the Limits of Mathematical Reasoning in Open Language Models》中，他们就已经提出了 GRPO 技术，并用在 DeepSeekMath 模型上。为了更好的讲解这个算法的原理，我们首先得了解一些基本的强化学习概念，有相关基础的朋友可以跳过下一小节。

### 强化学习基本概念

在强化学习中，我们通常会讨论下面的一个问题背景：一个智能体（agent）在某个环境中可以执行一些动作（action），它在执行某个动作之后会从一个状态（state）切换到另外一个状态，同时所在的环境也会给它一个反馈（reward），而我们的目标则是最大化这个 agent 在环境中所能获得的 reward，如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvatzSJXTiaQRxaywsjCIIhrB4LyPLoWGYm0cUibibGx6aLYkyaCX3xKyGA8hKVPria6oGPzU4aAgkCa4bw/640?wx_fmt=png&from=appmsg)

img

上面的问题背景通常被称为马尔可夫决策过程（Markov decision process)，简称 MDP。

既然说是马尔可夫决策过程，相信不少敏感的朋友会回想起马尔可夫随机过程。马尔可夫随机过程的一个重要性质就是：下一个时刻的状态只取决于当前状态，而不会受到过去状态的影响，即状态转移概率

。而在 MDP 中，下一个时刻的状态取决于当前的动作和当前状态，即，因此被称为马尔可夫决策过程。

下面简单介绍一些非常重要的概念：

*   **S 为状态空间，即所有可能状态的集合**
    
*   **A 为动作空间，即 agent 所有可能动作的集合**
    
*   为智能体采取的策略（policy)，策略是一个函数，表示在输入状态情况下采取动作的概率。
    
*   是状态转移函数，表示在状态执行动作后到达状态的概率
    
*   为奖励函数，此奖励同时取决于状态和动作，并且还被称为即时奖励，因为是不考虑未来状态，只考虑当前动作和状态下，环境给予的奖励
    
*   为回报 (Return)，它表示从时刻状态开始，直到终止状态时，所有奖励的衰减之和：
    
    ，其中为折扣因子。
    
*   为价值函数，它通常用来衡量 agent 处于某个状态 s 下的预期收益（不仅包含即时奖励，还考虑未来的奖励），通常我们用表示在 MDP 中基于策略的 ** 状态价值函数（state-value function)**。
    
*   为动作价值函数 (action-value function)，用表示在 MDP 遵循策略时，对当前状态执行动作得到的期望回报：
    

同时状态价值函数和动作价值函数之间还存在以下关系：**在使用策略中，状态的价值等于在该状态下基于策略采取所有动作的概率与相应的价值相乘再求和的结果**

### 强化学习算法的种类

在强化学习问题中，根据是否知道环境相关的信息，我们通常把问题分为两类：model-based 和 model-free。model-based 是指我们能提前知道奖励函数和状态转移概率，反之 model-free 则二者都不知道。model-based 的问题解决有一个固定的套路可以使用，就是 dynamic programming（动态规划），这里就不细说了。

在现实世界中，model-free 形式的强化学习问题要更为常见。为了解决这类问题，我们通常有两类方法：一种是基于价值 (value-based) 的方法（例如 Q-learning 和 DQN），另一种是基于策略 (policy-based) 的方法。二者的主要区别是：基于值函数的方法主要是学习值函数，然后根据值函数导出一个策略，学习过程中不存在一个显式的策略；而基于策略的方法则是直接显式地学习一个目标策略。今天我们所要讲解的 GRPO 算法则是一种基于策略的方法。

### 从策略梯度算法说起

GRPO 算法本质上来说是从策略梯度算法发源而来，因此先给大家介绍相关概念以便于理解后面复杂的公式。

通常，在强化学习任务中，agent 会不断地与环境进行交互直至最终结束，根据上述过程，我们可以得到一条轨迹，表示为，其中表示状态, 表示行动，, 表示 agent 在状态 1 的时候选择了动作 1，后面的以此类推，如下所示：

假设 agent 的策略网络参数为，那么我们可以用表示一条轨迹产生的概率：

在 agent 与环境交互的过程中，还存在一个奖励函数，当输入状态和动作时，奖励函数会返回，我们把轨迹当中的所有奖励都加起来就得到了。策略梯度算法则是为了找到一个最优策略网络参数，使得越大越好，也就是最大化的期望：

为了最大化这个期望值，我们会采用梯度上升的方式进行参数更新：

而事实上，最后推导出来的梯度项没有办法直接计算，通常使用采样次轨迹的方式去计算得到梯度：

但是这样会带来一个明显的问题，数据利用效率不高。每次参数更新前，都需要采样大量的数据，训练的时间开销全都集中在了数据采样上。为了解决采样时间开销大的问题，我们可以使用**重要性采样**技术。

重要性采样通常被用来估计一个分布的期望值。从数学上来说，它的原理是通过不同分布的样本进行估计，然后乘上一个重要性权重（即两个分布概率的比值），这样可以无需从原分布中采样，用另一个简单分布的样本也能计算原分布的期望：

使用重要性采样的好处是，可以让我们使用旧策略采样的数据，多次更新策略网络参数，从而大大节省了训练中采样数据的时间开销：

事情到这里看起来比较完美了，但事实上，在使用过程中我们还会遇到一些问题：比如在很多游戏场景中，得到的奖励总是正的。如果所有动作的奖励均为正，那么策略更新时会盲目提高所有动作的概率（只是提升幅度不同），**但无法区分哪些动作比其他动作更好**，即**缺乏相对比较**。同时，还有可能引起高方差问题：原始奖励的绝对值差异极大，导致梯度更新的方差过大，训练不稳定。因此，我们通常希望奖励值有正有负，通过将奖励减去一个基线 b，我们就能达到目标。而这个新的奖励值我们称为优势函数，基线 b 我们通常取所有动作的平均预期奖励：

其中，表示在状态下执行动作的预期累积奖励，表示在状态下所有动作的平均预期奖励。最终，我们的优化目标便可以写成下面这样：

### TRPO 算法和 PPO 算法

在理想情况下，如果我们能够采样足够多的次数，那么使用 importance sampling 估计得到的梯度和原梯度是相等的。但实际上，我们并不能采样那么多次数据，因此 TRPO 算法引入了置信域（trust region）的概念，避免和 差异过大而导致估计不准确。具体来说，TRPO 算法给上面的优化目标加上了一个 KL 约束项：

由于上面的优化目标是一个带约束的优化问题，因此 TRPO 算法的求解会比较复杂，涉及到 KKT 条件，共轭梯度等方法，此处就不做介绍，感兴趣的朋友可以再去研究下。

TRPO 算法求解的难度主要来源于它的 KL 约束条件。熟悉凸优化的朋友可能知道，求解带约束问题和无约束问题难度相差可是非常大。那么有没有一种方法，让我们既可以约束和的差异程度，又不将它作为约束条件呢。PPO 算法采用将 KL 散度项直接加入到优化目标中，将有约束优化问题转化为无约束优化问题进行求解：

但是在实践中，上述算法又会引来另外一个问题，很难选择一个固定的常数，使算法的效果比较好。因此，PPO 算法又进行了进一步优化，引入了一个自适应的 KL 惩罚，给出一个 KL 的可接受区间，当 KL 项大于时，说明和 差异过大，就增大; 当 KL 项小于时，就适当减小。**这个算法被称为 PPO1 算法**。

是否还有更简单的方式去优化一个类似的目标呢？PPO 的作者们又提出了一个更简化的版本：

这里的的函数表示将限定在的区间内。**此算法被称为 PPO2 算法**。**在实践中发现，通常 PPO2 算法的效果会优于 PPO1 算法**。

### 语言模型中的 PPO 算法

上面都是在纯强化背景下的 PPO 算法，那么在 LLM 领域，PPO 算法又是如何应用的呢？通常来说，我们会构建四个模型：

*   Actor Model：这其实代表的就是 agent，在 LLM 领域里这就是我们需要训练的语言模型；
    
*   Critic Model：它的作用是预估总收益，目的是学习一个值函数;
    
*   Reward Model: 它的目标是计算即时收益，即学习奖励函数；
    
*   Reference Model: 它的作用是给语言模型增加一些约束，防止需要训练的语言模型训歪。
    

通常来说，在 RLHF-PPO 阶段，Actor/Critic model 都是需要训练的，而 Reward/Reference Model 则参数冻结。我们通常采用下面的方式去初始化这些模型：actor/reference model 采用 sft 阶段训练后的语言模型初始化，critic model 和 reward model 则采用语言模型的 backbone + 各自的 value head 来进行初始化。一个完整的 RLHF-PPO 训练过程如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvatzSJXTiaQRxaywsjCIIhrB4ASPBENfwibhWOhgzrJZqNzkqcBK3oOfuVga7FpKOibgp5yXSuRiaF810Q/640?wx_fmt=png&from=appmsg)

1.  将一个 batch 的 prompts 送进 Actor 语言模型，语言模型产生回答 responses；
    
2.  将 prompt + responses 送进 Critic/Reward/Reference Model，让他们生成用于计算 actor/critic loss 的数据。按照强化学习的术语，我们将这些数据称为经验；
    
3.  最后根据这些经验计算出 actor/critic loss，然后更新 actor 和 critic model。
    

其中，我们的 actor loss 为：

为了区分动作和优势函数，这里我们将优势函数记为。而这个 loss 正是我们的 PPO2 算法里的优化目标。Critic loss 则定义为预估预期收益和实际预期收益的 MSE loss。

### GRPO 算法

铺垫了这么多，终于到我们的核心 GRPO 算法了。

在上面的 PPO 算法中，我们通常会引入一个 critic model（或者称为 value model）来判断每个动作的优劣，从而改进策略。但它的引入也同时带来了两个问题：价值函数估计可能不准确，在 LLM 的语境里，通常只有一个完整的句子会容易给出一个好坏的判断，而对中间过程生成的 token，我们很难给一个准确的奖励值，而价值函数估计不准确则会导致学习策略变差；其次就是模型显存占用高，消耗计算资源大，因为 critic model 通常和 actor model 参数量差不多。为了解决这个问题，GRPO 算法直接在模型层面删掉了 critic model。如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvatzSJXTiaQRxaywsjCIIhrB43icXZZevDrDb1LDK8e0u41JFib7OMibMqBuDtMZ1WPVoWqGkf455qhs0g/640?wx_fmt=png&from=appmsg)

具体来说，GRPO 算法的流程如下：

1.  从数据集中采样一个 batch 的数据，对于中的每个问题，从旧策略模型中采样个输出；
    
2.  使用 Reward model 为每个采样输出计算奖励;
    
3.  在组内进行相对优势估计（group relative advantage estimation）来计算每个输出中第个 token 的优势；
    
4.  通过最大化 GRPO 目标函数来更新策略模型（即我们的语言模型），其中 GRPO 目标函数如下：
    
    ![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvatzSJXTiaQRxaywsjCIIhrB4ibGj7Q7s4BAvEriaoJ1CV0f6s08WMIBOfOBiaaC7WBm9gxm2uQqZoCP4g/640?wx_fmt=png&from=appmsg)
    
5.  在 RL 迭代中，基于策略模型的采样结果为 Reward model 生成新的训练集，利用回放机制持续训练旧 reward model。之后将参考模型设为当前策略模型，继续用更新后的奖励模型训练策略模型；（这一步可选）
    
6.  不断持续迭代，直到满足停止条件。
    

需要注意的是，GRPO 算法存在两种类型的奖励：过程奖励和结果奖励。过程奖励会对模型输出的每一个 token 进行奖励，而结果奖励则只会对一个 response 中的最后一个 token 输出奖励。在计算优势函数时，GRPO 算法与之前的 PPO 算法也有所不同，具体而言：它将奖励减去组平均奖励并除以组标准差进行归一化，将归一化后的奖励作为优势，以结果奖励为例，优势函数可以写为：

过程奖励也是类似，用 reward model 对输出的每一步打分，同样进行归一化，再计算每个 token 之后步骤的归一化奖励之和作为优势。

简单总结一下 GRPO 算法和 PPO 算法的不同点：

*   PPO 算法属于 actor-critic 类型的强化学习算法，需要训练一个与策略模型规模相当的价值函数作为基线，而 GRPO 算法丢弃了 critic model，放弃了价值函数近似，通过对同一问题采样多个输出，用这些输出的平均奖励作为基线；
    
*   优势函数的计算方式不同，PPO 算法采用的广义优势估计（GAE）基于奖励和学习到的价值函数计算优势, 而 GRPO 算法根据组内输出的相对奖励计算优势。
    

至此，我们也就讲完了 GRPO 算法，是不是也不太复杂。

### GRPO 算法在 R1 中的应用

在 DeepSeek-R1 模型发布之前，大部分研究认为使用过程奖励模型（PRM）要优于使用结果奖励（也被称为规则奖励，rule-based reward，仅通过规则评判回答的正确性并给出奖励）。这一观点在 DeepSeek 提出 GRPO 的论文中有所体现，OpenAI 所公开的研究成果也持类似看法。然而，DeepSeek-R1 的出现打破了常规认知。它证明了仅使用 rule-based reward 就能在推理任务中取得令人瞩目的成果。

而在 R1 的训练中，主要使用了以下两种的 rule-based reward：

*   **准确性奖励**：这一奖励模型专注于评估模型的响应是否正确。以数学问题为例，若问题有确定性的答案，模型必须按照指定格式给出最终结果，这样就能依据预设的规则验证答案的正确性。同样，在面对 LeetCode 编程题时，会借助编译器根据预先设定的测试用例生成反馈，以此判断模型给出的代码是否正确；
    
*   **格式奖励**：除了准确性奖励，DeepSeek-R1 还引入了格式奖励模型。该模型要求模型将思考过程严格置于和标签之间。
    

仅仅通过这两项 rule-based reward，并结合 GRPO 算法，DeepSeek-R1-Zero 的推理能力就能获得不断提升。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvatzSJXTiaQRxaywsjCIIhrB4voRMorIJjleHCMClocMs8AaibpqAxBRSfAM1vcLQOicBfib8rQiaFCCuIA/640?wx_fmt=png&from=appmsg)

但是，在训练过程中，作者们发现 DeepSeek-R1-Zero 存在着语言混杂的问题。因此，他们又重新设计了训练流程，并且引入了语言一致性奖励，最终得到了拥有强悍性能的 R1 模型。

**总结（DeepSeek 版）**

本文系统梳理了深度强化学习中策略优化算法的技术脉络与最新进展，重点探讨了从策略梯度（PG）到信任域策略优化（TRPO）、近端策略优化（PPO），再到创新性算法 GRPO（Group Relative Policy Optimization）的演进路径，并深入分析了其在大型语言模型（LLM）的应用价值。相信在不久的将来，RL 在 LLM 领域的成功应用也会引发在 CV 领域一些新的探索和发现。

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/j3gficicyOvasVeMDmWoZ2zyN8iaSc6XWYjZ7Hx6Udjjk2BGLzC9ahJq7ibxDd1RGA0c9NYZc1husEsvb3tY4FcWPQ/640?wx_fmt=gif&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasVeMDmWoZ2zyN8iaSc6XWYj5q5PQEOc5ibURPb03vnRibrxC3UR8xzdyATfiawTYRV2vJvBnAIcE1FeQ/640?wx_fmt=png&from=appmsg)