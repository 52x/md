title: Java Lambda 表达式（又名闭包(Closure)/匿名函数) 笔记
date: 15 Nov 2016 2:01 AM
categories: "java"
tags: [java,]
---

根据[JSR 335](https://jcp.org/en/jsr/detail?id=335), Java 终于在 Java 8 中引入了 Lambda 表达式。也称之为闭包或者匿名函数。

<!--more-->

![http://harchiko.qiniudn.com/Lambda%20Expression%20Java%208.png](http://harchiko.qiniudn.com/Lambda%20Expression%20Java%208.png)

## JSR 335

所谓的 JSR （Java Specification Requests） 全称叫做 Java 规范提案。简单来说就是向 Java 社区提交新的 API 或 服务 请求的提案。这些提案将作为 Java 社区进行 Java 语言开发的需求，引导着开发的方向。

JSR 335 的提案内容摘要如下：

> This JSR will extend the Java Programming Language Specification and the Java Virtual Machine Specification to support the following features:
  - Lambda Expressions
  - SAM Conversion
  - Method References
  - Virtual Extension Methods

也就是如下几点：

1. 支持 lambda 表达式。
2. 支持 SAM conversion 用来向前兼容。
3. 方法引用 Method References
4. Virtual Extension Methods

在 Java 8 中，以上均已经实现,以上内容下文均有介绍。

## 为什么需要 Lambda 表达式?

Lambda 表达式，其实就是代码块。

![http://harchiko.qiniudn.com/56cabf5a499ed708%202.jpg](http://harchiko.qiniudn.com/56cabf5a499ed708%202.jpg)

### 原来怎么处理
在具体了解 lambda 之前，我们先往后退一步，看看之前我们是如何处理这些代码块的！

#### 例子一
当决定在单独的线程运行某程序时，你这样做的

```java
class Worker implements Runnable {
     public void run() {
        for (int i = 0; i < 1000; i++)
           doWork();
     }
     ...
  }
```

这样执行：

```java
Worker w = new Worker();
new Thread(w).start();
```

Worker 中包含了你要执行的代码块。

#### 例子二
如果你想实现根据字符串长度大小来排序，而不是默认的字母顺序，你可以自己来实现一个  Comparator 用来 Sort。

```java
class LengthComparator implements Comparator<String> {
     public int compare(String first, String second) {
        return Integer.compare(first.length(), second.length());
     }
  }
   
Arrays.sort(strings, new LengthComparator());
```

#### 例子三
另外一个例子，我选的是 Android 中的点击事件，同样是 Java:

```java
button.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        Toast.makeText(MainActivity.this, "Hello World!", Toast.LENGTH_SHORT).show();
    }
});
```

### 上面代码有什么问题呢？

![http://harchiko.qiniudn.com/c718cee7.jpg](http://harchiko.qiniudn.com/c718cee7.jpg)

它们都太复杂了啊！

>上述例子都是在某个类中实现某个接口，然后传递到另外一个方法中作为参数，然后用来执行。

但是本质上，他们要传递的就是接口中那一个方法的实现而已啊！有必要先创建类，再实例化，再传递给调用的位置吗？

> 因为 Java 是纯面向对象的语言，像其他语言那样随随便便传个方法过来，那可不行，必须要这样。

在其他语言中你可能可以，但是，在Java 中，不可以。

![http://harchiko.qiniudn.com/56cabf7011ab6750.jpg](http://harchiko.qiniudn.com/56cabf7011ab6750.jpg)

Java 设计人员为了 Java 的简洁跟连贯性，一直拒绝为Java添加这种功能。（这也是我喜欢Java而不喜欢Python的原因啊！！！)

经过多年的努力，开发人员终于找到了符合 Java 编程习惯的 Lambda 表达式！

## Lambda 表达式语法(Syntax)

考虑下前面的例子：

```java
Integer.compare(first.length(), second.length())
```

first和second都是 String 类型，Java 是强类型的语言，必须指定类型:

```java
(String first, String second)
     -> Integer.compare(first.length(), second.length())
```

![http://harchiko.qiniudn.com/14365393725281065.jpg](http://harchiko.qiniudn.com/14365393725281065.jpg)

看到没有！第一个 Lambda 表达式诞生了！！输入、输出简洁明了！

> 为什么叫 Lambda 呢，这个很多年以前，有位逻辑学家想要标准化的表示一些可以被计算的数学方程（实际上存在，但是很难被表示出来），他就用 ℷ 来表示。

重新介绍一下 Java 中 Lambda 表达式的格式:

>(参数) -> 表达式

### 多返回值
如果计算的结果并不由一个单一的表达式返回（换言之，返回值存在多种情况），使用“{}"，然后明确指定返回值。

```java
(String first, String second) -> {
     if (first.length() < second.length()) return -1;
     else if (first.length() > second.length()) return 1;
     else return 0;
}
```

### 无参数
如果没有参数，则 "()"中就空着。

```java
() -> { for (int i = 0; i < 1000; i++) doWork(); }
```

### 省略
如果参数的类型可以被推断出，则可以直接省略

```java
Comparator<String> comp
     = (first, second) // Same as (String first, String second)
        -> Integer.compare(first.length(), second.length());
```

这里，first和second可以被推断出是 String 类型，因为 是一个 String 类型的 Comparator。

如果单个参数可以被推断出，你连括号都可以省略：
```java
EventHandler<ActionEvent> listener = event ->
     System.out.println("Thanks for clicking!");
        // Instead of (event) -> or (ActionEvent event) ->
```

### 修饰符
你可以像对待其他方法一样，annotation，或者 使用 final 修饰符

```java
(final String name) -> ...
    (@NonNull String name) -> ...
```

**永远不要**定义 result 的类型，lambda 表达式总是从上下文中推断出来的：

```java
(String first, String second) -> Integer.compare(first.length(), second.length())
```

### 注意
注意，在lambda 表达式中，某些分支存在返回值，某些不存在返回值这样的情况是不允许的。
如 ` (int x) -> { if (x >= 0) return 1; }`这样是非法的。

## 函数式接口(Functional Interfaces/SAM)

> 要介绍 Java 中 lambda 表达式的实现，需要知道什么是 函数式接口。

什么叫作函数式接口呢(SAM)？

>函数式接口指的是只定义了唯一的抽象方法的接口（除了隐含的Object对象的公共方法）， 因此最开始也就做SAM类型的接口（Single Abstract Method）。

Lambda 表达式**向前兼容**这些接口。

### Comparable
举个例子 Array.sort:

```java
Arrays.sort(words,
     (first, second) -> Integer.compare(first.length(), second.length()));
```

Array.sort() 方法收到一个实现了 Comparable<String> 接口的实例。

其实可以把 Lambda 表达式想象成一个方法，而非一个对象，一个可以传入一个接口的方法。


### OnClickListener
再举个例子

```java
button.setOnClickListener(event ->
     System.out.println("Thanks for clicking!"));
```

你看，是不是更易读了呢？

Lambda 表达式能够向前兼容这些 interfaces, 太棒了！ 那 Lambda 表达式还能干什么呢？

实际上，将函数式接口转变成 lambda 表达式是你在 Java 中**唯一**能做的事情。

![http://harchiko.qiniudn.com/20150930185659_eMZyN.jpeg](http://harchiko.qiniudn.com/20150930185659_eMZyN.jpeg)

Why ？！！

在其他的语言中，你可以定义一些方便的方法类型，但在 Java 中，你甚至不能将一个Lambda表达式赋值给类型为 Object 的变量，因为 Object 变量不是一个 Functional Interface。

Java 的设计者们坚持使用熟悉的 interface 概念而不是为其引入新的 方法类型。

(这里我还要为设计者点赞！谨慎的设计，一方面降低了初学者的门槛，一方面方便了高级用户的使用。对比 python2和 python3，升级的不兼容让很多人一直停留在 python2)

## Method References

能不能再简洁一点？有的时候我们所要做的事情不过是调用其他类中方法来处理事件。

```java
button.setOnClickListener(event -> System.out.println(event));
```

如果这样呢？

```java
button.setOnAction(System.out::println);
```

表达式 `System.out::println` 属于一个方法引用（method reference）， 相当于 lambda 表达式 `x -> System.out.println(x)`

![http://harchiko.qiniudn.com/20151220232425_nWH23.jpeg](http://harchiko.qiniudn.com/20151220232425_nWH23.jpeg)

再举个例子，如果你想对字符串不管大小写进行排序,就可以这样写！

```java
Arrays.sort(strings, String::compareToIgnoreCase)
```

如上所见 `::`操作符将方法名与实例或者类分隔开。总体来说，又如下的规则:

* object::instanceMethod
* Class::staticMethod
* Class::instanceMethod

值得指出的是， `this`和`super`关键字可以在其中使用：

```java
class Greeter {
     public void greet() {
        System.out.println("Hello, world!");
     }
  }
```

```java   
class ConcurrentGreeter extends Greeter {
 public void greet() {
    Thread t = new Thread(super::greet);
    t.start();
 }
}
```

## 构造方法引用 Constructor References

跟上一个差不多，毕竟**构造方法** 也是方法啊！！不过方法名字为 new 。

但是！这个构造方法引用有一个牛逼的地方！

你知道 Array 是不能使用范型的对吧！（什么，你不知道？看看这里 [http://stackoverflow.com/questions/2927391/whats-the-reason-i-cant-create-generic-array-types-in-java](http://stackoverflow.com/questions/2927391/whats-the-reason-i-cant-create-generic-array-types-in-java)),你没有办法创建一个类型为 T 的 Array 。 new T[n] 将会被覆盖为 new Object[n]。

假设我们想要一个包含 buttons 的 Array。Stream interface 可以返回一个 Object array。

```java
Object[] buttons = stream.toArray();
```

不不不，我们可不想要 Object。Stream 库使用 构造方法引用解决了这个问题:

```java
Button[] buttons = stream.toArray(Button[]::new);
```

![http://harchiko.qiniudn.com/Screen%20Shot%202016-11-16%20at%204.30.23%20AM.png](http://harchiko.qiniudn.com/Screen%20Shot%202016-11-16%20at%204.30.23%20AM.png)

## 变量作用域
注意到我们在题目中写着 闭包（closure),实际上，闭包的定义是: 引用了自由变量的函数。

在之前，如果需要在匿名类的内部引用外部变量，需要将外部变量定义为 final ，现在有了 lambda 表达式，你不必再这么做了。但同样需要保证外部的自由变量不能在 lambda 表达式中被改变。

![http://harchiko.qiniudn.com/56cabf5d7d6dc247.jpg!600x600.jpg](http://harchiko.qiniudn.com/56cabf5d7d6dc247.jpg!600x600.jpg)
这是什么意思呢？ 不需要定义为 final，也不能改？

其实理解起来很简单，Java 8 中，不需要定义为 final ，但你其实可以直接把他当作 final，不要试图修改它就行了。

即便你用内部类，现在也无需定义为 final 了。

参考 StackOverFlow 链接: [http://stackoverflow.com/questions/4732544/why-are-only-final-variables-accessible-in-anonymous-class](http://stackoverflow.com/questions/4732544/why-are-only-final-variables-accessible-in-anonymous-class)


## Default Methods

由于历史原因，像是类似 Collection 这种接口，如果进行添加接口的话，那将会造成之前的代码出错。

Java 想了一个一劳永逸的方法解决这个问题， 使用 default 修饰符来提供默认的实现

比如 Collection  接口的源代码:

```java
default void remove() {
     throw new UnsupportedOperationException("remove");
}
```

当没有 override remove 这个方法是，调用的时候返回 UnsupportedOperationException 错误。


## Static Methods in Interfaces

Java 8 中，你可以在接口中添加静态方法了。 看起来好像并不符合接口的定义了。

一般用来生成一个简单实现该interface的实例。



参考链接：
1. [JSR 335: Lambda Expressions for the JavaTM Programming Language](https://jcp.org/en/jsr/detail?id=335)
2. [Java 8 新特性概述](https://www.ibm.com/developerworks/cn/java/j-lo-jdk8newfeature/)
3. [Lambda Expressions in Java 8](http://www.drdobbs.com/jvm/lambda-expressions-in-java-8/240166764?pgno=1)


![http://harchiko.qiniudn.com/44577950_p0.jpg](http://harchiko.qiniudn.com/44577950_p0.jpg) 
