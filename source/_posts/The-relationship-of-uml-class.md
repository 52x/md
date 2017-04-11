---
title: UML类之间的六大关系总结
date: 2016-08-02 13:02:29
tags: uml
categories: Java平台
---

## 在UML类图中，常见的有以下几种关系: 

 - 泛化（Generalization）
 - 实现（Realization）
 - 关联（Association)
 - 聚合（Aggregation）
 - 组合(Composition)
 - 依赖(Dependency)

各种关系的强弱顺序：

**泛化= 实现> 组合> 聚合> 关联> 依赖**

<!--more-->

## 泛化（Generalization）：

类之间的继承关系用泛化。

【箭头指向】：带三角箭头的实线，箭头指向父类

 ![泛化](http://7xsd89.com1.z0.glb.clouddn.com/uml/Generalization.jpg)

## 实现（Realization）

类实现接口的关系使用实现。

【箭头指向】：带三角箭头的虚线，箭头指向接口

![Realization](http://7xsd89.com1.z0.glb.clouddn.com/uml/Realization.jpg)

## 关联（Association)

类之间的拥有关系用关联。

【箭头及指向】：带普通箭头的实心线，指向被拥有者

![Association](http://7xsd89.com1.z0.glb.clouddn.com/uml/Association.jpg)

## 聚合（Aggregation）

聚合是一种弱的整体与部分的关系，整体可以脱离部分而单独存在。

【箭头及指向】：带空心菱形的实心线，菱形指向整体

![Aggregation](http://7xsd89.com1.z0.glb.clouddn.com/uml/Aggregation.jpg)

## 组合(Composition)

组合是一种强的整体与部分的关系，整体不可脱离部分而存在。

【箭头及指向】：带实心菱形的实线，菱形指向整体

![Composition](http://7xsd89.com1.z0.glb.clouddn.com/uml/Composition.jpg)

## 依赖(Dependency)

依赖是一种使用的关系。

【箭头及指向】：带箭头的虚线，指向被使用者

![Dependency](http://7xsd89.com1.z0.glb.clouddn.com/uml/Dependency.jpg)

## 总结

一个完整的uml类图。

 ![完整uml图](http://7xsd89.com1.z0.glb.clouddn.com/uml/uml.jpg)
