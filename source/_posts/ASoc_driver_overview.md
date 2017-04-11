title: ASoc_driver_overview
---

### ASoC Design
ASoC将嵌入式音频系统划分为多个组件的驱动：

- Codec class drivers: Codec class driver与平台无关，包含audio controls，audio interface capabilities，codec DAPM定义和Codec IO函数
- Platform class drivers: The platform class driver包含audio DMA gngine驱动，digital audio interface（DAI）驱动（I2S， AC97， PCM）以及audio DSP
- Machine class driver: machine driver class 将其他部分组件结合在一起生产一个声卡设备，处理与machine相关的controls和machine level audio events

### 参考

[kernel 3.10 document] 