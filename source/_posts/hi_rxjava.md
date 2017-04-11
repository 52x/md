title: Hi, RxJava
date: Sep 14, 2016 1:29 AM
categories: "android"
tags: [android,]
---

本文是一篇简单的 RxJava 的介绍。RxJava 介绍是 : Reactive Extensions for the JVM.


<!--more-->

![http://harchiko.qiniudn.com/rxjava-series-cover.jpg](http://harchiko.qiniudn.com/rxjava-series-cover.jpg)

## Reactive 

Rx 是 ReactiveX 的缩写。 Reactive 在字典中的解释是:

"readily responsive to a stimulus" 对刺激易于反应

RxJava 是 Rx 的 Java 实现。

## 观察者模式

观察者模式是 RxJava 消息通知使用的模式。

如果你知道『薛定谔的猫』，下面可能有助于你来理解。

薛定谔 -> Observer

猫 -> Observable

当猫的状态发生改变时，通知薛定谔。『具体观察者模式请参考相关书籍』

**在 Android 中， 相关操作在子线程完成后，在 主线程 更新 UI。**

## Reactive Programming

Reactive programming is programming with asynchronous data streams.

关键词: 异步/数据流

### 异步能解决什么问题？

举个例子：
* 使用 Rx 请求网络数据，不用阻塞主线程。
* 使用 Rx 读写数据库，不用阻塞主线程。

### 你可能会问，为什么不阻塞主线程？

因为，如果有界面，会造成界面卡顿，即便没界面，程序卡在那里也不好。

况且 Android 中， 网络相关操作在多少个版本之前就不能在主线程调用了，调用直接告诉你不能在主线程，你网络请求多快都不行！

RxJava 中，很简单，主线程执行相关UI操作，费事线程放到其他线程来做。

**你可以忘掉 AsyncTasks 了**

### 你可能还会问，我就想在主线程用Rxjava，可以吗？

当然可以，没有异步操作的话就行，比如 你发射一个 array , 然后对这个 array 进行简单操作。

这种操作就相当于 for 循环了。

### 数据流

Rx 涉及到一个 『流』 的概念。

万物皆可为流，事物在某个维度上的运动就可以成为流。

譬如： 

* 高速公路上的车流。
* 人随着时间的推移渐渐由婴儿到成年。『长大真的好烦啊』 

在 Rx 中， 流以时间为维度。

**数据流**就像一条河：它可以被观测，被过滤，被操作，或者为新的消费者与另外一条流合并为一条新的流。『这句话我摘抄的』

### 联合起来

* 网络请求

异步 + 数据流 =>  网络请求(数据传回之后，执行后续动作『观察者模式』) -> Json -> Objects -> 处理

* 多次点击事件

异步 + 数据流 => 点击多次(点击之后的动作也是观察者模式) -> 获取间隔短的点击 -> 合并点击事件 -> 过滤出 多次点击事件

见下图:

![http://harchiko.qiniudn.com/32YBZv.png](http://harchiko.qiniudn.com/32YBZv.png)

## 怎么用

以下是我学习 RxJava 时所看的一些资料，按照我认为重要程度及学习路线简单划分了一下:

1. 看一下 RxJava 解决了 Android 开发中的什么痛点：[http://www.andreamaglie.com/rxjava-android-where-to-start/](http://www.andreamaglie.com/rxjava-android-where-to-start/)
2. 学习下基本的 操作[https://gist.github.com/staltz/868e7e9bc2a7b8c1f754](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)
3. 重点看一下这个 [http://yarikx.github.io/NotRxJava/](http://yarikx.github.io/NotRxJava/)
4. 看 wiki [https://github.com/ReactiveX/RxJava/wiki](https://github.com/ReactiveX/RxJava/wiki)

借用『Yaroslav Heriatovych』的那句话，『Once you start using it in your project – you will use it everywhere.』


-----

可爱的萌妹子放到最后：

![http://harchiko.qiniudn.com/58950333_p0.jpg](http://harchiko.qiniudn.com/58950333_p0.jpg)






