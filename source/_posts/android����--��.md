title: Android开发--起步
date: 2014-03-07 09:19:17
tags: [Android, Eclipse]
categories: [Android]
description: 打算在暑假准备找工作之前再学习一个技能，思来想去选择了Android开发。在了解了下Android背景后，第一步就是开发环境的搭建，借助于ADT Bundle，环境搭建再也没有以前的类似痛楚。顺道发现了Eclipse+Emacs，哈哈～只要想做，那就开搞！

---
暑假找工作前再学习一个技能，这次坚决不加黄点，来点实在的。考虑自身将来的发展，我一直希望以后的工作围绕**用户体验**，不管是App开发、Web开发，甚至机器学习一类的研究开发，我都希望离用户近点。对于Android开发，我还几乎是零基础，Java也不太会，不过任何学问技术，在精通之前总是相对容易的，只有愿不愿意了！由于某些原因，这次得在windows下先混一段时间了，Go!

# ADT-Bundle快速搭建
ADT-Bundle是由[Google Android官方][ref1]提供的集成式IDE，其中包含了**Eclipse**，你无需再去下载**Eclipse**，并且里面已集成了插件，它解决了大部分新手通过**Eclipse**来配置Android开发环境的复杂问题。

## ADT-Bundle下载
下载链接：[ADT-Bundle下载][ref2]，我的下载包的文件名为：`adt-bundle-windows-x86-20131030.zip`，ADT-Bundle包括了开发所需的一切，解压后目录：![directory](/image/directory.PNG)

## 安装JDK
Android开发是基于Java语言、工具和库的，所以得安装Java。现在已经到了JDK 1.7版本，下载链接：[JDK下载][JDK]，关于如何安装，配置`JAVA_HOME`、`CLASSPATH`、`PATH`自行[Google][]之。检测是否配置成功：cmd终端里面`java -version`,`javac -version`看结果即可。

## 设置Tools PATH
为了方便以后在命令行使用Android SDK Tools，可以将`<your folder>/android-sdk/tools/`和`<yobur folder>/android-sdk/platform-tools/`两个路径添加到PATH环境变量中，其中`<your folder>`为你的`android-sdk`文件夹放置目录。

## SDK管理器
运行ADT-Bundle下载包的解压根目录下的`SDK Manager.exe`，进入界面选择需要的Tools以及API等级进行安装或更新。在**Eclipse**中也可以打开，打开过程：Window -> Android SDK Manager。

## 配置AVD
AVD（Android Virtual Device），即模拟器，运行在电脑上的安卓虚拟设备，几乎跟一台真正的安卓设备一样。打开Window -> Android Virtual Device Manager，然后`New`一个，见到下图界面：![avd](/image/avd.PNG)配置完后点击`Start`启动。

# Hello World
创建一个Hello World项目，通过File -> New -> Android Application Project，然后输入Hello World（项目名字),一路到底。模拟器调试运行：右击android工程 -> Run as -> Android Application。

# Eclipse + Emacs
现在写**Python**和写博客都习惯用**Emacs**编辑器，基本脱离鼠标，全键盘操作。在**Eclipse**里面写了几行代码，感觉不太爽，就到Window -> Preferences -> General -> Keys里面配置快捷键。来来回回的配置+体验，就在那么一瞬间发现了好东西，截图留作纪念：![keysSchema](/image/eclipse_emacs.png)真后悔没早点发现！发现后赶紧[Google][]，看看有没有更好的**Emacs**配置，后来发现了加强版插件**Emacs+**，也就是上面截图中的选择项了，关于如何安装，很简单，这里给个链接：[Emacsplus][]。

# 回头看看
虽然ADT-Bundle这么完善，这样方便了入门，却容易让人不知其所以然，所以建议还是了解下集成的部件是干嘛的，或者看看分步骤配置的过程。最后留几个知识点：
1. PATH环境变量：作用是指定命令搜索路径，在命令行下面执行命令如javac编译java程序时，它会到PATH变量所指定的路径中查找看是否能找到相应的命令程序。我们需要把jdk安装目录下的bin目录增加到现有的PATH变量中，bin目录中包含经常要用到的可执行文件如javac/java/javadoc等待，设置好PATH变量后，就可以在任何目录下执行javac/java等工具了。
2. CLASSPATH环境变量：作用是指定类搜索路径，要使用已经编写好的类，前提当然是能够找到它们了，JVM就是通过CLASSPATH来寻找类的。我们需要把jdk安装目录下的lib子目录中的dt.jar和tools.jar设置到CLASSPATH中，当然，当前目录“.”也必须加入到该变量中。
3. JAVA_HOME环境变量。它指向jdk的安装目录，Eclipse/NetBeans/Tomcat等软件就是通过搜索JAVA_HOME变量来找到并使用安装好的jdk。

[JDK]: http://www.oracle.com/technetwork/java/javase/downloads/index.html
[Google]: http://www.google.com.hk/
[Emacsplus]: http://www.mulgasoft.com/emacsplus
[ref1]: http://developer.android.com/index.html
[ref2]: http://developer.android.com/sdk/index.html