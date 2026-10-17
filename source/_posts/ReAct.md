---
title: ReAct
date: 2026-09-19 16:20:17
tags:
  - AI
  - ReAct
categories:
  - AI Agent
---

# ReAct

## 0x01 ~ 什么是ReAct循环

**ReAct**是Agent执行任务的核心模式，从**ReAct**的名字上看：

**ReAct = Reasoning + Acting**

实际上，**ReAct**包括了三个环节，即**思考/推理（Thought / Reasoning）**、**行动（Acting）**、**观察（Observing）**。



## 0x02 ~ ReAct的核心流程

**ReAct**的核心流程就是**推理和行动的交织循环**，即`想→做→看→想→做→看`的循环。

**ReAct** 循环从用户提问开始。模型根据当前消息进行推理，判断信息是否足够；若不足，则决定调用哪个工具并生成调用参数，由执行器执行工具。工具返回结果后，模型观察结果，进入下一轮推理。若此时信息已足够，模型直接输出最终答案，循环结束；否则继续下一轮。

> 不过，ReAct 并不是是死循环。在实际工程实现中，通常会设置**最大迭代次数（Max Iterations）**或**超时时间（Timeout）**，以防止模型陷入死循环或无限调用工具。

![image-20260919170135644](../images/image-20260919170135644-1789808816973-1.png)

# 0x03 ~ mini_ReAct

```python
# mini_react
from openai import OpenAI
import re

client = OpenAI(
    api_key = "",
    base_url = "https://api.deepseek.com"
)

# 定义工具
def get_weather(city: str) -> str:
    '''模拟天气查询'''
    simulate_db = {"北京":"晴，25℃","上海":"小雨，18℃","东京":"多云，20℃"}
    return simulate_db.get(city.strip(), f"未找到{city}的天气数据")


def calculate(expression: str) -> str:
    '''计算数学表达式'''
    try:
        return str(eval(expression.strip()))
    except Exception as e:
        return f"计算错误: {e}"

tools = {
    "get_weather": get_weather,
    "calculate": calculate
}

# 系统提示词
SYSTEM_PROMPT = '''
你是一个强大的 AI 助手，可以使用以下工具来回答问题：

1. get_weather: 获取指定城市的天气和温度。输入参数应为城市名称，例如：北京
2. calculate: 计算数学表达式。输入参数应为合法的 Python 数学表达式，例如：(25 * 2) + 10

请严格按照以下格式进行推理和行动（每次只输出一步 Action，然后立刻停止）：

Question: 用户提出的问题
Thought: 思考我需要做什么来回答这个问题
Action: 工具名称[输入参数]

（注意：输出完 Action 后立即停止生成，等待系统返回 Observation！）
当你收到 Observation 后，继续按照 Thought -> Action -> Observation 循环，直到你拥有足够的信息。
一旦你明确了最终答案，请严格按以下格式输出：

Thought: 我现在知道最终答案了
Final Answer: 给用户的最终回答
'''

def react_agent(question: str,max_steps: int=5):
    # 维护对话历史
    messages = [
        {"role": "system","content": SYSTEM_PROMPT},
        {"role": "user","content": f"Question: {question}"}
    ]

    for step in range(1,max_steps+1):
        print(f"\n---第{step}轮推理---")

        response = client.chat.completions.create(
            model = "deepseek-v4-flash",
            messages = messages,
            temperature = 0,
            stop = ["Observation:"]
        )

        reply = response.choices[0].message.content.strip()
        print(reply)

        messages.append({"role":"assistant","content":reply})

        if "Final Answer:" in reply:
            final_answer = reply.split("Final Answer:")[-1].strip()
            print(f"\n---任务完成--\n最终答案: {final_answer}")
            return final_answer


        match = re.search(r"Action:\s*(\w+)\[(.*?)\]", reply)
        if match:
            tool_name = match.group(1)
            tool_arg = match.group(2)

            if tool_name in tools:
                observation = tools[tool_name](tool_arg)
            else:
                observation = f"错误: 找不到工具 {tool_name}"
        else:
            observation = "格式错误: 请严格输出 Action: 工具名[参数] 或 Final Answer:..."

        messages.append({"role":"user","content":f"Observation: {observation}"})

    print("\n超出最大步数限制，任务终止。")


if __name__ == "__main__":
    react_agent("北京现在的温度是多少")
```

