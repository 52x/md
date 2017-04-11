---
title: Linux文件系统之sysfs
tags:
  - VFS
  - 设备模型
categories:
  - Kernel
  - 内核文件系统
date: 2014-02-28 11:01:21
---

原文链接：<http://blog.chinaunix.net/uid-20543183-id-1930812.html>

源码文件：
`./fs/sysfs/`

## 前言

在设备模型中，sysfs文件系统用来表示设备的结构，将设备的层次结构形象的反应到用户空间中。用户空间可以修改sysfs中的文件属性来修改设备的属性值，今天我们就来详细分析一下sysfs的实现。
<!--more-->

## sysfs的初始化和挂载

sysfs文件系统的初始化是在`sysfs_init()`中完成的,代码如下:
```c
int __init sysfs_init(void)
{
       int err = -ENOMEM;
       //创建一个分配sysfs_dirent的cache
       sysfs_dir_cachep = kmem_cache_create("sysfs_dir_cache",
                                         sizeof(struct sysfs_dirent),
                                         0, 0, NULL);
       if (!sysfs_dir_cachep)
              goto out;

       err = sysfs_inode_init();
       if (err)
              goto out_err;
       //注册sysfs文件系统s
       err = register_filesystem(&sysfs_fs_type);
       if (!err) {
              //挂载sysfs文件系统
              sysfs_mount = kern_mount(&sysfs_fs_type);
              if (IS_ERR(sysfs_mount)) {
                     printk(KERN_ERR "sysfs: could not mount!\n");
                     err = PTR_ERR(sysfs_mount);
                     sysfs_mount = NULL;
                     unregister_filesystem(&sysfs_fs_type);
                     goto out_err;
              }
       } else
              goto out_err;
out:
       return err;
out_err:
       kmem_cache_destroy(sysfs_dir_cachep);
       sysfs_dir_cachep = NULL;
       goto out;
}
```
每个`kobject`对应sysfs中的一个目录，`kobject`的每个属性对应sysfs文件系统中的文件。
`struct sysfs_dirent`就是用来做`kobject`与`dentry`的互相转换用的，它们的关系如下图所示:
{% asset_img sysfs.jpg %}
上图表示的是一个`kobject`的层次结构，`dentry`的`d_fsdata`字段指定该结点所表示的`sysfs_dirent`，`sysfs_dirent.s_parent`表示它的父`kobject`，`sysfs_dirent`.`s_sibling`表示它的兄弟结点，`sysfs_dirent.s_dir.children`表示它所属的子节点。

