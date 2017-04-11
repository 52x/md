title: 修复他人 Android 代码遗留 Bug 的小技巧
date: Sep 24, 2016 1:21 AM
categories: "android"
tags: [android,tips]
---
由于工作需求，近日接触到了他人的遗留代码，并要据此遗留代码进行 Bug 修复，总结一下在修复过程中使用到的一些小技巧。

<!--more-->
谁不喜欢萌妹子呢！
![http://harchiko.qiniudn.com/59097609_p0.jpg](http://harchiko.qiniudn.com/59097609_p0.jpg)

## 寻找 BUG 对应的 Activity

我会要求测试安装 [当前Activity](http://www.coolapk.com/apk/com.willme.topactivity) 这个应用，并在产生 Bug 的地方截图，效果大概是这样:

![http://harchiko.qiniudn.com/Screenshot_1474651833.png](http://harchiko.qiniudn.com/Screenshot_1474651833.png)

很容易就找到对应的 Activity 了对不对？

## 代码跳转

### 文件跳转
这里，我将 跳转到响应文件修改为 `CMD + P` ，原因嘛，为了跟 Sublime 使用起来一致。

![http://harchiko.qiniudn.com/Screen%20Shot%202016-09-24%20at%201.33.59%20AM.png](http://harchiko.qiniudn.com/Screen%20Shot%202016-09-24%20at%201.33.59%20AM.png)

### 方法跳转

我使用 `CMD+R`搜索文件中的方法，快速定位到如 `onCreate()`之类的方法。

### 跳到上一次代码编辑的地方

我使用插件 ideaVim, 然后使用 `CMD+[`, `CMD+]` 跳来跳去。

### 跳到近期编辑过的文件

不多说，快捷键 `CMD + E`

## 分析布局

在调试过程中，某些类似 ListView 这种没有办法直接从 XML 中看出 问题的地方，使用 LayoutInspector 一看就明白了。

这个按钮躲在一个小角落，见图:

![http://harchiko.qiniudn.com/Screen%20Shot%202016-09-24%20at%201.47.22%20AM.png](http://harchiko.qiniudn.com/Screen%20Shot%202016-09-24%20at%201.47.22%20AM.png)

## 事半功倍的日志库

[Logger](https://github.com/orhanobut/logger) 这个对于我这样的懒人用起来太合适了

普通日志长这样:
![http://harchiko.qiniudn.com/current-log.png](http://harchiko.qiniudn.com/current-log.png)

它的日志这样:

![http://harchiko.qiniudn.com/description.png](http://harchiko.qiniudn.com/description.png)

## 善用搜索

使用 `CMD+SHIFT+F`能进行全局搜索，这对于查找某些使用字符串拼接的地方特别有效果。


暂时就这么多吧！