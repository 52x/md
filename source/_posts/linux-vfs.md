---
title: 从文件 I/O 看 Linux 的虚拟文件系统
tags:
  - VFS
categories:
  - Kernel
  - 内核文件系统
date: 2014-02-25 14:53:28
---

原文出处：<http://www.ibm.com/developerworks/cn/linux/l-cn-vfs/>

**简介：** Linux 允许众多不同的文件系统共存，并支持跨文件系统的文件操作，这是因为有虚拟文件系统的存在。<span style="color: #ff0102;">虚拟文件系统交换器</span>，即VFS（<span style="color: #ff0102;">Virtual Filesystem Switch</span>）是 Linux 内核中的一个软件抽象层。它通过一些数据结构及其方法向实际的文件系统如 ext2，vfat 提供接口机制。本文在简要介绍 VFS 的相关数据结构后，以文件 I/O 为切入点深入 Linux 内核源代码，追踪了 sys_open 和 sys_read 两个系统调用的代码结构，并在追踪的过程中理清了跨文件系统的文件操作的基本原理和“一切皆是文件”的口号得以实现的根本。
<!--more-->

## 引言

Linux 中允许众多不同的文件系统共存，如 ext2, ext3, vfat 等。通过使用同一套文件 I/O 系统 调用即可对 Linux 中的任意文件进行操作而无需考虑其所在的具体文件系统格式；更进一步，对文件的 操作可以跨文件系统而执行。如图 1 所示，我们可以使用 cp 命令从 vfat 文件系统格式的硬盘拷贝数据到 ext3 文件系统格式的硬盘；而这样的操作涉及到两个不同的文件系统。
图 1\. 跨文件系统的文件操作
{% asset_img vfs-fig1.jpg %}
“一切皆是文件”是 Unix/Linux 的基本哲学之一。不仅普通的文件，目录、字符设备、块设备、 套接字等在 Unix/Linux 中都是以文件被对待；它们虽然类型不同，但是对其提供的却是同一套操作界面。
图 2\. 一切皆是文件
{% asset_img vfs-fig2.jpg %}
而虚拟文件系统正是实现上述两点 Linux 特性的关键所在。虚拟文件系统交换器（<span style="color:red">Virtual Filesystem Switch</span>， 简称 VFS）， 是 Linux 内核中的一个软件层，用于给用户空间的程序提供文件系统接口；同时，它也提供了内核中的一个 抽象功能，允许不同的文件系统共存。系统中所有的文件系统不但依赖 VFS 共存，而且也依靠 VFS 协同工作。
为了能够支持各种实际文件系统，VFS 定义了所有文件系统都支持的基本的、概念上的接口和数据 结构；同时实际文件系统也提供 VFS 所期望的抽象接口和数据结构，将自身的诸如文件、目录等概念在形式 上与VFS的定义保持一致。换句话说，一个实际的文件系统想要被 Linux 支持，就必须提供一个符合VFS标准 的接口，才能与 VFS 协同工作。实际文件系统在统一的接口和数据结构下隐藏了具体的实现细节，所以在VFS 层和内核的其他部分看来，所有文件系统都是相同的。图3显示了VFS在内核中与实际的文件系统的协同关系。
图3\. VFS在内核中与其他的内核模块的协同关系
{% asset_img vfs-fig3.jpg %}
我们已经知道，正是由于在内核中引入了VFS，跨文件系统的文件操作才能实现，“一切皆是文件” 的口号才能承诺。而为什么引入了VFS，就能实现这两个特性呢？在接下来，我们将以这样的一个思路来切入文章的正题：我们将先简要介绍下用以描述VFS模型的一些数据结构，总结出这些数据结构相互间的关系；然后 选择两个具有代表性的文件I/O操作`sys_open()`和`sys_read()`来详细说明内核是如何借助VFS和具体的文件系统打 交道以实现跨文件系统的文件操作和承诺“一切皆是文件”的口号

## VFS数据结构

### 一些基本概念

