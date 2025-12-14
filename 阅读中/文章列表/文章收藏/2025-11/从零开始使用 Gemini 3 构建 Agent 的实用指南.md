---
source: https://mp.weixin.qq.com/s/jmm4CLjW_xeErZL9HOXKKw
create: 2025-11-25 16:31
read: false
knowledge: false
---
当你看到一个 AI Agent 编辑多个文件、运行命令、处理错误并迭代解决问题时, 似乎很复杂, 感觉就像魔法一样。但事实并非如此。构建 Agent 的秘密就是没有秘密。

Agent 的核心出奇地简单: 它是一个在循环中运行的大型语言模型 (LLM), 配备了它可以选择使用的工具。

如果你能用 Python 写一个循环, 你就能构建一个 Agent。本指南将引导你完成整个过程, 从简单的 API 调用到一个功能完整的 CLI Agent。

## 什么是 Agent?

传统的软件工作流是规范性的, 遵循预定义的路径 (步骤 A -> 步骤 B -> 步骤 C), 而 Agent 是使用 LLM 动态决定应用程序控制流以实现用户目标的系统。

一个 Agent 通常由以下核心组件组成:

*   模型 (大脑): 推理引擎, 在我们的案例中是 Gemini 模型。它通过模糊性进行推理, 规划步骤, 并决定何时需要外部帮助。
    
*   工具 (手和眼睛):Agent 可以执行的函数, 用于与外部世界 / 环境交互 (例如, 搜索网络、读取文件、调用 API)。
    
*   上下文 / 记忆 (工作空间):Agent 在任何时刻可以访问的信息。有效地管理这一点, 被称为上下文工程。
    
*   循环 (生命): 一个 while 循环, 允许模型: 观察 → 思考 → 行动 → 再次观察, 直到任务完成。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vbERicIdYZbBpib4NoXMboZI402nrFnySEt3GFaAM0rZNZmxmsXIgRlzcXKMhMu43NdGSrUIX24dK70HvfHM0r9Q/640?wx_fmt=other&from=appmsg#imgIndex=0)

几乎每个 Agent 的 "循环" 都是一个迭代过程:

1.  定义工具集: 你使用结构化的 JSON 格式向模型描述可用的工具 (例如 get_weather)。
    
2.  调用 LLM: 你将用户的提示和工具定义发送给模型。
    
3.  模型决策: 模型分析请求。如果需要工具, 它会返回一个结构化的 tool use, 包含工具名称和参数。
    
4.  执行工具 (客户端责任): 客户端 / 应用程序代码拦截这个 tool use, 执行实际的代码或 API 调用, 并捕获结果。
    
5.  响应和迭代: 你将结果 (tool response) 发送回模型。模型使用这些新信息来决定下一步, 要么调用另一个工具, 要么生成最终响应。
    

## 构建 Agent

让我们逐步构建一个 Agent, 从基本的文本生成到使用 Gemini 3 Pro 和 Python SDK 的功能性 CLI Agent。

前提条件: 安装 SDK(pip install google-genai) 并设置你的 GEMINI_API_KEY 环境变量 (**在 AI Studio 中获取 [1]**)。

### 步骤 1: 基本文本生成和抽象

第一步是创建与 LLM 的基线交互, 对我们来说是 Gemini 3 Pro。我们将创建一个简单的 Agent 类抽象来构建我们的代码, 并在本指南中进行扩展。我们首先从一个维护对话历史的简单聊天机器人开始。

```
from google import genai
from google.genai import types
class Agent:
    def __init__(self, model: str):
        self.model = model
        self.client = genai.Client()
        self.contents = []
    def run(self, contents: str):
        self.contents.append({"role": "user", "parts": [{"text": contents}]})
        response = self.client.models.generate_content(model=self.model, contents=self.contents)
        self.contents.append(response.candidates[0].content)
        return response
agent = Agent(model="gemini-3-pro-preview")
response1 = agent.run(
    contents="你好,德国最值得游览的前 3 个城市是什么?只返回城市名称。"
)
print(f"模型: {response1.text}")
# 输出: Berlin, Munich, Cologne
response2 = agent.run(
    contents="告诉我关于第二个城市的一些事情。"
)
print(f"模型: {response2.text}")
# 输出: Munich is the capital of Bavaria and is known for its Oktoberfest.

```

