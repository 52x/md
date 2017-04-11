title: WIN7环境下IPython notebook的安装
date: 2015-12-07 20:59:49
categories: Codage|编程
tags: [Python, IPython notebook]
---

WIN7系统下 ipython notebook的安装过程备忘。
本机已安装python2.7.11，安装路径为 D:\stat-tools\Python27
以下安装过程以python2.7及相关路径为准。

<!-- more -->

## 环境支持
### 安装setuptools
setuptools用于便捷下载、搭建、安装、升级和卸载python包，其中的easy_install函数将在后面用到。

以管理员身份打开Powershell(打开后默认路径是`C:\Windows\system32>`，需要进入python安装路径)，输入(`>`符号后的命令)：

``` cmd
PS C:\Windows\system32> D:
PS D:\> cd stat-tools\python27
PS D:\stat-tools\python27> (Invoke-WebRequest https://bootstrap.pypa.io/ez_setup.py).Content | python -
```

如果以上命令无效，应该是powershell的版本较低，以上powershell命令需要3.0以上的版本(win7系统默认powershell版本为2.0，可输入`get-Host`查看版本信息)。有两种解决方案：

1. 官网下载.msu为扩展名的[powershell高版本安装包](https://www.microsoft.com/en-us/download/details.aspx?id=40855)，安装完成以后再运行以上命令
2. 安装setuptools前需要下载[ez_setup.py](https://bootstrap.pypa.io/ez_setup.py)(如果此链接失效，请去[python setuptools的网页](https://pypi.python.org/pypi/setuptools)中去找下载链接)。将ez_setup.py置于python2.7的安装路径下。然后输入`python ez_setup.py`安装。

### 安装pip
将[pip.py](https://bootstrap.pypa.io/get-pip.py)下载到与setuptools同样的路径下。更多相关信息可见[pip网站上的安装教程](https://pip.pypa.io/en/latest/installing/#install-or-upgrade-setuptools)。在powershell中输入:

``` cmd
PS D:\stat-tools\python27> python get-pip.py
```

python2.7自带了pip，但是上面的命令会将旧版本卸载并安装最新的版本。比如我的安装完成后会提示：

``` cmd
Installing collected packages: pip, wheel
  Found existing installation: pip 7.0.1
    Uninstalling pip-7.0.1:
      Successfully uninstalled pip-7.0.1
Successfully installed pip-7.1.2 wheel-0.26.0
```

### 添加环境变量
以管理员身份运行cmd，添加环境变量:

``` cmd
C:\Windows\system32> set PATH=%PATH%;D:\stat-tools\Python27;D:\stat-tools\Python27\Scripts
```
设定完成后请关闭重启cmd，输入`PATH`再次确认环境变量设置是否成功，有可能仅有`\Python27`设置成功，而后者没有。如果是这样，需要在此设置。

## 安装IPython notebook
### 安装IPython

cmd窗口中输入：
``` cmd
C:\Windows\system32> pip install ipython
```
命令将IPython安装到D:\stat-tools\Python27\Scripts下。
由于设定了环境变量，在C盘中输入的命令也能直接调用D盘的ipython。

``` cmd
C:\Windows\system32> ipython2
```

可以在窗口中看到如下的ipython2启动信息：

``` cmd
Python 2.7.10 (default, May 23 2015, 09:44:00) [MSC v.1500 64 bit (AMD64)]
Type "copyright", "credits" or "license" for more information.

IPython 4.0.1 -- An enhanced Interactive Python.
?         -> Introduction and overview of IPython's features.
%quickref -> Quick reference.
help      -> Python's own help system.
object?   -> Details about 'object', use 'object??' for extra details.
```

如果上面的提示之前还有warning信息，如：

``` cmd
WARNING: Readline services not available or not loaded.
WARNING: Proper color support under MS Windows requires the pyreadline library.
You can find it at:
http://ipython.org/pyreadline.html
```

说明之前ipython的安装并没有完全安装所有需求包，上面的这种情况，你需要另行安装`pyreadline`：

``` cmd
C:\Windows\system32> pip install pyreadline
```

如遇其它报错，请求助google。

### 安装notebook所需的包

安装三个包：`pyzmq`, `jinja2`和`tornado`。

``` cmd
C:\Windows\System32>pip install pyzmq
C:\Windows\System32>pip install jinja2
C:\Windows\System32>pip install tornado
```

### 运行IPython Notebook

在cmd中输入：

``` cmd
C:\Windows\system32> ipython2 notebook
```
如果遇到报错，如：

``` cmd
Traceback (most recent call last):
  File "d:\stat-tools\python27\lib\runpy.py", line 162, in _run_module_as_main
    "__main__", fname, loader, pkg_name)
  File "d:\stat-tools\python27\lib\runpy.py", line 72, in _run_code
    exec code in run_globals
  File "D:\stat-tools\Python27\Scripts\ipython2.exe\__main__.py", line 9, in <module>
  ...
```

尝试重新安装ipython:

``` cmd
C:\Windows\system32> pip uninstall ipython
C:\Windows\system32> pip install ipython[all]
```

这样做的原因是`pip install ipython`并不会安装所有的notebook依赖包，除非你在语句后指定`[all]`。

当需要再次运行ipython notebook的时候，只需要打开cmd，运行`ipython2 notebook`就可以了。

## Reference

* [Windows 下面安装和使用Python, IPython NoteBook (详细步骤)](http://my.oschina.net/u/1431433/blog/189337)