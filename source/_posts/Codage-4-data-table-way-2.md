title: data.table Way - Learning Note Part 2
date: 2015-09-25 20:41:04
categories: Codage|编程
tags: [R, data.table]
---

We resume the [learning note](http://papacochon.com/2015/09/23/Codage-3-data-table-way-1/).
<!-- more -->

``` r
library(data.table)
dtm <- as.data.table(mtcars)
```

The set() family
----------------

Overview of set family \* `set()` is a loopable low overhead version of `:=`; \* You can use `setnames()` to set or change column names; \* `setorder()` and `setcolorder()` reorder the rows, columns of a `data.table`. \* `setkey()` set keys in a `data.table`

### set()

To assign the column `am` to 1:

``` r
set(dtm, j = which(colnames(dtm)=='am'), value = 1)
head(dtm)
```

    ##     mpg cyl disp  hp drat    wt  qsec vs am gear carb
    ## 1: 21.0   6  160 110 3.90 2.620 16.46  0  1    4    4
    ## 2: 21.0   6  160 110 3.90 2.875 17.02  0  1    4    4
    ## 3: 22.8   4  108  93 3.85 2.320 18.61  1  1    4    1
    ## 4: 21.4   6  258 110 3.08 3.215 19.44  1  1    3    1
    ## 5: 18.7   8  360 175 3.15 3.440 17.02  0  1    3    2
    ## 6: 18.1   6  225 105 2.76 3.460 20.22  1  1    3    1

Now recover it.

``` r
set(dtm, j = which(colnames(dtm)=='am'), value = mtcars$am)
head(dtm)
```

    ##     mpg cyl disp  hp drat    wt  qsec vs am gear carb
    ## 1: 21.0   6  160 110 3.90 2.620 16.46  0  1    4    4
    ## 2: 21.0   6  160 110 3.90 2.875 17.02  0  1    4    4
    ## 3: 22.8   4  108  93 3.85 2.320 18.61  1  1    4    1
    ## 4: 21.4   6  258 110 3.08 3.215 19.44  1  0    3    1
    ## 5: 18.7   8  360 175 3.15 3.440 17.02  0  0    3    2
    ## 6: 18.1   6  225 105 2.76 3.460 20.22  1  0    3    1

We create a random data.table and randomly change its value

``` r
temp <- data.table(A = sample(1:9),
                   B = sample(1:9),
                   C = sample(1:9),
                   D = sample(1:9),
                   E = sample(1:9))

# for each row, randomly change 3 values to 10 
for (i in 1:9) set(temp, i, sample(5, 3), 10)
temp
```

    ##     A  B  C  D  E
    ## 1:  5 10  6 10 10
    ## 2: 10  5  2 10 10
    ## 3: 10 10  1  4 10
    ## 4: 10  3  4 10 10
    ## 5: 10  1 10 10  9
    ## 6:  7  6 10 10 10
    ## 7:  6 10 10  6 10
    ## 8:  8  9 10 10 10
    ## 9:  9  7 10 10 10

### setnames()

Still use the `data.table` we just created. Add ‘Col\_’ to the column names.

``` r
# setnames(x, old, new)
setnames(temp, names(temp), paste0('Col_', names(temp)))
head(temp, 3)
```

    ##    Col_A Col_B Col_C Col_D Col_E
    ## 1:     5    10     6    10    10
    ## 2:    10     5     2    10    10
    ## 3:    10    10     1     4    10

To remove the prefix `Col_`:

``` r
# when new is not provided, consider old as new
setnames(temp, gsub('^Col_', '', names(temp)))
head(temp, 3)
```

    ##     A  B C  D  E
    ## 1:  5 10 6 10 10
    ## 2: 10  5 2 10 10
    ## 3: 10 10 1  4 10

### setorder() and setcolorder()

To reorder the rows:

``` r
# wt in ascending order and mpg in descending order
setorder(dtm, am, -mpg)
# check the first 6 rows of each group
dtm[, head(.SD), by = am]
```

    ##     am  mpg cyl  disp  hp drat    wt  qsec vs gear carb
    ##  1:  0 24.4   4 146.7  62 3.69 3.190 20.00  1    4    2
    ##  2:  0 22.8   4 140.8  95 3.92 3.150 22.90  1    4    2
    ##  3:  0 21.5   4 120.1  97 3.70 2.465 20.01  1    3    1
    ##  4:  0 21.4   6 258.0 110 3.08 3.215 19.44  1    3    1
    ##  5:  0 19.2   6 167.6 123 3.92 3.440 18.30  1    4    4
    ##  6:  0 19.2   8 400.0 175 3.08 3.845 17.05  0    3    2
    ##  7:  1 33.9   4  71.1  65 4.22 1.835 19.90  1    4    1
    ##  8:  1 32.4   4  78.7  66 4.08 2.200 19.47  1    4    1
    ##  9:  1 30.4   4  75.7  52 4.93 1.615 18.52  1    4    2
    ## 10:  1 30.4   4  95.1 113 3.77 1.513 16.90  1    5    2
    ## 11:  1 27.3   4  79.0  66 4.08 1.935 18.90  1    4    1
    ## 12:  1 26.0   4 120.3  91 4.43 2.140 16.70  0    5    2

To reorder the columns by alphabetic order.

``` r
setcolorder(dtm, sort(names(dtm)))
head(dtm)
```

    ##    am carb cyl  disp drat gear  hp  mpg  qsec vs    wt
    ## 1:  0    2   4 146.7 3.69    4  62 24.4 20.00  1 3.190
    ## 2:  0    2   4 140.8 3.92    4  95 22.8 22.90  1 3.150
    ## 3:  0    1   4 120.1 3.70    3  97 21.5 20.01  1 2.465
    ## 4:  0    1   6 258.0 3.08    3 110 21.4 19.44  1 3.215
    ## 5:  0    4   6 167.6 3.92    4 123 19.2 18.30  1 3.440
    ## 6:  0    2   8 400.0 3.08    3 175 19.2 17.05  0 3.845

### setkey()

#### set character column as key

To make the `data.table` a database table. According to the document:

> setkey() sorts a data.table and marks it as sorted (with an attribute sorted). The sorted columns are the key. The key can be any columns in any order. The columns are sorted in ascending order always.

> setkey reorders (or sorts) the rows of a data.table by the columns provided. In versions 1.9+, for integer columns… It is extremely fast, but is limited by the range of integer values being \<= 1e5.

``` r
dti <- as.data.table(iris)

# Set Species as key
setkey(dti, Species)

# Query by Species == 'setosa'
head(dti['setosa'])
```

    ##    Sepal.Length Sepal.Width Petal.Length Petal.Width Species
    ## 1:          5.1         3.5          1.4         0.2  setosa
    ## 2:          4.9         3.0          1.4         0.2  setosa
    ## 3:          4.7         3.2          1.3         0.2  setosa
    ## 4:          4.6         3.1          1.5         0.2  setosa
    ## 5:          5.0         3.6          1.4         0.2  setosa
    ## 6:          5.4         3.9          1.7         0.4  setosa

``` r
# Select 'setosa' and 'versicolor' in the same time
dti[c('setosa', 'versicolor')][, head(.SD, 3), by = Species]
```

    ##       Species Sepal.Length Sepal.Width Petal.Length Petal.Width
    ## 1:     setosa          5.1         3.5          1.4         0.2
    ## 2:     setosa          4.9         3.0          1.4         0.2
    ## 3:     setosa          4.7         3.2          1.3         0.2
    ## 4: versicolor          7.0         3.2          4.7         1.4
    ## 5: versicolor          6.4         3.2          4.5         1.5
    ## 6: versicolor          6.9         3.1          4.9         1.5

#### set numeric column as key

``` r
dtm <- data.table(model = rownames(mtcars), mtcars)

# Set gear as key
setkey(dtm, gear)

# Select records with gear == 5
dtm[J(5)]
```

    ##             model  mpg cyl  disp  hp drat    wt qsec vs am gear carb
    ## 1:  Porsche 914-2 26.0   4 120.3  91 4.43 2.140 16.7  0  1    5    2
    ## 2:   Lotus Europa 30.4   4  95.1 113 3.77 1.513 16.9  1  1    5    2
    ## 3: Ford Pantera L 15.8   8 351.0 264 4.22 3.170 14.5  0  1    5    4
    ## 4:   Ferrari Dino 19.7   6 145.0 175 3.62 2.770 15.5  0  1    5    6
    ## 5:  Maserati Bora 15.0   8 301.0 335 3.54 3.570 14.6  0  1    5    8

``` r
# or 
dtm[.(5)]
```

    ##             model  mpg cyl  disp  hp drat    wt qsec vs am gear carb
    ## 1:  Porsche 914-2 26.0   4 120.3  91 4.43 2.140 16.7  0  1    5    2
    ## 2:   Lotus Europa 30.4   4  95.1 113 3.77 1.513 16.9  1  1    5    2
    ## 3: Ford Pantera L 15.8   8 351.0 264 4.22 3.170 14.5  0  1    5    4
    ## 4:   Ferrari Dino 19.7   6 145.0 175 3.62 2.770 15.5  0  1    5    6
    ## 5:  Maserati Bora 15.0   8 301.0 335 3.54 3.570 14.6  0  1    5    8

Be aware that dtm[5] only returns the 5th row.

#### set multiple columns as keys

``` r
# add a column: brand
dtm[, brand:=sapply(.SD[,model], function(x) strsplit(x, ' ')[[1]][1])]

setkey(dtm, brand, cyl)
dtm[.('Merc', 8)]
```

    ##          model  mpg cyl  disp  hp drat   wt qsec vs am gear carb brand
    ## 1:  Merc 450SE 16.4   8 275.8 180 3.07 4.07 17.4  0  0    3    3  Merc
    ## 2:  Merc 450SL 17.3   8 275.8 180 3.07 3.73 17.6  0  0    3    3  Merc
    ## 3: Merc 450SLC 15.2   8 275.8 180 3.07 3.78 18.0  0  0    3    3  Merc

What if there is no such key?

``` r
dtm[.('Renault', 8)]
```

    ##    model mpg cyl disp hp drat wt qsec vs am gear carb   brand
    ## 1:    NA  NA   8   NA NA   NA NA   NA NA NA   NA   NA Renault

It returns NA in all columns but the 2 keys.

#### select groups

By default, subsetting `data.table` returns all the records, it is related to argument `mult` of which the default value is `all`. You can also specify ‘first’ or ‘last’.

``` r
# Introduce argument mult
dti[c('setosa', 'versicolor'), mult = 'first']
```

    ##    Sepal.Length Sepal.Width Petal.Length Petal.Width    Species
    ## 1:          5.1         3.5          1.4         0.2     setosa
    ## 2:          7.0         3.2          4.7         1.4 versicolor

``` r
dti[c('setosa', 'versicolor'), mult = 'last']
```

    ##    Sepal.Length Sepal.Width Petal.Length Petal.Width    Species
    ## 1:          5.0         3.3          1.4         0.2     setosa
    ## 2:          5.7         2.8          4.1         1.3 versicolor

You could also specify certain rows (`i`). To display each i, you need to specify `by = .EACHI`.

> When i is a data.table, DT[i,j,by=.EACHI] evaluates j for the groups in ‘DT’ that each row in i joins to. That is, you can join (in i) and aggregate (in j) simultaneously. We call this **grouping by each i**.

``` r
dti[c('setosa', 'versicolor'), .SD[c(1, .N)], by = .EACHI]
```

    ##       Species Sepal.Length Sepal.Width Petal.Length Petal.Width
    ## 1:     setosa          5.1         3.5          1.4         0.2
    ## 2:     setosa          5.0         3.3          1.4         0.2
    ## 3: versicolor          7.0         3.2          4.7         1.4
    ## 4: versicolor          5.7         2.8          4.1         1.3

``` r
dti[c('setosa', 'versicolor'), {print(.SD[3:5]); .SD[c(1, .N)]}, by = .EACHI]
```

    ##    Sepal.Length Sepal.Width Petal.Length Petal.Width
    ## 1:          4.7         3.2          1.3         0.2
    ## 2:          4.6         3.1          1.5         0.2
    ## 3:          5.0         3.6          1.4         0.2
    ##    Sepal.Length Sepal.Width Petal.Length Petal.Width
    ## 1:          6.9         3.1          4.9         1.5
    ## 2:          5.5         2.3          4.0         1.3
    ## 3:          6.5         2.8          4.6         1.5

    ##       Species Sepal.Length Sepal.Width Petal.Length Petal.Width
    ## 1:     setosa          5.1         3.5          1.4         0.2
    ## 2:     setosa          5.0         3.3          1.4         0.2
    ## 3: versicolor          7.0         3.2          4.7         1.4
    ## 4: versicolor          5.7         2.8          4.1         1.3

#### Rolling joins

Let’s check the records of brand ‘Merc’ and 7 cylinders.

``` r
dtm[.('Merc', 5)]
```

    ##    model mpg cyl disp hp drat wt qsec vs am gear carb brand
    ## 1:    NA  NA   5   NA NA   NA NA   NA NA NA   NA   NA  Merc

There is no such record. How to return records without `NA`? Use argument `roll`.

`roll = TRUE`: i’s row matches to all but the last x join column, and its value in the last i join column falls in a gap (including after the last observation in x for that group), then the *prevailing* value in x is rolled **forward**.

``` r
dtm[.(c('Merc', 'Toyota'), 5), roll = TRUE]
```

    ##             model  mpg cyl  disp hp drat    wt qsec vs am gear carb  brand
    ## 1:       Merc 230 22.8   5 140.8 95 3.92 3.150 22.9  1  0    4    2   Merc
    ## 2: Toyota Corolla 33.9   5  71.1 65 4.22 1.835 19.9  1  1    4    1 Toyota

`roll = 'nearest'`: the nearest value is joined to.

``` r
# compare the commands below
dtm[.(c('Merc', 'Toyota'), 3), roll = TRUE]
```

    ##    model mpg cyl disp hp drat wt qsec vs am gear carb  brand
    ## 1:    NA  NA   3   NA NA   NA NA   NA NA NA   NA   NA   Merc
    ## 2:    NA  NA   3   NA NA   NA NA   NA NA NA   NA   NA Toyota

``` r
dtm[.(c('Merc', 'Toyota'), 3), roll = 'nearest']
```

    ##            model  mpg cyl  disp hp drat    wt  qsec vs am gear carb  brand
    ## 1:     Merc 240D 24.4   3 146.7 62 3.69 3.190 20.00  1  0    4    2   Merc
    ## 2: Toyota Corona 21.5   3 120.1 97 3.70 2.465 20.01  1  0    3    1 Toyota

`roll` can also be a finite number.

``` r
# compare the commands below
dtm[.('Merc', 6:10)]
```

    ##          model  mpg cyl  disp  hp drat   wt qsec vs am gear carb brand
    ## 1:    Merc 280 19.2   6 167.6 123 3.92 3.44 18.3  1  0    4    4  Merc
    ## 2:   Merc 280C 17.8   6 167.6 123 3.92 3.44 18.9  1  0    4    4  Merc
    ## 3:          NA   NA   7    NA  NA   NA   NA   NA NA NA   NA   NA  Merc
    ## 4:  Merc 450SE 16.4   8 275.8 180 3.07 4.07 17.4  0  0    3    3  Merc
    ## 5:  Merc 450SL 17.3   8 275.8 180 3.07 3.73 17.6  0  0    3    3  Merc
    ## 6: Merc 450SLC 15.2   8 275.8 180 3.07 3.78 18.0  0  0    3    3  Merc
    ## 7:          NA   NA   9    NA  NA   NA   NA   NA NA NA   NA   NA  Merc
    ## 8:          NA   NA  10    NA  NA   NA   NA   NA NA NA   NA   NA  Merc

``` r
dtm[.('Merc', 6:10), roll = 1]
```

    ##          model  mpg cyl  disp  hp drat   wt qsec vs am gear carb brand
    ## 1:    Merc 280 19.2   6 167.6 123 3.92 3.44 18.3  1  0    4    4  Merc
    ## 2:   Merc 280C 17.8   6 167.6 123 3.92 3.44 18.9  1  0    4    4  Merc
    ## 3:   Merc 280C 17.8   7 167.6 123 3.92 3.44 18.9  1  0    4    4  Merc
    ## 4:  Merc 450SE 16.4   8 275.8 180 3.07 4.07 17.4  0  0    3    3  Merc
    ## 5:  Merc 450SL 17.3   8 275.8 180 3.07 3.73 17.6  0  0    3    3  Merc
    ## 6: Merc 450SLC 15.2   8 275.8 180 3.07 3.78 18.0  0  0    3    3  Merc
    ## 7: Merc 450SLC 15.2   9 275.8 180 3.07 3.78 18.0  0  0    3    3  Merc
    ## 8:          NA   NA  10    NA  NA   NA   NA   NA NA NA   NA   NA  Merc

There is no Mercedez with 7, 9 and 10 cylinders, but we allow it to roll for distance of 1. So 7 and 9 cylinders are both joined but 10 because it’s more than 1 from the prevailing observation.

Now we take a look on argument `rollends`. This argument is actually a vector of two logical values (a single logical is recycled).

``` r
# compare the commands below
setkey(dtm, am, carb)
dtm[.(1, (-1):8)]
```

    ##              model  mpg cyl  disp  hp drat    wt  qsec vs am gear carb    brand
    ##  1:             NA   NA  NA    NA  NA   NA    NA    NA NA  1   NA   -1       NA
    ##  2:             NA   NA  NA    NA  NA   NA    NA    NA NA  1   NA    0       NA
    ##  3:     Datsun 710 22.8   4 108.0  93 3.85 2.320 18.61  1  1    4    1   Datsun
    ##  4:       Fiat 128 32.4   4  78.7  66 4.08 2.200 19.47  1  1    4    1     Fiat
    ##  5:      Fiat X1-9 27.3   4  79.0  66 4.08 1.935 18.90  1  1    4    1     Fiat
    ##  6: Toyota Corolla 33.9   4  71.1  65 4.22 1.835 19.90  1  1    4    1   Toyota
    ##  7:    Honda Civic 30.4   4  75.7  52 4.93 1.615 18.52  1  1    4    2    Honda
    ##  8:   Lotus Europa 30.4   4  95.1 113 3.77 1.513 16.90  1  1    5    2    Lotus
    ##  9:  Porsche 914-2 26.0   4 120.3  91 4.43 2.140 16.70  0  1    5    2  Porsche
    ## 10:     Volvo 142E 21.4   4 121.0 109 4.11 2.780 18.60  1  1    4    2    Volvo
    ## 11:             NA   NA  NA    NA  NA   NA    NA    NA NA  1   NA    3       NA
    ## 12: Ford Pantera L 15.8   8 351.0 264 4.22 3.170 14.50  0  1    5    4     Ford
    ## 13:      Mazda RX4 21.0   6 160.0 110 3.90 2.620 16.46  0  1    4    4    Mazda
    ## 14:  Mazda RX4 Wag 21.0   6 160.0 110 3.90 2.875 17.02  0  1    4    4    Mazda
    ## 15:             NA   NA  NA    NA  NA   NA    NA    NA NA  1   NA    5       NA
    ## 16:   Ferrari Dino 19.7   6 145.0 175 3.62 2.770 15.50  0  1    5    6  Ferrari
    ## 17:             NA   NA  NA    NA  NA   NA    NA    NA NA  1   NA    7       NA
    ## 18:  Maserati Bora 15.0   8 301.0 335 3.54 3.570 14.60  0  1    5    8 Maserati


``` r
dtm[.(1, (-1):8), roll = TRUE]
```

    ##              model  mpg cyl  disp  hp drat    wt  qsec vs am gear carb    brand
    ##  1:             NA   NA  NA    NA  NA   NA    NA    NA NA  1   NA   -1       NA
    ##  2:             NA   NA  NA    NA  NA   NA    NA    NA NA  1   NA    0       NA
    ##  3:     Datsun 710 22.8   4 108.0  93 3.85 2.320 18.61  1  1    4    1   Datsun
    ##  4:       Fiat 128 32.4   4  78.7  66 4.08 2.200 19.47  1  1    4    1     Fiat
    ##  5:      Fiat X1-9 27.3   4  79.0  66 4.08 1.935 18.90  1  1    4    1     Fiat
    ##  6: Toyota Corolla 33.9   4  71.1  65 4.22 1.835 19.90  1  1    4    1   Toyota
    ##  7:    Honda Civic 30.4   4  75.7  52 4.93 1.615 18.52  1  1    4    2    Honda
    ##  8:   Lotus Europa 30.4   4  95.1 113 3.77 1.513 16.90  1  1    5    2    Lotus
    ##  9:  Porsche 914-2 26.0   4 120.3  91 4.43 2.140 16.70  0  1    5    2  Porsche
    ## 10:     Volvo 142E 21.4   4 121.0 109 4.11 2.780 18.60  1  1    4    2    Volvo
    ## 11:     Volvo 142E 21.4   4 121.0 109 4.11 2.780 18.60  1  1    4    3    Volvo
    ## 12: Ford Pantera L 15.8   8 351.0 264 4.22 3.170 14.50  0  1    5    4     Ford
    ## 13:      Mazda RX4 21.0   6 160.0 110 3.90 2.620 16.46  0  1    4    4    Mazda
    ## 14:  Mazda RX4 Wag 21.0   6 160.0 110 3.90 2.875 17.02  0  1    4    4    Mazda
    ## 15:  Mazda RX4 Wag 21.0   6 160.0 110 3.90 2.875 17.02  0  1    4    5    Mazda
    ## 16:   Ferrari Dino 19.7   6 145.0 175 3.62 2.770 15.50  0  1    5    6  Ferrari
    ## 17:   Ferrari Dino 19.7   6 145.0 175 3.62 2.770 15.50  0  1    5    7  Ferrari
    ## 18:  Maserati Bora 15.0   8 301.0 335 3.54 3.570 14.60  0  1    5    8 Maserati

``` r
dtm[.(1, (-1):8), roll = TRUE, rollends = TRUE]
```

    ##              model  mpg cyl  disp  hp drat    wt  qsec vs am gear carb    brand
    ##  1:     Datsun 710 22.8   4 108.0  93 3.85 2.320 18.61  1  1    4   -1   Datsun
    ##  2:     Datsun 710 22.8   4 108.0  93 3.85 2.320 18.61  1  1    4    0   Datsun
    ##  3:     Datsun 710 22.8   4 108.0  93 3.85 2.320 18.61  1  1    4    1   Datsun
    ##  4:       Fiat 128 32.4   4  78.7  66 4.08 2.200 19.47  1  1    4    1     Fiat
    ##  5:      Fiat X1-9 27.3   4  79.0  66 4.08 1.935 18.90  1  1    4    1     Fiat
    ##  6: Toyota Corolla 33.9   4  71.1  65 4.22 1.835 19.90  1  1    4    1   Toyota
    ##  7:    Honda Civic 30.4   4  75.7  52 4.93 1.615 18.52  1  1    4    2    Honda
    ##  8:   Lotus Europa 30.4   4  95.1 113 3.77 1.513 16.90  1  1    5    2    Lotus
    ##  9:  Porsche 914-2 26.0   4 120.3  91 4.43 2.140 16.70  0  1    5    2  Porsche
    ## 10:     Volvo 142E 21.4   4 121.0 109 4.11 2.780 18.60  1  1    4    2    Volvo
    ## 11:     Volvo 142E 21.4   4 121.0 109 4.11 2.780 18.60  1  1    4    3    Volvo
    ## 12: Ford Pantera L 15.8   8 351.0 264 4.22 3.170 14.50  0  1    5    4     Ford
    ## 13:      Mazda RX4 21.0   6 160.0 110 3.90 2.620 16.46  0  1    4    4    Mazda
    ## 14:  Mazda RX4 Wag 21.0   6 160.0 110 3.90 2.875 17.02  0  1    4    4    Mazda
    ## 15:  Mazda RX4 Wag 21.0   6 160.0 110 3.90 2.875 17.02  0  1    4    5    Mazda
    ## 16:   Ferrari Dino 19.7   6 145.0 175 3.62 2.770 15.50  0  1    5    6  Ferrari
    ## 17:   Ferrari Dino 19.7   6 145.0 175 3.62 2.770 15.50  0  1    5    7  Ferrari
    ## 18:  Maserati Bora 15.0   8 301.0 335 3.54 3.570 14.60  0  1    5    8 Maserati

This argument allows `NA` records to look forward to the nearest‘matching’ records

Other data.table functions
--------------------------

### fread() fast read

Borrow the example from `data.table` document:

``` r
n=1e6
DT = data.table( a=sample(1:1000,n,replace=TRUE),
                 b=sample(1:1000,n,replace=TRUE),
                 c=rnorm(n),
                 d=sample(c("foo","bar","baz","qux","quux"),n,replace=TRUE),
                 e=rnorm(n),
                 f=sample(1:1000,n,replace=TRUE) )
DT[2,b:=NA_integer_]
DT[4,c:=NA_real_]
DT[3,d:=NA_character_]
DT[5,d:=""]
DT[2,e:=+Inf]
DT[3,e:=-Inf]

write.table(DT,"test.csv",sep=",",row.names=FALSE,quote=FALSE)
cat("File size (MB):", round(file.info("test.csv")$size/1024^2),"\n")
```

    ## File size (MB): 50

``` r
system.time(DF1 <-read.csv("test.csv",stringsAsFactors=FALSE))
```

    ##    user  system elapsed 
    ##  21.556   0.196  21.760

``` r
# first time in fresh R session

system.time(DF1 <- read.csv("test.csv",stringsAsFactors=FALSE))
```

    ##    user  system elapsed 
    ##  11.728   0.144  11.873

``` r
# immediate repeat is faster, varies

system.time(DF2 <- read.table("test.csv",header=TRUE,sep=",",quote="",
					stringsAsFactors=FALSE,comment.char="",nrows=n,
					colClasses=c("integer","integer","numeric",
					             "character","numeric","integer")))
```

    ##    user  system elapsed 
    ##   7.324   0.040   7.369

``` r
require(data.table)
system.time(DT <- fread("test.csv"))
```

    ##    user  system elapsed 
    ##   1.988   0.008   1.993

This is a powerful file reading function.

### others

Enter the command `library(help = data.table)` and you could see many other small functions, like `frank`, `foverlaps`, etc. All of them are aiming at speeding up the available functions with similar effect in `base` or other packages. You could take a look on them when necessary.

* * * * *

Reference
---------

-   [Data Analysis in R, the data.table Way](https://www.datacamp.com/courses/data-table-data-manipulation-r-tutorial).

