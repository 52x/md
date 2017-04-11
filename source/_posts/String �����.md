---
title: 几个 String 的知识点
date: 2015-07-22 00:40:06
tags: 
 - java 
 - String 
 - final
categories: 
 java
---

### String 类
不可变字符对象。
所以每次String对象的改变实际是创建了一个新对象。
String 是 final 的也不能被继承。
```java
String test1 = new String("ABCD");
String test2 = "ABCD";	//通过字符串常量创建一个String对象。
String test3 = "ABCD";
test3 += "EFGH";	     //创建了一个新的String对象。
StringBuilder test4 = new StringBuilder(test3);

System.out.println(test1==test2);
System.out.println(test1.equals(test2));
```

由于String每次相加时都是创建一个新对象，所以尽量避免创建大是的String对象。比如：
```java
for (int i = 0; i < 1000; i++) {
	test1 += test1;
}
```
<!--more-->
如果非得用这种方式的话，尽量用StringBuild来创建。
```java
for (int i = 0; i < 1000; i++) {
	test4.append("xxxx");
}
```

### 防乱码处现
如果IO流中有泛及到中文，那肯定有乱码问题。处现方式：
```java
if("teachername".equals(fileItem.getFieldName())){
		String value = fileItem.getString();
		teacherName = new String(value.getBytes("ISO-8859-1"),"UTF-8");
}
```

### 空窜注意
在使用split切割后判断字符串空串时，要注意，" "一个空格不能用一个空格" "来判断，而是用"" 无任何空格的空串来进行判断才能成功。便是单个string判断空串确没有问题？？？
```java
String str = "aa bb   ccc       ddd";
		String[] newStr = str.split(" ");
		System.out.println(Arrays.toString(newStr));
		for(String str2 : newStr){
			if(!"".equals(str2)){
				System.out.println(str2);
			}
}
```

    结果：
    [aa, bb, , , ccc, , , , , , , ddd]
    aa
    bb
    ccc
    ddd
