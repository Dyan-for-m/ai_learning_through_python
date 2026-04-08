## 大纲&快速跳转
- [[#参数选择]]
- [[#关于completion的构成和chunk]]
- [[#关于chunk 流式输出]]
- [[#关于agent和tools的使用]]
	- [[#大语言模型+代码工具的交互流程]]
	- [[#exec()沙箱、安全运行环境]]
	- [[#tools 管理工具]]
	- [[#工具市场]]
- [[#关于history的运用]]
	- [[#多用户隔离]]/ [[#并发安全 Lock]]/[[#持久化（关闭程序再打开，聊天还在）]]/ [[#精确控制历史（按 token 数量，不按条数）]]/ [[#历史消息总结]]
- [[#关于图片的上传]]
- [[#关于Thinking能力]]
- [[#关于api的兼容性，能否直接使用openai的框架但是使用kimi的api一些参数的调整]]
- [[#屏幕共享是怎么做到的？]]

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
生成器：一个可以暂停，恢复执行的函数
遇到yield就暂停，下次从暂停处继续
`next（生成器对象名称）`
ai的打字效果=生成器+ **flush = True** + 循环输出
生成器的特点是**只有在被迭代时才会生成数据**
- 服务器不会发送任何数据。
- `completion` 只是一个生成器对象，数据流没有被触发
当你调用 `completion=client.chat.completions.create(..., stream=True)` 时，客户端会向服务器发送一个 HTTP 请求，要求开启数据流。客户端的生成器会按需从网络缓冲区中读取数据。每次迭代时，生成器会触发一次网络读取，接收一个数据片段。

## 关于tools的使用

### 大语言模型+代码工具的交互流程

1. 提问以及工具的说明
2. → completion1 模型第一次返回，`stop，"finish_reason": "tool_calls"` ，生成一段生来解决问题的代码，总结出你在工具说明里面调用的工具以及对应需要的参数等等
3. →用户调度执行返回的代码
4. →completion2，添加到messages里面，返回结果给大模型，执行结果被包装成role=tool+结果+id识别符号
	- 返回的messages包括
		1. 追加assistant消息：原样带回模型的tool_calls
		2. 追加tool消息：传递工具执行结果，与tool_call_id一一对应
5. →模型利用结果返回输出答案completion2
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

### tools 管理工具
### 工具市场
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
#### fiber
fiber = Kimi 官方对 “一次工具执行任务” 的叫法，相当于一次工具调用的结果，包含了工具执行的状态、输出结果或者错误信息等内容。
需要得到fiber的结果包
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

## 关于api的兼容性，能否直接使用openai的框架但是使用kimi的api一些参数的调整

## 屏幕共享是怎么做到的？
