---
title: htaccess文件使用大全【转】
tags: [htaccess,vps,linux]
date: 2017-02-25 23:58:51
updated: 2017-02-25 23:58:51
categories: [Vps]
keywords: [htaccess]
description:
---

.htaccess文件(或者"分布式配置文件"提供了针对目录改变配置的方法， 即，在一个特定的文档目录中放置一个包含一个或多个指令的文件，以作用于此目录及其所有子目录。[1]作为用户，所能使用的命令受到限制。管理员可以通过Apache的AllowOverride指令来设置。子目录中的指令会覆盖更高级目录或者主服务器配置文件中的指令。

.htaccess必须以ASCII模式上传，最好将其权限设置为644。

错误文档的定位

常用的客户端请求错误返回代码：
401 Authorization Required
403 Forbidden
404 Not Found
405 Method Not Allowed
408 Request Timed Out
411 Content Length Required
412 Precondition Failed
413 Request Entity Too Long
414 Request URI Too Long
415 Unsupported Media Type
常见的服务器错误返回代码：
500 Internal Server Error

用户可以利用.htaccess指定自己事先制作好的错误提醒页面。一般情况下，人们可以专门设立一个目录，例如errors放置这些页面。然后再.htaccess中，加入如下的指令：

ErrorDocument 404 /errors/notfound.html
ErrorDocument 500 /errors/internalerror.html

一条指令一行。上述第一条指令的意思是对于404，也就是没有找到所需要的文档的时候得显示页面为/errors目录下的notfound.html页面。不难看出语法格式为：

ErrorDocument 错误代码 /目录名/文件名.扩展名

如果所需要提示的信息很少的话，不必专门制作页面，直接在指令中使用HTML号了，例如下面这个例子：

ErrorDocument 401 "
你没有权限访问该页面，请放弃！
"

文档访问的密码保护

要利用.htaccess对某个目录下的文档设定访问用户和对应的密码，首先要做的是生成一个.htpasswd的文本文档，例如：

zheng:y4E7Ep8e7EYV

这里密码经过加密，用户可以自己找些工具将密码加密成.htaccess支持的编码。该文档最好不要放在www目录下，建议放在www根目录文档之外，这样更为安全些。

有了授权用户文档，可以在.htaccess中加入如下指令了：

AuthUserFile .htpasswd的服务器目录
AuthGroupFile /dev/null （需要授权访问的目录）
AuthName EnterPassword
AuthType Basic （授权类型）

require user wsabstract （允许访问的用户，如果希望表中所有用户都允许，可以使用 require valid-user）

注，括号部分为学习时候自己添加的注释

拒绝来自某个IP的访问

如果我不想某个政府部门访问到我的站点的内容，那可以通过.htaccess中加入该部门的IP而将它们拒绝在外。

例如：

order allow,deny
deny from 210.10.56.32
deny from 219.5.45.
allow from all

第二行拒绝某个IP，第三行拒绝某个IP段，也就是219.5.45.0~219.2.45.255

想要拒绝所有人？用deny from all好了。不止用IP，也可以用域名来设定。

保护.htaccess文档

在使用.htaccess来设置目录的密码保护时，它包含了密码文件的路径。从安全考虑，有必要把.htaccess也保护起来，不让别人看到其中的内容。虽然可以用其他方式做到这点，比如文档的权限。不过，.htaccess本身也能做到，只需加入如下的指令：

order allow,deny
deny from all

URL转向

我们可能对网站进行重新规划，将文档进行了迁移，或者更改了目录。这时候，来自搜索引擎或者其他网站链接过来的访问就可能出错。这种情况下，可以通过如下指令来完成旧的URL自动转向到新的地址：

Redirect /旧目录/旧文档名 新文档的地址

或者整个目录的转向：

Redirect 旧目录 新目录

改变缺省的首页文件

一般情况下缺省的首页文件名有default、index等。不过，有些时候目录中没有缺省文件，而是某个特定的文件名，比如在pmwiki中是pmwiki.php。这种情况下，要用户记住文件名来访问很麻烦。在.htaccess中可以轻易的设置新的缺省文件名：

DirectoryIndex 新的缺省文件名

也可以列出多个，顺序表明它们之间的优先级别，例如：

DirectoryIndex filename.html index.cgi index.pl default.htm

防止盗链

如果不喜欢别人在他们的网页上连接自己的图片、文档的话，也可以通过htaccess的指令来做到。

所需要的指令如下：

RewriteEngine on
RewriteCond %{ HTTP_REFERER } !^$
RewriteCond %{ HTTP_REFERER } !^http://(www.)?mydomain.com/.*$ [NC]
RewriteRule .(gif&line;jpg)$ - [F]

如果觉得让别人的页面开个天窗不好看，那可以用一张图片来代替：

RewriteEngine on
RewriteCond %{ HTTP_REFERER } !^$
RewriteCond %{ HTTP_REFERER } !^http://(www.)?mydomain.com/.*$ [NC]
RewriteRule .(gif&line;jpg)$ http://www.mydomain.com/替代图片文件名 [R,L]



---

按：

几年前的文章，不知道是转的还是自己写的了。可能是转的。