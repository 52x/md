---
title: git push的一些总结
date: 2016-08-07 00:28:05
tags: [git pull,用法]
categories: java
---

## git pull
拉取并合并远程代码
git pull <远程主机名> <远程分支名>:<本地分支名>

### 1.拉取
git pull origin fast:master把远程的  
next 分支 拉到 本 master。

### 2.拉取到当前分支
git pull origin fast  
省略定法，表示拉取并合并自当前分支

### 3.等价上面的操作
git fetch origin  
git merge origin/fast

### 4.手动建立跟踪
跟踪不是只能跟踪 master，可以指定本地和远程不同的分支。  
意义在于可以使用简化命令 git push/pull，而不需要显示指定的版本库。  
git branch --set-upstream master origin/fast  
git branch --set-upstream develop origin/develop
