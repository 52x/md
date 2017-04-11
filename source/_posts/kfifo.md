---
title: linux内核环形队列kfifo
tags:
categories:
  - Kernel
  - 内核数据结构
date: 2014-03-29 23:07:35
---

Linux kernel 里面从来就不缺少简洁，优雅和高效的代码，只是我们缺少发现和品味的眼光。在Linux kernel里面，简洁并不表示代码使用神出鬼没的超然技巧，相反，它使用的不过是大家非常熟悉的基础数据结构，但是kernel开发者能从基础的数据结构中，提炼出优美的特性。

`kfifo`就是这样的一类优美代码，它十分简洁，绝无多余的一行代码，却非常高效。
关于kfifo信息如下：
<span style="color:blue;">
本文分析的原代码版本：2.6.25
kfifo的定义文件：kernel/kfifo.c
kfifo的头文件：include/linux/kfifo.h
</span>
<!--more-->

## kfifo概述

`kfifo`是内核里面的一个First In First Out数据结构，它采用环形循环队列的数据结构来实现；它提供一个无边界的字节流服务，最重要的一点是，它使用并行无锁编程技术，即当它用于只有一个入队线程和一个出队线程的场情时，两个线程可以并发操作，而不需要任何加锁行为，就可以保证`kfifo`的线程安全。

`kfifo`代码既然肩负着这么多特性，那我们先一瞥它的代码：
```c
struct kfifo {
        unsigned char *buffer;  /* the buffer holding the data */
        unsigned int size;      /* the size of the allocated buffer */
        unsigned int in;        /* data is added at offset (in % size) */
        unsigned int out;       /* data is extracted from off. (out % size) */
        spinlock_t *lock;       /* protects concurrent modifications */
};
```
这是`kfifo`的数据结构，`kfifo`主要提供了两个操作，`__kfifo_put`(入队操作)和`__kfifo_get`(出队操作)。 它的各个数据成员如下：
`buffer` 用于存放数据的缓存
`size`   buffer空间的大小，在初化时，将它向上扩展成2的幂
`lock`   如果使用不能保证任何时间最多只有一个读线程和写线程，需要使用该lock实施同步。
`in`,`out`和`buffer`一起构成一个循环队列。 `in`指向`buffer`中队头，而且`out`指向`buffer`中的队尾，它的结构如示图如下：
```
+--------------------------------------------------------------+
|           |<----------data---------->|                       |
+--------------------------------------------------------------+
            ^                          ^
            |                          |
            in                         out
```
当然，内核开发者使用了一种更好的技术处理了`in`,`out`和`buffer`的关系，我们将在下面进行详细的分析。

## kfifo 内存分配和初始化工作

**kfifo_alloc**
```c
/**
 * kfifo_alloc - allocates a new FIFO and its internal buffer
 * @size: the size of the internal buffer to be allocated.
 * @gfp_mask: get_free_pages mask, passed to kmalloc()
 * @lock: the lock to be used to protect the fifo buffer
 *
 * The size will be rounded-up to a power of 2.
 */
struct kfifo *kfifo_alloc(unsigned int size, gfp_t gfp_mask, spinlock_t *lock)
{
        unsigned char *buffer;
        struct kfifo *ret;

        /*
         * round up to the next power of 2, since our 'let the indices
         * wrap' tachnique works only in this case.
         */
        if (size & (size - 1)) {  /* size must be a power of 2 */
                BUG_ON(size > 0x80000000);
                size = roundup_pow_of_two(size);
        }

        buffer = kmalloc(size, gfp_mask);
        if (!buffer)
                return ERR_PTR(-ENOMEM);

        ret = kfifo_init(buffer, size, gfp_mask, lock);

        if (IS_ERR(ret))
                kfree(buffer);

        return ret;
}
EXPORT_SYMBOL(kfifo_alloc);
```
这里值得一提的是，`kfifo->size`的值总是在调用者传进来的`size`参数的基础上向2的幂扩展，这是内核一贯的做法。这样的好处不言而喻 —— 对`kfifo->size` <span style="color:red;">取模</span>运算可以转化为<span style="color:red;">与</span>运算，如下：
```c
kfifo->in % kfifo->size
```
可以转化为
```c
kfifo->in & (kfifo->size – 1)
```

在`kfifo_alloc`函数中，使用`size & (size – 1)`来判断`size`是否为 2 的幂，如果条件为真，则表示`size`不是 2 的幂，然后调用`roundup_pow_of_two`将之向上扩展为 2 的幂。 这些都是很常用的技巧，只不过大家没有将它们结合起来使用而已，下面要分析的 `__kfifo_put`和`__kfifo_get`则是将`kfifo->size`的特点发挥到了极致。

