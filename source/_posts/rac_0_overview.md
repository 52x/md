title: Reactive Cocoa Tutorial [0] = Overview
date: 2014-03-06 21:58:30
tags: Reactive Cocoa Tutorial
---
关于这系列（如果真能写下去的话）：说是教程有点狂，边学边总结，更像个笔记吧，等完全用透之后再写就会忘了一开始学习过程中遇到的问题了，Reactive Cocoa（RAC）现在资料真心少，中文英文加起来没几篇，还都是转来转去的。这是个好东西，相信以后用的人会变多，转了请留该文原地址哦~  by sunny
<!--more-->
> PS:
> 这篇文章原来发布在blogcn上，http://www.cnblogs.com/sunnyxx，
> 现在建了自己的blog后会在这上面继续写了。

------

###废话少说 --> **RAC**

　　是什么？怎么来的？干啥用的？ 怎么用的？ 可以观摩无网不剩的blog RAC介绍1和2，在此不啰唆了，简而言之，就是一个函数响应式编程思想在Cocoa下的实现。

###说说在RAC框架下做了一个项目的赶脚吧：

 - 挺新鲜挺有意思，开发人员水平很高，框架封装性和实用性一流，看了看人家对宏的使用发现原来用的纯小儿科，对编译器的控制，block的使用也很值得的学习。
 - 编程思想上的一些改变。原创的一个可能也不大恰当的比喻：原来的编程思想像是“走迷宫”，RAC的编程思想是“建迷宫”。意思是，之前的编程思路是命令式，大概都是“程序启动时执行xxxx，在用户点击后的回调函数执行xxx，收到一个Notification后执行xxx”等等，如同走迷宫一样，走出迷宫需要在不同时间段记住不同状态根据不同情况而做出一系列反应，继而走出迷宫；相比下，RAC的思想是建立联系，像钟表中的齿轮组一样，一个扣着一个，从转动发条到指针走动，一个齿轮一个齿轮的传导（Reactive），复杂但完整而自然。如同迷宫的建造者，在设计时早已决定了哪里是通路，哪里是死路或是哪个路口指向了出口，当一个挑战者（Event）走入迷宫时（Signal），他一定会在设置好的迷宫中的某个路线行走（传递），继而走到终点（Completion）或困死在里面（Error）。
 - 写出代码结构明显不一样。由于RAC将Cocoa中KVO，UIKit Event，delegate，selector等都增加了RAC支持，所以都不用去做很多跨函数的事，比如KVO个对象然后在回调里面xxx，从storyboard里面连个UIButton的IBAction出来xxx，或是设个UITextField的delegate出来去取输入的文本xxx。但在RAC下就像上面比喻的建迷宫，把这些大都放在“-viewDidLoad:”就可以了，当然像UITableView的delegate和data source这么大规模的代理模式就还是老老实实写吧。
简洁。举个栗子：




我就想干这么个事:

> 一个label一个text field，下面输啥上面显示啥





　　老写法大概做法是这个vc实现UITextFieldDelegate协议，把这个text field的delegate设到vc上面，然后在要改变text的那个delegate方法里面取当前text field的text值，再赋给label上；

使用RAC的话就一句话（当然得把这俩控件都IBOutlet出来）：

```
RAC(self.outputLabel, text) = self.inputTextField.rac_textSignal; 
```
看着就挺爽。

复杂的栗子先不举了。

　　总之吧，等今后维护RAC的开发者和使用者把更多的Cocoa的东西归入RAC的框架中，这个框架基本上都可以凌驾于Cocoa这个框架了，意思是甚至用不着知道那些delegate啊KVO啊苹果告诉你是咋用的，用RAC封装的就行了。RAC对于值的显示大都是和property“绑定”的关系，像使用storyboard构建页面时，对于有响应的控件基本都得IBOutlet出来作为一个property，而不是像原来一样连个IBAction出来或者连个delegate出来。对于视图层到model层之间的绑定就显得有些生硬了，相当于视图直接耦合了model，于是应运而生M-V-VM结构，说白了就是在View和Model之间增加了一个ViewModel来解耦，这样View里面要做的基本就是绑定VM以及一些纯视图的操作（比如用什么动画效果展示一个数据）；VM里面是和View相关的数据部分的储存和操作，比如说一个UITableView的data source，一个对email输入合法性的验证方法，当然还有的是对真正Model层的调用和结果的刷新，由于View已经和VM绑定，这样VM在刷新的时候只刷新自己的属性就得了。

　　其实重要的还是写代码思维方式的变化，如果全工程都使用RAC来实现，对于同一个业务逻辑终于可以在同一块代码里完成了，将UI事件，逻辑处理，文件或数据库操作，异步网络请求，UI结果显示，这一大套统统用函数式编程的思路嵌套起来，进入页面时搭建好这所有的关系，用户点击后妥妥的等着这一套联系一个个的按期望的逻辑和次序触发，最后显示给用户。感觉就像是搭好了一个精致的游乐场，然后不紧不慢地打开大门：@"Come on 熊孩子们！"

 

PS：写这个blog的时候用的RAC版本是2.2.3，现在RAC3.0-dev正开发中，还会有很多变化，可能没写完就出新了，需要修改的我会update。