从本质上讲，文件系统是特殊的数据分层存储结构，它包含文件、目录和相关的控制信息。为了描述 这个结构，Linux引入了一些基本概念:
**文件** 一组在逻辑上具有完整意义的信息项的系列。在Linux中，除了普通文件，其他诸如目录、设备、套接字等 也以文件被对待。总之，“一切皆文件”。
**目录** 目录好比一个文件夹，用来容纳相关文件。因为目录可以包含子目录，所以目录是可以层层嵌套，形成 文件路径。在Linux中，目录也是以一种特殊文件被对待的，所以用于文件的操作同样也可以用在目录上。
**目录项** 在一个文件路径中，路径中的每一部分都被称为目录项；如路径/home/source/helloworld.c中，目录 /, home, source和文件 helloworld.c都是一个目录项。
**索引节点** 用于存储文件的元数据的一个数据结构。文件的元数据，也就是文件的相关信息，和文件本身是两个不同 的概念。它包含的是诸如文件的大小、拥有者、创建时间、磁盘位置等和文件相关的信息。
**超级块** 用于存储文件系统的控制信息的数据结构。描述文件系统的状态、文件系统类型、大小、区块数、索引节 点数等，存放于磁盘的特定扇区中。
如上的几个概念在磁盘中的位置关系如图4所示。
图4\. 磁盘与文件系统
{% asset_img vfs-fig4.jpg %}
关于文件系统的三个易混淆的概念：
**创建** 以某种方式格式化磁盘的过程就是在其之上建立一个文件系统的过程。创建文现系统时，会在磁盘的特定位置写入 关于该文件系统的控制信息。
**注册** 向内核报到，声明自己能被内核支持。一般在编译内核的时侯注册；也可以加载模块的方式手动注册。注册过程实 际上是将表示各实际文件系统的数据结构`struct file_system_type`实例化。
**安装** 也就是我们熟悉的mount操作，将文件系统加入到Linux的根文件系统的目录树结构上；这样文件系统才能被访问。

### VFS数据结构

VFS依靠四个主要的数据结构和一些辅助的数据结构来描述其结构信息，这些数据结构表现得就像是对象；每个主要对象中都包含由操作函数表构成的操作对象，这些操作对象描述了内核针对这几个主要的对象可以进行的操作。

#### 超级块对象

存储一个已安装的文件系统的控制信息，代表一个已安装的文件系统；每次一个实际的文件系统被安装时，内核会从磁盘的特定位置读取一些控制信息来填充内存中的超级块对象。一个安装实例和一个超级块对象一一对应。 超级块通过其结构中的一个域`s_type`记录它所属的文件系统类型。
根据第三部分追踪源代码的需要，以下是对该超级块结构的部分相关成员域的描述，（如下同）：
清单1\. 超级块
```c
struct super_block { //超级块数据结构
       struct list_head s_list;                /*指向超级块链表的指针*/
        ……
       struct file_system_type  *s_type;       /*文件系统类型*/
       struct super_operations  *s_op;         /*超级块方法*/
        ……
        struct list_head         s_instances;   /*该类型文件系统*/
        ……
};

struct super_operations { //超级块方法
        ……
        //该函数在给定的超级块下创建并初始化一个新的索引节点对象
        struct inode *(*alloc_inode)(struct super_block *sb);
       ……
        //该函数从磁盘上读取索引节点，并动态填充内存中对应的索引节点对象的剩余部分
        void (*read_inode) (struct inode *);
       ……
};
```

#### 索引节点对象