**kfifo_init**
```c
/**
 * kfifo_init - allocates a new FIFO using a preallocated buffer
 * @buffer: the preallocated buffer to be used.
 * @size: the size of the internal buffer, this have to be a power of 2.
 * @gfp_mask: get_free_pages mask, passed to kmalloc()
 * @lock: the lock to be used to protect the fifo buffer
 *
 * Do NOT pass the kfifo to kfifo_free() after use! Simply free the
 * &struct kfifo with kfree().
 */
struct kfifo *kfifo_init(unsigned char *buffer, unsigned int size,
                         gfp_t gfp_mask, spinlock_t *lock)
{
        struct kfifo *fifo;

        /* size must be a power of 2 */
        BUG_ON(!is_power_of_2(size));

        fifo = kmalloc(sizeof(struct kfifo), gfp_mask);
        if (!fifo)
                return ERR_PTR(-ENOMEM);

        fifo->buffer = buffer;
        fifo->size = size;
        fifo->in = fifo->out = 0;
        fifo->lock = lock;

        return fifo;
}
EXPORT_SYMBOL(kfifo_init);
```

**kfifo_free**
```c
/**
 * kfifo_free - frees the FIFO
 * @fifo: the fifo to be freed.
 */
void kfifo_free(struct kfifo *fifo)
{
        kfree(fifo->buffer);
        kfree(fifo);
}
EXPORT_SYMBOL(kfifo_free);
```

## `__kfifo_put`和`__kfifo_get`，巧妙的入队和出队操作，无锁并发

`__kfifo_put`是入队操作，它先将数据放入`buffer`里面，最后才修改`in`参数；`__kfifo_get`是出队操作，它先将数据从`buffer`中移走，最后才修改`out`。你会发现`in`和`out`两者各司其职。计算机科学家已经证明，当只有一个读经程和一个写线程并发操作时，不需要任何额外的锁，就可以确保是线程安全的，也即`kfifo`使用了无锁编程技术，以提高kernel的并发。

**__kfifo_put**
```c
/**
 * __kfifo_put - puts some data into the FIFO, no locking version
 * @fifo: the fifo to be used.
 * @buffer: the data to be added.
 * @len: the length of the data to be added.
 *
 * This function copies at most @len bytes from the @buffer into
 * the FIFO depending on the free space, and returns the number of
 * bytes copied.
 *
 * Note that with only one concurrent reader and one concurrent
 * writer, you don't need extra locking to use these functions.
 */
unsigned int __kfifo_put(struct kfifo *fifo,
                         unsigned char *buffer, unsigned int len)
{
        unsigned int l;

        len = min(len, fifo->size - fifo->in + fifo->out);

        /*
         * Ensure that we sample the fifo->out index -before- we
         * start putting bytes into the kfifo.
         */

        smp_mb();

        /* first put the data starting from fifo->in to buffer end */
        l = min(len, fifo->size - (fifo->in & (fifo->size - 1)));
        memcpy(fifo->buffer + (fifo->in & (fifo->size - 1)), buffer, l);

        /* then put the rest (if any) at the beginning of the buffer */
        memcpy(fifo->buffer, buffer + l, len - l);

        /*
         * Ensure that we add the bytes to the kfifo -before-
         * we update the fifo->in index.
         */

        smp_wmb();

        fifo->in += len;

        return len;
}
EXPORT_SYMBOL(__kfifo_put);
```

**__kfifo_get**
```c
/**
 * __kfifo_get - gets some data from the FIFO, no locking version
 * @fifo: the fifo to be used.
 * @buffer: where the data must be copied.
 * @len: the size of the destination buffer.
 *
 * This function copies at most @len bytes from the FIFO into the
 * @buffer and returns the number of copied bytes.
 *
 * Note that with only one concurrent reader and one concurrent
 * writer, you don't need extra locking to use these functions.
 */
unsigned int __kfifo_get(struct kfifo *fifo,
                         unsigned char *buffer, unsigned int len)
{
        unsigned int l;

        len = min(len, fifo->in - fifo->out);

        /*
         * Ensure that we sample the fifo->in index -before- we
         * start removing bytes from the kfifo.
         */

        smp_rmb();

        /* first get the data from fifo->out until the end of the buffer */
        l = min(len, fifo->size - (fifo->out & (fifo->size - 1)));
        memcpy(buffer, fifo->buffer + (fifo->out & (fifo->size - 1)), l);

        /* then get the rest (if any) from the beginning of the buffer */
        memcpy(buffer + l, fifo->buffer, len - l);

        /*
         * Ensure that we remove the bytes from the kfifo -before-
         * we update the fifo->out index.
         */

        smp_mb();

        fifo->out += len;

        return len;
}
EXPORT_SYMBOL(__kfifo_get);
```
`kfifo`每次入队或出队，`kfifo->in`或`kfifo->out`只是简单地`kfifo->in` (or)`kfifo->out` `+= len`，并没有对`kfifo->size`进行取模运算。因此`kfifo->in`和`kfifo->out`总是一直增大，直到`unsigned in` 最大值时，回绕到 0 这一起始端。但始终满足 `kfifo->out < kfifo->in`，除非`kfifo->in` 回绕到了 0 的那一端，即使如此，代码中计算长度的性质仍然是保持的。

