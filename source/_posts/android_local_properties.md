title: Android 本地变量存储(API Key等)
date: Aug 22, 2016 10:45 AM
categories: "android"
tags: [android,]
---
处于方便替换和隐藏信息的原因，需要讲一些诸如 API KEY 之类的本地变量单独存储。
<!--more-->
## 定义
在`gradle.properties`中定义所需要存储的变量

如: `ApiKey="xxxxxxxxxxxxxxxxxxx"`

## 配置

在 build.gradle(app) 中将变量保存到 BuildConfig 中 添加（放到buildTypes之外）:

```gradle
    buildTypes.each {
        it.buildConfigField 'String', 'API_KEY', ApiKey
    }
```

## 使用
这样，就可以在源码中使用 `BuildConfig.API_KEY` 来调用此变量了。
