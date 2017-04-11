---
title: C语言单元测试框架Check
date: 2016-04-10 14:43:50
tags: Check 
toc: true
categories: C语言
---

## 什么是Check

[Check](http://libcheck.github.io/check/)是C语言的一个单元测试框架。它提供一个小巧的单元测试接口。测试案例运行在各自独立的地址空间，所以断言失败和代码错误造成的段错误或者其他的信号可以被捕捉到。另外，测试的结果显示也兼容以下这些格式：Subunit、TAP、XML和通用的日志格式。

> Check is a unit testing framework for C. It features a simple interface for defining unit tests, putting little in the way of the developer. Tests are run in a separate address space, so both assertion failures and code errors that cause segmentation faults or other signals can be caught. Test results are reportable in the following: Subunit, TAP, XML, and a generic logging format.

## Check支持的平台

Check支持大部分的UNIX兼容平台。

<!--more-->

可以通过以下命令得到Check的最新版本。

```bash
git clone https://github.com/libcheck/check.git
```

## Check使用

本人以一个只做减法的工程来进行说明Check的使用。

直接看工程目录结构：

```bash
.
├── include
│   ├── sub.h
│   └── unit_test.h
├── makefile
├── sub
│   └── sub.c
└── unit_test
    ├── test_main.c
    └── test_sub.c
```

### sub.c文件

```bash
#include "sub.h"

int sub(int a, int b) {
    return 0;
}
```

### sub.h文件

```bash
#ifndef _SUB_H
#define _SUB_H
int sub(int a, int b);
#endif
```

### unit_test.h文件

```bash
#ifndef _UNI_TEST_H
#define _UNI_TEST_H
#include "check.h"
Suite *make_sub_suite(void);
#endif
```

### test_sub.c文件

```bash
#include "check.h"
#include "unit_test.h"
#include "sub.h"
START_TEST(test_sub) {
    fail_unless(sub(6, 2) == 4, "error, 6 - 2 != 4"); 
}
END_TEST

Suite * make_sub_suite(void) {
    Suite *s = suite_create("sub");       // 建立Suite
    TCase *tc_sub = tcase_create("sub");  // 建立测试用例集
    suite_add_tcase(s, tc_sub);           // 将测试用例加到Suite中
    tcase_add_test(tc_sub, test_sub);     // 测试用例加到测试集中
    return s;
}
```

### test_main.c文件

```bash
#include "unit_test.h"
#include <stdlib.h>

int main(void) {
    int n, n1;
    SRunner *sr, *sr1;
    sr = srunner_create(make_sub_suite()); // 将Suite加入到SRunner
    srunner_run_all(sr, CK_NORMAL);
    n = srunner_ntests_failed(sr);         // 运行所有测试用例
    srunner_free(sr);
    return (n == 0) ? EXIT_SUCCESS : EXIT_FAILURE;
}
```

### makefile文件

```bash
vpath %.h include  #vpath 指定搜索路径
vpath %.c sub add
vpath %.c unit_test

objects = sub.o test_sub.o
test: test_main.c $(objects)
	gcc -I include $^ -o test -lcheck 

all: $(objects)
$(objects): %.o: %.c
	gcc -c -I include $< -o $@

.PHONY: clean
clean:
	rm *.o test
```

工程配置完成后，直接在工程目录下运行命令`make test`即可生成可执行文件test

```bash
gcc -c -I include sub/sub.c -o sub.o
gcc -c -I include unit_test/test_sub.c -o test_sub.o
gcc -I include unit_test/test_main.c sub.o test_sub.o -o test -lcheck 
```

之后运行`./test`就可以看到测试结果了。

```bash
Running suite(s): sub
0%: Checks: 1, Failures: 1, Errors: 0
unit_test/test_sub.c:12:F:sub:test_sub:0: error, 6 - 2 != 4
```

## 参考

* [Check—强大的C语言单元测试框架](http://blog.csdn.net/ZCF1002797280/article/details/50421336) 
* [Tutorial: Basic Unit Testing](http://libcheck.github.io/check/doc/check_html/check_3.html) 
