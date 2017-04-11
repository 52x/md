title: 微信推送从未如此简单--Server酱
date: 2016-12-30 2:51 AM
categories: ""
tags: [python,]
---
有一个非常简单的需求，每天查看一下以太坊的价格动向。

<!--more-->

![http://harchiko.qiniudn.com/WechatIMG27.jpeg](http://harchiko.qiniudn.com/WechatIMG27.jpeg)

## 方法

以太坊价格通过查询 YUNBI 的 API 知道:  https://yunbi.com/api/v2/tickers/ethcny.json 可以得到。

推送用到工具 Server酱， Server酱用起来实在是太方便了，一个 get 请求，信息就直接推送到微信上了。

直接向`http://sc.ftqq.com/[SCKEY(登入后可见)].send`发送GET请求即可:

```
接受两个参数：

text：消息标题，最长为256，必填。
desp：消息内容，最长64Kb，可空，支持MarkDown。
```
## 代码

使用Python来写:

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import requests

r = requests.get('https://yunbi.com:443/api/v2/tickers/ethcny.json')

json = r.json()

ticker = json['ticker']
sell = ticker['sell']
buy = ticker['buy']
last = ticker['last']
vol = ticker['vol']
high = ticker['high']
low = ticker['low']

price =  u"* 最高: "+high+u"\n* 最低: "+low+u"\n* 卖价: "+sell+u"\n* 买价: "+buy+u"\n* 上次成交: "+last+u"\n* 成交量: "+vol

payload = {'text':u'以太坊最新价格', 'desp': price}
requests.get('http://sc.ftqq.com/[Your Own Token].send', params=payload)
```
需要安装下 python requests包

`pip install requests`

设置下 crontab 任务，这样每天就可以推送价格了。

![http://harchiko.qiniudn.com/WechatIMG26.jpeg](http://harchiko.qiniudn.com/WechatIMG26.jpeg)