我们先用简单的例子来形象说明这些性质吧：
```
+----------------------------------------+
|                         |<---data--->| |
+----------------------------------------+
                          ^            ^
                          |            |
                          out          in
```
上图的`out`和`in`为`kfifo->buffer`的出队数据和入队数据的游标，方框为`buffer` 的内存区域。当有数据入队时，那么`in`的值可能超过`kfifo->size`的值，那么我们使用另一个虚拟的方框来表示`in`变化后，在`buffer`内对`kfifo->size` 取模的值。如下图所示：
```
 真实的buffer内存                             虚拟的buffer内存，方便查看in对kfifo->size取模后在buffer的下标
+----------------------------------------+ +------------------------------------+
|                          |<---data-----| |--------->|                         |
+----------------------------------------+ +------------------------------------+
                           ^                          ^
                           |                          |
                           out                        in
```
当用户调用`__kfifo_put`函数，入队的数据使`kfifo->in` 游标发生如上图所示的变化时，要分两次拷贝内存数据。

因为入队数据，一部存放在`kfifo->buffer`的尾部，另一部分存放在`kfifo->buffer` 的头部。计算公式非常简单：
```c
l = kfifo->size – kfifo->in & (kfifo->size – 1)
```
表示`in`下标到`buffer`末尾，还有多少空间。

如果`len`表示需要拷贝的长度的话，那么`len - l`则表示有多少字节需要拷贝到`buffer`开始之处。

这样，我们读`__kfifo_put`代码就很容易了。
```c
len = min(len, fifo->size - fifo->in + fifo->out);
```
`fifo->in – fifo->out`表示队列里面已使用的空间大小，`fifo->size - (fifo->in – fifo->out)`表示队列未使用的空间，

因此`len = min(…)`，取两者之小，表示实际要拷贝的字节数。

拷贝`len`个字符数，`fifo->in`到`buffer`末尾所剩的空间是多少，这里面计算：
```c
/* first put the data starting from fifo->in to buffer end */
l = min(len, fifo->size - (fifo->in & (fifo->size - 1)));
memcpy(fifo->buffer + (fifo->in & (fifo->size - 1)), buffer, l);

/* then put the rest (if any) at the beginning of the buffer */
memcpy(fifo->buffer, buffer + l, len - l);
```
`l`表示`len`或`fifo->in`到`buffer`末尾所剩的空间大小的最小值，因为需要拷贝`l`字节到 `fifo->buffer + (fifo->in & (fifo->size - 1))`的位置上；那么剩下要拷贝到`buffer`开始之处的长度为`len – l`，当然，此值可能会为0，为 0 时，memcpy函数不进行任何拷贝。

所有的拷贝完成后（可能是一次，也可能是两次 memcpy)，`fifo->in` 直接 `+= len`，不需要取模运算。

写到这里，细心的读者会发现，如果`fifo->in`超过了`unsigned int`的最大值时，而回绕到 0 这一端，上述的计算公式还正确吗？ 答案是肯定的。

因为`fifo->size`的大小是 2 的幂，而`unsigned int`空间的大小是 `2^32`，后者刚好是前者的倍数。如果从上述两个图的方式来描述，则表示`unsigned int`空间的数轴上刚好可以划成一定数量个 `kfifo->size`大小方框，没有长度多余。这样，`fifo->in` (or) `fifo->out`对`fifo->size`取模后，刚好落后对应的位置上。

现在假设往`kfifo`加入数据后，使用`fifo->in < fifo->out`关系，如下：
```c
+----------------------------------------+     +------------------------------------+
|<---data---|                            | ……  |                        |<---data---|
+----------------------------------------+     +------------------------------------+
|---------------------------------------------------------------------------------->|
0                                                                                   0xffffffff
                                               ^                        ^
                                               |                        |
                                               in                       out
```
假设`kfifo`中数据的长度为`ldata`，那么`fifo->in`和`fifo->out`有这样的关系：`fifo->in = fifo->out + ldata`，并且`fifo->in < fifo->out`。这说明`fifo->in`回绕到0这一端了，尽管如此，`fifo->in`和`fifo->out`的差距还是保持的，没有变化。即`fifo->in – fifo->out`仍然是`ldata`, 那么此时的可用空间是`fifo->size – ldata = fifo->size - (fifo->in – fifo->out) = fifo->size – fifo->in + fifo->out`。