这还不是一个 Agent。它是一个标准的聊天机器人。它维护状态但不能采取行动, 没有 "手或眼睛"。

### 步骤 2: 赋予它手和眼睛 (工具使用)

要开始将其转变为 Agent, 我们需要工具使用或函数调用。我们为 Agent 提供工具。这需要定义实现 (Python 代码) 和定义(LLM 看到的模式)。如果 LLM 认为该工具将有助于解决用户的提示, 它将返回一个结构化的请求来调用该函数, 而不仅仅是文本。

我们将创建 3 个工具:read_file、write_file 和 list_dir。工具定义是一个 JSON 模式, 定义了工具的 name、description 和 parameters。

最佳实践: 使用 description 字段来解释何时以及如何使用该工具。模型严重依赖这些来理解何时以及如何使用该工具。要明确和清晰。

```
import os
import json
read_file_definition = {
    "name": "read_file",
    "description": "读取文件并返回其内容。",
    "parameters": {
        "type": "object",
        "properties": {
            "file_path": {
                "type": "string",
                "description": "要读取的文件路径。",
            }
        },
        "required": ["file_path"],
    },
}
list_dir_definition = {
    "name": "list_dir",
    "description": "列出目录的内容。",
    "parameters": {
        "type": "object",
        "properties": {
            "directory_path": {
                "type": "string",
                "description": "要列出的目录路径。",
            }
        },
        "required": ["directory_path"],
    },
}
write_file_definition = {
    "name": "write_file",
    "description": "使用给定内容写入文件。",
    "parameters": {
        "type": "object",
        "properties": {
            "file_path": {
                "type": "string",
                "description": "要写入的文件路径。",
            },
            "contents": {
                "type": "string",
                "description": "要写入文件的内容。",
            },
        },
        "required": ["file_path", "contents"],
    },
}
def read_file(file_path: str) -> dict:
    with open(file_path, "r") as f:
        return f.read()
def write_file(file_path: str, contents: str) -> bool:
    """使用给定内容写入文件。"""
    with open(file_path, "w") as f:
        f.write(contents)
    return True
def list_dir(directory_path: str) -> list[str]:
    """列出目录的内容。"""
    full_path = os.path.expanduser(directory_path)
    return os.listdir(full_path)
file_tools = {
    "read_file": {"definition": read_file_definition, "function": read_file},
    "write_file": {"definition": write_file_definition, "function": write_file},
    "list_dir": {"definition": list_dir_definition, "function": list_dir},
}

```

现在我们将 tools 和函数调用与系统指令一起集成到我们的 Agent 类中。

```
from google import genai
from google.genai import types
class Agent:
    def __init__(self, model: str,tools: list[dict]):
        self.model = model
        self.client = genai.Client()
        self.contents = []
        self.tools = tools
    def run(self, contents: str):
        self.contents.append({"role": "user", "parts": [{"text": contents}]})
        config = types.GenerateContentConfig(
            tools=[types.Tool(function_declarations=[tool["definition"] for tool in self.tools.values()])],
        )
        response = self.client.models.generate_content(model=self.model, contents=self.contents, config=config)
        self.contents.append(response.candidates[0].content)
        return response
agent = Agent(model="gemini-3-pro-preview", tools=file_tools)
response = agent.run(
    contents="你能列出我当前目录中的文件吗?"
)
print(response.function_calls)
# 输出: [FunctionCall(name='list_dir', arguments={'directory_path': '.'})]

```

