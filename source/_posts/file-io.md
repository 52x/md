---
title: 文件 I/O
date: 2014-04-13 10:05:59
tags:
  - APUE
categories:
  - Programming
  - APUE
---

原文参考：[APUE读书笔记 之 文件I/O](http://www.cnblogs.com/CoreyGao/archive/2013/04/21/3031414.html)
文章中的英文图片都引用自：<http://infohost.nmt.edu/~eweiss/222_book/222_book/0201433079/toc.html>  
中文图片为（CoreyGao）原创，用Xmind所画。
<!--more-->

## 概要

UNIX下“一切皆文件”，UNIX下的I/O即对文件的操作。  
APUE将基本I/O相关内容分为 不带缓冲的I/O，文件和目录，标准I/O库 和 系统数据文件和信息。  
不带缓冲的I/O和标准I/O库 描述了最基本的I/O。 文件和目录，数据系统文件和信息 描述了UNIX基本的文件系统和权限控制。

## 文件I/O

文件I/O都是不带缓冲的I/O。不带缓冲的I/O是指在用户空间不缓冲的I/O操作，每一个`read`和`write`都调用内核中的一个系统调用。UNIX系统中的大多数文件I/O只需用到5个函数：`open`、`read`、`write`、`lseek`以及`close`。

**文件I/O部分可以概括为两个图：**

图1 是文件I/O的常用函数总结。文件I/O函数的核心是文件描述符。

对于内核而言，所有打开的文件都通过文件描述符引用。文件描述符是一个非负整数。当打开一个现有文件或创建一个新文件时，内核向进程返回一个文件描述符。当读或写一个文件时，使用`open`或`create`返回的文件描述符标识该文件，将其作为参数传送给`read`或`write`。UNIX惯例：`0`是标准输入，`1`是标准输出，`2`是标准出错输出。POSIX标准中，幻数`0`、`1`、`2`替换成符号常量`STDIN_FILENO`、`STDOUT_FILENO` 和 `STDERR_FILENO`，这些常量定义在头文件&lt;unistd.h&gt;中。

{% asset_img file-io.png 图1 文件I/O的常用函数 %}

图2 是内核用于所有I/O的数据结构。

`fd flags` 只有一种：`fd_cloexec`（用于执行`exec`后是否保持该`fd`)  
`fd`是进程独享的，`file table`和`v-node table`都是进程间共享的。`dup`之后的俩`fd`共享`file table`，但是不复制`fd flags`。

{% asset_img file-share.gif 图2  打开文件的内核数据结构 %}

内核使用三种数据结构表示打开的文件，它们之间的关系决定了在文件共享方面一个进程对另一个进程可能产生的影响。  
（1）每个进程在进程表中都有一个记录项，记录项中包含有一张打开文件描述符表，可将其视为一个矢量，每个描述符占用一项。与每个描述符相关联的是：
1.  文件描述符标志（close_on_exec）
2.  指向一个文件表项的指针

（2）内核为所有打开文件维持一张文件表。每个文件表项包含：
1.  文件状态标志（读、写、添写、同步和非阻塞等）
2.  当前文件偏移量
3.  指向该文件 `v` 节点表项的指针

（3）每个打开文件（或设备）都有一个 `v` 节点（`v-node`）结构。`v` 节点包含了文件类型和对此文件进行各种操作的函数的指针。对于大多数文件，`v` 节点还包含了该文件的 `i` 节点（`i-node`，索引节点）。这些信息是在打开文件时从磁盘上读入内存的，所以关于文件的信息都是快速可供使用的。例如， `i` 节点包含了文件的所有者、文件长度、文件所在的设备、指向文件实际数据块在磁盘上所在位置的指针等。（注：linux没有使用 `v` 节点，而是使用了通用 `i` 节点结构，虽然两种实现有所不同，但在概念上，`v` 节点与 `i` 节点是一样的。两者都是指向文件系统特有的 `i` 节点结构。）

如果两个独立进程各自打开了同一个文件，则有图2-1中所示的安排。我们假定第一个进程在文件描述符`3`上打开该文件，而另一个进程在文件描述符`4`上打开该文件。打开该文件的每个进程都得到一个文件表项，但对一个给定的文件只有一个 `v` 节点表项。<span style="color:red;">每个进程都有自己的文件表项的一个理由是：这种安排使每个进程都有它自己的对该文件的当前偏移量。</span>

{%asset_img file-share2.gif 图2-1 两个独立进程各自打开同一个文件 %}

## I/O函数

