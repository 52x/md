title: NSLog效率低下的原因及尝试lldb断点打印Log
date: 2014-04-22 19:47:41
tags: objc刨根问底
---

# 我是前言
打Log是我们debug时最简单朴素的方法，`NSLog`对于objc开发就像`printf`对于c一样重要。但在使用`NSLog`打印大量Log，尤其是在游戏开发时（如每一帧都打印数据），`NSLog`会明显的拖慢程序的运行速度（游戏帧速严重下滑）。本文探究了一下`NSLog`如此之慢的原因，并尝试使用lldb断点调试器替代NSLog进行debug log。

<!--more-->   

-----

# 小测试

测试下分别使用`NSLog`和`printf`打印10000次耗费的时间。`CFAbsoluteTimeGetCurrent()`函数可以打印出当前的时间戳，精度还是很高的，于是乎测试代码如下：

```
CFAbsoluteTime startNSLog = CFAbsoluteTimeGetCurrent();
for (int i = 0; i < 10000; i++) {
    NSLog(@"%d", i);
}
CFAbsoluteTime endNSLog = CFAbsoluteTimeGetCurrent();

CFAbsoluteTime startPrintf = CFAbsoluteTimeGetCurrent();
for (int i = 0; i < 10000; i++) {
    printf("%d\n", i);
}
CFAbsoluteTime endPrintf = CFAbsoluteTimeGetCurrent();

NSLog(@"NSLog time: %lf, printf time: %lf", endNSLog - startNSLog, endPrintf - startPrintf);
```

这个时间和机器肯定有关系，只看它们的差别就好。为了全面性，尝试了三种平台： 

```
NSLog time: 4.985445, printf time: 0.084193 // mac
NSLog time: 5.562460, printf time: 0.019408 // 模拟器
NSLog time: 10.471490, printf time: 0.090503 // 真机调试(iphone5)
```

可以发现，在mac上（模拟器其实也算是mac吧）速度差别达到了60倍左右，而真机调试甚至达到了离谱的100多倍。  

-----   

# 探究原因
基本上这种事情一定可以在Apple文档中找到，看`NSLog`的文档，第一句话就说：`Logs an error message to the Apple System Log facility.`，所以首先，`NSLog`就不是设计作为普通的debug log的，而是error log；其次，`NSLog`也并非是`printf`的简单封装，而是`Apple System Log`(后面简称ASL)的封装。   
## ASL
ASL是个啥？从[官方手册](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man3/asl.3.html)上，或者从终端执行`man 3 asl`都可以看到说明：   

>These routines provide an interface to the Apple System Log facility.  They are intended to be a
     replacement for the syslog(3) API, which will continue to be supported for backwards compatibility.   
   

