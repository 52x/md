title: Scraping Data Table from Website by R
date: 2015-09-08 22:11:37
categories: Codage|编程
tags: [R, dplyr, RMySQL, XML, quantmod]
---

For many quantitative analysis, we often consider factors such climate or economics data. Those data are usually displayed as tables on the websites and can be collected from there. However, to copy-paste with EXCEL is an extremely inefficient way. 
We can use R to read data directly from the web pages and store them uniformly without too may data clean steps.
<!-- more -->
To be clarified, most of the statistical models are **not instantly renewed**. The "instant" here means along with the data increase by time, the model need to be / can be renewed in the same time or in a short time.

Our task is to realize a semi-automated web data scraping program, whenever a modeling refresh is needed, the raw data should be prepared.
So we need to realize 2 things:
* read data by specifying various arguments;
* store the data into local data base server.

Here we take 2 examples:
* Climate Data (daily)
* Stock Index (daily)

Several packages need to be loaded before hand.
```
require(XML, quietly = T, warn.conflicts = F)
require(quantmod, quietly = T, warn.conflicts = F)
require(RMySQL, quietly = T, warn.conflicts = F)
require(dplyr, quietly = T, warn.conflicts = F)
```

## Climate Data Collection - XML::readHTMLTable
In XML package, there is a function which enable R to directly read tables from website (when it really contains tables).
Let's take a look on this function first:
``` r
readHTMLTable(doc, header = NA,
              colClasses = NULL, skip.rows = integer(), trim = TRUE,
              elFun = xmlValue, as.data.frame = TRUE, which = integer(),
              ...)
```
`readHTMLTable()` inherits `read.table()` in `base` package. You can specify more arguments than what you see here if you are familiar with the latter function.
`doc` requires a HTML document - either a file name or a URL. To simplify it, we can directly assign a URL to it.
`which` an integer vector identifying which tables to return from within the document. This applies to the method for the document, not individual tables. To judge the number of tables, you could inspect the elements of the website and count the number of `<table> ` tags.
We use "http://en.tutiempo.net/climate/" for gathering the climate data. The data in this website is on city basis. By selecting a city, year and month (i.e. China -> Beijing -> 2015 -> jun), you can see a URL like "http://en.tutiempo.net/climate/06-2015/ws-545110.html".
Open this page, there are 2 tables, one for climate data and one for data field interpretation. Right click on that page, select `Inspect elements`, it's also not hard to find 2 `<table>` tags.
To read the first table, specify the table number `which = 1`:
``` r
webtable <- readHTMLTable(http://en.tutiempo.net/climate/06-2015/ws-545110.html,
                          header = T, which = 1, stringsAsFactors = F
```

What if you want to download data of different countries and different period?
Noticed that the URL is composed of 3 parts, take the previous case for instance.
* head: "http://en.tutiempo.net/climate/"
* month-year: "06-2015"
* country-city code: "/ws-545110.html"

Changing the latter 2 parts, you could collect as many info as you want.

``` r
GrabClimate <- function(year, month, country){
  month <- paste(ifelse(nchar(month)==1,"0",""), month, sep = "")

  # link_tail: the collection of main APAC countries
  link_tail <- c(`AUSTRALIA` = "/ws-947680.html", # SYDNEY
                 `CHINA` = "/ws-545110.html", # BEIJING
                 `INDIA` = "/ws-421820.html", # NEW DEHLI
                 `INDONESIA` = "/ws-967490.html", # JAKARTA
                 `JAPAN` = "/ws-476620.html", # TOKYO
                 `KOREA` = "/ws-471080.html", # SEOUL
                 `MALAYSIA` = "/ws-486470.html", # KUALA LUMPUR
                 `NEW ZEALAND` = "/ws-934390.html", # WELLINGTON
                 `PHILIPPINES` = "/ws-984250.html", # MANILA
                 `THAILAND` = "/ws-484540.html", # BANGKOK
                 `TAIWAN` = "/ws-589680.html", # TAIPEI
                 `VIETNAM` = "/ws-488200.html")  # HA NOI

  # the address need to be adjusted according to the country to be modeled
  link <- paste("http://en.tutiempo.net/climate/", month, "-", year,
                link_tail[toupper(country)], sep = "")

  webtable <- readHTMLTable(link, header = T, which = 1, stringsAsFactors = F,
                            na.strings = c("-"))

  # Add 2 columns and remove the bottom 2 lines
  webtable <- data.frame(Country = toupper(country), Year = as.numeric(year),
                         Month = as.numeric(month), webtable[1:(nrow(webtable)-2),])

  # Remove unuseful columns and rename the table
  webtable <- select(webtable, -(VV:VG))
  names(webtable) <- c("Country", "Year", "Month", "Day", "Avg_Temp", "Max_Temp",
                       "Min_Temp", "Atm_Pressure", "Avg_Humidity", "Rainfall",
                       "Ind_Rain_Drizzle", "Ind_Snow", "Ind_Storm", "Ind_Fog")
  rownames(webtable) <- NULL

  # Replace "-" by NA.
  webtable <- mutate(webtable,
                     Day = as.numeric(Day),
                     Ind_Rain_Drizzle = ifelse(Ind_Rain_Drizzle == "o", 1, 0),
                     Ind_Snow = ifelse(Ind_Snow == "o", 1, 0),
                     Ind_Storm = ifelse(Ind_Storm == "o", 1, 0),
                     Ind_Fog = ifelse(Ind_Fog == "o", 1, 0))

  webtable
}
```
Using this function in embedded loops you can get the full set of data (Suppose we assign the data to `climate`, we will use it later.).

