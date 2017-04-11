title: 使用Sublime Text写Markdown
date: 2014-03-16 13:58:03
tags: 开发工具
---

默认Sublime Text是不支持Markdown语法高亮和预览的，对于万能的Sublime Text，这点事情一定有插件能解决。  

Sublime Text 3 安装`Package Control`，挺不能理解为什么不默认含在app里  
快捷键**ctrl+`**或者 View > Show Console 菜单打开控制台
```
import urllib.request,os;
pf='Package Control.sublime-package';
ipp=sublime.installed_packages_path();
urllib.request.install_opener(urllib.request.build_opener(urllib.request.ProxyHandler()));
open(os.path.join(ipp,pf),'wb').write(urllib.request.urlopen('http://sublime.wbond.net/'+pf.replace(' ','%20')).read())
```
安好重启后，用万能快捷键`Ctrl+Shift+P`，调出菜单`Package Control Install`，再输入`Markdown`关键字，就能发现很多Markdown插件。  
经过反复尝试，还是一个叫`Markdown Editing`的比较好使，直接使编辑器在编辑时所见即所得，只是这个默认灰色的颜色...
<!-- more -->
![](http://ww4.sinaimg.cn/large/51530583gw1eehp8t8zkfj20r60jzgo9.jpg)