# FastAPI 第一天学习笔记

## 1. 环境准备

首先进入自己已有的 Conda 环境：

```bash
conda activate spider
```
然后安装 FastAPI。这里使用官方推荐的标准安装方式：
```bash
pip install "fastapi[standard]"
```
这条命令会安装 FastAPI 以及常用的标准依赖，适合入门学习和日常开发。

## 2. 第一个 FastAPI 程序

先写一个最简单的 FastAPI 示例：
```python
from fastapi import FastAPI
app = FastAPI()
@app.get("/")
def first():
    return {"message": "Hello World"}
```
## 3. 启动开发服务器

运行下面的命令启动服务：

```bash
fastapi dev root.py
或
uvicorn root:app --reload
```

启动成功后，终端会显示类似内容：
```
Uvicorn running on http://127.0.0.1:8000
```
这说明当前应用已经运行在本地的 8000 端口上，可以通过浏览器访问：

http://127.0.0.1:8000
## 4. FastAPI 自动生成的文档

FastAPI 很方便的一点是，它会自动生成 API 文档。

启动项目后，可以访问下面几个地址：
```
/docs：交互式 API 文档
/redoc：另一种风格的 API 文档
/openapi.json：查看自动生成的 OpenAPI 描述文件
```
其中：
```
/docs 最常用，适合调试接口
/redoc 更适合阅读
/openapi.json 用来查看接口的结构化描述
```
例如访问 /openapi.json 时，可以看到类似这样的内容：
```
{
  "openapi": "3.1.0",
  "info": {
    "title": "FastAPI",
    "version": "0.1.0"
  },
  "paths": {
    "/items/": {
      "get": {
        "responses": {
          "200": {
            "description": "Successful Response",
            "content": {
              "application/json": {}
            }
          }
        }
      }
    }
  }
}
```
这说明 FastAPI 会自动根据代码生成接口文档，不需要手动编写。

## 5. HTTP 请求方法

在学习接口开发时，需要先认识常见的 HTTP 请求方法。

常见方法有：
```
GET
POST
PUT
DELETE

另外还有一些不太常见的方法：

OPTIONS
HEAD
PATCH
TRACE
```
目前第一天主要接触的是 GET 方法，因为最基础的接口通常都是从读取数据开始学习。

## 6. 路径参数

路径参数就是写在 URL 路径中的变量。

例如下面这段代码：
```python
@app.get("/item/me")
def read_item_me():
    return {"it123": "me123"}

@app.get("/item/{item_id}")
def read_item_str(item_id: str):
    return {"item": item_id}
```
这里有两个路径：
```
/item/me
/item/{item_id}
```
其中：
```
/item/me 是固定路径
/item/{item_id} 中的 item_id 是路径参数
```
当访问：
```
/item/abc
```
返回结果会是：
```
{"item": "abc"}
```
路径匹配顺序要注意

路径匹配是按顺序来的，谁先写，谁先匹配。

所以固定路径：
```
@app.get("/item/me")
```
要写在参数路径：
```
@app.get("/item/{item_id}")
```
前面。否则访问 /item/me 时，me 可能会被当作 item_id 的值。

这说明在定义路由时，顺序是很重要的。

## 7. 预定义值：枚举

如果一个路径参数只能取固定的几个值，就可以使用枚举来限制。

示例代码如下：
```python
from enum import Enum
from fastapi import FastAPI

class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"

app = FastAPI()

@app.get("/")
def root():
    return {"root": "这是root路径"}

@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    if model_name is ModelName.alexnet:
        return {"model_name": model_name, "message": "Deep Learning FTW!"}

    if model_name.value == "lenet":
        return {"model_name": model_name, "message": "LeCNN all the images"}

    return {"model_name": model_name, "message": "Have some residuals"}
```
这里定义了一个枚举类 ModelName，它的值只能是：
```
alexnet
resnet
lenet
```
这样做有几个好处：
```
1)限制参数取值范围
2)提高代码可读性
3)自动在文档中显示可选值
```
也就是说，枚举适合用来处理“只能从几个固定选项中选择一个”的场景。

## 8. 查询参数

查询参数是 URL 中 ? 后面的内容。

