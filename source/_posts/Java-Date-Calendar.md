---
title: Java中的Date和Calendar的常用用法
date: 2016-08-04 20:47:29
tags: Java
toc: true
categories: Java平台
---


在java中用到的最多的时间类莫过于 java.util.Date了
由于Date类中将getYear(),getMonth()等获取年、月、日的方法都废弃了
所以要借助于Calendar来获取年、月、日、周等比较常用的日期格式


> 注意：以下代码均已在jdk1.6中测试通过，其他版本可能使用不同，请注意！

Date与String的互转用法

```
/**
 * Date与String的互转用法，这里需要用到SimpleDateFormat
 */
Date currentTime = new Date();
SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd");
String dateString = formatter.format(currentTime);
Date date = formatter.parse(dateString);
```

<!--more-->

Date与Calendar之间的互转

```
/**
 * Date与Calendar之间的互转
 */
Calendar  cal = Calendar.getInstance();
cal.setTime(new Date());
Date date1 = cal.getTime();
```

利用Calendar获取年、月、周、日、小时等时间域

```
/**
 * 利用Calendar获取年、月、周、日、小时等时间域
 */
cal.get(Calendar.YEAR);
cal.get(Calendar.MONTH);
cal.get(Calendar.WEEK_OF_MONTH);
cal.get(Calendar.DAY_OF_MONTH);
```

对时间进行加减

```
/**
 * 对时间进行加减
 */
cal.add(Calendar.MONTH, 1);
System.out.println(cal.getTime());
```

算出给定日期是属于星期几

```
Calendarcal = Calendar.getInstance();
cal.set(2016,08,01);
String[] strDays = new String[] { "SUNDAY", "MONDAY", "TUESDAY",
                                  "WEDNESDAY", "THURSDAY", "FRIDAY", "SATURDAY"
                                };
System.out.println(strDays[cal.get(Calendar.DAY_OF_WEEK) - 1]);