从上图可知，如果要遍历一个结点下面的子结点，只需要找到`sysfs_dirent.s_dir.children`结点，然后按着子节点的`s_sibling`域遍历即可。当然，有时候也需要从`struct sysfs_dirent`导出它所属的`dentry`结点，我们在代码中遇到的时候再进行分析。
sysfs文件系统的`file_system_type`定义如下:
```c
static struct file_system_type sysfs_fs_type = {
       .name       = "sysfs",
       .get_sb     = sysfs_get_sb,
       .kill_sb    = kill_anon_super,
};
```
通过前面文件系统的相关分析,我们知道在`sys_mount()`中最终会调用`struct file_system_type`的`get_sb`函数来实现文件系统的挂载.它的代码如下:
```c
static int sysfs_get_sb(struct file_system_type *fs_type,
       int flags, const char *dev_name, void *data, struct vfsmount *mnt)
{
       return get_sb_single(fs_type, flags, data, sysfs_fill_super, mnt);
}
```
`get_sb_single()`的代码在前面已经涉及到,它对`super_block`以及挂载的`dentry`和`inode`的赋值是在回调函数`sysfs_fill_super`、`mnt`中完成的.代码如下:
```c
static int sysfs_fill_super(struct super_block *sb, void *data, int silent)
{
       struct inode *inode;
       struct dentry *root;

       sb->s_blocksize = PAGE_CACHE_SIZE;
       sb->s_blocksize_bits = PAGE_CACHE_SHIFT;
       sb->s_magic = SYSFS_MAGIC;
       sb->s_op = &sysfs_ops;
       sb->s_time_gran = 1;
       sysfs_sb = sb;

       /* get root inode, initialize and unlock it */
       inode = sysfs_get_inode(&sysfs_root);
       if (!inode) {
              pr_debug("sysfs: could not get root inode\n");
              return -ENOMEM;
       }

       /* instantiate and link root dentry */
       root = d_alloc_root(inode);
       if (!root) {
              pr_debug("%s: could not get root dentry!\n",__FUNCTION__);
              iput(inode);
              return -ENOMEM;
       }
       //将sysfs_root关联到root
       root->d_fsdata = &sysfs_root;
       sb->s_root = root;
       return 0;
}
```
在这里要注意几个全局量. `sysfs_sb`表示sysfs文件系统的`super_block`，`sysfs_root`表示sysfs文件系统根目录的`struct sysfs_dirent`。`sysfs_get_inode(&sysfs_root)`用来将`sysfs_root`导出相应的`inode`，代码如下:
```c
struct inode * sysfs_get_inode(struct sysfs_dirent *sd)
{
       struct inode *inode;
       //以super_block和sd->s_ino为哈希值,到哈希表中寻找相应的inode.如果不存在,则新建
       inode = iget_locked(sysfs_sb, sd->s_ino);
       //对新生成的inode进行初始化
       if (inode && (inode->i_state & I_NEW))
              sysfs_init_inode(sd, inode);

       return inode;
}
```
首先,它以sysfs文件系统的`super_block`和`struct sysfs_dirent`的`s_ino`成员的值做为哈希值到哈希表中寻找相应的`inode`。如果在哈希表中不存在这个`inode`,那就新建一个,并将它链入到哈希表.之后,调用`sysfs_init_inode()`对生成的`inode`进行初始化.显然.在mount的时候是不会生成`inode`的.必定会进入`sysfs_init_inode()`函数.代码如下:
```c
static void sysfs_init_inode(struct sysfs_dirent *sd, struct inode *inode)
{
       struct bin_attribute *bin_attr;

       inode->i_blocks = 0;
       inode->i_mapping->a_ops = &sysfs_aops;
       inode->i_mapping->backing_dev_info = &sysfs_backing_dev_info;
       inode->i_op = &sysfs_inode_operations;
       inode->i_ino = sd->s_ino;
       lockdep_set_class(&inode->i_mutex, &sysfs_inode_imutex_key);

       if (sd->s_iattr) {
              /* sysfs_dirent has non-default attributes
               * get them for the new inode from persistent copy
               * in sysfs_dirent
               */
              set_inode_attr(inode, sd->s_iattr);
       } else
              set_default_inode_attr(inode, sd->s_mode);

       /* initialize inode according to type */
       switch (sysfs_type(sd)) {
       case SYSFS_DIR:
              inode->i_op = &sysfs_dir_inode_operations;
              inode->i_fop = &sysfs_dir_operations;
              inode->i_nlink = sysfs_count_nlink(sd);
              break;
       case SYSFS_KOBJ_ATTR:
              inode->i_size = PAGE_SIZE;
              inode->i_fop = &sysfs_file_operations;
              break;
       case SYSFS_KOBJ_BIN_ATTR:
              bin_attr = sd->s_bin_attr.bin_attr;
              inode->i_size = bin_attr->size;
              inode->i_fop = &bin_fops;
              break;
       case SYSFS_KOBJ_LINK:
              inode->i_op = &sysfs_symlink_inode_operations;
              break;
       default:
              BUG();
       }

       unlock_new_inode(inode);
}
```
在这里,我们可以看到sysfs文件系统中的各种操作函数了..

