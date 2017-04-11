---
title: 文件和目录
date: 2014-04-17 10:02:30
tags:
  - APUE
categories:
  - Programming
  - APUE
---

参考：[APUE读书笔记 之 文件和目录](http://www.cnblogs.com/CoreyGao/archive/2013/04/21/3034745.html)

描述文件系统的特征和文件的性质。

UNIX用户层的文件系统主要包括两部分：

1.  文件的`stat`属性。
2.  由文件的某些属性与进程的属性相结合衍生出的权限控制系统。

本章还初步介绍了UFS（UNIX FIle System)软件层的基本结构。
<!--more-->

文件基本属性如图1:

{% asset_img ufs.png 图1 文件基本属性 %}

文件的权限控制如图2:

{% asset_img file_permission.png 图2 权限控制系统 %}

## stat、fstat 和 lstat 函数

本章讨论的中心是三个`stat`函数以及它们所返回的信息。

```c
#include <sys/stat.h>

int stat(const char *restrict pathname, struct stat *restrict buf);

int fstat(int filedes, struct stat *buf);

int lstat(const char *restrict pathname, struct stat *restrict buf);

    All three return: 0 if OK, -1 on error
```
一旦给出`pathname`，`stat`函数就返回与此命名文件有关的信息结构。`fstat`函数获取已在描述符`filedes`上打开文件的有关信息。`lstat`函数类似与`stat`，但是当命名文件是一个符号链接时，`lstat`返回该符号链接的有关信息，而不是由该符号链接引用文件的信息。

第二个参数`buf`是指针，它指向一个必须提供的结构，这些函数填写由`buf`指向的结构。该结构的实际定义可能随实现有所不同，但其基本形式是：
```c
struct stat {
    mode_t    st_mode;      /* file type & mode (permissions) */
    ino_t     st_ino;       /* i-node number (serial number) */
    dev_t     st_dev;       /* device number (file system) */
    dev_t     st_rdev;      /* device number for special files */
    nlink_t   st_nlink;     /* number of links */
    uid_t     st_uid;       /* user ID of owner */
    gid_t     st_gid;       /* group ID of owner */
    off_t     st_size;      /* size in bytes, for regular files */
    time_t    st_atime;     /* time of last access */
    time_t    st_mtime;     /* time of last modification */
    time_t    st_ctime;     /* time of last file status change */
    blksize_t st_blksize;   /* best I/O block size */
    blkcnt_t  st_blocks;    /* number of disk blocks allocated */
};
```

> 关于 restrict 修饰符：  
> C99 中新增加了 `restrict` 修饰的指针： 由 `restrict` 修饰的指针是最初唯一对指针所指向的对象进行存取的方法，仅当第二个指针基于第一个时，才能对对象进行存取。对对象的存取都限定于基于由 `restrict` 修饰的指针表达式中。  
> 由 `restrict` 修饰的指针主要用于函数形参，或指向由 `malloc()` 分配的内存空间。`restrict` 数据类型不改变程序的语义。编译器能通过作出 `restrict` 修饰的指针是存取对象的唯一方法的假设，更好地优化某些类型的例程。

## 文件类型

文件类型信息包含在`stat`结构的`st_mode`成员中，可以用表4-1中的宏确定文件类型，这些宏的参数都是`stat`结构中的`st_mode`成员。

表4-1 &lt;sys/stat.h&gt;中的文件类型宏

| 宏 | 文件类型 |
|----|----------|
| S_ISREG() | 普通文件 |
| S_ISDIR() | 目录文件 |
| S_ISCHR() | 字符特殊文件 |
| S_ISBLK() | 块特殊文件 |
| S_ISFIFO() | 管道或FIFO |
| S_ISLNK() | 符号链接 |
| S_ISSOCK() | 套接字 |

下面的程序代码对每个命令行参数打印其文件类型：
```c
#include "apue.h"

int
main(int argc, char *argv[])
{
    int         i;
    struct stat buf;
    char        *ptr;

    for (i = 1; i < argc; i++) {
        printf("%s: ", argv[i]);
        if (lstat(argv[i], &buf) < 0) {
            err_ret("lstat error");
            continue;

         }
         if (S_ISREG(buf.st_mode))
            ptr = "regular";
         else if (S_ISDIR(buf.st_mode))
            ptr = "directory";
         else if (S_ISCHR(buf.st_mode))
            ptr = "character special";
         else if (S_ISBLK(buf.st_mode))
            ptr = "block special";
         else if (S_ISFIFO(buf.st_mode))
            ptr = "fifo";
         else if (S_ISLNK(buf.st_mode))
            ptr = "symbolic link";
         else if (S_ISSOCK(buf.st_mode))
            ptr = "socket";
         else
            ptr = "** unknown mode **";
         printf("%s\n", ptr);
  }
   exit(0);
}
```
早前的UNIX系统版本并不提供`S_ISXXX`宏，于是就需要将`st_mode`与屏蔽字`S_IFMT`进行逻辑“与”运算，然后与名为`S_IFXXX`的常量相比较。大多数系统在文件&lt;sys/stat.h&gt;中定义了此屏蔽字和相关的常量。如若查看此文件，则可找到`S_ISDIR`宏定义为：
```c
#define S_ISDIR(mode) (((mode) & S_IFMT) == S_IFDIR)
```

## 设置用户ID（Set-User-ID）和设置组ID（Set-Group-ID）

**进程相关ID**

与一个进程相关联的ID有6个或更多，如表4-4所示：

表4-4 与每个进程相关联的用户ID和组ID

| ID | Description |
|----|-------------|
| 实际用户ID（real user ID）<br/>实际组ID（real group ID） | 我们实际上是谁 |
| 有效用户ID（effective user ID）<br/>有效组ID（effective group ID）<br/>附加组ID（supplementary group IDs） | 用于文件访问权限检查 |
| 保存的设置用户ID（saved set-user-ID ）<br/>保存的设置组ID（saved set-group-ID） | 由exec函数保存 |

*   实际用户ID和实际组ID标识我们究竟是谁。这两个字段在用户登录时取自口令文件中的登录项。通常，在一个登录会话期间这些值并不改变，但是超级用户进程有方法改变它们。
*   有效用户ID、有效组ID以及附加组ID决定了我们的文件访问权限。
*   保存的设置用户ID和保存的设置组ID在执行一个程序时包含了有效用户ID和有效组ID的副本。
通常，有效用户ID等于实际用户ID、有效组ID等于实际组ID。

**文件相关ID**

每个文件都有一个所有者和组所有者，所有者由`stat`结构中的`st_uid`成员表示，组所有者则由`st_gid`成员表示。