索引节点对象存储了文件的相关信息，代表了存储设备上的一个实际的物理文件。当一个文件首次被访问时，内核会在内存中组装相应的索引节点对象，以便向内核提供对一个文件进行操作时所必需的全部信息；这些信息一部分存储在磁盘特定位置，另外一部分是在加载时动态填充的。
清单2\. 索引节点
```c
struct inode { //索引节点结构
      ……
     struct inode_operations  *i_op;     /*索引节点操作表*/
     struct file_operations   *i_fop;	 /*该索引节点对应文件的文件操作集*/
     struct super_block       *i_sb;     /*相关的超级块*/
     ……
};

struct inode_operations { //索引节点方法
     ……
     //该函数为dentry对象所对应的文件创建一个新的索引节点，主要是由open()系统调用来调用
     int (*create) (struct inode *,struct dentry *,int, struct nameidata *);

     //在特定目录中寻找dentry对象所对应的索引节点
     struct dentry * (*lookup) (struct inode *,struct dentry *, struct nameidata *);
     ……
};
```

#### 目录项对象

引入目录项的概念主要是出于方便查找文件的目的。一个路径的各个组成部分，不管是目录还是 普通的文件，都是一个目录项对象。如，在路径/home/source/test.c中，目录 /, home, source和文件test.c都对应一个目录项对象。不同于前面的两个对象，目录项对象没有对应的磁盘数据结构，VFS在遍 历路径名的过程中现场将它们逐个地解析成目录项对象。
清单3\. 目录项
```c
struct dentry { //目录项结构
     ……
    struct inode *d_inode;           /*相关的索引节点*/
    struct dentry *d_parent;         /*父目录的目录项对象*/
    struct qstr d_name;              /*目录项的名字*/
    ……
     struct list_head d_subdirs;      /*子目录*/
    ……
     struct dentry_operations *d_op;  /*目录项操作表*/
    struct super_block *d_sb;        /*文件超级块*/
    ……
};

struct dentry_operations {
    //判断目录项是否有效;
    int (*d_revalidate)(struct dentry *, struct nameidata *);
    //为目录项生成散列值;
    int (*d_hash) (struct dentry *, struct qstr *);
    ……
};
```

#### 文件对象

文件对象是已打开的文件在内存中的表示，主要用于建立进程和磁盘上的文件的对应关系。它由`sys_open()`现场创建，由`sys_close()`销毁。文件对象和物理文件的关系有点像进程和程序的关系一样。当我们站在用户空间来看待VFS，我们像是只需与文件对象打交道，而无须关心超级块，索引节点或目录项。因为多个进程可以同时打开和操作同一个文件，所以同一个文件也可能存在多个对应的文件对象。文件对象仅仅在进程观点上代表已经打开的文件，它反过来指向目录项对象（反过来指向索引节点）。一个文件对应的文件对象可能不是惟一的，但是其对应的索引节点和目录项对象无疑是惟一的。
清单4\. 文件对象
```c
struct file {
    ……
    struct list_head        f_list;        /*文件对象链表*/
    struct dentry          *f_dentry;      /*相关目录项对象*/
    struct vfsmount        *f_vfsmnt;      /*相关的安装文件系统*/
    struct file_operations *f_op;          /*文件操作表*/
    ……
};

struct file_operations {
    ……
    //文件读操作
    ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
    ……
    //文件写操作
    ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
    ……
    int (*readdir) (struct file *, void *, filldir_t);
    ……
    //文件打开操作
    int (*open) (struct inode *, struct file *);
    ……
};
```

#### 其他VFS对象

##### <span style="color:red">和文件系统相关</span>

