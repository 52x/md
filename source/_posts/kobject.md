---
title: 关于 kobject，kset 和 ktype 的一切
tags:
  - 设备模型
categories:
  - Kernel
  - 内核文档
date: 2014-03-04 15:01:22
---

翻译原文链接：<http://blog.chinaunix.net/uid-20522771-id-3447116.html>
内核文档： Documentation/kobject.txt

> Everything you never wanted to know about kobjects, ksets, and ktypes
> 你永远不会想知道的关于 kobject，kset 和 ktype 的一切
> Greg Kroah-Hartman

Based on an original article by Jon Corbet for lwn.net written October 1,
2003 and located at <http://lwn.net/Articles/51437/>

Last updated December 19, 2007
<!--more-->
理解那些建立在`kobject`抽象之上的驱动模型的困难之一就是没有一个明确的入口。
使用`kobject`需要了解几种不同的类型，而这些类型又会相互引用。为了让这一切变
得简单些，我们将采取多通道方法，以模糊的术语开始，然后逐渐添加细节。为此，
现在给出一些今后会使用到的一些术语的简单定义。

*  `kobject`是一个`struct kobject`类型的对象。`kobject`包含一个名字和一个
   引用计数。同时一个`kobject`还包含一个父指针（允许对象被安排成层次结构）、
   一个特定的类型，通常情况下还有一个在`sysfs`虚拟文件系统里的表现。
   通常我们并不关注`kobject`本身，而应该关注那些嵌入了`kobject`的那些结构体。
   任何结构体都不允许包含一个以上的`kobject`。如果这么做，那么引用计数将肯
   定会出错，你的代码也将bug百出。所以千万别这么做。
*  `ktype`是嵌入了`kobject`的对象的类型。每个嵌入了`kobject`的对象都需要一个
   相应的`ktype`。`ktype`用来控制当`kobject`创建和销毁时所发生的操作。
*  `kset`是`kobject`的一组集合。这些`kobject`可以是同样的`ktype`，也可以分别
   属于不同的`ktype`。`kset`是`kobject`集合的基本容器类型。`kset`也包含它们自
   己的`kobject`，但是你可以放心的忽略这些`kobjects`，因为`kset`的核心代码会
   自动处理这些`kobject`。

   当你看到一个`sysfs`目录里全都是其它目录时，通常每一个目录都对应着一个在
   同一个`kset`里的`kobject`。

我们来看看如何创建和操作所有的这些类型。因为采用自下而上的方法，所以我们
首先回到`kobject`。

`kobject`结构定义如下：
```c
struct kobject {
        const char              *name;
        struct list_head        entry;
        struct kobject          *parent;
        struct kset             *kset;
        struct kobj_type        *ktype;
        struct sysfs_dirent     *sd;
        struct kref             kref;
        .
        .
        .
};
```

## 内嵌 kobject

就内核代码而言，基本上不会创建一个单独的`kobject`，但也有例外，这以后再说。
其实，`kobject`通常被用来控制一个更大的特定域对象。因此你将发现`kobject`都被
嵌入到了其他的结构体当中。如果从面向对象的观点出发，那么`kobject`可以被看做
是被其他类继承的、顶层的、抽象的类。一个`kobject`实现了一组对于它们自己不是
很有用，但对那些包含了这个`kobject`的对象很有用的功能。另外 C 语言不支持直接
使用继承，所以必须依靠其他的技术来实现，比如结构体嵌套。

（顺便说一句，对于那些熟悉内核链表实现的同志来说，这和“list_head”结构体很
类似。它们都是本身没什么用，其真正的价值是在嵌套进一个更大的结构体中才得以
体现。）

举一个例子，drivers/uio/uio.c 里的 UIO 代码包含一个定义了内存区域的 uio 设备。
```c
struct uio_map {
    struct kobject kobj;
    struct uio_mem *mem;
};
```
如果你有一个`struct uio_map`结构体，使用它的成员`kobj`就能找到嵌套的`kobject`。
但是操作`kobject`的代码往往会引出一个相反的问题：如果给定一个`struct kobject`
指针，那么包含这个指针的结构体又是什么呢？别投机取巧（比如假设这个`kobject`
是该结构体的第一个字段），你应该使用`container_of()`宏函数：
```c
container_of(pointer, type, member)
```
其中：