当执行一个程序文件时，进程的有效用户ID通常就是实际用户ID，有效组ID通常是实际组ID。但是可以在文件模式字（`st_mode`）中设置一个特殊标志，其含义是“<span style="color:red;">当执行此文件时，将进程的有效用户ID设置为文件所有者的用户ID（st_uid）</span>”。与此相类似，在文件模式字中可以设置另一位，它使得<span style="color:red;">将执行此文件的进程的有效组ID设置为文件的组所有者ID（`st_gid`）</span>。在文件模式字中的这两位被称为 _设置用户ID_（`set-user-ID`）位和 _设置组ID_（`set-group-ID`）位。

再返回到`stat`函数，设置用户ID位和设置组ID位都包含在`st_mode`值中，这两位可用常量 `S_ISUID` 和 `S_ISGID` 测试。

## 文件访问权限

`st_mode`值也包含了针对文件的访问权限位。所有文件类型（目录文件、字符特殊文件等）都有访问权限（access permission）。
每个文件有9个访问权限位，可将它们分成三类（读、写及执行），如表4-5所示：

<a name="file_permission">表4-5</a> 9个文件访问权限位，取自&lt;sys/stat.h&gt;

| st_mode mask | Meaning |
|--------------|---------|
| S_IRUSR<br/>S_IWUSR<br/>S_IXUSR | user-read<br/>user-write<br/>user-execute |
| S_IRGRP<br/>S_IWGRP<br/>S_IXGRP | group-read<br/>group-write<br/>group-execute |
| S_IROTH<br/>S_IWOTH<br/>S_IXOTH | other-read<br/>other-write<br/>other-execute |

```c
/* Protection bits.  */
#define __S_ISUID       04000   /* Set user ID on execution.  */
#define __S_ISGID       02000   /* Set group ID on execution.  */
#define __S_ISVTX       01000   /* Save swapped text after use (sticky).  */
#define __S_IREAD       0400    /* Read by owner.  */
#define __S_IWRITE      0200    /* Write by owner.  */
#define __S_IEXEC       0100    /* Execute by owner.  */

#define S_ISUID __S_ISUID       /* Set user ID on execution.  */
#define S_ISGID __S_ISGID       /* Set group ID on execution.  */
#define S_ISVTX __S_ISVTX       /* Save swapped text after use (sticky bit).  */

#define S_IRUSR __S_IREAD       /* Read by owner.  */
#define S_IWUSR __S_IWRITE      /* Write by owner.  */
#define S_IXUSR __S_IEXEC       /* Execute by owner.  */
/* Read, write, and execute by owner.  */
#define S_IRWXU (__S_IREAD|__S_IWRITE|__S_IEXEC)

#define S_IRGRP (S_IRUSR >> 3)  /* Read by group.  */
#define S_IWGRP (S_IWUSR >> 3)  /* Write by group.  */
#define S_IXGRP (S_IXUSR >> 3)  /* Execute by group.  */
/* Read, write, and execute by group.  */
#define S_IRWXG (S_IRWXU >> 3)

#define S_IROTH (S_IRGRP >> 3)  /* Read by others.  */
#define S_IWOTH (S_IWGRP >> 3)  /* Write by others.  */
#define S_IXOTH (S_IXGRP >> 3)  /* Execute by others.  */
/* Read, write, and execute by others.  */
#define S_IRWXO (S_IRWXG >> 3)
```
用名字打开任一类型的文件时，对该名字包含的每一个目录，包括它可能隐含的当前工作目录，都应具有执行权限，这也是为什么对于目录其执行权限位常被称为搜索位的原因。对于目录的读权限和执行权限，其意义是不同的：

*   读权限允许我们读目录，获得在该目录中所有文件名的列表；
*   当一个目录是我们要访问文件的路径名的一个组成部分时，对该目录的执行权限使我们可通过该目录（也即是搜索该目录，寻找一个特定的文件名）。

引用隐含目录的一个例子是，如果`PATH`环境变量指定了一个我们不具有执行权限的目录，那么shell决不会在该目录下找到可执行文件。

为了在一个目录中创建一个新文件，必须对该目录具有写权限和执行权限，为了删除一个现有文件，必须对包含该文件的目录具有写权限和执行权限，但对该文件本身则不需要有读、写权限。如果使用`exec`函数族中的任何一个执行某个文件，都必须对该文件具有执行权限，且该文件还必须是一个普通文件。

进程每次打开、创建或者删除一个文件时，内核就进行文件访问权限测试，而这种测试可能涉及文件的所有者（`st_uid` 和` st_gid`）、进程的有效ID（有效用户ID和有效组ID）以及进程的附加组ID（若支持）。两个所有者ID是文件的性质，而两个有效ID和附加组ID则是进程的性质。内核进行的测试是：  
（1）若进程的有效用户ID是`0`（超级用户），则允许访问。这给予了超级用户对整个文件系统进行处理的最充分的自由。  
（2）若进程的有效用户ID等于文件的所有者ID（也即该进程拥有此文件），那么，若所有者适当的访问权限位被设置，则允许访问，否则拒绝访问。适当的访问权限位指的是，若进程为读而打开该文件，则用户读位应为`1`；若进程为写而打开该文件，则用户写位应为`1`；若进程将执行该文件，则用户执行位应为`1`。  
（3）若进程的有效组ID或进程的附加组ID之一等于文件的组ID，那么，若组适当的访问权限位被设置，则允许访问，否则拒绝访问。  
（4）若其他用户适当的访问权限位被设置，则允许访问，否则拒绝访问。  
按顺序执行这四步。注意，如若进程拥有此文件，则按用户访问权限批准或拒绝该进程对文件的访问 —— 不查看组访问权限。类似的，若进程并不拥有该文件，但进程属于某个适当的组，则按组访问权限批准或拒绝该进程对文件的访问 —— 不查看其他用户的访问权限。

**新文件和目录的所有权**

新文件的用户ID设置为进程的有效用户ID，关于组ID，POSIX.1 允许实现选择下列之一作为新文件的组ID。  
（1）新文件的组ID可以是进程的有效组ID  
（2）新文件的组ID可以是它所在目录的组ID  

**粘住位**

