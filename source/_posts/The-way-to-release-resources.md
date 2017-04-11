---
title: Java中资源关闭的处理方式
date: 2016-08-23 20:28:12
tags: [Java]
toc: true
categories: Java平台
---

本文就关于IO资源的处理问题，提出三种方案。

- close()放在try块中
- close()放在finally块中
- 使用try-with-resource语句

##  close()放在try块中

<!--more-->

```
//close() is in try clause
try {
	PrintWriter out = new PrintWriter(
			new BufferedWriter(
			new FileWriter("out.txt", true)));
	out.println("the text");
	out.close();
} catch (IOException e) {
	e.printStackTrace();
}
```
这种方式容易造成IO资源的泄露，因为对于IO资源来说不管操作的结果如何都必须关闭。

## close()放在finally块中

```
//close() is in finally clause
PrintWriter out = null;
try {
	out = new PrintWriter(
		new BufferedWriter(
		new FileWriter("out.txt", true)));
	out.println("the text");
} catch (IOException e) {
	e.printStackTrace();
} finally {
	if (out != null) {
		out.close();
	}
```

这种方式在`JDK1.7`之前，推荐使用这种方式，但是，这种方式还是有问题，因为，在try块和finally块中可能都会发生Exception。

## 使用[try-with-resource](http://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html)语句

```
//try-with-resource statement
try (PrintWriter out2 = new PrintWriter(
			new BufferedWriter(
			new FileWriter("out.txt", true)))) {
	out2.println("the text");
} catch (IOException e) {
	e.printStackTrace();
}
```

这种方式可能是最好的，Java官方推荐使用这种方式，但是，使用的前提是你的jdk版本在1.7以上。

## 总结

因为不管什么情况下(异常或者非异常)资源都必须关闭，在jdk1.6之前，应该把close()放在finally块中，以确保资源的正确释放。

如果使用jdk1.7以上的版本，推荐使用[try-with-resources](http://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html)语句。

原文链接：[should-close-be-put-in-finally-block-or-not](http://www.programcreek.com/2013/12/should-close-be-put-in-finally-block-or-not/)

翻译：crane-yuan

[ 转载请保留原文出处、译者和译文链接。]


