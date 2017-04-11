title: Reactive Cocoa Tutorial [1] = 神奇的Macros
date: 2014-03-06 22:11:04
tags: Reactive Cocoa Tutorial
---
Reactive Cocoa Tutorial 系列，转载请注明该文源地址 -- by sunnyxx

---

##先说说RAC中必须要知道的宏：
```
RAC(TARGET, [KEYPATH, [NIL_VALUE]])
```
使用：
```
RAC(self.outputLabel, text) = self.inputTextField.rac_textSignal;

RAC(self.outputLabel, text, @"收到nil时就显示我") = self.inputTextField.rac_textSignal;
```
　　这个宏是最常用的，`RAC()`总是出现在等号左边，等号右边是一个`RACSignal`，表示的意义是将一个对象的一个`属性`和一个`signal`绑定，signal每产生一个value（id类型），都会自动执行：
```
[TARGET setValue:value ?: NIL_VALUE forKeyPath:KEYPATH];
```
　　数字值会升级为`NSNumber *`，当setValue:forKeyPath时会自动降级成基本类型（int, float ,BOOL等），所以RAC绑定一个基本类型的值是没有问题的

<!--more-->
 
```
　　· RACObserve(TARGET, KEYPATH)
```
　　作用是观察TARGET的KEYPATH属性，相当于`KVO`，产生一个`RACSignal`

　　最常用的使用，和RAC宏绑定属性：
```
RAC(self.outputLabel, text) = RACObserve(self.model, name);
```
　　上面的代码将label的输出和model的name属性绑定，实现联动，name但凡有变化都会使得label输出

```
@weakify(Obj);
@strongify(Obj);
```
　　这对宏在 `RACEXTScope.h` 中定义，RACFramework好像没有默认引入，需要单独import

　　**他们的作用主要是在block内部管理对self的引用**：
```
@weakify(self); // 定义了一个__weak的self_weak_变量
[RACObserve(self, name) subscribeNext:^(NSString *name) {
    @strongify(self); // 局域定义了一个__strong的self指针指向self_weak
    self.outputLabel.text = name;
}];
```
　　这个宏为什么这么吊，前面加@，其实就是一个啥都没干的@autoreleasepool {}前面的那个@，为了显眼罢了。

　　**这两个宏一定成对出现，先weak再strong**

 

##除了RAC中常用宏的使用，有一些宏的实现方法也很值得观摩。 

 

　　举个高级点的栗子：

　　要干的一件事，**计算一个可变参数列表的长度**。

　　第一反应就是用参数列表的api，`va_start` `va_arg` `va_end`遍历一遍计算个和，但仔细想想，对于可变参数这个事，在**编译前**其实就已经确定了，代码里括号里有多少个参数一目了然。

　　RAC中`Racmetamarcos.h`中就有一系列宏来完成这件事，硬是在预处理之后就拿到了可变参数个数：
```
#define metamacro_argcount(...) \
    metamacro_at(20, __VA_ARGS__, 20, 19, 18, 17, 16, 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1)
```
这个宏由几个工具宏一层层展开，现在模拟一下展开过程：

假如我们要计算的如下：
```
int count = metamacro_argcount(a, b, c);
```
于是乎**第一层**展开后：
```
int count = metamacro_at(20, a, b, c, 20, 19, 18, 17, 16, 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1)
```
再看metamacro_at的定义：
```
#define metamacro_at(N, ...) metamacro_concat(metamacro_at, N)(__VA_ARGS__)
// 下面是metamacro_concat做的事（简写一层）
#define metamacro_concat_(A, B) A ## B
```
于是乎**第二层**展开后：
```
int count = metamacro_at20(a, b, c, 20, 19, 18, 17, 16, 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1);
```
再看metamacro_at20这个宏干的事儿：
```
#define metamacro_at20(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, _18, _19, ...) metamacro_head(__VA_ARGS__)
```
于是乎**第三层**展开后，相当于截断了前20个参数，留下剩下几个：
```
int count = metamacro_head(3, 2, 1);
```
这个metamacro_head：
```
#define metamacro_head(...) metamacro_head_(__VA_ARGS__, 0)
#define metamacro_head_(FIRST, ...) FIRST
```
　　后面加个0，然后取参数列表第一个，于是乎：
```
int count = 3;
```
　　**大功告成。**

　　反正我看完之后感觉挺震惊，宏还能这么用，这样带来的好处不止是将计算在预处理时搞定，不拖延到运行时恶心cpu；但更重要的是编译检查。比如某些可变参数的实现要求可以填2个参数，可以填3个参数，其他的都不行，这样，也只有这样的宏的实现，才能在编译前就确定了错误。

##除了上面，还有一个神奇的宏的使用：

　　当使用诸如`RAC(self, outputLabel)`或`RACObserve(self, name)`时，发现写完逗号之后，**输入第二个property的时候会出现完全正确的代码提示**！这相当神奇。
![自动代码提示][1]


探究一下，关键的关键是如下一个宏：
```
#define keypath(...) \
    metamacro_if_eq(1, metamacro_argcount(__VA_ARGS__))(keypath1(__VA_ARGS__))(keypath2(__VA_ARGS__))
```
这个`metamacro_argcount`上面说过，是计算**可变参数**个数，所以`metamacro_if_eq`的作用就是判断参数个数，如果个数是1就执行后面的keypath1，若不是1就执行keypath2。

所以重点说一下keypath2：
```
#define keypath2(OBJ, PATH) \
    (((void)(NO && ((void)OBJ.PATH, NO)), # PATH))
```
　　乍一看真挺懵，先化简，由于Objc里面keypath是诸如"outputLabel.text"的字符串，所以这个宏的返回值应该是个字符串，可以简化成：
```
#define keypath2(OBJ, PATH) (???????, # PATH)
```
先不管"??????"是啥，这里不得不说C语言中一个不大常见的语法（第一个忽略）：
```
int a = 0, b = 0;
a = 1, b = 2;
int c = (a, b);
```
这些都是**逗号表达式**的合理用法，第三个最不常用了，c将被b赋值，而a是一个未使用的值，编译器会给出warning。

去除warning的方法很简单，强转成void就行了：
```
int c = ((void)a, b);
```
再看上面简化的keypath2宏，返回的就是PATH的字符串字面值了(单#号会将传入值转成字面字符串)

```
(((void)(NO && ((void)OBJ.PATH, NO)), # PATH))
```
对传入的第一个参数OBJ和第二个正要输入的PATH做了`点`操作，这也正是为什么输入第二个参数时编辑器会给出正确的代码提示。强转void就像上面说的去除了warning。

　但至于为什么加入与`NO`做`&&`，我不太能理解，我测试时其实没有时已经完成了功能，可能是作者为了屏蔽某些隐藏的问题吧。

　　这个宏的巧妙的地方就在于使得编译器以为我们要输入“点”出来的属性，保证了输入值的合法性（输了不存在的property直接报错的），同时利用了逗号表达式取逗号最后值的语法返回了正确的keypath。

 

##总之
RAC对宏的使用达到了很高的水平，还有诸如`RACTuplePack`，`RACTupleUnpack`的宏就不细说了，值得研究。

---
PS：上面介绍的metamacro和@strongify等宏确切来说来自RAC依赖的extobjc，作者是Justin Spahr-Summers，正是RAC作者之一。


  [1]: http://images.cnitblog.com/blog/401798/201402/112147518936541.png