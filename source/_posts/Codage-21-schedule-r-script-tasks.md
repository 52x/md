title: WIN7定期自动运行R脚本
subtitle: (其它脚本同样适用)
date: 2016-02-19 11:03:18
categories: Codage|编程
tags: [R, Task Scheduler, WIN7]
---

懒癌又犯了，最近要发给同事的一些东西，一点都不想每天都去手动运行了。
研究了一下如何用WIN7系统自动调用R脚本。
果然懒才是第一生产力啊……

<!-- more -->

> 思路：利用WIN7自带的**任务计划程序**调用`.bat`文件，用此文件运行相关程序和脚本。

先准备一个文件夹，在文件夹里创建一个`.bat`文件。
我将脚本和`.bat`文件放到了一个文件夹(E:\Code_Repository\working)下。
打开`.bat`文件并在其中输入：

```cmd
@echo off
"D:\stat-tools\R-3.2.3\bin\R.exe" CMD BATCH "E:\Code_Repository\working\Italy_merchandise.R"
```

其中，`"D:\stat-tools\R-3.2.3\bin\R.exe"`是R.exe的位置，`CMD BATCH`用于调用`"E:\Code_Repository\working\Italy_merchandise.R"`这个脚本。

然后打开**任务计划程序**：

> 开始 -> 所有程序 -> 附件 -> 系统工具 -> 任务计划程序
> Start -> START -> All Programs -> Accesories -> System Tools -> Scheduler

按照下图的顺序设定参数：

![step1](http://7xndoy.com1.z0.glb.clouddn.com/Codage-21-step1.png)

![step2](http://7xndoy.com1.z0.glb.clouddn.com/Codage-21-step2.png)

![step3](http://7xndoy.com1.z0.glb.clouddn.com/Codage-21-step3.png)

![step4](http://7xndoy.com1.z0.glb.clouddn.com/Codage-21-step4.png)

![step5](http://7xndoy.com1.z0.glb.clouddn.com/Codage-21-step5.png)

![step6](http://7xndoy.com1.z0.glb.clouddn.com/Codage-21-step6.png)

每次运行完成以后，可以在`.bat`文件所在的文件夹下看到运行的日志，文件以`.Rout`结尾。
如果任务执行完毕没有期望的输出，就可以通过`.Rout`判断哪里出了问题。

-----------------------------------

注意：
1. 脚本运行的环境与R的默认工作路径可能有异，需要在脚本中提前设定好工作路径
2. 包含中文的脚本，`UTF-8`的编码格式也可能无法运行，那么需要将脚本另存为`GBK`编码格式

-----------------------------------

Reference:
* [Scheduling R Script](http://stackoverflow.com/questions/2793389/scheduling-r-script)
* [Scheduling R Tasks via Windows Task Scheduler](http://www.r-bloggers.com/scheduling-r-tasks-via-windows-task-scheduler/)