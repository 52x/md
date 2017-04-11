title: R如何找对象？ - PART II
date: 2016-01-24 10:26:28
categories: Codage|编程
tags: [R, 译文, environment]
---

上一部分(你以为我想拆成两部分啊，太长了啊 o(╯□╰)o)讲到了

* 环境(空环境、全局环境、封装环境、包的关联环境、`namespace:base`)
* R的检索路线
* 依赖项和导入项的区别

<!-- more -->

## 紫色虚线
跟其它对象一样，函数也住在环境中。而函数自身也有个**指针属性**，指向它们可以运行的环境。当你创建一个函数时，这个属性被自动设定为指向函数创建的环境。所以函数创建和**将要**运行的环境，是一样儿一样儿的。

![img1](http://7xndoy.com1.z0.glb.clouddn.com/Codage-20-img1-environment-property.png)

“函数将要运行的环境”是什么意思呢？
上一部分曾经提到，函数一旦运行就会创建一个其特有的环境。我们也提到过每个环境都有一个封装环境。那么是哪个环境封装了函数的新环境呢？
这个封装环境被函数的**环境属性**所指定。这个环境也是“函数将要运行的环境”。它*并不一定*是拥有函数的环境。它被函数的环境属性控制。

我们可以通过`environment()`来获得函数对象的环境属性(回想一下，第一部分我们提到`environment()`只以函数作为输入参数)。

``` r
MyFunction <- function() {} 
environment(MyFunction) 
```

    ## <environment: R_GlobalEnv>

运行`MyFunction()`时，R在执行函数定义的`{}`中的语句，环境则如下图所示：

![img2](http://7xndoy.com1.z0.glb.clouddn.com/Codage-20-img2-environment-execution.png)

默认情况下，R将函数的环境属性设置为函数被创建的环境(拥有此函数的环境)。但函数的执行环境和拥有函数的环境并不一定非得是一样的。要不，换一个环境试试看？

``` r
MyFunction <- function() { } 
newEnvironment <- new.env() 
environment(MyFunction) <- newEnvironment 
environment(MyFunction) 
```

    ## <environment: 0x000000000e895628>

上面的代码执行以后，`environment(MyFunction)`已经不再返回`R_GlobalEnv`了。

另一种查看函数的环境属性的方法是直接打印这个函数。

``` r
MyFunction
```

    ## function() { } 
    ## <environment: 0x000000000e895628>

对函数`sd()`做同样的操作：

``` r
environment(sd) 
```

    ## <environment: namespace:stats> 

``` r
sd
```

    ## function (x, na.rm = FALSE) 
    ## { 
    ##   ... (removed for brevity) 
    ## } 
    ## <bytecode: 000000000E7F2EA0>
    ## <environment: namespace:stats>

再来看下面这段代码：

``` r
age <- 32 

MyFunction <- function(){ 
    age <- 22 
    FromLocal <- function() {print(age + 1)} 
    FromGlobal <- function() {print(age + 1)} 
    NoSearch <-  function() {age <- 11; print(age + 1)} 
    environment(FromGlobal) <- .GlobalEnv 
    FromLocal() 
    FromGlobal() 
    NoSearch()
} 

MyFunction() 
```

    ## [1] 23 
    ## [1] 33 
    ## [1] 12

这段代码是如何运作的？
1. `FromLocal()`的封装环境是`MyFunciton()`(内部)的环境，也就是`FromLocal()`被创建的环境，这也是R以默认规则执行的结果。
当R查找`FromLocal()`中的`age`变量时，(由于`FromLocal()`内并没有此变量的赋值，)它会找到`MyFunction()`的环境中，抓取`age <- 22`然后对这个变量加1；
2. `FromGlobal()`的封装环境是`R_GlobalEnv`因为我们用`environment()`函数将`.GlobalEnv`赋值给了它的环境属性。R在`FromGlobal()`内部查找不到`age`变量的赋值，便向其封装环境发起查找，抓取`R_GlobalEnv`中的语句`age <- 32`并在此变量基础上加1；
3. `NoSearch()`内部已经有了`age`对象，所以这个函数可以直接在其衍生的内部环境中找到想要的变量而不用向它被封装的环境求助。

这解释了我们地图里的那些紫色虚线。
如果你在`package:<name>`环境中检视函数的环境属性，你会看到他们全部指向`namespace:<name>`环境。

让我们来验证一下：
获取`package:stats`中的标准差函数并查看它的环境属性。

``` r
statsPackageEnv <- as.environment("package:stats") 
sdFunc <- get("sd", envir = statsPackageEnv) 
environment(sdFunc) 
```

    ## <environment: namespace:stats> 

``` r
statsNamespaceEnv <- environment(sdFunc) 
sdFunc2 <- get("sd", envir = statsNamespaceEnv) 
environment(sdFunc2) 
```

    ## <environment: namespace:stats> 

注意到`sd()`的环境属性指向`namespace:stats`。

这里也有更简单的办法获取一个命名空间的环境。

``` r
statsNamespaceEnv <- asNamespace("stats") 
statsNamespaceEnv 
```

    ## <environment: namespace:stats> 

所以，本质上讲，包的环境只是通往命名空间环境的通道而已。
包的环境会说“我不知道做啥，问我的函数”。
当我们问函数时，他们会说“当你执行我们的时候，你创建了一个新的环境，它的封装环境就是命名空间的环境。”
更准确地讲，函数只是贡献出了它们的环境属性。
我们也可以让这些虚线变成实线：

![img3](http://7xndoy.com1.z0.glb.clouddn.com/Codage-20-img3-map-of-the-world-complete.png)

碰巧这是另一种对“为什么对一个对象查询拥有它的环境没有简便方法”的解释。
当我们在一个环境中执行语句的时候，我们会关注它所拥有的对象，因为我们可能会用到其中的一个。当我们找到一个函数，我们也应当知道它应该在哪个环境下执行。
但在我们的工作流程中，并不是每一个对象我们都应当判定拥有它的环境是什么(并不是那么重要)。

如果你现在已经晕晕乎乎了，我建议你停下来，重新读一遍这一小节。函数的执行，是整块拼图里最复杂的一个碎片。

## 传递函数
(可选择直接跳过此小节)

因为函数有一个环境属性，所以它们可以被传递。将一个函数传递给另一个函数，是一个可以让你抓耳挠腮(也很强大)的特征。我们就不在这条道上黑太远了。
从一个较高的层面，你可以这样思考：
如果一个函数`FunctionA(someOtherFunction)`以另一个函数`someOtherFunction()`做参数，那么`FunctionA()`在它运行的方式上一定跟其它函数有些不同。这种不同被执行`someOtherFunction()`的过程所控制。我们编写`someOtherFunction()`的时候就期望它以一种特殊的方式运行。它应该有能力获取其被创建的环境内的所有对象，即使当它被移交给`FunctionA()`之后，这种能力仍然保留。
R虽然为`FunctionA()`创建了一个新环境，但这并不能妨碍以上功能的实现。
当`someOtherFunction()`最终被运行的时候，R检索此函数的环境属性，并在此环境中执行，而并非在`FunctionA()`的内部环境中运行。所以之前提到的能力仍能够被支持。
事实上，`FunctionA()`也能将`someOtherFunction()`传递给`FunctionB()`，后者也能将其传递给`FunctionC()`且`FunctionC()`不与`someOtherFunction()`的运行结果有直接联系。这是有关函数环境属性的魔法。

## 别理那个调用者(caller)
我们讨论到的查找机制不会用到调用栈。
调用栈是从代码开始运行到你当前所在的计算所涉及到的函数调用的序列。比如，`FunctionA()`调用`FunctionB()`，后者调用`FunctionC()`。那么调用栈就依调用的先后顺序将一个函数置于另一个之上。
思考查找机制的问题时，错误的思路就是顺着调用的顺序找。
假设`FunctionC()`会执行`FunctionD()`。按照调用的思路，如果`FunctionD()`不是在`FunctionC()`的执行环境中定义的，那么就需要在`FunctionB()`的执行环境中查找，然后是`FuctionA()`。
正确的方法应该是问自己“谁拥有`FunctionC`？”如果拥有者(owner)对`FunctionD()`一无所知，那么有可能拥有者的拥有者(owner's owner)知道，以此类推。

问题是，直觉上我们更倾向按调用栈(调用链)的思路去思考，而不是依照封装环境的链条。
需要记住的是，无论R何时估算一行代码，系统总是站在两条重要的环境链的顶端(也可能是底端，顶端易脑补)。一条是封装环境链，它涉及到一个范围问题，比如，如果当前环境的框架下找不到特定的变量名，接下来应该去哪里查找。这是我们关心的链条。另一条是调用栈，它是在一系列的函数调用中产生的。你可以忽略掉这个链条。只有在用到一些特殊的R函数时，才有必要通过这个链条去找一个变量。这些应用场景不在本文的讨论范围内。

**注意**：R(或一些R的读物)在两种链条的文档中都用到“母系”(parent)这个词条。`parent.env()`函数是我们已经谈过的用来寻找封装环境的函数，而`parent.frame()`函数用于查询调用栈。这种用词和函数命名肯定会让人疑惑而且也是一个历史的疏忽。“母系”一次不应被用作封装环境的替代品，而只应用在调用栈上。

## 终于，R怎么找对象呢？
R只是顺着那条紫色的公路找呀~
要不我们再看一个例子好了。

假设我们在找`ggplot()`函数。我们从`R_GlobalEnv`找起。如果`ggplot()`不在全局环境中，那它一定在一个包里。所以R遍历查询列表试图找到这个函数。这便是一个直观的封装环境链。最终R在一个包的环境中找到这个函数。尽管`ggplot()`是在包的环境(package environment)中找到的，R仍然在命名空间的环境(namespace environment)中执行这个函数，一如我们在上一节中讲到的一样。

如果此时`ggplot()`调用了另一个函数`MyFunction()`，则会发生以下事情：
1. 如果`MyFunction()`是在`ggplot()`内定义的，那么我们马上能够找到它，因为R首先在本地环境(local environment)中查找。这里，本地环境是运行`ggplot()`时创建的环境。
2. 如果没有找到，那么R会去`MyFunction()`的执行环境的封装环境(`namespace:ggplot2`)中查找。如果我们能找到`MyFunction()`，那么这种情况是同一个包里的一个函数调用另一个函数。
3. 如果`namespace:ggplot2`里也找不到`MyFunction()`，那么R会检查命名空间的环境(namespace environment)的封装环境，也就是导入项的环境(imports environment)。这样`ggplot()`就有机会在一些明确要求的依赖包里查找`MyFunction()`函数。可参见我们在第一部分提到过的`ggplot2`查找`plyr`中函数的例子。
4. 如果仍不能找到，那么我们就要去导入项环境的封装环境中查找，也就是`namespace:base`。你可以在这里找到基础函数(如`sd()`)，然后结束查找。
5. 如果`namespace:base`中也没有`MyFunction()`，那我们就要回到查询列表了。我们从`R_GlobalEnv`找起。当然这并不大可能。如果一个包期望用户在全局环境里定义函数，那使用这个包一定是个很差的体验。当然，用户可以以此为机会通过在全局环境里定义自己的`MyFunction()`函数来打断查找路线。
6. 如果`MyFunction()`存在于`ggplot2`的依赖包中，且这个依赖包存在于帮助文档中`Depends`项，而不是`Imports`项，那么我们能在查找列表中找到`MyFunction()`。此情况类似于前文提到的`ggplot()`寻找`reshape`包中的函数的例子。我们也要寄希望于没有其它的包以同样的名字定义了一个函数，这个包被加载的位置也不能太靠近全局环境(`reshape2`的案例)。

总之你只需要知道什么是*当前*或*本地*环境，然后顺着封装环境链一直查找下去，如此往复。

## 原文
[How R Searches and Finds Stuff](http://blog.obeautifulcode.com/R/How-R-Searches-And-Finds-Stuff/)