## Stock Index Data Collection - quantmod::getSymbols
In `quantmod` package, `getSymbols()` is well developed function. Specifying the index symbol, the function automatically returns the table read from Yahoo Finance (you can also set other website), and save it in the a variable named by the symbol. You can also specify the data start and end date.

``` r
# the list of major APAC stock index code
sym_stock <- c(`AUSTRALIA` = "^AORD", # All Ordinaries
               `CHINA` = "^SSEC", # Shanghai Composite
               `INDIA` = "^BSESN", # Bombay BSE SENSEX / BSE 30
               `INDONESIA` = "^JKSE", # Jakarta Composite
               `JAPAN` = "^N225", # Tokyo Share Nikkei 225
               `KOREA` = "^KS11", # Seoul Composite
               `MALAYSIA` = "^KLSE", # FTSE Bursa Malaysia KLCI
               `NEW ZEALAND` = "^NZ50", # New Zealand NZX 50 Index
               `PHILIPPINES` = "^PSI", # Manila
               `THAILAND` = "SET", # BANGKOK
               `TAIWAN` = "^TWII", # Taiwan Weighted
               `VIETNAM` = "VNM") # Vietnam Index

stock_ind <- data.frame(matrix(numeric(), ncol = 8,
                               dimnames = list(NULL,
                                               c("Country", "Date", "Open", "High", "Low",
                                                 "Close", "Volumne", "AdjClose"))))

for (i in 1:12){
  # auto.assign = F, do not assign the read table under the name of index code
  temp <- as.data.frame(suppressWarnings(getSymbols(sym_stock[i], auto.assign = F,
                                                    from = "2013-01-01",
                                                    to = "2015-07-31")))
  names(temp) <- c("Open", "High", "Low", "Close", "Volumne", "AdjClose")
  stock_ind <- rbind(stock_ind, data.frame(Country = names(sym_stock)[i],
                                           Date = rownames(temp), temp))
}
```

## Load Data to MySQL
`dplyr` and `RMySQL` both provide method for R to connect to MySQL.
``` r
# create a database "example"
con <- dbConnect(MySQL(), host = "xx.xxx.xxxx.xxx", port = 3306,
                 user = "Admin", password = "Power_Overwhelming")
dbSendQuery(con, "CREATE SCHEMA `example`")

# connection to MySQL db "example" using dplyr method
con1 <- src_mysql(dbname = "example", host = "xx.xxx.xxxx.xxx", port = 3306,
                  user = "Admin", password = "Power_Overwhelming")

# connection to MySQL db "example" using RMySQL method
con2 <- dbConnect(MySQL(), host = "xx.xxx.xxxx.xxx", port = 3306, db = "example",
                  user = "Admin", password = "Power_Overwhelming")
```
To create new tables, we can use `dplyr::copy_to` to create MySQL table.
``` r
copy_to(con1, climate, "climate", temporary = F)
copy_to(con1, stock_ind, "stock", temporary = F)
```
If the table already exists:
``` r
db_insert_into(con2, table = "climate", value = climate)
db_insert_into(con2, table = "stock", value = stock_ind)
```
Add index to optimize query speed:
``` r
dbSendQuery(con2, "ALTER TABLE climate
            ADD KEY climate(Country, Year, Month, Day,
                            Avg_Temp, Max_Temp, Min_Temp,
                            Rainfall, Ind_Rain, Ind_Snow,
                            Ind_Storm, Ind_Fog)")
dbSendQuery(con2, "ALTER TABLE stock
            ADD KEY stock(Open, High, Low, Close, Volumne, AdjClose)")
```

We can use `dplyr::tbl()` and a series of `dplyr` functions to manipulate data in MySQL databasess.
