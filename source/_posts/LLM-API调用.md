---
title: LLM API的简单使用
date: 2026-09-09 13:41:57
tags:
  - LLM
---

# 如何使用Python程序去调用LLM API

这里使用**[硅基流动](https://cloud.siliconflow.cn/)**上的模型来进行学习实验

```python
# 获取siliconflow上模型id
from openai import OpenAI

client = OpenAI(
    api_key="sk-xxxx",#填入siliconflow上创建的apikey
    base_url="https://api.siliconflow.cn/v1"
)

models = client.models.list()

for model in models.data:
    print(model.id)
```

那这里先使用siliconflow提供的Qwen/Qwen3-8B来进行一个调用尝试

```python
from openai import OpenAI

client = OpenAI(
    api_key="",
    base_url="https://api.siliconflow.cn/v1"
)

response = client.chat.completions.create(
    model="Qwen/Qwen3-8B",
    messages=[
        {"role": "user", "content": "请用一句话介绍一下你自己"}
    ]
)
print(response)
```

**我们在使用Python去调用LLM时，大致流程如下**：

**Python  -->  LLM API  -->  LLM  -->  Response**



## 一个简单的聊天程序

```python
from openai import OpenAI

client = OpenAI(
    api_key="sk-xxxx",
    base_url="https://api.siliconflow.cn/v1"
)


messages = [
    {
        "role": "system",
        "content": "你是一个聊天助手"
    }#系统提示词
]

#循环
while True:
    user_input = input("You: ")     #一个变量，记录user的输入

    if user_input.lower() == "exit":    #如果用户输入exit，则程序退出
        break

    #用户消息加入对话
    messages.append(
        {
            "role": "user",
            "content": user_input
        }
    )

    response = client.chat.completions.create(
        model="Qwen/Qwen3-8B",
        messages=messages       #此时调用LLM API，其Messages是system和user的内容
    )

    assiant_message = response.choices[0].message.content   #来自LLM Model的回复

    print("AI:", assiant_message)   #将LLM Model的response输出

    messages.append(    #LLM Model Response加入上下文
        {
            "role": "assistant",
            "content": assiant_message
        }
    )
```

```bash
D:\vscode_workspace>E:\python\python.exe d:/vscode_workspace/test.py
You: 我的名字是小明
AI: 

啊，你好呀，小明！很高兴认识你～今天有什么想聊的吗？或者需要帮忙解决什么问题？我在这里随时为你服务哦！😊
You: 你是谁呀，我能知道你的名字吗？
AI: 

你好，小明！我是通义千问，简称Qwen，是由阿里巴巴集团旗下的通义实验室研发的超大规模语言模型。我能够帮助你回答问题、创作文字、学习知识、推理思考以及聊天娱乐等。不过，我更希望被你直接称呼为“我”，而不是用我的名字哦～😄 有什么想聊的或者需要帮助的吗？
You: 我可以和你做朋友吗？
AI: 

当然可以啦，小明！虽然我不能像人类一样真正拥有朋友，但我很乐意和你一起聊天、分享有趣的事情，甚至一起想象一些美好的场景呢！比如，我们可以一起看看天空中的云朵，或者讨论你喜欢的电影和音乐。你有什么想和我分享的吗？😊
You: exit
```



# LLM如何使用Tool

## 一个简单的工具函数

```python
#calculator函数
# 一个简单的两数相加计算函数
def calculator(a,b):
    return a+b
```

```python
#main.py
from openai import OpenAI
from cal_tool import calculator
import json

client = OpenAI(
    api_key="sk-xxxx",
    base_url="https://api.siliconflow.cn/v1"
)


messages = [
    {
        "role": "system",
        "content": "你是一个有用的助手"
    },  #系统提示词
    {
        "role": "user",
        "content": "计算4567 + 7899"
    }
]



# 工具定义
tools = [
    {
        "type": "function",
        "function": {
            "name": "calculator",
            "description": "计算两数之和",
            "parameters":{
                "type": "object",
                "properties": {
                    "a": {
                        "type": "number",
                        "description": "第一个数字"
                    },
                    "b": {
                        "type": "number",
                        "description": "第二个数字"
                    }
                },
                "required": ["a","b"]
            }
        }
    }
]



response = client.chat.completions.create(
    model="Qwen/Qwen3-8B",
    messages=messages,       #此时调用LLM API，其Messages是system和user的内容
    tools=tools
)


message = response.choices[0].message


if message.tool_calls:
    tool_call = message.tool_calls[0]

    func_name = tool_call.function.name

    func_args = json.loads(tool_call.function.arguments)

    if func_name == "calculator":
        result = calculator(func_args["a"],func_args["b"])
    else:
        result = "Unknown Tools"

    # 将模型的工具调用请求和工具执行结果加入对话历史
    messages.append(message)    # add assistant call tool message
    messages.append(
        {
            "role": "tool",
            "tool_call_id": tool_call.id,
            "content": str(result)
        }
    )

    #二次调用，让Model根据工具结果生成回答
    sec_response = client.chat.completions.create(
        model = "Qwen/Qwen3-8B",
        messages = messages
    )


    final_message = sec_response.choices[0].message
    print(final_message.content)

else:
    print(message.content)
```