*   "`pointer`" 是指向被嵌入的`kobject`的指针。
*   "`type`" 是包含`kobject`的结构体类型。
*   "`member`" 是结构体中"`pointer`"所指向的字段名。
`container_of()`的返回值就是一个相应结构体的指针。例如，一个指向嵌套在`uio_map
`里的`struct kobject`的指针“`kp`”，可以这样获得包含它的结构体的指针：
```c
struct uio_map *u_map = container_of(kp, struct uio_map, kobj);
```
为方便起见，程序员通常会定义一个简单的宏，用于“反指”包含这个`kobject`的容器
类型指针。正因为这样，在以前的文件 drivers/uio/uio.c 中你可以看到：
```c
struct uio_map {
    struct kobject kobj;
    struct uio_mem *mem;
};

#define to_map(map) container_of(map, struct uio_map, kobj)
```
宏参数“`map`”是一个指向`struct kobject`的指针。这个宏函数将随后被调用：
```c
struct uio_map *map = to_map(kobj);
```

## kobject 的初始化

创建一个`kobject`的代码首先必须初始化这个对象。调用`kobject_init()`来设置一些
内部字段（强制性的）：
```c
 void kobject_init(struct kobject *kobj, struct kobj_type *ktype);
 ```
因为每一个`kobject`都有一个关联的`kobj_type`，所以正确创建一个`kobject`时
`ktype`是必须的。调用`kobject_init()`之后，必须是用`kobject_add()`在`sysfs`上
注册`kobject`。
```c
int kobject_add(struct kobject *kobj, struct kobject *parent, const char *fmt, ...);
```
这将为`kobject`设置`parent`和`name`。如果这个`kobject`将关联于一个指定的
`kset`，那么`kobj->kset`必须在`kobject_add()`之前被赋值。如果一个`ket`关联
于一个`kobject`，那么在调用`kobject_add()`时`parent`可以是`NULL`，这时`kobject`
的`parent`将会是这个`kset`本身。

当一个`kobject`的`name`已经被设置并添加至`kernel`之后，就不允许直接操作这个
`kobject`的`name`了。如果你必须修改这个`name`，请使用`kobject_rename()`:
```c
int kobject_rename(struct kobject *kobj, const char *new_name);
```
`kobject_rename`不会执行任何锁定操作，也不会验证`name`的有效性，所以调用者
必须提供自己的完整性检查和序列化。

这里有一个函数`kobject_set_name()`，但这个函数是遗留问题，而且在将来会被删
除。如果你的代码需要调用这个函数，那么这将是不正确的并且必须被修正。

要正确访问`kobject`的`name`，使用这个函数`kobject_name()`:
```c
const char *kobject_name(const struct kobject * kobj);
```

这里有一个辅助函数将同时初始化`kobject`并将其添加至 kernel，`kobject_init_and_add()`:
```c
int kobject_init_and_add(struct kobject *kobj, struct kobj_type *ktype,
                         struct kobject *parent, const char *fmt, ...);
```

其中的参数与`kobject_init()`和`kobject_add()`里描述的一致。

## Uevents

在一个`kobject`注册到核心(core)之后，你需要通过`kobject_uevent()`向系统宣布它被创建了。
```c
int kobject_uevent(struct kobject *kobj, enum kobject_action action);
```

当`kobject`首次被添加进 kernel 时，使用`KOBJ_ADD`动作。这个调用必须在`kobject`所有
的`attributes`或`children`都被正确初始化之后，因为当这个调用发生时，用户空间将会
立即开始寻找它们。

当`kobject`从 kernel 中移除时，`KOBJ_REMOVE`的`uevent`将会被`kobject`核心(core)
自动创建，所以调用者不必担心手动完成这些。

## 引用计数

一个`kobject`的主要功能之一就是在它被嵌入的对象中作为一个引用计数器。只要存
在对该对象的引用，对象（和支持它的代码）就必须继续存在。操作一个`kobject`的
引用计数的底层函数是：
```c
struct kobject *kobject_get(struct kobject *kobj);
void kobject_put(struct kobject *kobj);
```
对`kobject_get()`的成功调用将递增`kobject`的引用计数并且返回指向该`kobject`的指针。

