title: 廖雪峰python教程学习笔记2
date: 2016-06-23 15:20:28
categories: Codage|编程
tags: [Python, 廖雪峰]
---

## 函数式编程
### 高阶函数
> 把函数作为参数传入，这样的函数称为高阶函数，函数式编程就是指这种高度抽象的编程范式。

#### map/reduce
`map()`接受俩参数，一个是函数名，一个是`Iterable`对象，最后的output是一个`Iterator`

例：
``` python
def f(x):
    return x * x
r = map(f, [1, 2, 3, 4, 5, 6, 7, 8, 9])
list(r)
```

	## [1, 4, 9, 16, 25, 36, 49, 64, 81]

`map()`类似于R语言中的`apply()`族函数，将传入的函数依次作用到序列的每个元素，并把结果合并返回。

`reduce()`同样接受两个参数，把前一个元素的运算结果和下一个元素做累积计算。使用该函数前需要调用`functools`包。

``` python
reduce(f, [x1, x2, x3, x4]) = f(f(f(x1, x2), x3), x4)
```

##### 练习1 `map()`
将一个字符串的list变为首字母大写，其它小写的规范名字：`['adam', 'LISA', 'barT']`。

``` python
# -*- coding: utf-8 -*-
def normalize(name):
    head = name[0].upper()
    tail = name[1:].lower()
    name_norm = head + tail
    return name_norm

test_list = ['adam', 'LISA', 'barT']
print(map(normalize, test_list))
```

	## ['Adam', 'Lisa', 'Bart']

上一段代码的最后一句，用R的`apply`函数系写出来就是：

``` r
sapply(test_list, normalize)
```

##### 练习2 `reduce()`
模仿`sum()`的功能，写一个`prod()`(即乘法函数)。

``` python
# -*- coding: utf-8 -*-
from functools import reduce
def prod(L):
    def times(x, y):
        return x * y
    return reduce(times, L)

print('3 * 5 * 7 * 9 =', prod([3, 5, 7, 9]))
```

	## 3 * 5 * 7 * 9 = 945

##### 练习3 `map()` + `reduce()`
编写一个`str2float`函数，把字符串`'123.456'`转换成浮点数`123.456`：

``` python
# -*- coding: utf-8 -*-
from functools import reduce
def str2float(s):
    def str2int(s):
        return {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}[s]
    def str2num(x, y):
        return 10 * x + y
    a, b = s.split('.')        
    return (reduce(str2num, map(str2int, a)) + reduce(str2num, map(str2int, b)) * 10 ** -len(b))

print('str2float(\'123.456\') =', str2float('123.456'))
```

    ## str2float('123.456') = 123.456

官方解决方案[点此](https://github.com/michaelliao/learn-python3/blob/master/samples/functional/do_reduce.py)。官方代码考虑更周全。

#### filter
`filter`与`map`相似的地方是也用一个函数和一个序列为参数，不同之处在于前者把函数**依次**作用于每个元素，后者是同时作用于每个元素。然后`filter`依据返回的`True`或`False`来决定是否保留序列中的元素。
教程给出了利用生成器(generator)生成素数序列的方案：

``` python



```



#### sorted 