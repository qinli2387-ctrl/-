## 课程目标

第22课在第21课完整调用 Ollama 的基础上，增加请求参数验证和异常处理。

本课解决两个问题：

1. 用户提交的数据格式不正确时，FastAPI 能否提前拒绝？
2. Ollama 连接失败或响应超时时，客户端能否收到清晰的错误？

## 一、课程位置

当前仍属于学习计划中的“第5-6周：API 服务化”模块。

第21课主要完成正常请求链路：

~~~
请求 → FastAPI → Ollama → 模型回答
~~~

第22课增加异常链路：

~~~
请求格式错误 → FastAPI 返回 422
Ollama 连接失败 → FastAPI 返回 502
Ollama 响应超时 → FastAPI 返回 504
~~~

## 二、进入项目目录

~~~powershell
cd "$env:USERPROFILE\Desktop\fastapi-demo"
~~~

### 命令含义

- cd 表示进入目录。
- $env:USERPROFILE 表示当前 Windows 用户目录。
- Desktop\fastapi-demo 表示桌面上的 FastAPI 项目目录。

### 这一步的作用

确保后续修改的是正确的 main.py 文件，避免在错误目录中创建或修改代码。

## 三、修改 main.py

~~~powershell
notepad main.py
~~~

将文件内容替换为：

~~~python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field
import requests

app = FastAPI()

OLLAMA_URL = "http://localhost:11434/api/generate"
MODEL = "qwen2.5:3b"


class ChatRequest(BaseModel):
    message: str = Field(
        ...,
        min_length=1,
        max_length=2000,
        description="用户发送给模型的问题"
    )


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

        if not data.get("response"):
            raise HTTPException(
                status_code=502,
                detail="Ollama 返回内容为空"
            )

        return {
            "reply": data["response"],
            "model": data.get("model"),
            "done": data.get("done")
        }

    except requests.exceptions.Timeout:
        raise HTTPException(
            status_code=504,
            detail="Ollama 响应超时"
        )

    except requests.exceptions.ConnectionError:
        raise HTTPException(
            status_code=502,
            detail="无法连接 Ollama，请检查 Ollama 是否运行以及 11434 端口是否正常"
        )

    except requests.exceptions.RequestException as error:
        raise HTTPException(
            status_code=502,
            detail=f"Ollama API 调用失败：{error}"
        )
~~~

保存并关闭记事本。

## 四、代码含义

### 1. 使用 Field 验证请求参数

~~~python
message: str = Field(..., min_length=1, max_length=2000)
~~~

- message：请求中的问题字段。
- str：字段必须是字符串。
- ...：这个字段是必填字段。
- min_length=1：至少包含一个字符，不能提交空内容。
- max_length=2000：限制最大长度，避免提交过大的请求。

FastAPI 会在请求进入 chat 函数之前完成验证。

### 2. 处理 Ollama 返回为空

~~~python
if not data.get("response"):
    raise HTTPException(status_code=502, detail="Ollama 返回内容为空")
~~~

即使 Ollama 返回了 HTTP 成功状态，也要继续检查返回内容是否真的有回答。没有回答时，向客户端返回 502。

### 3. 设置请求超时时间

~~~python
timeout=120
~~~

最多等待 120 秒。模型第一次加载可能较慢，但不能无限等待，否则接口会一直占用资源。

### 4. 检查 HTTP 状态

~~~python
response.raise_for_status()
~~~

如果 Ollama 返回错误状态码，这一行会抛出异常，并进入异常处理逻辑。

### 5. 处理不同类型的错误

~~~python
except requests.exceptions.Timeout:
~~~

表示 Ollama 在规定时间内没有响应，返回 504。

~~~python
except requests.exceptions.ConnectionError:
~~~

表示无法连接 Ollama，常见原因是 Ollama 没有运行、地址错误或 11434 端口无法访问，返回 502。

~~~python
except requests.exceptions.RequestException:
~~~

捕获其他 HTTP 请求异常，返回 502。

## 五、启动服务

如果原来的 FastAPI 服务仍在运行并使用了 --reload，保存文件后会自动重新加载。

如果服务已经停止，执行：

~~~powershell
python -m uvicorn main:app --reload
~~~

启动成功应看到：

~~~text
Uvicorn running on http://127.0.0.1:8000
Application startup complete.
~~~

## 六、验证正常请求

打开：

~~~text
http://127.0.0.1:8000/docs
~~~

在 POST /chat 中输入：

~~~json
{
  "message": "请用三句话解释什么是系统部署。"
}
~~~

预期结果：

- 状态码为 200
- reply 有模型回答
- model 为 qwen2.5:3b
- done 为 true

## 七、验证缺少字段

输入：

~~~json
{}
~~~

预期结果：

~~~text
422 Unprocessable Entity
~~~

原因是请求缺少必填的 message 字段。这个错误由 FastAPI 直接返回，问题不会继续传到 Ollama。

## 八、验证空内容

输入：

~~~json
{
  "message": ""
}
~~~

预期结果仍然是：

~~~text
422 Unprocessable Entity
~~~

原因是 min_length=1 要求 message 至少包含一个字符。

## 九、状态码含义

| 状态码 | 含义 | 常见原因 |
| --- | --- | --- |
| 200 | 请求成功 | 模型回答正常返回 |
| 422 | 请求格式不符合要求 | 缺少字段、字段为空或类型错误 |
| 502 | 上游服务调用失败 | Ollama 无法连接或返回异常 |
| 504 | 上游服务响应超时 | 模型生成时间超过限制 |

## 十、本课结论

第22课让 API 从“只处理正常情况”升级为“能够识别和说明异常情况”。

本课已经验证：

- 正常请求返回 200
- 缺少 message 返回 422
- 空 message 返回 422
- Ollama 连接错误和超时有专门的处理代码

Ollama 故障的模拟测试留作后续作业，后面再深入理解异常链路。

第22课已完成。
