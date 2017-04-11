---
title: 谈谈 Swift 中的 map 和 flatMap
date: 2017-04-06 09:59:45
tags: [map,flatMap]
categories: swift学习笔记
toc: true
---

`map` 和 `flatMap` 是 `Swift` 中两个常用的函数，它们体现了 `Swift` 中很多的特性。对于简单的使用来说，它们的接口并不复杂，但它们内部的机制还是非常值得研究的，能够帮助我们够好的理解 `Swift` 语言。

# **map 简介**

首先，咱们说说 map 函数如何使用。

```
let numbers = [1,2,3,4]
let result = numbers.map { $0 + 2 }
print(result)  // [3,4,5,6]

```

`map` 方法接受一个闭包作为参数， 然后它会遍历整个 `numbers` 数组，并对数组中每一个元素执行闭包中定义的操作。 相当于对数组中的所有元素做了一个映射。 比如咱们这个例子里面的闭包是讲所有元素都加 `2` 。 这样它产生的结果数据就是 `[3,4,5,6]`。

初步了解之后，我们来看一下 `map` 的定义：

```
func map<T>(@noescape transform: (Self.Generator.Element) throws -> T) rethrows -> [T]

```

咱们抛开一些和关键逻辑无关的修饰符 `@noescape`,`throws` 这些，在整理一下就是这样：

```
func map<T>(transform: (Self.Generator.Element) -> T) rethrows -> [T]

```

`map` 函数接受一个闭包， 这个闭包的定义是这样的：

```
(Self.Generator.Element) -> T

```

它接受 `Self.Generator.Element` 类型的参数， 这个类型代表数组中当前元素的类型。 而这个闭包的返回值，是可以和传递进来的值不同的。 比如我们可以这样：

```
let stringResult = numbers.map { "No. \($0)" }
// ["No. 1", "No. 2", "No. 3", "No. 4"]

```

这次我们在闭包装把传递进来的数字拼接到一个字符串中， 然后返回一个组数， 这个数组中包含的数据类型，就是我们拼接好的字符串。

这就是关于 `map` 的初步了解， 我们继续来看 `flatMap`。

# **flatMap**

`map` 可以对一个集合类型的所有元素做一个映射操作。 那么 `flatMap` 呢？

让我们来看一个 `flatMap` 的例子：

```
result = numbers.flatMap { $0 + 2 }
// [3,4,5,6]

```

我们对同样的数组使用 `flatMap` 进行处理， 得到了同样的结果。 那 `flatMap` 和 `map` 到底有什么区别呢？

咱们再来看另一个例子：

```
let numbersCompound = [[1,2,3],[4,5,6]];
var res = numbersCompound.map { $0.map{ $0 + 2 } }
// [[3, 4, 5], [6, 7, 8]]
var flatRes = numbersCompound.flatMap { $0.map{ $0 + 2 } }
// [3, 4, 5, 6, 7, 8]

```

这里就看出差别了。 对于二维数组， `map` 和 `flatMap` 的结果就不同了。 我们先来看第一个调用：

```
var res = numbersCompound.map { $0.map{ $0 + 2 } }
// [[3, 4, 5], [6, 7, 8]]

```

`numbersCompound.map { ... }` 这个调用实际上是遍历了这里两个数组元素 `[1,2,3]` 和 `[4,5,6]`。 因为这两个元素依然是数组，所以我们可以对他们再次调用 `map` 函数：`$0.map{ $0 + 2 }`。 这个内部的调用最终将数组中所有的元素加 `2`。

再来看看 `flatMap` 的调用：

```
var flatRes = numbersCompound.flatMap { $0.map{ $0 + 2 } }
// [3, 4, 5, 6, 7, 8]

```

`flatMap` 依然会遍历数组的元素，并对这些元素执行闭包中定义的操作。 但唯一不同的是，它对最终的结果进行了所谓的 “降维” 操作。 本来原始数组是一个二维的， 但经过 `flatMap` 之后，它变成一维的了。

`flatMap` 是如何做到的呢，它的原理是什么，为什么会存在这样一个函数呢？ 相信此时你脑海中肯定会浮现出类似的问题。

下面咱们再来看一下 `flatMap` 的定义, 还是抛去 `@noescape`, `rethrows` 这些无关逻辑的关键字：

```
func flatMap<T>(transform: (Self.Generator.Element) throws -> T?) -> [T]
func flatMap<S : SequenceType>(transform: (Self.Generator.Element) -> S) -> [S.Generator.Element]

```

和 `map` 不同， `flatMap` 有两个重载。 参照我们刚才的示例， 我们调用的其实是第二个重载：

```
func flatMap<S : SequenceType>(transform: (Self.Generator.Element) -> S) -> [S.Generator.Element]

```

`flatMap` 的闭包接受的是数组的元素，但返回的是一个 `SequenceType` 类型，也就是另外一个数组。 这从我们刚才这个调用中不难看出：

```
numbersCompound.flatMap { $0.map{ $0 + 2 } }

```

我们传入给 `flatMap` 一个闭包 `$0.map{ $0 + 2 }` , 这个闭包中，又对 `$0` 调用了 `map` 方法， 从 `map` 方法的定义中我们能够知道，它返回的还是一个集合类型，也就是 `SequenceType`。 所以我们这个 `flatMap` 的调用对应的就是第二个重载形式。

那么为什么 `flatMap` 调用后会对数组降维呢？ 我们可以从它的源码中窥探一二（`Swift` 不是开源了吗~）。

