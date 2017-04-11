---
title: linux内核链表之klist
tags:
categories:
  - Kernel
  - 内核数据结构
date: 2014-03-09 12:47:25
---

`klist`是`list`的线程安全版本，它提供了整个链表的自旋锁，查找链表节点，对链表节点的插入和删除操作都要获得这个自旋锁。`klist`的节点数据结构是`klist_node`，`klist_node`引入引用计数，只有当引用计数减到0时才允许该`node`从链表中移除。当一个内核线程要移除一个`node`，必须要等待到`node`的引用计数释放，在此期间线程处于休眠状态。为了方便线程等待，`klist`引入等待移除节点者结构体`klist_waiter`，`klist_waiter`组成`klist_remove_waiters`（内核全局变量）链表，为保护`klist_remove_waiters`线程安全，引入`klist_remove_lock`（内核全局变量）自旋锁。为方便遍历`klist`，引入了迭代器`klist_iter`。
<!--more-->

## 数据结构及接口声明

include/linux/klist.h：
```c
struct klist_node;
struct klist {
        spinlock_t              k_lock;
        struct list_head        k_list;
        void                    (*get)(struct klist_node *);
        void                    (*put)(struct klist_node *);
} __attribute__ ((aligned (sizeof(void *))));

#define KLIST_INIT(_name, _get, _put)                                   \
        { .k_lock       = __SPIN_LOCK_UNLOCKED(_name.k_lock),           \
          .k_list       = LIST_HEAD_INIT(_name.k_list),                 \
          .get          = _get,                                         \
          .put          = _put, }

#define DEFINE_KLIST(_name, _get, _put)                                 \
        struct klist _name = KLIST_INIT(_name, _get, _put)

extern void klist_init(struct klist *k, void (*get)(struct klist_node *),
                       void (*put)(struct klist_node *));

struct klist_node {
        void                    *n_klist;       /* never access directly */
        struct list_head        n_node;
        struct kref             n_ref;
};

extern void klist_add_tail(struct klist_node *n, struct klist *k);
extern void klist_add_head(struct klist_node *n, struct klist *k);
extern void klist_add_after(struct klist_node *n, struct klist_node *pos);
extern void klist_add_before(struct klist_node *n, struct klist_node *pos);

extern void klist_del(struct klist_node *n);
extern void klist_remove(struct klist_node *n);

extern int klist_node_attached(struct klist_node *n);

struct klist_iter {
        struct klist            *i_klist;
        struct klist_node       *i_cur;
};

extern void klist_iter_init(struct klist *k, struct klist_iter *i);
extern void klist_iter_init_node(struct klist *k, struct klist_iter *i,
                                 struct klist_node *n);
extern void klist_iter_exit(struct klist_iter *i);
extern struct klist_node *klist_next(struct klist_iter *i);
```

**klist_node**

`klist_node`有个`dead`字段，联合在`n_klist`指针中，`n_klist`只能默认指向`klist`。`klist`的定义中，`__attribute__ ((aligned (sizeof(void *))))`要求按`void`指针类型大小进行内存字节对齐，这意味着`klist`实例的地址低2位（x86_32）或者低3位（x86_64）总是0，这些为0的低位刚好可以作为其他用处，对该指针解引用前需与掉相应位。来看源码：
```c
/*
 * Use the lowest bit of n_klist to mark deleted nodes and exclude
 * dead ones from iteration.
 */
#define KNODE_DEAD		1LU
#define KNODE_KLIST_MASK	~KNODE_DEAD

static struct klist *knode_klist(struct klist_node *knode)
{
	return (struct klist *)
		((unsigned long)knode->n_klist & KNODE_KLIST_MASK);
}

static bool knode_dead(struct klist_node *knode)
{
	return (unsigned long)knode->n_klist & KNODE_DEAD;
}

static void knode_set_klist(struct klist_node *knode, struct klist *klist)
{
	knode->n_klist = klist;
	/* no knode deserves to start its life dead */
	WARN_ON(knode_dead(knode));
}

static void knode_kill(struct klist_node *knode)
{
	/* and no knode should die twice ever either, see we're very humane */
	WARN_ON(knode_dead(knode));
	*(unsigned long *)&knode->n_klist |= KNODE_DEAD;
}
```
为什么要引入这个dead标识呢？