大概就是个系统级别的log工具吧，syslog的替代版，提供了一系列强大的log功能。不过一般我们接触不到，NSLog就对它提供了高层次的封装，如[这篇文档](https://developer.apple.com/library/mac/documentation/macosx/conceptual/bpsystemstartup/Chapters/LoggingErrorsAndWarnings.html#//apple_ref/doc/uid/10000172i-SW8-SW1)所提到的：  

>You can use two interfaces in OS X to log messages: ASL and Syslog. You can also use a number of higher-level approaches such as NSLog. However, because most daemons are not linked against Foundation or the Application Kit, the low-level APIs are often more appropriate   

一些底层相关的守护进程(deamons)不会link如Foundation等高层框架，所以asl用在这儿正合适；而对于应用层的用NSLog。   

在`CocoaLumberjack`的[文档](https://github.com/CocoaLumberjack/CocoaLumberjack/wiki/Performance)中也说了NSLog效率低下的问题： 

> NSLog does 2 things:  
> - It writes log messages to the Apple System Logging (asl) facility. This allows log messages to show up in Console.app.   
> - It also checks to see if the application's stderr stream is going to a terminal (such as when the application is being run via Xcode). If so it writes the log message to stderr (so that it shows up in the Xcode console).

> To send a log message to the ASL facility, you basically open a client connection to the ASL daemon and send the message. BUT - each thread must use a separate client connection. So, to be thread safe, every time NSLog is called it opens a new asl client connection, sends the message, and then closes the connection.   

意识大概是说，NSLog会向ASL写log，同时向Terminal写log，而且同时会出现在`Console.app`中（Mac自带软件，用NSLog打出的log在其中全部可见）；不仅如此，每一次NSLog都会新建一个ASL client并向ASL守护进程发起连接，log之后再关闭连接。所以说，当这个过程出现N次时，消耗大量资源导致程序变慢也就不奇怪了。  

## 时间和进程信息
主要原因已经找到，还有个值得注意的问题是`NSLog`每次会将当前的系统时间，进程和线程信息等作为前缀也打印出来，如：  

```
2012-34-56 12:34:56.789 XXXXXXXX[36818:303] xxxxxx
```
当然这些也可能是作为ASL的参数创建的，但不论如何，一定是有消耗的（虽然这个prefix十有八九不是我们需要的看到的）   

------ 


# 如何是好

NSLog有这样的消耗问题，那该怎么办呢？

1. 拒绝残留的Log。现在项目都是多人共同开发，我们应该只把Log作为错误日志或者重要信息的日志使用，commit前请把自己调试的log去掉（尤其是在循环里写log的小伙伴，简直不能一起快乐的玩耍了）
2. release版本中消除Log。debug归debug，再慢也不能波及到release版本，用预编译宏过滤下就好。
3. 是时候换个Log系统了，如`CocoaLumberjack`，自建一个简单的当然也挺好（其实为了项目需要自己也写了个小log系统，实现可以按名字和级别显示log和一些扩展功能，以后有机会分享下）

不过个人认为debug时最好还是用调试器进行调试（尤其是只需要知道某个变量值的时候）

-----


# 尝试使用断点+lldb调试器打Log

关于强大的`lldb`调试器用一个专题来讲都是应该，现在只了解一些皮毛，不过就算皮毛的功能也可以替代NSLog这种方法进行调试了，重要的一点是:**使用断点log不需要重新编译工程**，况且和Xcode已经结合的很好，在此先只说打Log这件事。   


## 简单断点+po(p)
断点时可以在xcode的lldb调试区使用`po`或`p`命令打印对象或变量，对于当前栈帧中引用到的变量都是可见的，所以说假如只是看一眼某个对象运行到这儿是不是存在，是什么值的话，设个断点就够了，况且IDE已经把这个功能集成，鼠标放变量上就可以了。  

lldb一些常用调试技巧可以这篇[入门教程](http://www.cimgf.com/2012/12/13/xcode-lldb-tutorial/)   

## Condition和Action断点
断点不止能把程序断住，触发时也按一定条件，而且可以执行（一个或多个）Action，在断点上右键选择`Edit Breakpoint`，弹出的断点设置中可以添加一些Action：    
![](http://ww2.sinaimg.cn/large/51530583tw1efobdj4pb3j205002wt8n.jpg)   
其中专门有一项就是`Log Message`，做个小测试：   

```
for (int i = 0; i < 10; i++)
{
    // break point here
}
```

设置断点后编辑断点：   

![](http://ww2.sinaimg.cn/large/51530583tw1efobktq0vsj20d806r74z.jpg)

输入框下面就有支持的格式，表达式(或变量)可以使用`@exp@`这种格式包起来。于是乎输出：    

```
break at: 'main()',  count: 4, sunnyxx says : 3
break at: 'main()',  count: 5, sunnyxx says : 4
break at: 'main()',  count: 6, sunnyxx says : 5
```
正如所料。  
更多的调试技巧还需要深入研究，不过可以肯定的是，比起单纯的使用`NSLog`，使用好的工具可以让我们debug的效率更高


-----

# 总结
- `NSLog`耗费比较大的资源
- `NSLog`被设计为error log，是ASL的高层封装
- 在项目中避免提交commit自己的Debug log，release版本更要注意去除`NSLog`，可以使用自建的log系统或好用的log系统来替代`NSLog`      
- debug不应只局限于log满天飞，`lldb`断点调试是一个优秀的debug方法，需要再深入研究下


-----


# References  
https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man3/asl.3.html
http://theonlylars.com/blog/2012/07/03/ditching-nslog-advanced-ios-logging-part-1/   
https://github.com/CocoaLumberjack/CocoaLumberjack/wiki/Performance   
https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/LoggingErrorsAndWarnings.html#//apple_ref/doc/uid/10000172i-SW8-SW1   
http://www.cimgf.com/2012/12/13/xcode-lldb-tutorial/


-----

原创文章，转载请注明源地址，[blog.sunnyxx.com](http://blog.sunnyxx.com)