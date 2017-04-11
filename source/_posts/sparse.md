---
title: Sparse 简介
tags:
categories:
  - Kernel
  - 内核文档
date: 2014-05-14 12:07:03
---

参考：
[内核工具 – Sparse 简介](http://www.cnblogs.com/wang_yb/p/3575039.html)
<http://en.wikipedia.org/wiki/Sparse>

Sparse (Semantic Parser) 是内核代码静态分析工具, 能够帮助我们找出代码中的隐患.
<!--more-->

## Sparse 介绍

Sparse 诞生于 2004 年, 是由linux之父开发的, 目的就是提供一个静态检查代码的工具, 从而减少linux内核的隐患.

其实在Sparse之前, 已经有了一个不错的代码静态检查工具("SWAT"), 只不过这个工具不是免费软件, 使用上有一些限制。所以 linus 还是自己开发了一个静态检查工具.

具体可以参考这篇文章(2004年的文章了):[Finding kernel problems automatically](http://lwn.net/Articles/87538/)

内核代码中还有一个简略的关于Sparse的说明文件: <span style="color:red;">_Documentation/sparse.txt_</span>

Sparse通过 gcc 的扩展属性 `__attribute__` 以及sparse定义的 `__context__` 来对代码进行静态检查.

这些属性如下(尽量整理的,可能还有些不全的地方):

| 宏名称 | 宏定义 | 检查点 |
|--------|--------|--------|
| `__bitwise` | `__attribute__((bitwise))` | 确保变量是相同的位方式(比如 `big-endian`, `little-endia`等) |
| `__user` | `__attribute__((noderef, address_space(1)))` | 指针地址必须在用户地址空间 |
| `__kernel` | `__attribute__((noderef, address_space(0)))` | 指针地址必须在内核地址空间 |
| `__iomem` | `__attribute__((noderef, address_space(2)))` | 指针地址必须在设备地址空间 |
| `__safe` | `__attribute__((safe))` | 变量可以为空 |
| `__force` | `__attribute__((force))` | 变量可以进行强制转换 |
| `__nocast` | `__attribute__((nocast))` | 参数类型与实际参数类型必须一致 |
| `__acquires(x)` | `__attribute__((context(x, 0, 1)))` | 参数x 在执行前引用计数必须是0,执行后,引用计数必须为1 |
| `__releases(x)` | `__attribute__((context(x, 1, 0)))` | 与 `__acquires(x)` 相反 |
| `__acquire(x)` | `__context__(x, 1)` | 参数x的引用计数 + 1 |
| `__release(x)` | `__context__(x, -1)` | 与 `__acquire(x)` 相反 |
| `__cond_lock(x,c)` | `((c) ? ({ __acquire(x); 1; }) : 0)` | 参数`c`不为0时,引用计数 + 1, 并返回1 |

其中 `__acquires(x)` 和 `__releases(x)`, `__acquire(x)` 和 `__release(x)` 必须配对使用, 否则 Sparse 会给出警告。

修饰符 `__attribute__((context(...)))` 可由Sparse宏 `__context__(...)`替代，`context(...` 原型为：`context(expression,in_context,out_context)`

## Linux 内核中的定义

Linux内核在 <span style="color:red;">_linux/compiler.h_</span> 和 <span style="color:red;">_linux/types.h_</span> 文件中定义了如下简短形式的预编译宏(如果编译的时候不使用 `__CHECKER__` 标记, 代码中所有的这些注记将被删除):
```c
#ifdef __CHECKER__
# define __user		__attribute__((noderef, address_space(1)))
# define __kernel	__attribute__((address_space(0)))
# define __iomem	__attribute__((noderef, address_space(2)))
# define __safe		__attribute__((safe))
# define __force	__attribute__((force))
# define __nocast	__attribute__((nocast))
# define __must_hold(x)	__attribute__((context(x,1,1)))
# define __acquires(x)	__attribute__((context(x,0,1)))
# define __releases(x)	__attribute__((context(x,1,0)))
# define __acquire(x)	__context__(x,1)
# define __release(x)	__context__(x,-1)
# define __cond_lock(x,c)	((c) ? ({ __acquire(x); 1; }) : 0)
# define __percpu	__attribute__((noderef, address_space(3)))
#ifdef CONFIG_SPARSE_RCU_POINTER
# define __rcu		__attribute__((noderef, address_space(4)))
#else
# define __rcu
#endif
extern void __chk_user_ptr(const volatile void __user *);
extern void __chk_io_ptr(const volatile void __iomem *);
#else
# define __user
# define __kernel
# define __iomem
# define __safe
# define __force
# define __nocast
# define __chk_user_ptr(x) (void)0
# define __chk_io_ptr(x) (void)0
# define __builtin_warning(x, y...) (1)
# define __must_hold(x)
# define __acquires(x)
# define __releases(x)
# define __acquire(x) (void)0
# define __release(x) (void)0
# define __cond_lock(x,c) (c)
# define __percpu
# define __rcu
#endif
[/cc]
[cc lang="c"]
#ifdef __CHECKER__
# define __bitwise__    __attribute__((bitwise))
#else
# define __bitwise__
#endif

#ifdef __CHECK_ENDIAN__
# define __bitwise      __bitwise__
#else
# define __bitwise
#endif
```

示例：
```c
typedef __u32 __bitwise    __le32;
typedef __u32 __bitwise    __be32;
```
类型 `__le32` 和 `__be32` 代表不同字节顺序的32位整数类型。然而，C 语言并未指定这些类型的变量不应混合在一起。按位(bitwise)属性是用来标记这些类型的限制，所以，如果这些类型或其他整型变量混合在一起，Sparse将给出警告信息。
为了标记(mark)限制类型( restricted types)之间的有效转换，使用强制(force)属性的以避免Sparse给予警告。

**Using sparse for lock checking**

The following macros are undefined for gcc and defined during a sparse run to use the "context" tracking feature of sparse, applied to locking. These annotations tell sparse when a lock is held, with regard to the annotated function's entry and exit.

> `__must_hold` - The specified lock is held on function entry and exit.
> `__acquires` - The specified lock is held on function exit, but not entry.
> `__releases` - The specified lock is held on function entry, but not exit.

If the function enters and exits without the lock held, acquiring and releasing the lock inside the function in a balanced way, noannotation is needed. The tree annotations above are for cases where sparse would otherwise report a context imbalance.

## Sparse 在编译内核中的使用

用 Sparse 对内核进行静态分析非常简单.

> \# 检查所有内核代码
> make C=1 检查所有重新编译的代码
> make C=2 检查所有代码, 不管是不是被重新编译
