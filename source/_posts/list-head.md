---
title: linux内核链表之list_head
tags:
categories:
  - Kernel
  - 内核数据结构
date: 2014-03-07 16:58:28
---

在linux内核中，有一种通用的双向循环链表，构成了各种队列的基础。链表的结构定义和相关函数均在include/linux/list.h中。

在阅读`list.h`文件之前，有一点必须注意：linux内核中的链表使用方法和一般数据结构中定义的链表是有所不同的。

一般的双向链表一般是如下的结构，
*   有个单独的头结点(`head`)
*   每个节点(`node`)除了包含必要的数据之外，还有2个指针(`pre`,`next`)
*   `pre`指针指向前一个节点(`node`)，`next`指针指向后一个节点(`node`)
*   头结点(`head`)的`pre`指针指向链表的最后一个节点
*   最后一个节点的`next`指针指向头结点(`head`)

具体见下图：
{% asset_img dlist.png %}
<!--more-->

传统的链表有个最大的缺点就是不好共通化，因为每个`node`中的`data1`，`data2`等等都是不确定的(无论是个数还是类型)。

linux中的链表巧妙的解决了这个问题，linux的链表不是将用户数据保存（嵌入）在链表节点中，而是将链表节点保存（嵌入）在用户数据中。

linux的链表节点只有2个指针(`pre`和`next`)，这样的话，链表的节点将独立于用户数据之外，便于实现链表的共同操作。

具体见下图：
{% asset_img dlist_kernel.png %}

linux链表中的最大问题是怎样通过链表的节点来取得用户数据？
和传统的链表不同，linux的链表节点(node)中没有包含用户的用户`data1`，`data2`等。

下面就来全面的介绍这一链表的各种API。

## 声明和初始化

```c
struct list_head {
	struct list_head *next, *prev;
};
```
这是链表的元素结构。因为是循环链表，表头和节点都是这一结构。有`prev`和`next`两个指针域，分别指向链表中前一节点和后一节点。

```c
#define LIST_HEAD_INIT(name) { &(name), &(name) }

#define LIST_HEAD(name) \
	struct list_head name = LIST_HEAD_INIT(name)

static inline void INIT_LIST_HEAD(struct list_head *list)
{
	list->next = list;
	list->prev = list;
}
```
初始化链表，链表头的`prev`和`next`都指向自身。

## 链表修改

**插入**
```c
static inline void __list_add(struct list_head *new,
			      struct list_head *prev,
			      struct list_head *next)
{
	next->prev = new;
	new->next = next;
	new->prev = prev;
	prev->next = new;
}

static inline void list_add(struct list_head *new, struct list_head *head)
{
	__list_add(new, head, head->next);
}

static inline void list_add_tail(struct list_head *new, struct list_head *head)
{
	__list_add(new, head->prev, head);
}
```
双向循环链表的实现，很少有例外情况，基本都可以用公共的方式来处理。这里无论是加第一个节点，还是其它的节点，使用的方法都一样。

另外，链表API实现时大致都是分为两层：一层外部的，如`list_add`、`list_add_tail`，用来消除一些例外情况，调用内部实现；一层是内部的，函数名前会加双下划线，如`__list_add`，往往是几个操作公共的部分，或者排除例外后的实现。

**删除**
```c
static inline void __list_del(struct list_head * prev, struct list_head * next)
{
	next->prev = prev;
	prev->next = next;
}

static inline void list_del(struct list_head *entry)
{
	__list_del(entry->prev, entry->next);
	entry->next = LIST_POISON1;
	entry->prev = LIST_POISON2;
}

static inline void list_del_init(struct list_head *entry)
{
	__list_del(entry->prev, entry->next);
	INIT_LIST_HEAD(entry);
}
```
`list_del`是链表中节点的删除。之所以在调用`__list_del`后又把被删除元素的`next`、`prev`指向特殊的`LIST_POSITION1`和`LIST_POSITION2`，是为了调试未定义的指针。

`list_del_init`则是删除节点后，随即把节点中指针再次初始化，这种删除方式更为实用。

**替换**
```c
static inline void list_replace(struct list_head *old,
				struct list_head *new)
{
	new->next = old->next;
	new->next->prev = new;
	new->prev = old->prev;
	new->prev->next = new;
}

static inline void list_replace_init(struct list_head *old,
					struct list_head *new)
{
	list_replace(old, new);
	INIT_LIST_HEAD(old);
}
```
`list_replace`是将链表中一个节点`old`，替换为另一个节点`new`。从实现来看，即使`old`所在地链表只有`old`一个节点，`new`也可以成功替换，这就是双向循环链表可怕的通用之处。

`list_replace_init`将被替换的`old`随即又初始化。