**open**函数：打开或创建一个文件。
```c
#include <fcntl.h>

int open(const char *pathname, int oflag, ... /* mode_t mode */ );

    Returns: file descriptor if OK, -1 on error
```
第三个参数写为 `...` ，ISO C 用这种方法表明余下参数的数量及其类型根据具体的调用会有所不同。对于`open`函数而言，仅当创建新文件时才使用第三个参数。在函数原型中将此参数放置在注释中。  
`pathname`表示要打开或创建文件的名字，`oflag`参数可用来说明此函数的多个选项。用下列一个或多个常量进行“或”运算构成`oflag`参数（这些常量定义在&lt;fcntl.h&gt;头文件中）：

| oflag | Description |
|-------|-------------|
| O_RDONLY | 只读打开 |
| O_WRONLY | 只写打开 |
| O_RDWR | 读、写打开 |

在这三个常量中必须指定一个且只能指定一个。下列常量则是可选的：

| oflag | Description |
|-------|-------------|
| O_APPEND | 每次写时都追加到文件的尾端。 |
| O_CREAT | 若此文件不存在，则创建它。使用此选项时，需要第三个参数`mode`，用其指定该新文件的访问权限 |
| O_EXCL | 如果同时指定了`O_CREAT`，而文件已经存在，则会出错。用此可以测试一个文件是否存在，如果不存在，则创建此文件，这使测试和创建两者成为一个原子操作。 |
| O_TRUNC | 如果此文件存在，而且为只写或读写成功打开，则将其长度截短为0。 |
| O_NOCTTY | 如果`pathname`指的是终端设备，则不将该设备分配作为此进程的控制终端。 |
| O_NONBLOCK | 如果`pathname`指的是一个FIFO、一个块特殊文件或一个字符特殊文件，则此选项为文件的本次打开操作和后续的I/O操作设置非阻塞模式 |

下面三个标志也是可选的。它们是Single UNIX Specification (以及POSIX.1) 中同步输入和输出选项的一部分。

| oflag | Description |
|-------|-------------|
| O_DSYNC | 使每次`write`等待物理I/O操作完成，但是如果写操作并不影响读取刚写入的数据，则不等待文件属性被更新。 |
| O_RSYNC | 使每一个以文件描述符作为参数的`read`操作等待，直至任何对文件同一部分进行的未决写操作都完成。 |
| O_SYNC | 使每次`write`都等待物理I/O操作完成，包括由`write`操作引起的文件属性更新所需的I/O。 |

由`open`返回的文件描述符一定是最小的未用描述符数值。这一点被某些应用程序用来在标准输入、标准输出或标准出错输出上打开新的文件。例如，一个应用程序可以先关闭标准输出（通常是文件描述符`1`），然后打开另一个文件，执行打开操作前就能了解到该文件一定会在文件描述符`1`上打开。使用 `dup2` 函数可以更好的保证在一个给定的描述符上打开一个文件。

**creat**函数：创建一个新文件。
```c
#include <fcntl.h>

int creat(const char *pathname, mode_t mode);

    Returns: file descriptor opened for write-only if OK, -1 on error
```
此函数等效于：
```c
open(pathname, O_WRONLY | O_CREAT | O_TRUNC, mode);
```
在早期的UNIX系统版本中，`open`的第二个参数只能是`0`、`1`或`2`，没有办法打开一个尚未存在的文件，因此需要另一个系统调用`creat`以创建新文件。现在，`open`函数提供了选项`O_CREAT`和`O_TRUNC`，于是也就不再需要`creat`函数了。

**close**函数：关闭一个打开的文件。
```c
#include <unistd.h>

int close(int filedes);

    Returns: 0 if OK, -1 on error
```
关闭一个文件时还会释放该进程加在该文件上的所有记录锁。当一个进程终止时，内核自动关闭它所有打开的文件。很多程序都利用了这一功能而不显式地用`close`关闭打开的文件。

**lseek**函数：显式地为一个打开的文件设置其偏移量。
```c
#include <unistd.h>

off_t lseek(int filedes, off_t offset, int whence);

    Returns: new file offset if OK, -1 on error
```
每个打开的文件都有一个与其相关联的“当前文件偏移量(`current file offset`)”。它通常是一个非负整数，用以度量从文件开始处计算的字节数。通常，读、写操作都从当前文件偏移量处开始，并使偏移量增加所读写的字节数。按系统默认的情况，当打开一个文件时，除非指定`O_APPEND`选项，否则该偏移量被设置为`0`。

对参数`offset`的解释与参数`whence`的值有关。