在syfs文件系统中,怎么样判断一个目录下是否有这个文件呢?
在前面有关文件系统的分析中我们可以看.有关文件的查找实际上都会由`inod->i_op->lookup()`函数进行判断.在sysfs中,这个函数对应为`sysfs_lookup()`.代码如下:
```c
static struct dentry * sysfs_lookup(struct inode *dir, struct dentry *dentry,
                            struct nameidata *nd)
{
       struct dentry *ret = NULL;
       //取得父结点对应的sysfs_dirent
       struct sysfs_dirent *parent_sd = dentry->d_parent->d_fsdata;
       struct sysfs_dirent *sd;
       struct inode *inode;

       mutex_lock(&sysfs_mutex);
       //父结点的sysfs_dirent中是否有相应的子结点
       sd = sysfs_find_dirent(parent_sd, dentry->d_name.name);

       /* no such entry */
       //如果没有.这个结点是不存在的
       if (!sd) {
              ret = ERR_PTR(-ENOENT);
              goto out_unlock;
       }

       /* attach dentry and inode */
       //如果有这个结点,为之生成inod结构.
       inode = sysfs_get_inode(sd);
       if (!inode) {
              ret = ERR_PTR(-ENOMEM);
              goto out_unlock;
       }

       /* instantiate and hash dentry */
       dentry->d_op = &sysfs_dentry_ops;
       //关联dentry与sysfs_dirent
       dentry->d_fsdata = sysfs_get(sd);
       d_instantiate(dentry, inode);
       d_rehash(dentry);

 out_unlock:
       mutex_unlock(&sysfs_mutex);
       return ret;
}
```
由此可见,它的判断会转入到相应的`sysfs_dirent`中进行判断.如果设备模型在创建目录/文件的时候并不会创建`dentry`或者`inode`.只会操作`sysfs_dirent`结构. 如果找到了这个结构,就为这个结构生成`inode`,并将其关联到`denry`中.
`sysfs_find_dirent()`如下:
```c
struct sysfs_dirent *sysfs_find_dirent(struct sysfs_dirent *parent_sd,
                                   const unsigned char *name)
{
       struct sysfs_dirent *sd;

       for (sd = parent_sd->s_dir.children; sd; sd = sd->s_sibling)
              if (!strcmp(sd->s_name, name))
                     return sd;
       return NULL;
}
```
它用的搜索方法就是我们在之前分析`sysfs_dirent`结构所讲述的.分析到这里,sysfs的大概轮廓就出现在我们的眼前了.^_^.接下来分析sysfs文件系统中目录的创建过程

## 在sysfs文件系统中创建目录

在linux设备模型中,每注册一个`kobject`.就会为之创建一个目录.具体的流程在分析linux设备模型的时候再给出详细的分析.创建目录的接口为: `sysfs_create_dir()`。代码如下:
```c
int sysfs_create_dir(struct kobject * kobj)
{
       struct sysfs_dirent *parent_sd, *sd;
       int error = 0;

       BUG_ON(!kobj);

       //如果kobject没有指定父结点,则将其父结点指定为sysfs的根目录syfs_root
       if (kobj->parent)
              parent_sd = kobj->parent->sd;
       else
              parent_sd = &sysfs_root;

       //创建目录
       error = create_dir(kobj, parent_sd, kobject_name(kobj), &sd);
       //kobj->sd 指向对应的sysfs_dirent
       if (!error)
              kobj->sd = sd;
       return error;
}
```
在这里,先为结点指定父目录,然后调用`create_dir()`在父目录下生成结点.代码如下:
```c
static int create_dir(struct kobject *kobj, struct sysfs_dirent *parent_sd,
                    const char *name, struct sysfs_dirent **p_sd)
{
       //指定目录的模式
       umode_t mode = S_IFDIR| S_IRWXU | S_IRUGO | S_IXUGO;
       struct sysfs_addrm_cxt acxt;
       struct sysfs_dirent *sd;
       int rc;

       /* allocate */
       //分配并初始化一个sysfs_dirent
       sd = sysfs_new_dirent(name, mode, SYSFS_DIR);
       if (!sd)
              return -ENOMEM;
       //初始化sd->s_dir.kobj字段
       sd->s_dir.kobj = kobj;

       /* link in */
       //acxt是一个临时变量.它用来存放父结点的相关信息

       //设置acxt->parent_sd为父结点的sysfs_dirent, acxt->parent_inode为父结点的inode
       sysfs_addrm_start(&acxt, parent_sd);
       //设置sd->s_parent.并按inod值按顺序链入父结点的children链表
       rc = sysfs_add_one(&acxt, sd);
       sysfs_addrm_finish(&acxt);

       if (rc == 0)
              *p_sd = sd;
       else
              sysfs_put(sd);

       return rc;
}
```
在这里,为子节点生成了对应的`sysfs_dirent`.设置了它的父结点域,并将其链入到父结点的`children`链表.这样,在文件系统中查找父目录下面的子结点了.

