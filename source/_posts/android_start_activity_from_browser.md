title: Android 如何根据浏览器URL启动应用
date: Aug 22, 2016 11:09 AM
categories: "android"
tags: [android,]
---
因为正在完成的应用需要用到`OAuth2`来进行身份认证，需要跳转到浏览器，然后再跳转回来，接收浏览器返回的信息。这里面用到了 `Implicit Intent`
<!--more-->

艾米莉亚镇楼:

![http://harchiko.qiniudn.com/58541110_p0.jpg](http://harchiko.qiniudn.com/58541110_p0.jpg)

## 假设

假设我们要解析的网址是: `https://zhaochunqi.inoreader`

很明显，打不开，这是我乱编的。

## 定义

在 `AndroidManifest`中定义 `intentfilter`

在需要回调 Activity 中定义

```xml
<intent-filter>
	<action android:name="android.intent.action.VIEW" />
	<category android:name="android.intent.category.BROWSABLE" />
	<category android:name="android.intent.category.DEFAULT" />
	<data
		android:host="zhaochunqi.inoreader"
		android:scheme="https"
		android:pathPattern="/*"/>
</intent-filter>
```

下面逐行解释:

`<action android:name="android.intent.action.VIEW" />`

>通知 Android 此 Activity 可以处理 ACTION_VIEW 类型的 Intent.

`<category android:name="android.intent.category.BROWSABLE" />`

>从浏览器中唤醒的应用必须包含此声明

`<category android:name="android.intent.category.DEFAULT" />`

>所有的 implicit Intent 必须包含此声明，否则将无法被感知，也就无法接受到 implicit Intent.

```xml
<data
		android:host="zhaochunqi.inoreader"
		android:scheme="https"
		android:pathPattern="/*"/>
```

`scheme`这个必须有，定义`uri`
`host`定义`uri`的主机部分
`pathPattern`匹配，这个可有可无，没有也毫无影响。

看看成果:

![http://harchiko.qiniudn.com/2016-08-22intent.png](http://harchiko.qiniudn.com/2016-08-22intent.png)

Looks Great!
