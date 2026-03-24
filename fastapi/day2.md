
# FastAPI 第二天学习笔记

## 1. 环境准备

如果你已经安装好了 FastAPI，通常可以直接继续学习。
如果还没有安装，可以先进入自己的虚拟环境，再安装 FastAPI。

先进入 Conda 环境：

```bash
conda activate spider
```

安装 FastAPI 及常用依赖：

```bash
pip install "fastapi[standard]"
```

如果想确认版本，尤其是后面会用到 `Annotated` 和查询参数模型这些功能，建议检查 FastAPI 是否较新：

```bash
pip show fastapi
```

说明：

* `Annotated` 推荐在较新的 FastAPI 版本中使用
* 查询参数模型功能从较新的 FastAPI 版本开始支持
* 学习过程中尽量保持 `fastapi` 和 `pydantic` 为较新版本

---

## 2. 第一个示例（核心入门代码）

今天学习的核心是：**请求体（Request Body）**。

下面先写一个最基础的请求体示例：

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None


@app.post("/items/")
async def create_item(item: Item):
    return item
```

这段代码的作用是：

* 使用 `BaseModel` 定义一个请求数据模型 `Item`
* 客户端向 `/items/` 发送 `POST` 请求时，可以提交一个 JSON 数据
* FastAPI 会自动把这个 JSON 转换成 `Item` 对象
* 如果数据格式不对，FastAPI 会自动返回错误信息

例如，请求体可以写成：

```python
{
    "name": "苹果",
    "description": "一种水果",
    "price": 5.5,
    "tax": 0.5
}
```

如果 `description` 和 `tax` 不传，也是可以的，因为它们有默认值 `None`。

---

## 3. 运行方式

启动开发服务器可以使用下面的命令：

```bash
fastapi dev main.py
```

或者：

```bash
uvicorn main:app --reload
```

如果文件名不是 `main.py`，要记得改成你自己的文件名。
例如你的文件叫 `root.py`，那就应该写成：

```bash
uvicorn root:app --reload
```

启动成功后，终端会显示类似内容：

```bash
Uvicorn running on http://127.0.0.1:8000
```

这表示当前项目已经运行在本地 `8000` 端口上。

可以在浏览器访问：

```bash
http://127.0.0.1:8000
```

如果接口是 `/items/`，那么测试地址一般是：

```bash
http://127.0.0.1:8000/items/
```

说明：

* `GET` 请求通常直接在浏览器访问
* `POST`、`PUT` 等带请求体的请求，更适合在 `/docs` 中测试
* `--reload` 表示代码修改后自动重启，适合开发阶段

---

## 4. 自动功能/框架特点（如文档、工具等）

FastAPI 的一个重要特点是：**自动生成接口文档**。
今天学习请求体、参数验证之后，这个特点会更加明显。

启动项目后，可以访问以下地址：

```bash
/docs
/redoc
/openapi.json
```

完整地址分别是：

```bash
http://127.0.0.1:8000/docs
http://127.0.0.1:8000/redoc
http://127.0.0.1:8000/openapi.json
```

它们的作用如下：

* `/docs`：交互式 API 文档，最常用，可以直接测试接口
* `/redoc`：另一种风格的文档页面，更适合阅读
* `/openapi.json`：自动生成的接口描述文件

今天学到的很多内容，都会自动体现在文档里，例如：

* 请求体的结构
* 字段类型
* 哪些字段必填
* 字符串长度限制
* 数字范围限制
* 参数标题、描述、是否弃用等信息

这说明 FastAPI 不只是“写接口”，还会帮你自动完成很多文档工作。

---

## 5. 核心基础概念

今天主要学习的是 FastAPI 中和**请求数据**有关的内容，核心概念有以下几类。

### 5.1 请求体（Request Body）

请求体就是客户端发送给服务器的数据。

例如前端提交一个商品信息：

```python
{
    "name": "Milk Tea",
    "price": 18.5
}
```

这个 JSON 数据就是请求体。

常见使用场景：

* 新增数据
* 修改数据
* 提交表单内容
* 提交复杂对象

通常以下请求方法更常使用请求体：

```bash
POST
PUT
PATCH
DELETE
```

注意：

* `GET` 一般不使用请求体
* 虽然 FastAPI 对极端情况也支持，但通常不推荐在 `GET` 中传请求体

---

### 5.2 Pydantic 模型

FastAPI 常用 Pydantic 来定义数据模型。

示例：

```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float
```

作用：

* 定义数据结构
* 自动校验类型
* 自动生成文档
* 提供编辑器提示和自动补全

也就是说，Pydantic 模型就像是“请求数据的标准模板”。

---

### 5.3 参数来源的自动识别

FastAPI 会根据参数类型自动判断数据来自哪里。

常见规则如下：

| 参数形式            | FastAPI 识别方式 | 示例                 |
| --------------- | ------------ | ------------------ |
| 在路径中声明的参数       | 路径参数         | `/items/{item_id}` |
| 普通类型参数          | 查询参数         | `q: str = None`    |
| Pydantic 模型类型参数 | 请求体          | `item: Item`       |

这也是 FastAPI 很方便的地方：
**你只要把参数类型写清楚，框架就能自动理解你的意思。**

---

### 5.4 参数验证

FastAPI 支持对参数做很多验证，例如：

* 字符串最小长度
* 字符串最大长度
* 正则匹配
* 数字范围限制
* 是否必填
* 是否允许额外字段

这些验证既能保护接口，也能让接口文档更清晰。

---

## 6. 核心语法/功能一（请求体与路径参数、Body、多请求体）

今天最核心的内容之一，就是**请求体的声明和使用**。

### 6.1 基础请求体示例

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None


@app.post("/items/")
async def create_item(item: Item):
    return item
```

