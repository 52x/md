---
title: UML类图几种关系的总结
date: 2014-08-18 23:01:05
tags:
  - UML
categories:
  - Programming
  - 设计模式
---

原文：<http://blog.csdn.net/tianhai110/article/details/6339565>

在UML类图中，常见的有以下几种关系：泛化（Generalization），实现（Realization），关联（Association），聚合（Aggregation），组合（Composition），依赖（Dependency）。
<!--more-->

## 泛化(Generalization)

<span style="color:red;">【泛化关系】：是一种继承关系，它指定了子类如何特化父类的所有特征和行为。</span>例如：老虎是动物的一种。

【箭头指向】：带三角箭头的实线，箭头指向父类。
{% asset_img generalization.png %}

## 实现（Realization)

<span style="color:red;">【实现关系】：是一种类与接口的关系，表示类是接口所有特征和行为的实现。</span>

【箭头指向】：带三角箭头的虚线，箭头指向接口。
{% asset_img realization.png %}

## 关联（Association）

<span style="color:red;">【关联关系】：是一种拥有的关系，它使一个类知道另一个类的属性和方法。</span>如：老师与学生、丈夫与妻子。

关联可以是双向的，也可以是单向的。双向的关联可以有两个箭头或者没有箭头，单向的关联有一个箭头。

<span style="color:green;">【代码体现】：成员变量。</span>

【箭头及指向】：带普通箭头的实心线，指向被拥有者。
{% asset_img association.png %}
上图中，老师与学生是双向关联，老师有多名学生，学生也可能有多名老师。但学生与某课程间的关系为单向关联，一名学生可能要上多门课程，课程是个抽象的东西，它不拥有学生。
{% asset_img association_self.png %}
上图为自身关联。

## 聚合（Aggregation）

<span style="color:red;">【聚合关系】：是整体与部分的关系。</span>如车和轮胎是整体和部分的关系。

聚合关系是关联关系的一种，是强的关联关系；关联和聚合在语法上无法区分，必须考察具体的逻辑关系。

<span style="color:green;">【代码体现】：成员变量。</span>

【箭头及指向】：带空心菱形的实心线，菱形指向整体。
{% asset_img aggregation.png %}

## 组合(Composition)

<span style="color:red;">【组合关系】：是整体与部分的关系。</span>没有公司就不存在部门。</span>

组合关系是关联关系的一种，是比聚合关系还要强的关系，它要求普通的聚合关系中代表整体的对象负责代表部分的对象的生命周期。

<span style="color:green;">【代码体现】：成员变量。</span>

【箭头及指向】：带实心菱形的实线，菱形指向整体。
{% asset_img composition.png %}

## 依赖(Dependency)

<span style="color:red;">【依赖关系】：是一种使用的关系，所以要尽量不使用双向的互相依赖。</span>

<span style="color:green;">【代码表现】：局部变量、方法的参数或者对静态方法的调用。</span>

【箭头及指向】：带箭头的虚线，指向被使用者。
{% asset_img dependency.png %}

## 各种关系的强弱顺序

**泛化= 实现> 组合> 聚合> 关联> 依赖**

下面这张UML图，比较形象地展示了各种类图关系：
{% asset_img summary.png %}

企鹅与气候的关系：关联 or 依赖？
> 程杰<<大话设计模式>>P14页这样解释："你看企鹅和气候两个类，企鹅是很特别的鸟，会游不会飞。更重要的是，它与气候有很大的关联。我们不去讨论为什么北极没有企鹅，为什么它们要每年长途跋涉。总之，企鹅需要‘知道’气候的变化，需要‘了解’气候规律。当一个类‘知道’另一个类时，可以用关联（association）。关联关系用实线箭头来表示。"
