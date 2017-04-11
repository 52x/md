title: Reactive Cocoa Tutorial [2] = 百变RACStream
date: 2014-03-06 22:34:27
tags: Reactive Cocoa Tutorial
---
Reactive Cocoa Tutorial 系列，转载请注明该文源地址 -- by sunny


##Overview

　　在RAC下开发干的最多的事就是建立RACSignal和subscribe RACSignal了，它是RAC的核心所在。本篇介绍了RAC的运作原理和设计思路，从函数式编程形成的RACStream继而介绍它的子类 - RAC最核心的部分RACSignal。

##函数式编程

　　我们知道Reactive Cocoa是函数式编程(Functional Programing)(FP)思想的实现。FP有一套成熟的理论，这里只讲讲我个人理解吧。

　　我觉得FP就是“**像计算函数表达式一样来解决一个问题**”，举个栗子，中学题：

<!--more-->

```
已知：f(x) = 2sin(x + π/2)， 求 f(π/2)的值。
```
　　其中x是这个函数的输入，f(x)为计算的输出结果，求f(π/2)时给定了x自然能计算出个结果来（说实话我真忘了咋算了）

当然，仔细看这个函数，其实是可以分解成几个小函数的：

```
f1(x) = x + π/2
f2(x) = sin(x)
f3(x) = 2x
```
　　而原来的f(x)可以被小函数组合：
```
f(x) = f3(f2(f1(x)))
```
　　所以不难得出这么个推论：要是我手上有足够的**基本函数**，我就能用上面的组合的方法组合出任意一个**复杂的函数**了。再想想事实上这些年来学数学的过程不就是在一个个积累基本函数的过程嘛，从基本运算，到三角函数，到乘方开方，再到微积分。基本函数越来越多，能解决的数学问题也越来越复杂。

　　再来看一个函数是怎么构成的，FP理论里叫`monads`，十分抽象，没读懂，但能理解出来：一个函数只要有一个对于输入值的运算方法和一个返回值，就够了。也容易理解，给它一个输入，干点事情，给出一个输出，就行了，当然现实情况要复杂得多（比如说输出值本身就是个函数？）有些函数是有输入的条件的，比如原来数学解个函数时候经常跟个作用域或者限制条件，比如`f(x) = 10 / x , (x不为0)`，要是传个0这个函数就认为计算错误。

　　对于像上面栗子的函数，每个函数都能接收上一个函数输出的结果，作为自己的输入，这样才能嵌套生成最终结果，同时，计算的顺序也是一定从里向外，所以换个写法可以写成：

```
start ---x--> f1(x) --(temp value1)--> f2(temp value1) --(temp value2)--> f3(temp value2) ---> result
```
　　于是乎**嵌套**就被表示成了**序列**，来个高大上的名字怎么样，就叫**流（Stream）**

##RACStream

　　这就是`RACStream`所表示的含义。

　　按照上面说的，其实`RACStream`的名字有点点歧义，对于一个`RACStream`对象，它在意义上等同于上面的f1(x),f2(x),f3(x)，而不是那一大串整体，表示整体的应该是最外层的和f(x)对应的那个对象，叫个RACStreamComponent比较好？理解时候得注意下。

　　所以作为一个基本函数的RACStream应该至少应该有：

 1. 怎么传入值
 2. 怎么返回值
 3. 怎么与其他函数组合
 4. 怎么实现函数的作用域(监测输入值来做处理)
 5. 这函数叫啥- -


得益于在Objc下实现，所以输入输出的“值”都用个`id`类型就行了，遇到多个值的组合就用`RACTurple`（可以把多个值压包和解包，类比WINRAR），1和2解决

RACStream从实例变量来看只有一个`name`，当然它也只应该有个name - -，5解决

　　里面重点问题就是上面的3和4了。由于**函数组合之后仍然是个函数**，所以也很容易理解**两个Stream对象的组合其实就是生成一个新的Stream对**象，它返回了分别由两个子Stream先后运算产生的最终结果

 　　观摩一下RACStream定义的基本方法：
```
+ (instancetype)empty;
+ (instancetype)return:(id)value;
- (instancetype)bind:(RACStreamBindBlock (^)(void))block; // for 4
- (instancetype)concat:(RACStream *)stream; // for 3
- (instancetype)zipWith:(RACStream *)stream; // for 3
```
　　RACStream作为一个描述抽象的父类，这几个基本方法并没有实现，是由具体子类来实现，RACStream的两个子类分别是`RACSignal`和`RACSequence`

　　

 - `+empty` 是一个不返回值，立刻结束(Completed)的函数，意思是执行它之后除了立刻结束啥都不会发生，可以理解为RAC里面的nil。



 - `+return:` 是一个直接返回给定值，然后立刻结束的函数，比如 f(x) = 213

 - `-bind:`是一个非常重要的函数，在Rac Doc中被描述为‘**basic primitives, particularly**’，它是RACStream监测“值”和控制“运行状态”的基本方法，个人认为看注释文档不能理解它是干嘛的，而且bind英语“捆绑，绑定，强迫，约束”这几个意思也感觉对不上，我觉得叫“**绑架**”倒是更贴切一点。在-bind：之后，之前的RACStream就处于被“绑架”的状态，被绑架的RACStream每产生一个值，都要经过“绑架者”来决定：

1. 是否使这个RACStream结束（被绑架者是否还能继续活着）

2. 用什么新的RACStream来替换被绑架的RACStream，传出的结果也成了新RACStream产生的值（绑匪可以选择再抓一个人质放之前那个前面）

 　　举个具体栗子，RACStream的 - take：方法，这个方法使一个RACStream只取前N次的值（有缩减）：

```
- (instancetype)take:(NSUInteger)count {
    Class class = self.class;
    
    return [[self bind:^{ // self被绑架
        __block NSUInteger taken = 0;

        return ^ id (id value, BOOL *stop) { // 这个block在被绑架的self每输出一个值得时候触发
            RACStream *result = class.empty;

            if (taken < count) result = [class return:value]; // 未达到N次时将原值原原本本的传递出去
            if (++taken >= count) *stop = YES; // 达到第N次值后干掉了被绑架的self

            return result; // 将被绑架的self替换为result
        };
    }]];
}
```
 　　`-concat:` 和 `-zipWith:` 就是将两个RACStream连接起来的基本方法了：

`[A concat:B]`中A和B像`皇上`和`太子`的关系，A是皇上，B是太子。皇上健在的时候统治天下发号施令（value），太子就候着，不发号施令（value），当皇上挂了（completed），太子登基当皇上，此时发出的号令（value）是太子的。
`[C zipWith:D]`可以比喻成一对`平等恩爱的夫妻`，两个人是“绑在一起“的关系来组成一个家庭，决定一件事（value）时必须两个人都提出意见（当且仅当C和D同时都产生了值的时候，一个value才被输出，CD只有其中一个有值时会挂起等待另一个的值，所以输出都是一对值（RACTuple）），当夫妻只要一个人先挂了（completed）这个家庭（组合起来的RACStream）就宣布解散（也就是无法凑成一对输出时就终止）
##然后呢？

　　除了上面几个基本方法，RACStream还有不少的Operation方法，这些操作方法的实现大都是组合基本的方法来达到特定的目的，虽然是RACStream这个基类实现的，但我觉得还是放在后面介绍RACSignal的时候作为它的使用方法来说比较合适，毕竟绝大多数编程的对象的都是RACStream的两个子类，后面再展开介绍好了。

