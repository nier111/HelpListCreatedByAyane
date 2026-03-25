# git 使用教程

## Overview

本文件主要记录 git 的一些常用指令，帮助新手快速上手。

## 常用指令

### 1.1 基础配置

```bash
sudo pacman -S git less
# 设置全局用户名和邮箱
git config --global user.name "YourName"
git config --global user.email "your@email.com"

# 查看配置
git config --list
```

### 1.2 仓库初始化/克隆

```bash
# 在当前目录初始化 Git 仓库
git init

# 克隆远程仓库
git clone https://github.com/user/repo.git

# 使用 SSH 克隆
git clone git@github.com:user/repo.git

```

### 1.3 添加文件并上传

```bash
# 查看当前状态
git status

# 添加单个文件
git add file.md

# 添加所有修改
git add .

# 提交
git commit -m "描述本次修改"

# 推送到远程主分支
git push

# 第一次推送新仓库：
git push -u origin main
```

### 1.4 指定远程仓库地址

```bash
# 查看当前远程地址
git remote -v

# 添加远程仓库
git remote add origin https://github.com/user/repo.git

# 修改远程仓库地址
git remote set-url origin git@github.com:user/repo.git

# 删除远程仓库
git remote remove origin
```

### 1.5 分支管理

```bash
# 查看分支
git branch

# 创建分支
git branch dev

# 切换分支
git checkout dev

# 创建并切换
git checkout -b feature-x

# 合并分支
git merge dev

# 删除分支
git branch -d dev

```

### 1.6 拉取远程更新

```bash
# 拉取并合并
git pull

# 只拉取不合并
git fetch

```

### 1.7 版本回退

- 查看历史
```bash
git log 
git log --oneline
```

- 回退到某个提交

```bash
git reset --hard 提交ID
```

- 回退但保留修改
```bash
git reset --soft 提交ID
```

- 撤销最后一次提交
```bash
git reset --soft HEAD~1
```

- 回复某个文件到指定版本
```bash
git checkout 提交ID -- 文件名
```

- 使用 revert 
```bash
git revert 提交ID
```

### 1.8 强制推送

```bash
git push --force
```

### 1.9 忽略文件

```bash
# 创建 .gitignore 文件
```

### 1.10 查看差异

```bash
git diff
git diff --staged
```

### 1.11 标签（发布版本）

```bash
# 创建标签
git tag v1.0
# 推送标签
git push origin v1.0
# 推送所有标签
git push --tags

```

### 1.12 查看文件归属

```bash
git blame 文件名

```

### 1.13 子目录单独上传

```bash
git add docs/
git commit -m "update docs"
git push
```

### 1.14 查看当前远程分支状态

```bash
git branch -vv
```

### 1.15 删除远程分支

```bash
git push origin --delete branch-name
```



