---
title: autotools使用介绍
date: 2016-04-04 16:37:27
tags: tools
toc: true
---

## autotools系列工具—-自动生成Makefile

在较大项目中, 如果手动维护Makefile, 那将是一件复杂并痛苦的事情. 那么, 有没有一种轻松的手段生成Makefile呢? autotools系列工具正是在这样的呼声中诞生的. 它只需用户输入简单的目标文件, 依赖文件, 文件目录等就可以轻松地生成Makefile了. 另外, 这些工具还可以完成系统配置信息的收集, 从而可以方便地处理各种移植性问题. autotools是系列工具, 它含有:

- autoscan
- aclocal
- autoconf
- autoheader
- automake

## autotools 使用流程
下面用一个简单的hello.c程序, 演示autotools的使用流程. hello.c如下:

```bash
username@pc:~/automake$ ls
hello.c
username@pc:~/automake$ cat hello.c
#include 
int main()
{
  printf("Hello, autotools!\n");
  return 0;
}
```

### 使用autoscan命令自动生成configure.scan文件

<!--more-->

它会在给定目录及其子目录树中检查源文件, 若没有给出目录, 就在当前目录及其子目录树中进行检查.它会搜索源文件以寻找一般的移植性问题并创建一个文件"configure.scan", 该文件就是接下来autoconf要用到的"configure.ac"原型.

```bash
username@pc:~/automake$ autoscan
username@pc:~/automake$ ls
autoscan.log  configure.scan  hello.c
```

### 将configure.scan重命名为configure.in 

并做适当修改 configure.scan的内容:

```bash
username@pc:~/automake$ cat configure.scan
#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.65])
AC_INIT([FULL-PACKAGE-NAME], [VERSION], [BUG-REPORT-ADDRESS])
AC_CONFIG_SRCDIR([hello.c])
AC_CONFIG_HEADERS([config.h])

# Checks for programs.
AC_PROG_CC

# Checks for libraries.

# Checks for header files.

# Checks for typedefs, structures, and compiler characteristics.

# Checks for library functions.

AC_OUTPUT
```

将configure.scan重命名为configure.ac

```bash
username@pc:~/automake$ mv configure.scan configure.in
```

根据具体情况, 适当修改, 以下加粗部分是修改的内容:

```bash
username@pc:~/automake$ cat configure.in
#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.65])
#AC_INIT([FULL-PACKAGE-NAME], [VERSION], [BUG-REPORT-ADDRESS]) 
AC_INIT(hello,1.0)
AC_CONFIG_SRCDIR([hello.c])
AC_CONFIG_HEADERS([config.h])
AM_INIT_AUTOMAKE

# Checks for programs.
AC_PROG_CC

# Checks for libraries.

# Checks for header files.

# Checks for typedefs, structures, and compiler characteristics.

# Checks for library functions.
AC_CONFIG_FILES([Makefile])
AC_OUTPUT
```

说明:

- 以"#"号开始的行为注释
- AC_PREREQ宏声明本文要求的autoconf版本, 如本例中的版本 2.65
- AC_INIT宏用来定义软件的名称和版本等信息, 在本例中省略了BUG-REPROT-ADDRESS, 一般为作者的E-mail
- AM_INIT_AUTOMAKE是手动添加的, 它是automake所必备的宏, 也同前面一样, PACKAGE是所要产生软件套件的名称,VERSION是版本编号.
- AC_CONFIG_SCRDIR宏用来侦测所指定的源码文件是否存在, 来确定源码目录的有效性. 在此处指当前目录下hello.c
- AC_CONFIG_FILES宏用于生成相应的Makefile文件.

### 运行aclocal命令,生成"aclocal.m4"文件

该文件主要处理本地的宏定义

```bash
username@pc:~/automake$ aclocal
username@pc:~/automake$ ls
aclocal.m4 autom4te.cache  autoscan.log  configure.in  hello.c
```

### 运行autoconf命令生成configure可执行文件

```bash
username@pc:~/automake$ autoconf
username@pc:~/automake$ ls
aclocal.m4 autom4te.cache  autoscan.log  configure  configure.in  hello.c
```

### 运行autoheader命令, 生成config.h.in文件.

该工具通常会从"acconfig.h"文件中复制用户附加的符号定义. 本例中没有附加的符号定义, 所以不需要创建"acconfig.h"文件.

```bash
username@pc:~/automake$ autoheader
username@pc:~/automake$ ls
aclocal.m4 autom4te.cache  autoscan.log  config.h.in  configure  configure.in  hello.c
```

### 运行automake命令, 生成Makefile.in文件

