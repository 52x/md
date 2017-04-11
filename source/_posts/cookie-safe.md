title: cookie的攻击与防御
date: 2015-11-18 12:42:21
tags: [安全,攻击防御,cookies]
---

## Cookie或Session的安全问题
用户登录的凭证，不管是Cookie还是Session，都会在浏览器端留下一段cookies，这就相当与给一道门留了一把钥匙，当黑客知晓怎么生成这把钥匙或者截获这把钥匙的时候，登录凭证这道门也就没有多少意义了。
## Cookie，Session的攻击方法
##### 当服务器端的判断是基于cookie是否存在的时候，黑客直接生成Cookie就可破解
这种服务器端验证方法显的格外幼稚，但是写程序的总归是人，总会犯一些低级的错误而不自知。比如以下：
* [http://www.wooyun.org/bugs/wooyun-2015-0148657](http://www.wooyun.org/bugs/wooyun-2015-0148657)
![](/mdimg/20151226230349.jpg)

* [http://wooyun.org/bugs/wooyun-2010-017767](http://wooyun.org/bugs/wooyun-2010-017767)
![](/mdimg/20151226231434.jpg)

##### xss攻击
当网络应用被xss成功攻击后，这把钥匙就会随着http请求发送到黑客自己的服务器上，从而可以获取到登录凭证，关于xss攻击，这里就不再叙述了

##### http嗅探
当用户的所在网络被黑客入侵，从而可以拦截到网络请求包的时候，因为http是带cookie的，所以这个时候cookie也可以被黑客获取到


##### cookie的逻辑错误
支付宝Cookie策略在15年9月爆出一个逻辑漏洞，可以根据一个cookie的加密字符来突破安全验证体系。

[http://developer.51cto.com/art/201510/493683.htm](http://developer.51cto.com/art/201510/493683.htm)

![](/mdimg/wKiom1YcW0uxh37EAAJyXD9NBWM730.jpg)


## cookie攻击的防御

* 服务器端合理安排cookie生成策略，比如附加一个key来验证cookie里的信息不是被伪造的，这个key要经过加盐和多重加密，不然可能会被撞击破解
* 附加的key要包含客户端信息：比如ip或者useragent，这样来防范用户通过别的手段盗取到用户cookie后直接来使用
* 启用`http only`，这个主要是针对xss攻击后通过js获取cookie
* 启用https

