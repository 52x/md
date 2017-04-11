---
title: LeetCode-Number of 1 Bits
date: 2016-05-18 18:50:41
tags: LeetCode
toc: true
categories: 算法
---

## Question

> Write a function that takes an unsigned integer and returns the number of ’1' bits it has (also known as the Hamming weight).

> For example, the 32-bit integer ’11' has binary representation 00000000000000000000000000001011, so the function should return 3.

## 解说

这道题的意思是统计32位整数二进制格式下的‘1’的个数。

<!--more-->

## Solution

### right-shift counting
The simplest technique is right-shift the number 32 times, counting the number of times you right-shift a one bit off.

```c
int hammingWeight_A(uint32_t num) {
    int ret = 0;
    int cur = num;
    int i;
    for (i = 0; i < 32; i++) {
        ret += cur & 1;
        cur >>= 1;
    }
    return ret;
}
```

A similar alternative checks each bit of the number from right to left.

```c
int hammingWeight_B(uint32_t num) {
 int ret = 0;
    int mask = 1;
    int i;
    for (i = 0; i < 32; i++) {
        if ((num & mask) != 0) ret++;
        mask <<= 1;
    }
    return ret;
}
```

### bit manipulation trick

In my interview at Microsoft, I composed essentially `hammingWeight_B()`, which impressed the interviewer well enough. But then he showed me a different technique, which I have to admit is pretty clever. It begins with the observation that, when you subtract a number by 1, all of the lowest bits change up to and including the lowest 1 bit; but the rest of the bits stay the same. So if I do a bitwise AND of n with n − 1, essentially I will remove the last one bit from n.

Once we observe this, we have only to write code that counts how many times we can remove the final bit in this way before we reach a number with no 1 bits at all (i.e., 0).

```c
int hammingWeight_C(uint32_t num) {
    int ret = 0;
    int cur = num;
    while (cur != 0) {
        cur &= cur - 1;
        ret++;
    }
    return ret;
}
```

### more better bit trick
Much later, I heard of yet another technique, which is yet more clever.

```c
int hammingWeight_D(uint32_t num) {
    int ret;
    ret = (num & 0x55555555)
        + ((num >> 1) & 0x55555555);
    ret = (ret & 0x33333333)
        + ((ret >> 2) & 0x33333333);
    ret = (ret & 0x0F0F0F0F)
        + ((ret >> 4) & 0x0F0F0F0F);
    ret = (ret & 0x00FF00FF)
        + ((ret >> 8) & 0x00FF00FF);
    ret = (ret & 0x0000FFFF)
        + ((ret >> 16) & 0x0000FFFF);
    return ret;
}
```
`hammingWeight_D()`主要是采用分治的思想，通过0x55555555（0x01010101010101010101010101010101），统计相邻两位的‘1’的个数，也就是`ret = (num & 0x55555555) + ((num >> 1) & 0x55555555);`  
通过0x33333333（0x00110011001100110011001100110011），统计相邻四位的‘1’的个数，也就是`ret = (ret & 0x33333333) + ((ret >> 2) & 0x33333333);` 后面的代码以此类推。

例子：

ret = 0x50005308（01010000000000000101001100001000）

|二进制数|十六进制数|操作|
|-|-|-|
|01010000000000000101001100001000|0x50005308|原数|
|01010000000000000101001000000100|0x50005204|第一次运算后|
|00100000000000000010001000000001|0x20002201|第二次运算后|
|00000010000000000000010000000001|0x2000401|第三次运算后|
|00000000000000100000000000000101|0x20005|第四次运算后|
|00000000000000000000000000000111|0x7|第五次运算后|

## Reference

-  [Using bit operators](http://www.toves.org/books/bitops/#s3.3)
- [LeetCode-Number of 1 Bits](https://leetcode.com/articles/number-1-bits/)
