---
title: React-Native 奇葩报错
tags: 8081port
categories: react-native学习笔记
toc: false
date: 2017-03-22 15:47:20
---
刚写了一个react-native小demo，完美运行，心中窃喜，于是关闭项目。过了一会儿，想再一睹刚才如行云流水般运行的项目，于是，运行，结果......纳尼？出现了一片姨妈红有没有？心中一万头草泥马呼啸而过有没有？到底发生了什么？
<!--more-->

NA！阿sir啊，我说了我没动过代码啦，出现下面这种错误不关我的事啊：
```
ProjectName has not been registered. 
This is either due to a require() error during initialization or failure to call AppRegistry.registerComponent.
```

分析错误原因：


## **1.第一种情况：**

程序入口处项目名称不一致。检查发现：

```
AppRegistry.registerComponent('ProjectName', () => ProjectName);

```

一模一样！为了担心怕自己的眼睛看到的不是真实的，特地粘贴复制了一遍！

第一种情况排除！

## **2.第二种情况**

**8081端口被占用**

`检验方法：到项目根目录下--------->>>打开终端--------->>>输入命令行：react-native start`

如果出现了`Packager can't listen on port 8081`,好的恭喜你，找到了症结所在，下面就是根据提示来就好了：

<1>.lsof -i : 8081 //列出被占用的端口
<2>.kill -9 < PID >  //找出与之对应的PID，杀死就ok了
<3>.重新运行项目
<4>.依旧完美

