---
title: Java中Scanner类和BufferReader类之间的区别
date: 2016-08-17 10:30:12
tags: Java
toc: true
categories: Java平台
---

java.util.Scanner类是一个简单的文本扫描类，它可以解析基本数据类型和字符串。它本质上是使用正则表达式去读取不同的数据类型。

Java.io.BufferedReader类为了能够高效的读取字符序列，从字符输入流和字符缓冲区读取文本。

下面是两个类的不同之处：

***当nextLine()被用在nextXXX()之后，用Scanner类有什么问题***

尝试去猜测下面代码的输出内容；

```
// Code using Scanner Class
import java.util.Scanner;
class Differ
{
     public static void main(String args[])
     {
         Scanner scn = new Scanner(System.in);
         System.out.println("Enter an integer");
         int a = scn.nextInt();
         System.out.println("Enter a String");
         String b = scn.nextLine();
         System.out.printf("You have entered:- "
                 + a + " " + "and name as " + b);
     }
}
```

<!--more-->

Input：

```
50
Geek
```
Output：

```
Enter an integer
Enter a String
You have entered:- 50 and name as
```

让我们尝试使用BufferReader类，并且使用相同的输入

```
// Code using BufferedReader Class
import java.io.*;
class Differ
{
    public static void main(String args[])
                  throws IOException
    {
        BufferedReader br = new BufferedReader(new
        InputStreamReader(System.in));
        System.out.println("Enter an integer");
        int a = Integer.parseInt(br.readLine());
        System.out.println("Enter a String");
        String b = br.readLine();
        System.out.printf("You have entered:- " + a +
                          " and name as " + b);
    }
}
```

Input：

```
50
Geek
```
Output：

```
Enter an integer
Enter a String
you have entered:- 50 and name as Geek
```
在Scanner类中如果我们在这任何7个nextXXX()方法之后调用nextLine()方法，这nextLine()方法不能够从控制台读取任何内容，并且，这游标不会进入控制台，它将跳过这一步。这nextXXX()方法是这些方法，`nextInt(),nextFloat(), nextByte(), nextShort(), nextDouble(), nextLong(), next()。`

在BufferReader类中就没有那种问题。这种问题仅仅出现在Scanner类中，由于nextXXX()方法忽略***换行符***，但是，nextLine()并不忽略它。如果我们在nextXXX()方法和nextLine()方法之间使用超过一个以上的nextLine()方法，这个问题将不会出现了；因为nextLine()把换行符消耗了。可以参考这个程序的[正确写法](http://code.geeksforgeeks.org/CErAhD)。这个问题和[C/C++](http://www.geeksforgeeks.org/problem-with-scanf-when-there-is-fgetsgetsscanf-after-it/)中的scanf()方法紧跟gets()方法的问题一样。

其他的不同点：

- BufferedReader是支持同步的，而Scanner不支持。如果我们处理多线程程序，BufferedReader应当使用。
- BufferedReader相对于Scanner有足够大的缓冲区内存。
- Scanner有很少的缓冲区(1KB字符缓冲)相对于BufferedReader(8KB字节缓冲)，但是这是绰绰有余的。
- BufferedReader相对于Scanner来说要快一点，因为Scanner对输入数据进行类解析，而BufferedReader只是简单地读取字符序列。

原文链接：[Difference between Scanner and BufferReader Class in Java](http://www.geeksforgeeks.org/difference-between-scanner-and-bufferreader-class-in-java/)

翻译：crane-yuan

[ 转载请保留原文出处、译者和译文链接。]







