---
title: 记录内核对象生命周期的kref
tags:
  - 设备模型
categories:
  - Kernel
  - 内核文档
date: 2014-03-07 10:40:14
---

参考原文链接：[linux内核部件分析（三）——记录生命周期的kref](http://blog.csdn.net/qb_2008/article/details/6840387)
另外关于kref的使用可参考：
[Linux内核里的“智能指针”](http://www.cnblogs.com/wwang/archive/2010/12/02/1894847.html)
[Linux内核里的“智能指针” (续)](http://www.cnblogs.com/wwang/archive/2010/12/03/1895852.html)

`kref`是一个引用计数器，它被嵌套进其它的结构中，记录所嵌套结构的引用计数，并在计数为0时调用相应的清理函数。`kref`的原理和实现都非常简单，但要想用好却不容易，或者说`kref`被创建就是为了跟踪复杂情况下结构的引用和销毁。所以这里先介绍`kref`的实现，再介绍其使用规则。
<!--more-->
### kref 的定义实现

`kref`的头文件在`include/linux/kref.h`，实现在`lib/kref.c`。

`kref`的定义非常简单，其结构体里只有一个原子变量。
```c
struct kref {
    atomic_t refcount;
};
```
Linux内核定义了下面三个函数接口来使用`kref`：
```c
void kref_init(struct kref *kref);
void kref_get(struct kref *kref);
int kref_put(struct kref *kref, void (*release) (struct kref *kref));
```

`kref_init` 初始化`kref`的计数值为1。
```c
/**
 * kref_init - initialize object.
 * @kref: object in question.
 */
void kref_init(struct kref *kref)
{
	atomic_set(&kref->refcount,1);
}
```

`kref_get` 递增`kref`的计数值。
```c
/**
 * kref_get - increment refcount for object.
 * @kref: object.
 */
void kref_get(struct kref *kref)
{
	WARN_ON(!atomic_read(&kref->refcount));
	atomic_inc(&kref->refcount);
}
```

`kref_put` 递减`kref`的计数值，如果计数值减为0，说明`kref`所指向的结构生命周期结束，会执行`release`释放函数。
```c
/**
 * kref_put - decrement refcount for object.
 * @kref: object.
 * @release: pointer to the function that will clean up the object when the
 *	     last reference to the object is released.
 *	     This pointer is required, and it is not acceptable to pass kfree
 *	     in as this function.
 *
 * Decrement the refcount, and if 0, call release().
 * Return 1 if the object was removed, otherwise return 0\.  Beware, if this
 * function returns 0, you still can not count on the kref from remaining in
 * memory.  Only use the return value if you want to see if the kref is now
 * gone, not present.
 */
int kref_put(struct kref *kref, void (*release)(struct kref *kref))
{
	WARN_ON(release == NULL);
	WARN_ON(release == (void (*)(struct kref *))kfree);

	if (atomic_dec_and_test(&kref->refcount)) {
		release(kref);
		return 1;
	}
	return 0;
}
```

### kref 的使用

`kref`设计得如此简单，是为了能灵活地用在各种结构的生命周期管理中。要用好它可不简单，好在Documentation/kref.txt中为我们总结了一些使用规则，下面简单翻译一下。

对于那些用在多种场合，被到处传递的结构，如果没有引用计数，bug几乎总是肯定的事。所以我们需要`kref`，`kref`允许我们在已有的结构中方便地添加引用计数。

你可以以如下方式添加`kref`到你的数据结构中：
```c
struct my_data {
    ...
    struct kref refcount;
    ...
};
```
`kref`可以出现在你结构中的任意位置。

在分配`kref`后你必须初始化它，可以调用`kref_init`，把`kref`计数值初始为1。
```c
struct my_data *data;

data = kmalloc(sizeof(*data), GFP_KERNEL);
if(!data)
    return -ENOMEM;
kref_init(&data->refcount);
```

初始化之后，`kref`的使用应该遵循以下三条规则：

1） 如果你创建了一个结构指针的非暂时性副本，特别是当这个副本指针会被传递到其它执行线程时，你必须在传递副本指针之前执行kref_get：
```c
kref_put(&data->refcount);
```
2）当你使用完，不再需要结构的指针，必须执行kref_put。如果这是结构指针的最后一个引用，release函数将会被调用。如果代码绝不会在没有拥有引用计数的请求下去调用kref_get，在kref_put时就不需要加锁。
```c
kref_put(&data->refcount, data_release);
```
3）如果代码试图在还没拥有引用计数的情况下就调用kref_get，就必须串行化kref_put和kref_get的执行。因为很可能在kref_get执行之前或者执行中，kref_put就被调用并把整个结构释放掉了。

例如，你分配了一些数据并把它传递到其它线程去处理：
```c
void data_release(struct kref *kref)
{
    struct my_data *data = container_of(kref, struct my_data, refcount);
    kree(data);
}

void more_data_handling(void *cb_data)
{
    struct my_data *data = cb_data;
    .
    .  do stuff with data here
    .
    kref_put(&data->refcount, data_release);
}

int my_data_handler(void)
{
    int rv = 0;
    struct my_data *data;
    struct task_struct *task;
    data = kmalloc(sizeof(*data), GFP_KERNEL);
     if (!data)
        return -ENOMEM;
    kref_init(&data->refcount);
    kref_get(&data->refcount);
    task = kthread_run(more_data_handling, data, "more_data_handling");
    if (task == ERR_PTR(-ENOMEM)){
         rv = -ENOMEM;
         goto out;
    }
    .
    .  do stuff with data here
    .
out:
    kref_put(&data->refcount, data_release);
    return rv;
}
```
这样做，无论两个线程的执行顺序是怎样的都无所谓，`kref_put`知道何时数据不再有引用计数，可以被销毁。`kref_get()`调用不需要加锁，因为在`my_data_handler`中调用`kref_get`时已经拥有一个引用。同样地原因，`kref_put`也不需要加锁。

要注意规则一中的要求，必须在传递指针之前调用`kref_get`。决不能写下面的代码：
```c
task = kthread_run(more_data_handling, data, "more_data_handling");
if(task == ERR_PTR(-ENOMEM)) {
    rv = -ENOMEM;
    goto out;
}
else {
     /* BAD BAD BAD - get is after the handoff */
    kref_get(&data->refcount);
```
不要认为自己在使用上面的代码时知道自己在做什么。首先，你可能并不知道你在做什么。其次，你可能知道你在做什么（在部分加锁情况下上面的代码也是正确的），但一些修改或者复制你代码的人并不知道你在做什么。这是一种坏的使用方式。

当然在部分情况下也可以优化对`get`和`put`的使用。例如，你已经完成了对这个数据的处理，并要把它传递给其它线程，就不需要再做多余的`get`和`put`了。
```c
/* Silly extra get and put */
kref_get(&obj->ref);
enqueue(obj);
kref_put(&obj->ref, obj_cleanup);
```

只需要做enqueue操作即可，可以在其后加一条注释。
```c
enqueue(obj);
/* We are done with obj , so we pass our refcount off to the queue. DON'T TOUCH obj AFTER HERE! */
```

第三条规则是处理起来最麻烦的。例如，你有一列数据，每条数据都有`kref`计数，你希望获取第一条数据。但你不能简单地把第一条数据从链表中取出并调用`kref_get`。这违背了第三条，在调用`kref_get`前你并没有一个引用。你需要增加一个`mutex`（或者其它锁）。
```c
static DEFINE_MUTEX(mutex);
static LIST_HEAD(q);
struct my_data
{
	struct kref refcount;
	struct list_head link;
};

static struct my_data *get_entry()
{
	struct my_data *entry = NULL;
	mutex_lock(&mutex);
	if(!list_empty(&q)){
		entry = container_of(q.next, struct my_q_entry, link);
		kref_get(&entry->refcount);
	}
	mutex_unlock(&mutex);
	return entry;
}

static void release_entry(struct kref *ref)
{
	struct my_data *entry = container_of(ref, struct my_data, refcount);

	list_del(&entry->link);
	kfree(entry);
}

static void put_entry(struct my_data *entry)
{
	mutex_lock(&mutex);
	kref_put(&entry->refcount, release_entry);
	mutex_unlock(&mutex);
}
```

如果你不想在整个释放过程中都加锁，`kref_put`的返回值就有用了。例如你不想在加锁情况下调用`kfree`，你可以如下使用`kref_put`。
```c
static void release_entry(struct kref *ref)
{
        /* All work is done after the return from kref_put(). */
}

static void put_entry(struct my_data *entry)
{
        mutex_lock(&mutex);
        if (kref_put(&entry->refcount, release_entry)) {
                list_del(&entry->link);
                mutex_unlock(&mutex);
                kfree(entry);
        } else
                mutex_unlock(&mutex);
}
```
如果你在撤销结构的过程中需要调用其它的需要较长时间的函数，或者函数也可能要获取同样地互斥锁，这样做就很有用了。但要注意在`release`函数中做完撤销工作会使代码看起来更整洁。
