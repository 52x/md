title: data.table Way - Learning Note Part 1
date: 2015-09-23 21:05:42
categories: Codage|编程
tags: [R, data.table]
---
This is a learning note for the course [Data Analysis in R, the data.table Way](https://www.datacamp.com/courses/data-table-data-manipulation-r-tutorial).
This course introduces the package ‘data.table’ which is a very powerful tool for data manipulation. I borrow two built-in data frames: `mtcars` and `iris` throughout the entire article.

<!-- more -->

``` r
library('data.table')
```

Startup
-------

First time using `data.table` package, we can compare it to `data.frame`. The fundemental functionality between the 2 are quite similar.

### Create a data.table

To create a `data.table`, we use the same method as we do for creating a `data.frame`.

``` r
dt1 <- data.table(A = letters[1:6], B = 1:2, C = 8L)
df1 <- data.frame(A = letters[1:6], B = 1:2, C = 8L)
dt1
```

    ##    A B C
    ## 1: a 1 8
    ## 2: b 2 8
    ## 3: c 1 8
    ## 4: d 2 8
    ## 5: e 1 8
    ## 6: f 2 8

``` r
df1
```

    ##   A B C
    ## 1 a 1 8
    ## 2 b 2 8
    ## 3 c 1 8
    ## 4 d 2 8
    ## 5 e 1 8
    ## 6 f 2 8

It’s obvious the column `B` and `C` are created by recycling.

And that’s check the class of `dt1`:

``` r
class(dt1)
```

    ## [1] "data.table" "data.frame"

The `data.table` package inherits the class `data.frame` from `base` package and develops a sub-class `data.table`.

### Convert a data.frame or a matrix

``` r
dt2 <- as.data.table(df1)                     # convert a data.frame to data.table
dt3 <- as.data.table(matrix(1:9, ncol= 3))    # convert a matrix to data.table
class(dt2)
```

    ## [1] "data.table" "data.frame"

``` r
class(dt3)
```

    ## [1] "data.table" "data.frame"

### Take a look

Head, tail, fire at will.

``` r
# now we start using mtcars & iris
dt_mtcars <- as.data.table(mtcars)
dt_iris <- as.data.table(iris)
head(mtcars)
```

    ##                    mpg cyl disp  hp drat    wt  qsec vs am gear carb
    ## Mazda RX4         21.0   6  160 110 3.90 2.620 16.46  0  1    4    4
    ## Mazda RX4 Wag     21.0   6  160 110 3.90 2.875 17.02  0  1    4    4
    ## Datsun 710        22.8   4  108  93 3.85 2.320 18.61  1  1    4    1
    ## Hornet 4 Drive    21.4   6  258 110 3.08 3.215 19.44  1  0    3    1
    ## Hornet Sportabout 18.7   8  360 175 3.15 3.440 17.02  0  0    3    2
    ## Valiant           18.1   6  225 105 2.76 3.460 20.22  1  0    3    1

``` r
tail(iris)
```

    ##     Sepal.Length Sepal.Width Petal.Length Petal.Width   Species
    ## 145          6.7         3.3          5.7         2.5 virginica
    ## 146          6.7         3.0          5.2         2.3 virginica
    ## 147          6.3         2.5          5.0         1.9 virginica
    ## 148          6.5         3.0          5.2         2.0 virginica
    ## 149          6.2         3.4          5.4         2.3 virginica
    ## 150          5.9         3.0          5.1         1.8 virginica

  ----------------------------------------------------------------------------
  **BEFORE MOVING TO THE NEXT PART**
  Several things need to be mentioned:
  1. `dt[i, j, by]` - standard subsetting clause, `dt` is a `data.table`;
  2. `.()` - an abbreviated method representing `list`;
  3. `()` - an abbreviated method representing `vector`;
  4. `:=` - assignment function
  5. `data.table` internal variable:
  - .N - number of rows
  - .SD - **s**ubset of **d**ata.table, excluding cols used in by (or keyby)
  - .SDcols - column index
  ----------------------------------------------------------------------------

Subsetting
----------

Prepare the example `data.table`:

``` r
dtm <- copy(dt_mtcars)
dti <- copy(dt_iris)
```

Remember that subsetting `data.table` is quite than subsetting `data.frame`. *`dtm` is not Document-Term Matrix LOL*

### Subset the rows

#### Subset normally

Return the 3rd row of dataset mtcars. Furthermore, several rows…

``` r
dtm[3]               # mtcars[3, ]
```

    ##     mpg cyl disp hp drat   wt  qsec vs am gear carb
    ## 1: 22.8   4  108 93 3.85 2.32 18.61  1  1    4    1

``` r
dtm[2:3]             # mtcars[2:3, ]
```

    ##     mpg cyl disp  hp drat    wt  qsec vs am gear carb
    ## 1: 21.0   6  160 110 3.90 2.875 17.02  0  1    4    4
    ## 2: 22.8   4  108  93 3.85 2.320 18.61  1  1    4    1

``` r
dtm[c(1,4,5)]        # mtcars[c(1,4,5), ]
```

    ##     mpg cyl disp  hp drat    wt  qsec vs am gear carb
    ## 1: 21.0   6  160 110 3.90 2.620 16.46  0  1    4    4
    ## 2: 21.4   6  258 110 3.08 3.215 19.44  1  0    3    1
    ## 3: 18.7   8  360 175 3.15 3.440 17.02  0  0    3    2

Given `i` the first argument by default, we don’t need to specify any arg. name and we don’t need the comma as `data.frame` does. **Don’t forget the 3 commands applied on `data.frame` are used for subsetting columns.**

#### Subset by internal variable .N

`.N` is an internal variable mostly used in `i`. Return the penultimate row of `dtm`.

``` r
dtm[.N-1]
```

    ##    mpg cyl disp  hp drat   wt qsec vs am gear carb
    ## 1:  15   8  301 335 3.54 3.57 14.6  0  1    5    8

To be noticed that when you convert a `data.frame` to a `data.table`, you lose the row names.

### Subset the columns

`j` should be either a column name or list of column names wrapped by `.()`. In this case, you need to skip the first argument `i`.

``` r
head(dtm[, .(mpg, cyl)])
```

    ##     mpg cyl
    ## 1: 21.0   6
    ## 2: 21.0   6
    ## 3: 22.8   4
    ## 4: 21.4   6
    ## 5: 18.7   8
    ## 6: 18.1   6

Note that the subsetting result is a `data.table`. What if you want to drop the structure, keep only a vector?

``` r
dtm[1:10, mpg]
```

    ##  [1] 21.0 21.0 22.8 21.4 18.7 18.1 14.3 24.4 22.8 19.2

``` r
# to compare with:
# dtm[1:10, .(mpg)]
```

Imagine what’s the result of `dtm[, c(mpg, cyl)]`?

### Subset conditionally

Replay the conditional subsetting in `data.frame`:

``` r
head(iris[iris$Species == 'setosa',])
```

    ##   Sepal.Length Sepal.Width Petal.Length Petal.Width Species
    ## 1          5.1         3.5          1.4         0.2  setosa
    ## 2          4.9         3.0          1.4         0.2  setosa
    ## 3          4.7         3.2          1.3         0.2  setosa
    ## 4          4.6         3.1          1.5         0.2  setosa
    ## 5          5.0         3.6          1.4         0.2  setosa
    ## 6          5.4         3.9          1.7         0.4  setosa

How do we do in `data.table`? Return the first 6 rows of Species ‘setosa’:

``` r
head(dti[Species == 'setosa'])
```

    ##    Sepal.Length Sepal.Width Petal.Length Petal.Width Species
    ## 1:          5.1         3.5          1.4         0.2  setosa
    ## 2:          4.9         3.0          1.4         0.2  setosa
    ## 3:          4.7         3.2          1.3         0.2  setosa
    ## 4:          4.6         3.1          1.5         0.2  setosa
    ## 5:          5.0         3.6          1.4         0.2  setosa
    ## 6:          5.4         3.9          1.7         0.4  setosa

Select all the rows where species is either ‘setosa’ and ‘virginica’:

``` r
temp <- dti[Species %in% c('setosa', 'virginica')]
table(temp$Species)
```

    ## 
    ##     setosa versicolor  virginica 
    ##         50          0         50

Select the rows where the sepal area is greater than 2O:

``` r
head(dti[Sepal.Width*Sepal.Length > 20])
```

    ##    Sepal.Length Sepal.Width Petal.Length Petal.Width Species
    ## 1:          5.4         3.9          1.7         0.4  setosa
    ## 2:          5.8         4.0          1.2         0.2  setosa
    ## 3:          5.7         4.4          1.5         0.4  setosa
    ## 4:          5.4         3.9          1.3         0.4  setosa
    ## 5:          5.7         3.8          1.7         0.3  setosa
    ## 6:          5.2         4.1          1.5         0.1  setosa

### Automatic indexing

There is a rule that “if `i` is a single variable name, it is evaluated in calling scope, otherwise inside DT’s scope”. This is a very important rule if you want to conceptually understand what is going on when using column names in `i`. Only single columns on the left of operators benefit from automatic indexing.

``` r
# Add a boolean column
dti[, IsLarge := Sepal.Width*Sepal.Length > 20]

# Select the rows where IsLarge is TRUE
# There are 2 methods
head(dti[IsLarge == TRUE])
```

    ##    Sepal.Length Sepal.Width Petal.Length Petal.Width Species IsLarge
    ## 1:          5.4         3.9          1.7         0.4  setosa    TRUE
    ## 2:          5.8         4.0          1.2         0.2  setosa    TRUE
    ## 3:          5.7         4.4          1.5         0.4  setosa    TRUE
    ## 4:          5.4         3.9          1.3         0.4  setosa    TRUE
    ## 5:          5.7         3.8          1.7         0.3  setosa    TRUE
    ## 6:          5.2         4.1          1.5         0.1  setosa    TRUE

``` r
head(dti[(IsLarge)])
```

    ##    Sepal.Length Sepal.Width Petal.Length Petal.Width Species IsLarge
    ## 1:          5.4         3.9          1.7         0.4  setosa    TRUE
    ## 2:          5.8         4.0          1.2         0.2  setosa    TRUE
    ## 3:          5.7         4.4          1.5         0.4  setosa    TRUE
    ## 4:          5.4         3.9          1.3         0.4  setosa    TRUE
    ## 5:          5.7         3.8          1.7         0.3  setosa    TRUE
    ## 6:          5.2         4.1          1.5         0.1  setosa    TRUE

Manipulating data.table
-----------------------

### Add

As mentioned before, converting a `data.frame` to a `data.table` discards its row names. Here, we can add the vehicle model names to the `dtm`.

``` r
dtm <- data.table(model = rownames(mtcars), dtm)
head(dtm)
```

    ##                model  mpg cyl disp  hp drat    wt  qsec vs am gear carb
    ## 1:         Mazda RX4 21.0   6  160 110 3.90 2.620 16.46  0  1    4    4
    ## 2:     Mazda RX4 Wag 21.0   6  160 110 3.90 2.875 17.02  0  1    4    4
    ## 3:        Datsun 710 22.8   4  108  93 3.85 2.320 18.61  1  1    4    1
    ## 4:    Hornet 4 Drive 21.4   6  258 110 3.08 3.215 19.44  1  0    3    1
    ## 5: Hornet Sportabout 18.7   8  360 175 3.15 3.440 17.02  0  0    3    2
    ## 6:           Valiant 18.1   6  225 105 2.76 3.460 20.22  1  0    3    1

We introduce the assignment function here: `:=`, to realize the step above in a simple way.

``` r
dtm <- dtm[,  model := rownames(mtcars)]
```

In this case, the new column is added as the last column.

### Select

To check the vehicle model and their fuel efficiency (return only the first 10 rows):

``` r
dtm[1:10, .(model, mpg)]
```

    ##                 model  mpg
    ##  1:         Mazda RX4 21.0
    ##  2:     Mazda RX4 Wag 21.0
    ##  3:        Datsun 710 22.8
    ##  4:    Hornet 4 Drive 21.4
    ##  5: Hornet Sportabout 18.7
    ##  6:           Valiant 18.1
    ##  7:        Duster 360 14.3
    ##  8:         Merc 240D 24.4
    ##  9:          Merc 230 22.8
    ## 10:          Merc 280 19.2

Also, if I want to check the `Sepal.Length` and `Petal.Length` in one column, `data.table` can do that by recycling.

``` r
temp <- dti[1:5]
temp
```

    ##    Sepal.Length Sepal.Width Petal.Length Petal.Width Species IsLarge
    ## 1:          5.1         3.5          1.4         0.2  setosa   FALSE
    ## 2:          4.9         3.0          1.4         0.2  setosa   FALSE
    ## 3:          4.7         3.2          1.3         0.2  setosa   FALSE
    ## 4:          4.6         3.1          1.5         0.2  setosa   FALSE
    ## 5:          5.0         3.6          1.4         0.2  setosa   FALSE

``` r
# compare the command below with temp
temp[, .(Species, Length = c(Sepal.Length, Petal.Length))]
```

    ##     Species Length
    ##  1:  setosa    5.1
    ##  2:  setosa    4.9
    ##  3:  setosa    4.7
    ##  4:  setosa    4.6
    ##  5:  setosa    5.0
    ##  6:  setosa    1.4
    ##  7:  setosa    1.4
    ##  8:  setosa    1.3
    ##  9:  setosa    1.5
    ## 10:  setosa    1.4

### Remove

To drop a column, we can simply assign `NULL` to this column with `:=`.

``` r
head(temp)
```

    ##    Sepal.Length Sepal.Width Petal.Length Petal.Width Species IsLarge
    ## 1:          5.1         3.5          1.4         0.2  setosa   FALSE
    ## 2:          4.9         3.0          1.4         0.2  setosa   FALSE
    ## 3:          4.7         3.2          1.3         0.2  setosa   FALSE
    ## 4:          4.6         3.1          1.5         0.2  setosa   FALSE
    ## 5:          5.0         3.6          1.4         0.2  setosa   FALSE

``` r
# remove the 1st column
temp[, Sepal.Length:=NULL]
head(temp)
```

    ##    Sepal.Width Petal.Length Petal.Width Species IsLarge
    ## 1:         3.5          1.4         0.2  setosa   FALSE
    ## 2:         3.0          1.4         0.2  setosa   FALSE
    ## 3:         3.2          1.3         0.2  setosa   FALSE
    ## 4:         3.1          1.5         0.2  setosa   FALSE
    ## 5:         3.6          1.4         0.2  setosa   FALSE

Noticed that **the removing action is applied automatically to the `data.table` itself** without assigning to `temp` again.

There is also an equivalent way to remove certain column by specifying the column number.

``` r
# remove the 1st column again
temp[, 1:=NULL]
head(temp)
```

    ##    Petal.Length Petal.Width Species IsLarge
    ## 1:          1.4         0.2  setosa   FALSE
    ## 2:          1.4         0.2  setosa   FALSE
    ## 3:          1.3         0.2  setosa   FALSE
    ## 4:          1.5         0.2  setosa   FALSE
    ## 5:          1.4         0.2  setosa   FALSE

To remove multiple columns, the column names or indexes need to be fed as a vector.

``` r
temp[, c('Petal.Length', 'Petal.Width'):=NULL]
temp
```

    ##    Species IsLarge
    ## 1:  setosa   FALSE
    ## 2:  setosa   FALSE
    ## 3:  setosa   FALSE
    ## 4:  setosa   FALSE
    ## 5:  setosa   FALSE

or

``` r
# restore temp
temp <- dti[1:5]
temp[, (1:2):=NULL] # or temp[, 1:2:=NULL] 

tbr <- c('Petal.Length', 'Petal.Width')
temp[, (tbr):=NULL]
```

or like this:

``` r
# restore temp
temp <- dti[1:5]
temp[, (1:2):=NULL] # or temp[, 1:2:=NULL] 

temp[, grep('^Petal', names(temp)):=NULL]
```

### Update

Borrowing the same logic, we use `:=` to assign new values to existing colums. In column `Species`, we keep only the first 3 letters:

``` r
temp <- dti[1:5]
temp[, Species:=substr(Species, 1, 3)]
temp
```

    ##    Sepal.Length Sepal.Width Petal.Length Petal.Width Species IsLarge
    ## 1:          5.1         3.5          1.4         0.2     set   FALSE
    ## 2:          4.9         3.0          1.4         0.2     set   FALSE
    ## 3:          4.7         3.2          1.3         0.2     set   FALSE
    ## 4:          4.6         3.1          1.5         0.2     set   FALSE
    ## 5:          5.0         3.6          1.4         0.2     set   FALSE

How do we update multiple columns? We can try to add 1 to column `vs` and `am`, using `:=` as a function name.

``` r
temp <- dtm[1:32]
temp[, `:=`(vs = vs + 1, am = am + 1)]
head(temp)
```

    ##                model  mpg cyl disp  hp drat    wt  qsec vs am gear carb
    ## 1:         Mazda RX4 21.0   6  160 110 3.90 2.620 16.46  1  2    4    4
    ## 2:     Mazda RX4 Wag 21.0   6  160 110 3.90 2.875 17.02  1  2    4    4
    ## 3:        Datsun 710 22.8   4  108  93 3.85 2.320 18.61  2  2    4    1
    ## 4:    Hornet 4 Drive 21.4   6  258 110 3.08 3.215 19.44  2  1    3    1
    ## 5: Hornet Sportabout 18.7   8  360 175 3.15 3.440 17.02  1  1    3    2
    ## 6:           Valiant 18.1   6  225 105 2.76 3.460 20.22  2  1    3    1

### Summarise

First of all, let’s take a look on the simplest way for summarising. To check the efficiency of unit cylinder per model, we can select the 1st column of updated `dtm` and create a new variable using `mpg` divided by `cyl` (return only the first 10 rows).

``` r
dtm[1:10, .(model, efcy_cly = round(mpg/cyl, 3))]
```

    ##                 model efcy_cly
    ##  1:         Mazda RX4    3.500
    ##  2:     Mazda RX4 Wag    3.500
    ##  3:        Datsun 710    5.700
    ##  4:    Hornet 4 Drive    3.567
    ##  5: Hornet Sportabout    2.337
    ##  6:           Valiant    3.017
    ##  7:        Duster 360    1.788
    ##  8:         Merc 240D    6.100
    ##  9:          Merc 230    5.700
    ## 10:          Merc 280    3.200

To keep the column names, it’s always necessary to wrap the columns selected by `.()`.

#### Introduce basic by

`by` is the third argument by default used for picking up the group. Now we check the average fuel efficiency (`mpg`) for number of cylinders that a vehicle has.

``` r
dtm[, mean(mpg), by = cyl]
```

    ##    cyl       V1
    ## 1:   6 19.74286
    ## 2:   4 26.66364
    ## 3:   8 15.10000

The second column doesn’t have a colum name, try the command below:

``` r
dtm[, .(avg_mpg = mean(mpg)), by = cyl]
```

    ##    cyl  avg_mpg
    ## 1:   6 19.74286
    ## 2:   4 26.66364
    ## 3:   8 15.10000

#### Apply function on by

It’s feasible to apply function on `by`. For example I want to check the mean value of `qsec` - the time cost to reach 1/4 mile - grouped by the first 3 letters of vehicle models.

``` r
head(dtm[, .(avg_qsec = mean(qsec)), by = substr(model, 1, 3)])
```

    ##    substr avg_qsec
    ## 1:    Maz 16.74000
    ## 2:    Dat 18.61000
    ## 3:    Hor 18.23000
    ## 4:    Val 20.22000
    ## 5:    Dus 15.84000
    ## 6:    Mer 19.01429

What if you want to get the number of each group under certain grouping rule?

``` r
dti[, .N, by = .(area_group = 10*round(Sepal.Length*Sepal.Width/10))]
```

    ##    area_group   N
    ## 1:         20 117
    ## 2:         10  29
    ## 3:         30   4

if you are familiar with `dplyr` package, you can try to write a command using `n()`.

#### Return multiple numbers in j

So far, we tried returning single numbers in each group (in `j`). `data.table` also allows returning multiple values.

Check the fuel top 2 records of `mpg` for each group concerning `cyl` and `am` (transmission method).

``` r
dtm[, .(mpg = head(mpg, 2)), by = .(cyl, am)]
```

    ##     cyl am  mpg
    ##  1:   6  1 21.0
    ##  2:   6  1 21.0
    ##  3:   4  1 22.8
    ##  4:   4  1 32.4
    ##  5:   6  0 21.4
    ##  6:   6  0 18.1
    ##  7:   8  0 18.7
    ##  8:   8  0 14.3
    ##  9:   4  0 24.4
    ## 10:   4  0 22.8
    ## 11:   8  1 15.8
    ## 12:   8  1 15.0

Calculate the cumulative weight of vehicle grouped by `cyl` and `gear`.

``` r
dtm[, .(cumulative_su = cumsum(wt)), by = .(cyl, gear)]
```

    ##     cyl gear cumulative_su
    ##  1:   6    4         2.620
    ##  2:   6    4         5.495
    ##  3:   6    4         8.935
    ##  4:   6    4        12.375
    ##  5:   4    4         2.320
    ##  6:   4    4         5.510
    ##  7:   4    4         8.660
    ##  8:   4    4        10.860
    ##  9:   4    4        12.475
    ## 10:   4    4        14.310
    ## 11:   4    4        16.245
    ## 12:   4    4        19.025
    ## 13:   6    3         3.215
    ## 14:   6    3         6.675
    ## 15:   8    3         3.440
    ## 16:   8    3         7.010
    ## 17:   8    3        11.080
    ## 18:   8    3        14.810
    ## 19:   8    3        18.590
    ## 20:   8    3        23.840
    ## 21:   8    3        29.264
    ## 22:   8    3        34.609
    ## 23:   8    3        38.129
    ## 24:   8    3        41.564
    ## 25:   8    3        45.404
    ## 26:   8    3        49.249
    ## 27:   4    3         2.465
    ## 28:   4    5         2.140
    ## 29:   4    5         3.653
    ## 30:   8    5         3.170
    ## 31:   8    5         6.740
    ## 32:   6    5         2.770
    ##     cyl gear cumulative_su

#### Chaining

List the `mpg` in descending order grouped by `cyl` and `am`, then show only the first 2 fuel-efficient records in each group. This cannot be done in one step.

``` r
temp <- dtm[, .(model, mpg), by = .(cyl, am)]
temp <- temp[order(-mpg)]
temp[, .(model = head(model, 2), mpg = head(mpg, 2)), by = .(cyl, am)]
```

    ##     cyl am             model  mpg
    ##  1:   4  1    Toyota Corolla 33.9
    ##  2:   4  1          Fiat 128 32.4
    ##  3:   4  0         Merc 240D 24.4
    ##  4:   4  0          Merc 230 22.8
    ##  5:   6  0    Hornet 4 Drive 21.4
    ##  6:   6  0          Merc 280 19.2
    ##  7:   6  1         Mazda RX4 21.0
    ##  8:   6  1     Mazda RX4 Wag 21.0
    ##  9:   8  0  Pontiac Firebird 19.2
    ## 10:   8  0 Hornet Sportabout 18.7
    ## 11:   8  1    Ford Pantera L 15.8
    ## 12:   8  1     Maserati Bora 15.0

We can simplify the process by chaining:

``` r
dtm[, .(model, mpg), by = .(cyl, am)][order(-mpg)][, .(model = head(model, 2), mpg = head(mpg, 2)), by = .(cyl, am)]
```

    ##     cyl am             model  mpg
    ##  1:   4  1    Toyota Corolla 33.9
    ##  2:   4  1          Fiat 128 32.4
    ##  3:   4  0         Merc 240D 24.4
    ##  4:   4  0          Merc 230 22.8
    ##  5:   6  0    Hornet 4 Drive 21.4
    ##  6:   6  0          Merc 280 19.2
    ##  7:   6  1         Mazda RX4 21.0
    ##  8:   6  1     Mazda RX4 Wag 21.0
    ##  9:   8  0  Pontiac Firebird 19.2
    ## 10:   8  0 Hornet Sportabout 18.7
    ## 11:   8  1    Ford Pantera L 15.8
    ## 12:   8  1     Maserati Bora 15.0

#### Summarise with .SD and .SDcols

Calculate the median of `Sepal.Length`, `Sepal.Width`, `Petal.Length` and `Petal.Width` by species:

``` r
dti[, .(Sepal.Length = median(Sepal.Length),
        Sepal.Width = median(Sepal.Width),
        Petal.Length = median(Petal.Length),
        Petal.Width = median(Petal.Width)), by = Species][order(-Species)]
```

    ##       Species Sepal.Length Sepal.Width Petal.Length Petal.Width
    ## 1:  virginica          6.5         3.0         5.55         2.0
    ## 2: versicolor          5.9         2.8         4.35         1.3
    ## 3:     setosa          5.0         3.4         1.50         0.2

We can use `.SD` to make this life easier.

``` r
dti[, lapply(.SD, median), by = Species][order(-Species)]
```

    ##       Species Sepal.Length Sepal.Width Petal.Length Petal.Width IsLarge
    ## 1:  virginica          6.5         3.0         5.55         2.0       1
    ## 2: versicolor          5.9         2.8         4.35         1.3       0
    ## 3:     setosa          5.0         3.4         1.50         0.2       0

How to select specified columns? For example I want to select the 2nd to 5th columns in `dtm` and calculate their average value.

``` r
dtm[, lapply(.SD, mean), .SDcols = 2:5]
```

    ##         mpg    cyl     disp       hp
    ## 1: 20.09062 6.1875 230.7219 146.6875

You could also directly specify the column names:

``` r
dtm[, lapply(.SD, mean), .SDcols = names(dtm)[2:5]]
```

    ##         mpg    cyl     disp       hp
    ## 1: 20.09062 6.1875 230.7219 146.6875

To mix all the internal variables together, we try more exciting exercises. We culculate the average fuel efficiency per cylinder settings grouped by `carb` \> 3 and `wt` \> 3.3.

``` r
dtm[, efcy_cly := round(mpg/cyl, 3)][, lapply(.SD, mean), by = .(carb > 3, wt > 3.3), .SDcols = c('efcy_cly', 'disp', 'hp')]
```

    ##     carb    wt efcy_cly    disp        hp
    ## 1:  TRUE FALSE 3.064500 204.000 164.75000
    ## 2: FALSE FALSE 6.407667 117.875  84.91667
    ## 3: FALSE  TRUE 2.213125 304.300 161.87500
    ## 4:  TRUE  TRUE 1.991250 339.775 215.12500

* * * * *

Somethings to be noticed
------------------------

### The problem of assignment

To copy a `data.frame`, we just need to assign it to a new symbol. You may find that we used `copy()` to make a copy of a `data.table`. Why don’t we use `<-`?

``` r
DT <- as.data.table(mtcars[1:5, 1:5])
DT1 <- DT

# Check DT1
DT1
```

    ##     mpg cyl disp  hp drat
    ## 1: 21.0   6  160 110 3.90
    ## 2: 21.0   6  160 110 3.90
    ## 3: 22.8   4  108  93 3.85
    ## 4: 21.4   6  258 110 3.08
    ## 5: 18.7   8  360 175 3.15

``` r
# Now we change DT1 - add the vehicle model name
DT1 <- DT[, model:=rownames(mtcars)[1:5]]

# Check DT
DT
```

If you know python, you’ll find that this is the same logic. Different labels point to the same memory location. You change `DT1` also change `DT`. To realize static storage as normal `data.frame` does, there are at least 2 ways.

``` r
# Use the DT just created
# 1. copy()
DT2 <- copy(DT)

# 2. subsetting []
DT3 <- DT[1:nrow(DT)]

DT2[, model:=NULL]
DT3[, model:=NULL]

# Check DT, DT2 and DT3
DT # DT not changed
```

    ##     mpg cyl disp  hp drat             model
    ## 1: 21.0   6  160 110 3.90         Mazda RX4
    ## 2: 21.0   6  160 110 3.90     Mazda RX4 Wag
    ## 3: 22.8   4  108  93 3.85        Datsun 710
    ## 4: 21.4   6  258 110 3.08    Hornet 4 Drive
    ## 5: 18.7   8  360 175 3.15 Hornet Sportabout

### Difference between `=` and `:=` in data.table

Both operators assign calculation result to a new column. However, the former one does not change the data.table it self, it generates and displays a temporary data.table; the latter does. Therefore, when you use `=` to create a new data.table, you need to assign it to a symbol either existed or not, while `:=` does not need to.