因此无论`fifo->in`和`fifo->out`谁大谁小，计算`fifo`剩余空间大小的公式`fifo->size – fifo->in + fifo->out`都正确，故可以保证`__kfifo_put`函数里面的长度计算均是正确的。

`__kfifo_get`函数使用`fifo->in – fifo->out`来计算`fifo` 内数据的空间长度，然后拷贝需要出队的数据。是否需要两次拷贝，其中原理和方法都和`__kfifo_put`是一样的。

## kfifo_put 和 kfifo_get

`kfifo_put`和`kfifo_get`在调用`__kfifo_put`和`__kfifo_get`过程都进行加锁，防止并发。

**kfifo_put**
```c
/**
 * kfifo_put - puts some data into the FIFO
 * @fifo: the fifo to be used.
 * @buffer: the data to be added.
 * @len: the length of the data to be added.
 *
 * This function copies at most @len bytes from the @buffer into
 * the FIFO depending on the free space, and returns the number of
 * bytes copied.
 */
static inline unsigned int kfifo_put(struct kfifo *fifo,
                                     unsigned char *buffer, unsigned int len)
{
        unsigned long flags;
        unsigned int ret;

        spin_lock_irqsave(fifo->lock, flags);

        ret = __kfifo_put(fifo, buffer, len);

        spin_unlock_irqrestore(fifo->lock, flags);

        return ret;
}
```
**kfifo_get**
```c
/**
 * kfifo_get - gets some data from the FIFO
 * @fifo: the fifo to be used.
 * @buffer: where the data must be copied.
 * @len: the size of the destination buffer.
 *
 * This function copies at most @len bytes from the FIFO into the
 * @buffer and returns the number of copied bytes.
 */
static inline unsigned int kfifo_get(struct kfifo *fifo,
                                     unsigned char *buffer, unsigned int len)
{
        unsigned long flags;
        unsigned int ret;

        spin_lock_irqsave(fifo->lock, flags);

        ret = __kfifo_get(fifo, buffer, len);

        /*
         * optimization: if the FIFO is empty, set the indices to 0
         * so we don't wrap the next time
         */
        if (fifo->in == fifo->out)
                fifo->in = fifo->out = 0;

        spin_unlock_irqrestore(fifo->lock, flags);

        return ret;
}
```

## 其他函数

**kfifo_reset**
```c
/**
 * __kfifo_reset - removes the entire FIFO contents, no locking version
 * @fifo: the fifo to be emptied.
 */
static inline void __kfifo_reset(struct kfifo *fifo)
{
        fifo->in = fifo->out = 0;
}

/**
 * kfifo_reset - removes the entire FIFO contents
 * @fifo: the fifo to be emptied.
 */
static inline void kfifo_reset(struct kfifo *fifo)
{
        unsigned long flags;

        spin_lock_irqsave(fifo->lock, flags);

        __kfifo_reset(fifo);

        spin_unlock_irqrestore(fifo->lock, flags);
}
```

**kfifo_len**
```c
/**
 * __kfifo_len - returns the number of bytes available in the FIFO, no locking version
 * @fifo: the fifo to be used.
 */
static inline unsigned int __kfifo_len(struct kfifo *fifo)
{
        return fifo->in - fifo->out;
}

/**
 * kfifo_len - returns the number of bytes available in the FIFO
 * @fifo: the fifo to be used.
 */
static inline unsigned int kfifo_len(struct kfifo *fifo)
{
        unsigned long flags;
        unsigned int ret;

        spin_lock_irqsave(fifo->lock, flags);

        ret = __kfifo_len(fifo);

        spin_unlock_irqrestore(fifo->lock, flags);

        return ret;
}
```

## 参考
[巧夺天工的kfifo](http://blog.csdn.net/linyt/article/details/5764312)
[linux内核数据结构之kfifo](http://www.cnblogs.com/Anker/p/3481373.html)
[《Linux内核设计与实现》读书笔记（六）- 内核数据结构](http://www.cnblogs.com/wang_yb/archive/2013/04/16/3023892.html)

<span style="color:red;">2014.03.31 更新</span>
新的内核版本中，kfifo 接口已经改写，新的 kfifo 接口请查阅最新的内核源码。
相关的接口修改历史可查阅以下链接：
<http://lwn.net/Articles/345015/>
<http://lwn.net/Articles/347619/>
