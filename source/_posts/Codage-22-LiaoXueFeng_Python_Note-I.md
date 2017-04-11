title: 廖雪峰python教程学习笔记
date: 2016-05-18 16:00:28
categories: Codage|编程
tags: [Python, 廖雪峰]
---

针对不熟悉的知识点，按照原教程框架和顺序进行记录。
廖兄的教程大致分为三个大的部分：基础对象介绍、python编程方式及后续开发知识点。
这一篇是第一部分的知识点拾遗。
<!-- more -->

## 基本数据类型相关
### tuple的不可修改性
list可修改而tuple不可修改(tuple也是一种list)。tuple不可修改针对的是内存的指向，如：
``` python
L = ("a", "b", ["c", "d"])
```
第一个元素和第二个元素均不能修改，因为tuple的前两个元素直接指向了内存中的"a"和"b"。第三个元素是["c", "d"]这个list，不可修改仅针对他最基础的那层属性，即list，不可修改为其它属性的对象，而list内的元素是可以变更的。

### 判断一个key在dict中是否存在
方法1：`in`: 如`'a' in dict`, 如不存在，返回`False`
方法2: `.get()`: 如`dict.get('a')`，如不存在返回`None`。可指定返回值：`dict.get('a', -1)`，如不存在，则返回-1

## 函数相关
### 空函数
`pass`语句什么都不做，那有什么用？实际上`pass`可以用来作为占位符，比如现在还没想好怎么写函数的代码，就可以先放一个`pass`，让代码能运行起来。

### 函数返回多个值
如果函数内部定义了同时返回多个值，比如函数`test(lambda)`返回`x`, `y`, `z`，运行后分别在函数内部赋值为1, 2, 3。我们将结果保存：
```python
a, b, c = test(lambda)
print(a, b, c)
```
可以得到返回值为
```python
1, 2, 3
```
如果将函数仅赋值为一个变量：
```python
p = test(lambda)
print(p)
```
那么我们可以得到如下结果：
```python
(1, 2, 3)
```
返回值是一个tuple。
> 在语法上，返回一个tuple可以省略括号，而多个变量可以同时接收一个tuple，按位置赋给对应的值。所以，Python的函数返回多值其实就是返回一个tuple，但写起来更方便。

### 可变(长度)参数
当函数可处理的参数个数不确定时，如：
``` python
def calc(numbers):
    sum = 0
    for n in numbers:
        sum = sum + n * n
    return sum
```
调用这个函数时，如果要计算多个值相加的结果，需要将这些值塞进一个list或者一个tuple，而不能直接写成`calc(1, 2, 3)`。想要实现后一种调用方式，则需要在参数前添加符号：`*`。如下:
``` python
def calc(*numbers):
    sum = 0
    for n in numbers:
        sum = sum + n * n
    return sum
```
在函数内部，`*`的作用是让参数`numbers`接收到的是一个tuple，这样，调用这个函数时，可以传入任意数量(理论上来说，0到正无穷)的参数。比如有了一个list:
``` python
nums = [1, 2, 3]
```
我们并不用这样调用：
```python
calc(nums[0], nums[1], nums[2])
```
而是可以直接在`nums`前添加`*`号，再传入函数：
```python
calc(*nums)
```

### 关键字参数
``**``加参数名即为关键字参数，允许传入0个或任意个含参数名的参数，并在函数内部自动组成dict。
上面提到的可变参数——在函数内部自动组成tuple，在函数调用时，可以直接使用list和tuple。关键字参数的使用方法也一样，函数调用时，也可以以dict的形式设定参数。

函数示例：
``` python
def person(name, age, **kw):
    print('name:', name, 'age:', age, 'other:', kw)
```

可以这样调用：
``` python
person('Adam', 45, city='Beijing', job='Engineer')
```

和可变参数类似，也可以先组装出一个dict，然后，把该dict转换为关键字参数传进去：

``` python
extra = {'city': 'Beijing', 'job': 'Engineer'}
person('Jack', 24, city=extra['city'], job=extra['job'])
```
当然，上面复杂的调用可以用简化的写法：

``` python
extra = {'city': 'Beijing', 'job': 'Engineer'}
person('Jack', 24, **extra)
```
> `**extra`表示把extra这个dict的所有key-value用关键字参数传入到函数的``**kw``参数，kw将获得一个dict，注意`kw`获得的dict是`extra`的一份拷贝，对kw的改动不会影响到函数外的`extra`

### 命名关键字参数
可变参数和关键字参数的好处是参数的传入不受限制，但如果想控制，比如关键字参数传入了哪些，就必须在函数内部进行检查。
如果想限制关键字参数的名字而省掉关键字的检查步骤，则应考虑使用**命名关键字参数**。比如，只接受`city`和`job`做为关键字参数：

