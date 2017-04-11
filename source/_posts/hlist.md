---
title: linux内核链表之hlist
tags:
categories:
  - Kernel
  - 内核数据结构
date: 2014-03-08 11:54:51
---

在Linux内核中，hlist（哈希链表）使用非常广泛，本文将对其数据结构和核心函数进行分析。

和hlist相关的数据结构有两个
1. hlist_head
2. hlist_node
```c
 struct hlist_head {
         struct hlist_node *first;
 };

 struct hlist_node {
         struct hlist_node *next, **pprev;
 };
```
顾名思义，`hlist_head`表示哈希表的头结点。哈希表中每一个 entry (hlist_head) 所对应的都是一个链表（hlist)，该链表的结点由`hlist_node`表示。
{% asset_img hlist.jpg %}
`hlist_head` 结构体只有一个域，即`first`。`first`指针指向该 hlist 链表的第一个节点。

`hlist_node`结构体有两个域，`next`和`pprev`。 `next`指针很容易理解，它指向下个`hlist_node`结点，倘若该节点是链表的最后一个节点，`next`指向`NULL`。

`pprev`是一个二级指针，为什么我们需要这样一个指针呢？它的好处是什么？
<!--more-->

在回答这个问题之前，我们先研究另一个问题：为什么散列表的实现需要两个不同的数据结构？

散列表的目的是为了方便快速的查找，所以散列表通常是一个比较大的数组，否则“冲突”的概率会非常大，这样也就失去了散列表的意义。如何做到既能维护一张大表，又能不使用过多的内存呢？这就只能从数据结构上下功夫了。所以对于散列表的每个entry，它的结构体中只存放一个指针，解决了占用空间的问题。现在又出现了另一个问题：数据结构不一致。显然，如果`hlist_node`采用传统的`next`, `prev`指针，对于第一个节点和后面其他节点的处理会不一致。这样并不优雅，而且效率上也有损失。

`hlist_node`巧妙地将`pprev`指向上一个节点的`next`指针的地址，由于`hlist_head`和`hlist_node`指向的下一个节点的指针类型相同，这样就解决了通用性！

下面我们再来看一看`hlist_node`这样设计之后，插入、删除这些基本操作会有什么不一样。
```c
static inline void __hlist_del(struct hlist_node *n)
{
        struct hlist_node *next = n->next;
        struct hlist_node **pprev = n->pprev;
        *pprev = next;
        if (next)
                next->pprev = pprev;
}
```
`__hlist_del`用于删除节点`n`。

首先获取`n`的下一个节点`next`，`n->pprev`指向`n`的前一个节点的`next`指针的地址，这样`*pprev`就代表`n`前一个节点的下一个节点（现在即`n`本身），第三行代码`*pprev=next;`就将`n`的前一个节点和下一个节点关联起来了。至此，`n` 节点的前一个节点的关联工作就完成了，现在再来完成后一个节点的关联工作。如果`n`是链表的最后一个节点，那么`n->next`即为空，则无需任何操作，否则，`next->pprev = pprev`。

给链表增加一个节点需要考虑两个条件：
1. 链表首节点
2. 链表普通节点

```c
static inline void hlist_add_head(struct hlist_node *n, struct hlist_head *h)
 {
         struct hlist_node *first = h->first;
         n->next = first;
         if (first)
                 first->pprev = &n->next;
         h->first = n;
         n->pprev = &h->first;
 }
```
首先讨论条件1：
`first = h->first;`获取当前链表的首个节点；
`n->next = fist;` 将`n`作为链表的首个节点，让`first`往后靠；
先来看最后一行`n->pprev = &h->first;`将`n`的`pprev`指向`hlist_head`的`first`指针，至此关于节点`n`的关联工作就做完了。
再来看倒数第二行`h->first = n;`将节点`h`的关联工作做完；
最后我们再来看原先的第一个节点的关联工作，对于它来说，仅仅需要更新一下`pprev`的关联信息：`first->pprev = &n->next;`

接下来讨论条件2，这里也包括两种情况：
a) 插在当前节点的前面
b) 插在当前节点的后面

```c
/* next must be != NULL */
 static inline void hlist_add_before(struct hlist_node *n,
                                         struct hlist_node *next)
 {
         n->pprev = next->pprev;
         n->next = next;
         next->pprev = &n->next;
         *(n->pprev) = n;
 }
```
先讨论情况 a)：将节点`n`插到`next`之前（`n`是新插入的节点)

还是一个一个节点的搞定（一共三个节点），先搞定节点`n`，

```c
n->pprev = next->prev;
```
将`next`的`pprev`赋值给`n->pprev`，`n`取代`next`的位置，

```c
n->next = next;
```
将`next`作为`n`的下一个节点，至此节点`n`的关联动作完成。

