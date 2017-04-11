title: YAML简介
date: 2015-11-24 03:02:25
categories: 
tags: [dev,]
---
## YAML简介

`YAML` **[YAML Ain't Markup Language]**

和GNU一样，YAML是一个递归着说“不”的名字。不同的是，GNU对UNIX说不，YAML说不的对象是XML。

<!--more-->

![image](http://harchiko.qiniudn.com/3389850854144361888.jpeg)

## 语法

`Structure`通过**空格**来展示。`Sequence`里的项用"-"来代表，`Map`里的键值对用":"分隔.
这几乎就是所有的语法了.

> 注意是空格不是`TAB`, `Makefile`刚好相反。

## 举例

```yaml
name: John Smith
age: 37
spouse:
    name: Jane Smith
    age: 25
children:
    -   name: Jimmy Smith
        age: 15
    -   name: Jenny Smith
        age: 12
```

John今年37岁，有一个幸福的四口之家。两个孩子Jimmy 和Jenny活泼可爱。妻子Jane年轻美貌。
可见YAML的可读性是不错。

顺便推荐下在线yaml解析器：[http://yaml-online-parser.appspot.com/](http://yaml-online-parser.appspot.com/)

参考链接： [http://www.ibm.com/developerworks/cn/xml/x-cn-yamlintro/](http://www.ibm.com/developerworks/cn/xml/x-cn-yamlintro/)