访问方式：

```bash
POST /items/
```

请求体示例：

```python
{
    "name": "奶茶",
    "description": "珍珠奶茶",
    "price": 16.5,
    "tax": 1.5
}
```

返回结果：

```python
{
    "name": "奶茶",
    "description": "珍珠奶茶",
    "price": 16.5,
    "tax": 1.5
}
```

说明：

* `item: Item` 表示这个参数从请求体中读取
* FastAPI 会自动把 JSON 转成 `Item` 对象
* 如果 `price` 传成字符串且无法转换，会自动报错

---

### 6.2 在函数内部使用模型

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None


@app.post("/items/")
async def create_item(item: Item):
    item_dict = item.model_dump()
    if item.tax is not None:
        item_dict.update({"price_with_tax": item.price + item.tax})
    return item_dict
```

请求体：

```python
{
    "name": "咖啡",
    "price": 20,
    "tax": 2
}
```

返回结果：

```python
{
    "name": "咖啡",
    "description": null,
    "price": 20.0,
    "tax": 2.0,
    "price_with_tax": 22.0
}
```

说明：

* `item.model_dump()` 可以把模型转成字典
* 可以直接访问 `item.price`、`item.tax` 等属性
* 比直接处理 `dict` 更清晰，也更安全

---

### 6.3 请求体 + 路径参数

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    return {"item_id": item_id, **item.model_dump()}
```

访问示例：

```bash
PUT /items/101
```

请求体：

```python
{
    "name": "耳机",
    "price": 199.0
}
```

返回结果：

```python
{
    "item_id": 101,
    "name": "耳机",
    "description": null,
    "price": 199.0,
    "tax": null
}
```

说明：

* `item_id` 是路径参数
* `item` 是请求体参数
* FastAPI 会自动区分它们的来源

---

### 6.4 请求体 + 路径参数 + 查询参数

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item, q: str | None = None):
    result = {"item_id": item_id, **item.model_dump()}
    if q:
        result.update({"q": q})
    return result
```

访问示例：

```bash
PUT /items/8?q=discount
```

请求体：

```python
{
    "name": "键盘",
    "price": 299
}
```

返回结果：

```python
{
    "item_id": 8,
    "name": "键盘",
    "description": null,
    "price": 299.0,
    "tax": null,
    "q": "discount"
}
```

---

### 6.5 多个请求体参数

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None


class User(BaseModel):
    username: str
    full_name: str | None = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item, user: User):
    return {"item_id": item_id, "item": item, "user": user}
```

这时请求体要写成：

