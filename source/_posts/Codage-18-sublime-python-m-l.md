title: Ubuntu中基于Sublime Text2的Python简单环境搭建
date: 2016-01-07 23:52:24
categories: Codage|编程
tags: [Python, Sublime Text 2, Ubuntu]
---

本想就在WIN7上用python学习来着，不知何故环境变量设置经常失效，ipython也用不了。索性“弃暗投明”现在ubuntu上搭建一个环境起来，有时间再在那台机子上装ipython notebook。

<!-- more -->

以下均基于已安装好的Python2.7及Sublime Text2进行操作。

## 安装相关包
与机器学习相关的基础包有4个：NumPy, Scipy, Matplotlib及Scikit-Learn。
除Matplotlib(用于作图)之外，其它三个包均可以用`pip install`安装，Matplotlib有一些dependencies的需求pip install无法完成，需要借助于`apt-get`。
安装过程中要创建一些文件夹，所以提前进入super admin模式进行安装。

``` cmd
sudo -i
pip install numpy
pip install scipy
apt-get install python-matplotlib
pip install scikit-learn
```

## Sublime Text 2配置
### Python.sublime-build设置
点击**Preferences -> Browse Packages**在弹出的文件夹中，找到Python文件夹下的Python.sublime-build文件，打开后输入：

``` cmd
{
    "cmd": ["python", "-u", "$file"],
    "path":"/usr/bin",
    "file_regex": "^[ ]*File \"(...*?)\", line ([0-9]*)",
    "selector": "source.python"
}
```

花括号内的第一行和三四行是默认值，你需要做的是添加第二行——Python的路径。在Ubuntu中，Python的默认安装路径是`/usr/bin`。

### 必要包的安装
#### Package Control
这是Sublime Text 2中的基础包，安装完成后其它所有插件都可以通过此包来进行安装。

1. Sublime Text 2界面中按**Ctrl+`**
2. 在弹出的对话框中输入
```
import urllib2,os;pf='Package Control.sublime-package';ipp=sublime.installed_packages_path();os.makedirs(ipp) if not os.path.exists(ipp) else None;open(os.path.join(ipp,pf),'wb').write(urllib2.urlopen('http://sublime.wbond.net/'+pf.replace(' ','%20')).read())
```
3. 重启Sublime Text 2，如果在 **Preferences -> Package Settings**中见到Package Control这一项，就说明安装成功了。同理，如果以下提到的包能在Package Settings中找到，就说明安装成功了。

#### 其他包
**Ctrl+Shift+P**调出对话框，输入`install packages`找到对应程序，点击，并在后续对话框中输入想要安装的包的名字：

- **All Autocomplete**：自动完成当前文件的单词。
- **SublimeCodeIntel**: 为部分语言增强自动完成功能，其中包括Python。
- **SublimeREPL**: 允许你在编辑界面直接运行Python解释器。SublimeREPL安装完成后可以点选**Preference -> Key Bindings-User**设定运行代码的快捷键组合。由于我是个重度R user，所以把快捷键设置成了**"ctrl+r"**:

	``` cmd
	[{"keys":["ctrl+r"],
	"caption": "SublimeREPL: Python - RUN current file",
	"command": "run_existing_window_command", "args":
	{
	"id": "repl_python_run",
	"file": "config/Python/Main.sublime-menu"
	}}
	]
	```
	在尝试用**ctrl+r**调用SublimeREPL包运行`*.py`代码时，可能会碰到`getenv()`的报错，需要通过**Preferences -> Package Settings -> SublimeREPL -> Settings-User**来设定。

	``` cmd
	{
	    "getenv_command": false
	}
	```

在Sublime Text 2中，除了SublimeREPL外，还有一个简单的解决方案：**Tools -> Build System -> Python**。那么设定完成后，当你想运行当前python文件时，你可以点选**Tools -> Build**，也可以按快捷键**Ctrl+B**。


## References

- [设置 Sublime Text 的 Python 开发环境](http://www.open-open.com/lib/view/open1369960978459.html)
- [Sublime Text2安装Package Control](http://blog.csdn.net/del1214/article/details/8092266)
- [How do I run Python code from Sublime Text 2](http://stackoverflow.com/questions/8551735/how-do-i-run-python-code-from-sublime-text-2)