## 在sysfs中创建一般属性文件

`kobject`的每一项属性都对应在sysfs文件系统中`kobject`对应的目录下的一个文件，文件名称与属性名称相同。
创建一般属性的接口为`sysfs_create_file()`，代码如下:
```c
int sysfs_create_file(struct kobject * kobj, const struct attribute * attr)
{
       BUG_ON(!kobj || !kobj->sd || !attr);

       //kobject->sd: 为kobject表示目录的struct sysfs_dirent结构
       return sysfs_add_file(kobj->sd, attr, SYSFS_KOBJ_ATTR);

}
```
最终会调用`sysfs_add_file()`, 参数`attr`是要生成文件的属性值.
```c
int sysfs_add_file(struct sysfs_dirent *dir_sd, const struct attribute *attr,
                 int type)
{
       //文件对应的属性
       umode_t mode = (attr->mode & S_IALLUGO) | S_IFREG;
       struct sysfs_addrm_cxt acxt;
       struct sysfs_dirent *sd;
       int rc;

       //创建一个新的sysfs_dirent.对应的名称为attr->name.即属性的名称
       sd = sysfs_new_dirent(attr->name, mode, type);
       if (!sd)
              return -ENOMEM;
       //设置属性值
       sd->s_attr.attr = (void *)attr;

       //将子结点的struct sysfs_dirent结构关链到父结点
       sysfs_addrm_start(&acxt, dir_sd);
       rc = sysfs_add_one(&acxt, sd);
       sysfs_addrm_finish(&acxt);

       if (rc)
              sysfs_put(sd);

       return rc;
}
```
这个流程与创建目录的流程大部份相同.不相同的只是创建目录时,它的父目录为上一层结点,创建文件时,它的父目录就是`kobject`对应的`struct sysfs_dirent`.
这样,在`kobject`对应的目录下面就可以看到这个文件了.^_^

