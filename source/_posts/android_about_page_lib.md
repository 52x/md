title: Android About Page 生成
date: 2016-06-13 10:32 AM
categories: "android"
tags: [android,]
---
在写Android程序时经常需要写 About Page 来介绍公司状况，以及联系方式等，这个库能让你在**2分钟**之内完成About Page的编写。

<!--more-->

![http://harchiko.qiniudn.com/android_about_page_cover.png](http://harchiko.qiniudn.com/android_about_page_cover.png)

## 安装
Available on Jcenter, Maven and JitPack

```groovy
compile 'com.github.medyo:android-about-page:1.0.8'
```


## 用法
### 1. Add Description

```java
setDescription(String)
```

### 2. Add Image
```java
setImage(Int)
```

### 3. Add predefined Social network
The library has already some predefined social networks like :  

* Facebook
* Twitter
* Instagram
* Youtube
* PlayStore

```java
addFacebook(String PageID)
addTwitter(String AccountID)
addYoutube(String AccountID)
addPlayStore(String PackageName)
addInstagram(String AccountID)
addGitHub(String AccountID)
```

### 4. Add Custom Element
For example `app version` :

```java
Element versionElement = new Element();
versionElement.setTitle("Version 6.2");
addItem(versionElement)
```

### 5. Available attributes for Element Class

| Function        | Description  |
| ------------- |:-------------:| -----:|
| setTitle(String) | Set title of the element|
| setColor(Int) | Set color of the element|
| setIcon(Int) | Set icon of the element|
| setValue(String) | Set Element value like Facebook ID|
| setTag(String) | Set a unique tag value to the element|
| setIntent(Intent) | Set an intent to be called on `onClickListener` |
| setGravity(Gravity) | Set a Gravity for the element  |
| setOnClickListener(View.OnClickListener) | If `intent` isn't suitable for you need, implement your custom behaviour by overriding the click listener|


## Sample Project
[medyo/android-about-page/app/](https://github.com/medyo/android-about-page/tree/master/app)

---------
以上说明来自github

## 配置 image 大小属性
![http://harchiko.qiniudn.com/Screen%20Shot%202016-06-13%20at%2010.40.04%20AM.png](http://harchiko.qiniudn.com/Screen%20Shot%202016-06-13%20at%2010.40.04%20AM.png)

可以看到在库文件中 `style.xml` 中定义了各元素的属性，在自己的项目中添加 style 属性即可。

```xml

    <style name="about_image">
        <item name="android:layout_width">match_parent</item>
        <item name="android:layout_height">match_parent</item>
        <item name="android:adjustViewBounds">true</item>
        <item name="android:scaleType">centerInside</item>
        <item name="android:layout_weight">1</item>
        <item name="android:gravity">center_horizontal</item>
        <item name="android:padding">100dp</item>
    </style>
```

顺便，关于android图片缩小且不失比例的方法见链接: [http://stackoverflow.com/questions/8232608/fit-image-into-imageview-keep-aspect-ratio-and-then-resize-imageview-to-image-d](http://stackoverflow.com/questions/8232608/fit-image-into-imageview-keep-aspect-ratio-and-then-resize-imageview-to-image-d)

## 配置Description

同理，配置 Description 等在 ｀String.xml｀ 中定义好即可。