```cmd
---第1轮推理---
Question: 北京现在的温度是多少
Thought: 我需要获取北京当前的天气和温度信息。
Action: get_weather[北京]

---第2轮推理---
Thought: 我现在知道最终答案了
Final Answer: 北京现在天气晴，温度为25℃。

---任务完成--
最终答案: 北京现在天气晴，温度为25℃。
```



## 0x04 ~ ReAct 的结构化实现：Function Calling

这里使用LangChain结构化实现，使用的是硅基流动中转站的模型

```python
# weather.py
# agent调用的工具定义

import requests
import json

weather_URL = "https://uapis.cn/api/v1/misc/weather"

def get_weather(city):
    data = requests.get(weather_URL,params={"city": city})
    if data.status_code == 200:
        return data.json()
    else:
        print("天气API请求失败，状态码：", data.status_code)
        return None
```

```python
import os
from dotenv import load_dotenv

load_dotenv(override=True)

from langchain.chat_models import init_chat_model
from weather import get_weather

BASE_URL = os.environ.get("BASE_URL")
SILICONFLOW_API_KEY = os.environ.get("SILICONFLOW_API_KEY")

SYSTEM_PROMPT = "你是一名天气查询助手，你可以帮助用户查询城市天气。"
TOOLS = [get_weather]
MAX_ITERATIONS = 10


def init_model():
    model = init_chat_model(
        model="Qwen/Qwen3.6-27B",
        model_provider="openai",
        base_url=BASE_URL,
        api_key=SILICONFLOW_API_KEY,
    )
    return model


def init_messages():
    return [{"role": "system", "content": SYSTEM_PROMPT}]


def execute_tool(tool_call):
    tool_name = tool_call["name"]
    tool_args = tool_call.get("args", {})
    tool_call_id = tool_call["id"]

    if tool_name == "get_weather":
        city = tool_args.get("city")
        weather_info = get_weather(city)
        if weather_info:
            return {
                "role": "tool",
                "content": f"天气信息: {weather_info}",
                "tool_call_id": tool_call_id,
            }
        return {
            "role": "tool",
            "content": "获取天气信息失败。",
            "tool_call_id": tool_call_id,
        }
    else:
        return {
            "role": "tool",
            "content": f"未知工具: {tool_name}",
            "tool_call_id": tool_call_id,
        }


def agent_loop(model, messages):
    model_with_tools = model.bind_tools(tools=TOOLS)

    while True:
        user_input = input("You: ")
        if user_input.lower() == "exit":
            print("退出对话。")
            break

        messages.append({"role": "user", "content": user_input})

        for i in range(MAX_ITERATIONS):
            response = model_with_tools.invoke(messages)
            messages.append(response)

            if not response.tool_calls:
                print("AI:", response.content)
                break

            for tool_call in response.tool_calls:
                tool_msg = execute_tool(tool_call)
                messages.append(tool_msg)
        else:
            print("AI: 达到最大迭代次数，停止。")


def main():
    model = init_model()
    messages = init_messages()
    agent_loop(model, messages)


if __name__ == "__main__":
    main()
```



## 0x05 ~ 代码执行流程

以 **0x04** 的代码为例，其执行流程如下：

