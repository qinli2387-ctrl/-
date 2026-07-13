## 课程定位阶段二：核心技能
模块：大模型部署深化 / API 服务化基础
上一课：看懂 API 返回结果字段
本课：学会 API 调用失败时怎么查
## 目标
1. 是 Ollama 服务问题
2. 是模型问题
3. 是请求格式问题
4. 是中文编码问题
5. 是性能资源问题
### 一、API 排查总原则
API 出问题，不要先改模型，先看请求有没有到服务。
排查顺序：
1. 服务通不通
2. 模型在不在
3. 请求对不对
4. 返回说什么
5. 资源够不够
### 二、问题 1：连接不上 11434
无法连接
Connection refused
No connection could be made
优先怀疑
Ollama 服务没有运行
11434 端口没有监听
地址写错

检查顺序：ollama --version
ollama list
可以用浏览器打开：http://localhost:11434
如果 Ollama 正常，通常会看到类似：Ollama is running
（Ollama is running）

## 三、问题 2：模型不存在
现象可能是：
model not found
pull model manifest
模型列表里没有
优先怀疑：model 名称写错，本地没有下载这个模型

命令行检查：ollama list（看有没有请求的模型）
ollama run qwen2.5:3b（如果没有就需要通过命令行去运行）
判断：模型不存在，不是 API 坏了，是本地没有这个模型或模型名写错。

## 四、问题 3：请求格式错误
出现现象：400 Bad Request
invalid character
unexpected token
json parse error

优先怀疑：JSON 格式写错
引号不对
逗号不对
字段名写错
PowerShell 换行导致内容异常
**正确写法**
$body = @{
  model = "qwen2.5:3b"
  prompt = "请用三句话解释什么是系统部署。"
  stream = $false
} | ConvertTo-Json

Invoke-RestMethod -Uri "http://localhost:11434/api/generate" -Method Post -ContentType "application/json; charset=utf-8" -Body $body
判断：JSON 报错，优先检查请求体格式。

## 五、问题 4：中文乱码或中文没被理解
**出现现象**：模型回答说看不懂
中文变成问号
中文 prompt 没正常传过去
**优先怀疑**：PowerShell 编码
JSON 字符串编码
Content-Type 没指定 utf-8

解决方法：使用 ConvertTo-Json 生成请求体
Content-Type 使用 application/json; charset=utf-8
判断：API 有 response、done=True，但回答不对，可能是编码或 prompt 问题。
API 技术成功：response 有内容，done=True
业务回答正确：回答内容符合你的问题

## 六、问题 5：调用很慢
**出现现象**：很久才返回
第一次特别慢
一直等待
**优先看返回字段**：total_duration
load_duration
eval_duration
eval_count
load_duration 大：模型加载慢
eval_duration 大：生成回答慢
eval_count 大：回答太长
第一次慢、第二次快：多半是模型首次加载

除了看这些字段还需要看本机资源；CPU
内存
显存
同时运行的容器数量
模型大小
判断：慢不一定是失败，可能是模型大或资源不足。

## 七、问题 6：返回为空
**出现现象**：response 为空
done=True
没有明显报错
**可能原因**：prompt 太短或不清楚
模型输出被限制
请求参数有问题
模型异常停止
**处理方法**：换一个明确的问题
换英文短问题测试
换小模型测试
查看 done_reason

先用短暂的命令测试：Say hello in one short sentence.
先用最简单 prompt 验证链路，再测试复杂问题。

## 八、API 排查口诀
连不上，查服务。
找不到，查模型。
格式错，查 JSON。
中文乱，查编码。
速度慢，查资源。
返回空，换问题。

```
response 有内容，done=True，但中文问题回答不对
```
response 有内容、done=True，说明 API 技术调用成功。如果中文问题回答不对，优先检查中文编码和 prompt 是否正确传入，也要检查 Content-Type 是否指定 utf-8。可以先用英文短问题验证链路，再用 ConvertTo-Json 方式重新发送中文问题。

连不上：服务问题
model not found：模型问题
JSON 报错：格式问题
response 有但中文不对：编码或 prompt 问题
很慢：资源或模型大小问题
