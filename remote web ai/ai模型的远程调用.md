
## 关于completion的构成和chunk

|     | completion                                            | chunk                                                                                                                                                                                                                                                                                  |
| --- | ----------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 非流式 | `completion` 是一个普通的 Python 对象（通常是一个字典），包含所有生成的内容和元数据。 | \                                                                                                                                                                                                                                                                                      |
| 流式  | `completion` 是一个**生成器对象**， `chunk` 是返回一个数据片段          | {<br>  "id": "cmpl-123",<br>  "object": "chat.completion.chunk",<br>  "created": 1698999575,<br>  "model": "kimi-k2.5",<br>  "choices": [<br>    {<br>      "index": 0,<br>      "delta": {<br>        "content": "你好"<br>      },<br>      "finish_reason": null<br>    }<br>  ]<br>} |
``` json
 ChatCompletionChunk(
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
 )
```
``` json
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

## 关于agent和tools的使用

#### 大语言模型+代码工具的交互流程

提问
→模型第一次返回，生成一段生来解决问题的代码，比如调用函数
→用户调度执行返回的代码
→返回结果给大模型，执行结果被包装成role=tool
→模型利用结果返回输出答案

#### exec()沙箱、安全运行环境

**沙箱 = 给代码建一个 “监狱”**

- 代码只能在里面运行
- 不能访问你的文件
- 不能访问网络
- 不能破坏系统
- 不能读取密钥

## 关于history的运用
其实最简单的就是创造一个列表，里面装着各种messages，作为对话内容直接提供给ai，现在还没有添加长对话的功能，只是简单粗暴的告诉他们

## 关于图片的上传
两种格式，一个是url，一个是base64
``` python
with open("您的图片地址", 'rb') as f: img_base = base64.b64encode(f.read()).decode('utf-8')
```

```json
"url":f"[data](data:image/jpeg;base64,{img_base})"
```
## 关于api的兼容性，能否直接使用openai的框架但是使用kimi的api一些参数的调整

## 屏幕共享是怎么做到的？