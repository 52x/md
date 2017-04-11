title: Content Provider 指南
date: 2016-04-10 23:58:05
categories: "android"
tags: [android,dev]
---

## 介绍

以下摘自开发文档

> Content providers are one of the primary building blocks of Android applications, providing content to applications. They encapsulate data and provide it to applications through the single ContentResolver interface. A content provider is only required if you need to share data between multiple applications. For example, the contacts data is used by multiple applications and must be stored in a content provider. **If you don't need to share data amongst multiple applications you can use a database directly via SQLiteDatabase. **

<!--more-->

## 使用场景

假设现在你需要开发一个应用，拍摄个人名片，然后将名片上的联系人信息储存，或许你之后会丢失名片，但是你的联系人已经保存下来了。

但是保存联系人本身需要操作联系人的数据库，如何确保我们使用的数据库跟手机上默认的联系人数据存储一致呢？

这个时候，你就需要使用 ContentProvider，它能够作为一个中间人来处理数据与数据库之间的联系。

### 你可能在想，为什么我需要一个中间人来做这件事情呢？我不能直接访问数据库来做这件事情吗？

这样做有几个好处:

1. 修改相关的数据方便。
2. 能够利用Android的各种相关Class类，如Cursor Adapter。
3. 能够让更多的应用能够Access，而且相应的安全性也会提高（ContentProvider用到的什么方法呢？要研究一下）。


所以上面那个问题怎么解决呢？

使用自己的数据库？不，这行不通，因为系统的数据库无法访问你自己创建的数据库，答案就是使用contentprovider,ContentProvider的处理对你来说完全是BlackBox.

### 怎么用？
![detail](http://harchiko.qiniudn.com/content_provider_explicit.png)
如上

```java
ContentResolver contentResolver = getContentResolver();
Cursor cursor = contentResolver.query(UserDictionary.Words.CONTENT_URI, null,null,null,null);
```