```python
{
    "item": {
        "name": "鼠标",
        "description": "无线鼠标",
        "price": 89.9,
        "tax": 1.2
    },
    "user": {
        "username": "wq",
        "full_name": "Wang Qiao"
    }
}
```

返回结果会包含 `item_id`、`item`、`user` 三部分内容。

说明：

* 当有多个 Pydantic 模型参数时
* FastAPI 会把参数名作为请求体中的键名

---

### 6.6 单个普通值放进请求体中

默认情况下，像 `importance: int` 这种普通类型参数，会被识别为查询参数。
如果你希望它放在请求体里，就要用 `Body()`。

```python
from typing import Annotated
from fastapi import Body, FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    price: float


class User(BaseModel):
    username: str


@app.put("/items/{item_id}")
async def update_item(
    item_id: int,
    item: Item,
    user: User,
    importance: Annotated[int, Body()]
):
    return {"item_id": item_id, "item": item, "user": user, "importance": importance}
```

请求体：

```python
{
    "item": {
        "name": "书",
        "price": 50
    },
    "user": {
        "username": "wq"
    },
    "importance": 5
}
```

---

### 6.7 单个请求体参数嵌入到键中

```python
from typing import Annotated
from fastapi import Body, FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    price: float


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Annotated[Item, Body(embed=True)]):
    return {"item_id": item_id, "item": item}
```

这时请求体要写成：

```python
{
    "item": {
        "name": "平板",
        "price": 2999
    }
}
```

而不是直接写：

```python
{
    "name": "平板",
    "price": 2999
}
```

---

### ⚠️ 注意事项

* 请求体一般搭配 `POST`、`PUT`、`PATCH` 使用
* Pydantic 模型字段有默认值时，就是可选字段
* 没有默认值时，就是必填字段
* 普通类型默认会被识别为查询参数，不是请求体
* 如果只有一个请求体模型参数，默认请求体就是这个模型本身
* 如果用了 `Body(embed=True)`，则必须多包一层键名

---

## 7. 核心语法/功能二（模型验证、Field、嵌套模型）

今天第二个重点是：**如何让请求体更严格、更规范。**

### 7.1 使用 Field 给模型字段加验证

```python
from typing import Annotated
from fastapi import Body, FastAPI
from pydantic import BaseModel, Field

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str | None = Field(
        default=None,
        title="商品描述",
        max_length=300
    )
    price: float = Field(gt=0, description="价格必须大于 0")
    tax: float | None = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Annotated[Item, Body(embed=True)]):
    return {"item_id": item_id, "item": item}
```

说明：

* `Field()` 用于模型内部字段的验证和元数据声明
* `gt=0` 表示必须大于 0
* `max_length=300` 表示最大长度 300
* `title` 和 `description` 会显示到文档中

使用场景：

* 需要对请求体字段做更精细限制
* 希望接口文档更完整
* 希望前后端对字段含义更清楚

优点总结：

1. 字段规则更明确
2. 文档更完整
3. 错误提示更友好
4. 便于大型项目统一数据规范

---

### 7.2 列表字段与集合字段

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    price: float
    tags: list[str] = []


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    return {"item_id": item_id, "item": item}
```

请求体示例：

```python
{
    "name": "机械键盘",
    "price": 399,
    "tags": ["数码", "键盘", "办公"]
}
```

如果想自动去重，可以使用 `set[str]`：

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    price: float
    tags: set[str] = set()


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    return {"item_id": item_id, "item": item}
```

说明：

* `list[str]` 表示字符串列表
* `set[str]` 表示字符串集合，自动去重
* 适合标签、分类等场景

---

### 7.3 嵌套模型

```python
from fastapi import FastAPI
from pydantic import BaseModel, HttpUrl

app = FastAPI()


class Image(BaseModel):
    url: HttpUrl
    name: str


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: set[str] = set()
    image: Image | None = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    return {"item_id": item_id, "item": item}
```

请求体示例：

```python
{
    "name": "演唱会门票",
    "description": "VIP票",
    "price": 888.0,
    "tax": 20.0,
    "tags": ["音乐", "现场", "热门"],
    "image": {
        "url": "http://example.com/poster.jpg",
        "name": "poster"
    }
}
```

说明：

