---
title: Java中被搁置的“goto”保留字
date: 2016-08-30 20:48:42
tags: [Java]
toc: true
categories: Java平台
---

goto语句一直被人所诟病，说它使得代码结构复杂化，但是语言设计者们还是没有放弃goto这个功能强大的语句。Java以面向对象所著称也没能够放弃goto，而是把它当做保留字，但是并未在语言中得到正式使用。

然而，从Java的break和continue这两个关键字的身上，我们依然能够看出一些goto的影子。

下面是《Thinking In Java 4th》中关于“goto”的介绍：

> ### 臭名昭著的“goto”
goto 关键字很早就在程序设计语言中出现。事实上，goto 是汇编语言的程序控制结构的始祖：“若条件A，则跳到这里；否则跳到那里”。若阅读由几乎所有编译器生成的汇编代码，就会发现程序控制里包含了许多
跳转。然而，goto 是在源码的级别跳转的，所以招致了不好的声誉。若程序总是从一个地方跳到另一个地方，还有什么办法能识别代码的流程呢？随着Edsger Dijkstra 著名的“Goto 有害”论的问世，goto 便从此
失宠。

事实上，真正的问题并不在于使用goto，而在于goto 的滥用。而且在一些少见的情况下，goto 是组织控制流程的最佳手段。

<!--more-->

尽管goto 仍是Java 的一个保留字，但并未在语言中得到正式使用；Java 没有goto。***然而，在break 和continue 这两个关键字的身上，我们仍然能看出一些goto 的影子。***它并不属于一次跳转，而是中断循环语句的一种方法。之所以把它们纳入goto 问题中一起讨论，是由于它们使用了相同的机制：标签。

## Java中的标签

“标签”是后面跟一个冒号的标识符，就象下面这样：
`label1:`

***对Java 来说，唯一用到标签的地方是在循环语句之前。***进一步说，它实际需要紧靠在循环语句的前方——在标签和循环之间置入任何语句都是不明智的。而在循环之前设置标签的唯一理由是：我们希望在其中嵌套另
一个循环或者一个开关。这是由于break 和continue 关键字通常只中断当前循环，但若随同标签使用，它们就会中断到存在标签的地方。如下所示：

```
label1:
外部循环{
内部循环{
//...
break; //1
//...
continue; //2
//...
continue label1; //3
//...
break label1; //4
}
}
```

在条件1 中，break 中断内部循环，并在外部循环结束。在条件2 中，continue 移回内部循环的起始处。但在条件3 中，continue label1 却同时中断内部循环以及外部循环，并移至label1 处。随后，它实际是继续循环，但却从外部循环开始。在条件4 中，break label1 也会中断所有循环，并回到label1 处，但并不重新进入循环。也就是说，它实际是完全中止了两个循环。


## 代码测试（java）

> 一下代码均已在jdk1.6版本中测试通过

### break语句测试

```
public static void testLabel()
{
    for (int i = 0; i < 2; i++) {
        System.out.println("L1----"+i);
        for (int j = 0; j < 4; j++) {
            if (j == 2) {
                break;
            }
            System.out.println("--------L2---"+j);
        }
    }
}
```
执行结果：

```
L1----0
--------L2---0
--------L2---1
L1----1
--------L2---0
--------L2---1
```

这个代码中break直接中断内部的for循环。

### break+label语句测试

```
public static void testLabel3()
{
label1:
    for (int i = 0; i < 2; i++) {
        System.out.println("L1----"+i);
        for (int j = 0; j < 4; j++) {
            if (j == 2) {
                break label1;
            }
            System.out.println("--------L2---"+j);
        }
    }
}
```

执行结果：

```
L1----0
--------L2---0
--------L2---1
```

在这个代码中break中断标签label1处的外部for循环。

### continue语句测试

```
public static void testLabel2() {
		for (int i = 0; i < 2; i++) {
			System.out.println("L1----"+i);
			for (int j = 0; j < 4; j++) {
				if (j == 2) {
					continue;
				}
				System.out.println("--------L2---"+j);
			}
		}
	}
```

执行结果：

```
L1----0
--------L2---0
--------L2---1
--------L2---3
L1----1
--------L2---0
--------L2---1
--------L2---3
```

在这个代码中continue中断掉内部的for循环后继续执行内部for循环。

### continue+label语句测试

```
public static void testLabel4()
{
label1:
    for (int i = 0; i < 2; i++) {
        System.out.println("L1----"+i);
        for (int j = 0; j < 4; j++) {
            if (j == 2) {
                continue label1;
            }
            System.out.println("--------L2---"+j);
        }
    }
}
```

执行结果：

```
L1----0
--------L2---0
--------L2---1
L1----1
--------L2---0
--------L2---1
```

在这个代码中continue中断掉内部的for循环后继续执行跳到标签label1处的外部for循环，继续执行。
