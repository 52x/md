title: \"> /dev/null 2>&1\" 什么意思？
date: 2016-06-26 12:19 AM
categories: ""
tags: [linux,]
---
很长一段时间以来，我一直无法理解这个命令，我能大概猜测出 `>/dev/null` 意思是将输出隐藏，但还是不能理解 `2>&1` 的意思。

<!--more-->

![http://harchiko.qiniudn.com/Screen%20Shot%202016-06-26%20at%2012.37.26%20AM.png](http://harchiko.qiniudn.com/Screen%20Shot%202016-06-26%20at%2012.37.26%20AM.png)

举例来说:


```bash
git push -f origin master > /dev/null 2>&1
```

这是一个简单的 `git push`脚本,用于 Travis 自动构建过程中隐藏信息。



`> ` is for redirect 

`/dev/null ` is a black hole where any data sent, will be discarded

`2 ` is the file descriptor for Standard Error

`> ` is for redirect

`& ` is the symbol for file descriptor (without it, the following 1 would be considered a filename)

`1 ` is the file descriptor for Standard Out

Therefore >/dev/null 2>&1 is redirect the output of your program to /dev/null. Include both the Standard Error and Standard Out.


所以， `/dev/null 2>&1` 将所有的输出重定向到 /dev/null 中，标准错误和输出都不会显示。

那么， `> /dev/null` 是什么意思呢就不难猜出吧？

> `> /dev/null`即标准输出不显示，标准错误正常显示。

参考文章: 
1. [http://www.xaprb.com/blog/2006/06/06/what-does-devnull-21-mean/](http://www.xaprb.com/blog/2006/06/06/what-does-devnull-21-mean/)
2. [http://unix.stackexchange.com/questions/163352/what-does-dev-null-21-mean-in-this-article-of-crontab-basics](http://unix.stackexchange.com/questions/163352/what-does-dev-null-21-mean-in-this-article-of-crontab-basics)