## 快速索引
- [pathlib 路径的优雅处理](#pathlib-路径的优雅处理)
- [float | None = None的新理解](#float--none--none的新理解)
- [raise](#raise)
- [调用系统命令subprocess.run()](#调用系统命令subprocessrun)
- [解析json json.loads() VS eval()](#解析json-jsonloads-vs-eval)
- [临时文件：文件的句柄+ delete = False](#临时文件文件的句柄-delete--false)
- [文档字符串 Docstring](#文档字符串-docstring)
- [pydantic](#pydantic)
	- [pydantic基础教程](#pydantic基础教程)
- [dic.get(key): 字典的安全取值方式](#dicgetkey-字典的安全取值方式)
- [语法糖](#语法糖)
- [my_dict.itmes()](#my_dictitmes)
- [self 实例自己](#self-实例自己)
- [__双下划线](#__双下划线)
- [_ 单下划线](#_-单下划线)
- [hasattr & getattr的深度解析](#hasattr--getattr的深度解析)
- ["""+.strip()](#strip)
- [global关键词](#global关键词)
- [main](#main)
- [append VS extend](#append-vs-extend)
- [async & await](#async--await)
- [argparse](#argparse)
- [assert](#assert)
- [try  & except & finally](#try--except--finally)
- [去重并且保留顺序](#去重并且保留顺序)
- [dict(zip())](#dictzip)
- [生成器（Generator）](#生成器generator)
- [any()](#any)
- [set()集合](#set集合)
- [DataFrame行迭代器](#dataframe行迭代器)

## pathlib 路径的优雅处理

- **将路径作为「对象」处理**，支持**链式调用**
	- `Path(path)` 接受一个路径字符串（./test.mp4）返回一个Path实例对象，自带大量的路径处理
```python
from pathlib import Path
p = Path("./test.mp4") print(p.exists()) # 判断路径是否存在（文件/文件夹都可）→ True/False 
print(p.is_file()) # 判断是否是文件
print(p.is_dir()) # 判断是否是文件夹
print(p.name) # 获取完整文件名
print(p.stem)# 获取文件名（不含后缀）
print(p.suffix) # 获取文件后缀 
print(p.parent) # 获取父目录路径对象
print(p.absolute())# 获取绝对路径对象
```
- **路径拼接：替代os.path.join 更加直观**

```python
base = Path("/root/myaipython") 
new_path = base / "project1" / "test.py" # 路径拼接，自动适配系统分隔符 
print(new_path) # → /root/myaipython/project1/test.py
```
- 创建文件夹，判断空文件夹
```python
# 创建文件夹（parents=True：父目录不存在则自动创建；exist_ok=True：已存在不报错）
Path("./new_dir/sub_dir").mkdir(parents=True, exist_ok=True) # 判断文件夹是否为空 
print(Path("./new_dir").is_empty()) # → True/False
```

## float | None = None的新理解
分为两个部分 
- `float | None` 输入值的或选择
- 设置 **= None** 参数默认值 
- 语法糖
`Union[float, None] = None`类型或
## raise
**主动触发一个指定类型的异常**，让程序立即终止执行
被`try/except` 包裹捕获可以继续执行，并且有一个兜底结果，让调用方继续执行
## 调用系统命令subprocess.run()
列表字符串输入
返回值是subprocess.CompletedProcess类的实例
```python
probe.stdout #标准输出
probe.stderr #错误信息
probe.returncode #命令执行状态码 0成功 非零失败
```

## 解析json json.loads() VS eval()
```python
json.loads() #将JSON 格式的字符串转成 Python 的字典/列表
json.dumps() #反向操作
eval(string) #将传入的字符串「当作 Python 代码执行」，并返回执行结果
```
`eval()`会执行代码，会有安全问题但是`json.loads` 不会有执行的操作
## 临时文件：文件的句柄+ delete = False

- 文件句柄（File Handle）是**操作系统给打开的文件分配的一个「唯一标识 / 索引」**，可以理解为**文件的「身份证」+「操作通道**
- 当执行`f.close()`或程序结束时，操作系统会**关闭句柄**，释放这个标识
- 句柄是 Python 和操作系统之间「沟通文件」的桥梁
- `tempfile.NamedTemporaryFile`用于创建**临时文件**，默认文件句柄被关闭的时候自动删除
## 文档字符串 Docstring
``` python
""" """
```
可通过`help(函数名)`或`函数名.__doc__`直接在 Python 终端查看文档

## pydantic

API 返回的数据**需要严格的规则**
**pydantic**是一个「**严格的键值对容器**」，专门用来**校验数据的格式和类型**，保证数据「符合规则」
- 严格的规则约束
- 自带的实用功能
	- 编辑器中提示所有字段
- 无法直接json序列化，需要`model_dump()`,把 Pydantic 对象「转成普通的 Python 字典」
	- 内容一致格式变化
	- pydantic对象如果有可选字段，字段值是 None， model_dump()会自动忽略，但是可以通过`model_dump(exclude_none = False)` 保留
	- 直接转化成json  `model_dump_json()`
	- 字典转成pydantic对象，自动校验model_validate
### pydantic基础教程

``` bash
uv add pydantic
```

``` python
from pydantic import BaseModel
#定义模型类，继承BaseModel，打造严格的容器
class a(BaseModel)
	#必选字段，只写类型注释没有默认值，必须要传入值
	#不能为空（min_length=1），备注说明
	#Field为更进阶的限制，如果不需要可以不添加
    path:str = Field(..., min_length=1, description="视频文件路径，不能为空") 
    #可选的字段，默认值就是None
    #可选浮点数，必须≥0
    start_time: float | None = Field(None, ge=0, description="开始时间，≥0秒") 
    end_time: float | None = None
    #模型初始化以后自动执行，进一步校验一些逻辑
    def model_post_init(self. __context)
    if self.start_time is not None and self.end_time is not None:
	    if self.end_time <= self.start_time:
		    raise ValueError("结束时间必须要大于开始时间")
try
	params = VideoToolParams(path="/test.mp4", start_time=8.0, end_time=13.0) 
	print("参数校验通过！") 
except ValidationError as e: 
# 捕获Pydantic的校验异常，打印详细错误信息 
	print(f"参数校验失败：{e.json()}")

#普通字典（比如API返回的参数） 
api_dict = {"path": "/test.mp4", "start_time": 8.0, "end_time": 13.0} 
# 字典转Pydantic对象，自动校验 
params = a.model_validate(api_dict) 
print(params.path) # →/test.mp4

```

## dic.get(key): 字典的安全取值方式
防止普通取值的致命问题：如果没有这个键，会抛出 **KeyError** 异常而使得程序崩溃
`dic.get(key)` 如果没有值会返回default或者None

```python
dic.get("start_time", 0.0)
```
## 语法糖

**语法糖**就是 Python 为了让代码**更简洁、更易读**设计的**简化写法**

```python
dic = {i: i*2 for i in [1,2,3]}

if (n := len([1,2,3])) > 2:  #:=海象运算符
print(n)

```
## my_dict.itmes()
返回固定值，**打包成「(key, value)」格式的元组**，并返回一个包含所有元组的迭代器
元组解包, 解包变量数量和元素数量不一致会报错
```python
# 元组解包：把元组的2个元素，依次赋值给a和b 
a, b = (1, 2) 
print(a) # →1，
print(b)→2 

# 列表解包：同理 
c, d = [3, 4] 
print(c) # →3，
print(d)→4

for k, v in my_dict.items():
	print(f"键：{k}，值：{v}")
```

## self 实例自己
- 类中的**所有实例方法**（给实例用的方法），**第一个参数必须是 self**，这是 Python 的**强制语法**；
- 调用方法时，**不用手动给 self 传值**，Python 会自动把「当前实例对象」传给 self；
- 在方法内部，通过`self.属性名`/`self.方法名()`，就能操作**当前实例**的属性和方法。

## __双下划线
- **用法 1：命名约定（私有 / 内部参数）**
	- 这个变量 / 参数是「内部的、仅供函数 / 类内部使用」，外部代码不要直接调用 / 修改
	- 你可以把`__context`理解为「占位参数」，为了符合 Pydantic 的方法格式
- **用法 2：魔法方法（Python 内置的特殊方法）**
	- `__init__`是实例初始化
	- `__str__`是实例的字符串打印
	- `__dict__`是实例的属性字典

```python
class Student: 
  def __init__(self, name, age): 
  self.name = name 
  self.age = age 
  # 自定义打印格式，Python自动调用 
  def __str__(self): 
  return f"学生：{self.name}，年龄：{self.age}" 
  
xiaoming = Student("小明", 10) 
print(xiaoming) # 自动调用__str__，输出：学生：小明，年龄：10 
# 没有__str__的话，会打印：<__main__.Student object at 0x7fxxxx>（内存地址）
```

## _ 单下划线
- 表示 临时变量 / 不用的变量
- **模块内的私有变量 / 函数**：
## hasattr & getattr的深度解析
内置的反射函数，作用是安全判断获取对象属性（处理reasoning_content）
- `hasattr(obj, attr_name)` 判断是否有某个属性
	- 检查`obj`（任意 Python 对象）中是否存在名为`attr_name`（字符串）的属性 / 方法，返回布尔值`True/False`
- `getattr(obj, attr_name, default=None)`：获取对象的某个属性
	- 若传了`default`，返回默认值；
	- 若没传`default`，直接抛出`AttributeError`（属性不存在错误）
可以在**运行时动态判断 / 获取属性**，而不是写死在代码里
OpenAI开发的SDK中，ChoiceDelta中没有“resoning_content”
> 如果直接写`choice.delta.reasoning_content`，Python 会检查`ChoiceDelta`类是否有这个属性，发现没有就直接抛出 **`AttributeError`**，程序崩溃

Kimi 返回的**实例对象**：在官方属性基础上，兼容 OpenAI SDK,**动态加了`reasoning_content`属性**（动态拓展）。
***hasattr & getattr 优雅处理属性不存在的情况***

动态绑定属性操作（其实就是替换）
```python
# 1. 接收kimi API返回的JSON字符串 
api_json = '{"choices": [{"delta": {"reasoning_content": "思考内容", "content": null}}]}' api_data = json.loads(api_json) 
# 2. OpenAI SDK原生解析为实例 
delta = ChoiceDelta(**api_data["choices"][0]["delta"]) # 原生仅解析content/role等官方字段 
# 3. Kimi官方拓展：动态绑定reasoning_content属性 
if "reasoning_content" in api_data["choices"][0]["delta"]: 
delta.reasoning_content = api_data["choices"][0]["delta"]["reasoning_content"]
```
## """+.strip()
面对多行字符串的标准写法，并且可在其中包含不用转义的引号
`.strip()` 删除首尾空白问题
## global关键词
在局部作用域中，声明使用「全局作用域」的变量

## main
**只有直接运行这个文件时，才执行 main**
被别的文件 import 时，**不执行！**
这就是：**可导入、可复用、不污染、不自动执行**

## append VS extend
- append添加在后面
- extend添加在前面
## async & await
异步处理```python
```python
async def f(): 
	await 网络请求 # 等，但不卡住别人 
	做别的
#运作整个异步程序
asyncio.run(main())
	
```
但是不能直接被调用，而是必须使用 `asyncio.run()` 调用
## argparse
 可以在运行代码的时候直接在命令行加参数
 ```python
 parser = argparse.ArgumentParser(description="Chat with formula tools") #创建一个参数解析器
 parser.add_argument( 
	 "--formula", 
	 action="append", #可以传入多个参数
	 default=["moonshot/web-search:latest"], 
	 help="Formula URIs",
 )
 ```
- type = 参数类型限制
- required = True必须传入这个参数
- choices = ["a", "b", "c"]只能在给定的列表里面选择
- nargs = “？”可选参数，“\*”接受多个值，放进列表
- dest = "new_name" 给参数起内部别名
## assert
 `assert 条件，错误信息`
 如果条件不成立，直接报错并停止程序
## try  & except & finally
```python
try: 
	运行可能出错的代码 
except Exception:
	如果try出错那就跳转到这里
finally: 
	无论是否出错，一定执行
	
# 不管程序正常结束、崩溃、报错，都一定关闭网络连接！
try: 
	聊天逻辑 
finally: 
	await client.close()
```

## 去重并且保留顺序
1. `dict.fromkeys(normalized_formulas)`
	- 字典的key不会重复，所以此时会被整理为`key: None`
2. list(dict...)
	- 把字典的键变回列表
## dict(zip())
把两列数据，一对一对拼成字典！
拉链函数zip逐行配对

## 生成器（Generator）
表达式：`(表达式 for 变量 in 可迭代对象)`
生成器 = 一边循环、一边计算、一边吐出值的 “惰性迭代器”
写法上就是把列表推导式的 `[]` 换成 `()`，或者用 `yield`
本质上等价于一个带 `yield` 的函数
- **不会立刻生成全部结果**
- 只返回一个 “生成器对象”，**内存极小**
- 只有遍历它时，才一个个计算
## any()
惰性计算 + 节省内存 + 提前终止
语法规则：
- 接受一个**可迭代对象**（生成器就是）
- 内部会自动遍历它
- 只要拿到一个 `True`，就**立刻停止遍历**（短路）
- 不继续往下算，节省性能
## set()集合
set = 无序、不重复的容器
自动去重，可以取差集，做集合的减法
## DataFrame行迭代器
`for index, row in df.iterrows()`

