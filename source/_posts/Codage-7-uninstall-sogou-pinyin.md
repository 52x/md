title: Ubuntu 14.04 卸载搜狗拼音输入法及后续问题解决
date: 2015-10-02 14:00:08
categories: Codage|编程
tags: [Ubuntu, sogoupinyin, Linux]
---

安装搜狗拼音折腾了很久，安装好了以后，法语和日语键盘死活不能调用出来，也不知道为什么。
想想现在版本下的默认拼音功能已经足够，于是就准备删除搜狗了。

<!--more-->

## 删除步骤
1. 打开terminal, 输入命令卸载搜狗拼音
	``` bash
	$ sudo apt-get purge sogoupinyin
	```
2. 在Ubuntu桌面右上角中点击`System Settings` 并选择`Language Support`，在第一个tab最下面的下拉框中(`Keyboard input method system`)选择`IBus`。
3. 卸载fcitx
	``` bash
	$ sudo apt-get purge fcitx
	```
4. 自动卸载fcitx相关配置
	``` bash
	$ sudo apt-get autoremove
	```
5. 注销后重新登录

## 后遗症
重新登录的时候就傻逼了，输入密码并没有提示错误，而是短暂黑屏以后迅速闪回登陆界面。尝试多少次都一直这样。
只好先切换到命令行：`Ctrl`+`Alt`+`F1`输入:
``` bash
grep -nr fcitx
```
结果显示并不是所有的包含fcitx想关信息的sogou文件都被清除了。
在`/etc/X11/Xsession.d`和`/etc/X11/xinit/xinput.d`中都有sogou相关文件未被清除。那么把它们删除了试试看吧。
``` bash
$ cd /etc/X11/Xsession.d
$ sudo rm -f 72sogoupinyin
$ cd /etc/X11/xinit/xinput.d
$ sudo rm -f 55-sogoupinyin.sh
```
按`Ctrl`+`Alt`+`F7`返回图形界面，重新登录，一切都回来了。

-------------------------

## Reference
1. [ubuntu 彻底卸载搜狗拼音输入法(显然并不是很彻底。。。)](http://jingyan.baidu.com/article/9faa723154c3dc473d28cb41.html)
2. [Ubuntu 卸载Fcitx输入法后无法登录桌面问题解决](http://www.gejoin.com/archives/1783)