这一步是创建Makefile很重要的一步, automake要用的脚本配置文件是Makefile.am, 用户需要自己创建相应的文件. 之后, automake工具将自动转换成Makefile.in 本例中, 创建的文件为Makefile.am, 内容如下:

```bash
username@pc:~/automake$ cat Makefile.am
AUTOMAKE_OPTIONS=foreign
bin_PROGRAMS=hello
hello_SOURCES=hello.c
```

说明:

- 其中的AUTOMAKE_OPTIONS为设置automake的选项. 由于GNU对自己发布的软件有严格的规范, 比如必须附带许可证声明文件COPYING等, 否则automake执行时会报错. automake提供了3中软件等级:foreign, gnu和gnits, 供用户选择. 默认级别是gnu. 在本例中, 使用了foreign等级, 它只检测必须的文件.
- bin_PROGRAMS定义要产生的执行文件名. 如果要产生多个执行文件, 每个文件名用空格隔开
- hello_SOURCES 定义"hello"这个可执行程序所需的原始文件. 如果"hello"这个程序是由多个源文件所产生的, 则必须把它所用到的所有源文件都列出来, 并用空格隔开. 如果要定义多个可执行程序, 那么需要对每个可执行程序建立对应的file_SOURCES.

在这里使用"--add-missiing"选项可以让automake自动添加一些必须的脚本文件.

```bash
username@pc:~/automake$ automake --add-missing
configure.in:7: installing `./install-sh'
configure.in:7: installing `./missing'
Makefile.am: installing `./depcomp'
username@pc:~/automake$ ls
aclocal.m4      autoscan.log  configure     depcomp  install-sh   Makefile.in
autom4te.cache  config.h.in   configure.in  hello.c  Makefile.am  missing
```

### 运行configure, 生成Makfefile文件

```bash
username@pc:~/automake$ ./configure
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /bin/mkdir -p
checking for gawk... gawk
checking whether make sets $(MAKE)... yes
checking for gcc... gcc
checking whether the C compiler works... yes
checking for C compiler default output file name... a.out
checking for suffix of executables...
checking whether we are cross compiling... no
checking for suffix of object files... o
checking whether we are using the GNU C compiler... yes
checking whether gcc accepts -g... yes
checking for gcc option to accept ISO C89... none needed
checking for style of include used by make... GNU
checking dependency style of gcc... gcc3
configure: creating ./config.status
config.status: creating Makefile
config.status: creating config.h
config.status: executing depfiles commands
username@pc:~/automake$ ls
aclocal.m4      config.h     config.status  depcomp     Makefile     missing
autom4te.cache  config.h.in  configure      hello.c     Makefile.am  stamp-h1
autoscan.log    config.log   configure.in   install-sh  Makefile.in
```

autotools生成Makefile流程图如下: 

![autotools生成的Makefile流程图](http://7xsd89.com1.z0.glb.clouddn.com/autotools_1.jpg)
![autotools生成Makefile的流程图](http://7xsd89.com1.z0.glb.clouddn.com/autotools_process_meitu_1.jpg)

## 使用由autotools生成的Makefile

autotools生成的Makefile具有以下主要功能: 

### make 编译源程序

键入make, 默认执行"make all"命令

```bash
username@pc:~/automake$ make
make  all-am
make[1]: Entering directory '/home/username/automake'
gcc -DHAVE_CONFIG_H -I.     -g -O2 -MT hello.o -MD -MP -MF .deps/hello.Tpo -c -o hello.o hello.c
mv -f .deps/hello.Tpo .deps/hello.Po
gcc  -g -O2   -o hello hello.o
make[1]: Leaving directory `/home/username/automake'
```

此时在本目录下就生成了可执行文件"hello", 运行"./hello"就能看到程序的执行结果:

```bash
username@pc:~/automake$ ./hello
Hello, autotools!
```

### make install 执行该命令

可以把程序安装到系统目录中

```bash
username@pc:~/automake$ sudo make install
```

此时, 直接在console输入hello, 就可以看到程序的运行结果 

### make clean 

清除之前所编译的可执行文件及目标文件

```bash
username@pc:~/automake$ make clean
test -z "hello" || rm -f hello
rm -f *.o
```

### make dist 

将程序和相关的文档打包为一个压缩文档以供发布

```bash
username@pc:~/automake$ make dist
username@pc:~/automake$ ls -l hello-1.0.tar.gz
hello-1.0.tar.gz
```

可见该命令生成了一个hello-1.0.tar.gz的压缩文档.

## 参考

- [autotools系列工具--自动生成Makefile](http://www.worldhello.net/2010/04/07/954.html)
- [autotools官方教程文档](https://www.lrde.epita.fr/~adl/dl/autotools.pdf)