太棒了! 模型已成功调用了工具。现在, 我们需要将工具执行逻辑添加到我们的 Agent 类中, 并循环将结果返回给模型。

### 步骤 3: 闭合循环 (Agent)

Agent 不是生成一个工具调用, 而是生成一系列工具调用, 将结果返回给模型, 然后生成另一个工具调用, 依此类推, 直到任务完成。

Agent 类处理核心循环: 拦截 FunctionCall, 在客户端执行工具, 并发送回 FunctionResponse。我们还向模型添加了 SystemInstruction, 以指导模型该做什么。

注意:Gemini 3 使用**思维签名 [2]** 在 API 调用之间维护推理上下文。你必须将这些签名原样返回给模型, 就像接收时一样。

```
# ... 步骤 2 中的工具和工具定义代码应该在这里 ...
from google import genai
from google.genai import types
class Agent:
    def __init__(self, model: str,tools: list[dict], system_instruction: str = "你是一个有帮助的助手。"):
        self.model = model
        self.client = genai.Client()
        self.contents = []
        self.tools = tools
        self.system_instruction = system_instruction
    def run(self, contents: str | list[dict[str, str]]):
        if isinstance(contents, list):
            self.contents.append({"role": "user", "parts": contents})
        else:
            self.contents.append({"role": "user", "parts": [{"text": contents}]})
        config = types.GenerateContentConfig(
            system_instruction=self.system_instruction,
            tools=[types.Tool(function_declarations=[tool["definition"] for tool in self.tools.values()])],
        )
        response = self.client.models.generate_content(model=self.model, contents=self.contents, config=config)
        self.contents.append(response.candidates[0].content)
        if response.function_calls:
            functions_response_parts = []
            for tool_call in response.function_calls:
                print(f"[函数调用] {tool_call}")
                if tool_call.name in self.tools:
                    result = {"result": self.tools[tool_call.name]["function"](**tool_call.args)}
                else:
                    result = {"error": "未找到工具"}
                print(f"[函数响应] {result}")
                functions_response_parts.append({"functionResponse": {"name": tool_call.name, "response": result}})
            return self.run(functions_response_parts)
        return response
agent = Agent(
    model="gemini-3-pro-preview",
    tools=file_tools,
    system_instruction="你是一个有帮助的编码助手。回应时像 Linus Torvalds 一样。"
)
response = agent.run(
    contents="你能列出我当前目录中的文件吗?"
)
print(response.text)
# 输出: [函数调用] id=None args={'directory_path': '.'} name='list_dir'
# [函数响应] {'result': ['.venv', ... ]}
# There. Your current directory contains: `LICENSE`,

```

恭喜你。你刚刚构建了你的第一个功能性 Agent。

### 阶段 4: 多轮 CLI Agent

现在我们可以在一个简单的 CLI 循环中运行我们的 Agent。创建高度功能性的行为所需的代码少得令人惊讶。

```
# ... 步骤 3 中的 Agent、工具和工具定义代码应该在这里 ...
agent = Agent(
    model="gemini-3-pro-preview",
    tools=file_tools,
    system_instruction="你是一个有帮助的编码助手。回应时像 Linus Torvalds 一样。"
)
print("Agent 已就绪。让它检查此目录中的文件。")
while True:
    user_input = input("你: ")
    if user_input.lower() in ['exit', 'quit']:
        break
    response = agent.run(user_input)
    print(f"Linus: {response.text}\n")

```

## Agent 工程的最佳实践

构建循环很容易; 使其可靠、透明和可控却很难。以下是源自顶级行业实践的关键工程原则, 按功能领域分组。

### 1. 工具定义和人机工程学

你的工具是模型的接口。不要只是包装你现有的内部 API。如果一个工具对人类来说很混乱, 那么对模型来说也很混乱:

*   清晰的命名: 使用明显的名称, 如 search_customer_database, 而不是 cust_db_v2_query。
    
