---
title: "linux中的awk命令"
date: 2015-02-24 10:33:41 +0800
tags:
comments: true
categories: linux
keywords: linux, awk, 命令
---

本文主要介绍了Linux中的awk命令的一些知识以及如何使用awk编程。不同于grep的查找、sed的编辑等命令，awk命令在文本处理和生成报告等地方是经常用到的一个强大命令。

<!--more-->

原文链接：<http://tianweili.github.io/blog/2015/02/24/linux-awk/>

## 简介

awk命令主要用于文本分析。它的处理方式是读入文本，将每行记录以一定的分隔符（默认为空格）分割成不同的域，然后对不同的域进行各种处理与输出。

## 命令格式

awk命令的一个基本格式如下：

```bash
awk '{pattern + action}' {filenames}
```

无论awk命令简单还是复杂，基本的格式如上所示。其中引号为必须，引号内代表一个awk程序。大括号非必须，括起来用于根据特定的模式对一系列指令进行分组。pattern是在数据中查找内容，支持正则匹配。action对查找出来的记录执行相应的处理，比如打印和输出等。

## awk三种调用方式

### 命令行方式

```bash
awk [-F 'field-separator'] 'commands' input-file(s)
```

其中的`-F`指令是可选的，后面跟着指定的域分隔符，比如tab键等（默认是空格）。后面的`commands`是真正的awk命令。`input-file(s)`代表输入的一个或多个文件

命令行调用方式是最经常使用的一种方式，也是本文所讲的重点。

### shell脚本方式

把平时所写的shell脚本的首行`#!/bin/sh`换成`#!/bin/awk`。把所有的awk命令插入脚本中，通过调用脚本来执行awk命令。

### 插入文件调用

把所有的awk命令插入单独的文件中，然后通过以下命令调用awk：

```bash
awk -f awk-script-file input-file(s)
```

其中`-f`指定了要调用的包含awk命令的文件。

## awk应用示例

### 打印指定字段

打印当前目录下所有的文件名和文件大小列表，以tab键分割：

```bash
ls -lh | awk '{print $5"\t"$9}'
```

$0变量是指当前一行记录，$1是指第一个域数据，$2指第二个域数据……以此类推。

### print与printf

awk提供了print与printf两种打印输出的函数。

print的参数可以是变量、数值和字符串。参数用逗号分割，字符串必须用双引号引用。

printf与C语言中的printf函数类似，可以用来格式化字符串。

```bash
awk -F ':' '{printf("filename:%10s,linenumber:%s,columns:%s,linecontent:%s\n",FILENAME,NR,NF,$0)}' /etc/passwd
```

### 根据指定分隔符切割域

```bash
ll | awk -F '\t' 'print $9'
```

### BEGIN...END

```bash
ls -lh | awk 'BEGIN {print "size\tfilename"}  {print $5"\t"$9} END {print "---end---"}'
```

`BEGIN...END`语句的执行流程是，awk命令读入数据，然后从BEGIN语句开始，依次读取每一行记录，并打印相应的域，当所有记录都处理后再执行END语句后的程序。也就是说`BEGIN...END`语句块中的内容在读取数据过程中会反复执行，直到数据读取完成。

### pattern正则匹配

下面的例子表示打印当前目录下，所有以.bat后缀结尾的文件名列表：

```bash
ls -l | awk -F: '/\.dat$/{print $9}'
```

## awk内置变量

awk有许多内置变量用来设置环境变量信息，这些变量都可以被改变。常用的内置变量和作用如下所示：

```bash
ARGC               命令行参数个数
ARGV               命令行参数排列
ENVIRON            支持队列中系统环境变量的使用
FILENAME           awk浏览的文件名
FNR                浏览文件的记录数
FS                 设置输入域分隔符，等价于命令行-F选项
NF                 浏览记录的域的个数
NR                 已读的记录数
OFS                输出域分隔符
ORS                输出记录分隔符
RS                 指定用来切片的分隔符
```
awk中的内置变量都是很有用处的，可以直接使用。比如上面讲过的指定分隔符操作就可以用FS变量来代替：

```bash
ll | awk '{FS="\t";} {print $9}'
```
下面会有很多实用awk内置变量的例子。

## awk编程

### 定义变量和运算

awk可以自定义变量，并参与运算。

比如统计当前目录下列出的文件总大小，以M为单位显示出来：

```bash
ls -l | awk 'BEGIN {size=0;} {size+=$5;} END {print "size is ", size}'
```
注意此统计没有把文件夹下的所有文件算在内。

自定义的变量有时候可以不用作初始化操作，不过正规起见，还是建议作初始化操作为好。

### 条件语句

awk中的条件语句跟C语言类似，声明方式如下：

```bash
if(expression){
	statement1;
	statement2;
}

if(expression){
	statement1;
} else {
	statement2;
}

if(expression1){
	statement1;
} else if (expression2) {
	statement2;
} else {
	statement3;
}
```

看下面例子，将第三列为12，第六列为0的行打印输出：

```bash
awk 'BEGIN {FS="\t"}{if($3==12 && $6==0) print $0} END' incoming_daily_20150223.dat
```

### 循环语句

awk中的循环语句同样与C语言中的类似，支持while、do/while、for、break、continue关键字。

看下面的例子，输出每行的行号和第一列的数据：

```bash
awk 'BEGIN {FS="\t";} {data[NR] = $1} END {for(i=1; i<=NR; i++) print i"\t"data[i]}' incoming_daily_20150223.dat
```


### 数组

看下面例子，统计第六列每一个值出现的次数：

```bash
awk 'BEGIN {FS="\t"}{count[$6]++} END {for(x in count) print x,count[x]}' incoming_daily_20150223.dat
```
---

作者：[李天炜](http://tianweili.github.io/)

原文链接：<http://tianweili.github.io/blog/2015/02/24/linux-awk/>

转载请注明作者及出处，谢谢。