*   若`whence`是`SEEK_SET`，则将该文件的偏移量设置为距文件开始处`offset`个字节。
*   若`whence`是`SEEK_CUR`，则将该文件的偏移量设置为其当前值加`offset`，`offset`可为正或负。
*   若`whence`是`SEEK_END`，则将该文件的偏移量设置为文件长度加`offset`，`offset`可为正或负。

若`lseek`成功执行，则返回新的文件偏移量，为此可以用下列方式确定打开文件的当前偏移量：
```c
off_t currpos;
currpos = lseek(fd, 0, SEEK_CUR);
```
这种方法也可用来确定所涉及的文件是否可以设置偏移量。如果文件描述符引用的是一个管道、FIFO或网络套接字，则`lseek`返回 `-1`，并将`errno`设置为`ESPIPE`。

通常，文件的当前偏移量应当是一个非负整数，但是，某些设备也可能允许负的偏移量。但对于普通文件，则其偏移量必须是非负值。因为偏移量可能是负值，所以在比较`lseek`的返回值时应当谨慎，不要测试它是否小于`0`，而要测试它是否等于 `-1`。

`lseek`仅将当前的文件偏移量记录在内核中，它并不引起任何I/O操作。然后，该偏移量用于下一个读或写操作。文件偏移量可以大于文件的当前长度，在这种情况下，对该文件的下一次写将加长该文件，并在文件中构成一个空洞，这一点是允许的。位于文件中但没有写过的字节都被读为`0`。文件中的空洞并不要求在磁盘上占用存储区。具体处理方式与文件系统的实现有关，当定位到超出文件尾端之后写时，对于新写的数据需要分配磁盘块，但是对于原文件尾端和新开始写位置之间的部分则不需要分配磁盘块。

**read**函数：从打开的文件中读数据。
```c
#include <unistd.h>

ssize_t read(int filedes, void *buf, size_t nbytes);

    Returns: number of bytes read, 0 if end of file, -1 on error
```
如果`read`成功，则返回读到的字节数。如已到达文件结尾，则返回`0`。有多种情况可使实际读到的字节数少于要求读的字节数：

*   读普通文件，在读到要求字节数之前已到达了文件尾端。
*   读终端设备，通常一次最多读一行。
*   读网络套接字，网络中的缓冲机制可能造成返回值小于所要求读的字节数。
*   读管道或FIFO，如若管道包含的字节少于所需的数量，那么`read`将只返回实际可用的字节数。
*   读某些面向记录的设备，一次最多返回一条记录。
*   当某一信号造成中断，而已经读了部分数据量时。

读操作从文件的当前偏移量处开始，在成功返回之前，该偏移量将增加实际读到的字节数。
```bash
man 2 read
```
查看`read`系统调用帮助文档。

`readn`函数：从打开的文件中读取 `n` 字节数据。
```c
ssize_t             /* Read "n" bytes from a descriptor  */
readn(int fd, void *ptr, size_t n)
{
    size_t      nleft;
    ssize_t     nread;

    nleft = n;
    while (nleft > 0) {
        if ((nread = read(fd, ptr, nleft)) < 0) {
            if (nleft == n)
                return(-1); /* error, return -1 */
            else
                break;      /* error, return amount read so far */
        } else if (nread == 0) {
            break;          /* EOF */
        }
        nleft -= nread;
        ptr   += nread;
    }
    return(n - nleft);      /* return >= 0 */
}
```

**write**函数：向打开的文件写数据。
```c
#include <unistd.h>

ssize_t write(int filedes, const void *buf, size_t nbytes);

    Returns: number of bytes written if OK, -1 on error
```
其返回值通常与参数`nbytes`的值相同，否则表示出错。`write`出错的一个常见原因是：磁盘已写满，或者超过了一个给定进程的文件长度限制。
对于普通文件，写操作从文件的当前偏移量处开始。如果打开该文件时，指定了`O_APPEND`选项，则在每次写操作之前，将文件偏移量设置在文件的当前结尾处。在一次成功写之后，该文件偏移量增加实际写的字节数。
```bash
man 2 write
```
查看`write`系统调用帮助文档。

`writen`函数：向打开的文件写入 `n` 字节数据
```c
ssize_t             /* Write "n" bytes to a descriptor  */
writen(int fd, const void *ptr, size_t n)
{
    size_t      nleft;
    ssize_t     nwritten;

    nleft = n;
    while (nleft > 0) {
        if ((nwritten = write(fd, ptr, nleft)) < 0) {
            if (nleft == n)
                return(-1); /* error, return -1 */
            else
                break;      /* error, return amount written so far */
        } else if (nwritten == 0) {
            break;
        }
        nleft -= nwritten;
        ptr   += nwritten;
    }
    return(n - nleft);      /* return >= 0 */
}
```