当一个引用被释放时，调用`kobject_put()`将递减引用计数，并且可能的话，释放对象。
请注意，`kobject_init()`将引用计数设置为 1，所以在建立`kobject`的代码里需要执行
`kobject_put()`用于最终释放该引用。

因为`kobject`是动态的，所以它们不能被声明为静态的或者在栈区分配空间，它们
应该始终被动态分配。未来的 kernel 版本将会包含一个对`kobject`是否静态创建的
运行时检查，并且会警告开发人员这种不正确的使用。

如果你使用`kobject`的理由仅仅是使用引用计数的话，那么请使用`struct kref`替代
`kobject`。更多关于`struct kref`的信息请参考 Linux 内核文档 Documentation/kref.txt

## 创建“简单”的 kobject

有时开发人员希望有一种创建一个在 sysfs 层次中简单目录的方式，而并不想搞乱本来
就错综复杂的`ksets`，`show` 和`store`函数，或一些其他的细节。这是一个创建单独的
`kobjects`的一个例外。要创建这样的条目，使用这个函数：
```c
struct kobject *kobject_create_and_add(char *name, struct kobject *parent);
```

此函数将创建一个`kobject`，并将其放置在指定的父`kobject`在 sysfs 中的目录下。
要创建简单的与这个`kobject`关联的属性，使用函数：
```c
int sysfs_create_file(struct kobject *kobj, struct attribute *attr);
```
或者
```c
int sysfs_create_group(struct kobject *kobj, struct attribute_group *grp);
```

这两种使用`kobject_create_and_add()`创建的`kobject`的属性类型都可以是
`kobj_attribute`，所以没有创建自定义属性的需要。

查看示例模块 samples/kobject/kobject-example.c，一个简单的`kobject`
及其属性的实现。

## ktypes 和 release 方法

到目前为止我们遗漏了一个重要的事情，那就是当一个`kobject`的引用计数达到 0 时将
会发生什么。通常，创建`kobject`的代码并不知道这种情况何时发生。如果它们知道何
时发生，那么把`kobject`放在结构体的首位可能会有那么一点点帮助。即使是一个可预
测的对象生命周期也将变得复杂，特别是在 sysfs 作为 kernel 的一部分被引入时，它
可以获取到任意在系统中注册的`kobject`的引用。

结论就是一个被`kobject`保护的结构体不能在这个`kobject`引用计数到 0 之前被释放。
而这个引用计数又不被创建这个`kobject`的代码所直接控制，所以必须在这个`kobject`
的最后一个引用消失时异步通知这些代码。

一旦你通过`kobject_add()` 注册你的`kobject`之后，永远也不要使用`kfree()`去释
放直接它。唯一安全的途径是使用`kobject_put()`。总是在`kobject_init()`之后使
用`kobject_put()`是避免错误蔓延的很好的做法。

这个通知是通过`kobject`的`release()`函数完成的。该函数通常具有这样的形式：
```c
void my_object_release(struct kobject *kobj)
{
   struct my_object *mine = container_of(kobj, struct my_object, kobj);

   kfree(mine);
}
```
很重要的一点怎么强调也不过分：每个`kobject`必须有一个`release()`方法，而且
在这个方法被调用之前`kobject`必须继续存在（保持一致的状态）。如果不符合这
些限制，那么代码是有缺陷的。需要注意的是，如果你忘记提供一个`release()`方法，
kernel 会警告你。不要试图提供一个“空”的`release`函数来摆脱这个警告，如果你
这样做你会受到`kobject`维护者们无情的嘲笑。

注意，尽管在`release`函数中`kobject`的`name`是可用的，但是千万不要在这个回调函
数中修改它。否则将会在`kobject`核心(core)中发生令人不愉快的内存泄露问题。

有趣的是`release()`方法并没有保存在`kobject`之中，而是关联在它的`ktype`
成员中。让我们来介绍`struct kobj_type`:
```c
struct kobj_type {
    void (*release)(struct kobject *);
    const struct sysfs_ops *sysfs_ops;
    struct attribute **default_attrs;
};
```

这个结构体是用来描述一个特定类型的`kobject`（或者更确切的说，包含它的对象）。
每个`kobject`都需要一个关联的`kobj_type`结构体，当你调用`kobject_init()`或
`kobject_init_and_add()`时必须指定一个指向`kobj_type`结构体的指针。