如果一个内核线程要让某个`node`无效，不能简单的从`klist`中把`node`摘下来，只能减少`node`的引用计数，但是由于其他内核线程也拥有该`node`的引用计数，所以节点还是在`klist`链中，遍历节点等操作时无法避开该`node`。引入这个标识后，只要设置这个标识，尽管该`node`还在`klist`链上，但是迭代操作的时候通过这个标识避开`dead`的节点。这样在该节点上不会有新的操作，通过链表遍历也无法获取到该节点，当其他内核线程不再引用该`node`后，该`node`自动从`klist`链中移除。所以`dead`的作用是禁止再使用该`node`，但是已经被人家在用的还是可以继续使用。调用`klist_del()`会标示该`node`为`dead`，并最终删除节点。

前面的四个函数都是内部静态函数，用于辅助API实现：
`knode_klist`是从节点找到链表头，`knode_dead`是检查该节点是否已被请求删除，
`knode_set_klist`设置节点的链表头，`knode_kill`将请求删除一个节点。
细心的话大家会发现这四个函数是对称的，而且都是操作节点的内部函数。

**klist 的定义和初始化**

常规定义：
```c
struct klist myklist;
klist_init(&myklist,get,put);
```
便捷宏方式定义：
```c
DEFINE_KLIST(myklist,get,put);//定义一个myklist，并初始化
```
```c
/**
 * klist_init - Initialize a klist structure.
 * @k: The klist we're initializing.
 * @get: The get function for the embedding object (NULL if none)
 * @put: The put function for the embedding object (NULL if none)
 *
 * Initialises the klist structure.  If the klist_node structures are
 * going to be embedded in refcounted objects (necessary for safe
 * deletion) then the get/put arguments are used to initialise
 * functions that take and release references on the embedding
 * objects.
 */
void klist_init(struct klist *k, void (*get)(struct klist_node *),
                void (*put)(struct klist_node *))
{
        INIT_LIST_HEAD(&k->k_list);
        spin_lock_init(&k->k_lock);
        k->get = get;
        k->put = put;
}
EXPORT_SYMBOL_GPL(klist_init);
```
注意函数注释中对get/put参数（函数指针）的说明。

