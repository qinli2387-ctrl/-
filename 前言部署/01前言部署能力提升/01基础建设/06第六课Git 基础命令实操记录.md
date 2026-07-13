# Git 基础命令实操记录

## 1. 当前课程定位

本课程属于阶段一：基础建设。

前一课已经理解 Git/GitHub 的基础概念，本课开始进行本地 Git 命令实操，目标是完成一次完整的本地版本提交。

## 2. 实操目标

本次实操目标：

1. 检查 Git 是否安装。
2. 初始化一个本地 Git 仓库。
3. 查看文件状态。
4. 将文件加入暂存区。
5. 完成第一次提交。
6. 查看提交历史。
7. 记录实操中出现的问题和解决方式。

## 3. 实操目录

本次实操目录：

```
C:\Users\123\Documents\New project\前沿部署实操环境\01-git-basic
```

该目录用于练习 Git 基础命令，并已包含：

```
README.md
first-note.md
```

## 4. 实操过程

### 4.1 检查 Git 版本

命令：

```
git --version
```

结果：

```
git version 2.55.0.windows.2
```

说明：

Git 已经安装成功，可以继续进行后续操作。

### 4.2 查看初始状态

命令：

```
git status
```

结果：

```
fatal: not a git repository (or any of the parent directories): .git
```

说明：

当前文件夹还不是 Git 仓库，需要先执行初始化。

### 4.3 初始化 Git 仓库

命令：

```
git init
```

结果：

```
Initialized empty Git repository
```

说明：

当前目录已经被初始化为 Git 仓库。

### 4.4 查看未跟踪文件

命令：

```
git status
```

结果显示：

```
Untracked files:
  README.md
  first-note.md
```

说明：

文件已经存在，但还没有被 Git 纳入版本管理。

### 4.5 添加文件到暂存区

命令：

```
git add .
```

说明：

该命令会把当前目录下新增或修改的文件添加到暂存区，准备提交。

Windows 环境中出现换行符提醒：

```
LF will be replaced by CRLF
```

该提醒和 Windows 换行符有关，当前阶段不影响本次实操。

### 4.6 查看暂存区状态

命令：

```
git status
```

结果显示：

```
Changes to be committed:
  new file: README.md
  new file: first-note.md
```

说明：

文件已经进入暂存区，可以提交版本。

### 4.7 第一次提交失败

命令：

```
git commit -m "first git practice"
```

报错：

```
Author identity unknown
Please tell me who you are.
```

原因：

Git 不知道当前提交人的用户名和邮箱，因此拒绝提交。

解决方式：

```
git config user.name "Frontier Deploy Student"
git config user.email "student@example.com"
```

说明：

本次只配置当前练习仓库的用户名和邮箱，不影响其他项目。

### 4.8 第一次提交成功

命令：

```
git commit -m "first git practice"
```

结果：

```
[master (root-commit) 1add605] first git practice
2 files changed, 105 insertions(+)
create mode 100644 README.md
create mode 100644 first-note.md
```

说明：

第一次版本提交成功。

### 4.9 查看提交历史

命令：

```
git log --oneline
```

结果：

```
1add605 first git practice
```

说明：

当前仓库已有一条提交记录。

## 5. 本次实操涉及命令

|命令|作用|
|---|---|
|`git --version`|查看 Git 是否安装和当前版本|
|`git status`|查看当前仓库文件状态|
|`git init`|初始化 Git 仓库|
|`git add .`|将文件添加到暂存区|
|`git commit -m "说明"`|提交一个版本|
|`git log --oneline`|简洁查看提交历史|
|`git config user.name "名称"`|配置提交用户名|
|`git config user.email "邮箱"`|配置提交邮箱|

## 6. 本次问题记录

|问题|原因|解决方式|
|---|---|---|
|`not a git repository`|当前目录还没有初始化 Git 仓库|执行 `git init`|
|`Author identity unknown`|未配置 Git 用户名和邮箱|执行 `git config user.name` 和 `git config user.email`|
|`LF will be replaced by CRLF`|Windows 换行符提醒|当前阶段可先忽略，后续学习换行符配置|

## 7. 本课总结

本次完成了 Git 本地版本管理的第一条完整流程：

```
检查 Git -> 初始化仓库 -> 查看状态 -> 添加文件 -> 提交版本 -> 查看历史
```

当前需要掌握的重点不是背命令，而是理解 Git 的状态变化：

```
未跟踪文件 -> 暂存区 -> 提交记录
```

## 8. 面试表达

```
我做过 Git 本地基础实操，理解从初始化仓库、查看状态、添加文件、提交版本到查
```