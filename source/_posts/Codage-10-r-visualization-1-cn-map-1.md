title: R Visual. - China Map Part I
date: 2015-10-10 23:34:30
categories: Codage|编程
tags: [R, visualization, ggvis, ggplot2]
---

This is going to be a series of visualization with R. The main packages used are `ggplot2` and `ggvis`, and probably `shiny`. Hadley Wickham sees `ggvis` as the next generation of `ggplot2`, it is worth plotting the map by both packages. The first research subject is the China map.

The files (of different format) used for plotting geometry graphics are called ArcGIS files. The 3 key files are of extended name ".shp", ".dbf" and ".shx".

<!-- more -->

For the current subject, we only need the 3 files.

-   [`bou2_4p.shp`](http://note.youdao.com/share/?id=1d7447acad7976a355d795f1dea6abbf&type=note): The main file that stores the feature geometry.
-   [`bou2_4p.dbf`](http://note.youdao.com/share/?id=ea75b3342d26a4b93def35635899d8dd&type=note): The dBASE table that stores the attribute information of features. **There is a one-to-one relationship between geometry and attributes**, which is based on record number. Attribute records in the dBASE file must be in the same order as records in the main file.
-   [`bou2_4p.shx`](http://note.youdao.com/share/?id=3d1e0c02ab9cde35125880a6e440e9c2&type=note): The index file that stores the index of the feature geometry.

We use 4 packages for the study.

``` r
library(maptools)
library(dplyr)
library(ggplot2)
library(ggvis)
```

Data Structure of .shp File
---------------------------

Normally before we plot something, we need to know the structure of data. The function used to read shape files is `maptools::readShapePoly()`:

``` r
cnmap <- readShapePoly("bou2_4p.shp")
class(cnmap)
```

    ## [1] "SpatialPolygonsDataFrame"
    ## attr(,"package")
    ## [1] "sp"

The direct method to check the structure is to use `str(cnmap)`. I don't suggest to do that but... If you already did... You will find there are 5 main slots:

-   `@data`: a `data.frame` mapping the 925 polygons which consist the
    entire China map.
-   `@polygons`: a list of 925 polygons containing geometric points data
-   `@plotOrder`: orders to plot these polygons
-   `@bbox` and `@proj4string`: not used

``` r
str(cnmap@data)
```

    ## 'data.frame':    925 obs. of  7 variables:
    ##  $ AREA      : num  54.4 129.1 175.6 21.3 15.6 ...
    ##  $ PERIMETER : num  68.5 129.9 84.9 41.2 38.4 ...
    ##  $ BOU2_4M_  : int  2 3 4 5 6 7 8 9 10 11 ...
    ##  $ BOU2_4M_ID: int  23 15 65 22 21 62 13 11 292 292 ...
    ##  $ ADCODE93  : int  230000 150000 650000 220000 210000 620000 130000 110000 210000 210000 ...
    ##  $ ADCODE99  : int  230000 150000 650000 220000 210000 620000 130000 110000 210000 210000 ...
    ##  $ NAME      : Factor w/ 33 levels "¸£½¨Ê¡","¸ÊËàÊ¡",..: 33 13 15 5 11 2 28 3 11 11 ...
    ##  - attr(*, "data_types")= chr  "N" "N" "N" "N" ...

``` r
str(cnmap@polygons[[1]]) # only check the structure of the first polygon
```

    ## Formal class 'Polygons' [package "sp"] with 5 slots
    ##   ..@ Polygons :List of 1
    ##   .. ..$ :Formal class 'Polygon' [package "sp"] with 5 slots
    ##   .. .. .. ..@ labpt  : num [1:2] 127.8 47.9
    ##   .. .. .. ..@ area   : num 54.4
    ##   .. .. .. ..@ hole   : logi FALSE
    ##   .. .. .. ..@ ringDir: int 1
    ##   .. .. .. ..@ coords : num [1:5784, 1:2] 121 121 122 122 122 ...
    ##   ..@ plotOrder: int 1
    ##   ..@ labpt    : num [1:2] 127.8 47.9
    ##   ..@ ID       : chr "0"
    ##   ..@ area     : num 54.4

If you use English operation system, you may find messy codes in Column `NAME` of the 1st slot. Try this (under WIN7):

``` r
Sys.setlocale("LC_ALL", locale = "chinese")
```

    ## [1] "LC_COLLATE=Chinese (Simplified)_People's Republic of China.936;LC_CTYPE=Chinese (Simplified)_People's Republic of China.936;LC_MONETARY=Chinese (Simplified)_People's Republic of China.936;LC_NUMERIC=C;LC_TIME=Chinese (Simplified)_People's Republic of China.936"

    # in Linux or OS X system, the value for `locale` should be "zh_CN"

Then set the default text coding as `UTF-8`.

If you want to label the province in the future in English, here is a solution:

``` r
cn_prov_name <- na.omit(unique(cnmap$NAME))
en_prov_name <- c("Heilongjiang", "Inner Mongolia", "Xinjiang", "Jilin",
                  "Liaoning", "Gansu", "Hebei", "Beijing", "Shanxi",
                  "Tianjin", "Shaanxi", "Ningxia", "Qinghai", "Shandong",
                  "Tibet", "Henan", "Jiangsu", "Anhui", "Sichuan", "Hubei",
                  "Chongqing", "Shanghai", "Zhejiang", "Hunan", "Jiangxi",
                  "Yunan", "Guizhou", "Fujian", "Guangxi", "Taiwan", 
                  "Guangdong", "Hong Kong", "Hainan")
# length(cn_prov_name) == length(en_prov_name)
prov_name <- data.frame(cn_prov_name, en_prov_name)
```

Directly Draw China Map
-----------------------

`plot()` is the easiest function we can use for the map.

``` r
plot(cnmap)
```

![cnmap_plot1](http://7xndoy.com1.z0.glb.clouddn.com/vis-1-plot1.png)

In the map, we can see Nansha (Spratly) Islands and lots of the small islands surrounding the south-east coast (that's why there are more than 900 polygons). We can simply remove them by filtering the `AREA` (the 1st column of the 1st slot).

``` r
cnmap@data <- cnmap@data[-899,] # remove the NA records
cnmap_clean <- subset(cnmap, AREA > 0.005) 
# subset() directly works on the data.frame slot

nrow(cnmap_clean@data)
```

    ## [1] 56

Now we only need to plot 56 polygons.

``` r
plot(cnmap_clean)
```

![cnmap_plot2](http://7xndoy.com1.z0.glb.clouddn.com/vis-1-plot2.png)

Draw China Map Using `ggplot2` Package
--------------------------------------

Now we try `ggplot2`. `ggplot2::fortify()` is a amazing function which converts a generic R object into a data frame useful for plotting. When you plot a `SpatialPolygonsDataFrame` data, `ggplot2` converts it by default to `data.frame`.

``` r
str(fortify(cnmap_clean))
```

    ## Regions defined for each Polygons

    ## 'data.frame':    79095 obs. of  7 variables:
    ##  $ long : num  121 121 122 122 122 ...
    ##  $ lat  : num  53.3 53.3 53.3 53.3 53.3 ...
    ##  $ order: int  1 2 3 4 5 6 7 8 9 10 ...
    ##  $ hole : logi  FALSE FALSE FALSE FALSE FALSE FALSE ...
    ##  $ piece: Factor w/ 53 levels "1","2","3","4",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ group: Factor w/ 109 levels "0.1","1.1","2.1",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ id   : chr  "0" "0" "0" "0" ...

The first 2 columns are `long` (longitude) and `lat` (latitude) which are used to sketch the contour of polygons.

``` r
cnmap1 <- cnmap_clean                               %>%
  ggplot(aes(x = long, y= lat, group = group))       +
  geom_polygon(fill = "white", colour= "grey")

## Regions defined for each Polygons

cnmap2 <- cnmap_clean                               %>%
  ggplot(aes(x = long, y= lat, group = group))       +
  geom_polygon(fill = "white", colour= "grey")       +
  coord_map("polyconic")

## Regions defined for each Polygons

cnmap1
```

![cnmap_plot3](http://7xndoy.com1.z0.glb.clouddn.com/vis-1-plot3.png)

``` r
cnmap2
```

![cnmap_plot4](http://7xndoy.com1.z0.glb.clouddn.com/vis-1-plot4.png)

The difference between the 2 plots is the map projected coordinate system. There are dozens of projection methods, you could enter `?mapproject` to check more.

Draw China Map Using `ggvis` Package
------------------------------------

I didn't find if there is a function like `fortify()` in `ggvis` package, we need to manually convert it to a `data.frame` and use `group_by` to group the polygons.

``` r
cnmap_clean                                         %>%
  fortify()                                         %>%
  group_by(group)                                   %>%
  ggvis(~long, ~lat)                                %>%
  layer_paths(fill := "white", stroke := "grey")
```

![cnmap_plot5](http://7xndoy.com1.z0.glb.clouddn.com/vis-1-plot5.png)

------------------------------------------------------------------------

Reference
---------

1.  [ArcGIS问题：dbf shp shx sbn sbx mdb adf等类型的文件的解释](http://gisman.blog.163.com/blog/static/34493388201022254341339/)
2.  [COS访谈第九期：Hadley Wickham](http://cos.name/2013/09/a-conversation-with-hadley-wickham/)
3.  [用R软件绘制中国分省市地图](http://cos.name/2009/07/drawing-china-map-using-r/)
