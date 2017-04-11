title: VBA Dictionary - Part II
date: 2015-11-16 22:13:02
categories: Codage|编程
tags: [VBA, Dictionary]
---

上一篇文章[VBA Dictionary - Part I](http://papacochon.com/2015/11/11/Codage-13-vba-dictionary-1/)提到`Dictionary`的创建、添加key、添加和修改item、`.Items`和`.Keys`的属性以及检查一个key是否存在的`.Exists()`方法。

我们还可以对`Dictionary`做一些别的操作：

* 复制item
* 删除key和item

另外我们需要知道`Dictionary`返回的结果需要用什么样的方式来呈现。

<!-- more -->

## 复制item
当两个key同时对应一个item的时候，我们可以直接将第一个item赋值给第二个。

``` VB
With CreateObject("scripting.dictionary")
    .Add "item1", "Price1"
    .Item("item2") = .Item("item1") 

    MsgBox .Item("item1") = .Item("item2")    'True
End With
```

## 删除key和对应item
用`.Remove`方法删除指定key及对应item。

``` VB
With CreateObject("scripting.dictionary")
    .Add "item1", "Price1"
    .Item("item2") = .Item("item1") 

    MsgBox Join(.Keys, vbNewLine)

    .Remove("item2")
    MsgBox Join(.Keys, vbNewLine)
End With
```

用`.RemoveAll`删除所有key。

``` VB
With CreateObject("scripting.dictionary")
    .Add "item1", "Price1"
    .Item("item2") = .Item("item1") 

    MsgBox Join(.Keys, vbNewLine)

    .RemoveAll
    MsgBox .Exists("item1")
End With
```

## `Dictionary`的应用
假设我现有两张表，一张“库存”一张“销售”。

**库存 表例** `Range("A1:D6")`

商品编号|商品名称|现有库存|库存状态
--------|--------|--------|--------
A2015001|  产品1 |   24   |  正常  
A2015002|  产品2 |   35   |  正常  
A2015003|  产品3 |   19   |  正常  
A2015004|  产品4 |   11   |  破损  
A2015005|  产品5 |   10   |  过期  

**销售 表例** `Range("A11:E16")`

商品编号|商品名称|日均销量|产品品牌|产品分类
--------|--------|--------|--------|--------
A2015001|  产品1 |    8   |  品牌1 |  分类1
A2015002|  产品2 |   13   |  品牌2 |  分类2
A2015003|  产品3 |    5   |  品牌3 |  分类3
A2015004|  产品4 |    1   |  品牌4 |  分类1
A2015005|  产品5 |    2   |  品牌5 |  分类2

### 单列查询
我想以库存信息为基础，看到每个商品的日均销量。

``` VB
Dim sales_id, stock_id, avg_sales, arr
Dim dict

set dict = CreateObject("scripting.dictionary")

stock_id = Range("A1:A6")
sales_id = Range("A11:A16")
avg_sales = Range("C11:C16")

'将日均销量的值，对应商品id，逐条插入字典
For i = 2 to UBound(sales_id)
    dict(sales_id(i, 1)) = avg_sales(i, 1)
Next

Redim arr(2 To UBound(stock_id), 1 To 1)

'调用字典，将查询结果存入arr数组
For j = 2 to UBound(stock_id)
    arr(j, 1) = dict(stock_id(j, 1))
Nex

'复制数组至指定位置
Range("E1").Value = "日均销量"
Range("E2").Resize(UBound(arr) - 1, 1) = arr

set dict = Nothing
```

### 多列查询
我想把日均销量、产品品牌及产品分类都放到库存目录后，思路是将上一个版本中插入的item从单一的数值变成一个一维数组，一个数组即包含了单个商品的对应销量信息。

``` VB
Dim sales_id, stock_id, sales
Dim dict

set dict = CreateObject("scripting.dictionary")

stock_id = Range("A1:A6")
sales_id = Range("A11:A16")
sales = Range("C11:E16")

'将三列信息的值，对应商品id，逐条插入字典
For i = 2 to UBound(sales_id)
    dict(sales_id(i, 1)) = Application.Index(sales, i, 0)
Next

'补足新加列的表头
Range("C11:E11").copy Range("E1")

'调用字典，将查询结果直接写入表格
For j = 2 to UBound(stock_id)
    Range("E" & i & ":G" & i) = dict(stock_id(j, 1))
Next

set dict = Nothing
```