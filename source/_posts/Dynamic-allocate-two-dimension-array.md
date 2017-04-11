---
title: 动态分配二维数组
date: 2016-05-21 16:37:49
toc: true
categories: C语言
---

为二维数组动态分配内存涉及以下两个问题：

- 数组元素是否需要连续
- 数组是否规则

在这里我们暂时不考虑数组是否规则，我们从数组元素的分配是否连续考虑。

##  已知第二维

```c
// 数组指针
char (*a1)[COLUMNS];
a1 = (char(*)[COLUMNS])calloc(ROWS,sizeof(char*));
```

<!--more-->

## 已知第一维

```c
	// 指针数组
	char *a2[ROWS];
	for(int i = 0; i < ROWS; i++) {
		a2[i] = (char*)calloc(COLUMNS, sizeof(char));
	}
```

## 已知第一维，一次分配内存（保证内存分配的连续性）

```c
    // 指针数组
    char *a3[ROWS];
    a3[0] = (char *)calloc(ROWS*COLUMNS, sizeof(char));
    for(int i = 1; i < ROWS; i++) {
        a3[i] = a3[i-1] + COLUMNS;
    }
```

## 两维都未知

```c
    // 二级指针
    char **a4;
    a4 = (char **)calloc(ROWS, sizeof(char*));
    for(int i = 0; i < ROWS; i++) {
        a4[i] = (char*)calloc(COLUMNS, sizeof(char));
    }
```

## 两维都未知，一次分配内存（保证内存分配的连续性）

```c
    // 二级指针
    char **a5;
    a5 = (char**)calloc(ROWS, sizeof(char*));
    a5[0] = (char*)calloc(ROWS*COLUMNS, sizeof(char));
    for(int i = 1; i < ROWS; i++) {
        a5[i] = a5[i-1] + COLUMNS;
    }
```

## 完整代码
```c
/*
 * 二维数组的动态内存分配
 * 测试指针和数组之间的关系
 */

#include <stdio.h>
#include <stdlib.h>

#define ROWS 6
#define COLUMNS 5

#define P(a) \
        { \
            printf("sizeof(%s):\t%d\n",#a, (sizeof((a)))); \
            printf("sizeof(%s[0]):\t%d\n",#a,(sizeof((a)[0]))); \
        }

#define OUT(a) \
        {        \
         printf("%s:\t  addr:%p\n",(#a),(a)); \
         for(int i = 0; i < ROWS; i++){ \
            printf("%s[%d]:\t  addr:%p\n",#a,i,((a)[i])); \
         } \
         for(int i = 0; i < ROWS; i++){ \
            for(int j = 0; j < COLUMNS; j++){ \
                printf("%s[%d][%d]: addr:%p\t",(#a),i,j,&a[i][j]);\
            } \
            printf("\n"); \
         } \
        }

// 打印数组的地址信息
#define INFO(a) \
        { \
         P(a) \
         OUT(a) \
        }

int main(int argc, char *argv[])
{
    printf("-----二维数组-----\n");
    printf("a[%d][%d]\n",ROWS,COLUMNS);
    // #1 已知第二维
    // 数组指针
    char (*a1)[COLUMNS];
    a1 = (char(*)[COLUMNS])calloc(ROWS,sizeof(char*));
    printf("*****数组指针*****\n");
    INFO(a1)

    // #2 已知第一维
    // 指针数组
    char *a2[ROWS];
    for(int i = 0; i < ROWS; i++) {
        a2[i] = (char*)calloc(COLUMNS, sizeof(char));
    }
    printf("*****指针数组*****\n");
    INFO(a2)

    // #3 已知第一维，一次分配内存（保证内存分配的连续性）
    // 指针数组
    char *a3[ROWS];
    a3[0] = (char *)calloc(ROWS*COLUMNS, sizeof(char));
    for(int i = 1; i < ROWS; i++) {
        a3[i] = a3[i-1] + COLUMNS;
    }
    printf("*****指针数组，一次分配内存*****\n");
    INFO(a3)

    // #4 两维都未知
    // 二级指针
    char **a4;
    a4 = (char **)calloc(ROWS, sizeof(char*));
    for(int i = 0; i < ROWS; i++) {
        a4[i] = (char*)calloc(COLUMNS, sizeof(char));
    }
    printf("*****两维都未知*****\n");
    INFO(a4)

    // #5 两维都未知，一次分配内存（保证内存分配的连续性）
    // 二级指针
    char **a5;
    a5 = (char**)calloc(ROWS, sizeof(char*));
    a5[0] = (char*)calloc(ROWS*COLUMNS, sizeof(char));
    for(int i = 1; i < ROWS; i++) {
        a5[i] = a5[i-1] + COLUMNS;
    }
    printf("*****两维都未知，一次分配内存*****\n");
    INFO(a5)

    // 统一释放申请的内存空间
    // free a1
    free(a1);
    // free a2
    for(int i = 0; i < ROWS; i++) {
        free(a2[i]);
    }
    // free a3
    free(a3[0]);
    // free a4
    for(int i = 0; i < ROWS; i++) {
        free(a4[i]);
    }
    free(a4);
    // free a5
    free(a5[0]);
    free(a5);
    return 0;
}
```

