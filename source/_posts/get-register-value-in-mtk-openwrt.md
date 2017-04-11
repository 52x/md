---
title: mtk openwrt读取寄存器值
date: 2016-07-06 16:50:33
tags:
 - register
 - openwrt
categories:
 - openwrt
---

datasheet: MT7628_ProgrammingGuide_20140428(E2).pdf

### 确定寄存器物理BASE地址 ###

根据`datasheet`的`1.3 Memory Map Summary`找到`SYSCTL`的`Start`地址是： `10000000`，如下

```
 1.3 Memory Map Summary

| Start     |   | End       | Size       | Description |
|           |   |           |            |             |
| 0000.0000 | - | 0FFF.FFFF | 256 MBytes | DDR 256 MB  |
| 1000.0000 | - | 1000.00FF | 256 Bytes  | SYSCTL      |
| 1000.0100 | - | 1000.01FF | 256 Bytes  | TIMER       |
```

### 确定寄存器物理地址 ###

比如要查看`port 0`的`Link Status`。
根据`datasheet`，找到地址`10110080`，如下：

```
 10110080   POA       Port Ability Offset
| Bit  |   31    |   30    | 29 | 28 | 27 | 26 | 25 | 24 | 23 |
| Name | G1_LINK | G0_LINK |          LINK          | G1_TXC  |
| Type |   RO    |   RO    |           RO           |    RO   |

----------------------------------------------------------------------------------------------------
Bit(s)      Name        Description
----------------------------------------------------------------------------------------------------
  31        G1_LINK     Port 6 Link Status
                        1: Link up
                        0: Link down
  30        G0_LINK     Port 5 Link Status
                        1: Link up
                        0: Link down
 29:25      LINK        Port 4 ~ Port 0 Link Status
                        1: Link up
                        0: Link down
 24:23      G1_TXC      Flow Control Status of Port 6
                        The flow control capability status bit after Auto-negotiation or force mode.
                        1xb: full duplex and tx flow control ON
                        x1b: full duplex and rx flow control ON
                        00b: flow control off
```

可以看到，`Port 0`的`Link Status`在`25`位。

### 确定寄存器虚拟BASE地址 ###

```
# printk打印，dmesg可看
reg s 2
```

### 修改寄存器虚拟BASE地址 ###

```
# 根据linux源码/arch/mips/include/asm/mach-ralink/rt_mmap.h的 RALINK_SYSCTL_BASE 而来。
reg s 0xb0000000

# 或者直接将base切换为system register
reg s 0
```

### 获取寄存器的值 ###

物理地址(0x10110080) - 物理BASE地址(0x10000000) = 偏移量(0x110080)
虚拟BASE地址(0xb0000000) + 偏移量(0x110080) = 虚拟地址(0xb0110080)
因为`reg`命令只需要一个偏移量就够了，所以没用到虚拟地址。

```
# printk打印，dmesg可看
reg r 110080

# 获取返回值可以用下面命令，查看reg.c源码有`p`参数
reg p 110080
```

得到的值再进行`与`操作，就可以拿到对应端口的`Link Status`了。