根据文件系统所在的物理介质和数据在物理介质上的组织方式来区分不同的文件系统类型的。`file_system_type`结构用于描述具体的文件系统的类型信息。被Linux支持的文件系统，都有且仅有一个`file_system_type`结构而不管它有零个或多个实例被安装到系统中。
而与此对应的是每当一个文件系统被实际安装，就有一个`vfsmount`结构体被创建，这个结构体对应一个安装点。
清单5\. 和文件系统相关
```c
struct file_system_type {
        const char *name;                /*文件系统的名字*/
        struct subsystem subsys;         /*sysfs子系统对象*/
        int fs_flags;                    /*文件系统类型标志*/

        /*在文件系统被安装时，从磁盘中读取超级块,在内存中组装超级块对象*/
        struct super_block *(*get_sb) (struct file_system_type*,
                                        int, const char*, void *);

        void (*kill_sb) (struct super_block *);  /*终止访问超级块*/
        struct module *owner;                    /*文件系统模块*/
        struct file_system_type * next;          /*链表中的下一个文件系统类型*/
        struct list_head fs_supers;              /*具有同一种文件系统类型的超级块对象链表*/
};

struct vfsmount
{
        struct list_head mnt_hash;               /*散列表*/
        struct vfsmount *mnt_parent;             /*父文件系统*/
        struct dentry *mnt_mountpoint;           /*安装点的目录项对象*/
        struct dentry *mnt_root;                 /*该文件系统的根目录项对象*/
        struct super_block *mnt_sb;              /*该文件系统的超级块*/
        struct list_head mnt_mounts;             /*子文件系统链表*/
        struct list_head mnt_child;              /*子文件系统链表*/
        atomic_t mnt_count;                      /*使用计数*/
        int mnt_flags;                           /*安装标志*/
        char *mnt_devname;                       /*设备文件名*/
        struct list_head mnt_list;               /*描述符链表*/
        struct list_head mnt_fslink;             /*具体文件系统的到期列表*/
        struct namespace *mnt_namespace;         /*相关的名字空间*/
};
```
##### <span style="color:red">和进程相关</span>

清单6\. 打开的文件集
```c
struct files_struct {//打开的文件集
        atomic_t count;              /*结构的使用计数*/
        ……
        int max_fds;                 /*文件对象数的上限*/
        int max_fdset;               /*文件描述符的上限*/
        int next_fd;                 /*下一个文件描述符*/
        struct file ** fd;           /*全部文件对象数组*/
        ……
 };

struct fs_struct {//建立进程与文件系统的关系
         atomic_t count;              /*结构的使用计数*/
        rwlock_t lock;               /*保护该结构体的锁*/
        int umask；                  /*默认的文件访问权限*/
        struct dentry * root;        /*根目录的目录项对象*/
        struct dentry * pwd;         /*当前工作目录的目录项对象*/
        struct dentry * altroot；    /*可供选择的根目录的目录项对象*/
        struct vfsmount * rootmnt;   /*根目录的安装点对象*/
        struct vfsmount * pwdmnt;    /*pwd的安装点对象*/
        struct vfsmount * altrootmnt;/*可供选择的根目录的安装点对象*/
};
```

##### <span style="color:red">和路径查找相关</span>

清单7\. 辅助查找
```c
struct nameidata {
        struct dentry  *dentry;     /*目录项对象的地址*/
        struct vfsmount  *mnt;      /*安装点的数据*/
        struct qstr  last;          /*路径中的最后一个component*/
        unsigned int  flags;        /*查找标识*/
        int  last_type;             /*路径中的最后一个component的类型*/
        unsigned  depth;            /*当前symbolic link的嵌套深度，不能大于6*/
        char   *saved_names[MAX_NESTED_LINKS + 1];/
                                    /*和嵌套symbolic link 相关的pathname*/
        union {
            struct open_intent open; /*说明文件该如何访问*/
        } intent;   /*专用数据*/
};
```

#### 对象间的联系

如上的数据结构并不是孤立存在的。正是通过它们的有机联系，VFS才能正常工作。如下的几张图是对它们之间的联系的描述。
如图5所示，被Linux支持的文件系统，都有且仅有一个`file_system_type`结构而不管它有零个或多个实例被安装到系统中。每安装一个文件系统，就对应有一个超级块和安装点。超级块通过它的一个域`s_type`指向其对应的具体的文件系统类型。具体的文件系统通过`file_system_type`中的一个域`fs_supers`链接具有同一种文件类型的超级块。同一种文件系统类型的超级块通过域`s_instances`链接。
图5\. 超级块、安装点和具体的文件系统的关系
{% asset_img vfs-fig5.jpg %}
从图6可知：进程通过`task_struct`中的一个域`files_struct files`来了解它当前所打开的文件对象；而我们通常所说的文件描述符其实是进程打开的文件对象数组的索引值。文件对象通过域`f_dentry`找到它对应的`dentry`对象，再由`dentry`对象的域`d_inode`找到它对应的索引结点，这样就建立了文件对象与实际的物理文件的关联。最后，还有一点很重要的是,文件对象所对应的文件操作函数列表是通过索引结点的域`i_fop`得到的。图6对第三部分源码的理解起到很大的作用。
图6\. 进程与超级块、文件、索引结点、目录项的关系
{% asset_img vfs-fig6.jpg %}