文件建好之后,要怎么样去读写呢? 回忆一下,在sysfs文件系统中,`inode`的初始化:
```c
static void sysfs_init_inode(struct sysfs_dirent *sd, struct inode *inode)
{
       ……
       …….
       case SYSFS_KOBJ_ATTR:
       inode->i_size = PAGE_SIZE;
       inode->i_fop = &sysfs_file_operations;
       ……
}
```
`sysfs_file_operations`的定义如下:
```c
const struct file_operations sysfs_file_operations = {
       .read       = sysfs_read_file,
       .write      = sysfs_write_file,
       .llseek     = generic_file_llseek,
       .open       = sysfs_open_file,
       .release    = sysfs_release,
       .poll       = sysfs_poll,
};
```
文件的操作全部都在这里了,我们从打开文件说起.
`sysfs_open_file()`代码如下:
```c
static int sysfs_open_file(struct inode *inode, struct file *file)
{
       struct sysfs_dirent *attr_sd = file->f_path.dentry->d_fsdata;
       struct kobject *kobj = attr_sd->s_parent->s_dir.kobj;
       struct sysfs_buffer *buffer;
       struct sysfs_ops *ops;
       int error = -EACCES;

       /* need attr_sd for attr and ops, its parent for kobj */
       if (!sysfs_get_active_two(attr_sd))
              return -ENODEV;

       /* every kobject with an attribute needs a ktype assigned */
       //将buffer->ops设置为kobj->ktype->sysfs_ops
       if (kobj->ktype && kobj->ktype->sysfs_ops)
              ops = kobj->ktype->sysfs_ops;
       else {
              printk(KERN_ERR "missing sysfs attribute operations for "
                     "kobject: %s\n", kobject_name(kobj));
              WARN_ON(1);
              goto err_out;
       }

       /* File needs write support.
        * The inode's perms must say it's ok,
        * and we must have a store method.
        */
       if (file->f_mode & FMODE_WRITE) {
              if (!(inode->i_mode & S_IWUGO) || !ops->store)
                     goto err_out;
       }

       /* File needs read support.
        * The inode's perms must say it's ok, and we there
        * must be a show method for it.
        */
       if (file->f_mode & FMODE_READ) {
              if (!(inode->i_mode & S_IRUGO) || !ops->show)
                     goto err_out;
       }

       /* No error? Great, allocate a buffer for the file, and store it
        * it in file->private_data for easy access.
        */
       error = -ENOMEM;
       buffer = kzalloc(sizeof(struct sysfs_buffer), GFP_KERNEL);
       if (!buffer)
              goto err_out;

       mutex_init(&buffer->mutex);
       buffer->needs_read_fill = 1;
       buffer->ops = ops;
       file->private_data = buffer;

       /* make sure we have open dirent struct */
       //将buffer链至attr_sd->s_attr.open链表上
       error = sysfs_get_open_dirent(attr_sd, buffer);
       if (error)
              goto err_free;

       /* open succeeded, put active references */
       sysfs_put_active_two(attr_sd);
       return 0;

 err_free:
       kfree(buffer);
 err_out:
       sysfs_put_active_two(attr_sd);
       return error;
}
```
在这段代码中,需要注意以下几个操作,
1、`buffer`链接在`file->private_data`，`buffer`还被链接在`sysfs_dirent->s_attr.open`，这样，VFS通过`file`，设备模型通过`kobject->sd->s_attr.open`都能找到这个要操作的 `buffer`。
2、`buffer->ops`被设置为了`kobject->ktype->sysfs_ops`。

文件的写操作入口如下:
```c
static ssize_t
sysfs_write_file(struct file *file, const char __user *buf, size_t count, loff_t *ppos)
{
       struct sysfs_buffer * buffer = file->private_data;
       ssize_t len;

       mutex_lock(&buffer->mutex);
       //将buf中的内容copy到了buffer->page
       len = fill_write_buffer(buffer, buf, count);
       //与设备模型的交互
       if (len > 0)
              len = flush_write_buffer(file->f_path.dentry, buffer, len);
       //更新ppos
       if (len > 0)
              *ppos += len;
       mutex_unlock(&buffer->mutex);
       return len;
}
```
首先，调用`fill_write_buffer()`将用户空间传值下来的数值copy到`buffer->page`，然后再调用`flush_write_buffer()`与设备模型进行交互。
`flush_wirte_buffer()`代码如下:
```c
static int
flush_write_buffer(struct dentry * dentry, struct sysfs_buffer * buffer, size_t count)
{
       struct sysfs_dirent *attr_sd = dentry->d_fsdata;
       struct kobject *kobj = attr_sd->s_parent->s_dir.kobj;
       struct sysfs_ops * ops = buffer->ops;
       int rc;

       /* need attr_sd for attr and ops, its parent for kobj */
       if (!sysfs_get_active_two(attr_sd))
              return -ENODEV;

       rc = ops->store(kobj, attr_sd->s_attr.attr, buffer->page, count);

       sysfs_put_active_two(attr_sd);

       return rc;
}
```
我们在分析`open()`操作的时候曾分析到，`buffer`的`ops`是`kobject->ktype->ops`.在这里,它相当于调用了`kobject->ktype->ops->store()`.参数分别为:操作的`kobject`，文件对应的属性，写入的值和值的长度。
Sysfs这样设计,主要是在VFS保持一个统一的接口,因为每一个`kobject`对应的属性值都不相同,相应的,操作方法也不一样,这样,在`ktype`中就区别开来了.