> `S_ISVTX` 位有一段有趣的历史。在UNIX尚未使用分页技术的早期版本中，`S_ISVTX` 位被称为 _粘住位_（stick bit）。如果一个可执行程序文件的这一位被设置了，那么在该程序第一次被执行并结束时，其程序正文部分的一个副本仍被保存在交换区。这使得下次执行该程序时能较快的将其装入内存区。其原因是：交换区占用连续磁盘空间，可将它视为连续文件，而且一个程序的正文部分在交换区中也是连续存放的，而在一般的UNIX文件系统中，文件的各数据块很可能是随机存放的。对于常用的应用程序，例如文本编辑器和 C 编译器，常常设置它们所在文件的粘住位。自然，对于在交换区中可以同时存放的设置了粘住位的文件数是有一定限制的，以免过多占用交换区空间，但无论如何，这是一个有用的技术。因为在系统再次自举前，文件的正文部分总是在交换区中，所以使用了名字“粘住”。后来的UNIX版本称它为_保存正文位_（saved-text bit），因此也就有了常量 `S_ISVTX` 。现今较新的UNIX系统大多数都配置有虚拟存储系统以及快速文件系统，所以不再需要使用这种技术。

现今的系统扩展了粘住位的使用范围，Single UNIX Specification <span style="color:red;">允许针对目录设置粘住位</span>。如果对一个目录设置了粘住位，则只有对该目录具有写权限的用户在满足下列条件之一的情况下，才能删除或更名该目录下的文件：

*   拥有此文件
*   拥有此目录
*   是超级用户

目录 /tmp 是设置粘住位的典型候选者 —— 任何用户都可在这个目录中创建文件。任一用户（用户、组和其他）对这个目录的权限通常都是读、写和执行。但是用户不应能删除或更名属于其他人的文件，为此在这个目录的文件模式中设置了粘住位。

## 与文件访问权限相关的函数

**access**函数：测试文件访问权限。

```c
#include <unistd.h>

int access(const char *pathname, int mode);

    Returns: 0 if OK, -1 on error
```
其中，`mode`是下列常量的按位或（取自&lt;unistd.h&gt;）。
```c
/* Values for the second argument to access.
   These may be OR'd together.  */
#define R_OK    4               /* Test for read permission.  */
#define W_OK    2               /* Test for write permission.  */
#define X_OK    1               /* Test for execute permission.  */
#define F_OK    0               /* Test for existence.  */
```
如前所述，当用`open`函数打开一个文件时，内核以进程的有效用户ID或有效组ID为基础执行其访问权限测试，而`access`函数则是按实际用户ID和实际组ID进行访问权限测试的。  
`access`函数示例：
```c
#include "apue.h"
#include <fcntl.h>

int
main(int argc, char *argv[])
{
    if (argc != 2)
        err_quit("usage: a.out <pathname>");
    if (access(argv[1], R_OK) < 0)
        err_ret("access error for %s", argv[1]);
    else
        printf("read access OK\n");
    if (open(argv[1], O_RDONLY) < 0)
        err_ret("open error for %s", argv[1]);
    else
        printf("open for reading OK\n");
    exit(0);
}
```

