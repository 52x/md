---
title: 碰到一个debian8使用apt-get安装的phpmyadmin的问题
tags: [debian,phpmyadmin,linux,nginx]
date: 2017-03-14 13:09:47
updated: 2017-03-14 13:09:47
categories: [Linux]
keywords: [debian,phpmyadmin,linux,nginx,404]
description:
---

今天有个问题卡了我1个来小时，最后才发现。。。

最近沾google 的光，试用GCE，选的最小配置，但是600M的内存，装个ss后，再装个nginx+php+mysql也足够了。

装了mysql后，顺手把PhpMyAdmin也装上了，因为以前一直用centos，这回用debian也是摸着石头过河。

使用命令`sudo apt-get install phpmyadmin`进行了安装，因为选择项apache2 或者lighttpd都没有，用的是nginx，所以在界面配置中，直接cancel退出了。

安装好了PhpMyAdmin，使用`ln -s /usr/share/phpmyadmin/ /var/www/phpmyadmin` 软连接过去。

配置了config.inc.php。 然后登陆。输入正确的用户名密码，登录后跳转时候居然是**404**！检查了地址，发现少了一个phpmyadmin这个目录名。开始检查

* 检查nginx的目录配置，参照网上多种说法，各配置，无果
* 检查各种PhpMyAdmin的配置，各种配置，无果
* 添加了PhpMyAdmin配置中的$cfg['PmaAbsoluteUri']属性，无果
* 改目录名，无果（我有多蠢。）
* 检查了PhpMyAdmin的index文件，感觉也没问题
* 检查php配置，没找到问题。


终于忍不了了，直接上官网，手动下载zip文件，解压，将原来目录重命名，将新解压的目录连接到/var/  www中，重新试验，居然好了.....好了。。。。。。。。

无语，实在无语了。

只能感觉是debian 的源更新的版本的问题了。

记录下来，供大家查看

