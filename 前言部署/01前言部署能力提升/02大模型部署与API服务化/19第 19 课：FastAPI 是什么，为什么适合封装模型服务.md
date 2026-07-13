## 课程定位
阶段二：核心技能
模块：大模型部署深化 / API 服务化基础
上一课：API 服务化是什么
本课：认识 FastAPI 的角色
## 目标
1. FastAPI 是什么
2. FastAPI 和 Ollama 的关系
3. 什么是接口路径、请求方法、请求体、返回体
4. 为什么 FastAPI 适合封装模型服务
## 一，FastAPI 是什么
**FastAPI 是一个 Python Web API 框架。**
先不用被“框架”这个词吓到，你可以这样理解：
FastAPI 是用 Python 快速搭建接口服务的工具

它可以帮我们创建这样的接口：
POST /chat
GET /health
POST /summarize
POST /classify
别人访问这些接口，就可以使用我们封装好的能力。
比如：POST /chat
向聊天接口发送问题，拿到模型回答。
## 二、FastAPI 不是模型
FastAPI 不负责生成回答。
Ollama 和大模型负责生成回答。
**FastAPI 负责的是：**
接收请求
检查参数
调用 Ollama
整理结果
返回结果
所以它像一个“服务中转层”。
**链路是：**
业务系统
  -> FastAPI /chat
  -> Ollama /api/generate
  -> 本地模型
  -> Ollama 返回 response
  -> FastAPI 返回 answer
## 三、为什么要有 FastAPI 这一层
因为真实业务系统不想直接面对 Ollama 的底层参数。

Ollama 原始请求可能是：
{
  "model": "qwen2.5:3b",
  "prompt": "请解释系统部署",
  "stream": false
}
但业务系统只想传：
{
  "question": "请解释系统部署"
}
然后得到答案
{
  "answer": "系统部署是..."
}

FastAPI 就负责中间这层转换。他可以把question
转换成 Ollama 需要的：model + prompt + stream
再把 Ollama 返回的：整理成业务系统更好理解的answer

## 四、接口的 4 个基本概念
接口路径
请求方法
请求体
返回体
### 1. 接口路径
接口路径就是访问地址后面的功能入口，例如
/chat  表示聊天接口。
/health  表示健康检查接口。

### 2. 请求方法
请求方法表示你要做什么类型的操作，常见有：
GET：获取信息
POST：提交数据

例如 GET /health  表示查看服务是否正常
POST /chat  表示提交一个问题，让模型回答
### 3. 请求体
请求体就是你发给接口的数据，例如
{
  "question": "什么是系统部署？"
}
### 4. 返回体
返回体就是接口返回给你的数据，例如
{
  "answer": "系统部署是把系统配置到目标环境中..."
}

## 五、为什么 FastAPI 适合封装模型服务
主要是因为以下优点：
1. 写接口比较简单
2. 适合 Python 生态
3. 可以自动生成接口文档
4. 容易接入 Ollama、数据库、日志、鉴权
5. 适合做轻量级 AI 服务封装
## 六、部署岗位怎么理解 FastAPI
把 AI 模型能力封装成业务接口的工具。
需要做到：
1. 服务能启动
2. 接口能访问
3. 能调用 Ollama
4. 能返回统一格式
5. 出问题能排查

FastAPI 是 Python 的 Web API 框架。
FastAPI 不是模型。
FastAPI 负责接收请求和返回结果。
Ollama 负责调用本地模型生成回答。
FastAPI 可以把底层模型能力封装成业务接口。
