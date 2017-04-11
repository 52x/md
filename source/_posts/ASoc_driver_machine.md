title: ASoc Machine Driver
---

### ASoc machine driver
ASoC machine driver 将所有的componentdriver联系起来。

    struct snd_soc_card {
		char *name;
	
		...
	
		int (*probe)(struct platform_device *pdev);
		int (*remove)(struct platform_device *pdev);
	
		/* the pre and post PM functions are used to do any PM work before and
		 * after the codec and DAIs do any PM work. */
		int (*suspend_pre)(struct platform_device *pdev, pm_message_t state);
		int (*suspend_post)(struct platform_device *pdev, pm_message_t state);
		int (*resume_pre)(struct platform_device *pdev);
		int (*resume_post)(struct platform_device *pdev);
	
		...
	
		/* CPU <--> Codec DAI links  */
		struct snd_soc_dai_link *dai_link;
		int num_links;
	
		...
    };

### Machine DAI Configuration
Machine DAI配置将CPU DAI和codec结合在一起。 struct snd_soc_dai_link用来设置machine中的每个DAI。

### Machine Controls
Machine specific audio mixer controls可以在DAI的init函数总体按键。