## 基于VFS的文件I/O

到目前为止，文章主要都是从理论上来讲述VFS的运行机制；接下来我们将深入源代码层中，通过阐述两个具有代表性的系统调用`sys_open()`和`sys_read()`来更好地理解VFS向具体文件系统提供的接口机制。由于本文更关注的是文件操作的整个流程体制，所以我们在追踪源代码时，对一些细节性的处理不予关心。又由于篇幅所限，只列出相关代码。本文中的源代码来自于linux-2.6.17内核版本。
在深入`sys_open()`和`sys_read()`之前，我们先概览下调用`sys_read()`的上下文。图7描述了从用户空间的`read()`调用到数据从磁盘读出的整个流程。当在用户应用程序调用文件I/O `read()`操作时，系统调用`sys_read()`被激发，`sys_read()`找到文件所在的具体文件系统，把控制权传给该文件系统，最后由具体文件系统与物理介质交互，从介质中读出数据。
图7\. 从物理介质读数据的过程
{% asset_img vfs-fig7.jpg %}

### sys_open()

`sys_open()`系统调用打开或创建一个文件，成功返回该文件的文件描述符。图8是`sys_open()`实现代码中主要的函数调用关系图
图8\. `sys_open`函数调用关系图
{% asset_img vfs-fig8.jpg %}
由于`sys_open()`的代码量大，函数调用关系复杂，以下主要是对该函数做整体的解析；而对其中的一些关键点，则列出其关键代码。
**a. 从`sys_open()`的函数调用关系图可以看到，`sys_open()`在做了一些简单的参数检验后，就把接力棒传给`do_sys_open()`：**
1）、首先，`get_unused_fd()`得到一个可用的文件描述符；通过该函数，可知文件描述符实质是进程打开文件列表中对应某个文件对象的索引值；
2）、接着，`do_filp_open()`打开文件，返回一个`file`对象，代表由该进程打开的一个文件；进程通过这样的一个数据结构对物理文件进行读写操作。
3）、最后，`fd_install()`建立文件描述符与`file`对象的联系，以后进程对文件的读写都是通过操纵该文件描述符而进行。
**b. `do_filp_open()`用于打开文件，返回一个`file`对象；而打开之前需要先找到该文件：**
1）、`open_namei()`用于根据文件路径名查找文件，借助一个持有路径信息的数据结构`nameidata`而进行；
2）、查找结束后将填充有路径信息的`nameidata`返回给接下来的函数`nameidata_to_filp()`从而得到最终的`file`对象；当达到目的后，`nameidata`这个数据结构将会马上被释放。
**c. `open_namei()`用于查找一个文件：**
1）、`path_lookup_open()`实现文件的查找功能；要打开的文件若不存在，还需要有一个新建的过程，则调用`path_lookup_create()`，后者和前者封装的是同一个实际的路径查找函数，只是参数不一样，使它们在处理细节上有所偏差；
2）、当是以新建文件的方式打开文件时，即设置了`O_CREAT`标识时需要创建一个新的索引节点，代表创建一个文件。在`vfs_create()`里的一句核心语句`dir->i_op->create(dir, dentry, mode, nd)`可知它调用了具体的文件系统所提供的创建索引节点的方法。注意：这边的索引节点的概念，还只是位于内存之中，它和磁盘上的物理的索引节点的关系就像位于内存中和位于磁盘中的文件一样。此时新建的索引节点还不能完全标志一个物理文件的成功创建，只有当把索引节点回写到磁盘上才是一个物理文件的真正创建。想想我们以新建的方式打开一个文件，对其读写但最终没有保存而关闭，则位于内存中的索引节点会经历从新建到消失的过程，而磁盘却始终不知道有人曾经想过创建一个文件，这是因为索引节点没有回写的缘故。
3）、`path_to_nameidata()`填充`nameidata`数据结构；
4）、`may_open()`检查是否可以打开该文件；一些文件如链接文件和只有写权限的目录是不能被打开的，先检查`nd->dentry->inode`所指的文件是否是这一类文件，是的话则错误返回。还有一些文件是不能以`TRUNC`的方式打开的，若`nd->dentry->inode`所指的文件属于这一类，则显式地关闭`TRUNC`标志位。接着如果有以`TRUNC`方式打开文件的，则更新`nd->dentry->inode`的信息

