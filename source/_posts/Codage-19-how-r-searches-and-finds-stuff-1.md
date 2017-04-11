title: R如何找对象？ - PART I
date: 2016-01-23 23:52:24
categories: Codage|编程
tags: [R, 译文, environment]
---

或者是……
如何将自己推进环境、命名空间、导入、导出、框架、封装、继承和函数调用的巨坑中？

(个人翻译，原文在全文页面底部)

<!-- more -->

## 动机
为何读此贴？

1. 避免被坑
迄今为止你都躲开了上述那些坑，但现在是时候入坑了（避免被坑的前提是熟悉它而不是碰巧躲开它）。不幸的是，你说的是人类语言，但R的帮助文档只说“原始C语言”（脑补一个80年代毛发旺盛的C语言程序员——极其聪明但是说话像含了一嘴腌萝卜一样——并不擅长沟通）。

2. R很傻缺
你的函数一度工作的很好，但现在却甩你一脸报错。当然，你的函数并没有做过任何修改，你只是模糊的记得报错前你装了一个新包，但是那有什么关系呢？可是，的确有关系……

3. R找错对象
你加载了一个包叫[matlab](https://cran.r-project.org/web/packages/matlab/index.html)然后调用`sum()`函数去计算一个矩阵。返回的结果是每列求和后的向量(`base`包里的`colSums()`函数的效果)，而不是一个数值。脑海里奔腾过无数匹神兽。
让R像Matlab一样运行，你想啥呢？

4. 你想让R找点儿别的东西
你喜欢一个包里的绘图函数。如果这个函数能直接被自己的计算调用就完美了。看上去像是黑魔法，但是用函数的完整功能来支持你的小把戏多少有点诡异。那么，暗黑艺术欢迎你。

5. 写包
你写了一个包，你的这个娃跟CRAN里其它的娃玩儿的开心伐？

## R在哪儿存储对象？
R里所有的东西都存在于**环境**中。我们最终想解决的问题是，搞清楚什么对象存放于什么环境中，并且如何调用到它。
一个环境，也是R的一个对象。对象用于存储。环境比较特殊，只能用于存储两种东西：

1. 框架(a frame)
这是一个**命名对象**的集合。什么是命名对象？R里所有的东西都是一个对象——函数、数值、字符、逻辑值，等等。这些对象可以被命名为(比如)`myFunction`，`myNumeric`，`myCharacter`，`myLogical`。当你在命令行输入`myVar = "chirlie"`时，你就创建了一个叫做"myVar"的字符型变量，这个变量里存储了"charlie"这个字符串。
因为所有的对象都存储在环境中，框架就是环境存储这些对象的地方。刚刚你创建的`myVar`就存在于某个环境的框架中。

2. 封装环境(the enclosing environment)
一个环境的所有者即封装环境，它是对另一个环境的引用。
![img1](http://7xndoy.com1.z0.glb.clouddn.com/Codage-19-img1-environment-structure.png)

上图可以看出封装环境的链条止步于一个特殊环境，叫做空环境(empty environment)。你可以通过执行`emptyenv()`来获取这个对象。
对于一个给定的环境对象，你可以对其查询两件重要的事情：此环境的所有者(母环境、封装环境)还有它的框架下的对象。

## 对所有者和指针撒个小谎
笔者使用*所有关系*和*包含关系*两个松散的概念。当我说到*拥有*和*包含*的时候，那么我真的是在讨论*指针*。如果你对*指针*没有了解，那么你可以直接跳到下一个环节。

接下来我会将环境描述为*拥有*对象，尤其是拥有函数。事实上，函数就是存储在内存某处的指令，他们可以在查询表里检索，一旦找到，就会被指针指出内存所在位置。这篇文章的核心概念就是指针的不可知性以及对于精通R的搜索机制而言，指针并不是必须的。
事实上，R很努力的在隐藏指针。
对，技术上的不精确让我很痛苦，但我会尝试着让文章保持简单，毕竟还是有很多人能理解*所有关系*。我们有很多东西要谈。


## 跟环境玩一玩
创建一个环境（环境是个对象）
``` r
myEnvironment = new.env() 

# 打印出来
myEnvironment 
```

    ## <environment: 0x0000000006ce0920>

除空环境(`R_EmptyEnv`)外，任何环境都有个封装环境。`myEnvironment`的封装环境是什么？是`R_GlobalEnv`，用`parent.env()`函数来找到它。

``` r
parent.env(myEnvironment) 
```

    ## <environment: R_GlobalEnv>

`R_GlobalEnv`的封装环境是什么？是一个叫`package:stats`的环境 (以个人电脑里包的安装顺序而定，每个人的可能有差异)。

``` r
parent.env(parent.env(myEnvironment)) 
```

    ## <environment: package:stats>
    ## attr(,"name")
    ## [1] "package:stats"
	## attr(,"path")
    ## [1] "C:/R/R-2.14.1/library/stats"

`R_GlobalEnv`也可以由以下两种标识来提取。
`.GlobalEnv`和函数`globalenv()`。`R_GlobalEnv`我们会稍后讨论。
``` r
parent.env(.GlobalEnv) 
```

    ## <environment: package:stats>
    ## attr(,"name")
    ## [1] "package:stats"
    ## attr(,"path")
    ## [1] "C:/R/R-2.14.1/library/stats"

``` r
parent.env(globalenv())
```

    ## <environment: package:stats>
    ## attr(,"name")
    ## [1] "package:stats"
    ## attr(,"path")
    ## [1] "C:/R/R-2.14.1/library/stats"

用`emptyenv()`获取空环境。

``` r
emptyenv()
```

    ## <environment: R_EmptyEnv>

为什么`myEnvironment`有个恶心的名字：0x0000000006ce0920? 
那个名字只是环境在内存中的位置。我们可以对它的`name`属性(attribute)赋值，让它看起来不那么让人敬而远之。
然而R并不会用这个正常点儿的名字去替代原有的名字。我们可以用`environmentName()`函数去提取那个正常的名字。

``` r
attr(myEnvironment, "name") = "Cool Name" 
myEnvironment 
```

    ## <environment: 0x0000000006ce0920>
    ## attr(,"name")
    ## [1] "Cool Name"
    ## > environmentName(myEnvironment) 
    ## [1] "Cool Name"

然后我们创建一个数值变量。
``` r
myValue = 5
```

创建的对象默认存放在当前的环境下，可用`environment()`函数获取当前环境。

``` r
environment() 
```

    ## <environment: R_GlobalEnv>

用`ls()`可在当前环境下查询所属框架内的所有对象。这里我们确认了`myEnvironment`和`myValue`都存在于当前的环境`R_GlobalEnv`中。

``` r
ls(envir = environment())
```

    ## [1] "myEnvironment" "myValue"      

当然我们也可以无视默认的存储行为，踩在当前环境的脸上把新创建的对象存储在其它环境中。我们用`assign()`函数去实现上述目标——在`myEnvironment`中创建一个叫"myLogical"的变量。
先用`ls()` 确认在赋值前，myEnvironment中没有任何东西。然后再赋值后再次用`ls()`去确认已经成功在`myEnvironment`中创建`myLogical`。

``` r
ls(envir = myEnvironment)
```

    ## character(0)

``` r
assign("myLogical" , c(FALSE, TRUE), envir = myEnvironment)
ls(envir = myEnvironment)
```

    ## [1] "myLogical"

通过调用`get()`函数，我们可以从任何环境中获取任何对象。

``` r
get("myLogical" , envir = myEnvironment)
```

    ## [1] FALSE  TRUE

创建对象之前，我怎么知道`myEnvironment`的封装环境是`R_GlobalEnv`? 
所以再强调一次，R以当前环境为默认存储环境。你可以使用`parent.env()`函数来替换一个环境的封装环境。

``` r
myEnvironment2 = new.env()
parent.env(myEnvironment2)
```

    ## <environment: R_GlobalEnv>

``` r
parent.env(myEnvironment2) = myEnvironment
parent.env(myEnvironment2)
```

    ## <environment: 0x0000000006ce0920>
    ## attr(,"name")
    ## [1] "Cool Name"

从另一个角度去理解“当前”或“本地”环境：我们新建一个函数，让它调用`environment()`去查询本地环境。当R执行一个函数时，它会自动为函数创建一个（子）环境。当变量或者对象在函数内部被创建时，他们只存在于函数内部的环境中。调用`Test()`函数去验证时，并不会返回`R_GlobalEnv`。我们没有在`Test()`内部创建任何对象。如果我们创建了，他们便会存在于`0x0000000006ce9b58`环境中。当函数完成运算，函数内部环境就会消失。

``` r
Test = function() { print(environment()) } 
environment()
```

    ## <environment: R_GlobalEnv>

``` r
Test() 
```

    ## <environment: 0x0000000006ce9b58>

当然我们也可以尝试一下返回函数中子环境的封装环境。
``` r
Test = function() { print(parent.env(environment())) }
Test()
```

    ## <environment: R_GlobalEnv>

## 简短的答案：R如何找对象
有跟着上面的代码运行一遍吗？从运行结果看上去，环境并不只是对象的静态仓库。R运行一段语句时，从事伴随着一个*本地*或*当前*环境。简单点讲，就是一个当前被*激活*的环境（对立面是目前未被使用的*未被激活*的环境。或是更直白地说，语句总是在一个指定的环境内运行的。
也就是说，R可以在任何时间提一个问题(此处有翻译腔)“嘿，伙计，本地环境是什么？”，且R经常这么问。每当它需要找到一个命名变量的时候就需要问一次这个问题。
我们知道每当R运行一个函数他就需要创建一个心的本地环境，所以当我们优雅地跑程序时，函数会调用其它函数，而环境会随之滋生然后湮灭。

脑补一下我们在任意一段语句处冻结系统。当R搜寻那段语句中的一个名称时，它会首先在当前环境中进行查找。如果当前环境中找不到，R跑去其封装环境中找对象，以此类推。这就是R如何找对象的，它从本地环境开始遍历每一级封装环境，一级一级直至找到第一个出现对象名称的环境为止。

这次大家满足了吧？
No No No。我们继续。

## 世界地图
我们刚刚谈到的R找对象的方式，多少有点像一个限定了方向的寻宝游戏。要找到宝藏，我们只需要准备一个世界地图！

![img2](http://7xndoy.com1.z0.glb.clouddn.com/Codage-19-img2-map-of-the-world.png)

第一次启动R，所有环境的状态如上图所示。每一个方框代表了一个唯一的环境，紫色实线代表了封装环境之间的关系。紫色虚线会稍后提到，暂时将它视作一种与封装环境类似的关系。

## 全局环境(the global environment)
`R_GlobalEnv`是一个特殊的环境。在上面的地图中，它是绿色的。绿色表示*开始*。精确地讲，全局环境是你启动R时的*本地*或*当前*环境，如果你在命令行上进行赋值的操作，那么命名对象会被储存在`R_GlobalEnv`中。

`ls()`函数返回指定环境中定义的所有对象。下面的代码中我们用`.GlobalEnv`来指代全局环境。
我们会看到当我们刚刚启动R时，全局环境下没有任何对象。但是当我们给`myVariable`赋值后，全局环境中便包含了这个变量的名字。

``` r
ls(envir = .GlobalEnv) 
```

    ## character(0)

``` r
myVariable = 0 
ls(envir = .GlobalEnv)
```

    ## [1] "myVariable"

注意`environment()`函数。
当你看到它返回`NULL`的时候可能以为这是一个错误的结果。(然而并不是。)
查一下帮助文档，你会知道这个`environment()`只以函数为输入参数。
`myVariable`是一个数值变量，不是函数。
`environment()`的目的并不是告诉你一个对象的封装环境是什么。

``` r
environment(myVariable)
```

    ## NULL

## 搜索列表(the search list)
搜索列表是从`R_GlobalEnv`到`R_EmptyEnv`的封装环境链条。在上面的世界地图里，你可以将它视作一条高速公路。R从`R_GlobalEnv`出发在这条路上行驶。所有的道路都最终汇集到这条高速公路上。你可以在命令行输入`search()`从而得到这个链条。

``` r
search
```

    ## [1] ".GlobalEnv"        "package:stats"   "package:graphics" "package:utils" "package:datasets" 
    ## [6] "package:grDevices" "package:methods" "Autoloads"        "package:base"

## 包 vs. 命名空间 vs. 导入项 vs. 环境
package vs. namespace vs. imports vs. environment

凝视我们那张世界地图一会儿，你会发现一个美女从图中显现……
阿不，这不是3d图……
你会注意到每个R的包都有3个关联的环境。如果你觉得这让人很疑惑，那么恭喜你，你还是正常人类。我第一次碰到这个关联的时候，差点儿没被它逼疯。
不过相信我，唯一吊诡的地方就是命名规则，这三个关联非常有用且设计良好。

![img3](http://7xndoy.com1.z0.glb.clouddn.com/Codage-19-img3-package-namespace-imports.png)

让我们从左到右分解：
1. 包的环境(package environment)
包内*导出*的对象存储在此环境中。简单来讲，这些对象是包的作者希望你看到的，且它们大多数都是函数。一个典型的已经发布的包会提供特定主题或者领域相关的函数。就传统的OOP(Object Oriented Programming)而言，这与"public"类型或方法功能相近。

2. 命名空间的环境(namespace environment)
包内*所有*的对象均存储在此环境中。其中也包括那些包的作者并不想让终端用户接触到的“隐藏”对象(并不是真正的被隐藏，如果你想获取，仍然可以接触到)。这些对象用于支持那些“可见”的对象。
比如，`HardCalculation()`函数可能会使用到`MakeResultsPretty()`的处理复杂的文本格式的功能。但是作者并不想让你直接调用后者，后者唯一的功能就是给`HardCalculation()`傲娇的输出结果赋予特定的格式。这与OOP中的"private"或"internal"的类型或方法功能相近。
等等，包的作者想让我看见的对象，既存在与包的环境又存在于命名空间的环境中？
是又不是。
是，两个环境均有一个框架列举了有同样名字的对象；不是，并同时不存在两个同样的对象集合。两个环境都有一个[**指向同一函数的指针**](http://stackoverflow.com/questions/9278715/value-reference-equality-for-same-named-function-in-package-namespace-environmen)。如果你看不懂，那就姑且认为有两份同样的对象集合好了。这个安排看上去很诡异，但是这种安排的用处马上就会显现出来。这也是为什么查询一个对象所处的环境并不是那么容易——有可能拥有同一对象的环境有两个甚至两个以上。

3. 导入项环境(imports environment)
这个环境存储着从其它包中导入的对象，这些包用于支持当前包的正常使用。在CRAN上发布的包大多不是孤立的，它们基于其它包提供的功能编写而成。以[`ggplot2`](http://cran.r-project.org/web/packages/ggplot2/index.html)为例，在CRAN的页面上，"Imports"区块中这个包要求`plyr`及其它包(原文截图于2012年)。那么`imports::ggplot2`环境便包含了`plyr`包中所有的对象。
![img4](http://7xndoy.com1.z0.glb.clouddn.com/Codage-19-img4-ggplot2.png)

## 导入项(imports) vs. 依赖项(depends)
上面的截图里，`Depends`和`Imports`很让人疑惑。如果`Imports`声明了一个包对其它包的需求，那`Depends`是用来干啥的？这是一个糟糕的命名规则。`Depends`*同样*也列举了`ggplot2`需要的包。两者的差别是我们在地图上的何处放置这些需求。地图限定了R寻找对象的路径，所以在`Imports`或者`Depends`中指定需求，对R寻找依赖包来说有不同的结果。

如果依赖包是在`Imports`中指定，那么这个包中的内容将会存储于“导入项”环境中。在`ggplot2`的例子中，`plyr`的对象会出现在`imports:ggplot2`环境中。要注意的是`plyr`并没有生成一个独立的环境，它漂亮的将自己隐藏在了`imports::ggplot2`环境中。下图紫色的虚线我们会在后面提到。

![img5](http://7xndoy.com1.z0.glb.clouddn.com/Codage-19-img5-imports-depends-plyr.png)

如果依赖包是在`Depends`种指定(比如`reshape`包)，那么这个包的载入方式就像你在命令行输入`library()`或`require()`的效果一样。也就是说，依赖包的包、命名空间以及导入项这三个环境都会被创建。`reshape`包在`ggplot2`前被加载，`package:reshape`环境变成`package:ggplot2`的封装环境。

![img6](http://7xndoy.com1.z0.glb.clouddn.com/Codage-19-img6-imports-depends-reshape.png)

那么我们可以在`Depends`和`Imports`间任意选择吗？No No No...
`library()`命令(或者更广泛的说加载一个库)将包的环境置于`R_Global`之下。更精确地说，加载包的环境变成`R_Global`的封装环境。`R_Global`的旧有封装环境现在封装加载包的环境。下图中你能看到这点。下图中的[`reshape2`](http://cran.r-project.org/web/packages/reshape2/index.html)是原有的`reshape`包的重写/升级版。

`reshape`和`reshape2`均包含了`cast()`函数。我们假设`ggplot2`有一个函数(其实并没有)叫`FunctionThatCallsCast()`。你可以猜到，这个函数能调用`cast()`函数。不用深究R怎么去找对象，我们仅仅需要跟着那条“紫色线路”走。我们从1走到2并找到`FunctionThatCallsCast()`函数。
提醒：包的环境和命名空间的环境都引用了一个公共函数。
运行它就需要找到`cast()`函数，我们从3走到5最后在6找到了它并停止检索。但这并不是我们想要寻找的那个`cast()`。这是`reshape2`的`cast()`，但`ggplot2`需要的是`reshape`包里的那个。这种错误的调用可能会导致很可怕的后果。

![img7](http://7xndoy.com1.z0.glb.clouddn.com/Codage-19-img7-reshape-reshape2-cast-conflict.png)

更好的解决方案是遵循`Imports`特征，直接在`imports:ggplot2`中找到`reshape`的`cast()`，那样的话，我们只要走到3就可以停下。现在，你应该明白了为什么`Imports`和`Depends`间的选择并不是随意的。
由于CRAN上已经有太多的包，不同包的函数拥有同样名字的情况已经司空见惯。`Depends`现在看起来已经不那么安全，它让自己的包更容易被其它加载的包所攻击(同名函数被覆盖)。
译者注：下图是2016年1月23日的`ggplot2`包的截图，可以看到帮助文档里已经取消了`Depends`项中除R本身以外所有的依赖项。

![img4bis](http://7xndoy.com1.z0.glb.clouddn.com/Codage-19-img4bis.PNG)

## namespace:base
还有一个事实就是，所有的`imports:<name>`环境均以`namespace:base`作为其封装环境，就把它当做是创建一个包的赠品吧。由于基础函数被调用的频率最高，`base`包被绝大多数包设置为`Depends`或者`Imports`。没有`namespace:base`的话，R可能要走很远的路去找到`package:base`。
基础函数跟其它包中重名的几率也很大，而包的作者既不可能预先知道谁会载入他的包，也不可能知道你决定写一个你自己的基础函数，所以，包的作者都希望R能在`Imports`后马上找到基础函数，不允许有任何误用。

（未完待续）


## 原文
[How R Searches and Finds Stuff](http://blog.obeautifulcode.com/R/How-R-Searches-And-Finds-Stuff/)