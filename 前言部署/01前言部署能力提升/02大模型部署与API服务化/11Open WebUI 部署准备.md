## 1. 当前课程定位

本课程属于阶段二：核心技能。

阶段一已经完成了 Ollama 本地模型运行、PowerShell 基础、部署流程和环境核验。本课开始进入真正的 AI 工具部署实践：准备部署 `Ollama + Open WebUI` 本地大模型服务。

本课不是马上追求复杂功能，而是先理解 Open WebUI 在整个部署链路中的位置，并完成部署前检查。

## 2. 本课学习目标

学完本课后，需要能够说清楚：

1. Open WebUI 是什么。
2. Open WebUI 和 Ollama 的关系。
3. 为什么部署 Open WebUI 常用 Docker。
4. 部署前需要检查哪些环境。
5. 如何判断当前电脑是否具备部署条件。
6. 如果部署失败，优先排查哪些方向。

## 3. Open WebUI 是什么

Open WebUI 可以理解为本地大模型的网页操作界面。

Ollama 负责在后台运行模型，例如 `qwen3:0.6b`。

Open WebUI 负责提供一个类似 ChatGPT 的网页界面，让用户可以在浏览器里选择模型、输入问题、查看回答、保存对话记录。

简单说：

|组件|作用|
|---|---|
|Ollama|模型运行服务|
|大模型|真正生成回答的模型文件|
|Open WebUI|网页聊天界面|
|Docker|把 Open WebUI 打包运行的容器工具|
|浏览器|用户访问入口|

## 4. 为什么要部署 Open WebUI

只用 Ollama 命令行也能和模型对话，但不适合普通用户长期使用。

部署 Open WebUI 后，可以获得这些能力：

1. 用网页访问本地 AI 服务。
2. 不需要每次在命令行里输入模型命令。
3. 可以保留聊天记录。
4. 可以管理多个模型。
5. 更接近真实交付场景。
6. 方便向面试官展示作品。

部署岗位要关注的不是“我能自己用”，而是“我能把它变成别人也能稳定使用的服务”。

## 5. Ollama + Open WebUI 的基本架构

```mermaid
flowchart LR
    A[用户浏览器] --> B[Open WebUI 网页服务]
    B --> C[Ollama 本地模型服务]
    C --> D[本地大模型文件]
```

可以这样理解：

1. 用户打开浏览器。
2. 浏览器访问 Open WebUI。
3. Open WebUI 把用户的问题转发给 Ollama。
4. Ollama 调用本地模型生成回答。
5. 回答返回到 Open WebUI 页面。

## 6. 关键端口

|服务|常见端口|说明|
|---|---|---|
|Open WebUI|`3000`|浏览器访问页面，例如 `http://localhost:3000`|
|Open WebUI 容器内部|`8080`|容器内实际运行端口|
|Ollama|`11434`|Ollama 本地 API 服务端口|

端口可以理解为服务的门牌号。

如果端口被占用，服务可能启动失败；如果防火墙拦截，浏览器可能打不开页面。

## 7. 部署前环境核验

### 7.1 检查 Docker 是否安装

在 PowerShell 中执行：

```powershell
docker --version
```

如果能看到 Docker 版本，说明 Docker 命令已经安装。

如果提示无法识别 `docker`，说明 Docker 没有安装，或者环境变量没有配置好。

### 7.2 检查 Docker 服务是否运行

执行：

```powershell
docker ps
```

可能出现三类结果：

|结果|含义|处理方式|
|---|---|---|
|能看到容器列表|Docker 正常运行|可以继续部署|
|提示无法连接 Docker API|Docker Desktop 没启动|打开 Docker Desktop，等待启动完成|
|提示权限或配置文件访问失败|当前用户权限或配置异常|先记录报错，再判断是否需要管理员权限|

### 7.3 检查 Ollama 是否可用

执行：

```powershell
ollama --version
```

如果能看到版本，说明 Ollama 命令可用。

如果提示无法识别 `ollama`，需要确认：