**pread**和**pwrite**函数：在给定的文件偏移处从文件读取或向文件写入数据。
Single UNIX Specification包括了XSI扩展，该扩展允许原子性的定位搜索（seek）和执行I/O。`pread`和`pwrite`就是这种扩展。
```c
#include <unistd.h>

ssize_t pread(int filedes, void *buf, size_t nbytes, off_t offset);

    Returns: number of bytes read, 0 if end of file, -1 on error

ssize_t pwrite(int filedes, const void *buf, size_t nbytes, off_t offset);

    Returns: number of bytes written if OK, -1 on error
```
调用`pread`相当于顺序调用`lseek`和`read`，但是`pread`又与这种顺序调用有下列重要区别：

*   调用`pread`时，无法中断其定位和读操作
*   不更新文件指针

调用`pwrite`相当于顺序调用`lseek`和`write`，但也与它们有类似的区别。

**dup**和**dup2**函数：复制一个现存的文件描述符。
```c
#include <unistd.h>

int dup(int oldfd);
int dup2(int oldfd, int newfd);

    Both return: new file descriptor if OK, -1 on error
```
由`dup`返回的新文件描述符一定是当前可用文件描述符中的最小数值。用`dup2`则可以用`newfd`参数指定新描述符的数值。如果`newfd`已经打开，则先将其关闭。

*   如果`oldfd`是无效的文件描述符，则调用失败，`newfd`不会被关闭。
*   如果`newfd`等于`oldfd`，则`dup2`返回`newfd`，而不关闭它。

这些函数返回的新文件描述符与参数`oldfd`共享同一个文件表项。图3 显示了这种情况。  
在此图中，假定进程执行了：
```c
newfd = dup(1);
```
当此函数开始执行时，假定下一个可用的描述符是`3`。因为两个描述符指向同一文件表项，所以它们共享同一文件状态标志（读、写、添写等）以及同一当前文件偏移量。  
每个文件描述符都有它自己的一套文件描述符标志，但新描述符的执行时关闭（close-on-exec）标志总是由`dup`函数清除。

{% asset_img dup.gif 图3 执行dup后的内核数据结构 %}

复制一个描述符的另一种方法是使用`fcntl`函数，实际上，调用
```c
dup(oldfd);
```
等效于
```c
fcntl(oldfd, F_DUPFD, 0);
```
而调用
```c
dup2(oldfd, newfd);
```
等效于
```c
close(newfd);
fcntl(oldfd, F_DUPFD, newfd);
```
后一种情况，`dup2`并不完全等同于`close`加上`fcntl`。它们之间的区别是：  
（1）`dup2`是一个原子操作，而`close`及`fcntl`则包括两个函数调用。有可能在`close`和`fcntl`之间插入执行信号捕获函数，它可能修改文件描述符。  
（2）`dup2`和`fcntl`有某些不同的`errno`。

**sync**、**fsync**和**fdatasync**函数

传统的UNIX实现在内核中设有缓冲区高速缓存或页面高速缓存，大多数磁盘I/O都通过缓冲进行。当将数据写入文件时，内核通常先将数据复制到其中的一个缓冲区中，如果该缓冲区尚未写满，则并不将其排入输出队列，而是等待其写满或者当内核需要重用该缓冲区以便存放其他磁盘块数据时，再将该缓冲排入输出队列，然后待其到达队首时，才进行实际的I/O操作。这种输出方式被称为<span style="color:red;">延迟写</span>。  
延迟写减少了磁盘读写次数，但是降低了文件内容的更新速度，使得欲写到文件中的数据在一段时间内并没有写到磁盘上。当系统发生故障时，这种延迟写可能造成文件更新内容的丢失。为保证磁盘上实际文件系统与缓冲区高速缓存中内容一致，UNIX系统提供了`sync`、`fsync`和`fdatasync`三个函数。
```c
#include <unistd.h>

int fsync(int filedes);
int fdatasync(int filedes);

    Returns: 0 if OK, -1 on error

void sync(void);
```
`sync` 函数只是将所有修改过的块缓冲区排入写队列，然后就返回，它并不等待实际写磁盘操作结束。通常称为`update`的系统守护进程会周期性地（一般每隔30秒）调用`sync`函数，这就保证了定期冲洗内核的块缓冲区。命令`sync`也调用`sync`函数。  
`fsync` 函数只对由文件描述符`filedes`指定的单一文件起作用，并且等待写磁盘操作结束，然后返回。`fsync`可用于数据库这样的应用程序，这种应用程序需要确保将修改过的块立即写到磁盘上。  
`fdatasync` 函数类似于`fsync`，但它只影向文件的数据部分。而除数据外，`fsync`还会同步更新文件的属性。

