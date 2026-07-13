## 课程目标

本课在第20课 FastAPI 最小服务的基础上，让 `/chat` 接口真正调用 Ollama，并由 `qwen2.5:3b` 生成回答。

本课先完成一条可运行的完整链路：

```text
客户端提交问题
    ↓
FastAPI 接收 /chat 请求
    ↓
FastAPI 调用 Ollama API
    ↓
Ollama 调用 qwen2.5:3b 模型
    ↓
FastAPI 整理模型返回结果
    ↓
客户端收到回答
```

本课重点是先把全链路跑通。代码细节、参数修改、故障模拟和独立排错，放到后续作业中逐步深入。

## 一、前置条件

第20课已经完成以下内容：

- Python 3.10.11 已安装
- FastAPI 已安装
- Uvicorn 已安装
- FastAPI 项目目录为 `Desktop\fastapi-demo`
- 项目中有 `main.py`
- FastAPI 能通过 `127.0.0.1:8000` 访问
- `/health` 和 `/chat` 接口已经建立
- Ollama 已安装并可以运行

## 二、检查 Ollama 模型

```powershell
ollama list
```

### 命令含义

- `ollama`：调用 Ollama 命令行工具。
- `list`：列出本机已经下载的模型。

### 这一步的作用

确认 FastAPI 后面要调用的模型确实存在。本课使用的模型是：

```text
qwen2.5:3b
```

如果模型名称不存在，FastAPI 即使能够连接 Ollama，也会因为找不到模型而调用失败。

## 三、进入 FastAPI 项目目录

```powershell
cd "$env:USERPROFILE\Desktop\fastapi-demo"
```

### 命令含义

- `cd`：进入目录。
- `$env:USERPROFILE`：当前 Windows 用户的主目录。
- `Desktop\fastapi-demo`：桌面上的 FastAPI 项目目录。

### 这一步的作用

确保后续打开和修改的是正确的 `main.py` 文件。

使用完整路径可以避免已经在项目目录中时再次拼接路径，减少出现下面这种错误：

```text
C:\Users\Administrator\Desktop\fastapi-demo\Desktop\fastapi-demo
```

## 四、安装 requests

```powershell
python -m pip install requests
```

### 命令含义

- `python -m pip`：使用当前 Python 环境中的 pip。
- `install requests`：安装 requests HTTP 请求库。

### 这一步的作用

FastAPI 需要通过 HTTP 请求访问 Ollama 的接口。`requests` 可以帮助 Python 向 Ollama 发送 POST 请求，并接收 JSON 返回结果。

## 五、修改 main.py

```powershell
notepad main.py
```

打开文件后，将内容替换为：

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import requests

app = FastAPI()

OLLAMA_URL = "http://localhost:11434/api/generate"
MODEL = "qwen2.5:3b"


class ChatRequest(BaseModel):
    message: str


@app.get("/health")
def health():
    return {"status": "ok"}


@app.post("/chat")
def chat(request: ChatRequest):
    payload = {
        "model": MODEL,
        "prompt": request.message,
        "stream": False
    }

    try:
        response = requests.post(
            OLLAMA_URL,
            json=payload,
            timeout=120
        )

        response.raise_for_status()
        data = response.json()

        return {
            "reply": data.get("response", ""),
            "model": data.get("model"),
            "done": data.get("done")
        }

    except requests.exceptions.RequestException as error:
        raise HTTPException(
            status_code=502,
            detail=f"Ollama API 调用失败：{error}"
        )
```

保存并关闭记事本。

## 六、代码逐部分解释

### 1. 导入工具

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import requests
```

- `FastAPI`：创建 Web API 服务。
- `HTTPException`：当 Ollama 调用失败时，向客户端返回明确的 HTTP 错误。
- `BaseModel`：定义请求体格式。
- `requests`：让 FastAPI 能够发送 HTTP 请求给 Ollama。

### 2. 定义 Ollama 地址和模型

```python
OLLAMA_URL = "http://localhost:11434/api/generate"
MODEL = "qwen2.5:3b"
```

`OLLAMA_URL` 是 Ollama 的生成接口：

- `localhost`：当前 FastAPI 服务所在的 Windows 电脑。
- `11434`：Ollama API 使用的端口。
- `/api/generate`：生成回答的接口路径。

`MODEL` 指定具体调用的模型名称。

这里的 FastAPI 直接运行在 Windows 上，所以 `localhost:11434` 指向本机 Ollama。之前 Docker 中的 Open WebUI 如果要访问宿主机 Ollama，才需要考虑 `host.docker.internal`。

### 3. 定义请求体

```python
class ChatRequest(BaseModel):
    message: str
```

规定客户端提交给 `/chat` 的数据格式：

```json
{
  "message": "请用三句话解释什么是系统部署。"
}
```

FastAPI 会先检查请求体是否符合这个格式，再交给函数处理。

### 4. 保留健康检查接口

```python
@app.get("/health")
def health():
    return {"status": "ok"}
```

这个接口只检查 FastAPI 是否正常运行，不会调用 Ollama。因此即使模型调用失败，健康检查接口仍然可以帮助判断 FastAPI 服务本身是否正常。

### 5. 接收聊天请求

```python
@app.post("/chat")
def chat(request: ChatRequest):
```

表示创建一个 POST `/chat` 接口，并接收符合 `ChatRequest` 格式的数据。

客户端发送的问题可以通过下面的方式取得：

```python
request.message
```

### 6. 组装发送给 Ollama 的数据

