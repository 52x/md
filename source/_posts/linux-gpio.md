title: Linux GPIO 驱动
---

### GPIO是什么
全称General Purpose Input/Output，是一种软件可控的数字信号，每个GPIO代表芯片的一个引脚或者BGA封装的一个ball。  
GPIO包含包含以下一些选项：

- 可是设置输出值(0或者1)
- 可以读取gpio值(0或者1)
- 有些可以用作IRQ中断信号
- 可以设置为输入输出

### GPIO表示
内核中用一个unsigned integers来表示一个GPIO，在0..MAX_INT中取值，负值用来表示GPIO不存在或者有错。

    int gpio_is_valid(int number); /* 判断一个gpio number是否合法 */

### GPIO使用
系统需要做的第一件事就是分配gpio

    gpio_request()
接着注册设备时需要标注GPIO的方向

    int gpio_direction_input(unsigned gpio);
	int gpio_direction_output(unsigned gpio, int value);

### Spinlock-Safe GPIO接口
大部分GPIO控制器能直接通过地址读写访问，这些GPIO访问不需要睡眠，可以在硬中断中调用。

    int gpio_get_value(unsigned gpio);
    void gpio_set_value(unsigned gpio, int value);

### GPIO访问可能需要睡眠的情况
对于一些需要通过IIC或者SPI总线访问的GPIO控制器，读写这些GPIO需要等待。

    int gpio_cansleep(unsigned gpio);
    int gpio_get_value_cansleep(unsigned gpio);
    void gpio_set_value_cansleep(unsigned gpio, int value);

### 请求与释放GPIO
使用下面的接口来请求与释放GPIO

    int gpio_request(unsigned gpio, const char *label);
    void gpio_free(unsigned gpio);
使用这两个函数有两个目的，标记引脚当作GPIO来使用，防止多个驱动都申请同一个gpio。

对于那些pinctrl子系统管理的gpio，gpiolib驱动的.request操作可能调用pinctrl_request_gpio(), .free会调用pinctrl_free_gpio()。

大多数情况下请求到gpio后都会立刻进行配置，因此定义了额外的三个函数

    int gpio_request_one(unsigned gpio, unsigned long flags, const char *label);
    int gpio_request_array(struct gpio *array, size_t num);
    void gpio_free_array(struct gpio *array, size_t num);

flags定义了以下属性

- GPIOF_DIR_IN
- GPIOF_DIR_OUT
- GPIOF_INIT_LOW
- GPIOF_INIT_HIGH
- GPIOF_OPEN_DRAIN
- GPIOF_OPEN_SOURCE
- GPIOF_EXPORT_DIR_FIXED
- GPIOF_EXPORT_DIR_CHANGEABLE

合法的组合

- GPIOF_IN
- GPIOF_OUT_INIT_LOW
- GPIOF_OUT_INIT_HIGH

为了方便使用，内核定义了下面的gpio结构体

    struct gpio {
		unsigned	gpio;
		unsigned long	flags;
		const char	*label;
	};

### sysfs用户空间接口
使用gpiolib可能会提供sysfs的用户接口。

#### sysfs路径
在/sys/class/gpio下面有三种类型的路径
- 控制接口
- gpio自身
- gpio控制器

##### 控制接口
/sys/class/gpio/

- export，用户可以通过写入gpio数字来为用户空间导入gpio控制
- unexport, 与上面相反

/sys/class/gpio/gpioN/

- direction, 读取返回"in"或者"out"，写入"out"默认输出低电平，写入"low"或者"high"改变输出值
- value, 读取返回"0"或者"1"， 如果gpio被配置成可写，写入非零值输出高
- edge, 读取返回"none", "rising", "failing", "both"
- active_low

### 导出gpio
    int gpio_export(unsigned gpio, bool direction_may_change);
    void gpio_unexport();
    int gpio_export_link(struct device *dev, const char *name,
		unsigned gpio)
    int gpio_sysfs_set_active_low(unsigned gpio, int value);

### 参考
内核文档gpio.txt
