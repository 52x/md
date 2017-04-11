title: ALSA audio api
---

### 理解音频接口
音频接口就是一种计算机可以读或者写音频数据的设备。在音频接口内部有一个硬件buffer，当buffer中收集到足够多的音频数据后，会产生中断通知电脑读取数据，或者在发送数据的时候通知电脑可以写数据到buffer中。

要执行发送或者接收音频数据，需要配置一些参数。

- 转换的格式
- 取样率
- 当数据到达多少时产生中断
- 硬件buffer大小

### 典型的音频应用

    open_the_device();
    set_the_parameters_of_the_device();
    while (!done) {
        /* one or both of these */
        receive_audio_data_from_the_device();
	    deliver_audio_data_to_the_device();
    }
    close the device