当然`struct kobj_type`中的`release`字段就是一个指向同类型对象`release()`方法的
函数指针。另外两个字段（`sysfs_ops`和`default_attrs`）是用来控制这些类型的对象在
sysfs 里是如何表现的，这已经超出了本文的讨论范围。

`default_attrs`成员指针是一个在任何属于这个`ktype`的`kobject`注册时自动创
建的默认属性列表。

## ksets

`kset`仅仅是一个需要相互关联的`kobject`集合。在这里没有任何规定它们必须是同
样的`ktype`，但如果它们不是一样的`ktype`，则一定要小心处理。

一个`kset`提供以下功能：

*   它就像一个装有一堆对象袋子。`kset`可以被 kernel 用来跟踪像“所有的块设备”
   或者“所有的 PCI 设备驱动”这样的东西。
*   一个`kset`也是一个 sysfs 里的子目录，该目录里能够看见这些相关的`kobject`。
   每个`kset`都包含一个`kobject`，这个`kobject`可以用来设置成其他`kobjects`
   的`parent`。sysfs 层次结构中的顶层目录就是通过这样的方法构建的。
*  `kset`还可以支持`kobject`的“热插拔”，并会影响`uevent`事件如何报告给用户空间。

以面向对象的观点来看，“`kset`” 是一个顶层容器类。`kset`包含有它们自己的`kobject`，
这个`kobject`是在`kset`代码管理之下的，而且不允许其他任何用户对其操作。

一个`kset`使用标准的 kernel 链表来保存它的`children`。`kobjects`通过它们的`kset`
字段回指向包含它们的`kset`。在几乎所有的情况下，属于某`kset`的`kobject`的
`parent`都指向这个`kset`（严格来说是嵌套进这个`kset`的`kobject`）。

正是因为一个`kset`包含了一个`kobject`，就应该始终动态创建这个`kset`，千万
不要将其声明为静态的或在栈区分配空间。创建一个`kset`使用：
```c
struct kset *kset_create_and_add(const char *name,
                                 struct kset_uevent_ops *u,
                                 struct kobject *parent);
```
当你使用完一个`kset`时，调用这个函数：
```c
void kset_unregister(struct kset *kset);
```
来销毁它。

内核树中的 samples/kobject/kset-example.c 文件是一个
有关于如何使用`kset`的示例。

如果一个`kset`希望控制那些与它相关联的`kobject`的`uevent`操作，可以使用
`struct kset_uevent_ops`处理。
```c
struct kset_uevent_ops {
    int (*filter)(struct kset *kset, struct kobject *kobj);
    const char *(*name)(struct kset *kset, struct kobject *kobj);
    int (*uevent)(struct kset *kset, struct kobject *kobj,
                  struct kobj_uevent_env *env);
};
```

`filter`函数允许`kset`阻止一个特定的`kobject`的`uevent`是否发送到用户空间。如果
这个函数返回 0，则`uevent`将不会被发出。

`name`函数用来重写那些将被`uevent`发送到用户空间的`kset`的默认名称。默认情况下，
这个名称应该和`kset`本身的名称相同，但如果提供这个函数，则可以改写这个名称。

`uevent`函数会向即将被发送到用户空间的`uevent`添加更多的环境变量。

有人可能会问，当一个`kobject`加入到一个`kset` 但没有提供完成这些功能的函数时，情况会怎样？
答案就是这些任务将由`kobject_add()`来完成，当一个`kobject`传递给`kobject_add()`函数，它的`kset`成员必将指向它将被
加入到的`kset`，然后`kobject_add()`会帮你干完剩下的活。

如果一个属于某个`kset`的`kobject`没有设置它的`parent kobject`，那么它将被
添加到`kset`的目录中去。但并不是`kset`的所有成员都一定存在于`kset`目录下。
如果在`kobject`被添加前就指明了它的`parent kobject`， 那么该`kobject`将被
注册到这个`kset`下，然后添加到它的`parent kobject`下。

## kobject 的移除

当一个`kobject`成功的注册到`kobject`核心(core)之后，这个`kobject`必须在代码完成对它的
使用时销毁。要做到这一点，请调用`kobject_put()`。通过这个调用，`kobject` 核心(core)会
自动清理所有通过这个`kobject`分配的内存空间。如果曾经为了这个`kobject`发送过一个
`KOBJ_ADD` uevent，那么一个相应的`KOBJ_REMOVE` uevent 将会被发送，并且任何
其他的 sysfs 维护者都将为这个调用作出相应的处理。

