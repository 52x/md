---
title: 在Debian服务器上安装Golang1.8
tags: [go,Debian]
date: 2017-03-13 11:36:59
updated: 2017-03-13 11:36:59
categories: [Go,Linux]
keywords:
description:
---

最近Google Cloud 的服务器提供1年免费试用，来占个小便宜，在服务器上，准备跑个go语言的程序，现在没有环境，特搭建之。
服务器是debian的环境。 记录操作办法如下。
1. 下载`wget https://storage.googleapis.com/golang/go1.8.linux-amd64.tar.gz`

2. 解压`tar xf go1.8.linux-amd64.tar.gz`

3. 删除安装包 `rm go1.8.linux-amd64.tar.gz`

4. 设置路径 在.bashrc中添加

   ```
   #golang
   export GOROOT=$HOME/go
   export GOPATH=$HOME/gopath
   export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
   ```

5. 重新登录 或者执行 `. ~/.bashrc`

   ​