文件的读操作相应接口为`sysfs_read_file()`，代码如下:
```c
static ssize_t
sysfs_read_file(struct file *file, char __user *buf, size_t count, loff_t *ppos)
{
       struct sysfs_buffer * buffer = file->private_data;
       ssize_t retval = 0;

       mutex_lock(&buffer->mutex);
       //从设备模型中将值取出.并存入buffer->page中
       if (buffer->needs_read_fill) {
              retval = fill_read_buffer(file->f_path.dentry,buffer);
              if (retval)
                     goto out;
       }
       //将buffer->page中的值copy到用户空间的buf
       pr_debug("%s: count = %zd, ppos = %lld, buf = %s\n",
               __FUNCTION__, count, *ppos, buffer->page);
       retval = simple_read_from_buffer(buf, count, ppos, buffer->page,
                                    buffer->count);
out:
       mutex_unlock(&buffer->mutex);
       return retval;
}
```
读操作的流程刚好和写操作流程相反.它先从设备模型中取值,然后再copy到用户空间.
`fill_read_buffer`的代码如下:
```c
static int fill_read_buffer(struct dentry * dentry, struct sysfs_buffer * buffer)
{
       struct sysfs_dirent *attr_sd = dentry->d_fsdata;
       struct kobject *kobj = attr_sd->s_parent->s_dir.kobj;
       struct sysfs_ops * ops = buffer->ops;
       int ret = 0;
       ssize_t count;

       if (!buffer->page)
              buffer->page = (char *) get_zeroed_page(GFP_KERNEL);
       if (!buffer->page)
              return -ENOMEM;

       /* need attr_sd for attr and ops, its parent for kobj */
       if (!sysfs_get_active_two(attr_sd))
              return -ENODEV;

       buffer->event = atomic_read(&attr_sd->s_attr.open->event);
       count = ops->show(kobj, attr_sd->s_attr.attr, buffer->page);

       sysfs_put_active_two(attr_sd);

       /*
        * The code works fine with PAGE_SIZE return but it's likely to
        * indicate truncated result or overflow in normal use cases.
        */
       if (count >= (ssize_t)PAGE_SIZE) {
              print_symbol("fill_read_buffer: %s returned bad count\n",
                     (unsigned long)ops->show);
              /* Try to struggle along */
              count = PAGE_SIZE - 1;
       }
       if (count >= 0) {
              buffer->needs_read_fill = 0;
              buffer->count = count;
       } else {
              ret = count;
       }
       return ret;
}
```
在这里,我们看到,最终会调用`kobject->ktype->ops->show()`方法.参数含义同写操作中是一样的.

## 在sysfs中创建二进制属性文件

