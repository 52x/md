---
title: "linux中的sort命令"
date: 2015-02-25 23:44:09 +0800
tags:
comments: true
categories: linux
keywords: linux, sort, 命令
---

sort命令是根据不同的数据类型以行为单位对数据进行排序。

<!--more-->

原文链接：

<http://tianweili.github.io/blog/2015/02/25/linux-sort/>

## 简介

sort命令是根据不同的数据类型以行为单位对数据进行排序。

sort的默认比较规则是从首字符向后，按照ASCII码值进行比较，将结果按照升序输出。

## 用法

sort命令的基本格式如下：

```bash
sort [-bcfMnrtk] [source-file] [-o output-file]
```
sort命令可使用的参数有：

```
-b   忽略每行前面的所有空格字符，从第一个可见字符开始比较。
-c   检查文件是否已经排好序，如果乱序则输出第一个乱序行的相关信息，最后返回1
-C   检查文件是否已经排好序，如果乱序，则不输出内容，仅返回1
-f   排序时忽略大小写字母。
-M   将前面3个字母依照月份的缩写进行排序，比如JAN小于FEB。
-n   依照数值的大小排序。
-o   将排序后的结果存入指定的文件
-r   降序输出
-t   <分隔字符>   指定排序时所用的栏位分隔字符
-u   在输出行中去除重复行
-k   选择以哪个区间进行排序。
```
下面将会对这些参数进行介绍，其中简单的参数就不再赘述了。

## 参数

### -o选项

sort是把排序后结果输出到标准输出，所以需要使用重定向将结果写入指定的文件，比如`sort file > newfile`。

但是重定向的方式在遇到这种需求就无能为力了——把结果输出到原文件中。

如果还是使用重定向的方式，则会把原文件给清空。

而使用`-o`参数则可以完美解决这个问题：

```bash
sort -r test.dat -o test.dat
```

### -t与-k选项

对于某些有固定格式的文件，比如：
```text
	a	12
	b	32
	c	3
```

如果想以第二列数值大小降序输出，则需要使用-t和-k参数了。其中-k指定分隔符，-k指定待排序的列。

```bash
sort -nr -t\t -k2 test.bat -o test.bat
```


作者：[李天炜](http://tianweili.github.com/)

原文链接：<http://tianweili.github.io/blog/2015/02/25/linux-sort/>

转载请注明作者及出处，谢谢。