**搬移**
```c
static inline void list_move(struct list_head *list, struct list_head *head)
{
	__list_del(list->prev, list->next);
	list_add(list, head);
}

static inline void list_move_tail(struct list_head *list,
				  struct list_head *head)
{
	__list_del(list->prev, list->next);
	list_add_tail(list, head);
}
```
`list_move`的作用是把`list`节点从原链表中去除，并加入新的链表`head`中。

`list_move_tail`只在加入新链表时与`list_move`有所不同，`list_move`是加到<span style="color:red;">head之后</span>的链表<span style="color:red;">头部</span>，而`list_move_tail`是加到<span style="color:red;">head之前</span>的链表<span style="color:red;">尾部</span>。

**拆分**
```c
static inline void __list_cut_position(struct list_head *list,
		struct list_head *head, struct list_head *entry)
{
	struct list_head *new_first = entry->next;
	list->next = head->next;
	list->next->prev = list;
	list->prev = entry;
	entry->next = list;
	head->next = new_first;
	new_first->prev = head;
}

static inline void list_cut_position(struct list_head *list,
		struct list_head *head, struct list_head *entry)
{
	if (list_empty(head))
		return;
	if (list_is_singular(head) &&
		(head->next != entry && head != entry))
		return;
	if (entry == head)
		INIT_LIST_HEAD(list);
	else
		__list_cut_position(list, head, entry);
}
```
`list_cut_position`用于把`head`链表分为两个部分。`head`链表中的`head->next`一直到`entry`（包括`entry`）被删除，并加入到新链表`list`的头部。新链表`list`应该是空的，或者原来的节点都可以被忽略掉。可以看到，`list_cut_position`中排除了一些意外情况，保证调用`__list_cut_position`时至少有一个元素会被加入到新链表`list`。

**合并**
```c
static inline void __list_splice(const struct list_head *list,
				 struct list_head *prev,
				 struct list_head *next)
{
	struct list_head *first = list->next;
	struct list_head *last = list->prev;

	first->prev = prev;
	prev->next = first;

	last->next = next;
	next->prev = last;
}

static inline void list_splice(const struct list_head *list,
				struct list_head *head)
{
	if (!list_empty(list))
		__list_splice(list, head, head->next);
}

static inline void list_splice_tail(struct list_head *list,
				struct list_head *head)
{
	if (!list_empty(list))
		__list_splice(list, head->prev, head);
}
```
`list_splice`的功能和`list_cut_position`正相反，它合并两个链表。`list_splice`把`list`链表中的节点加入`head`链表中。在实际操作之前，要先判断`list`链表是否为空。它保证调用`__list_splice`时`list`链表中至少有一个节点可以被合并到`head`链表中。

`list_splice_tail`只是在合并链表时插入的位置不同。`list_splice`是把原来`list`链表中的节点全加到`head`链表的头部，而`list_splice_tail`则是把原来`list`链表中的节点全加到head链表的尾部。
```c
static inline void list_splice_init(struct list_head *list,
				    struct list_head *head)
{
	if (!list_empty(list)) {
		__list_splice(list, head, head->next);
		INIT_LIST_HEAD(list);
	}
}

static inline void list_splice_tail_init(struct list_head *list,
					 struct list_head *head)
{
	if (!list_empty(list)) {
		__list_splice(list, head->prev, head);
		INIT_LIST_HEAD(list);
	}
}
```
`list_splice_init`除了完成`list_splice`的功能，还把变空了的`list`链表头重新初始化。

`list_splice_tail_init`除了完成`list_splice_tail`的功能，还把变空了得`list`链表头重新初始化。

list操作的API大致如以上所列，包括链表节点添加与删除、节点从一个链表转移到另一个链表、链表中一个节点被替换为另一个节点、链表的合并与拆分、查看链表当前是否为空或者只有一个节点。

## 链表判断

**表尾**
```c
static inline int list_is_last(const struct list_head *list,
				const struct list_head *head)
{
	return list->next == head;
}
```
`list_is_last`判断`list`是否处于`head`链表的尾部。

**空链表**
```c
static inline int list_empty(const struct list_head *head)
{
	return head->next == head;
}

static inline int list_empty_careful(const struct list_head *head)
{
	struct list_head *next = head->next;
	return (next == head) && (next == head->prev);
}
```
`list_empty`判断`head`链表是否为空，为空的意思就是只有一个链表头`head`。

`list_empty_careful`同样是判断`head`链表是否为空，只是检查更为严格。

**单节点**
```c
static inline int list_is_singular(const struct list_head *head)
{
	return !list_empty(head) && (head->next == head->prev);
}
```
`list_is_singular`判断`head`中是否只有一个节点，即除链表头`head`外只有一个节点。