如果你需要分两步来删除一个`kobject`的话（也就是说在你需要销毁一个对象时不允
许 sleep），那么请使用`kobject_del()`从 sysfs 注销这个`kobject`。这将使得该
`kobject` “不可见”，但是它并没有被清理，并且它的引用计数也没变。在稍后的时候
调用`kobject_put()`去完成与这个`kobject`相关的内存空间的清理。

如果建立了循环引用，`kobject_del()`可以用来删除指向`parent`对象的引用。
一些情况下，某个`parnet`对象引用了它的`child`是合法的，循环引用必须
显式的通过调用`kobject_del()` 来打破，这样做之后将会调用一个`release`函数，
先前在循环引用中的对象才会彼此释放。

## 示例代码

想要获得更完整的关于正确使用`kset`和`kobject`的例子，请参考示例程序
samples/kobject/{kobject-example.c,kset-example.c}。可以通过选择编译条件
`CONFIG_SAMPLE_KOBJECT`来把这些示例编译成可装载模块。

samples/kobject/kobject-example.c
```c
/*
 * Sample kobject implementation
 *
 * Copyright (C) 2004-2007 Greg Kroah-Hartman <greg@kroah.com>
 * Copyright (C) 2007 Novell Inc.
 *
 * Released under the GPL version 2 only.
 *
 */
#include <linux/kobject.h>
#include <linux/string.h>
#include <linux/sysfs.h>
#include <linux/module.h>
#include <linux/init.h>

/*
 * This module shows how to create a simple subdirectory in sysfs called
 * /sys/kernel/kobject-example  In that directory, 3 files are created:
 * "foo", "baz", and "bar".  If an integer is written to these files, it can be
 * later read out of it.
 */

static int foo;
static int baz;
static int bar;

/*
 * The "foo" file where a static variable is read from and written to.
 */
static ssize_t foo_show(struct kobject *kobj, struct kobj_attribute *attr,
			char *buf)
{
	return sprintf(buf, "%d\n", foo);
}

static ssize_t foo_store(struct kobject *kobj, struct kobj_attribute *attr,
			 const char *buf, size_t count)
{
	sscanf(buf, "%du", &foo);
	return count;
}

static struct kobj_attribute foo_attribute =
	__ATTR(foo, 0666, foo_show, foo_store);

/*
 * More complex function where we determine which variable is being accessed by
 * looking at the attribute for the "baz" and "bar" files.
 */
static ssize_t b_show(struct kobject *kobj, struct kobj_attribute *attr,
		      char *buf)
{
	int var;

	if (strcmp(attr->attr.name, "baz") == 0)
		var = baz;
	else
		var = bar;
	return sprintf(buf, "%d\n", var);
}

static ssize_t b_store(struct kobject *kobj, struct kobj_attribute *attr,
		       const char *buf, size_t count)
{
	int var;

	sscanf(buf, "%du", &var);
	if (strcmp(attr->attr.name, "baz") == 0)
		baz = var;
	else
		bar = var;
	return count;
}

static struct kobj_attribute baz_attribute =
	__ATTR(baz, 0666, b_show, b_store);
static struct kobj_attribute bar_attribute =
	__ATTR(bar, 0666, b_show, b_store);

/*
 * Create a group of attributes so that we can create and destroy them all
 * at once.
 */
static struct attribute *attrs[] = {
	&foo_attribute.attr,
	&baz_attribute.attr,
	&bar_attribute.attr,
	NULL,	/* need to NULL terminate the list of attributes */
};

/*
 * An unnamed attribute group will put all of the attributes directly in
 * the kobject directory.  If we specify a name, a subdirectory will be
 * created for the attributes with the directory being the name of the
 * attribute group.
 */
static struct attribute_group attr_group = {
	.attrs = attrs,
};

static struct kobject *example_kobj;

static int __init example_init(void)
{
	int retval;

	/*
	 * Create a simple kobject with the name of "kobject_example",
	 * located under /sys/kernel/
	 *
	 * As this is a simple directory, no uevent will be sent to
	 * userspace.  That is why this function should not be used for
	 * any type of dynamic kobjects, where the name and number are
	 * not known ahead of time.
	 */
	example_kobj = kobject_create_and_add("kobject_example", kernel_kobj);
	if (!example_kobj)
		return -ENOMEM;

	/* Create the files associated with this kobject */
	retval = sysfs_create_group(example_kobj, &attr_group);
	if (retval)
		kobject_put(example_kobj);

	return retval;
}

static void __exit example_exit(void)
{
	kobject_put(example_kobj);
}

module_init(example_init);
module_exit(example_exit);
MODULE_LICENSE("GPL");
MODULE_AUTHOR("Greg Kroah-Hartman <greg@kroah.com>");
```

