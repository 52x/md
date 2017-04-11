title: R Visual. - China Map Part II
date: 2015-10-15 23:34:30
categories: Codage|编程
tags: [R, visualization, ggvis, ggplot2]
---

Keep the visualization study, try to realize the following tasks:

1. Plot one province or multiple provinces
2. Add labels to capitals
3. Plot the provincial population based on the latest census
4. Plot the provincial population evoluation
5. Realize the plots above using both `ggplot2` and `ggvis`

<!-- more -->

## Preparation

Load packages and modify local settings in case of Chinese characters not displaying.
``` r
library(maptools)
library(dplyr)
library(ggplot2)
library(ggvis)

Sys.setlocale("LC_ALL", "chinese")
```

### Raw Data
Subset the polygons with `AREA` larger than 0.005 and remove the `N/A`.

``` r
cnmap <- readShapePoly("bou2_4p.shp")
# cnmap@data <- na.omit(cnmap@data)
cnmap@data$ID <- row.names(cnmap@data)
cnmap <- subset(cnmap, AREA > 0.005)
cnmap@data <- na.omit(cnmap@data)
```

Create a `data.frame` called `cnmapdf` which contains `id`, `prov_en` and `prov_cn` and key map plotting info:

``` r
prov_cn <- unique(cnmap$NAME)
prov_en <- c("Heilongjiang", "Inner Mongolia", "Xinjiang", "Jilin",
             "Liaoning", "Gansu", "Hebei", "Beijing", "Shanxi",
             "Tianjin", "Shaanxi", "Ningxia", "Qinghai", "Shandong",
             "Tibet", "Henan", "Jiangsu", "Anhui", "Sichuan", "Hubei",
             "Chongqing", "Shanghai", "Zhejiang", "Hunan", "Jiangxi",
             "Yunnan", "Guizhou", "Fujian", "Guangxi", "Taiwan", 
             "Guangdong", "Hong Kong", "Hainan")

prov <- data.frame(prov_cn, prov_en)

id_prov <- cnmap@data                                               %>%
  mutate(prov_en = sapply(NAME, 
                          function(x)
                            prov$prov_en[which(prov_cn == x)]))     %>%
  mutate(prov_cn = as.character(NAME),
         prov_en = as.character(prov_en))                           %>%
  select(id = ID, prov_cn, prov_en)

# you can borrow plyr package in a more efficient way
cnmapdf <- plyr::join(fortify(cnmap), id_prov, by = "id")

# head(cnmapdf)
```

You can also use `dplyr` to realize the last step, but it is much slower.

``` r
cnmapdf <- fortify(cnmap)                                           %>%
  mutate(prov_en = sapply(id,
                          function(x)id_prov$prov_en[which(id == x)]),
         prov_cn = sapply(id,
                          function(x)id_prov$prov_cn[which(id == x)]))
```

### Coordinates of Province Capitals
Set the capital coordinates of each provinces.

``` r
cap_coord <- c(
  "Beijing", "北京", "Beijing", 116.4666667, 39.9,
  "Shanghai", "上海", "Shanghai", 121.4833333, 31.23333333,
  "Tianjin", "天津", "Tianjin", 117.1833333, 39.15,
  "Chongqing", "重庆", "Chongqing", 106.5333333, 29.53333333,
  "Harbin", "哈尔滨", "Heilongjiang", 126.6833333, 45.75,
  "Changchun", "长春", "Jilin", 125.3166667, 43.86666667,
  "Shenyang", "沈阳", "Liaoning", 123.4, 41.83333333,
  "Hohhot", "呼和浩特", "Inner Mongolia", 111.8, 40.81666667,
  "Shijiazhuang", "石家庄", "Hebei", 114.4666667, 38.03333333,
  "Taiyuan", "太原", "Shanxi", 112.5666667, 37.86666667,
  "Jinan", "济南","Shandong", 117, 36.63333333,
  "Zhengzhou", "郑州", "Henan", 113.7, 34.8, 
  "Xi'an", "西安", "Shaanxi", 108.9, 34.26666667,
  "Lanzhou", "兰州", "Gansu", 103.8166667, 36.05,
  "Yinchuan", "银川", "Ningxia", 106.2666667, 38.33333333,
  "Xining", "西宁", "Qinghai", 101.75, 36.63333333,
  "Urumqi", "乌鲁木齐", "Xinjiang", 87.6, 43.8,
  "Hefei", "合肥", "Anhui", 117.3, 31.85,
  "Nanjing", "南京", "Jiangsu", 118.8333333, 32.03333333,
  "Hangzhou", "杭州", "Zhejiang", 120.15, 30.23333333,
  "Changsha", "长沙", "Hunan", 113, 28.18333333,
  "Nanchang", "南昌", "Jiangxi", 115.8666667, 28.68333333,
  "Wuhan", "武汉", "Hubei", 114.35, 30.61666667,
  "Chengdu", "成都", "Sichuan", 104.0833333, 30.65,
  "Guiyang", "贵阳", "Guizhou", 106.7, 26.58333333,
  "Fuzhou", "福州", "Fujian", 119.3, 26.08333333,
  "Taibei", "台北", "Taiwan", 121.5166667, 25.05,
  "Guangzhou", "广州", "Guangdong", 113.25, 23.13333333,
  "Haikou", "海口", "Hainan", 110.3333333, 20.03333333,
  "Nanning", "南宁", "Guangxi", 108.3333333, 22.8,
  "Kunming", "昆明", "Yunnan", 102.6833333, 25,
  "Lhasa", "拉萨", "Tibet", 91.16666667, 29.66666667,
  "Hong Kong", "香港", "Hong Kong", 114.1666667, 22.3,
  "Macau", "澳门", "Macau", 113.5, 22.2)

cap_coord <- as.data.frame(matrix(cap_coord, nrow = 34, byrow = TRUE))
names(cap_coord) <- c("city_en", "city_cn", "prov_en", "long", "lat")

cap_coord <- cap_coord                                              %>%
  mutate(prov_en = as.vector(prov_en),
         city_en = as.vector(city_en),
         city_cn = as.vector(city_cn),
         cap_long = as.double(as.vector(long)),
         cap_lat = as.double(as.vector(lat)))                       %>%
  select(prov_en, city_en, city_cn, cap_long, cap_lat)

# cnmapdf <- plyr::join(cnmapdf, cap_coord, by = "prov_en", type = "full")
```