*   精确的描述:Gemini 读取函数文档字符串以理解何时以及如何使用工具。花时间仔细编写这些, 这本质上是工具的 "提示工程"。
    
*   返回有意义的错误: 不要返回 50 行的 Java 堆栈跟踪。如果工具失败, 返回一个清晰的字符串, 如 错误: 未找到文件。你是指'data.csv' 吗?。这允许 Agent 自我纠正。
    
*   容忍模糊输入: 如果模型经常猜错文件路径, 更新你的工具以处理相对路径或模糊输入, 而不仅仅是出错。
    

### 2. 上下文工程

模型有有限的 "注意力预算"。管理什么信息进入上下文对性能和成本至关重要。

*   不要 "倾倒" 数据: 不要有一个返回整个 10MB 数据库表的工具。而不是 get_all_users(), 创建 search_users(query: str)。
    
*   即时加载: 而不是预加载所有数据 (传统的 RAG), 使用即时策略。Agent 应该维护轻量级标识符 (文件路径、ID), 并仅在需要时使用工具动态加载内容。
    
*   压缩: 对于运行时间很长的 Agent, 总结历史记录, 删除旧上下文或开始新会话。
    
*   Agent 记忆: 允许 Agent 维护在上下文窗口之外持久化的笔记或草稿本, 仅在相关时将它们拉回。
    

### 3. 不要过度工程化

构建复杂的多 Agent 系统很诱人。不要这样做。

*   首先最大化单个 Agent: 不要立即构建复杂的多 Agent 系统。Gemini 在单个提示中处理数十个工具的能力非常强。
    
*   逃生舱口: 确保循环可以停止, 如 max_iterations 中断 (例如, 15 轮)。
    
*   护栏和系统指令: 使用 system_instruction 用硬性规则引导模型 (例如,"你严格禁止提供超过 50 美元的退款") 或使用外部分类器。
    
*   人在回路中: 对于敏感操作 (如 send_email 或 execute_code), 在实际执行工具之前暂停循环并要求用户确认。
    
*   优先考虑透明度和调试: 记录工具调用和参数。分析模型的推理有助于识别问题, 并随着时间的推移改进 Agent。
    

## 结论

构建 Agent 不再是魔法; 这是一项实际的工程任务。正如我们所展示的, 你可以在不到 100 行代码中构建一个可工作的原型。虽然理解这些基础知识是关键, 但不要陷入一遍又一遍地重新设计相同模式的困境。AI 社区创建了出色的开源库, 可以帮助你更快地构建更复杂和健壮的 Agent。

感谢阅读! 如果你有任何问题或反馈, 请在 **Twitter[3]** 或 **LinkedIn[4]** 上告诉我。

### 参考资料

[1]

在 AI Studio 中获取: _https://aistudio.google.com/app/apikey_

[2]

思维签名: _https://ai.google.dev/gemini-api/docs/gemini-3?thinking=high[#thought](javascript:;)_signatures_

[3]

Twitter: _https://twitter.com/_philschmid_

[4]

LinkedIn: _https://www.linkedin.com/in/philipp-schmid-a6a2bb196/_

原文地址：https://www.philschmid.de/building-agents

**加入 ThinkInAI 社区**

如果你也对 AI 工具充满兴趣，欢迎加入我们的 ThinkInAI 社区，在这里，你可以：

*   获取最新 AI 工具资讯
    
*   参与实战经验分享
    
*   结识志同道合的伙伴
    
*   共同探讨 AI 应用方向
    

扫描文末二维码，加入 ThinkInAI 社区，一起拥抱 AI 新时代！

![](https://mmbiz.qpic.cn/sz_mmbiz_png/vbERicIdYZbCOFX2rx16TEbiaRZxrXkleuoZ6TLPxVKfGT699u4Vw4PBHy11BgFNFS8sSfatgzdjiaSSmaPsS69Ng/640?wx_fmt=png&from=appmsg#imgIndex=1)