```python
payload = {
    "model": MODEL,
    "prompt": request.message,
    "stream": False
}
```

这一步把外部请求转换成 Ollama API 能理解的格式：

- `model`：指定使用哪个模型。
- `prompt`：传给模型的问题。
- `stream`：是否流式返回。
- `False`：等待模型回答完整后一次性返回。

这就是 FastAPI 的封装作用之一：把业务请求整理成 Ollama 需要的标准格式。

### 7. 调用 Ollama

```python
response = requests.post(
    OLLAMA_URL,
    json=payload,
    timeout=120
)
```

这段代码向 `http://localhost:11434/api/generate` 发送 POST 请求。

- `json=payload`：把 Python 字典转换成 JSON 请求体。
- `timeout=120`：最多等待 120 秒，给模型加载和生成回答留出时间。

### 8. 检查 HTTP 请求是否成功

```python
response.raise_for_status()
```

如果返回的是错误状态码，例如 `404` 或 `500`，这一行会抛出异常，程序进入错误处理部分。

### 9. 解析 Ollama 返回结果

```python
data = response.json()
```

把 Ollama 返回的 JSON 数据转换成 Python 字典，方便读取其中的字段。

### 10. 整理返回给客户端的结果

```python
return {
    "reply": data.get("response", ""),
    "model": data.get("model"),
    "done": data.get("done")
}
```

FastAPI 没有把 Ollama 的全部内部字段直接返回，而是整理出业务需要的字段：

- `reply`：模型回答内容。
- `model`：实际使用的模型。
- `done`：模型是否生成完成。

### 11. 捕获调用错误

```python
except requests.exceptions.RequestException as error:
    raise HTTPException(
        status_code=502,
        detail=f"Ollama API 调用失败：{error}"
    )
```

如果 Ollama 没启动、地址错误、端口无法访问或请求超时，程序会返回 `502`，并给出错误原因。

这样客户端可以区分：

- FastAPI 自身是否运行
- FastAPI 调用 Ollama 是否成功

## 七、启动或自动加载服务

如果原来的 FastAPI 窗口仍然运行，并且启动时使用了 `--reload`，保存文件后服务会自动重新加载。

如果原来的服务已经停止，执行：

```powershell
python -m uvicorn main:app --reload
```

### 命令含义

- `python -m uvicorn`：启动 Uvicorn 服务器。
- `main:app`：加载 `main.py` 中的 `app`。
- `--reload`：修改代码后自动重载。

启动成功应看到：

```text
Uvicorn running on http://127.0.0.1:8000
Application startup complete.
```

## 八、使用 Swagger 验证完整链路

打开：

```text
http://127.0.0.1:8000/docs
```

打开 `POST /chat`，点击 `Try it out`，输入：

```json
{
  "message": "请用三句话解释什么是系统部署。"
}
```

点击 `Execute`。

本次实际返回结果包含：

```json
{
  "reply": "模型生成的回答",
  "model": "qwen2.5:3b",
  "done": true
}
```

## 九、验证结果解释

### `200`

表示 FastAPI 成功接收并处理了请求，整个接口调用没有返回 HTTP 错误。

### `reply`

表示 Ollama 模型生成的实际回答，说明问题已经到达模型并且结果成功返回。

### `model: qwen2.5:3b`

表示本次调用确实使用了指定的 `qwen2.5:3b` 模型。

### `done: true`

表示模型已经完成本次生成。因为设置了 `stream: false`，所以客户端收到的是一次性完整结果。

## 十、两个端口的区别

本课同时涉及两个服务和两个端口：

| 服务 | 端口 | 作用 |
| --- | --- | --- |
| FastAPI/Uvicorn | `8000` | 接收浏览器或业务系统的请求 |
| Ollama API | `11434` | 接收 FastAPI 发来的模型调用请求 |

完整方向是：

```text
浏览器或业务系统 → 127.0.0.1:8000 → FastAPI
FastAPI → localhost:11434 → Ollama
```

`8000` 是业务接口端口，`11434` 是模型服务端口。FastAPI 处在两者之间，负责接收、转换、调用和整理结果。

## 十一、本课与第20课的区别

第20课的 `/chat`：

```text
收到消息 → 原样返回消息
```

第21课的 `/chat`：

```text
收到消息 → 组装 Ollama 请求 → 调用模型 → 返回模型回答
```

所以第20课建立的是 API 外壳，第21课让这个外壳具备了真正的模型调用能力。

## 十二、本课暂时不深入的内容

本课先完成全链路验证，以下内容留到后续作业：

- 修改模型名称并观察 `model not found` 错误
- 修改 Ollama 地址并观察连接失败
- 修改 `stream` 参数，理解流式和非流式返回
- 修改 `timeout`，理解等待时间
- 设计更合理的请求体和返回体
- 让错误信息更加清晰
- 增加日志和状态检查
- 尝试不依赖 Swagger，直接使用 PowerShell 调用接口
- 让 FastAPI 服务更接近实际业务使用

## 十三、本课结论

本课已经成功跑通：

```text
Swagger
  → FastAPI /chat
  → requests.post
  → Ollama /api/generate
  → qwen2.5:3b
  → FastAPI 整理 response
  → 返回 reply、model、done
```

当前阶段的重点不是一次记住所有代码，而是先建立整体认识：

```text
FastAPI 是业务接口层
Ollama 是模型服务层
requests 是两者之间的调用工具
8000 是 FastAPI 端口
11434 是 Ollama 端口
```

第21课全链路实操已完成，后续通过作业继续深入理解后面的几层内容。
