## 课程目标

本课通过实操创建一个最小 FastAPI 服务，理解如何创建项目、编写接口、启动服务，并通过浏览器和 Swagger UI 验证接口。

本课最终完成两个接口：

- `GET /health`：检查服务是否正常
- `POST /chat`：接收消息并返回结果

## 一、创建项目目录

```powershell
mkdir fastapi-demo
```

### 命令含义

- `mkdir` 表示创建文件夹。
- `fastapi-demo` 是文件夹名称。

### 这一步的作用

为 FastAPI 项目建立独立的工作目录，方便管理代码，避免项目文件散落在桌面上。

## 二、进入项目目录

```powershell
cd fastapi-demo
```

### 命令含义

- `cd` 表示进入指定目录。
- `fastapi-demo` 表示进入刚才创建的文件夹。

进入后，命令行路径应类似：

```text
C:\Users\Administrator\Desktop\fastapi-demo
```

### 遇到的问题

已经在 `fastapi-demo` 目录时，不能再次输入：

```powershell
cd Desktop\fastapi-demo
```

否则 PowerShell 会拼接出错误路径：

```text
C:\Users\Administrator\Desktop\fastapi-demo\Desktop\fastapi-demo
```

判断当前目录的方法是查看命令提示符中的完整路径。

## 三、创建 Python 文件

```powershell
notepad main.py
```

### 命令含义

- `notepad` 表示打开 Windows 记事本。
- `main.py` 是 FastAPI 服务的 Python 文件。

### 这一步的作用

FastAPI 代码必须写入 Python 文件中，不能直接把 `from`、`def`、`@app.get` 等 Python 代码输入 PowerShell。PowerShell 负责执行系统命令，Python 文件负责保存程序代码。

## 四、编写 FastAPI 服务

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class ChatRequest(BaseModel):
    message: str


@app.get("/health")
def health():
    return {"status": "ok"}


@app.post("/chat")
def chat(request: ChatRequest):
    return {"reply": f"你发送的是：{request.message}"}
```

### 1. 导入 FastAPI

```python
from fastapi import FastAPI
```

导入 FastAPI 框架，用来创建 Web API 服务。

### 2. 导入 BaseModel

```python
from pydantic import BaseModel
```

导入数据模型工具，用来规定请求体的格式和字段类型。

### 3. 创建应用对象

```python
app = FastAPI()
```

创建 FastAPI 应用对象。启动命令中的 `main:app` 表示加载 `main.py` 文件中的 `app` 对象：

- `main`：Python 文件名，不包含 `.py`
- `app`：文件中的 FastAPI 应用对象

### 4. 定义请求体

```python
class ChatRequest(BaseModel):
    message: str
```

规定 `/chat` 接口接收的数据必须包含字符串类型的 `message` 字段，例如：

```json
{
  "message": "你好"
}
```

### 5. 定义健康检查接口

```python
@app.get("/health")
def health():
    return {"status": "ok"}
```

`@app.get` 创建 GET 接口，`/health` 是接口地址。这个接口不处理实际业务，只返回状态，用来判断服务是否能够正常运行。

### 6. 定义聊天接口

```python
@app.post("/chat")
def chat(request: ChatRequest):
    return {"reply": f"你发送的是：{request.message}"}
```

`@app.post` 创建 POST 接口。接口接收符合 `ChatRequest` 结构的数据，读取 `message` 字段，再把内容放进 `reply` 字段返回。

当前接口只是把消息原样返回，还没有连接 Ollama。它的重点是理解 API 的请求、处理和返回过程。

## 五、启动服务

```powershell
python -m uvicorn main:app --reload
```

### 命令含义

- `python -m uvicorn`：使用 Python 启动 Uvicorn 服务。
- `main:app`：加载 `main.py` 中的 `app` 对象。
- `--reload`：代码修改后自动重新加载服务。

启动成功时会看到：

```text
Uvicorn running on http://127.0.0.1:8000
Application startup complete.
```

### 这一步的作用

把 Python 代码启动成一个可以通过网络访问的 Web API 服务。

### 为什么 PowerShell 看起来像卡住

服务启动后，PowerShell 会持续运行服务并等待请求，不会重新显示命令提示符。这不是卡死，而是服务正在工作。停止服务可以按 `Ctrl+C`。

## 六、验证健康检查接口

浏览器访问：

```text
http://127.0.0.1:8000/health
```

正常返回：

```json
{
  "status": "ok"
}
```

### 验证结果含义

返回 `200 OK` 说明：

- FastAPI 服务已启动
- 8000 端口可以访问
- `/health` 路由存在
- 服务能够正确返回 JSON 数据

## 七、使用 Swagger UI 验证接口

浏览器访问：

```text
http://127.0.0.1:8000/docs
```

FastAPI 会自动生成 Swagger UI，用来查看接口、查看请求体格式、发送测试请求并查看返回结果。

页面中出现 `GET /health` 和 `POST /chat`，说明两个接口都已经被 FastAPI 识别。

## 八、验证 POST /chat

在 Swagger UI 中：

1. 打开绿色的 `POST /chat`。
2. 点击 `Try it out`。
3. 输入请求体：

```json
{
  "message": "你好"
}
```

4. 点击 `Execute`。

正常返回：

```json
{
  "reply": "你发送的是：你好"
}
```

状态码为 `200`，说明请求体、接口处理逻辑和返回体都正常。

## 九、请求体和返回体

请求体是客户端发送给服务的数据：

```json
{
  "message": "你好"
}
```

返回体是服务处理后返回给客户端的数据：

```json
{
  "reply": "你发送的是：你好"
}
```

可以简单理解为：

- 请求体：我给服务什么信息
- 返回体：服务处理后给我什么结果

## 十、这几个提示的含义

### PowerShell 配置文件报错

PowerShell 启动配置文件的执行策略提示与 FastAPI 本身无关。因为 FastAPI 后续已经成功启动，所以本课不需要处理它。

### `favicon.ico 404 Not Found`

浏览器会自动请求网页图标。如果项目没有配置图标，就会出现这个提示。它不影响 FastAPI 服务、`/health` 接口和 `/chat` 接口。

## 十一、本课的完整工作链路

```text
创建项目目录
    ↓
编写 main.py
    ↓
启动 Uvicorn 服务
    ↓
浏览器访问 /health 检查服务
    ↓
Swagger UI 发现并测试接口
    ↓
POST /chat 接收请求
    ↓
FastAPI 处理请求并返回 JSON
```

## 十二、本课最终成果

本课成功创建并验证了一个最小 FastAPI 服务：

- `GET /health`：服务健康检查
- `POST /chat`：接收消息并返回结果
- Swagger UI：接口查看和测试工具
- Uvicorn：FastAPI 服务运行服务器
- Pydantic：请求数据格式验证工具

当前 `/chat` 还没有调用 Ollama，只是完成了 API 服务外壳。后续需要把简单的返回逻辑替换为：

```text
接收用户问题
    ↓
FastAPI 调用 Ollama API
    ↓
Ollama 调用模型
    ↓
FastAPI 整理模型回答
    ↓
返回给客户端
```

第20课已完成。
