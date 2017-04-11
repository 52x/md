---
title: "解决Windows下使用go get时报fatal: Unable to find remote helper for 'https'"
tags: [go]
date: 2017-03-01 14:13:01
updated: 2017-03-01 14:13:01
categories: [Go]
keywords: [go,git]
description:
---

尝试go语言，使用go get命令时候提示`fatal: Unable to find remote helper for 'https'`

搜索发现，都提示重新安装git软件，尝试无果，多方查找比对原因后，发现了解决办法如下

1. 在windows的path配置里面，将git的安装目录下移至最下
2. 新添加一个path在最下，`C:\Program Files\Git\mingw64\libexec\git-core` （根据具体安装目录修改）
3. 重启一个cmd 或者重启gogland，重新执行命令，解决。

