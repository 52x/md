title: R Visual. - Bubbles and Bar Charts on China Map
date: 2015-10-18 23:24:12
categories: Codage|编程
tags: [R, visualization, ggvis, ggplot2]
---

Keep exploring the things we can do on a map with `ggplot2` and `ggvis`.

<!-- more -->

About visualization on a map:

* [R Visualization Series - China Map Part I](http://papacochon.com/2015/10/10/Codage-10-r-visualization-1-cn-map-1/)
* [R Visualization Series - China Map Part II](http://papacochon.com/2015/10/15/Codage-11-r-visualization-2-cn-map-2/)

The data to be used:

* China map shape file:

    - [`bou2_4p.shp`](http://note.youdao.com/share/?id=1d7447acad7976a355d795f1dea6abbf&type=note)
    - [`bou2_4p.dbf`](http://note.youdao.com/share/?id=ea75b3342d26a4b93def35635899d8dd&type=note)
    - [`bou2_4p.shx`](http://note.youdao.com/share/?id=3d1e0c02ab9cde35125880a6e440e9c2&type=note)

* China demographic of 6 censuses:

    - [`DemoChina.csv`](http://note.youdao.com/share/?id=81c4a0e4ea281d0ff96e61f7cc1f88f9&type=note)

* The coordinates of province capitals in China:

    - [`prov_capital.csv`](http://note.youdao.com/share/?id=3fa56717b7f98bacdcde045a2254b365&type=note)

* The private vehicle ownership per province (source: [NBSC](http://data.stats.gov.cn/easyquery.htm?cn=E0103), Unit: k; Year: 1998, 2003, 2008, 2013; Hong Kong, Macau & Taiwan excl.):

    - [`car_ownership.csv`](http://note.youdao.com/share/?id=ce748a9f4ee26786eb4f945eef17f56a&type=note)

## Preparation

Load packages and set the language environment:
``` r
library(maptools)
library(dplyr)
library(ggplot2)
library(ggvis)

Sys.setlocale("LC_ALL", "chinese")
```

```
## [1] "LC_COLLATE=Chinese (Simplified)_People's Republic of China.936;LC_CTYPE=Chinese (Simplified)_People's Republic of China.936;LC_MONETARY=Chinese (Simplified)_People's Republic of China.936;LC_NUMERIC=C;LC_TIME=Chinese (Simplified)_People's Republic of China.936"
```

Load and clean the raw data. Get 2 data frame `cnmapdf` and `cap_coord` as we did in the [previous visualization article](http://papacochon.com/2015/10/15/Codage-11-r-visualization-2-cn-map-2/).

As we are going to draw pie and bar chart for each province, we need to add a bit more information to `cap_coord`.

``` r
car_prov <- read.csv("car_ownership.csv", stringsAsFactors = F)

cap_bubble <- cap_coord                                                 %>%
  plyr::join(subset(car_prov, year == 2013), by = "prov_en")            %>%
  na.omit()

cap_bar <- cap_coord                                                    %>%
  plyr::join(car_prov, by = "prov_en")                                  %>%
  na.omit()                                                             %>%
  filter(city_en %in% c("Beijing", "Shanghai", "Guangzhou", "Chengdu")) %>%
  mutate(year = paste("Y", year, sep = ""))                             %>%
  tidyr::spread(year, no_car_k)
```

## Bubbles on A Map
### Bubbles with A Single Color - `ggplot2`
A bubble can be considered as a point, of which the size is controlled by the car ownership statistics.

``` r
ggplot()                                                                 +
  geom_polygon(data = cnmapdf, aes(long, lat, group = group), 
               fill = "skyblue", colour = "grey")                        +
  geom_point(data = cap_bubble, aes(cap_long, cap_lat, size = no_car_k), 
             shape = 21, fill = "red", alpha = .5)                       +
  scale_size_area(max_size=20)
```

![vis3-plot1](http://7xndoy.com1.z0.glb.clouddn.com/vis3-plot1.png)

### Bubbles with Effect of Heat Map - `ggplot2`
Calculate the growth rate the private car from 2008 to 2013, assign the data to argument `fill` of `geom_point()`.

``` r
cap_bubble["growth"] <- subset(car_prov, year == 2013)[,3,drop = F] / 
  subset(car_prov, year == 2008)[,3,drop = F]-1

ggplot()                                                                 +
  geom_polygon(data = cnmapdf, aes(long, lat, group = group), 
               fill = "skyblue", colour = "grey")                        +
  geom_point(data = cap_bubble, shape = 21,
             aes(cap_long, cap_lat, size = no_car_k,
                                 fill = growth, alpha = .5))             +
  scale_fill_gradient(low = "red", high = "yellow")                      +
  scale_size_area(max_size=20)
```

![vis3-plot2](http://7xndoy.com1.z0.glb.clouddn.com/vis3-plot2.png)

The bubbles with transparency cannot display the difference clearly when the growth rates are similar across provinces. Let's try smaller bubbles without specifying the transparency. We can also remove the legend to keep the map simple and clear.

``` r
ggplot()                                                                 +
  geom_polygon(data = cnmapdf, aes(long, lat, group = group), 
               fill = "skyblue", colour = "grey")                        +
  geom_point(data = cap_bubble, shape = 21, colour = "white",
             aes(cap_long, cap_lat, size = no_car_k, fill = growth))     +
  scale_fill_gradient(low = "red", high = "yellow")                      +
  scale_size_area(max_size=15)                                           +
  theme(legend.position = "none")    
```

![vis3-plot3](http://7xndoy.com1.z0.glb.clouddn.com/vis3-plot3.png)

It seems that from 2008 to 2013, Hainan and Gansu are the 2 provinces with the quickest growth.

### Bubbles by `ggvis`
``` r
cnmapdf                                                                 %>%
  group_by(group)                                                       %>%
  ggvis(x = ~long, y = ~lat)                                            %>%
  layer_paths(fill := "skyblue", stroke := "grey")                      %>%
  layer_points(data = cap_bubble, x = ~cap_long, y = ~cap_lat,
               fill = ~growth, size = ~no_car_k, stroke := "white")     %>%
  scale_numeric("fill", range = c("red", "yellow"))                     %>%
  scale_numeric("size", range = c(50, 500))                             %>%
  hide_legend(scales = c("fill", "size"))                               %>%
  add_axis("x", title = "Longitude")                                    %>%
  add_axis("y", title = "Latitude")
```

![vis3-plot4](http://7xndoy.com1.z0.glb.clouddn.com/vis3-plot4.png)

To remove the legend, `ggvis` provides `hide_legend()`, within this function, you can specify the names of scales to be hidden. 

## Bar Charts on A Map
### Bar Charts by `ggplot2`
It is more complicated to place a bar chart than plot just a bubble on certain spot.
To locate a bubble, you just need to give `aes()` the coordinates which have nothing to do with the size and color of bubble.
Regarding bar chart, we need to assign at least 2 pairs of `x` and `y`, one for bar chart location and the other for the height and width of the bar chart. In other words, we need to know the coordinates of 4 corners for a bar.

Since a bar chart may takes more space and looks more complicated than a bubble, we just plot 4 cities.

``` r
p1 <- ggplot()                                                           +
  geom_polygon(data = cnmapdf, aes(long, lat, group = group), 
               fill = "beige", colour = "grey")                          +
  geom_errorbar(data = cap_bar, size =3, colour = "brown", 
                alpha = .8, width = 0,
                aes(x = cap_long - 1, ymin = cap_lat,
                    ymax = cap_lat + Y2003 / Y2013 * 3))                 +
  geom_errorbar(data = cap_bar, size =3, colour = "red", 
                alpha = .8, width = 0,
                aes(x = cap_long, ymin = cap_lat,
                    ymax = cap_lat + Y2008 / Y2013 * 3))                 +
  geom_errorbar(data = cap_bar, size =3, colour = "orange", 
                alpha = .9, width = 0,
                aes(x = cap_long + 1, ymin = cap_lat,
                    ymax = cap_lat + Y2013 / Y2013 * 3))                 +
  geom_text(aes(85, 24), colour = "brown", 
            label = "Car Ownership - 2003")                              +
  geom_text(aes(85, 22), colour = "red", 
            label = "Car Ownership - 2008")                              +
  geom_text(aes(85, 20), colour = "orange", 
            label = "Car Ownership - 2013")

p1
```

![vis3-plot5](http://7xndoy.com1.z0.glb.clouddn.com/vis3-plot5.png)

We could add more details, i.e. the avg. growth rate in the from 2003 to 2013.

``` r
# define a function to find the root
root10find <- function(x, y){
  
  low <- 0
  high <- y/x
  ans <- (low + high)/2
  
  while (abs(x * ans^10 - y) > 0.001){
    if (x * ans^10 < y){
      low <- ans
    }else{
      high <- ans
    }
    ans <- (high + low)/2
  }
  return(ans)
}

for (i in 1:4){
  cap_bar[i,"growth_10y"] <- root10find(cap_bar[i,"Y2003"], cap_bar[i,"Y2013"])-1
}

p1                                                                       +
  ggtitle("Yearly Growth of Private Vehicles")                           +
  geom_text(data = cap_bar, 
            aes(cap_long, cap_lat - 0.5,
                label = paste(prov_en, ": ", 
                              round(growth_10y * 100, 2),
                              "%", sep = "")))
```

![vis3-plot6](http://7xndoy.com1.z0.glb.clouddn.com/vis3-plot6.png)

### Bar Charts by `ggvis`
It is the same logic to plot by `ggvis`.
``` r
year_labels <- data.frame(long = c(80, 80, 80),
                          lat = c(24, 22, 20),
                          label = c("Car Ownership - 2003",
                                    "Car Ownership - 2008",
                                    "Car Ownership - 2013"),
                          fill = c("brown", "red", "orange"))

cnmapdf                                                                 %>%
  group_by(group)                                                       %>%
  ggvis(x = ~long, y = ~lat, fill := "beige", stroke := "grey")         %>%
  layer_paths()                                                         %>%
  add_axis("x", title = "longtitude")                                   %>%
  add_axis("y", title = "latitude")                                     %>%
  layer_rects(data = cap_bar, fill := "brown", stroke := "white",
              x = ~cap_long - 1.5, x2 = ~cap_long - .5,
              y = ~cap_lat, y2 = ~cap_lat + Y2003 / Y2013 * 3)          %>%
  layer_rects(data = cap_bar, fill := "red", stroke := "white",
              x = ~cap_long - .5, x2 = ~cap_long + .5,
              y = ~cap_lat, y2 = ~cap_lat + Y2008 / Y2013 * 3)          %>%
  layer_rects(data = cap_bar, fill := "orange", stroke := "white",
              x = ~cap_long + .5, x2 = ~cap_long + 1.5,
              y = ~cap_lat, y2 = ~cap_lat + Y2013 / Y2013 * 3)          %>%
  layer_text(data = year_labels, x = ~long, y = ~lat, 
             text := ~label, stroke := ~fill)                            %>%
  scale_nominal("fill", range = c("brown", "red", "orange"))            %>%
  layer_text(data = cap_bar, x = ~cap_long -.5, y = ~cap_lat - 1,
             text := ~prov_en, stroke:= "black", align:="right")        %>%
  layer_text(data = cap_bar, x = ~cap_long +.5, y = ~cap_lat - 1,
             text := ~round(growth_10y*100 ,2), 
             stroke:= "black", align:="left")
```

![vis3-plot7](http://7xndoy.com1.z0.glb.clouddn.com/vis3-plot7.png)

I tried to combine the last 2 `layer_text()` functions together (the chunk below) and add a "%" to the end of the percentage point but it returns `undefined` at the position which is supposed to be like `Beijing 14.78%`. Need to figure out why the function does not support multiple embedding.

``` r
layer_text(data = cap_bar, x = ~cap_long -.5, y = ~cap_lat - 1,
           text := ~paste(prov_en, ": ", round(growth_10y * 100, 2),
                          "%", sep = ""),
           fill := "black", align := "center")
```

-----------

## Reference

1. [刘万祥: ggplot绘制商务图表--热力气泡中国地图](http://mp.weixin.qq.com/s?__biz=MzA5OTM1NDQwOA==&mid=301839416&idx=1&sn=f301a6e1bde4fd33a7cf5ae26c1488f8&scene=1&srcid=1016dQkNwckBmrGkS36herzL&key=b410d3164f5f798e38fef99f9c537830722ae774347c2d03dc14da24dc3fdf784c5999cba9699fb48fc46911d15bad42&ascene=1&uin=MTU5MzIwMTkxNQ%3D%3D&devicetype=Windows+7&version=61050016&pass_ticket=R%2Bn5ah4D6uROw3ZEyFq5kUficdfwc8CkaKox2zMOP5pj5oAL8M2XaXXXUqT6Mcff)
2. [刘万祥: ggplot绘制商务图表--地图上的迷你柱形图](http://mp.weixin.qq.com/s?__biz=MzA5OTM1NDQwOA==&mid=308574860&idx=1&sn=d75ec36b8adc1aca2d7947f4df3a9c72&scene=1&srcid=1016YR3jAkt87MM5BrdkdBBU&key=b410d3164f5f798e54adf1f73ea22fd8d4ac056586b21647b2560e8c531c10ff39641ede580b1466863aaccecd48a3ea&ascene=1&uin=MTU5MzIwMTkxNQ%3D%3D&devicetype=Windows+7&version=61050016&pass_ticket=R%2Bn5ah4D6uROw3ZEyFq5kUficdfwc8CkaKox2zMOP5pj5oAL8M2XaXXXUqT6Mcff)