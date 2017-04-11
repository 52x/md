---
title: Java中Map按Value排序
date: 2016-08-15 13:17:29
tags: Java
toc: true
categories: Java平台
---

## 前言
> Map是键值对的集合接口，它的实现类主要包括：HashMap,TreeMap,Hashtable以及LinkedHashMap等。

### TreeMap
基于红黑树（Red-Black tree）的 NavigableMap 实现，该映射根据其键的自然顺序进行排序，或者根据创建映射时提供的 Comparator 进行排序，具体取决于使用的构造方法。

### HashMap
HashMap的值是没有顺序的，它是按照key的HashCode来实现的，对于这个无序的HashMap我们要怎么来实现排序呢？参照TreeMap的value排序。

> Map.Entry返回Collections视图。

## 按key排序

TreeMap默认是升序的，如果我们需要改变排序方式，则需要使用比较器：Comparator。Comparator可以对集合对象或者数组进行排序的比较器接口，实现该接口的public compare(T o1,To2)方法即可实现排序。

> 注意：以下代码均已在Jdk1.6测试通过了

<!--more-->

### TreeMap默认按key升序排序

```
public static void keyUpSort() {
    // 默认情况，TreeMap按key升序排序
    Map<String, Integer> map = new TreeMap<String, Integer>();
    map.put("acb1", 5);
    map.put("bac1", 3);
    map.put("bca1", 20);
    map.put("cab1", 80);
    map.put("cba1", 1);
    map.put("abc1", 10);
    map.put("abc2", 12);

    // 默认情况下，TreeMap对key进行升序排序
    System.out.println("------------正常情况，TreeMap按key升序排序--------------------");
    for (Map.Entry<String, Integer> entry : map.entrySet()) {
        System.out.println(entry.getKey() + ":" + entry.getValue());
    }
}
```

### 修改TreeMap的排序方式，按key降序排序

```
public static void keyDownSort() {
    // TreeMap,按key降序排序
    // 降序排序比较器
    Comparator<String> keyComparator = new Comparator<String>() {

        @Override
        public int compare(String o1, String o2) {
            // TODO Auto-generated method stub
            return o2.compareTo(o1);
        }
    };

    Map<String, Integer> map = new TreeMap<String, Integer>(keyComparator);
    map.put("acb1", 5);
    map.put("bac1", 3);
    map.put("bca1", 20);
    map.put("cab1", 80);
    map.put("cba1", 1);
    map.put("abc1", 10);
    map.put("abc2", 12);

    System.out.println("------------TreeMap按key降序排序--------------------");
    for (Map.Entry<String, Integer> entry : map.entrySet()) {
        System.out.println(entry.getKey() + ":" + entry.getValue());
    }
}
```

## 按Value排序

> 以下只演示按TreeMap按Value升序排序，这同样适用于HashMap。

### 修改TreeMap的排序方式，按Value升序排序

> 注意：正常情况下Map是不可以使用Collections.sort()方法进行排序的，不过可以将Map转换成list之后再进行排序。

```
public static void valueUpSort() {
    // 默认情况，TreeMap按key升序排序
    Map<String, Integer> map = new TreeMap<String, Integer>();
    map.put("acb1", 5);
    map.put("bac1", 3);
    map.put("bca1", 20);
    map.put("cab1", 80);
    map.put("cba1", 1);
    map.put("abc1", 10);
    map.put("abc2", 12);
    
    // 升序比较器
    Comparator<Map.Entry<String, Integer>> valueComparator = new Comparator<Map.Entry<String,Integer>>() {
        @Override
        public int compare(Entry<String, Integer> o1,
                Entry<String, Integer> o2) {
            // TODO Auto-generated method stub
            return o1.getValue()-o2.getValue();
        }
    };

    // map转换成list进行排序
    List<Map.Entry<String, Integer>> list = new ArrayList<Map.Entry<String,Integer>>(map.entrySet());

    // 排序
    Collections.sort(list,valueComparator);

    // 默认情况下，TreeMap对key进行升序排序
    System.out.println("------------map按照value升序排序--------------------");
    for (Map.Entry<String, Integer> entry : list) {
        System.out.println(entry.getKey() + ":" + entry.getValue());
    }
}
```

### 测试结果

```
------------正常情况，TreeMap按key升序排序--------------------
abc1:10
abc2:12
acb1:5
bac1:3
bca1:20
cab1:80
cba1:1
------------TreeMap按key降序排序--------------------
cba1:1
cab1:80
bca1:20
bac1:3
acb1:5
abc2:12
abc1:10
------------map按照value升序排序--------------------
cba1:1
bac1:3
acb1:5
abc1:10
abc2:12
bca1:20
cab1:80
```
