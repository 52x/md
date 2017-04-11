---
title: luci中英文翻译
date: 2016-05-28 15:43:42
tags:
 - LuCI
categories:
 - LuCI
---

## 添加中英文模块 ##

`make menuconfig`中选上以下两个模块：

- luci-i18n-chinese
- luci-i18n-english

## 翻译文件格式 ##

```
msgid "Action"
msgstr "动作"
```

`msgid`：是指`ID`，不是英文译文，并不是指英文译文就一定是跟此`msgid`一致了。

`msgstr`：是指译文。

> 注意：中英文的话，`luci/po/zh_CN`和`luci/po/en`底下对应的`po`文件都要改。
> 例如只改了中文，没添加英文，则使用此`msgid`时，选择英文语言会找不到英文译文，则直接显示此`msgid`。 

## 添加/修改翻译文件 ##

- 方式一

`base.po`是选上了中英文模块之后，就会自动编译上去的，可以直接改。

> 注意：`luci/po/zh_CN/base.po`和`luci/po/en/base.po`都得写上对应翻译。

- 方式二

在`luci/applications`目录，按照该目录的其它文件夹，再添加一个文件夹：如`example`文件夹。

则需要在`make menuconfig`的时候选上`luci-app-example`模块。

再添加`luci/po/zh_CN/example.po`和`luci/po/en/example.po`，写上对应翻译。

- 方式三

`po`文件最终是要编译成`lmo`文件的，所以可以按照格式手动添加`po`文件，如：`luci/po/zh_CN/test.po`和`luci/po/en/test.po`。

然后利用`po2lmo`工具直接编译成`lmo`，`po2lmo`工具是编译过固件才有的，即`make`之后才有的，具体位置`find`一下吧。

```
po2lmo test.po test.en.lmo    # 必须添加en标识，否则不识别
po2lmo test.po test.zh-cn.lmo # 必须添加zh-cn标识，否则不识别
```

然后把生成的`lmo`文件扔到路由器的`/usr/lib/lua/luci/i18n/`目录下

然后需要在路由器的`/usr/lib/lua/luci/controller`里面的文件指定使用此`lmo`翻译

```
entry({"admin", "system", "system"}, cbi("admin_system/system"), _("System"), 1).i18n = "test"
```
## 使用方法 ##

在`htm`文件里
```
<%+header%>                                                                    
<h1><%:Action%></h1>                                                      
<%+footer%>
```

## 语言切换 ##

`cat /etc/config/luci`

```
config core 'main'
    option resourcebase '/luci-static/resources'
    option mediaurlbase '/luci-static/openwrt.org'
    option lang 'en' # 或者 zh_cn，取值是根据下面的option来的

config internal 'languages'
    option zh_cn 'chinese'
    option en 'English'
```


