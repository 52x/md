title: ALSA lib 概念
---

- Frame  
一帧就是要播放的一个采样数据，与通道个数无关。  
    - 对于一个环绕48khz 16bit PCM流来说一帧就是4个字节
    - 对于一个5.1声道48khz 16bit PCM流来说一帧就是12个字节

- period  
两次中断间的帧数，poll函数每个period会返回一次