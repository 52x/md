title: ASOC codec platform
---

### ASoC Platform Driver
ASoC platform driver class被分成auido DMA driver, SOC DAI drivers和DSP drivers。

### Audio DMA
The platform DMA driver支持下面的ALSA操作

    struct snd_soc_ops {
		int (*startup)(struct snd_pcm_substream *);
		void (*shutdown)(struct snd_pcm_substream *);
		int (*hw_params)(struct snd_pcm_substream *, struct snd_pcm_hw_params *);
		int (*hw_free)(struct snd_pcm_substream *);
		int (*prepare)(struct snd_pcm_substream *);
		int (*trigger)(struct snd_pcm_substream *, int);
	};

The platform驱动通过struct
snd_soc_platform_driver来输出它的DMA能力
    struct snd_soc_platform_driver {
		char *name;
	
		int (*probe)(struct platform_device *pdev);
		int (*remove)(struct platform_device *pdev);
		int (*suspend)(struct platform_device *pdev, struct snd_soc_cpu_dai *cpu_dai);
		int (*resume)(struct platform_device *pdev, struct snd_soc_cpu_dai *cpu_dai);
	
		/* pcm creation and destruction */
		int (*pcm_new)(struct snd_card *, struct snd_soc_codec_dai *, struct snd_pcm *);
		void (*pcm_free)(struct snd_pcm *);
	
		/*
		 * For platform caused delay reporting.
		 * Optional.
		 */
		snd_pcm_sframes_t (*delay)(struct snd_pcm_substream *,
			struct snd_soc_dai *);
	
		/* platform stream ops */
		struct snd_pcm_ops *pcm_ops;
	};