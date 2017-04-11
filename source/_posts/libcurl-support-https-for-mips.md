---
title: libcurl.so支持https协议(mips平台)
date: 2016-11-07 11:26:44
tags:
 - linux
categories:
 - linux
---

### 1. dependencies:

------------

`autoconf`

### 2. mips tools:

------------

`echo $PATH`(查看环境变量，确保环境变量当中包含有`mips-linux-gcc`等工具的路径)

可以用: `export PATH=/xxx/xxx/:$PATH` 添加包含`mips-linux-gcc`等工具的路径

### 3. download curl from git:

------------

```shell
git clone https://github.com/curl/curl.git curl.git
```

### 4. install(安装步骤可看GIT-INFO):

------------

```shell
cd curl.git

./buildconf #查看依赖包是否安装，均安装的话会生成./configure

./configure --host=mips-linux CC=mips-linux-gcc CFLAGS=-I/xxx/xxx/include LDFLAGS=-L/xxx/xxx/lib LIBS="-lssl -lcrypto"
# CFLAGS=-I/xxx/xxx/include 指向包含openssl（里面有关于ssl的头文件）文件夹的绝对路径。
# LDFLAGS=-L/xxx/xxx/lib 指向包含libssl.so和libcrypt.so库的绝对路径。注意：这两个库也得经过mips-linux-gcc交叉编译过才行。
# 添加--prefix参数可以指定安装路径。

make

sudo env PATH=$PATH make install
```

最后，经过`mips-linux-gcc`交叉编译的`libcurl.so`便默认位于`/usr/local/lib`目录里。