## 链表遍历

遍历是链表最经常的操作之一，为了方便核心应用遍历链表，Linux链表将遍历操作抽象成几个宏。在介绍遍历宏之前，我们先看看如何从链表中访问到我们真正需要的数据项。

**list_entry**

我们知道，Linux 链表中仅保存了数据项结构中 `list_head` 成员变量的地址，那么我们如何通过这个`list_head`成员访问到作为它的所有者的节点数据呢？ Linux 为此提供了一个`list_entry(ptr,type,member)`宏，其中`ptr`是指向该数据中`list_head`成员的指针，也就是存储在链表中的地址值，`type`是数据项的类型，`member`则是数据项类型定义中`list_head`成员的变量名，例如，我们要访问`nf_sockopts`链表中首个`nf_sockopt_ops`变量，则如此调用：
```c
list_entry(nf_sockopts->next, struct nf_sockopt_ops, list);
```
这里`list`正是`nf_sockopt_ops`结构中定义的用于链表操作的节点成员变量名。

`list_entry`的使用相当简单，相比之下，它的实现则有一些难懂：
```c
#define list_entry(ptr, type, member) \
	container_of(ptr, type, member)
```
`container_of`宏定义在[include/linux/kernel.h]中：
```c
#define container_of(ptr, type, member) ({			\
        const typeof( ((type *)0)->member ) *__mptr = (ptr);	\
        (type *)( (char *)__mptr - offsetof(type,member) );})
```
`offsetof`宏定义在[include/linux/stddef.h]中：
```c
#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)
```
`size_t`最终定义为`unsigned int`（注：i386平台）。

这里使用的是一个利用编译器技术的小技巧，即先求得结构成员在与结构中的偏移量，然后根据成员变量的地址反过来得出属主结构变量的地址。

`container_of()`和`offsetof()`并不仅用于链表操作，这里最有趣的地方是`((type *)0)->member`，它将`0`地址强制"转换"为`type`结构的指针，再访问到`type`结构中的`member`成员。在`container_of`宏中，它用来给`typeof()`提供参数（`typeof()`是gcc的扩展，和`sizeof()`类似 ），以获`member`成员的数据类型；在`offsetof()`中，这个`member`成员的地址实际上就是`type` 数据结构中`member`成员相对于结构变量的偏移量。

如果这么说还不好理解的话，不妨看看下面这张图：`offsetof()`宏的原理
{% asset_img offsetof.gif %}

对于给定一个结构，`offsetof(type,member)`是一个常量，`list_entry()`正是利用这个不变的偏移量来求得链表数据项的变量地址。
```c
#define list_first_entry(ptr, type, member) \
	list_entry((ptr)->next, type, member)
```
`list_first_entry`是将`ptr`看完一个链表的链表头，取出其中第一个节点对应的结构地址。使用`list_first_entry`是应保证链表中至少有一个节点。

**list_for_each**
```c
#define list_for_each(pos, head) \
	for (pos = (head)->next; prefetch(pos->next), pos != (head); \
        	pos = pos->next)
```
`list_for_each`循环遍历链表中的每个节点，从链表头部的第一个节点，一直到链表尾部。中间的`prefetch`是为了利用平台特性加速链表遍历，在某些平台下定义为空，可以忽略。
```c
#define __list_for_each(pos, head) \
	for (pos = (head)->next; pos != (head); pos = pos->next)
```
`__list_for_each`与`list_for_each`没什么不同，只是少了`prefetch`的内容，实现上更为简单易懂。
```c
#define list_for_each_prev(pos, head) \
	for (pos = (head)->prev; prefetch(pos->prev), pos != (head); \
        	pos = pos->prev)
```
`list_for_each_prev`与`list_for_each`的遍历顺序相反，从链表尾逆向遍历到链表头。

**list_for_each_safe**
```c
#define list_for_each_safe(pos, n, head) \
	for (pos = (head)->next, n = pos->next; pos != (head); \
		pos = n, n = pos->next)
```
`list_for_each_safe`也是链表顺序遍历，只是更加安全。即使在遍历过程中，当前节点从链表中删除，也不会影响链表的遍历。参数上需要加一个暂存的链表节点指针`n`。
```c
#define list_for_each_prev_safe(pos, n, head) \
	for (pos = (head)->prev, n = pos->prev; \
	     prefetch(pos->prev), pos != (head); \
	     pos = n, n = pos->prev)
```
`list_for_each_prev_safe`与`list_for_each_prev`同样是链表逆序遍历，只是加了链表节点删除保护。

