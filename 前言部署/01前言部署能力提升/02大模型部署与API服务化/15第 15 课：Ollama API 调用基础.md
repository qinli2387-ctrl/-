# 课程定位阶段二：核心技能
模块：大模型部署深化 / API 服务化基础
当前目标：从“网页里使用模型”升级到“通过接口调用模型”

## 需要达成目标
1. API 是什么
2. Ollama 的 11434 端口是什么
3. 网页调用和 API 调用有什么区别
4. 如何用接口向本地模型提问
5. 如何判断 API 调用是否成功
## 一，API是什么
网页是给人用的。
API 是给程序用的。

Open WebUI 是网页界面，人打开浏览器、输入问题、点击发送。
API 是程序接口，别的程序可以直接把问题发给 Ollama，然后拿到模型回答。

Open WebUI：人和模型对话
Ollama API：程序和模型对话

## 二、11434 端口是什么
Ollama 默认会在本机提供一个服务入口：http://localhost:11434
这个端口就是 Ollama 的 API 服务端口
3000：Open WebUI 网页入口
8080：Open WebUI 容器内部端口
11434：Ollama API 入口

## 三、API 调用的基本结构
一次API调用通常有4个部分
1. 请求地址：发给谁
2. 请求方法：用什么方式发
3. 请求内容：发什么问题
4. 返回结果：模型回答了什么
调用 Ollama 生成回答，常见接口是：POST http://localhost:11434/api/generate
POST：表示提交一段数据
/api/generate：表示让模型生成回答

## 四、PowerShell 实操命令
Invoke-RestMethod -Uri "http://localhost:11434/api/generate" -Method Post -ContentType "application/json" -Body '{
  "model": "qwen2.5:3b",
  "prompt": "请用三句话解释什么是系统部署。",
  "stream": false
}'
model：调用哪个模型
prompt：给模型的问题
stream：是否流式返回
`"stream": false` 的意思是：等模型回答完整后，一次性返回结果

## 五、API 调用结果验证
本次通过 PowerShell 调用 Ollama API 成功。

返回结果中重点字段：

model：qwen2.5:3b
response：模型返回的回答内容
done：True，表示请求完成
total_duration：本次请求总耗时

如果 response 有内容，并且 done=True，说明 API 技术调用成功。
## 六、第一次中文返回异常
第一次直接在 JSON 字符串里写中文 prompt 时，模型返回了英文，并表示没有理解问题。
判断原因：API 本身调用成功，但中文内容可能在 PowerShell 传输时出现编码问题。

核心结论：
API 成功不等于业务回答正确。
技术层成功看 response 和 done。
业务层成功看回答内容是否符合预期。
## 七，解决方式
$body = @{
  model = "qwen2.5:3b"
  prompt = "请用三句话解释什么是系统部署。"
  stream = $false
} | ConvertTo-Json

Invoke-RestMethod -Uri "http://localhost:11434/api/generate" -Method Post -ContentType "application/json; charset=utf-8" -Body $body
使用 PowerShell 哈希表加 ConvertTo-Json 生成请求体，并指定 charset=utf-8 后，中文 prompt 可以正常传递，模型返回了中文回答。

# 总结
本课完成了 Ollama API 的第一次调用。通过 /api/generate 接口，可以不经过 Open WebUI 页面，直接让程序向本地模型发送问题。11434 是 Ollama 的 API 服务端口，/api/generate 用于生成回答。判断 API 是否成功，主要看 response 是否有内容，以及 done 是否为 True。
**面试表达**
我不仅通过 Open WebUI 页面验证了本地大模型可用，也尝试通过 Ollama API 调用模型。实操中使用 PowerShell 向 http://localhost:11434/api/generate 发送 POST 请求，并通过 model、prompt、stream 参数控制模型调用。返回结果中 response 有内容且 done=True，说明 API 调用成功。
