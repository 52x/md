title: 如何动态选择 Android 需要启动的 Activity
date: Oct 18, 2016 9:27 AM
categories: "android"
tags: [activity,]
---
> Android 开发过程中经常遇到第一次启动时需要登录或者一段介绍什么的。但之后再次启动时则不在需要的情况，对于这种情况，如何设定启动的 Activity 呢?

<!--more-->

## 场景

第一次登录需要跳转到登录界面，而后来登录过后则只需直接跳转到需要显示的界面即可。

## 方法1

设定一个无界面的跳转 Activity:

```java
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    Intent intent;
    if (condition) {
       intent = new Intent(this, ClassA.class);
    } else {
       intent = new Intent(this, ClassB.class);
    }
    startActivity(intent);
    finish();
    // note we never called setContentView()
}
```
**注意: 没有 `setContentView()`。否则你会看到一个跳转界面。**

## 方法2

动态设置布局:

```java
if (!userIsLoggedIn) {
    setContentView(R.layout.signup);
} else {
    setContentView(R.layout.homescreen);
}
```

反正我不推荐用这个。


参考链接:

* [http://stackoverflow.com/questions/4856539/dynamic-start-activity-in-android](http://stackoverflow.com/questions/4856539/dynamic-start-activity-in-android)