二制制属性通常用于firmware中，它用来更新firmware的固件。
它的接口为`sysfs_create_bin_file()`，代码如下:
```c
int sysfs_create_bin_file(struct kobject * kobj, struct bin_attribute * attr)
{
       BUG_ON(!kobj || !kobj->sd || !attr);

       return sysfs_add_file(kobj->sd, &attr->attr, SYSFS_KOBJ_BIN_ATTR);
}
```
`sysfs_add_file()`这个函数我们在之前已经分析过,在这个地方,可能会引起迷糊,因为在`sysfs_add_file()`中有:
```c
int sysfs_add_file(struct sysfs_dirent *dir_sd, const struct attribute *attr,
                 int type)
{
       ……
       sd->s_attr.attr = (void *)attr;
       ……
}
```
这里为什么是`sd->s_attr`呢? 应该是`sd-> s_bin_attr`才对吧!
仔细观察`struct sysfs_dirent`的结构,如下:
```c
struct sysfs_dirent {
       atomic_t         s_count;
       atomic_t         s_active;
       struct sysfs_dirent  *s_parent;
       struct sysfs_dirent  *s_sibling;
       const char           *s_name;

       union {
              struct sysfs_elem_dir        s_dir;
              struct sysfs_elem_symlink    s_symlink;
              struct sysfs_elem_attr       s_attr;
              struct sysfs_elem_bin_attr   s_bin_attr;
       };

       unsigned int           s_flags;
       ino_t                  s_ino;
       umode_t                s_mode;
       struct iattr           *s_iattr;
};
```
注意中间是一个`union`结构，实际上只占用一个内存空间，而且`s_attr`与`s_bin_attr`的第一个属性都为`struct attribute`，所以在这里,`sd->s_attr`与`sd-> s_bin_attr`;的效果是一样的。内核这样处理，又少用了一个接口。看来作者在设计的时候,花了很多的心思.

二进制的文件读写与普通属性的文件读写方式大部份都一样，所不同的是，二进制文件的读写接口分别是:`sysfs_dirent->s_bin_attr.bin_attr->read`和`sysfs_dirent->s_bin_attr.bin_attr->write`。

## sysfs文件系统中的链接文件

创建链接文件的接口为:`sysfs_create_link()`.代码如下:
```c
int sysfs_create_link(struct kobject * kobj, struct kobject * target, const char * name)
{
       struct sysfs_dirent *parent_sd = NULL;
       struct sysfs_dirent *target_sd = NULL;
       struct sysfs_dirent *sd = NULL;
       struct sysfs_addrm_cxt acxt;
       int error;

       BUG_ON(!name);

       if (!kobj)
              parent_sd = &sysfs_root;
       else
              parent_sd = kobj->sd;

       error = -EFAULT;
       if (!parent_sd)
              goto out_put;

       /* target->sd can go away beneath us but is protected with
        * sysfs_assoc_lock.  Fetch target_sd from it.
        */
       spin_lock(&sysfs_assoc_lock);
       if (target->sd)
              target_sd = sysfs_get(target->sd);
       spin_unlock(&sysfs_assoc_lock);

       error = -ENOENT;
       if (!target_sd)
              goto out_put;

       error = -ENOMEM;
       sd = sysfs_new_dirent(name, S_IFLNK|S_IRWXUGO, SYSFS_KOBJ_LINK);
       if (!sd)
              goto out_put;

       sd->s_symlink.target_sd = target_sd;
       target_sd = NULL; /* reference is now owned by the symlink */

       sysfs_addrm_start(&acxt, parent_sd);
       error = sysfs_add_one(&acxt, sd);
       sysfs_addrm_finish(&acxt);

       if (error)
              goto out_put;

       return 0;

 out_put:
       sysfs_put(target_sd);
       sysfs_put(sd);
       return error;
}
```
上面的操作大部份都与普通文件的创建相似,所不同的只是下面这段代码的区别:
```c
sd->s_symlink.target_sd = target_sd;
```
就是在`sd->s_symlink.target_sd`保存到链接目的地的`sysfs_dirent`.
符号链接的操作如下所示:
```c
const struct inode_operations sysfs_symlink_inode_operations = {
       .readlink = generic_readlink,
       .follow_link = sysfs_follow_link,
       .put_link = sysfs_put_link,
};
```
在通过符号链接查找文件的时候,在VFS中会调用`inod->i_op->readlink()`进行操作.它的代码如下:
```c
int generic_readlink(struct dentry *dentry, char __user *buffer, int buflen)
{
       struct nameidata nd;
       void *cookie;

       nd.depth = 0;
       cookie = dentry->d_inode->i_op->follow_link(dentry, &nd);
       if (!IS_ERR(cookie)) {
              int res = vfs_readlink(dentry, buffer, buflen, nd_get_link(&nd));
              if (dentry->d_inode->i_op->put_link)
                     dentry->d_inode->i_op->put_link(dentry, &nd, cookie);
              cookie = ERR_PTR(res);
       }
       return PTR_ERR(cookie);
}
```
它的操作和其它文件系统一样,都是通用`follow_link()`取得目的地的路径,然后保存到`nd->saved_names[]`中,然后,调用`vfs_readlink()`将目标路径copy到`buffer`中.接着,调用`put_link`进行事后处理工作.

