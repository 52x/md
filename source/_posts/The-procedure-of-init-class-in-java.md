---
title: Java中class的初始化顺序
date: 2016-08-15 15:18:12
tags: Java
toc: true
categories: Java平台
---

## class的装载

在讲class的初始化之前，我们来讲解下class的装载顺序。

以下摘自《Thinking in Java 4》
> 由于Java 中的一切东西都是对象，所以许多活动
变得更加简单，这个问题便是其中的一例。正如下一章会讲到的那样，每个对象的代码都存在于独立的文件
中。除非真的需要代码，否则那个文件是不会载入的。通常，我们可认为除非那个类的一个对象构造完毕，
否则代码不会真的载入。由于static 方法存在一些细微的歧义，所以也能认为“类代码在首次使用的时候载入”。
首次使用的地方也是static 初始化发生的地方。装载的时候，所有static 对象和static 代码块都会按照本
来的顺序初始化（亦即它们在类定义代码里写入的顺序）。当然，static 数据只会初始化一次。

简要的说就是，在类有继承关系时，类加载器上溯造型，进行相关类的加载工作。

比如：

```
Class B extends Class A
当我们new B()时，类加载器自动加载A的代码
```

## class的初始化顺序

通常是以下这样的初始化顺序：

```
(static对象和static代码块，依据他们的顺序进行初始化)>成员变量>构造函数
```
![class初始化](http://img.blog.csdn.net/20160815151050452)

## 测试代码

<!--more-->

```
public class ClassInit {

	/**
	 * @Title: 			main
	 * @Description: 	类初始化顺序测试
	 * @param: 			@param args   
	 * @return: 		void   
	 * @throws
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub		
		new B();
	}

}

class A {
	static{
		System.out.println("A的static代码块...");
	}
	public String s1 = prtString("A的成员变量...");
	public static String s2 = prtString("A的static变量...");
	public A(){
		System.out.println("A的构造函数...");
	}
	
	public static String prtString(String str) {
		System.out.println(str);
		return null;
	}
}

class B extends A{
	public String ss1 = prtString("B的成员变量...");
	public static String ss2 = prtString("B的static变量...");
	public B(){
		System.out.println("B的构造函数...");
	}
	private static A a = new A();
	static{
		System.out.println("B的static代码块...");
	}
	{
		System.out.println("代码块...");
	}	
}
```

## 测试结果

```
A的static代码块...
A的static变量...
B的static变量...
A的成员变量...
A的构造函数...
B的static代码块...
A的成员变量...
A的构造函数...
B的成员变量...
代码块...
B的构造函数...
```
