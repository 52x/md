---
title: eclipse+tomcat的问题能启动，但量不能访问
date:  2015-10-6 13:43:44
tags: [java, tomcat]
categories: [java,tomcat]
---

今天还发现了一种情况tomcat起不来，配置是默认的，就是下面两张图的第一张图的配置。但是就是起不来，原因是web.xml配置有问题也会出现这种情况。

如果，按钮是灰的不能选择，那就把包含的项目先删掉，再clean一下就可以。

原来的解决方法：
tomcat启动了但是却访问不了是因为：
![原来配置](eclipse-tomcat-start-question/1.png)
改成以下即可：
![原来配置](eclipse-tomcat-start-question/2.png)