* `image: Image | None` 表示一个字段本身又是另一个模型
* `HttpUrl` 可以自动验证 URL 是否合法
* 这种写法很适合描述复杂 JSON 数据

---

### 7.4 嵌套模型列表

```python
from fastapi import FastAPI
from pydantic import BaseModel, HttpUrl

app = FastAPI()


class Image(BaseModel):
    url: HttpUrl
    name: str


class Item(BaseModel):
    name: str
    price: float
    images: list[Image] | None = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    return {"item_id": item_id, "item": item}
```

请求体示例：

```python
{
    "name": "相册",
    "price": 99.9,
    "images": [
        {
            "url": "http://example.com/1.jpg",
            "name": "封面"
        },
        {
            "url": "http://example.com/2.jpg",
            "name": "内页"
        }
    ]
}
```

---

### 7.5 深层嵌套模型

```python
from fastapi import FastAPI
from pydantic import BaseModel, HttpUrl

app = FastAPI()


class Image(BaseModel):
    url: HttpUrl
    name: str


class Item(BaseModel):
    name: str
    price: float
    images: list[Image] | None = None


class Offer(BaseModel):
    name: str
    description: str | None = None
    price: float
    items: list[Item]


@app.post("/offers/")
async def create_offer(offer: Offer):
    return offer
```

这说明：

* 一个模型里可以放另一个模型列表
* 可以一层套一层
* FastAPI 依然能够自动验证和生成文档

---

## 8. 核心语法/功能三（查询参数验证、Path 数字验证、查询参数模型）

今天除了请求体，也学习了更强大的参数验证方式。

### 8.1 基础查询参数验证

```python
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: Annotated[str | None, Query(max_length=50)] = None):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

访问示例：

```bash
/items/?q=milk-tea
```

返回结果：

```python
{
    "items": [
        {"item_id": "Foo"},
        {"item_id": "Bar"}
    ],
    "q": "milk-tea"
}
```

说明：

* `Query(max_length=50)` 表示查询参数最大长度为 50
* `= None` 表示这个参数可选
* 推荐使用 `Annotated`

---

### 8.2 更完整的字符串验证

```python
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(
    q: Annotated[
        str | None,
        Query(min_length=3, max_length=50, pattern="^fixedquery$")
    ] = None,
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

说明：

* `min_length=3`：最少 3 个字符
* `max_length=50`：最多 50 个字符
* `pattern="^fixedquery$"`：必须完全匹配 `fixedquery`

---

### 8.3 查询参数元数据

```python
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(
    q: Annotated[
        str | None,
        Query(
            alias="item-query",
            title="Query string",
            description="用于搜索商品的查询字符串",
            min_length=3,
            max_length=50,
            deprecated=True,
        ),
    ] = None,
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

访问示例：

```bash
/items/?item-query=phone
```

说明：

* `alias="item-query"`：外部参数名改为 `item-query`
* `title`：参数标题
* `description`：参数说明
* `deprecated=True`：标记为弃用

---

### 8.4 多值查询参数

```python
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: Annotated[list[str] | None, Query()] = None):
    return {"q": q}