samples/kobject/kset-example.c
```c
/*
 * Sample kset and ktype implementation
 *
 * Copyright (C) 2004-2007 Greg Kroah-Hartman <greg@kroah.com>
 * Copyright (C) 2007 Novell Inc.
 *
 * Released under the GPL version 2 only.
 *
 */
#include <linux/kobject.h>
#include <linux/string.h>
#include <linux/sysfs.h>
#include <linux/slab.h>
#include <linux/module.h>
#include <linux/init.h>

/*
 * This module shows how to create a kset in sysfs called
 * /sys/kernel/kset-example
 * Then tree kobjects are created and assigned to this kset, "foo", "baz",
 * and "bar".  In those kobjects, attributes of the same name are also
 * created and if an integer is written to these files, it can be later
 * read out of it.
 */

/*
 * This is our "object" that we will create a few of and register them with
 * sysfs.
 */
struct foo_obj {
	struct kobject kobj;
	int foo;
	int baz;
	int bar;
};
#define to_foo_obj(x) container_of(x, struct foo_obj, kobj)

/* a custom attribute that works just for a struct foo_obj. */
struct foo_attribute {
	struct attribute attr;
	ssize_t (*show)(struct foo_obj *foo, struct foo_attribute *attr, char *buf);
	ssize_t (*store)(struct foo_obj *foo, struct foo_attribute *attr, const char *buf, size_t count);
};
#define to_foo_attr(x) container_of(x, struct foo_attribute, attr)

/*
 * The default show function that must be passed to sysfs.  This will be
 * called by sysfs for whenever a show function is called by the user on a
 * sysfs file associated with the kobjects we have registered.  We need to
 * transpose back from a "default" kobject to our custom struct foo_obj and
 * then call the show function for that specific object.
 */
static ssize_t foo_attr_show(struct kobject *kobj,
			     struct attribute *attr,
			     char *buf)
{
	struct foo_attribute *attribute;
	struct foo_obj *foo;

	attribute = to_foo_attr(attr);
	foo = to_foo_obj(kobj);

	if (!attribute->show)
		return -EIO;

	return attribute->show(foo, attribute, buf);
}

/*
 * Just like the default show function above, but this one is for when the
 * sysfs "store" is requested (when a value is written to a file.)
 */
static ssize_t foo_attr_store(struct kobject *kobj,
			      struct attribute *attr,
			      const char *buf, size_t len)
{
	struct foo_attribute *attribute;
	struct foo_obj *foo;

	attribute = to_foo_attr(attr);
	foo = to_foo_obj(kobj);

	if (!attribute->store)
		return -EIO;

	return attribute->store(foo, attribute, buf, len);
}

/* Our custom sysfs_ops that we will associate with our ktype later on */
static const struct sysfs_ops foo_sysfs_ops = {
	.show = foo_attr_show,
	.store = foo_attr_store,
};

/*
 * The release function for our object.  This is REQUIRED by the kernel to
 * have.  We free the memory held in our object here.
 *
 * NEVER try to get away with just a "blank" release function to try to be
 * smarter than the kernel.  Turns out, no one ever is...
 */
static void foo_release(struct kobject *kobj)
{
	struct foo_obj *foo;

	foo = to_foo_obj(kobj);
	kfree(foo);
}

/*
 * The "foo" file where the .foo variable is read from and written to.
 */
static ssize_t foo_show(struct foo_obj *foo_obj, struct foo_attribute *attr,
			char *buf)
{
	return sprintf(buf, "%d\n", foo_obj->foo);
}

static ssize_t foo_store(struct foo_obj *foo_obj, struct foo_attribute *attr,
			 const char *buf, size_t count)
{
	sscanf(buf, "%du", &foo_obj->foo);
	return count;
}

static struct foo_attribute foo_attribute =
	__ATTR(foo, 0666, foo_show, foo_store);

/*
 * More complex function where we determine which variable is being accessed by
 * looking at the attribute for the "baz" and "bar" files.
 */
static ssize_t b_show(struct foo_obj *foo_obj, struct foo_attribute *attr,
		      char *buf)
{
	int var;

	if (strcmp(attr->attr.name, "baz") == 0)
		var = foo_obj->baz;
	else
		var = foo_obj->bar;
	return sprintf(buf, "%d\n", var);
}

static ssize_t b_store(struct foo_obj *foo_obj, struct foo_attribute *attr,
		       const char *buf, size_t count)
{
	int var;

	sscanf(buf, "%du", &var);
	if (strcmp(attr->attr.name, "baz") == 0)
		foo_obj->baz = var;
	else
		foo_obj->bar = var;
	return count;
}

static struct foo_attribute baz_attribute =
	__ATTR(baz, 0666, b_show, b_store);
static struct foo_attribute bar_attribute =
	__ATTR(bar, 0666, b_show, b_store);

/*
 * Create a group of attributes so that we can create and destroy them all
 * at once.
 */
static struct attribute *foo_default_attrs[] = {
	&foo_attribute.attr,
	&baz_attribute.attr,
	&bar_attribute.attr,
	NULL,	/* need to NULL terminate the list of attributes */
};

/*
 * Our own ktype for our kobjects.  Here we specify our sysfs ops, the
 * release function, and the set of default attributes we want created
 * whenever a kobject of this type is registered with the kernel.
 */
static struct kobj_type foo_ktype = {
	.sysfs_ops = &foo_sysfs_ops,
	.release = foo_release,
	.default_attrs = foo_default_attrs,
};

static struct kset *example_kset;
static struct foo_obj *foo_obj;
static struct foo_obj *bar_obj;
static struct foo_obj *baz_obj;

static struct foo_obj *create_foo_obj(const char *name)
{
	struct foo_obj *foo;
	int retval;

	/* allocate the memory for the whole object */
	foo = kzalloc(sizeof(*foo), GFP_KERNEL);
	if (!foo)
		return NULL;

	/*
	 * As we have a kset for this kobject, we need to set it before calling
	 * the kobject core.
	 */
	foo->kobj.kset = example_kset;

	/*
	 * Initialize and add the kobject to the kernel.  All the default files
	 * will be created here.  As we have already specified a kset for this
	 * kobject, we don't have to set a parent for the kobject, the kobject
	 * will be placed beneath that kset automatically.
	 */
	retval = kobject_init_and_add(&foo->kobj, &foo_ktype, NULL, "%s", name);
	if (retval) {
		kobject_put(&foo->kobj);
		return NULL;
	}

	/*
	 * We are always responsible for sending the uevent that the kobject
	 * was added to the system.
	 */
	kobject_uevent(&foo->kobj, KOBJ_ADD);

	return foo;
}

static void destroy_foo_obj(struct foo_obj *foo)
{
	kobject_put(&foo->kobj);
}

static int __init example_init(void)
{
	/*
	 * Create a kset with the name of "kset_example",
	 * located under /sys/kernel/
	 */
	example_kset = kset_create_and_add("kset_example", NULL, kernel_kobj);
	if (!example_kset)
		return -ENOMEM;

	/*
	 * Create three objects and register them with our kset
	 */
	foo_obj = create_foo_obj("foo");
	if (!foo_obj)
		goto foo_error;

	bar_obj = create_foo_obj("bar");
	if (!bar_obj)
		goto bar_error;

	baz_obj = create_foo_obj("baz");
	if (!baz_obj)
		goto baz_error;

	return 0;

baz_error:
	destroy_foo_obj(bar_obj);
bar_error:
	destroy_foo_obj(foo_obj);
foo_error:
	kset_unregister(example_kset);
	return -EINVAL;
}

static void __exit example_exit(void)
{
	destroy_foo_obj(baz_obj);
	destroy_foo_obj(bar_obj);
	destroy_foo_obj(foo_obj);
	kset_unregister(example_kset);
}

module_init(example_init);
module_exit(example_exit);
MODULE_LICENSE("GPL");
MODULE_AUTHOR("Greg Kroah-Hartman <greg@kroah.com>");
```