## Plot Map for One Province
Let's try to plot Shanghai first.

``` r
shanghai <- cnmapdf[cnmapdf$prov_en == "Shanghai",]
# or
# shanghai <- cnmapdf[prov_cn == "浙江省",]

shanghai                                                            %>%
  ggplot(aes(x = long, y = lat, group = group))                      +
  geom_polygon(fill = "beige", color = "grey")                       +
  ggtitle("上海直辖市")                                              +
  xlab("经度")                                                       +
  ylab("维度")
```

![vis2-plot1](http://7xndoy.com1.z0.glb.clouddn.com/vis2-plot1.png)

## Plot Map for Several Provinces
To plot a group of provinces, say Jiangsu, Zhejiang and Shanghai, the Yangtze River Delta.

``` r
map1 <- cnmapdf                                                     %>%
  filter(prov_en %in% c("Jiangsu", "Zhejiang", "Shanghai"))         %>%
  ggplot()                                                           +
  geom_polygon(aes(x = long, y = lat, 
                   group = group, fill = prov_cn), 
               color = "grey")

map1                                                                 +
  geom_point(x = 121.48333, y = 31.23333, fill = "beige")            +
  geom_point(x = 120.15000, y = 30.23333, fill = "beige")            +
  geom_point(x = 118.83333, y = 32.03333, fill = "beige")            +
  annotate("text", x = 121.48333, y =  30.93333, label = "上海")     +
  annotate("text", x = 120.15000, y =  29.93333, label = "杭州")     +
  annotate("text", x = 119.13333, y =  32.03333, label = "南京")     +
  ggtitle("长江三角洲")                                              +
  xlab("经度")                                                       +
  ylab("维度")
```

![vis2-plot2](http://7xndoy.com1.z0.glb.clouddn.com/vis2-plot2.png)

Why do we declare `aes()` inside `geom_polygon()` instead of `ggplot()`? Run the code below and try to move it to the aesthetic mapping in `ggplot()` and you will find the answer.

It's apparently a bit complicated using `annotate()` to add labels to provinces. We can do that this way:

```  r
coord_delta_cap <- subset(cap_coord, prov_en %in% c("Zhejiang", "Shanghai", "Jiangsu"))

map1                                                                 +
  geom_point(data = coord_delta_cap, 
             aes(x = cap_long, y = cap_lat))                         +
  geom_text(data = coord_delta_cap, 
            aes(cap_long, cap_lat - .25, label = city_cn))           +
  ggtitle("长江三角洲")                                              +
  xlab("经度")                                                       +
  ylab("维度")
```

![vis2-plot3](http://7xndoy.com1.z0.glb.clouddn.com/vis2-plot3.png)

How can we do that via `ggvis`?
In the current version (0.4.2) there is no function like `ggplot2::ggtitle()` to directly add a title to a `ggvis` package. So we can define one ourselves.

``` r
add_title <- function(vis, ..., x_lab = "", title = "Plot Title"){
  add_axis(vis, "x", title = x_lab)                                 %>% 
    add_axis("x", orient = "top", ticks = 0, title = title,
             properties = axis_props(
               axis = list(stroke = "white"),
               title = list(fontSize = 20),
               labels = list(fontSize = 0)
             ), ...)
}
```

Then apply `add_title()` to our plot

``` r
cnmapdf                                                             %>%
  filter(prov_en %in% c("Jiangsu", "Zhejiang", "Shanghai"))         %>%
  group_by(group)                                                   %>%
  ggvis(x = ~long, y = ~lat, fill = ~prov_en, stroke := "grey")     %>%
  layer_paths()                                                     %>%
  layer_points(x = ~cap_long, y = ~cap_lat, fill:= "beige",
               data = coord_delta_cap)                              %>%
  layer_text(x = ~cap_long + .2, y = ~cap_lat - .2, 
             stroke := "black", fontSize := 14,
             text := ~city_en, data = coord_delta_cap)              %>%
  add_axis("x", title = "longtitude")                               %>%
  add_axis("y", title = "latitude")                                 %>%
  add_title(title = "Yangtze River Delta")
```

![vis2-plot4](http://7xndoy.com1.z0.glb.clouddn.com/vis2-plot4.png)

## Combining Population Data
### Add Population Data
Macau is not specified in the raw data, therefore we only plot the other 33 provinces / municipalities / autonomous regions. 
Population data is scrapped from Wikipedia based on Census. The population Hong Kong and Taiwan are independently collected, so I pick the data in the most adjacent years.
You could download the `.xlsx` raw data [DemoChinaRaw.xlsx]() and the `.csv` cleaned data [DemoChina.csv]().

Need to pay attention to the changes of provinces:

* Chongqing was a city of Sichuan Province and became a municipality in 1997.
* Hainan was a part of Guangdong Province and became an independent province in 1988. This is not reflected in the result of Census in 1990).
* From 1949 to now, Tianjin is a municipality but once was a city of Hebei Province between 1958 and 1967.

``` r
democn <- read.csv("DemoChina.csv", stringsAsFactors = F)

# check the data per year and per province
library(tidyr)
head(spread(democn, year, population))
```

### Plot the Population per Province by `ggplot`
Combine the population from Census in 2010 with map data:

``` r
map2df <- cnmapdf                                                   %>% 
  plyr::join(subset(democn, year == "Census-2010"), by = "prov_en") %>%
  mutate(population = as.numeric(population))

map2df                                                              %>%
  ggplot(aes(x = long, y = lat, group = group, fill = population))   +
  geom_polygon(color = "grey")
```

![vis2-plot5](http://7xndoy.com1.z0.glb.clouddn.com/vis2-plot5.png)

To make the heat map by another color scale:

``` r
map2df                                                              %>%
  ggplot(aes(x = long, y = lat, group = group, fill = population))   +
  geom_polygon(color = "grey")                                       +
  scale_fill_gradient(low = "red", high = "yellow")
```

![vis2-plot6](http://7xndoy.com1.z0.glb.clouddn.com/vis2-plot6.png)

### Plot the Population per Province by `ggvis`
Let's try `ggvis`:

``` r
map2df                                                              %>%
  group_by(group)                                                   %>%
  ggvis(x = ~long, y = ~lat)                                        %>%
  layer_paths(fill = ~population, stroke := "grey")                 %>%
  scale_numeric("fill", range = c("red", "yellow"))
```

![vis2-plot7](http://7xndoy.com1.z0.glb.clouddn.com/vis2-plot7.png)

### Plot the Population per Year
To check the population evolution per each census by `ggplot2`.

``` r
map3df <- cnmapdf                                                   %>% 
  plyr::join(democn, by = "prov_en")                                %>%
  mutate(population = as.numeric(population))                       %>%
  na.omit()

map3df                                                              %>%
  ggplot(aes(x = long, y = lat, group = group, fill = population))   +
  geom_polygon(color = "grey", lwd = .1)                             +
  coord_equal()                                                      +
  facet_wrap(~year)
```

![vis2-plot8](http://7xndoy.com1.z0.glb.clouddn.com/vis2-plot8.png)

Based on the census in 1982 and 1990, the population in Sichuan Province is even higher than in the following years. Should be aware that before the census in 2000, Chongqing is a city of Sichuan. That is how the illusion comes.

It is a shame that `ggvis` cannot do faceting yet.


--------------

## Reference:
1. [Add a plot title to ggvis](http://stackoverflow.com/questions/25018598/add-a-plot-title-to-ggvis)
2. [Introduction to ggplot2](http://spatialanalysis.co.uk/wp-content/uploads/2013/04/james_cheshire_ggplot_intro_blog.pdf)











