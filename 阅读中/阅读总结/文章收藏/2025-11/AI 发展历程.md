---
source:
  - "[[这大概是我见过最通俗易懂的 AI 发展历程科普详文了]]"
create: 2025-11-24
---

## 1. 核心知识点

### 1.1. AI 的定义与核心能力

- **人工智能 (AI)**: 让机器模拟人类智能的技术总称。
- **核心能力**: 让机器具备“**感知、思考、决策、执行**”的能力。
    - **感知**: 如眼睛、耳朵，获取外界信息。
    - **思考**: 分析和推理信息。
    - **决策**: 从多个选项中做出决定。
    - **执行**: 实施决策，采取行动。

![过马路示意图](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j3gficicyOvavSZsbSmdnMumMXZibBPGboKlEJElAbKnXGkibBDuWW8ialrwYlVmdK0yYsz0iaSaIVWzCjBXQwvtgAug/640?wx_fmt=jpeg&from=appmsg#imgIndex=3)

### 1.2. AI 发展三阶段

#### 1.2.1. 初生期 (1956-1989)：规则式 AI

- **特征**: 机器死板地按人类制定的规则执行，不懂变通。
- **核心技术**: **自然语言处理 (NLP)** 的早期应用，基于固定的词典和语法规则。
- **缺陷**: 缺乏灵活性和“语感”，无法处理规则之外的情况。
- **类比**: 一个只会死记硬背的**小学生**。

#### 1.2.2. 成长期 (1990-2016)：统计式 AI

- **特征**: 机器从大量数据中自己学习规律，而非依赖固定规则。
- **核心技术**: **机器学习 (Machine Learning, ML)**。
    - **AI 模型 (Model)**: 从数据中提炼出的“规律”或“经验”。
    - **监督学习 (Supervised Learning)**: 给机器带有明确“标签”的数据进行训练。
- **缺陷**: 模型能力局限于训练数据的领域，存在“偏科”问题。
- **类比**: 一个靠刷题总结规律，但容易偏科的**中学生**。

![垃圾邮件技术原理](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j3gficicyOvavSZsbSmdnMumMXZibBPGboK7R2ia9buXT9iaV183ibZEgG7niaGduQibnj2wQxLYEDRUodazK7WsJ83P5g/640?wx_fmt=jpeg&from=appmsg#imgIndex=9)

#### 1.2.3. 爆发期 (2017-至今)：深度学习 / 大模型 AI

- **特征**: 模型参数和数据量巨大，具备通用和跨领域知识，能力全面爆发。
- **关键技术突破**:
    - **Transformer 架构 (2017)**: 核心是 **自注意力机制 (Self-Attention Mechanism)**，能并行处理信息并理解全局上下文，是引爆 AI 爆发期的基石。
    - **大模型 (Large Model)**: 参数规模巨大（通常指 100 亿以上）的模型，如 GPT 系列。
    - **深度学习 (Deep Learning)**: 使用深度神经网络（如 Transformer）进行机器学习。
- **重要概念**:
    - **多模态 (Multimodal)**: 模型能同时处理多种类型的信息（如文本+图像）。
    - **单模态 (Unimodal)**: 模型只能处理单一类型的信息（如纯文本）。
      
    ![“单模态”和 “多模态” 对比](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvavSZsbSmdnMumMXZibBPGboKsw84icmELywQKdCzq7oRibZFpUUbrlaRWzbAHpvUob6gJRL3FvHBC6bg/640?wx_fmt=png&from=appmsg#imgIndex=18)
- **类比**: 一个博览群书、拥有实习经验的**大学生**。

### 1.3. AI 应用与进阶

#### 1.3.1. 智能体 (Agent)

- **定义**: 能够感知环境、进行决策，并**自主采取行动**以实现目标的系统。
- **与大模型的关系**:
    - **大模型是“大脑”**: 提供知识和思考能力。
    - **智能体是“手和脚”**: 将思考转化为行动，完成复杂任务。

![“智能体” 演进](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvavSZsbSmdnMumMXZibBPGboKn2McqXyAic432nM2G0IeBbofSrkiaYNLyXO5QLZZZAsJRVffnvK3BDTA/640?wx_fmt=png&from=appmsg#imgIndex=20)

#### 1.3.2. 智能体调优与关键问题

- **目标**: 提升 AI 输出的准确性、相关性和可靠性。
- **核心调优方法**:
    - **提示词工程 (Prompt Engineering)**: 编写清晰、具体的指令。
    - **检索增强生成 (RAG)**: 让 AI 在回答前先查阅外部知识库（**开卷考试**），以提供更准确、有时效性的信息。
    - **微调 (Fine-tuning)**: 在预训练好的大模型基础上，用特定领域的数据进行二次训练，使其更专业。
- **关键挑战**:
    **幻觉 (Hallucination)**: AI 一本正经地胡说八道。这是当前大模型的通用问题，可通过上述调优方法缓解，但难以完全根除。

### 1.4. 未来展望

- AI 将从“工具”演变为人类的“伙伴”。
- 发展离不开**数据、算力、算法**三大基石。
- 未来方向包括 **AGI (通用人工智能)**、**具身智能**等，AI 将更深入地与物理世界结合。