`follow_link()`的操作如下示:
```c
static void *sysfs_follow_link(struct dentry *dentry, struct nameidata *nd)
{
       int error = -ENOMEM;
       unsigned long page = get_zeroed_page(GFP_KERNEL);
       if (page)
              error = sysfs_getlink(dentry, (char *) page);
       nd_set_link(nd, error ? ERR_PTR(error) : (char *)page);
       return NULL;
}
```
`nd_set_link()`是将`page`中的值copy到`nd->saved_name[]`中.
`sysfs_getlink()`的代码如下:
```c
static int sysfs_getlink(struct dentry *dentry, char * path)
{
	struct sysfs_dirent *sd = dentry->d_fsdata;
	struct sysfs_dirent *parent_sd = sd->s_parent;
	struct sysfs_dirent *target_sd = sd->s_symlink.target_sd;
	int error;

	mutex_lock(&sysfs_mutex);
	error = sysfs_get_target_path(parent_sd, target_sd, path);
	mutex_unlock(&sysfs_mutex);

	return error;
}
```
`sysfs_get_target_path()`代码如下：
```c
static int sysfs_get_target_path(struct sysfs_dirent *parent_sd,
                             struct sysfs_dirent *target_sd, char *path)
{
       struct sysfs_dirent *base, *sd;
       char *s = path;
       int len = 0;

       /* go up to the root, stop at the base */
       base = parent_sd;
       while (base->s_parent) {
              sd = target_sd->s_parent;
              while (sd->s_parent && base != sd)
                     sd = sd->s_parent;

              if (base == sd)
                     break;

              strcpy(s, "../");
              s += 3;
              base = base->s_parent;
       }

       /* determine end of target string for reverse fillup */
       sd = target_sd;
       while (sd->s_parent && sd != base) {
              len += strlen(sd->s_name) + 1;
              sd = sd->s_parent;
       }
       /* check limits */
       if (len < 2)
              return -EINVAL;
       len--;
       if ((s - path) + len > PATH_MAX)
              return -ENAMETOOLONG;
       /* reverse fillup of target string from target to base */
       sd = target_sd;
       while (sd->s_parent && sd != base) {
              int slen = strlen(sd->s_name);

              len -= slen;
              strncpy(s + len, sd->s_name, slen);
              if (len)
                     s[--len] = '/';

              sd = sd->s_parent;
       }
       return 0;
}
```
这段代码的逻辑比较简单.它先是找到目标路径和当前路径相同的父结点,然后再沿着目标结点往相同的父结点向上走,将路径依次从缓存区后面往前面保存.
例如: `/sys/eric/kernel/test`链接到了`/sys/sys/device`.
它先找到两个路径共有的父结点`/sys
`此时缓存区为:`/sys
`然后,沿着`/sys/sys/device`往`/sys`移动,路径加从缓存区的后面往前面加.依次为:
1: /sys/     /device
2:/sys/sys/device
这样就找到了目的地的路径. ^_^.
后面`sysfs_put_link()`的操作就不再讲述了,它只是释放掉缓存区.

## 小结

在本小节里,我们深入探讨了sysfs文件系统的实现机理.这对于我们理解linux设备模型是很有帮助的.