例如：
```
/item/?skip=1&limit=2
```
这里的：
```
skip=1
limit=2
```

就是查询参数。

### 8.1 基础查询参数示例
```python
from fastapi import FastAPI
app = FastAPI()
items = [{"1": "我"}, {"2": "爱"}, {"3": "学习"}]

@app.get("/item/")
async def get_item(skip: int = 0, limit: int = 1):
    return items[skip: skip + limit]
```
在这个例子中：

skip 默认值是 0,
limit 默认值是 1

如果访问：
```
/item/?skip=1&limit=2
```
就会从列表中跳过第一个元素，然后取两个元素。

这说明查询参数通常用于：

分页
筛选
控制返回结果数量
### 8.2 可选查询参数
```python
from typing import Optional

@app.get("/items/{item_id}")
async def read_item(item_id: str, q: Optional[str] = None):
    if q:
        return {"item_id": item_id, "q": q}
    return {"item_id": item_id}
```
这个例子中：
```
item_id 是路径参数
q 是查询参数
```
q: Optional[str] = None 表示：
q 是字符串类型

这个参数可以不传
不传时默认值是 None

如果访问：
```
/items/123
```
返回：
```
{"item_id": "123"}
```
如果访问：
```
/items/123?q=hello
```
返回：
```
{"item_id": "123", "q": "hello"}
```
### 8.3 路径参数和查询参数一起使用
```python
from typing import Optional
@app.get("/wq/{opt}/milk-tea/{order}")
async def shop(opt: str, order: str, num: Optional[int] = None, money: Optional[int] = None):
    print("opt =", opt, "order =", order, "num =", num, "money =", money)

    if num is not None and money is not None:
        return {"opt": opt, "order": order, "num": num, "money": money}
    elif num is not None:
        return {"opt": opt, "order": order, "num": num}
    elif money is not None:
        return {"opt": opt, "order": order, "money": money}
    else:
        return {"opt": opt, "order": order, "mes": "看了看没舍得买"}
```
这个例子中：

opt 和 order 是路径参数, 
num 和 money 是查询参数

例如访问：
```
/wq/buy/milk-tea/pearl?num=2&money=30
```
可以得到带有完整信息的返回结果。

这个例子说明了 FastAPI 可以同时处理：

路径中的参数
查询字符串中的参数

这也是接口开发中非常常见的一种写法。

## 9. 今天学到的核心内容

通过第一天的学习，我对 FastAPI 有了一个基础认识，主要包括以下内容。

### 9.1 FastAPI 的基本特点
```
1)写法简洁
2)支持类型注解
3)自动生成 API 文档
4)支持异步
5)运行性能高
```
### 9.2 参数的几种形式

在 FastAPI 中，常见参数主要有三类：
| 参数类型 | 说明               | 示例              |
|----------|--------------------|-------------------|
| 路径参数 | 写在 URL 路径中    | /items/123        |
| 查询参数 | 写在 ? 后面        | /items/?skip=1    |
| 枚举参数 | 限制可选值范围     | alexnet、resnet   |
### 9.3 需要特别注意的地方
```
路由匹配顺序很重要
类型注解不仅是提示，也会参与校验
FastAPI 会自动生成接口文档
查询参数一般通过默认值来设置是否可选
```
## 10. 第一天学习总结

今天主要完成了 FastAPI 的入门学习，内容包括：
```
1)安装 FastAPI
2)编写第一个 FastAPI 程序
3)启动本地开发服务器
4)了解自动生成的接口文档
5)学习 HTTP 请求方法
6)学习路径参数
7)学习枚举参数
8)学习查询参数
```
这些内容是后续继续学习请求体、Pydantic、POST 接口和数据库操作的基础。

## 11. 我的理解

经过第一天的学习，我觉得 FastAPI 最大的特点就是：
```
1)上手简单
2)代码清晰
3)文档自动生成非常方便
4)类型注解让接口定义更规范
```
相比于只写普通函数，FastAPI 更像是在用“声明参数类型”的方式定义接口，这样不仅代码更直观，而且开发和调试都更方便。

## 12. 一句话总结

FastAPI 是一个基于 Python 类型注解的高性能 Web 框架，可以快速开发接口，并自动生成清晰的 API 文档。