## 输出结果
```bash
-----二维数组-----
a[6][5]
*****数组指针*****
sizeof(a1):	4
sizeof(a1[0]):	5
a1:	  addr:0x9bb2008
a1[0]:	  addr:0x9bb2008
a1[1]:	  addr:0x9bb200d
a1[2]:	  addr:0x9bb2012
a1[3]:	  addr:0x9bb2017
a1[4]:	  addr:0x9bb201c
a1[5]:	  addr:0x9bb2021
a1[0][0]: addr:0x9bb2008	a1[0][1]: addr:0x9bb2009	a1[0][2]: addr:0x9bb200a	a1[0][3]: addr:0x9bb200b	a1[0][4]: addr:0x9bb200c	
a1[1][0]: addr:0x9bb200d	a1[1][1]: addr:0x9bb200e	a1[1][2]: addr:0x9bb200f	a1[1][3]: addr:0x9bb2010	a1[1][4]: addr:0x9bb2011	
a1[2][0]: addr:0x9bb2012	a1[2][1]: addr:0x9bb2013	a1[2][2]: addr:0x9bb2014	a1[2][3]: addr:0x9bb2015	a1[2][4]: addr:0x9bb2016	
a1[3][0]: addr:0x9bb2017	a1[3][1]: addr:0x9bb2018	a1[3][2]: addr:0x9bb2019	a1[3][3]: addr:0x9bb201a	a1[3][4]: addr:0x9bb201b	
a1[4][0]: addr:0x9bb201c	a1[4][1]: addr:0x9bb201d	a1[4][2]: addr:0x9bb201e	a1[4][3]: addr:0x9bb201f	a1[4][4]: addr:0x9bb2020	
a1[5][0]: addr:0x9bb2021	a1[5][1]: addr:0x9bb2022	a1[5][2]: addr:0x9bb2023	a1[5][3]: addr:0x9bb2024	a1[5][4]: addr:0x9bb2025	
*****指针数组*****
sizeof(a2):	24
sizeof(a2[0]):	4
a2:	  addr:0xbfbbbc60
a2[0]:	  addr:0x9bb2028
a2[1]:	  addr:0x9bb2038
a2[2]:	  addr:0x9bb2048
a2[3]:	  addr:0x9bb2058
a2[4]:	  addr:0x9bb2068
a2[5]:	  addr:0x9bb2078
a2[0][0]: addr:0x9bb2028	a2[0][1]: addr:0x9bb2029	a2[0][2]: addr:0x9bb202a	a2[0][3]: addr:0x9bb202b	a2[0][4]: addr:0x9bb202c	
a2[1][0]: addr:0x9bb2038	a2[1][1]: addr:0x9bb2039	a2[1][2]: addr:0x9bb203a	a2[1][3]: addr:0x9bb203b	a2[1][4]: addr:0x9bb203c	
a2[2][0]: addr:0x9bb2048	a2[2][1]: addr:0x9bb2049	a2[2][2]: addr:0x9bb204a	a2[2][3]: addr:0x9bb204b	a2[2][4]: addr:0x9bb204c	
a2[3][0]: addr:0x9bb2058	a2[3][1]: addr:0x9bb2059	a2[3][2]: addr:0x9bb205a	a2[3][3]: addr:0x9bb205b	a2[3][4]: addr:0x9bb205c	
a2[4][0]: addr:0x9bb2068	a2[4][1]: addr:0x9bb2069	a2[4][2]: addr:0x9bb206a	a2[4][3]: addr:0x9bb206b	a2[4][4]: addr:0x9bb206c	
a2[5][0]: addr:0x9bb2078	a2[5][1]: addr:0x9bb2079	a2[5][2]: addr:0x9bb207a	a2[5][3]: addr:0x9bb207b	a2[5][4]: addr:0x9bb207c	
*****指针数组，一次分配内存*****
sizeof(a3):	24
sizeof(a3[0]):	4
a3:	  addr:0xbfbbbc78
a3[0]:	  addr:0x9bb2088
a3[1]:	  addr:0x9bb208d
a3[2]:	  addr:0x9bb2092
a3[3]:	  addr:0x9bb2097
a3[4]:	  addr:0x9bb209c
a3[5]:	  addr:0x9bb20a1
a3[0][0]: addr:0x9bb2088	a3[0][1]: addr:0x9bb2089	a3[0][2]: addr:0x9bb208a	a3[0][3]: addr:0x9bb208b	a3[0][4]: addr:0x9bb208c	
a3[1][0]: addr:0x9bb208d	a3[1][1]: addr:0x9bb208e	a3[1][2]: addr:0x9bb208f	a3[1][3]: addr:0x9bb2090	a3[1][4]: addr:0x9bb2091	
a3[2][0]: addr:0x9bb2092	a3[2][1]: addr:0x9bb2093	a3[2][2]: addr:0x9bb2094	a3[2][3]: addr:0x9bb2095	a3[2][4]: addr:0x9bb2096	
a3[3][0]: addr:0x9bb2097	a3[3][1]: addr:0x9bb2098	a3[3][2]: addr:0x9bb2099	a3[3][3]: addr:0x9bb209a	a3[3][4]: addr:0x9bb209b	
a3[4][0]: addr:0x9bb209c	a3[4][1]: addr:0x9bb209d	a3[4][2]: addr:0x9bb209e	a3[4][3]: addr:0x9bb209f	a3[4][4]: addr:0x9bb20a0	
a3[5][0]: addr:0x9bb20a1	a3[5][1]: addr:0x9bb20a2	a3[5][2]: addr:0x9bb20a3	a3[5][3]: addr:0x9bb20a4	a3[5][4]: addr:0x9bb20a5	
*****两维都未知*****
sizeof(a4):	4
sizeof(a4[0]):	4
a4:	  addr:0x9bb20b0
a4[0]:	  addr:0x9bb20d0
a4[1]:	  addr:0x9bb20e0
a4[2]:	  addr:0x9bb20f0
a4[3]:	  addr:0x9bb2100
a4[4]:	  addr:0x9bb2110
a4[5]:	  addr:0x9bb2120
a4[0][0]: addr:0x9bb20d0	a4[0][1]: addr:0x9bb20d1	a4[0][2]: addr:0x9bb20d2	a4[0][3]: addr:0x9bb20d3	a4[0][4]: addr:0x9bb20d4	
a4[1][0]: addr:0x9bb20e0	a4[1][1]: addr:0x9bb20e1	a4[1][2]: addr:0x9bb20e2	a4[1][3]: addr:0x9bb20e3	a4[1][4]: addr:0x9bb20e4	
a4[2][0]: addr:0x9bb20f0	a4[2][1]: addr:0x9bb20f1	a4[2][2]: addr:0x9bb20f2	a4[2][3]: addr:0x9bb20f3	a4[2][4]: addr:0x9bb20f4	
a4[3][0]: addr:0x9bb2100	a4[3][1]: addr:0x9bb2101	a4[3][2]: addr:0x9bb2102	a4[3][3]: addr:0x9bb2103	a4[3][4]: addr:0x9bb2104	
a4[4][0]: addr:0x9bb2110	a4[4][1]: addr:0x9bb2111	a4[4][2]: addr:0x9bb2112	a4[4][3]: addr:0x9bb2113	a4[4][4]: addr:0x9bb2114	
a4[5][0]: addr:0x9bb2120	a4[5][1]: addr:0x9bb2121	a4[5][2]: addr:0x9bb2122	a4[5][3]: addr:0x9bb2123	a4[5][4]: addr:0x9bb2124	
*****两维都未知，一次分配内存*****
sizeof(a5):	4
sizeof(a5[0]):	4
a5:	  addr:0x9bb2130
a5[0]:	  addr:0x9bb2150
a5[1]:	  addr:0x9bb2155
a5[2]:	  addr:0x9bb215a
a5[3]:	  addr:0x9bb215f
a5[4]:	  addr:0x9bb2164
a5[5]:	  addr:0x9bb2169
a5[0][0]: addr:0x9bb2150	a5[0][1]: addr:0x9bb2151	a5[0][2]: addr:0x9bb2152	a5[0][3]: addr:0x9bb2153	a5[0][4]: addr:0x9bb2154	
a5[1][0]: addr:0x9bb2155	a5[1][1]: addr:0x9bb2156	a5[1][2]: addr:0x9bb2157	a5[1][3]: addr:0x9bb2158	a5[1][4]: addr:0x9bb2159	
a5[2][0]: addr:0x9bb215a	a5[2][1]: addr:0x9bb215b	a5[2][2]: addr:0x9bb215c	a5[2][3]: addr:0x9bb215d	a5[2][4]: addr:0x9bb215e	
a5[3][0]: addr:0x9bb215f	a5[3][1]: addr:0x9bb2160	a5[3][2]: addr:0x9bb2161	a5[3][3]: addr:0x9bb2162	a5[3][4]: addr:0x9bb2163	
a5[4][0]: addr:0x9bb2164	a5[4][1]: addr:0x9bb2165	a5[4][2]: addr:0x9bb2166	a5[4][3]: addr:0x9bb2167	a5[4][4]: addr:0x9bb2168	
a5[5][0]: addr:0x9bb2169	a5[5][1]: addr:0x9bb216a	a5[5][2]: addr:0x9bb216b	a5[5][3]: addr:0x9bb216c	a5[5][4]: addr:0x9bb216d	
```

## 参考

- [二维数组的动态分配及参数传递](http://www.cnblogs.com/bigshow/archive/2009/01/03/1367661.html) 