1. Ollama 是否已经安装。
2. 安装后是否重新打开过 PowerShell。
3. Ollama 是否加入系统环境变量。
4. 是否在另一台电脑上做过上一课实操，而当前电脑还没有安装。

### 7.4 检查模型是否存在

执行：

```powershell
ollama list
```

如果列表里有 `qwen3:0.6b`，说明之前下载的模型还在。

如果没有，可以后续重新执行：

```powershell
ollama run qwen3:0.6b
```

## 8. 当前电脑检查记录

本次检查结果：

|检查项|结果|判断|
|---|---|---|
|Docker 容器状态|`open-webui` 显示 `Up 6 minutes (Healthy)`|Open WebUI 容器正在正常运行|
|Open WebUI 端口|`0.0.0.0:3000->8080/tcp`|浏览器可以访问 `http://localhost:3000`|
|Ollama 命令|`ollama version is 0.31.1`|Ollama 已安装并可正常识别|
|其他服务|Dify、Nginx、Postgres、Redis、Weaviate 等容器也在运行|当前 Docker 环境已有多个服务，需要注意端口占用|

结论：

当前部署前检查通过，可以进入下一步：访问 Open WebUI 页面，并验证它能否连接 Ollama 模型服务。

下一步访问地址：

```text
http://localhost:3000
```

## 9. 官方推荐的 Docker 部署命令

Open WebUI 官方推荐 Docker 方式部署。

基础命令如下：

```powershell
docker pull ghcr.io/open-webui/open-webui:main
docker run -d -p 3000:8080 -v open-webui:/app/backend/data --name open-webui ghcr.io/open-webui/open-webui:main
```

命令含义：

|参数|作用|
|---|---|
|`docker pull`|下载 Open WebUI 镜像|
|`docker run -d`|后台运行容器|
|`-p 3000:8080`|把电脑的 3000 端口映射到容器的 8080 端口|
|`-v open-webui:/app/backend/data`|保存 Open WebUI 数据，避免重启后丢失|
|`--name open-webui`|给容器命名|
|`ghcr.io/open-webui/open-webui:main`|使用官方主版本镜像|

部署成功后，在浏览器访问：

```text
http://localhost:3000
```

## 10. 本课先不急着执行部署命令的原因

部署不是复制命令。

真正的部署顺序应该是：

1. 先确认环境。
2. 再执行安装。
3. 再验证服务。
4. 再记录问题。
5. 最后形成 SOP。

当前 Docker 服务和 Ollama 命令还没有全部确认通过，所以本课重点是部署准备。

## 11. 本课作业

请在 PowerShell 中依次执行下面三个命令，并把结果记录下来：

```powershell
docker --version
docker ps
ollama --version
```

记录格式：

|命令|结果|我的判断|
|---|---|---|
|`docker --version`|||
|`docker ps`|||
|`ollama --version`|||

然后用自己的话回答：

1. Open WebUI 和 Ollama 分别负责什么？
2. 为什么 Open WebUI 部署前要先检查 Docker？
3. 如果 `docker ps` 报错，第一步应该做什么？
4. 如果 `ollama --version` 无法识别，可能有哪些原因？

## 12. 面试表达

```text
我理解的 Ollama + Open WebUI 部署，是把本地大模型从命令行使用升级成网页服务。Ollama 负责运行模型，Open WebUI 负责提供网页交互界面，Docker 用来快速部署和管理 Open WebUI 服务。部署前我会先检查 Docker、Ollama、端口、网络和权限，确认环境满足后再执行部署命令，并通过浏览器访问和模型问答来验证服务是否可用。
```

## 13. 本课总结

本课完成了阶段二第一步：Open WebUI 部署准备。

需要记住：

1. Open WebUI 是网页界面，不是模型本身。
2. Ollama 是模型运行服务。
3. Docker 是部署 Open WebUI 的常用方式。
4. `3000` 是浏览器访问 Open WebUI 的常见端口。
5. `11434` 是 Ollama API 的常见端口。
6. 部署前必须先做环境核验。
7. 当前重点不是马上跑通，而是按部署流程确认条件。