```c
next->pprev = &n->next;
```
`next`的关联动作完成。

```c
*(n->pprev) = n;
```
`n->pprev`表示`n`的前一个节点的next指针，`*(n->pprev)`则表示`n`的前一个节点`next`指针所指向下一个节点的内容，这里将`n`赋值给它，正好完成它的关联工作。

```c
static inline void hlist_add_after(struct hlist_node *n,
                                         struct hlist_node *next)
 {
         next->next = n->next;
         n->next = next;
         next->pprev = &n->next;

         if(next->next)
                 next->next->pprev  = &next->next;
 }
```
再来看情况 b)：将结点`next`插入到n之后(`next`是新插入的节点）
具体步骤就不分析了，应该也很容易。

下面我还要介绍一个函数：
```c
static inline int hlist_unhashed(const struct hlist_node *h)
 {
         return !h->pprev;
 }
```
这个函数的目的是判断该节点是否已经存在hash表中。这里处理得很巧妙，判断指向前一个节点的next指针的地址是否为空。

## 其他接口

**初始化**

```c
#define HLIST_HEAD_INIT { .first = NULL }
#define HLIST_HEAD(name) struct hlist_head name = {  .first = NULL }
#define INIT_HLIST_HEAD(ptr) ((ptr)->first = NULL)
static inline void INIT_HLIST_NODE(struct hlist_node *h)
{
        h->next = NULL;
        h->pprev = NULL;
}
```

**链表判空**

```c
static inline int hlist_empty(const struct hlist_head *h)
{
        return !h->first;
}
```

**节点删除**

```c
static inline void hlist_del(struct hlist_node *n)
{
        __hlist_del(n);
        n->next = LIST_POISON1;
        n->pprev = LIST_POISON2;
}

static inline void hlist_del_init(struct hlist_node *n)
{
        if (!hlist_unhashed(n)) {
                __hlist_del(n);
                INIT_HLIST_NODE(n);
        }
}
```

## 链表遍历

**hlist_entry**

```c
#define hlist_entry(ptr, type, member) container_of(ptr,type,member)

#define hlist_entry_safe(ptr, type, member) \
        ({ typeof(ptr) ____ptr = (ptr); \
           ____ptr ? hlist_entry(____ptr, type, member) : NULL; \
        })
```

**hlist_for_each**

```c
#define hlist_for_each(pos, head) \
        for (pos = (head)->first; pos ; pos = pos->next)

#define hlist_for_each_safe(pos, n, head) \
        for (pos = (head)->first; pos && ({ n = pos->next; 1; }); \
             pos = n)
```

**hlist_for_each_entry**

```c
/**
 * hlist_for_each_entry - iterate over list of given type
 * @pos:        the type * to use as a loop cursor.
 * @head:       the head for your list.
 * @member:     the name of the hlist_node within the struct.
 */
#define hlist_for_each_entry(pos, head, member)                         \
        for (pos = hlist_entry_safe((head)->first, typeof(*(pos)), member);\
             pos;                                                       \
             pos = hlist_entry_safe((pos)->member.next, typeof(*(pos)), member))

/**
 * hlist_for_each_entry_continue - iterate over a hlist continuing after current point
 * @pos:        the type * to use as a loop cursor.
 * @member:     the name of the hlist_node within the struct.
 */
#define hlist_for_each_entry_continue(pos, member)                      \
        for (pos = hlist_entry_safe((pos)->member.next, typeof(*(pos)), member);\
             pos;                                                       \
             pos = hlist_entry_safe((pos)->member.next, typeof(*(pos)), member))

/**
 * hlist_for_each_entry_from - iterate over a hlist continuing from current point
 * @pos:        the type * to use as a loop cursor.
 * @member:     the name of the hlist_node within the struct.
 */
#define hlist_for_each_entry_from(pos, member)                          \
        for (; pos;                                                     \
             pos = hlist_entry_safe((pos)->member.next, typeof(*(pos)), member))

/**
 * hlist_for_each_entry_safe - iterate over list of given type safe against removal of list entry
 * @pos:        the type * to use as a loop cursor.
 * @n:          another &struct hlist_node to use as temporary storage
 * @head:       the head for your list.
 * @member:     the name of the hlist_node within the struct.
 */
#define hlist_for_each_entry_safe(pos, n, head, member)                 \
        for (pos = hlist_entry_safe((head)->first, typeof(*pos), member);\
             pos && ({ n = pos->member.next; 1; });                     \
             pos = hlist_entry_safe(n, typeof(*pos), member))
```

## 参考原文

[linux内核hlist分析](http://blog.csdn.net/zhanglei4214/article/details/6767288)
[内核数据结构：hlist_head ](http://blog.csdn.net/ronliu/article/details/6425368)