**fcntl**函数：改变已打开文件的性质。
```c
#include <fcntl.h>

int fcntl(int filedes, int cmd, ... /* arg */ );

    Returns: depends on cmd if OK (see following), -1 on error
```
`fcntl`函数有5种功能：  
（1）复制一个现有的描述符（`cmd = F_DUPFD`）。  
（2）获得/设置文件描述符标志（`cmd = F_GETFD` 或 `F_SETFD`）。  
（3）获得/设置文件状态标志（`cmd = F_GETFL` 或 `F_SETFL`）。  
（4）获得/设置异步I/O所有权（`cmd = F_GETOWN` 或 `F_SETOWN`）。  
（5）获得/设置记录锁（`cmd = F_GETLK`、`F_SETLK` 或 `F_SETLKW`）。  
前4种功能中（`cmd`值中的前7种），第三个参数`arg`是一个整型数。`cmd`值中的后3种，都与记录锁有关，此时函数的第三个参数`arg`指向一个结构的指针。

| cmd | Descriptiong |
|-----|--------------|
| F_DUPFD | 复制文件描述符`filedes`，新文件描述符作为函数值返回。它是尚未打开的各描述符中大于或等于第三个参数值（取为整型值）中各值的最小值。新描述符与`filedes`共享同一文件表项，但是，新描述符有它自己的一套文件描述符标志，其`FD_CLOSEXEC`文件描述符标志被清除 |
| F_GETFD | 对应于`filedes`的文件描述符标志作为函数值返回。当前只定义了一个文件描述符标志`F_CLOSEXEC`。 |
| F_SETFD | 对于`filedes`设置文件描述符标志。新标志值按第三个参数（取为整型值）设置。 |
| F_GETFL | 对应于`filedes`的文件状态标志作为函数值返回，文件状态标志对应于`open`函数中的`flags`参数。 |
| F_SETFL | 将文件状态标志设置为第三个参数的值（取为整型值）。可以更改的几个标志是：`O_APPEND`、`O_NONBLOCK`、`O_SYNC`（等待写完成，包括数据和属性）、`O_DSYNC`（等待写完成，仅数据） 和` O_RSYNC`（同步读、写） |
| F_GETOWN | 获取当前接收`SIGIO`和`SIGURG`信号的进程ID或进程组ID |
| F_SETOWN | 设置接收`SIGIO`和`SIGURG`信号的进程ID或进程组ID。正的`arg`指定一个进程ID，负的`arg`表示等于`arg`绝对值的一个进程组ID。 |

`fcntl`的返回值与命令有关。如果出错，所有命令都返回`-1`,如果成功则返回某个其他值。下列四个命令有特定返回值：`F_DUPFD`、`F_GETFD`、`F_GETFL`以及`F_GETOWN`。第一个返回新的文件描述符，接下来两个返回相应标志，最后一个返回一个正的进程ID或负的进程组ID。

在修改文件描述符标志或文件状态标志时必须谨慎，先要取得现有的标志值，然后根据需要修改它，最后设置新标志值。不能只是执行`F_SETFD`或`F_SETFL`命令，这样会关闭以前设置的标志位。
下面代码显示了一个对一个文件描述符设置一个或多个文件状态标志的函数：
```c
#include "apue.h"
#include <fcntl.h>

void
set_fl(int fd, int flags) /* flags are file status flags to turn on */
{
    int     val;

    if ((val = fcntl(fd, F_GETFL, 0)) < 0)
        err_sys("fcntl F_GETFL error");

    val |= flags;       /* turn on flags */

    if (fcntl(fd, F_SETFL, val) < 0)
        err_sys("fcntl F_SETFL error");
}
```
如果将中间的一条语句改为：
```c
val &= ~flags;  /* turn flags off */
```
就构成另一个函数，称其为 `clr_fl` 。此语句使当前文件状态标志值`val`与`flags`的补码进行逻辑“与”运算。

**ioctl**函数：I/O操作的杂物箱。
```c
int ioctl(int fd, int request, ...);
```
主要用于设备驱动程序，终端I/O是`ioctl`的最大使用方面。
