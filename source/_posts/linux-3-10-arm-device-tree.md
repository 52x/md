---
title: Linux 3.10 ARM Device Tree 的初始化
tags:
  - DevTree
categories:
  - Kernel
  - 内核设备树
date: 2015-05-10 11:46:09
---

原文：<http://blog.chinaunix.net/uid-20522771-id-3785808.html>

本文代码均来自标准 linux kernel 3.10，可以到这里下载 <https://www.kernel.org/>
以 arch/arm/mach-msm/board-dt-8960.c 为例，在该文件中的`msm_dt_init`函数的作用就是利用 dt（device tree）结构初始化 platform device。
```c
static void __init msm_dt_init(void)
{
    of_platform_populate(NULL, of_default_bus_match_table, NULL, NULL);
}
```
<!--more-->

`of_platform_populate` 实现在 drivers/of/platform.c，是 OF 的标准函数。
```c
int of_platform_populate(struct device_node *root,
                         const struct of_device_id *matches,
                         const struct of_dev_auxdata *lookup,
                         struct device *parent)
{
    struct device_node *child;
    int rc = 0;

    root = root ? of_node_get(root) : of_find_node_by_path("/");
    if (!root)
        return -EINVAL;

    for_each_child_of_node(root, child) {
        rc = of_platform_bus_create(child, matches, lookup, parent, true);
        if (rc)
            break;
    }

    of_node_put(root);
    return rc;
}
```
其中`of_platform_populate`函数的注释写得很明白：“Populate platform_devices from device tree data”。但是这个“device tree data”又是从那里来的呢？
在`of_platform_populate`中如果 root 为 NULL，则将 root 赋值为根节点，这个根节点是用`of_find_node_by_path`取到的。
```c
struct device_node *of_find_node_by_path(const char *path)
{
    struct device_node *np = of_allnodes;
    unsigned long flags;

    raw_spin_lock_irqsave(&devtree_lock, flags);
    for (; np; np = np->allnext) {
        if (np->full_name && (of_node_cmp(np->full_name, path) == 0)
            && of_node_get(np))
            break;
    }
    raw_spin_unlock_irqrestore(&devtree_lock, flags);
    return np;
}
```
在这个函数中有一个很关键的全局变量：`of_allnodes`，它的定义是在 drivers/of/base.c 里面。
```c
struct device_node *of_allnodes;
```
这应该所就是那个所谓的“device tree data”了。它应该指向了 device tree 的根节点。问题又来了，这个`of_allnodes`又是咋来的呢？我们知道 device tree 是由 DTC（Device Tree Compiler）编译成二进制文件 DTB（Ddevice Tree Blob）的，然后在系统上电之后由 bootloader 加载到内存中去，这个时候还没有 device tree，而在内存中只有一个所谓的 DTB，这只是一个以某个内存地址开始的一堆原始的 dt 数据，没有树结构。kernel 的任务需要把这些数据转换成一个树结构然后再把这棵树的根节点的地址赋值给 of_allnodes 就行了。这个过程一定是非常重要，因为没有这个 device tree 那所有的设备就没办法初始化，所以这个 dt 树的形成一定在 kernel 刚刚启动的时候就完成了。
既然如此，我们来看看 kernel 初始化的代码（init/main.c），大部分代码与本主题无关用 ... 代替。
```c
asmlinkage void __init start_kernel(void)
{
    ...
    setup_arch(&command_line);
    ...
}
```
这个` setup_arch` 就是各个架构自己的设置函数（arch 是 architecture 的缩写），哪个参与了编译就调用哪个，在本文中应当是 arch/arm/kernel/setup.c 中的 setup_arch。
同样，无关的代码以 ... 代替。
```c
void __init setup_arch(char **cmdline_p)
{
    ...
    mdesc = setup_machine_fdt(__atags_pointer);
    ...
    unflatten_device_tree();
    ...
}
```
看到了吧，`setup_machine_fdt`，其中 fdt 的 f 就是扁平（flat）的意思。这个时候 DTB 只是加载到内存中的 .dtb 文件而已，这个文件中不仅包含数据结构，还包含了一些文件头等信息，kernel 需要从这些信息中获取到数据结构相关的信息，然后再生成设备树。这个函数的调用还有个参数 `__atags_pointer`，看名字似乎这是一个指针，干嘛的呢？以后再说，先进入函数看看。
```c
/* kernel/arch/arm/kernel/devtree.c */
struct machine_desc * __init setup_machine_fdt(unsigned int dt_phys)
{
    ...
    devtree = phys_to_virt(dt_phys);
    ...
    initial_boot_params = devtree;
    ...
}
```
现在有点意思了，`phys_to_virt`字面上的意思是物理地址转换成虚拟地址，那就是说`__atags_pointer`是一个物理地址，这就印证了我们的猜测，`__atags_pointer`的确是一个指针，再看变量 devtree 它指向了一个`struct boot_param_header`结构体。随后 kernel 把这个指针赋给了全局变量`initial_boot_params`。也就是说以后 kernel 会是用这个指针指向的数据去初始化 device tree。
```c
struct boot_param_header {
    __be32 magic; /* magic word OF_DT_HEADER */
    __be32 totalsize; /* total size of DT block */
    __be32 off_dt_struct; /* offset to structure */
    __be32 off_dt_strings; /* offset to strings */
    __be32 off_mem_rsvmap; /* offset to memory reserve map */
    __be32 version; /* format version */
    __be32 last_comp_version; /* last compatible version */
    /* version 2 fields below */
    __be32 boot_cpuid_phys; /* Physical CPU id we're booting on */
    /* version 3 fields below */
    __be32 dt_strings_size; /* size of the DT strings block */
    /* version 17 fields below */
    __be32 dt_struct_size; /* size of the DT structure block */
};
```
    看这个结构体，很像之前所说的文件头，有魔数、大小、数据结构偏移量、版本等等，kernel 就应该通过这个结构获取数据，并最终生成设备树。现在回到 `setup_arch`，果然在随后的代码中有这么一个函数。
```c
/* kernel/drivers/of/fdt.c */
void __init unflatten_device_tree(void)
{
    __unflatten_device_tree(initial_boot_params, &of_allnodes,
                early_init_dt_alloc_memory_arch);
    ...
}
```
    看见了吧，`of_allnodes`就是在这里赋值的，device tree 也是在这里建立完成的。`__unflatten_device_tree` 函数我们就不去深究了，推测其功能应该就是解析数据、申请内存、填充结构等等。
    到此为止，device tree 的初始化就算完成了，在以后的启动过程中，kernel 就会依据这个 dt 来初始化各个设备。但是还有一个小问题，那就是在 `setup_arch` 函数中 `__atags_pointer` 从何而来。全局搜索这个变量，结构在 arch/arm/kernel/head-common.S 中发现了它的踪迹。
```asm
#define ATAG_CORE 0x54410001
...
__mmap_switched:
    adr r3, __mmap_switched_data

    ldmia {r4, r5, r6, r7}
    ...
    THUMB( ldr sp, [r3, #16] )
    ...
    str r2, [r6] @ Save atags pointer
    ...
__mmap_switched_data:
    ...
    .long __atags_pointer @ r6
    ...
```
arm 汇编不懂，大概能看出来给 `__atags_pointer` 赋值的过程（‘@’之后是注释）。
