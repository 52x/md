title: golang的centos安装
date: 2016-01-07 15:59:15
tags: [golang, centos]
---


## Linux下安装(基于centos)
## 下载golang
根据自己的系统位数（32位/64位）下载适合自己的版本的golang安装包

下载地址：https://golang.org/dl/

## 安装
```js
tar -zxvf go1.5.1.linux-amd64.tar.gz -C /data/soft/golang
//  /data/soft/golang 为安装位置
```

## 设置GOPATH
```js
vi /etc/profile
```

在最后添加如下代码
```js
export GOROOT=/data/soft/golang/go   //安装位置
export GOPATH=/data/host/go //自定义代码位置
export GOBIN=$GOROOT/bin
export GOPKG=$GOPATH/pkg/linux_amd64
export GOARCH=amd64
export GOOS=linux
export PATH=$PATH:$GOBIN:$GOPKG:$GOPATH/bin

```
然后编译生效
```js
source /etc/profile
```
## 验证安装
````js
go version
````
如果出现
```js
go version go1.5.1 linux/amd64
```
则说明安装成功


