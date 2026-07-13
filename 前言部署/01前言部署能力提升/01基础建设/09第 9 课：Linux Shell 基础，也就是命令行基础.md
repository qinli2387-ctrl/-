## 1.实操目标
本实操用于掌握部署岗位常用的命令行 
- 查看当前所在目录。
- 切换目录。
- 查看文件列表。
- 创建文件夹。
- 创建和查看文本文件。
- 查看磁盘、进程和端口等基础信息。
- 记录命令输出和问题
## 二，为什么要学命令行
部署工作中经常需要用命令行完成
- 检查环境
- 启动服务
- 查看日志
- 查看端口
- 查看进程
- 排查报错
- 管理文件和配置
## 三，3. Windows PowerShell 与 Linux Shell 对照
|目标|PowerShell|Linux/Shell|
|---|---|---|
|查看当前目录|`pwd`|`pwd`|
|查看文件列表|`ls`|`ls`|
|切换目录|`cd 路径`|`cd 路径`|
|创建文件夹|`mkdir 文件夹名`|`mkdir 文件夹名`|
|查看文件内容|`Get-Content 文件名`|`cat 文件名`|
|查看进程|`Get-Process`|`ps aux`|
|查看端口|`netstat -ano`|`ss -lntp`|
|测试网络|`ping 地址`|`ping 地址`|

## 4. 本课实操命令
请在 PowerShell 中进入本目录：
cd "C:\Users\123\Documents\New project\前沿部署实操环境\09-linux-shell-basic"
依次进行下面的操作
pwd
ls
mkdir test-folder
ls
New-Item test-note.txt -ItemType File
Set-Content test-note.txt "这是我的第九课命令行实操记录"
Get-Content test-note.txt
Get-Process | Select-Object -First 5
netstat -ano | Select-Object -First 10
ping 127.0.0.1
pwd：看自己在哪个目录
ls：看目录里有什么
mkdir：创建文件夹
New-Item：创建文件
Set-Content：写入内容
Get-Content：查看文件内容
Get-Process：查看正在运行的进程
netstat -ano：查看端口和连接
ping 127.0.0.1：测试本机网络是否正常