```

访问示例：

```bash
/items/?q=apple&q=banana&q=orange
```

返回结果：

```python
{
    "q": ["apple", "banana", "orange"]
}
```

说明：

* 同一个查询参数可以出现多次
* FastAPI 会自动收集成一个列表

---

### 8.5 路径参数数字验证

```python
from typing import Annotated
from fastapi import FastAPI, Path, Query

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(
    item_id: Annotated[int, Path(title="商品ID", gt=0, le=1000)],
    q: Annotated[str | None, Query(alias="item-query")] = None,
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

访问示例：

```bash
/items/15?item-query=keyboard
```

返回结果：

```python
{
    "item_id": 15,
    "q": "keyboard"
}
```

说明：

* `gt=0`：必须大于 0
* `le=1000`：必须小于等于 1000
* 路径参数始终是必填的

---

### 8.6 查询参数模型

如果一组查询参数经常一起使用，可以统一写成一个模型。

```python
from typing import Annotated, Literal
from fastapi import FastAPI, Query
from pydantic import BaseModel, Field

app = FastAPI()


class FilterParams(BaseModel):
    limit: int = Field(100, gt=0, le=100)
    offset: int = Field(0, ge=0)
    order_by: Literal["created_at", "updated_at"] = "created_at"
    tags: list[str] = []


@app.get("/items/")
async def read_items(filter_query: Annotated[FilterParams, Query()]):
    return filter_query
```

访问示例：

```bash
/items/?limit=10&offset=0&order_by=created_at&tags=python&tags=fastapi
```

返回结果类似：

```python
{
    "limit": 10,
    "offset": 0,
    "order_by": "created_at",
    "tags": ["python", "fastapi"]
}
```

这说明查询参数也可以像请求体一样模块化管理。

---

### 8.7 禁止额外查询参数

```python
from typing import Annotated, Literal
from fastapi import FastAPI, Query
from pydantic import BaseModel, Field

app = FastAPI()


class FilterParams(BaseModel):
    model_config = {"extra": "forbid"}

    limit: int = Field(100, gt=0, le=100)
    offset: int = Field(0, ge=0)
    order_by: Literal["created_at", "updated_at"] = "created_at"
    tags: list[str] = []


@app.get("/items/")
async def read_items(filter_query: Annotated[FilterParams, Query()]):
    return filter_query
```

如果传入未定义的参数，例如：

```bash
/items/?limit=10&tool=abc
```

就会报错。

说明：

* 适合接口要求严格的场景
* 可以防止客户端随意传无关参数

---

## 9. 今日核心知识总结

今天的重点可以概括为以下几个方面：

1. 学会了使用 `BaseModel` 定义请求体模型
2. 知道了 FastAPI 会自动把 JSON 请求体转换为 Python 对象
3. 学会了同时处理请求体、路径参数、查询参数
4. 学会了使用 `Body()` 控制普通值也从请求体中读取
5. 学会了使用 `Body(embed=True)` 改变请求体结构
6. 学会了使用 `Field()` 为模型字段添加校验和说明
7. 学会了使用嵌套模型、列表模型、深层模型来表示复杂数据
8. 学会了使用 `Query()` 对查询参数做长度、正则、别名、弃用等设置
9. 学会了使用 `Path()` 对路径参数做数字范围限制
10. 学会了使用查询参数模型统一管理一组查询参数

---

## 10. 今日学习总结

今天主要学习了 FastAPI 中更重要、更实用的参数处理部分，内容包括：

1. 学习了什么是请求体
2. 学习了如何使用 Pydantic 模型接收请求体
3. 学习了请求体与路径参数、查询参数混合使用
4. 学习了多个请求体参数的写法
5. 学习了 `Body()` 和 `Body(embed=True)` 的用法
6. 学习了 `Field()` 对模型字段进行验证
7. 学习了嵌套模型与复杂 JSON 的处理
8. 学习了查询参数的字符串验证
9. 学习了路径参数的数字验证
10. 学习了查询参数模型及禁止额外参数的写法

这些内容已经让 FastAPI 从“会写简单接口”进入到了“能写更规范接口”的阶段。

---

## 11. 我的理解

经过第二天的学习，我觉得 FastAPI 真正强大的地方，不只是“写接口快”，而是它把**参数声明、数据校验、文档生成、编辑器提示**这几件事统一起来了。

以前如果只用普通 Python 函数处理请求，往往需要自己判断：

* 这个字段有没有传
* 类型对不对
* 长度是否合法
* 文档要不要手写

但在 FastAPI 里，只要把类型和规则写好，很多事情框架就自动帮你做了。

尤其是今天学到的请求体和参数验证，让我感觉 FastAPI 很像是在“先把数据规则设计清楚，再写业务逻辑”。
这样代码会更规范，也更适合团队协作。

我觉得：

* `BaseModel` 是数据结构的核心
* `Query`、`Path`、`Body` 是参数控制的核心
* `Field` 是模型精细化设计的核心

如果把这些学扎实，后面学习 POST 接口、数据库、用户系统时会轻松很多。

---

## 12. 一句话总结

FastAPI 可以通过 Pydantic 模型和参数验证机制，清晰地定义请求体、查询参数和路径参数，并自动完成数据校验与接口文档生成。