> main()
>  ├─ init_model()      创建模型
>  ├─ init_messages()   初始化上下文（system prompt）
>  └─ agent_loop()      ReAct 循环入口
>       └─ while True            外层：多轮对话
>            ├─ 读用户输入
>            ├─ 追加 user 消息
>            └─ for i in range(MAX_ITERATIONS)   内层：ReAct 循环
>                 ├─ model_with_tools.invoke(messages)   ← Thought + Action
>                 ├─ 追加 assistant 消息
>                 ├─ 判断 response.tool_calls
>                 │    ├─ 为空 → 输出最终答案，break
>                 │    └─ 不为空 → 逐个执行工具
>                 │         ├─ execute_tool(tool_call)   ← Action 执行
>                 │         └─ 追加 tool 消息            ← Observation
>                 └─ 回到 for，进入下一轮推理



### **一次完整的执行追踪**：

#### 第一轮：

`messages` 状态：

```python
messages = [
    {"role": "system", "content": "你是一名天气查询助手，你可以帮助用户查询城市天气。"},
    {"role": "user",   "content": "北京"},
]
```

`Model` 进行推理思考，决策是否调用工具：

```python
response = model_with_tools.invoke(messages)
```

模型返回的是 `AIMessage` ：

```python
AIMessage(
    content='\n\n',
    tool_calls=[{
        "name": "get_weather",
        "args": {"city": "北京"},
        "id": "01a0c341cd4b02c3288793ff7ecf7c03",
        "type": "tool_call"
    }],
    response_metadata={
        "finish_reason": "tool_calls",
        "model_name": "Qwen/Qwen3.6-27B",
        ...
    },
    usage_metadata={
        "input_tokens": 275,
        "output_tokens": 465,
        "output_token_details": {"reasoning": 439},
        ...
    }
)
```

追加 `assistant` 消息：

```python
messages.append(response)          # 追加 assistant 消息（含 tool_calls）
```

此时 `messages` 为：

```python
[
    {"role": "system", "content": "你是一名天气查询助手..."},
    {"role": "user",   "content": "北京"},
    AIMessage(
        content='\n\n',
        tool_calls=[{
            "name": "get_weather",
            "args": {"city": "北京"},
            "id": "01a0c341cd4b02c3288793ff7ecf7c03",
            "type": "tool_call"
        }],
        ...
    ),
]
```

执行工具，追加 `tool` 后

```python
for tool_call in response.tool_calls:
    tool_msg = execute_tool(tool_call)
    messages.append(tool_msg)
```

此时 `messages` 是：

```python
[
    {"role": "system", "content": "你是一名天气查询助手..."},
    {"role": "user",   "content": "北京"},
    AIMessage(
        content='\n\n',
        tool_calls=[{
            "name": "get_weather",
            "args": {"city": "北京"},
            "id": "01a0c341cd4b02c3288793ff7ecf7c03",
            "type": "tool_call"
        }],
        ...
    ),
    {
        "role": "tool",
        "content": "天气信息: {'province': '北京市', 'city': '北京', 'weather': '晴', 'temperature': 28, ...}",
        "tool_call_id": "01a0c341cd4b02c3288793ff7ecf7c03"
    }
]
```

#### 第二轮：

```python
response = model_with_tools.invoke(messages)
```

模型看到 `tool` 消息中有天气数据，返回：

```python
AIMessage(
    content="北京今天晴，气温 28℃，西南风 3 级，湿度 39%……",
    tool_calls=[],
    response_metadata={"finish_reason": "stop", ...}
)
```

判断并退出

```python
if not response.tool_calls:
    print("AI:", response.content)
    break
```

当前这一次用户请求的 `ReAct` 循环结束。



## 0x05 ~ 消息角色对照表

`0x04` 中的消息与 `ReAct` 环节的对应

| 消息                                      | 角色/role   | ReAct 环节                 |
| :---------------------------------------- | :---------- | :------------------------- |
| `system`                                  | `system`    | 设定身份                   |
| `user`                                    | `user`      | Question                   |
| `AIMessage(tool_calls=[...])`             | `assistant` | Thought(Thinking) + Action |
| `{"role": "tool", ...}`                   | `tool`      | Observation                |
| `AIMessage(content="...", tool_calls=[])` | `assistant` | Final Answer               |

```markdown
- system 不在ReAct循环中，循环起点是user
- system 为背景约束
```

