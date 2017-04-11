---
title: 位图排序的简单实现
date: 2016-03-30 19:08:05
tags: sort
categories: 算法
toc: true
---

## 位图排序简介

位图排序的直接思路是想通过有限位数（比如1位）去映射一个整数，从而节省存储空间，而间接带来的好处是给指定数据集合排序了。

## 实际案例介绍

本案例摘抄自《编程珠玑》一书。

输入：

>所输入的是一个文件，至多包含n个正整数，每个正整数都要小于n，这里n=10^7。如果输入时某一个整数出现了两次，就会产生一个致命的错误。这些整数与其他任何数据都不关联。

输出：

>以非递减形式输出经过排序的整数列表。

约束：

>至多（大概）只要1MB的可用主存；但是可用磁盘空间非常充足。运行时间至多只允许几分钟；10分钟是最适宜的运行时间。

## 代码实现如下

<!--more-->

``` java
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <math.h>

#define BITSPERWORD     32
#define SHIFT           5
#define MASK            0x1F
#define N               10000000

int a[1 + N/BITSPERWORD];    
int x[N];

void set(int i);
void clr(int i);
int test(int i);
int randint(int l, int u);
void swap(int *a, int *b);
void generate_rand_num();
void bitmap_sort();

int main(int argc, char *argv[])
{
    bitmap_sort();
}

void set(int i)
{
    a[i>>SHIFT] |= (1<<(i & MASK));
}

void clr(int i)
{
    a[i>>SHIFT] &= ~(1<<(i & MASK));
}

int test(int i)
{
    return a[i>>SHIFT] & (1<<(i & MASK));
}

int randint(int l, int u)
{
    int temp;
    srand((unsigned)time(NULL));
    temp = floor(1 + (1.0*rand()/RAND_MAX)*(u - l + 1));
    return temp;
}

void swap(int *a, int *b)
{
    int temp;
    temp = *a;
    *a = *b;
    *b = temp;
}

/*
 * 生成1～N之间的随机数
 */
void generate_rand_num()
{
    FILE *fp;
    int k = N;
    int i;
    fp = fopen("in.txt", "w");

    for (i = 0; i < N; i++){
        x[i] = i + 1;
    }
    for(i = 0; i < k; i++){
        swap(&x[i], &x[randint(i, N-1)]);
        fprintf(fp,"%d\n",x[i]);
    }
    fclose(fp);
}

/*
 * 位图排序
 */
void bitmap_sort()
{
    int i;
    FILE *in, *out;
    int num;
    in = fopen("in.txt", "r"); 
    out = fopen("out.txt", "w");

    generate_rand_num();

    for(i = 0; i < N; i++){
        clr(i);
    }
    while(fscanf(in, "%d", &num) != EOF){
        set(num);
    }
    for(i = 0; i < N; i++){
        if(test(i)){
            fprintf(out, "%d\n", i);
        }
    }
    fclose(in);
    fclose(out);
}
```