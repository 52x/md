---
title: swift 之循环语句和闭包
date: 2017-03-08 17:17:00
tags: closure
categories: swift学习笔记
toc: true
---
swift简介: 苹果于2014年WWDC（苹果开发者大会）发布的新开发语言，可与Objective-C共同运行于Mac OS和iOS平台，用于搭建基于苹果平台的应用程序。2015年12月4日，苹果公司宣布其Swift编程语言现在开放源代码。
<!--more-->
## 1.流程控制

swift使用三种语句控制流程：`for-in`、`for`、`switch-case`、`while`和`repeat-while`，且判断条件的括号可以省略

* `for-in` 循环

```
let names = ["Jack", "Rose", "Mike", "Puppy"]
for name in names {
print("Hello, \(name)!")
}

//如果不需要使用到迭代的值，使用下划线`_`忽略该值
for _ in 1...10
print("hello")
```
* `if` 语句

原来在oc中,if后的条件语句可以是这样:

```
假设judge是一个BOOL型或NSString或NSArray或NSDictionary等类型
if (judge){
//write code...
}
```

但在swift中,if后的条件返回值必须是Bool型,在写成上面的那种情形就会报错....  所以在swift中这样写:

```
if judge == true {
//write code...
}
```

在Swift2.0以后，不支持`do-while`语句，使用`repeat-whil`e代替，用法与`do-while`一样:

```
repeat {  
print("repeat while : \(j)")  
j++  
} while j < 3
```

* `guard-else` 保镖模式

在执行操作前，进行检查，如果不符合，则拦截，使用方式与if有些类似，如果与let结合使用，可以对可选类型解包，先看看普通的`if-else`模式:

```
func test(i: Int?) {
if let i = i where i > 0 {
// 符合条件的处理
return
}

// 不符合条件的处理
}
```

上面的处理把条件放在了条件判断内部，使用guard与之相反，把正确的情况放在最外部，而异常情况放在条件判断内部:

```
func test(i: Int?) {
guard let i = i where i > 0 else {
// 在这里拦截，处理不符合条件的情况
return
}

// 符合条件的处理，这个时候已经对i进行了拆包，i是非可选类型，可以直接使用
print(i)
}
```

保镖模式可以避免代码中过多的流程判断代码导致过多的代码块嵌套，增强可读性!!!

**保镖模式`guard-else`内的代码块必须包含`break`, `return`等跳出代码块的关键字**

* `switch-case`

swift中的`switch`语句最明显的区别于oc的地方在于,不需要break

## 2. 函数(方法)

### 1.基本形式

```

///单个返回值
func  函数名称(参数1: 参数1类型, 参数2: 参数2类型) -> 返回值 {
//函数体
}

///多个返回值(元组)
func  函数名称(参数1: 参数1类型, 参数2: 参数2类型) -> (x: String, y: Int) {
//函数体
return ("abc", 3)
}


///无返回值
func  函数名 {

}

```

还有一种相当于oc中的加号方法:

```
class func 函数名(参数: 参数类型) -> 返回值类型 {

}
```

### 2.闭包

```
///闭包函数声明形式:
{ (parameters) -> returnType in
statements      // 可以有多行
}
```

闭包函数

```
//定义一个函数变量
var addfunc: (Int, Int) -> Int

//闭包的写法
// 1. 完整写法
addfunc = {(a: Int, b: Int) -> (Int) in
//var c = a + 1       //函数体可以有多条语句，如果在同一行，需要用分号隔开，函数体不需要大括号
return a + b
}
// 2. 前面的addfunc变量可以推断出后面函数的参数类型和返回值类型，故可以省略
addfunc = {(a, b) in return a + b}

// 3. 参数列表括号可以省去，函数只有一条语句时，return可以省略
addfunc = {a, b in a + b}

// 4. 参数和in可以省去，通过$和索引取得参数
addfunc = {$0 + $1}

// 操作符需要的参数与函数参数一致，可以省去参数，并使用括号括起来，作为参数时，可不用括号
addfunc = (+)
```

### 3.Trailing(尾行)闭包

如果函数作为另一个函数的参数，并且是最后一个参数时，可以通过`Trainling`闭包来增强函数的可读性

```
func someFunctionThatTakesAClosure(a: Int, closure: () -> ()) {
// 函数体部分
}

// 1. 一般形式
someFunctionThatTakesAClosure(10, closure: {
// 闭包主体部分
})

// 2. Trainling闭包的方式
someFunctionThatTakesAClosure(10) {
// 闭包主体部分
}

// 3. 如果没有其他参数时，可以省略括号
someFunctionThatTakesAClosure {
// 闭包主体部分
}
```

### 4.Escaping（逃逸）闭包

如果一个闭包/函数作为参数传给另外一个函数，但这个闭包在传入函数返回之后才会执行，就称该闭包在函数中"逃逸"，需要在函数参数添加`@escaping`声明，来声明该闭包/函数允许从函数中"逃逸"，如下:

```
var completionHandlers: [() -> Void] = []

// 传入的闭包/函数并没有在函数内执行，需要在函数类型钱添加@escaping声明
func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
completionHandlers.append(completionHandler)
}
```

**逃逸闭包只是一个声明，以增强函数的意图**

### 5.自动闭包

对于没有参数的闭包，swift提供了一种简写的方式，直接写函数体，不需要函数形式（返回值和参数列表），如下:

```
// 声明一个自动闭包（无参数，可以有返回值，返回值类型swift可以自动识别）
let sayHello = { print("hello world") }

//调用闭包函数
sayHello()
```
**自动闭包只是闭包的一种简写方式**

如果一个函数接受一个不带参数的闭包:

```
func logIfTrue(predicate: () -> Bool) {
if predicate() {
print("True")
}
}
```

调用:

```
logIfTrue(predicate: { return 1 < 2 })

// 可以简化return
logIfTrue(predicate: { 1 < 2 })
```

上面代码看起来可读性不是很好，swift引入了一个关键字`@autoclosure`，简化自动闭包的大括号，在闭包类型前面添加该关键字声明:

```
func logIfTrue(predicate: @autoclosure () -> Bool) {
if predicate() {
print("True")
}
}

// 调用
logIfTrue(predicate:1 < 2)
```

@autoclosure 关键字是为了简化闭包的写法，增强可读性，这里的例子比较简单，可以参考：[了解更多点击这里](http://swifter.tips/autoclosure/)

