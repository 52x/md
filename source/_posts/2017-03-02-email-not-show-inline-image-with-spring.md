---
title: 记一例Spring中使用inline image 图片不显示的问题
tags: [spring]
date: 2017-03-02 08:42:30
updated: 2017-03-02 08:42:30
categories: [Spring]
keywords: [spring,inline image,email]
description:
---

环境Spring Boot，使用smtp发送邮件，使用Thymeleaf模板，碰到一个问题，使用`message.addInline(image.get("imageResourceName"), imageSource, image.get("imageContentType"));`，总是看不到图片。

查找原因，发现生成的模板、代入的参数等都是正常的，百思不得其解。

google之，发现了一个提醒，才知道问题出在那里

我把addInline 放在了setText之前导致的问题

详见[addInline javadoc](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/mail/javamail/MimeMessageHelper.html#addInline-java.lang.String-org.springframework.core.io.Resource-) 其中的Note有一句

>  **NOTE:** Invoke `addInline` *after* [`setText(java.lang.String)`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/mail/javamail/MimeMessageHelper.html#setText-java.lang.String-); else, mail readers might not be able to resolve inline references correctly.	

这样就真相大白了。

特记录我这小白问题，哈哈