**插入节点**
```c
struct klist_node mynode;
klist_add_tail(&mynode,&mylist);
```
`klist_add_xxx`函数初始化`node`，并将其插入链表，插入链表后，引用计数为1
`klist_add_tail`向后插入，`klist_add_head`向前插入；
`kilst_add_after`在某个节点的后面插入，`klist_add_before`在某个节点的前面插入。
```c
static void add_head(struct klist *k, struct klist_node *n)
{
	spin_lock(&k->k_lock);
	list_add(&n->n_node, &k->k_list);
	spin_unlock(&k->k_lock);
}

static void add_tail(struct klist *k, struct klist_node *n)
{
	spin_lock(&k->k_lock);
	list_add_tail(&n->n_node, &k->k_list);
	spin_unlock(&k->k_lock);
}

static void klist_node_init(struct klist *k, struct klist_node *n)
{
	INIT_LIST_HEAD(&n->n_node);
	kref_init(&n->n_ref);
	knode_set_klist(n, k);
	if (k->get)
		k->get(n);
}
```
三个内部函数，`add_head`将节点加入链表头，`add_tail`将节点加入链表尾，`klist_node_init`是初始化节点。
```c
/**
 * klist_add_head - Initialize a klist_node and add it to front.
 * @n: node we're adding.
 * @k: klist it's going on.
 */
void klist_add_head(struct klist_node *n, struct klist *k)
{
        klist_node_init(k, n);
        add_head(k, n);
}
EXPORT_SYMBOL_GPL(klist_add_head);

/**
 * klist_add_tail - Initialize a klist_node and add it to back.
 * @n: node we're adding.
 * @k: klist it's going on.
 */
void klist_add_tail(struct klist_node *n, struct klist *k)
{
        klist_node_init(k, n);
        add_tail(k, n);
}
EXPORT_SYMBOL_GPL(klist_add_tail);
```
`klist_add_head`将节点初始化，并将其加入链表头，`klist_add_tail`将节点初始化，并将其加入链表尾。
```c
/**
 * klist_add_after - Init a klist_node and add it after an existing node
 * @n: node we're adding.
 * @pos: node to put @n after
 */
void klist_add_after(struct klist_node *n, struct klist_node *pos)
{
        struct klist *k = knode_klist(pos);

        klist_node_init(k, n);
        spin_lock(&k->k_lock);
        list_add(&n->n_node, &pos->n_node);
        spin_unlock(&k->k_lock);
}
EXPORT_SYMBOL_GPL(klist_add_after);

/**
 * klist_add_before - Init a klist_node and add it before an existing node
 * @n: node we're adding.
 * @pos: node to put @n after
 */
void klist_add_before(struct klist_node *n, struct klist_node *pos)
{
        struct klist *k = knode_klist(pos);

        klist_node_init(k, n);
        spin_lock(&k->k_lock);
        list_add_tail(&n->n_node, &pos->n_node);
        spin_unlock(&k->k_lock);
}
EXPORT_SYMBOL_GPL(klist_add_before);
```
`klist_add_after`将节点加到指定节点后面，`klist_add_before`将节点加到指定节点前面。

**删除节点**
```c
klist_del(&mynode);
```
`klist_del`调用`klist_put`，减少引用计数，并设`dead`标记，当引用计数减为0时，自动调用`klist_release`，把节点从`klist`中删除。
```c
klist_remove(&mynode);
```
`klist_remove`把当前线程加入等待移除链表，减少引用计数，如果有其他内核线程占用引用计数，把当前线程休眠。

