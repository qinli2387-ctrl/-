## 一、学习目标
命令行操作，并能够编写基础 Shell 脚本，用于检查目录、文件、进程、端口和网络状态
## 二、命令行的作用
命令行主要用于：

- 检查运行环境
    
- 管理文件和目录
    
- 启动和停止服务
    
- 查看进程和端口
    
- 查看日志
    
- 排查部署问题
    
- 编写自动化脚本
## 三、常用命令对照
|作用|PowerShell|Linux Shell|
|---|---|---|
|查看当前目录|`pwd`|`pwd`|
|查看文件|`ls`|`ls`|
|切换目录|`cd 路径`|`cd 路径`|
|创建文件夹|`mkdir 名称`|`mkdir 名称`|
|查看文件|`Get-Content 文件名`|`cat 文件名`|
|查看进程|`Get-Process`|`ps aux`|
|查看端口|`netstat -ano`|`ss -lntp`|
|测试网络|`ping 地址`|`ping 地址`|
## 四、基础环境核验脚本
在 PowerShell 中创建 `check-env.ps1`：
```
Write-Host "=== Basic Environment Check ==="

Write-Host "`n[1] Current directory"
Get-Location

Write-Host "`n[2] Files and folders"
Get-ChildItem

Write-Host "`n[3] Running processes"
Get-Process | Select-Object -First 5 Name, Id

Write-Host "`n[4] Listening ports"
Get-NetTCPConnection -State Listen |
    Select-Object -First 10 LocalAddress, LocalPort, State

Write-Host "`n[5] Network test"
Test-Connection 127.0.0.1 -Count 1

Write-Host "`nCheck completed."
```
运行脚本：powershell -ExecutionPolicy Bypass -File .\check-env.ps1
## 五、脚本中每部分的作用
- `Write-Host`：输出提示信息。
    
- `Get-Location`：查看当前目录。
    
- `Get-ChildItem`：查看文件和文件夹。
    
- `Get-Process`：查看正在运行的进程。
    
- `Get-NetTCPConnection`：查看正在监听的端口。
    
- `Test-Connection`：测试网络连通性。
    
- `-ExecutionPolicy Bypass`：本次运行临时绕过脚本执行限制。
