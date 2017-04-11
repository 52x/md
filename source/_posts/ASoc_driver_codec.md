title: ASOC codec driver
---

Codec驱动配置codec来提供音频的捕捉与播放，是通用的与硬件无关。

Codec驱动必须提供下面的功能：

- Codec DAI and PCM configuration
- Codec control IO
- Mixer and audio controls
- Codec audio operations

选择性提供：

- DAPM description
- DAPM event handler
- DAC Digital mute control

### ASoC Codec驱动 

1. Codec DAI and PCM configuration   
每个Codec驱动都必须包含一个struct snd_soc_dai_driver结构体来定义它的DAI和PCM能力及操作。

2. Codec control IO   
Codec驱动提供读写codec寄存器的函数，还提供了一个register cache

        void *control_data;
        void *reg_cache;
Codec读写函数应该将数据整理好并调用硬件读写函数

        unsigned int (*read)(struct snd_soc_codec *, unsigned int);
        int (*write)(struct snd_soc_codec *, unsigned int, unsigned int);
Codec硬件IO函数
        hw_write_t hw_write;
        hw_read_t hw_read;

3. Mixers and audio controls   
所有的codec mixer和audio control都可以使用soc.h里的宏定义

        #define SOC_SINGLE(xname, reg, shift, mask, invert)

4. Codec Audio Operations   
Codec提供以下ALSA操作

	    struct snd_soc_ops {
			int (*startup)(struct snd_pcm_substream *);
			void (*shutdown)(struct snd_pcm_substream *);
			int (*hw_params)(struct snd_pcm_substream *, struct snd_pcm_hw_params *);
			int (*hw_free)(struct snd_pcm_substream *);
			int (*prepare)(struct snd_pcm_substream *);
	    };

5. DAPM描述   
DAPM(Dynamic Audio Power Management)想ASoC core描述codec的电源组件及之间的关系，以及寄存器。
 
6. DAPM event handler   
这个函数用来作为codec domain PM call和system domain PM call(e.g. suspend and resume)的回调函数。

7. Codec DAC digital mute control   
大部分codec在DACS前包含一个digital mute，用来阻止数字数据进入DAC。

### 参考

[kernel 3.10 document] 