**list_for_each_entry**
```c
#define list_for_each_entry(pos, head, member)				\
	for (pos = list_entry((head)->next, typeof(*pos), member);	\
	     prefetch(pos->member.next), &pos->member != (head); 	\
	     pos = list_entry(pos->member.next, typeof(*pos), member))
```
`list_for_each_entry`是遍历链表节点，而是遍历链表节点所有者的数据结构。这个实现上较为复杂，但可以等价于 <span style="color:red;">list_for_each</span> 加上 <span style="color:red;">list_entry</span> 的组合。
```c
#define list_for_each_entry_reverse(pos, head, member)			\
	for (pos = list_entry((head)->prev, typeof(*pos), member);	\
	     prefetch(pos->member.prev), &pos->member != (head); 	\
	     pos = list_entry(pos->member.prev, typeof(*pos), member))
```
`list_for_each_entry_reverse`是逆序遍历链表节点所有者的数据结构，等价于 `list_for_each_prev`加上`list_etnry`的组合。
```c
#define list_for_each_entry_continue(pos, head, member) 		\
	for (pos = list_entry(pos->member.next, typeof(*pos), member);	\
	     prefetch(pos->member.next), &pos->member != (head);	\
	     pos = list_entry(pos->member.next, typeof(*pos), member))
```
`list_for_each_entry_continue`也是遍历链表上的节点所有者的数据结构。只是并非从链表头开始，而是从结构指针的下一个结构开始，一直到链表尾部。
```c
#define list_for_each_entry_continue_reverse(pos, head, member)		\
	for (pos = list_entry(pos->member.prev, typeof(*pos), member);	\
	     prefetch(pos->member.prev), &pos->member != (head);	\
	     pos = list_entry(pos->member.prev, typeof(*pos), member))
```
`list_for_each_entry_continue_reverse`是逆序遍历链表上的节点所有者的数据结构。只是并非从链表尾开始，而是从结构指针的前一个结构开始，一直到链表头部。
```c
#define list_for_each_entry_from(pos, head, member) 			\
	for (; prefetch(pos->member.next), &pos->member != (head);	\
	     pos = list_entry(pos->member.next, typeof(*pos), member))
```
`list_for_each_entry_from`是从当前结构指针pos开始，顺序遍历链表上的结构指针。

**list_for_each_entry_safe**
```c
#define list_for_each_entry_safe(pos, n, head, member)			\
	for (pos = list_entry((head)->next, typeof(*pos), member),	\
		n = list_entry(pos->member.next, typeof(*pos), member);	\
	     &pos->member != (head); 					\
	     pos = n, n = list_entry(n->member.next, typeof(*n), member))
```
`list_for_each_entry_safe`也是顺序遍历链表上节点所有者的数据结构。只是加了删除节点的保护。
```c
#define list_for_each_entry_safe_continue(pos, n, head, member) 		\
	for (pos = list_entry(pos->member.next, typeof(*pos), member), 		\
		n = list_entry(pos->member.next, typeof(*pos), member);		\
	     &pos->member != (head);						\
	     pos = n, n = list_entry(n->member.next, typeof(*n), member))
```
`list_for_each_entry_safe_continue`是从pos的下一个结构指针开始，顺序遍历链表上的结构指针，同时加了节点删除保护。
```c
#define list_for_each_entry_safe_from(pos, n, head, member) 			\
	for (n = list_entry(pos->member.next, typeof(*pos), member);		\
	     &pos->member != (head);						\
	     pos = n, n = list_entry(n->member.next, typeof(*n), member))
```
`list_for_each_entry_safe_from`是从pos开始，顺序遍历链表上的结构指针，同时加了节点删除保护。
```c
#define list_for_each_entry_safe_reverse(pos, n, head, member)		\
	for (pos = list_entry((head)->prev, typeof(*pos), member),	\
		n = list_entry(pos->member.prev, typeof(*pos), member);	\
	     &pos->member != (head); 					\
	     pos = n, n = list_entry(n->member.prev, typeof(*n), member))
```
`list_for_each_entry_safe_reverse`是从pos的前一个结构指针开始，逆序遍历链表上的结构指针，同时加了节点删除保护。

至此为止，我们介绍了linux中双向循环链表的结构、所有的操作函数和遍历宏定义。相信以后在linux代码中遇到链表的使用，不会再陌生。

## 参考原文

[linux内核部件分析（一）——连通世界的list ](http://blog.csdn.net/qb_2008/article/details/6839230)
[深入分析 Linux 内核链表](https://www.ibm.com/developerworks/cn/linux/kernel/l-chain/)
[《Linux内核设计与实现》读书笔记（六）- 内核数据结构](http://www.cnblogs.com/wang_yb/archive/2013/04/16/3023892.html)