```
extension Sequence {
    
    //...
    
public func flatMap<S : Sequence>(
    @noescape transform: (${GElement}) throws -> S
  ) rethrows -> [S.${GElement}] {
    var result: [S.${GElement}] = []
    for element in self {
      result.append(contentsOf: try transform(element))
    }
    return result
  }
  
  //...
  
}

```

这就是 `flatMap` 的完整源码了， 它的源码也很简单， 对遍历的每一个元素调用 `try transform(element)`。 `transform` 函数就是我们传递进来的闭包。

然后将闭包的返回值通过 `result.append(contentsOf:)` 函数添加到 `result` 数组中。

那我们再来看一下 `result.append(contentsOf:)` 都做了什么， 它的文档定义是这样：

> Append the elements of newElements to self.

简单说就是将一个集合中的所有元素，添加到另一个集合。 还以我们刚才这个二维数组为例：

```
let numbersCompound = [[1,2,3],[4,5,6]];
var flatRes = numbersCompound.flatMap { $0.map{ $0 + 2 } }
// [3, 4, 5, 6, 7, 8]

```

`flatMap` 首先会遍历这个数组的两个元素 `[1,2,3]` 和 `[4,5,6]`， 因为这两个元素依然是数组， 所以我们可以对他们再进行 `map` 操作： `$0.map{ $0 + 2 }`。

这样， 内部的 `$0.map{ $0 + 2 }` 调用返回值类型还是数组， 它会返回 `[3,4,5]` 和 `[6,7,8]`。

然后， `flatMap` 接收到内部闭包的这两个返回结果， 进而调用 `result.append(contentsOf:)` 将它们的数组中的内容添加到结果集中，而不是数组本身。

那么我们最终的调用结果理所当然就应该是 `[3, 4, 5, 6, 7, 8]` 了。

仔细想想是不是这样呢~

# **flatMap 的另一个重载**

我们刚才分析了半天， 其实只分析到 `flatMap` 的一种重载情况， 那么另外一种重载又是怎么回事呢：

```
func flatMap<T>(transform: (Self.Generator.Element) -> T?) -> [T]

```

从定义中我们看出， 它的闭包接收的是 `Self.Generator.Element` 类型， 返回的是一个 `T?` 。 我们都知道，在 `Swift` 中类型后面跟随一个 `?`， 代表的是 `Optional` 值。 也就是说这个重载中接收的闭包返回的是一个 `Optional` 值。 更进一步来说，就是闭包可以返回 `nil`。

我们来看一个例子：

```
let optionalArray: [String?] = ["AA", nil, "BB", "CC"];
var optionalResult = optionalArray.flatMap{ $0 }
// ["AA", "BB", "CC"]

```

这样竟然没有报错， 并且 `flatMap` 的返回结果中， 成功的将原数组中的 `nil` 值过滤掉了。 再仔细观察，你会发现更多。 使用 `flatMap` 调用之后， 数组中的所有元素都被解包了， 如果同样使用 `print` 函数输出原始数组的话， 大概会得到这样的结果:

```
[Optional("AA"), nil, Optional("BB"), Optional("CC")]

```
而使用 `print` 函数输出 `flatMap` 的结果集时，会得到这样的输出：

```
["AA", "BB", "CC"]

```

也就是说原始数组的类型是 `[String?]` 而 `flatMap` 调用后变成了 `[String]`。 这也是 `flatMap` 和 `map` 的一个重大区别。 如果同样的数组，我们使用 `map` 来调用， 得到的是这样的输出：

```
[Optional("AA"), nil, Optional("BB"), Optional("CC")]

```

这就和原始数组一样了。 这两者的区别就是这样。 map 函数值对元素进行变换操作。 但不会对数组的结构造成影响。 而 flatMap 会影响数组的结构。再进一步分析之前，我们暂且这样理解。

`flatMap` 的这种机制，而已帮助我们方便的对数据进行验证，比如我们有一组图片文件名， 我们可以使用 `flatMap` 将无效的图片过滤掉：

```
var imageNames = ["test.png", "aa.png", "icon.png"];
imageNames.flatMap{ UIImage(named: $0) }

```

那么 `flatMap` 是如何实现过滤掉 `nil` 值的呢？ 我们还是来看一下源码：

```
extension Sequence {
  
  // ... 
  public func flatMap<T>(
    @noescape transform: (${GElement}) throws -> T?
  ) rethrows -> [T] {
    var result: [T] = []
    for element in self {
      if let newElement = try transform(element) {
        result.append(newElement)
      }
    }
    return result
  }
  
  // ... 
  
}

```

依然是遍历所有元素，并应用 `try transform(element)` 闭包的调用， 但关键一点是，这里面用到了 `if let` 语句， 对那些只有解包成功的元素，才会添加到结果集中:

```
if let newElement = try transform(element) {
    result.append(newElement)
}

```

这样， 就实现了我们刚才看到的自动去掉 `nil` 值的效果了。

# **结尾**

关于 `Swift` 中的 `map` 和 `flatMap`， 看完这篇内容是不会会对你有所启发呢。 当然， 关于这两个函数我们这里并没有完全讨论完。 它们背后还有着更多的思想。 关于本篇文章的代码，大家还可以来 `Github` 上面 [参看.](https://github.com/swiftcafex/mapAndFlatmap)


转自：https://www.swiftcafe.io/2016/03/28/about-map/?utm_source=tuicool&utm_medium=referral