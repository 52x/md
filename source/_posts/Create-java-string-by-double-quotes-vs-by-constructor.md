---
title: 使用双引号""创建Java字符串还是使用String构造函数？
date: 2016-08-15 21:26:02
tags: Java
toc: true
categories: Java平台
---

在Java中，一个字符串可以使用下面这两种方式进行创建：

```
String x = "abc";
String y = new String("abc");
```

## 这两种创建字符串的方式有什么不同呢？

### 双引号 VS 构造函数

这个问题可以用下面这两个简单的代码实例来回答。

- 例子1：

```
String a = "abcd";
String b = "abcd";
System.out.println(a == b);  // True
System.out.println(a.equals(b)); // True
```
a==b是true，因为a和b都引用同一块内存地址。

当相同字符内容的字符串多次创建时，编译器只为其分配一块内存，这叫做“字符串驻留机制”。Java中所有的编译期间常量都将自动“驻留”。

<!--more-->

- 例子2

```
String c = new String("abcd");
String d = new String("abcd");
System.out.println(c == d);  // False
System.out.println(c.equals(d)); // True
```
c==d是false，因为c和d在堆中引用两个不同的对象。不同的对象总是会有不同的内存地址引用。

下面这插图将解释上面两种情况：
![双引号VS构造函数](http://img.blog.csdn.net/20160815210050342)

### 运行期字符串驻留技术

感谢LukasEder（他的讲解如下）：

尽管两个字符串使用字符串构造函数进行的创建，字符串驻留仍然可以在运行期间进行。

```
String c = new String("abcd").intern();
String d = new String("abcd").intern();
System.out.println(c == d);  // Now true
System.out.println(c.equals(d)); // True
```

### 两种方法的使用场景

因为字符串“abcd”已经是String类型，使用字符串构造函数将会创建额外没有必要的对象。因此，如果你只想要创建一个字符串对象时，双引号“”推荐使用。如果你的确想要在堆中创建一个新的对象，字符串构造函数将可以使用。

原文链接：[Create Java String Using ” ” or Constructor? ](http://www.programcreek.com/2014/03/create-java-string-by-double-quotes-vs-by-constructor/)
翻译：crane-yuan
[ 转载请保留原文出处、译者和译文链接。]