**umask**函数：设置文件模式创建屏蔽字。
```c
#include <sys/stat.h>

mode_t umask(mode_t cmask);

    Returns: previous file mode creation mask
```
其中，参数`cmask`由[表4-5](#file_permission)中列出的9个常量（`S_IRUSR`、`S_IWUSR`等）中的若干个按位“或”构成。  
`umask`函数为进程设置文件模式创建屏蔽字，并返回以前的值。（这是少数几个没有出错返回函数中的一个。）在进程创建一个新文件或目录时，就一定会使用文件模式创建屏蔽字。对于任何在文件模式创建屏蔽字中为 `1` 的位，在文件`mode`中的相应位则一定被关闭。  
umask函数示例：
```c
#include "apue.h"
#include <fcntl.h>

#define RWRWRW (S_IRUSR|S_IWUSR|S_IRGRP|S_IWGRP|S_IROTH|S_IWOTH)

int
main(void)
{
    umask(0);
    if (creat("foo", RWRWRW) < 0)
        err_sys("creat error for foo");
    umask(S_IRGRP | S_IWGRP | S_IROTH | S_IWOTH);
    if (creat("bar", RWRWRW) < 0)
        err_sys("creat error for bar");
    exit(0);
}
```
上述程序创建两个文件，创建第一个时，`umask`值为`0`，创建第二个时，`umask`值禁止所有组和其他用户的访问权限。若运行此程序可得如下结果，从中可见访问权限是如何设置的。
```bash
$ umask                #first print the current file mode creation mask
002
$ ./a.out
$ ls -l foo bar
-rw------- 1 sar            0 Dec 7 21:20 bar
-rw-rw-rw- 1 sar            0 Dec 7 21:20 foo
$ umask                #see if the file mode creation mask changed
002
```
UNIX系统的大多数用户从不处理他们的`umask`值。通常在登录时，由shell的启动文件设置一次，然后从不改变。尽管如此，当编写创建新文件的程序时，如果我们想确保指定的访问权限位已经激活，那么必须在进程运行时修改`umask`值。例如，如果我们想确保任何用户都能读文件，则应将`umask`设置为`0`。否则，当我们的进程运行时，有效的`umask`值可能关闭该权限位。  
在前面的示例中，我们用shell的`umask`命令在运行程序的前、后打印文件模式创建屏蔽字。从中可见，<span style="color:red;">更改进程的文件模式创建屏蔽字并不影响其父进程（常常是shell）的屏蔽字</span>。  
用户可设置`umask`值以控制他们所创建文件的默认权限。该值表示成八进制数（参考[表4-5](#file_permission)的宏定义），一位代表一种要屏蔽的权限。设置了相应位后，它所对应的权限就会被拒绝。常用的几种`umask`值是`002`、`022`和`027`，`002`阻止其他用户写文件，`022`阻止同组成员和其他用户写文件，`027`阻住同组成员写文件以及其他用户读、写或执行文件。

**chmod**和**fchmod**函数：改变现有文件的访问权限。
```c
#include <sys/stat.h>

int chmod(const char *pathname, mode_t mode);
int fchmod(int filedes, mode_t mode);

    Both return: 0 if OK, -1 on error
```
`chmod`函数在指定的文件上进行操作，而`fchmod`函数则对已打开的文件进行操作。为改变一个文件的权限位，进程的有效用户ID必须等于文件的所有者ID，或者该进程必须具有超级用户权限。  
参数`mode`是[表4-5](#file_permission)中所示的9个文件访问权限位外加以下6项常量的“按位或”：两个设置ID常量（`S_ISUID` 和 `S_ISGID`）、保存正文常量（`S_ISVTX`），以及三个组合常量（`S_IRWXU`、`S_IRWXG` 和 `S_IRWXO`）。  
chmod函数示例：
```c
#include "apue.h"

int
main(void)
{
     struct stat      statbuf;

     /* turn on set-group-ID and turn off group-execute */

     if (stat("foo", &statbuf) < 0)
         err_sys("stat error for foo");
     if (chmod("foo", (statbuf.st_mode & ~S_IXGRP) | S_ISGID) < 0)
         err_sys("chmod error for foo");

     /* set absolute mode to "rw-r--r--" */

     if (chmod("bar", S_IRUSR | S_IWUSR | S_IRGRP | S_IROTH) < 0)
         err_sys("chmod error for bar");

     exit(0);
}
```
**chown**、**fchown**和**lchown**函数：更改文件的用户ID和组ID。
```c
#include <unistd.h>

int chown(const char *pathname, uid_t owner, gid_t group);
int fchown(int filedes, uid_t owner, gid_t group);
int lchown(const char *pathname, uid_t owner, gid_t group);

    All three return: 0 if OK, -1 on error
```
除了所引用的文件是符号链接外，这三个函数的操作相似。在符号链接的情况下，`lchown`更改符号链接本身的所有者，而不是该符号链接所指向的文件。  
若两个参数`owner`或`group`中的任意一个是`-1`，则对应的ID不变。  
基于BSD的系统一直规定只有超级用户才能更改一个文件的所有者。这样做的原因是防止用户改变其文件的所有者从而摆脱磁盘空间限额对他们的限制。系统V则允许任一用户更改他们所拥有的文件的所有者。  
若 `_POSIX_CHOWN_RESTRICTED`（此选项与所引用的文件有关 —— 可在每个文件系统基础上，使该选项起作用或不起作用） 对指定的文件起作用，则  
（1）只有超级用户进程能更改该文件的用户ID。  
（2）若满足下列条件，一个非超级用户进程可以更改该文件的组ID：

*   进程拥有此文件（其有效用户ID等于该文件的用户ID）。
*   参数`owner`等于`-1`或文件的用户ID，并且参数`group`等于进程的有效组ID或进程的附加组ID之一。

这意味着，当 `_POSIX_CHOWN_RESTRICTED` 起作用时，不能更改其他用户文件的用户ID。你可以更改你所拥有的文件的组ID，但只能改到你所属的组。如果这些函数由非超级用户进程调用，则在成功返回时，该文件的设置用户ID位和设置组ID位都会被清除。

## 文件长度

`stat`结构成员`st_size`表示以字节为单位的文件长度。此字段只对普通文件、目录文件和符号链接有意义。对于普通文件，其长度可以是`0`，在读这种文件时，将得到文件结束(end-of-file)指示。对于目录，文件长度通常是一个数（例如`16`或`512`）的倍数。对于符号链接，文件长度是文件名中的实际字节数。例如
```bash
lrwxrwxrwx 1 root           7 Sep 25 07:14 lib -> usr/lib
```
其中，文件长度 `7` 就是路径名`usr/lib`的长度（注意，因为符号链接文件长度总是由`st_size`指示，所以它并不包含通常C语言用作名字结尾的`null`字符）。
现今，大多数UNIX系统提供字段`st_blksize`和`st_blocks`。其中，第一个是对文件I/O较合适的块长度，第二个是所分配的实际512<sub><span style="color:red;">注</span></sub>字节块数量。<span style="color:red;">当`st_blksize`用于读操作时，读一个文件所需的时间量最少。</span>为了效率的缘故，标准I/O库也试图一次读、写`st_blksize`个字节。
<span style="color:red;">注</span>：不同的UNIX版本，其`st_blocks`所用的单位可能不是`512`字节的块，使用此值并不是可移植的。

**文件中的空洞**

普通文件可以包含空洞，空洞是由所设置的偏移量超过文件尾端，并写了某些数据后造成的。例如：
```bash
$ ls -l core
-rw-r--r-- 1 sar       8483248 Nov 18 12:18 core
$ du -s core
272        core
```
文件`core`的长度刚好超过8 MB字节，而`du`命令则报告该文件所使用的磁盘空间总量是272个512字节块（139264字节），很明显，此文件中有很多空洞。对于没有写过的字节位置，`read`函数读到的字节是`0`。如果执行：
```bash
$ wc -c core
8483248 core
```
`wc`命令的`-c`选项表示统计文件中的字符（字节）数

如果使用实用程序（例如`cat`）复制这种文件，那么所有这些空洞都会被填满，其中所有实际数据字节皆填写为`0`.
```bash
$ cat core > core.copy
$ ls -l core*
-rw-r--r--  1 sar      8483248 Nov 18 12:18 core
-rw-rw-r--  1 sar      8483248 Nov 18 12:27 core.copy
$ du -s core*
272     core
16592   core.copy
```
从中可见，新文件所用的字节数是8495104（512 x 16592）。此长度与ls命令报告的长度不同，其原因是，文件系统使用了若干块以存放指向实际数据块的各个指针。

**文件截短**

有时我们需要在文件尾端处截去一些数据以缩短文件。将一个文件清空为`0`是一个特列，在打开文件时使用`O_TRUNC`标志可以做到这一点。
```c
#include <unistd.h>

int truncate(const char *pathname, off_t length);
int ftruncate(int filedes, off_t length);

    Both return: 0 if OK, -1 on error
```
这两个函数将把现有的文件长度截短为`length`字节。如果该文件以前的长度大于`length`，则超过`length`以外的数据就不能再访问；如果以前的长度短于`length`，则可能在文件中创建一个空洞。

## 文件时间

对每个文件保持有三个时间字段，它们的意义示于表4-10中。

表4-10 与每个文件相关的三个时间值

| 字段 | 说明 | 例子 | ls 选项 |
|------|------|------|---------|
| st_atime<br/>st_mtime<br/>st_ctime | 文件数据的最后访问时间<br/>文件数据的最后修改时间<br/>i节点状态的最后更改时间 | read<br/>write<br/>chmod, chown | -u<br/>默认<br/>-c |

注意修改时间（`st_mtime`）和状态更改时间（`st_ctime`）之间的区别。修改时间是文件内容最后一次被修改的时间。状态更改时间是该文件的i节点最后一次被修改的时间。很多影响到i节点的操作，例如，更改文件的访问权限、用户ID、链接数等，但它们并没有更改文件的实际内容。因为i节点中的所有信息都是与文件的实际内容分开存放的，所以，除了文件数据修改时间以外，还需要状态更改的时间。  
注意，系统并不保存对一个i节点的最后一次访问时间，所以`access`和`stat`函数并不更改这三个时间中的任一个。

**utime**函数：更改一个文件的访问和修改时间。
```c
#include <utime.h>

int utime(const char *pathname, const struct utimbuf *times);

    Returns: 0 if OK, -1 on error
```
此函数使用的数据结构是：
```c
struct utimbuf {
  time_t actime;    /* access time */
  time_t modtime;   /* modification time */
}
```
结构中的两个时间值是日历时间，这是自1970年1月1日00:00:00以来国际标准时间所经过的秒数。

函数的操作以及执行时所要求的特权取决于`times`参数是否为`NULL`。

*   如果`times`是一个空指针，则访问时间和修改时间两者都设置为当前时间。为执行此操作必须满足下列两个条件之一：进程的有效用户ID必须等于文件的所有者ID，或者进程对该文件必须具有写权限。
*   如果`times`是非空指针，则访问时间和修改时间被设置为`times`所指向结构中的值。此时进程的有效用户ID必须等于该文件的所有者ID，或者进程必须是一个超级用户进程。对文件具有写权限是不够的。

注意，不能对状态更改时间`st_ctime`指定一个值，当调用`utime`函数时，此字段将被自动更新。在某些UNIX系统版本中，`touch`命令使用了`utime`函数。

`utime`函数使用示例：
```c
#include "apue.h"
#include <fcntl.h>
#include <utime.h>

int
main(int argc, char *argv[])
{
    int             i, fd;
    struct stat     statbuf;
    struct utimbuf  timebuf;

    for (i = 1; i < argc; i++) {
        if (stat(argv[i], &statbuf) < 0) { /* fetch current times */
            err_ret("%s: stat error", argv[i]);
            continue;
        }
        if ((fd = open(argv[i], O_RDWR | O_TRUNC)) < 0) { /* truncate */
            err_ret("%s: open error", argv[i]);
            continue;

        }
        close(fd);
        timebuf.actime  =  statbuf.st_atime;
        timebuf.modtime =  statbuf.st_mtime;
        if (utime(argv[i], &timebuf) < 0) {     /* reset times */
            err_ret("%s: utime error", argv[i]);
            continue;
        }
    }
    exit(0);
}
```

## 文件系统

**UNIX 文件系统的基本结构**

一个磁盘可以分成多个分区，每个分区可以包含一个文件系统（如图4-1）。

{% asset_img unix_fs.gif 图4-1 Disk drive, partitions, and a file system %}

`i`节点是固定长度的记录项，它包含有关文件的大部分信息。

如果更仔细地观察一个柱面组的i节点和数据块部分，则可以看到图4-2所示的情况。

{% asset_img i-node.gif 图4-2 Cylinder group's i-nodes and data blocks in more detail %}

注意图4-2中的下列各点：

*   在图中有两个目录项指向同一个`i`节点。每个`i`节点中都有一个链接计数，其值是指向该`i`节点的目录项数。只有当链接计数减少至`0`时，才可删除该文件（也即是可以释放该文件占用的数据块）。这就是为什么“解除对一个文件的链接”操作并不总是意味着“释放该文件占用的磁盘块”的原因。这也是为什么删除一个目录项的函数被称为`unlink`而不是`delete`的原因。在`stat`结构中，链接计数包含在`st_nlink`成员中，其基本系统数据类型是`nlink_t`，这种链接类型称为硬链接。
*   另外一种链接类型称为符号链接（symbolic link）。对于这种链接，该文件的实际内容（在数据块）包含了该符号链接所指向的文件的名字。在下例中：
    ```bash
    lrwxrwxrwx 1 root         7 Sep 25 07:14 lib -> usr/lib
    ```
    该目录项中的文件名是`3`字符的字符串`lib`，而在该文件中包含了`7`个数据字节`usr/lib`。
该i节点的文件类型是`S_IFLINK`，于是系统知道这是一个符号链接。
*   i节点包含了大多数与文件有关的信息：文件类型、文件访问权限、文件长度和指向该文件所占用的数据块的指针等等。`stat`结构中的大多数信息都取自`i`节点。只有两项数据存放在目录项中：文件名和`i`节点编号。`i`节点编号的数据类型是`ino_t`。
*   每个文件系统各自对它们的`i`节点进行编号，因此目录项中的`i`节点编号数是指向同一个文件系统中的相应i节点，不能使一个目录项指向另一个文件系统的`i`节点。这就是为什么`ln`命令（构造一个指向一个现有文件的新目录项）不能跨越文件系统的原因。
*   当在不更换文件系统的情况下为一个文件更名时，该文件的实际内容并未移动，只需构造一个指向现有`i`节点的新目录项，并解除与旧目录项的链接。例如，为将文件`/usr/lib/foo`更名为`/usr/foo`，如果目录`/usr/lib`和`/usr`在同一个文件系统中，则文件`foo`的内容无需移动。这就是`mv`命令的通常操作方式。

我们说明了普通文件的链接计数概念，但是对于目录文件的链接计数字段又如何呢？假定我们在工作目录中构造了一个新目录：
```bash
$ mkdir testdir
```
图4-3显示了其结果。注意，该图显式的显示了 `.` 和 `..` 目录项。

{% asset_img dentry.gif 图4-3 Sample cylinder group after creating the directory testdir %}

对于编号为`2549`的`i`节点，其类型字段表示它是一个目录，而链接计数为`2`.任何一个叶目录（不包含任何其他目录的目录）的链接计数总是`2`，数值`2`来自于命名该目录（`testdir`）的目录项以及在该目录中的 `.` 项。对于编号为`1267`的`i`节点，其类型字段表示它是一个目录，而其链接计数则大于或等于`3`。它大于或等于`3`的原因是，至少有三个目录项指向它：一个是命名它的目录项（在图4-3中没有表示出来），第二个是在该目录中的 `.` 项，第三个是在其子目录`testdir`中的 `..` 项。注意，父目录中的每一个子目录都会使该父目录项的链接计数增`1`。

**link**、**unlink**、**remove** 和 **rename** 函数

`link`函数：创建一个指向现有文件的链接。
```c
#include <unistd.h>

int link(const char *existingpath, const char *newpath);

    Returns: 0 if OK, -1 on error
```
任何一个文件可以有多个目录项指向其`i`节点，此函数创建一个新目录项`newpath`，它引用现有的文件`existingpath`。如若`newpath`已经存在，则返回出错。只创建`newpath`中的最后一个分量，路径中的其他部分应当已经存在。  
创建新目录项以及增加链接计数应当是个原子操作。POSIX.1允许实现支持跨文件系统的链接，但大多数实现要求这两个路径名在同一个文件系统中。很多文件系统实现不允许创建指向目录的硬链接，其理由是指向目录的硬链接可能在文件系统中形成循环，而大多数处理文件系统的实用程序都不能处理这种情况。

`unlink`函数：删除一个现有的目录项。
```c
#include <unistd.h>

int unlink(const char *pathname);

    Returns: 0 if OK, -1 on error
```
此函数删除目录项，并将由`pathname`所引用文件的链接计数减`1`.如果还有指向该文件的其他链接，则仍可以通过其他链接访问该文件的数据。如果出错，则不对该文件做任何更改。
为解除对文件的链接，进程必须对包含该目录项的目录具有写和执行权限。如果对目录设置了粘住位，则对该目录必须具有写权限，并且具备下面三个条件之一：

*   拥有该文件。
*   拥有该目录。
*   具有超级用户特权。

只有当链接计数达到`0`时，该文件的内容才可被删除。只要仍有进程打开了该文件，其文件内容不能被删除。关闭一个文件时，内核首先检查打开该文件的进程数，如果该数达到`0`，则检查其链接数，如果这个数也是`0`，则删除该文件内容。

`unlink`函数使用示例：
```c
#include "apue.h"
#include <fcntl.h>

int
main(void)
{
    if (open("tempfile", O_RDWR) < 0)
        err_sys("open error");
    if (unlink("tempfile") < 0)
        err_sys("unlink error");
    printf("file unlinked\n");
    sleep(15);
    printf("done\n");
    exit(0);
}
```
运行该程序，其结果是：
```bash
$ ls -l tempfile            #look at how big the file is
-rw-r----- 1 sar     413265408 Jan 21 07:14 tempfile
$ df /home                  #check how much free space is available
Filesystem  1K-blocks     Used  Available  Use%  Mounted  on
/dev/hda4    11021440  1956332    9065108   18%  /home
$ ./a.out &                 #run the program above in the background
1364                        #the shell prints its process ID
$ file unlinked             #the file is unlinked
ls -l tempfile              #see if the filename is still there
ls: tempfile: No such file or directory   #the directory entry is gone
$ df /home                  #see if the space is available yet
Filesystem  1K-blocks     Used  Available  Use%  Mounted  on
/dev/hda4    11021440  1956332    9065108   18%  /home
$ done                      #the program is done, all open files are closed
df /home                    #now the disk space should be available
Filesystem  1K-blocks     Used  Available  Use%  Mounted on
/dev/hda4    11021440  1552352    9469088   15%  /home
                            #now the 394.1 MB of disk space are available
```
<span style="color:red;">`unlink`的这种性质经常被程序用来确保即使是在该程序崩溃时，它所创建的临时文件也不会遗留下来。</span>进程用`open`或`create`创建一个文件，然后立即调用`unlink`。因为该文件仍旧是打开的，所以不会将其内容删除。只有当进程关闭该文件或终止时（在这种情况下，内核会关闭该进程打开的全部文件），该文件的内容才会被删除。  
如果`pathname`是符号链接，那么`unlink`删除该符号链接，而不会删除由该链接所引用的文件。给出符号链接名情况下，没有一个函数能删除由该链接所引用的文件。超级用户可以调用`unlink`，其参数`pathname`指向一个目录，但通常应当使用`rmdir`函数，而不使用这种方式。

`remove`函数：解除对一个文件或目录的链接。
```c
#include <stdio.h>

int remove(const char *pathname);

    Returns: 0 if OK, -1 on error
```
对于文件，`remove`的功能与`unlink`相同；对于目录，`remove`的功能与`rmdir`相同。

> ISO C 指定`remove`函数删除一个文件，这更改了UNIX历来使用的名字`unlink`，其原因是实现C标准的大多数非UNIX系统并不支持文件链接。

`rename`函数：更改文件或目录的名称。
```c
#include <stdio.h>

int rename(const char *oldname, const char *newname);

    Returns: 0 if OK, -1 on error
```

> ISO C对文件定义了此函数（C标准不处理目录）。POSIX.1扩展此定义，使其包含目录和符号链接。

根据`oldname`指向文件还是目录，有以下几种情况：  
（1）如果`oldname`指向一个文件而不是目录，那么为该文件或符号链接更名。如果`newname`已存在，则它不能引用一个目录。如果`newname`已存在，而且不是一个目录，则先将该目录项删除然后将`oldname`更名为`newname`。对包含`oldname`的目录以及包含`newname`的目录，调用进程必须具有写权限，因为将更改这两个目录。  
（2）如果`oldname`指向一个目录，那么为该目录更名。如果`newname`已存在，则它必须引用一个目录，而且该目录应当是空目录（空目录指的是该目录中只有 `.` 和 `..` 项）。如果`newname`存在（而且是一个空目录），则先将其删除，然后将`oldname`更名为`newname`。另外，当为一个目录更名时，`newname`不能包含`oldname`作为其路径前缀。例如，不能将`/usr/foo`更名为`/usr/foo/testdir`，因为旧名字（`/usr/foo`）是新名字的路径前缀，因而不能将其删除。  
（3）如果`oldname`或`newname`引用符号链接，则处理的是符号链接本身，而不是它所引用的文件。  
（4）作为一个特例，如果`oldname`和`newname`引用同一个文件，则函数不做任何更改而成功返回。

**符号链接**

符号链接是指向一个文件的间接指针，它与硬链接不同，硬链接直接指向文件的i节点。引入符号链接的原因是为了避开硬链接的一些限制：

*   硬链接通常要求链接和文件位于同一个文件系统中。
*   只有超级用户才能创建指向目录的硬链接。

对符号链接以及它指向何种对象并无任何文件系统限制，任何用户都可创建指向目录的符号链接。符号链接一般用于将一个文件或整个目录结构移到系统中的另一个位置。  
当使用<span style="color:red;">以名字引用文件</span>的函数时，应当了解该函数是否处理符号链接。也即是该函数是否<span style="color:red;">跟随符号链接到达它所链接的文件</span>。如若该函数具有处理符号链接的功能，则其路径名参数引用由符号链接指向的文件。否则，路径名参数将引用链接本身，而不是该链接指向的文件。  
用`open`打开文件时，如果传递给`open`函数的路径名指定了一个符号链接，`open`函数将跟随此链接到达所指向的文件。若此符号链接所指向的文件并不存在，则`open`返回出错，表示它不能打开该文件。这可能会使不熟悉符号链接的用户感到迷惑，例如：
```bash
$ ln -s /no/such/file myfile            #create a symbolic link
$ ls myfile
myfile                                  #ls says it's there
$ cat myfile                            #so we try to look at it
cat: myfile: No such file or directory
$ ls -l myfile                          #try -l option
lrwxrwxrwx 1 sar        13 Jan 22 00:26 myfile -> /no/such/file
```

**symlink** 和 **readlink** 函数

`symlink`函数：创建一个符号链接。
```c
#include <unistd.h>

int symlink(const char *actualpath, const char *sympath);

    Returns: 0 if OK, -1 on error
```
该函数创建一个指向`actualpath`的新目录项`sympath`，在创建此符号链接时，并不要求`actualpath`已经存在，并且，`actualpath`和`sympath`并不需要位于同一个文件系统中。

`readlink`函数：读取符号链接中的名字。
```c
#include <unistd.h>

ssize_t readlink(const char* restrict pathname, char *restrict buf,
                 size_t bufsize);

    Returns: number of bytes read if OK, -1 on error
```
因为`open`函数跟随符号链接，所以需要有一种方法打开该符号链接本身，并读取该链接中的名字。`readlink`函数组合了`open`、`read`和`close`的所有操作。如果此函数成功执行，则它返回读入`buf`的字节数。<span style="color:red;">在`buf`中返回的符号链接的内容不以`null`字符终止。</span>

**mkdir** 和 **rmdir** 函数

`mkdir`函数：创建目录
```c
#include <sys/stat.h>

int mkdir(const char *pathname, mode_t mode);

    Returns: 0 if OK, -1 on error
```
此函数创建一个空目录。其中 `.` 和 `..` 目录项是自动创建的。所指定的文件访问权限`mode`由进程的文件模式创建屏蔽字修改。常见的错误是指定与文件相同的`mode`（只指定读、写权限），对于目录，通常至少要设置`1`个执行权限位，以允许访问该目录中的文件名。

`rmdir`函数：删除一个空目录，空目录是只包含 `.` 和 `..` 这两项的目录。
```c
#include <unistd.h>

int rmdir(const char *pathname);

    Returns: 0 if OK, -1 on error
```

**读目录**

对某个目录具有访问权限的任一用户都可读该目录，但是，为了防止文件系统产生混乱，只有内核才能写目录。一个目录的写权限位和执行权限位决定了在该目录中能否创建新文件以及删除文件，它们并不表示能否写目录本身。

目录的实际格式依赖于UNIX系统，特别是其文件系统的具体设计和实现。早期的系统有一个比较简单的结构：每个目录项是`16`个字节，其中`14`个字节是文件名，`2`个字节是i节点编号。而对于`4`.2BSD而言，由于它允许相当长的文件名，所以每个目录项的长度是可变的。这就意味着读目录的程序与系统相关。为简化这种情况，UNIX现在包含了一套与读目录相关的例程，它们是POSIX.1的一部分。很多实现阻止应用程序使用`read`函数读取目录的内容，从而进一步将应用程序与目录格式中与实现相关的细节隔离开。
```c
#include <dirent.h>

DIR *opendir(const char *pathname);

    Returns: pointer if OK, NULL on error

struct dirent *readdir(DIR *dp);

    Returns: pointer if OK, NULL at end of directory or error

void rewinddir(DIR *dp);

int closedir(DIR *dp);

    Returns: 0 if OK, -1 on error

long telldir(DIR *dp);

    Returns: current location in directory associated with dp

void seekdir(DIR *dp, long loc);
```
`telldir`和`seekdir`函数不是基本POSIX.1标准的组成部分，它们是Single UNIX Specification中的XSI扩展，所以所有遵循UNIX系统的实现都会提供这两个函数。

头文件&lt;dirent.h&gt;中定义的`dirent`结构与实现有关。几种典型的UNIX实现对此结构所作的定义至少包含下列两个成员：
```c
struct dirent {
    ino_t d_ino;                  /* i-node number */
    char  d_name[NAME_MAX + 1];   /* null-terminated filename */
}
```
`NAME_MAX` 的常用值是`255`。因为文件名是以`null`字符结束的，所以在头文件中如何定义数组`d_name`并无多大关系，数组大小并不表示文件名的长度。

`DIR` 结构是一个内部结构，上述6个函数用这个内部结构保存当前正被读的目录的有关信息。其作用类似于 `FILE` 结构，`FILE`结构由标准I/O库维护。

由`opendir`返回的指向`DIR`结构的指针由另外5个函数使用。`opendir`执行初始化操作，使第一个`readdir`读目录中的第一个目录项。目录中各目录项的顺序与实现有关，它们通常并不按字母顺序排列。

**chdir**、**fchdir**和**getcwd**函数

每个进程都有一个当前工作目录，此目录是搜索所有相对路径名的起点（不以斜杠开始的路径名为相对路径名）。当用户登录到UNIX系统时，其当前工作目录通常是口令文件（`/etc/passwd`）中该用户登录项的第6个字段——用户的起始目录（home directory）。当前工作目录是进程的一个属性，起始目录则是登录名的一个属性。

进程通过调用`chdir`或`fchdir`函数更改当前工作目录。
```c
#include <unistd.h>

int chdir(const char *pathname);
int fchdir(int filedes);

    Both return: 0 if OK, -1 on error
```
在这两个函数中，分别用`pathname`或打开的文件描述符来指定新的当前工作目录。

> `fchdir`不是基本POSIX.1规范的所属部分，在Single UNIX Specification中，它是XSI扩展部分。

因为当前工作目录是进程的一个属性，所以它只影响调用`chdir`的进程本身，而不影响其他进程。

chdir函数示例：
```c
#include "apue.h"

int
main(void)
{

     if (chdir("/tmp") < 0)
         err_sys("chdir failed");
     printf("chdir to /tmp succeeded\n");
     exit(0);
}
```
如果编译执行上述程序，并且调用其可执行目标代码文件`mycd`，则可以得到下列结果：
```bash
$ pwd
/usr/lib
$ mycd
chdir to /tmp succeeded
$ pwd
/usr/lib
```
从中可以看出，执行`mycd`程序的shell的当前工作目录并没有改变，其原因是shell创建了一个子进程，由该子进程具体执行`mycd`程序。由此可见，为了改变shell进程自己的工作目录，shell应当直接调用`chdir`函数，为此 `cd` 命令的执行程序直接包含在shell程序中（内建命令）。

因为内核保持有当前工作目录的信息，所以我们应能取其当前值。不幸的是，内核为每个进程只保存指向该目录 `v` 节点的指针等目录本身的信息，并不保存该目录的完整路径名。

我们需要一个函数，它从当前工作目录（ `.` 目录 ）开始，用 `..` 目录项找到其上一级目录，然后读其目录项，直到该目录项中的 `i` 节点编号与工作目录 `i` 节点编号相同，这样就找到了其对应的文件名。按照这种方法，逐层上移，直到遇到根（ `/` ），这样就得到了当前工作目录完整的绝对路径名。很幸运，函数`getcwd`就提供了这种功能。
```c
#include <unistd.h>

char *getcwd(char *buf, size_t size);

    Returns: buf if OK, NULL on error
```
向此函数传递两个参数，一个是缓冲地址`buf`，另一个是缓冲的长度`size`（单位：字节）。该缓冲必须有足够的长度以容纳绝对路径名再加上一个`null`终止符，否则返回出错。

`getcwd`函数使用示例：将工作目录更改至一个指定的目录，然后调用`getcwd`，最后打印该工作目录。
```c
#include "apue.h"

int
main(void)
{
    char    *ptr;
    int     size;
    if (chdir("/usr/spool/uucppublic") < 0)
        err_sys("chdir failed");

    ptr = path_alloc(&size); /* our own function */
    if (getcwd(ptr, size) == NULL)
        err_sys("getcwd failed");

    printf("cwd = %s\n", ptr);
    exit(0);
}
```
编译运行上述程序，可得：
```bash
$ ./a.out
cwd = /var/spool/uucppublic
$ ls -l /usr/spool
lrwxrwxrwx 1 root 12 Jan 31 07:57 /usr/spool -> ../var/spool
```
> 注意，`chdir`跟随符号链接，但是当`getcwd`沿目录树上溯到`/var/spool`目录时，它并不了解该目录由符号链接`/usr/spool`所指向。这是符号链接的一种特性。

当一个应用程序需要在文件系统中返回到其工作的起点时，`getcwd`函数是有用的。在更换工作目录之前，可以调用`getcwd`函数先将其保存起来。在完成了处理后，就可将从`getcwd`获得的路径名作为调用参数传送给`chdir`，这样就返回到了文件系统中的起点。  
`fchdir`函数提供了一种完成此任务的便捷方法。在更换到文件系统中的不同位置前，无需调用`getcwd`函数，而是使用`open`打开当前工作目录，然后保存文件描述符。当希望回到原工作目录时，只需要简单的将该文件描述符传递给`fchdir`。

## 设备特殊文件

`st_dev`和`st_rdev`这两个字段经常引起混淆，而使用这两个字段，有关规则很简单：

*   每个文件系统所在的存储设备都由其主、次设备号表示。设备号所用的数据类型是基本系统数据类型`dev_t`。主设备号标识设备驱动程序，次设备号标识特定的子设备。磁盘驱动器经常包含若干个文件系统，在同一磁盘驱动器上的各文件系统通常具有相同的主设备号，但它们的次设备号却不同。
*   通常可以使用两个宏`major`和`minor`来访问主、次设备号，大多数实现都定义了这两个宏。这就意味着无需关心这两个数是如何存放在`dev_t`对象中的。
*   系统中与每个文件名关联的`st_dev`值是文件系统的设备号，该文件系统包含了这一文件名以及与其对应的`i`节点。
*   只有字符特殊文件和块特殊文件才有`st_rdev`值。此值包含实际设备的设备号。

> POSIX.1说明`dev_t`类型是存在的，但没有定义它包含什么，或如何取得其内容。大多数实现定义了宏`major`和`minor`，但在哪一个头文件中定义它们则与实现有关。基于BSD的UNIX系统将它们定义在&lt;sys/types.h&gt;中；Solaris将它们定义在&lt;sys/mkdev.h&gt;中；Linux将它们定义在&lt;sys/sysmacros.h&gt;中，而该头文件又包括在&lt;sys/types.h&gt;中。

下面来看一个实例：打印`st_dev`和`st_rdev`值。
```c
#include "apue.h"
#ifdef SOLARIS
#include <sys/mkdev.h>
#endif

int
main(int argc, char *argv[])
{

    int         i;
    struct stat buf;

    for (i = 1; i < argc; i++) {
        printf("%s: ", argv[i]);
        if (stat(argv[i], &buf) < 0) {
            err_ret("stat error");
            continue;
         }

         printf("dev = %d/%d", major(buf.st_dev), minor(buf.st_dev));
         if (S_ISCHR(buf.st_mode) || S_ISBLK(buf.st_mode)) {
             printf(" (%s) rdev = %d/%d",
                     (S_ISCHR(buf.st_mode)) ? "character" : "block",
                     major(buf.st_rdev), minor(buf.st_rdev));

         }
         printf("\n");
    }

    exit(0);

}
```
运行此程序得到下面的结果：
```bash
$ ./a.out / /home/sar /dev/tty[01]
/: dev = 3/3
/home/sar: dev = 3/4
/dev/tty0: dev = 0/7 (character) rdev = 4/0
/dev/tty1: dev = 0/7 (character) rdev = 4/1
$ mount             #which directories are mounted on which devices?
/dev/hda3 on / type ext2 (rw,noatime)
/dev/hda4 on /home type ext2 (rw,noatime)
$ ls -lL /dev/tty[01] /dev/hda[34]
brw-------  1 root       3,   3 Dec 31  1969 /dev/hda3
brw-------  1 root       3,   4 Dec 31  1969 /dev/hda4
crw-------  1 root       4,   0 Dec 31  1969 /dev/tty0
crw-------  1 root       4,   1 Jan 18 15:36 /dev/tty1
```
传递给该程序的前两个参数是目录（`/` 和 `/home/sar`），后两个是设备名`/dev/tty[01]`。（用shell正则表达式语言以缩短设备名，shell将扩展该字符串`/dev/tty[01]`为`/dev/tty0` `/dev/tty1`）  
这两个设备是字符特殊设备。从程序的输出可见，根目录`(/)`和`/home/sar`目录的设备号不同，这表示它们位于不同的文件系统中。运行`mount`命令证明了这一点。  
然后用`ls`命令查看由`mount`命令报告的两个磁盘设备和两个终端设备。这两个磁盘设备是块特殊文件，而两个终端设备则是字符特殊文件。  
注意，两个终端设备（`st_dev`）的文件名和`i`节点在设备`0/7`上（`devfs`伪文件系统，它实现了`/dev`文件系统），但是它们的实际设备号是`4/0`和`4/1`。

## 小结

本章内容围绕`stat`函数，详细介绍了`stat`结构中的每一个成员。这使我们对UNIX文件的各个属性都有所了解。对于文件的所有属性以及操作文件的所有函数有完整的了解对UNIX编程是非常重要的。
