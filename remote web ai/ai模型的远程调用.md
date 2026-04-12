## 大纲&快速跳转
- [大纲\&快速跳转](#大纲快速跳转)
- [Token](#token)
	- [自回归生成（Autoregressive Generation）](#自回归生成autoregressive-generation)
	- [编号](#编号)
- [参数选择](#参数选择)
- [关于completion的构成和chunk](#关于completion的构成和chunk)
- [关于chunk 流式输出](#关于chunk-流式输出)
	- [生成器+sse](#生成器sse)
	- [多个回答的流式输出](#多个回答的流式输出)
- [关于tools的使用](#关于tools的使用)
	- [大语言模型+代码工具的交互流程](#大语言模型代码工具的交互流程)
	- [exec()沙箱、安全运行环境](#exec沙箱安全运行环境)
	- [流式输出里面的工具调用](#流式输出里面的工具调用)
	- [tools 管理工具](#tools-管理工具)
	- [工具市场](#工具市场)
		- [请求头 \& 鉴权密钥（API Key）](#请求头--鉴权密钥api-key)
		- [**URI** 统一资源标识符 资源的唯一名字 / 地址](#uri-统一资源标识符-资源的唯一名字--地址)
		- [fiber](#fiber)
		- [curl 调用](#curl-调用)
- [关于history的运用](#关于history的运用)
	- [多用户隔离](#多用户隔离)
	- [并发安全 Lock](#并发安全-lock)
	- [持久化（关闭程序再打开，聊天还在）](#持久化关闭程序再打开聊天还在)
	- [精确控制历史（按 token 数量，不按条数）](#精确控制历史按-token-数量不按条数)
	- [历史消息总结](#历史消息总结)
- [关于图片的上传](#关于图片的上传)
- [关于Thinking能力](#关于thinking能力)
- [json mode](#json-mode)
- [partial Mode](#partial-mode)
	- [背后逻辑](#背后逻辑)
	- [角色设定](#角色设定)
		- [参数name](#参数name)
		- [保持角色一致性技巧](#保持角色一致性技巧)
- [文件上传逻辑](#文件上传逻辑)
	- [正确使用姿势（官方推荐）](#正确使用姿势官方推荐)
- [联网搜索功能](#联网搜索功能)
- [Agent的建设](#agent的建设)
	- [agent构建工具](#agent构建工具)
		- [LangChain](#langchain)
		- [LangGraph（下一代](#langgraph下一代)

> 大模型会说 “我很难过”，但它完全不难过
> 它只是根据概率，输出了最合理的情绪文字
> 因为它模仿人类语言的结构太完美了。
> 就像：
> 一个超级会模仿的鹦鹉
> 一个能完美演戏的演员
> 一个能写漂亮文章的机器
> 它看起来像有思想，
> 但内部一片黑暗，没有任何体验。

## Token
Token(数字编号) = 大模型读写语言的最小单位
模型只认识 **数字（Token ID）**
### 自回归生成（Autoregressive Generation）
**模型一次只能生成 1 个 Token**， 然后根据这个 Token，预测下一个
这叫 自回归生成
每次只看下前面所有文字 → 预测下一个最合理的字
模仿人类文本的统计规律
### 编号
每一个 “单元” 都有一个唯一编号
按「子词」subword决定，
编号由模型厂商在训练模型时提前定好，永远不变
Tokenizer
- 把人类文字 → 切成一串 token 单元
- 给每个单元一个固定数字编号（ID）
## 参数选择 

- 采样温度 temperature
	- 简单说：**控制 AI 说话的 “随机性 / 创造力”**
	- 越高越容易“胡说”
- 频率惩罚 frequency_penalty
	- 控制 “重复说话” 的程度
	- 越低越重复
- 存在惩罚 presence_penalty
	- 控制 “是否愿意聊新话题 / 引入新内容”
	- 越高越聊新的话题
- 多步工具调用必须保留reasoning_content
## 关于completion的构成和chunk

|     | completion                                            | chunk                                                                                                                                                                                                                                                                                  |
| --- | ----------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 非流式 | `completion` 是一个普通的 Python 对象（通常是一个字典），包含所有生成的内容和元数据。 | \                                                                                                                                                                                                                                                                                      |
| 流式  | `completion` 是一个**生成器对象**， `chunk` 是返回一个数据片段          | {<br>  "id": "cmpl-123",<br>  "object": "chat.completion.chunk",<br>  "created": 1698999575,<br>  "model": "kimi-k2.5",<br>  "choices": [<br>    {<br>      "index": 0,<br>      "delta": {<br>        "content": "你好"<br>      },<br>      "finish_reason": null<br>    }<br>  ]<br>} |
|     |                                                       |                                                                                                                                                                                                                                                                                        |
``` python
#ChatCompletionChunk
 {
	 id='chatcmpl-69cf964e74b466368a035a61', 
	 choices=[
		 Choice(
			 delta=ChoiceDelta(
				 content='时分', 
				 function_call=None, 
				 refusal=None, role=None, 
				 tool_calls=None
				), 
		 finish_reason=None, 
		 index=0, 
		 logprobs=None
		 )
	 ], 
	 created=1775212110, 
	 model='moonshot-v1-8k', 
	 object='chat.completion.chunk', 
	 service_tier=None, 
	 system_fingerprint='fpv0_3f4f71b6', 
	 usage=None
 }
```
``` python
#Completion
{
  "id": "chatcmpl-69cf809374b466368a027307",
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "logprobs": null,
      "message": {
        "content": "...",
        "refusal": null,
        "role": "assistant",
        "annotations": null,
        "audio": null,
        "function_call": null,
        "tool_calls": null
      }
    }
  ],
  "created": 1775206547,
  "model": "moonshot-v1-8k",
  "object": "chat.completion",
  "service_tier": null,
  "system_fingerprint": null,
  "usage": {
    "completion_tokens": 2,
    "prompt_tokens": 47,
    "total_tokens": 49,
    "completion_tokens_details": null,
    "prompt_tokens_details": null
  }
}
```


## 关于chunk 流式输出
### 生成器+sse
遇到yield就暂停，下次从暂停处继续
`next（生成器对象名称）`
ai的打字效果=生成器+ **flush = True** + 循环输出
生成器的特点是**只有在被迭代时才会生成数据**
- 服务器不会发送任何数据。
- `completion` 只是一个生成器对象，数据流没有被触发
当你调用 `completion=client.chat.completions.create(..., stream=True)` 时，客户端会向服务器发送一个 HTTP 请求，要求开启数据流。客户端的生成器会按需从网络缓冲区中读取数据。每次迭代时，生成器会触发一次网络读取，接收一个数据片段。
当启用流式输出模式（stream=True）时，Kimi 大模型不再返回一个 JSON 格式（`Content-Type: application/json`）的响应，而是使用 `Content-Type: text/event-stream`（简称 SSE）
在 [SSE](https://kimi.moonshot.cn/share/cr7boh3dqn37a5q9tds0) 的响应体中，我们约定数据块均以 `data:` 为前缀，紧跟一个合法的 JSON 对象，随后以两个换行符 `\n\n` 结束当前传输的数据块。

### 多个回答的流式输出
使用index来区分属于第几个回答
```python
for choice in chunk["choices"]: 
index = choice["index"] # 拿到编号：0 或 1 
message = messages[index] # 找到对应的盒子 
content = delta.get("content") # 拿到文字 
if content: message["content"] += content # 把文字追加到对应盒子里
```

## 关于tools的使用

### 大语言模型+代码工具的交互流程

1. 提问以及工具的说明
	- JSON Schema
2. → completion1 模型第一次返回，`stop，"finish_reason": "tool_calls"` ，生成一段生来解决问题的代码，总结出你在工具说明里面调用的工具以及对应需要的参数等等
3. →用户调度执行返回的代码
4. →completion2，添加到messages里面，返回结果给大模型，执行结果被包装成role=tool+结果+id识别符号
	- 返回的messages包括
		1. 追加assistant消息：原样带回模型的tool_calls
		2. 追加tool消息：传递工具执行结果，与tool_call_id一一对应
5. →模型利用结果返回输出答案completion2

- JSON Schema
```json
{
	"type": "function",
	"function": {
		"name": "NAME",
		"description": "DESCRIPTION",
		"parameters": {
			"type": "object",
			"properties": {
				
			}
		}
	}
}
```
### exec()沙箱、安全运行环境

**沙箱 = 给代码建一个 “监狱”**

- 代码只能在里面运行
- 不能访问你的文件
- 不能访问网络
- 不能破坏系统
- 不能读取密钥

```python
local_vars = {} 
exec(code, {"__builtins__":{}}, local_vars)
```

`exec(代码, 全局命名空间, 局部命名空间)`
`{"__builtins__":{}}` 清空所有的工具，不能调用任何危险功能
`local_vars` 结果


**高级沙盒：**
- Docker 容器
- PyPy Sandbox / RestrictedPython
- 云函数 / 在线代码沙箱服务

### 流式输出里面的工具调用
难点：
- 并非依靠finish_reason判断是否调用工具，而是delta.tool_calls是否存在
- 先输出content后输出tool_calls
- tool_call结构是分片的
- 多个tool_calls使用index区分
拼接tool_calls
注意arguments分多片
```python
if tool_call_function_arguments: 
	if "arguments" not in tool_call_object["function"]: # 第一次来，直接赋值 
	tool_call_object["function"]["arguments"] = tool_call_function_arguments 
	else: # 不是第一次，追加拼接！ 
	tool_call_object["function"]["arguments"] += tool_call_function_arguments
```
### tools 管理工具
请看[agent构建工具](#agent构建工具)
### 工具市场
Formula
#### 请求头 & 鉴权密钥（API Key）
**HTTP 请求头**：
- 你向服务器发请求时，除了请求的核心数据（如 JSON、表单），还会携带一组**键值对元信息**，用于告诉服务器「请求的类型、身份、格式」等
**鉴权密钥**：
- Bearer Tocken 鉴权`Bearer {api_key}`，无加密的简单令牌，简洁高效，适合开放平台 API
- 其他鉴权
	- **Basic Auth** ：将「用户名：密码」做 Base64 编码后放在请求头，安全性较低，适合内部接口；
	- **HMAC 签名**：将请求参数 + 时间戳做加密签名，放在请求头，安全性极高，适合金融、支付等敏感接口
#### **URI** 统一资源标识符 资源的唯一名字 / 地址
URL 是 URI 的子集
- **URL**：不仅标识资源，还**给出了资源的具体访问地址**
- **URI**：只做「唯一标识」，**不一定包含访问地址**，eg `厂商/工具名:版本` `moonshot/date:latest`
	- `namespace/name:tag`
#### fiber
fiber = Kimi 官方对 “一次工具执行任务” 的叫法，相当于一次工具调用的结果，包含了工具执行的状态、输出结果或者错误信息等内容。
需要得到fiber的结果包
#### curl 调用
```bash
curl -X POST https://api.moonshot.cn/v1/formulas/moonshot/web-search:latest/fibers 
-H "Authorization: Bearer $API_KEY" 
-d '{ 
	"name": "web_search", 
	"arguments": "{\"query\":\"月之暗面最近有什么消息\"}" }'

```
内容被加密直接返回加密内容给ai里面的content，并且保持id的一致
## 关于history的运用
其实最简单的就是创造一个列表，里面装着各种messages，作为对话内容直接提供给ai，现在还没有添加长对话的功能，只是简单粗暴的告诉他们

***一些问题*** 以及简单的解决方式
### 多用户隔离
用「用户 ID」做 key，字典存每个用户的历史
### 并发安全 Lock
如果同时来 2 个请求，同时修改 `history`，会导致：
- 消息错乱
- 列表越界
- 程序崩溃
解决方案：**加锁（Lock）**
- 锁 = 一个 “只能一个人用” 的通行证
- 同一时间，只有一个人能拿到锁，其他人必须排队等
```python
import threading
lock = threading.Lock()
with lock:
	...
```
with lock = **自动上锁 + 自动解锁**
- 进去时：`lock.acquire()`
- 出来时：`lock.release()`
### 持久化（关闭程序再打开，聊天还在）
把每个用户的 history 保存到 JSON 文件，包含用户id，使用时加载，结束后保存
### 精确控制历史（按 token 数量，不按条数）
使用官方的token计算器，而非简单的messages条数
eg.
```python
def count_tokens(messages): 
	try: 
		response = client.chat.completions.create( 
			model=model, 
			messages=messages, 
			max_tokens=1 
		) 
		return response.usage.prompt_tokens 
	except: 
		return 0
```
或者直接简单的TOKEN计算
```python
def count_tokens(messages):
    approx = 0
    for msg in messages:
        approx += len(str(msg.get("content", ""))) // 2 + 5
    return approx
```
### 历史消息总结
把旧消息送给 AI 总结 → 用一段总结代替旧对话   

## 关于图片的上传
两种格式，一个是url，一个是base64
``` python
with open("您的图片地址", 'rb') as f: img_base = base64.b64encode(f.read()).decode('utf-8')
```

```json
"url":f"[data](data:image/jpeg;base64,{img_base})"
```
## 关于Thinking能力
拓展openai的输出，添加了resoning_content部分，可以打印思考过程
- `hasattr(choice.delta, "reasoning_content")`
- `getattr(choice.delta, "reasoning_content")`

## json mode
强制返回一段标准、干净、可直接解析的 JSON 格式字符串

```python
response_format={"type": "json_object"}
```
- 必须在提示词里告诉 AI JSON 结构
- 只能返回 JSON 对象{}，不能返回数组[]
## partial Mode
**Partial Mode**让 AI **接着你给的句子继续往下说**
同时适用于内容被截断后的继续生成
### 背后逻辑
```plaintext
[对话历史tokens] + [partial_tokens] 
						↓ 
					模型从这里开始生成下一个token
```
Partial 必须是 `role=assistant`, 因为**只有模型自己的输出可以被续写**

### 角色设定
#### 参数name
强化模型对角色的认知，强制模型以 `name` 指定的角色的口吻输出内容
- 在 token 序列前插入角色标识
- 让模型的**注意力机制（attention）绑定角色**
- 强制模型生成时**保持该角色的语言模型分布**
#### 保持角色一致性技巧
- 在设置角色时，详细介绍他们的个性、背景以及可能具有的任何具体特征或怪癖
- 扮演的角色的细节，例如说话的语气、风格、个性，甚至背景，如背景故事和动机
- 指导在各种情况下如何行动：如果预计角色会遇到某些特定类型的用户输入，或者希望控制模型在角色扮演互动中的某些情况下的输出
- 定期使用系统提示词 system prompt 强化角色的设定

## 文件上传逻辑
1. **上传文件** → 服务器帮你解析
2. **提取文件文本内容**
3. **把内容塞进 messages “system”里** 再提问

```python
file_object = client.files.create(file=Path("a.pdf"), purpose="file-extract") 
file_content = client.files.content(file_id=file_object.id).text
```
### 正确使用姿势（官方推荐）

1. 上传文件
2. 提取内容 `file_content`
3. **把内容保存到本地**（txt / 数据库都行）
4. **删除服务器上的文件**
5. 下次直接用本地保存的内容，不用再上传


## 联网搜索功能
在实现联网搜索的过程中，我们需要自己实现 `search` 和 `crawl` 函数，这其中可能包括：

1. 调用搜索引擎接口，或自己实现内容搜索；
2. 获取搜索结果，包括 URL 和摘要等信息；
3. 根据 URL 获取网页内容，可能需要针对不同的网站应用不同的读取规则；
4. 将获取的网页内容清洗并整理成模型便于识别的格式，例如 Markdown；
5. 处理各种错误和异常情况，例如无搜索结果、网页内容获取失败等；
提供的内置工具函数`builtin_function.$web_search`
```python
tools = [
	{
		"type": "builtin_function",  # <-- 我们使用 builtin_function 来表示 Kimi 内置工具，也用于区分普通 function
		"function": {
			"name": "$web_search",
		},
	},
]
```

```python
def search_impl(arguments: Dict[str, Any]) -> Any: 
	return arguments # 原封不动返回！
```
但是会存在大量的token使用
以下方式减少token的使用
1. 增加提示词，如：要求： 1. 只搜索最相关的 1–2 篇来源 2. 摘要式返回，不要全文 3. 控制总长度，精简回答
2. 官方 Context Caching 适合短时间多次同类搜索，结果缓存，在extra_body里面开启缓存`extra_body={ "cache": { "enabled": True, "ttl": 3600 # 缓存 1 小时 }`
## Agent的建设
- system prompt
- tools
- call tool，行动能力，如何handle response
- 递归和循环
### agent构建工具
#### LangChain
- 提示词模板
- 记忆管理
- 工具调用
- 结构化输出
- 检索 RAG
- 智能体流程
- 多模型适配
LangChain Agent 工作流(基础版)
1. 用户输入
2. 进入 Prompt 模板（含 scratchpad)
3. LLM 思考宀输出 tool_call
4. Agent 解析工具
5. Executor 执行工具
6. 结果放回 scratchpad
7. 再送给 LLM 继续思考
8. 直到 stop
#### LangGraph（下一代
解决 LangChain Agent 的缺点：
- 不可控流程
- 无法断点
- 无法多智能体
- 无法循环判断
- 无法可视化
 LangGraph 核心概念：

1. State（状态）
	所有数据存在一个状态字典里。
 2. Node（节点）
	每个节点是一个函数：LLM、工具、代码、逻辑判断。
3. Edge（边）
	控制流程走向：
	- 顺序执行
	- 条件分支
	- 循环
	- 多智能体通信
4. 循环支持
	可以无限循环直到完成任务。