``` python
def person(name, age, *, city, job):
    print(name, age, city, job)
```

命名关键字参数需要一个特殊分隔符`*`，`*`后面的参数被视为命名关键字参数。如果函数定义中已经有了一个可变参数，则后面不再需要一个特殊分隔符`*`。
> 使用命名关键字参数时，要特别注意，如果没有可变参数，就必须加一个*作为特殊分隔符。如果缺少*，Python解释器将无法识别位置参数和命名关键字参数

``` python
def person(name, age, *, city, job):
    print(name, age, city, job)

# or

def person(name, age, *args, city, job):
    print(name, age, args, city, job)
```

要注意的是命名关键字参数必须传入参数名，如果没有传入参数名，python视所有参数为位置参数，进而报错。而上面定义的`person()`只有两个位置参数。

> 在Python中定义函数，可以用必选参数、默认参数、可变参数、关键字参数和命名关键字参数，这5种参数都可以组合使用。但是请注意，`**`参数定义的顺序必须是：必选参数、默认参数、可变参数、命名关键字参数和关键字参数`**`。

### 递归函数的尾递归优化
先贴一句让我撞墙的话：
> 在计算机中，函数调用是通过栈（stack）这种数据结构实现的，每当进入一个函数调用，栈就会加一层栈帧，每当函数返回，栈就会减一层栈帧。由于栈的大小不是无限的，所以，递归调用的次数过多，会导致栈溢出。

举的例子是阶乘函数：
``` python
def fact(n):
    if n==1:
        return 1
    return n * fact(n - 1)

print(fact(1))
```
    ## 1

``` python
print(fact(5))
```
    ## 120

``` python
print(fact(10))
```
    ## 3628800

然而当我们尝试`fact(1000)`的时候，函数会报错。于是有了这一小节开头的那一句话。解决方案是使用**尾递归优化**。

> 尾递归是指，在函数返回的时候，调用自身本身，并且，`return`语句不能包含表达式。这样，编译器或者解释器就可以把尾递归做优化，使递归本身无论调用多少次，都只占用一个栈帧，不会出现栈溢出的情况。

``` python
def fact(n):
    return fact_iter(n, 1)

def fact_iter(num, product):
    if num == 1:
        return product
    return fact_iter(num - 1, num * product)
```

两种递归的计算步骤对比是：
递归：
``` python
===> fact(5)
===> 5 * fact(4)
===> 5 * (4 * fact(3))
===> 5 * (4 * (3 * fact(2)))
===> 5 * (4 * (3 * (2 * fact(1))))
===> 5 * (4 * (3 * (2 * 1)))
===> 5 * (4 * (3 * 2))
===> 5 * (4 * 6)
===> 5 * 24
===> 120
```
尾递归优化：
``` python
===> fact_iter(5, 1)
===> fact_iter(4, 5)
===> fact_iter(3, 20)
===> fact_iter(2, 60)
===> fact_iter(1, 120)
===> 120
```
> 尾递归调用时，如果做了优化，栈不会增长，因此，无论多少次调用也不会导致栈溢出。
> 遗憾的是，大多数编程语言没有针对尾递归做优化，Python解释器也没有做优化，所以，即使把上面的fact(n)函数改成尾递归方式，也会导致栈溢出。

## 高级特性
### 切片(Slicing)
在一个可提取子集的对象后用`[:]`切片。
如果加入第二个`:`如`[::]`，则第二个`:`表示切片的步长。
如:
``` python
L = list(range(100))
L[:10:2]
```

    ## [0, 2, 4, 6, 8]

``` python
L[::5]
```

    ## [0, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50, 55, 60, 65, 70, 75, 80, 85, 90, 95]

### 迭代(Iteration)
判断一个对象是否可以迭代：通过collections模块中的Iterable：
``` python
from collections import Iterable
isinstance('abc', Iterable)    # True
isinstance([1,2,3], Iterable)  # True
isinstance(123, Iterable)      # False
```

enumerate可以吧一个list变成*索引-元素对*。
``` python
for i, value in enumerate(['A', 'B', 'C']):
    print(i, value)
```

    ## 0 A
    ## 1 B
    ## 2 C

### 列表生成器(List Comprehensions)
用于简化列表生成的python内置功能。
生成一个`[1, 4, 9, ..., 100]`的llist，常规方法是循环生成：
``` python
for x in range(1, 11):
    L.append(x * x)
```
列表生成式提供了一种用一行既可以替代循环生成list的方式：
``` python
[x * x for x in range(1, 11)]
```
这种生成方式可以与SQL语句对比(假设`range(1,11)`在数据库中为表`temp`，该表只有一个field：x)：
``` SQL
SELECT x * x FROM temp
```