#### __path_lookup_intent_open()

不管是`path_lookup_open()`还是`path_lookup_create()`最终都是调用`__path_lookup_intent_open()`来实现查找文件的功能。查找时，在遍历路径的过程中，会逐层地将各个路径组成部分解析成目录项对象，如果此目录项对象在目录项缓存中，则直接从缓存中获得；如果该目录项在缓存中不存在，则进行一次实际的读盘操作，从磁盘中读取该目录项所对应的索引节点。得到索引节点后，则建立索引节点与该目录项的联系。如此循环，直到最终找到目标文件对应的目录项，也就找到了索引节点，而由索引节点找到对应的超级块对象就可知道该文件所在的文件系统的类型。从磁盘中读取该目录项所对应的索引节点；这将引发VFS和实际的文件系统的一次交互。从前面的VFS理论介绍可知，读索引节点方法是由超级块来提供的。而当安装一个实际的文件系统时，在内存中创建的超级块的信息是由一个实际文件系统的相关信息来填充的，这里的相关信息就包括了实际文件系统所定义的超级块的操作函数列表，当然也就包括了读索引节点的具体执行方式。当继续追踪一个实际文件系统ext3的ext3_read_inode()时，可发现这个函数很重要的一个工作就是为不同的文件类型设置不同的索引节点操作函数表和文件操作函数表。
清单8\. `ext3_read_inode`
```c
    void ext3_read_inode(struct inode * inode)
    {
       ……
       //是普通文件
       if (S_ISREG(inode->i_mode)) {
          inode->i_op = &ext3_file_inode_operations;
          inode->i_fop = &ext3_file_operations;
          ext3_set_aops(inode);
       } else if (S_ISDIR(inode->i_mode)) {
          //是目录文件
          inode->i_op = &ext3_dir_inode_operations;
          inode->i_fop = &ext3_dir_operations;
       } else if (S_ISLNK(inode->i_mode)) {
          // 是连接文件
          ……
       } else {
          // 如果以上三种情况都排除了，则是设备驱动
          //这里的设备还包括套结字、FIFO等伪设备
          ……
}
```

#### nameidata_to_filp子函数：__dentry_open

这是VFS与实际的文件系统联系的一个关键点。从3.1.1小节分析中可知，调用实际文件系统读取索引节点的方法读取索引节点时，实际文件系统会根据文件的不同类型赋予索引节点不同的文件操作函数集，如普通文件有普通文件对应的一套操作函数，设备文件有设备文件对应的一套操作函数。这样当把对应的索引节点的文件操作函数集赋予文件对象，以后对该文件进行操作时，比如读操作，VFS虽然对各种不同文件都是执行同一个read()操作界面，但是真正读时，内核却知道怎么区分对待不同的文件类型。
清单9\. `__dentry_open`
```c
    static struct file *__dentry_open(struct dentry *dentry, struct vfsmount *mnt,
					int flags, struct file *f,
					int (*open)(struct inode *, struct file *))
    {
        struct inode *inode;
        ……
        //整个函数的工作在于填充一个file对象
        ……
         f->f_mapping = inode->i_mapping;
        f->f_dentry = dentry;
        f->f_vfsmnt = mnt;
        f->f_pos = 0;
        //将对应的索引节点的文件操作函数集赋予文件对象的操作列表
        f->f_op = fops_get(inode->i_fop);
        ……
        //若文件自己定义了open操作，则执行这个特定的open操作。
        if (!open && f->f_op)
           open = f->f_op->open;
        if (open) {
           error = open(inode, f);
           if (error)
              goto cleanup_all;
        ……
        return f;
}
```

