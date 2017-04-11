title: R读取Excel 2007版本(及以上)文件
date: 2015-11-24 21:46:44
categories: Codage|编程
tags: [R, Excel, xlsx]
---

从本地读取的文档时，除了用`read.csv`和`read.table`读取`.csv`或者`.txt`文件以外，还不可避免的碰到大量需要从`.xlsx`格式的文件中读取数据的情况，将其转换格式为`.csv`或者`.txt`当然是一种方法，曲径通幽的存入数据库，再用R调用数据库内容亦可，但是最直接的方式无非还是直接读取。下面是几种常见读取方式的比较，至于最终选择哪种读取方式，见仁见智。

<!-- more -->

## `xlsx`包

我们先随机生成一个文件，将它写入`demo.xlsx`文件中。我们将用到`xlsx`包中的`write.xlsx()`函数。

``` r
library(xlsx)
A = sample(LETTERS, 1000, replace = T)
B = rnorm(1000)
C = paste(sample(LETTERS, 1000, replace = T),
          sample(LETTERS, 1000, replace = T),
          sample(LETTERS, 1000, replace = T),
          sep = "")
D = rpois(1000, .2)

df <- data.frame(A, B, C, D)

write.xlsx(df, "demo.xlsx", sheetName = "demo", row.names = F)
```

这个包的依赖包包括`rJava`和`xslxjars`，在使用本包之前必须保证Java或jdk已经安装好、环境变量也已设置好。
`xlsx`提供两种读取的函数：`read.xlsx()`和`read.xlsx2()`，前者可直接读取，并自动判断数据类型；后者读取速度更快，但是需要手动补充每列的数据类型。

``` r
system.time(read.xlsx("demo.xlsx", sheetIndex = 1))
```

    ##    user  system elapsed 
    ##    6.83    0.07    4.37

```r
system.time(read.xlsx2("demo.xlsx", sheetIndex = 1,
                       colClasses = rep(c("numeric","character"),2)))
```

    ##    user  system elapsed 
    ##    0.66    0.04    0.22

可以看出第二种读取方式比第一种要快很多。

## `XLConnect`包

此包同样基于java：

``` r
library(XLConnect)

wb <- loadWorkbook("demo.xlsx")
system.time(readWorksheet(wb, sheet = "demo", header = T))
```
    ##    user  system elapsed 
    ##    0.36    0.03    0.14

与`xlsx`包的第二个读取函数相比，速度更快。
`xlsx`包里有`read.xlsx()`和`write.xlsx()`一对函数，同样的，`XLConnect`包里也有与`readWorksheet()`对应的`writeWorksheet()`函数。

-----------------
补充于 12/7/2015:
此函数无法自动将excel中的公式转换为数值。
如果一个excel文件中含有公式，则会报错。

-----------------

## `gdata`包

这个包是基于Perl语言的。Linux和Mac系统自带Perl但是Windows系统需要自行安装。如果R无法自动获取Perl编译器，则需要手动指定编译器的位置。

``` r
library(gdata)

perl_int <- "D:/Programming Tools/Perl64/bin/perl5.20.2.exe"
system.time(read.xls("demo.xlsx", sheet = 1, header = TRUE, perl = perl_int))
```

    ##    user  system elapsed 
    ##    0.00    0.00    1.92

同样也是很快速的读取方法。

## 其它包

-   32位系统中，可以使用`RODBC`包的`odbcConnectExcel()`函数来读取数据。但是这个函数只能用于32位系统，64位无法使用。
-   `xlsReadWrite`包中的`read.xls()`函数可用于读取`.xls`格式的文件，但无法读取`.xlsx`文件。

Reference
---------

[Read Excel files from R](http://www.r-bloggers.com/read-excel-files-from-r/)
