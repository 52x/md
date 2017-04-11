title: 如何确定python运行所在的文件位置
date: 2015-01-23 01:56:38
tags: python 
---
如何确定python脚本的位置？
python脚本有个`__file__`属性，能够确定脚本所在的位置。但由于运行文件夹的不同，`__file__`的相对路径也不同。

<!--more-->

## 如何确定当前脚本的绝对路径

```python
# -*- coding: utf-8 -*-
import os
print os.path.realpath(__file__)
```

## 如何确定当前脚本所在的文件夹

```python
# -*- coding: utf-8 -*-
import os
print os.path.dirname(os.path.realpath(__file__))
```

## 如何确定当前脚本所在的文件夹的父文件夹

```python
# -*- coding: utf-8 -*-
import os
print os.path.dirname(os.path.dirname(os.path.realpath(__file__)))
```

## 一种编程方式

在编写程序的时候，想要使用一些绝对路径相关的变量，会单独定义一个脚本如 `pathinfo.py`

```python
import os

def src_dir():
	return os.path.dirname(os.path.realpath(__file__))
	
def code_dir():
	return os.path.dirname(src_dir())
	
def log_dir():
	return code_dir()+'/log'
	
def conf_dir():
	return code_dir()+'/conf'
```

这样，使用的时候`import pathinfo`即可