### sys_read()

`sys_read()`系统调用用于从已打开的文件读取数据。如read成功，则返回读到的字节数。如已到达文件的尾端，则返回0。图9是`sys_read()`实现代码中的函数调用关系图。
图9\. `sys_read`函数调用关系图
{% asset_img vfs-fig9.jpg %}
对文件进行读操作时，需要先打开它。从3.1小结可知，打开一个文件时，会在内存组装一个文件对象，希望对该文件执行的操作方法已在文件对象设置好。所以对文件进行读操作时，VFS在做了一些简单的转换后（由文件描述符得到其对应的文件对象；其核心思想是返回`current->files->fd[fd]`所指向的文件对象），就可以通过语句`file->f_op->read(file, buf, count, pos)`轻松调用实际文件系统的相应方法对文件进行读操作了。

## 4 解决问题

### 跨文件系统的文件操作的基本原理

到此，我们也就能够解释在Linux中为什么能够跨文件系统地操作文件了。举个例子，将vfat格式的磁盘上的一个文件a.txt拷贝到ext3格式的磁盘上，命名为b.txt。这包含两个过程，对a.txt进行读操作，对b.txt进行写操作。读写操作前，需要先打开文件。由前面的分析可知，打开文件时，VFS会知道该文件对应的文件系统格式，以后操作该文件时，VFS会调用其对应的实际文件系统的操作方法。所以，VFS调用vfat的读文件方法将a.txt的数据读入内存；在将a.txt在内存中的数据映射到b.txt对应的内存空间后，VFS调用ext3的写文件方法将b.txt写入磁盘；从而实现了最终的跨文件系统的复制操作。

### “一切皆是文件”的实现根本

不论是普通的文件，还是特殊的目录、设备等，VFS都将它们同等看待成文件，通过同一套文件操作界面来对它们进行操作。操作文件时需先打开；打开文件时，VFS会知道该文件对应的文件系统格式；当VFS把控制权传给实际的文件系统时，实际的文件系统再做出具体区分，对不同的文件类型执行不同的操作。这也就是“一切皆是文件”的根本所在。

## 5 总结

VFS即虚拟文件系统是Linux文件系统中的一个抽象软件层；因为它的支持，众多不同的实际文件系统才能在Linux中共存，跨文件系统操作才能实现。VFS借助它四个主要的数据结构即超级块、索引节点、目录项和文件对象以及一些辅助的数据结构，向Linux中不管是普通的文件还是目录、设备、套接字等都提供同样的操作界面，如打开、读写、关闭等。只有当把控制权传给实际的文件系统时，实际的文件系统才会做出区分，对不同的文件类型执行不同的操作。由此可见，正是有了VFS的存在，跨文件系统操作才能执行，Unix/Linux中的“一切皆是文件”的口号才能够得以实现。

## 参考文献

[1].Claudia Salzberg Rodriguez, Gordon Fischer, Steven Smolski. The Linux Kernel Primer.机械工业出版社.2006.7
[2].Robert Love.Linux内核设计与实现(第二版).机械工业出版社.2007.1
[3].Stevens W.Richard.Unix环境高级编程(第二版).人民邮电出版社.2006
[4].杨芙清，陈向群.操作系统教程.北京大学出版社.2005.7
[5].Linux-2.6.17.13内核源代码
