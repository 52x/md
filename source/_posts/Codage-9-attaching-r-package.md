title: "Things to Know about Loading R Packages"
date: 2015-10-09 21:58:09
categories: Codage|编程
tags: [R, library, require]
---

Just read an [article](http://yihui.name/en/2014/07/library-vs-require/) from Yihui's blog in which he recommends a lot using `library()` instead of `require()` to attach/load a package .

<!-- more -->

Output of `require()`
---------------------

So I wonder how does the source code of `require()` look like.

``` r
require
```

The source code pops up and shows that it's mainly about how to deal with the arguments using `if`/`else` control flow and `library()`.

There are 3 kinds of output:

1.  **TRUE** - successfully load a package or the package if already loaded;
2.  **Warning message and FALSE** - can't allocate the package;
3.  **Error message and FALSE** - find the package but can't load it.

What's tricky is when there is no such package, you will not find it out until the code try to access this package. Try this:

``` r
library(abcd)
```

    ## Error in library(abcd): there is no package called 'abcd'

``` r
require(abcd)
```

    ## Loading required package: abcd

    ## Warning in library(package, lib.loc = lib.loc, character.only = TRUE,
    ## logical.return = TRUE, : there is no package called 'abcd'

There is no package called "abcd", `library(abcd)` throws an error message which stops the process while `require(abcd)` just generates a warning message. That doesn't halt the program at all.

Searching for An Attached Package
---------------------------------

Could `library()` tell you whether a package is already attached? No. But this function returns a logical value telling you if the "load" action is successfully executed.

So another big difference between the 2 functions is `require()` can tell you the status of a package - if it is already attached - before it runs the "load" action (yes, using exactly the `library()` function).

How does the `require()` search for a attached package? In the source code, we can see that:

``` r
loaded <- paste("package", package, sep = ":") %in% search()
```

`search()` gives a list of **attached** packages. Don't mix it with `installed.packages()` which gives, apprently, the list of **installed** packages. For example, in my PC, the `dplyr` package is already installed but not attached:

``` r
"dplyr" %in% installed.packages()
```

    ## [1] TRUE

``` r
"package:dplyr" %in% search()
```

    ## [1] FALSE

``` r
# `base` package is loaded by default 
"package:base" %in% search()
```

    ## [1] TRUE

This gives you an alternative way to see if a package is loaded or not.

Double Colon Operator
---------------------

Another simple way to use the functions from a package is using `::` operator. This operator returns the function from package even you didn't load it. Compare the 2 chunks below:

``` r
# chunk 1
summarise(group_by(select(mtcars, c(cyl, mpg, wt)), cyl),
          avg_mpg = mean(mpg),
          avg_wt = mean(wt))
```

    ## Error in eval(expr, envir, enclos): could not find function "summarise"

``` r
# chunk 2
dplyr::summarise(
  dplyr::group_by(
    dplyr::select(mtcars, c(cyl, mpg, wt)), cyl), 
  avg_mpg = mean(mpg),avg_wt = mean(wt))
```

    ## Source: local data frame [3 x 3]
    ## 
    ##     cyl  avg_mpg   avg_wt
    ##   (dbl)    (dbl)    (dbl)
    ## 1     4 26.66364 2.285727
    ## 2     6 19.74286 3.117143
    ## 3     8 15.10000 3.999214

When should we use `::`? It depends on the context.

* Is there any function masked by the lately loaded package? Do you need to use them? 
* Does the program frequently use those functions? Do you want to code like `chunk1` after `library(dplyr)` or you can stand adding `dplyr::` before every `dplyr` functions?

There is also an operator `:::`, help yourself to figure it out (I don't use it, for now).

Detaching A Package
-------------------

Now comes the question: how to detach a package?

``` r
library(dplyr)
```

    ## Attaching package: 'dplyr'
    ## 
    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag
    ## 
    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
detach("package:dplyr", unload = T)
"package:dplyr" %in% search()
```

    ## [1] FALSE

------------------------------------------------------------------------

Reference
---------

1.  [library() vs require() in R | Yihui Xie]((http://yihui.name/en/2014/07/library-vs-require/))
2.  [StackOverflow: What is the difference between require() and library()?](http://stackoverflow.com/questions/5595512/what-is-the-difference-between-require-and-library)
3.  [StackOverflow: Is it a good practice to call functions in a package via ::](http://stackoverflow.com/questions/23232791/is-it-a-good-practice-to-call-functions-in-a-package-via)
