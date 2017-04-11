title: DAPM
---

Dynamic Audio Power Management(DAPM)是为了节省移动linux设备audio子系统电量而设计的。

在DAPM中划分了4个电源域：

1. Codec domain: VREF, VMID(core codec and audio power). 通常在codec的probe/remove，syspend/resume时进行控制。
2. Platform/Machine domain: 物理上连接的输入输出，与特定的platform/machine相关，通过machine读取来配置.
3. Path domain: audio子系统信号路径。
4. Stream domain: DAC和ADC, 当播放或捕捉流时打开或关闭。