`klist`是怎么迫使`remove`某个`node`的线程休眠的，又是怎么唤醒的？为了方便进程管理，引入了`klist_waiter`结构，如下：
```c
struct klist_waiter {
        struct list_head list;
        struct klist_node *node; //等待删除的node
        struct task_struct *process; //进程或者线程指针
        int woken; //唤醒标记
};

static DEFINE_SPINLOCK(klist_remove_lock);
static LIST_HEAD(klist_remove_waiters);
```
来看使线程进入休眠的代码，klist_remove函数：
```c
/**
 * klist_remove - Decrement the refcount of node and wait for it to go away.
 * @n: node we're removing.
 */
void klist_remove(struct klist_node *n)
{
        struct klist_waiter waiter;  //创建一个waiter

        waiter.node = n;
        waiter.process = current;
        waiter.woken = 0;
        spin_lock(&klist_remove_lock);  //锁住klist_remove_lock，
                                            //klist_remove_lock专门是用来保护
                                            //klist_remove_waiters的
        list_add(&waiter.list, &klist_remove_waiters);  //把waiter加入到klist_remove_waiters中
                                                            //这里把一个局部变量加入到一个全局的链表结构中，
                                                            //会不会引起内存越界后续讨论
        spin_unlock(&klist_remove_lock);

        klist_del(n);  //减少引用计数并判死刑

        for (;;) {
                set_current_state(TASK_UNINTERRUPTIBLE);  //设置进程进入休眠状态
                if (waiter.woken)
                        break;
                schedule();  //调度进程时当前进程进入休眠状态
        }
        __set_current_state(TASK_RUNNING);
}
EXPORT_SYMBOL_GPL(klist_remove);
```
`klist_del`函数调用`klist_put`调用`klist_dec_and_del`调用`kref_put`，`kref_put`当引用计数减到0时回调`klist_release`函数，`klist_release`会释放等待者。
进程的休眠与唤醒是`klist_remove`和`klist_release`共同作用的结果，我们来看`klist_release`的源码：
```c
static void klist_release(struct kref *kref)
{
        struct klist_waiter *waiter, *tmp;
        struct klist_node *n = container_of(kref, struct klist_node, n_ref);

        WARN_ON(!knode_dead(n));  //要释放的节点一定是被判死刑的节点
        list_del(&n->n_node);     //把node从klist移除
        spin_lock(&klist_remove_lock);  //保护klist_remove_waiters
        /* 遍历klist_remove_waiter */
        list_for_each_entry_safe(waiter, tmp, &klist_remove_waiters, list) {
                if (waiter->node != n)
                        continue;

                list_del(&waiter->list);  //把waiter结构体从klist_remove_waiters中移除
                waiter->woken = 1;  //设唤醒标示
                mb();
                wake_up_process(waiter->process);  //唤醒该进程
        }
        spin_unlock(&klist_remove_lock);
        knode_set_klist(n, NULL);
}

static int klist_dec_and_del(struct klist_node *n)
{
        return kref_put(&n->n_ref, klist_release);
}

static void klist_put(struct klist_node *n, bool kill)
{
        struct klist *k = knode_klist(n);
        void (*put)(struct klist_node *) = k->put;

        spin_lock(&k->k_lock);
        if (kill)
                knode_kill(n);
        if (!klist_dec_and_del(n))
                put = NULL;
        spin_unlock(&k->k_lock);
        if (put)
                put(n);
}

/**
 * klist_del - Decrement the reference count of node and try to remove.
 * @n: node we're deleting.
 */
void klist_del(struct klist_node *n)
{
        klist_put(n, true);
}
EXPORT_SYMBOL_GPL(klist_del);
```
`klist_remove`函数把局部变量加入到全局链表中，但是由于`klist_remove`会使线程休眠，它返回前总是由`klist_release`把`waiter`从`klist_remove_waiters`移走，所以不会导致崩溃。其实`klist_remove_waiters`的链表节点实际都是一些内核栈中的`waiter`结构，这些线程都休眠在`klist_remove`中。

**遍历klist**