`for`循环后面可以加上`if`判断，筛选出偶数的平方：
``` python
[x * x for x in range(1, 11) if x % 2 == 0]
```
对比SQL：
``` SQL
SELECT x * x FROM temp WHERE x % 2 = 0
```

列表生成器也可以使用多层循环(一般最多两层)：
``` python
[m + n for m in 'ABC' for n in 'XYZ']
```

### 生成器(Generator)
列表生成器的局限是无法生成一个过长的list，因为太长的list会占用太大的内存以至于无法实现这类list的创建。如果只需要对这种列表的前几个元素进行计算，那么后面的元素所占用的空间就被浪费了
生成器与列表生成器的区别就在于，每一个元素可以在循环中创建，而不用创建完整的list。元素的创建都依靠算法，一次只占用一个单位的内存空间。
生成器最直接的创建方式是讲列表生成式的`[]`替换为`()`
``` python
L = [x * x for x in range(10)]
L
```

    ## [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

``` python
g = (x * x for x in range(10))
g
```

    ## <generator object <genexpr> at 0x1022ef630>

想要看到`g`(刚刚创建的生成器)的内容，有两种方式，可以通过`next(g)`来查看下一个返回值，直到返回`StopIteration`的错误信息。
``` python
next(g)
```

    ## 0

``` python
next(g)
```

    ## 1

``` python
next(g)
```

    ## 4

``` python
next(g)
```

    ## 9

``` python
next(g)
```

    ## 16

``` python
next(g)
```

    ## 25

``` python
next(g)
```

    ## 36

``` python
next(g)
```

    ## 49

``` python
next(g)
```

    ## 64

``` python
next(g)
```

    ## 81

``` python
next(g)
```

    ## Traceback (most recent call last):
    ##     File "<stdin>", line 1, in <module>
    ## StopIteration

或者使用`for`循环：

``` python
for n in g:
    print(n)
```

    ## 0
    ## 1
    ## 4
    ## 9
    ## 16
    ## 25
    ## 36
    ## 49
    ## 64
    ## 81

回到generator的创建上，另一种方法是在函数中使用`yield`。以一个能生成斐波那契数列的函数为例：
``` python
def fib1(max):
    n, a, b = 0, 0, 1
    while n < max:
        print(b)
        a, b = b, a + b
        n = n + 1
    return 'done'
```
而generator只需要更改其中的一个语句：
``` python
def fib2(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b
        a, b = b, a + b
        n = n + 1
    return 'done'
```
来看看效果：
```python
fib1(6)
```

    ## 1
    ## 1
    ## 2
    ## 3
    ## 5
    ## 8
    ## 'done'

``` python
f = fib(6)
f
```

    ## <generator object fib at 0x104feaaa0>

``` python
next(f)
```

    ## 1

``` python
next(f)
```

    ## 1

``` python
next(f)
```

    ## 2

``` python
next(f)
```

    ## 3

两个例子对比可以看出：

* 函数里`return`是一个触发器，遇到`return`的时候，程序返回结果
* 生成器里，`yield`是一个触发器。每次调用next()就执行一次生成器，遇到`yield`就返回结果，再次执行时，从上次返回的`yield`的地方继续执行

当然用`next()`来获取返回值比较不效率，一般可以直接使用`for`循环来迭代：

``` python
for n in f:
    print(n)
```

    ## 1
    ## 1
    ## 2
    ## 3
    ## 5
    ## 8

### 迭代器(Iterator)

> 可以被next()函数调用并不断返回下一个值的对象称为迭代器：`Iterator`。
> 可以使用`isinstance()`判断一个对象是否是`Iterator`对象

``` python
from collections import Iterator
isinstance((x for x in range(10)), Iterator)  # True
isinstance([], Iterator)                      # False
isinstance({}, Iterator)                      # False
isinstance('abc', Iterator)                   # False
```

> 为什么list、dict、str等数据类型不是Iterator？
> 这是因为Python的Iterator对象表示的是一个数据流，Iterator对象可以被next()函数调用并不断返回下一个数据，直到没有数据时抛出StopIteration错误。可以把这个数据流看做是一个有序序列，但我们却不能提前知道序列的长度，只能不断通过next()函数实现按需计算下一个数据，所以Iterator的计算是惰性的，只有在需要返回下一个数据时它才会计算。
> Iterator甚至可以表示一个无限大的数据流，例如全体自然数。而使用list是永远不可能存储全体自然数的。