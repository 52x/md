---
title: 编译二进制的Makefile
date: 2016-08-27 10:55:35
tags:
 - linux
 - makefile
categories:
 - linux
---

一个编译二进制的Makefile，本Makefile修改自[https://github.com/latelee/Makefile_templet/blob/master/mult_dir_project_new/Makefile](https://github.com/latelee/Makefile_templet/blob/master/mult_dir_project_new/Makefile "https://github.com/latelee/Makefile_templet/blob/master/mult_dir_project_new/Makefile")

增加或删除文件夹时，只需修改`SOURCE_DIRS`和`INCLUDE_DIRS`。

 * `SOURCE_DIRS`：表示所有点C目录
 * `INCLUDE_DIRS`：表示所有点H目录

案例（可直接make）：[https://github.com/aeiouaoeiuv/commonly_makefile](https://github.com/aeiouaoeiuv/commonly_makefile "https://github.com/aeiouaoeiuv/commonly_makefile")

```
#
# This Makefile was modified from:
#     https://github.com/latelee/Makefile_templet/blob/master/mult_dir_project_new/Makefile
#
# usage: $ make
#        $ make CROSS_COMPILE=arm-arago-linux-gnueabi-
#
###############################################################################

# cross compile
CROSS_COMPILE =

CC = $(CROSS_COMPILE)gcc

# name of executable binary
TARGET = main

# directories of .c files, modified here if add/del directory
SOURCE_DIRS = src mod1 mod2

# directories of .h files, modified here if add/del directory
INCLUDE_DIRS = include mod1 mod2

# all .h directories
INCS = $(foreach n, $(INCLUDE_DIRS), -I./$(n))

# all .c files
SRCS = $(foreach n, $(SOURCE_DIRS), $(wildcard $(n)/*.c))

# all .o files
OBJS = $(patsubst %.c,%.o,$(SRCS))

CFLAGS =  -Wall -O2
CFLAGS += $(INCS)

LDFLAGS =

###############################################################################

ALL: $(TARGET)

$(TARGET): $(OBJS)
	$(CC) $(CFLAGS) $^ -o $(TARGET) $(LDFLAGS)

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

.PHONY: clean
clean:
	-rm -rf $(TARGET) $(OBJS)


```