`klist`没有像`list`一样定义一系列的`list_for_each_xxx`宏。`klist`提供专门的迭代器结构提`klist_iter`，遍历前首先要初始化迭代器 ，`klist_next()`函数把当前迭代器向后移动，并返回移动后的`node`。`klist_iter_init()`，`klist_iter_init_node()`都是迭代器初始化函数，前者把迭代器当前位置设置为`NULL`，`klist_next()`自动从第一个`node`开始，后者可以指定当前`node`。迭代器指向`node`时会增加`node`的引用计数，当迭代器不用时必须调用`klist_iter_exit`退出迭代器，释放当前`node`的引用计数。
```c
/**
 * klist_node_attached - Say whether a node is bound to a list or not.
 * @n: Node that we're testing.
 */
int klist_node_attached(struct klist_node *n)
{
        return (n->n_klist != NULL);
}
EXPORT_SYMBOL_GPL(klist_node_attached);
```
`klist_node_attached`检查节点是否被包含在某链表中。
```c
/**
 * klist_iter_init_node - Initialize a klist_iter structure.
 * @k: klist we're iterating.
 * @i: klist_iter we're filling.
 * @n: node to start with.
 *
 * Similar to klist_iter_init(), but starts the action off with @n,
 * instead of with the list head.
 */
void klist_iter_init_node(struct klist *k, struct klist_iter *i,
                          struct klist_node *n)
{
        i->i_klist = k;
        i->i_cur = n;
        if (n)
                kref_get(&n->n_ref);
}
EXPORT_SYMBOL_GPL(klist_iter_init_node);

/**
 * klist_iter_init - Iniitalize a klist_iter structure.
 * @k: klist we're iterating.
 * @i: klist_iter structure we're filling.
 *
 * Similar to klist_iter_init_node(), but start with the list head.
 */
void klist_iter_init(struct klist *k, struct klist_iter *i)
{
        klist_iter_init_node(k, i, NULL);
}
EXPORT_SYMBOL_GPL(klist_iter_init);
```
`klist_iter_init_node`是从`klist`中的某个节点开始遍历，而`klist_iter_init`是从链表头开始遍历的。要注意，`klist_iter_init`和`klist_iter_init_node`的用法不同。`klist_iter_init_node`可以在其后直接对当前节点进行访问，也可以调用`klist_next`访问下一节点，而`klist_iter_init`只能调用`klist_next`访问下一节点。或许，`klist_iter_init_node`的本意不是从当前节点开始，而是从当前节点的下一节点开始。
```c
/**
 * klist_iter_exit - Finish a list iteration.
 * @i: Iterator structure.
 *
 * Must be called when done iterating over list, as it decrements the
 * refcount of the current node. Necessary in case iteration exited before
 * the end of the list was reached, and always good form.
 */
void klist_iter_exit(struct klist_iter *i)
{
        if (i->i_cur) {
                klist_put(i->i_cur, false);
                i->i_cur = NULL;
        }
}
EXPORT_SYMBOL_GPL(klist_iter_exit);
```
`klist_iter_exit`遍历结束函数。在遍历完成时调不调无所谓，但如果想中途结束，就一定要调用`klist_iter_exit`。
```c
static struct klist_node *to_klist_node(struct list_head *n)
{
        return container_of(n, struct klist_node, n_node);
}

/**
 * klist_next - Ante up next node in list.
 * @i: Iterator structure.
 *
 * First grab list lock. Decrement the reference count of the previous
 * node, if there was one. Grab the next node, increment its reference
 * count, drop the lock, and return that next node.
 */
struct klist_node *klist_next(struct klist_iter *i)
{
        void (*put)(struct klist_node *) = i->i_klist->put;
        struct klist_node *last = i->i_cur;
        struct klist_node *next;

        spin_lock(&i->i_klist->k_lock);

        if (last) {
                next = to_klist_node(last->n_node.next);
                if (!klist_dec_and_del(last))
                        put = NULL;
        } else
                next = to_klist_node(i->i_klist->k_list.next);

        i->i_cur = NULL;
        while (next != to_klist_node(&i->i_klist->k_list)) {
                if (likely(!knode_dead(next))) {
                        kref_get(&next->n_ref);
                        i->i_cur = next;
                        break;
                }
                next = to_klist_node(next->n_node.next);
        }

        spin_unlock(&i->i_klist->k_lock);

        if (put && last)
                put(last);
        return i->i_cur;
}
EXPORT_SYMBOL_GPL(klist_next);
```
`klist_next`是将循环进行到下一节点，实现中需要注意两点问题：
1. 加锁，根据经验，单纯对某个节点操作不需要加锁，但对影响整个链表的操作需要加自旋锁。比如之前klist_iter_init_node中对节点增加引用计数，就不需要加锁，因为只有已经拥有节点引用计数的线程才会特别地从那个节点开始。而之后klist_next中则需要加锁，因为当前线程很可能没有引用计数，所以需要加锁，让情况固定下来。这既是保护链表，也是保护节点有效。符合kref引用计数的使用原则。
2. 要注意，虽然在节点切换的过程中是加锁的，但切换完访问当前节点时是解锁的，中间可能有节点被删除，也可能有节点被请求删除，这就需要注意。首先要忽略链表中已被请求删除的节点，然后在减少前一个节点引用计数时，可能就把前一个节点删除了。这里之所以不调用klist_put，是因为本身已处于加锁状态，但仍要有它的实现。这里的实现和klist_put中类似，代码不介意在加锁状态下唤醒另一个线程，但却不希望在加锁状态下调用put函数，那可能会涉及释放另一个更大的结构。

## 参考

[linux内核部件分析（四）——更强的链表klist](http://blog.csdn.net/qb_2008/article/details/6845854)
[linux内核源代码分析----内核基础设施之klist](http://www.cnblogs.com/httpftpli/archive/2012/07/24/klist.html)

