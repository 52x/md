title: VBA Dictionary - Part I
date: 2015-11-11 22:34:38
categories: Codage|编程
tags: [VBA, Dictionary]
---

虽然VBA可以直接用`Application.WorksheetFunction`来直接调用EXCEL中的`Vlookup`函数(可缩略为`Application.Vlookup`或者`WorksheetFunction.Vlookup`)。可用用`For ... Next` 或 `Do While ... Loop` 循环逐行调用`.Vlookup`，或者用`Range().FormulaR1C1 = "=VLOOKUP()"`来给一列数据直接处理。
但这两种方法效率都不高，最好是能有一种VBA自有的数据存储方案，可以高效的调用和查询数据，所以有了这篇笔记(主要参考内容取自VBA for smarties站点，地址见底部reference)

<!-- more -->

## Dictionary 简介
`Dictionary`是VBA中`collection`对象的一种，可用于整合和组织不同类型的数据。`Dictionary`中的数据均临时存储于内存中，因而可以快速的读取和操作。比起EXCEL函数从硬盘调用数据并运算，这种处理方式让程序有可观的提高。

`Dictionary`可整合拥有共同属性的数据，所谓共同属性，即Key。一个`Dictionary`只能拥有不重复的key。就好像在EXCEL中用`VLOOKUP`函数查询时，被查询的数据集里只能拥有唯一的索引ID一样，否则就无法实现精确查找。

`Dictionary`并不是VBA标准库中的一员，它出自[Microsof Scripting Runtime Object Library](https://msdn.microsoft.com/en-us/library/office/aa164509%28v=office.10%29.aspx)。VBE调用前，可以先行引用此库(中文版VBE中：工具 -> 引用 -> 勾选"Microsoft Scripting Runtime")。

## 创建Dictionary
创建`Dictionary`的语句是`CreateObject("Scripting.Dictionary")`。简单的创建一个`Dictionary`可使用以下语句：

``` VB
With CreateObject("scripting.Dictionary")

End With
```

如果要添加一个条目(Item)，可使用`.Add`这个method：

``` VB
With CreateObject("scripting.Dictionary")
    .Add "Name", "Alex Ruffle
    MsgBox .Count
End With
```

那么`Name`就是这个条目的Key，而`Alex Ruffle`是key对应的值。
`MsgBox .Count`弹出消息框，告诉你这个`Dictionary`中有多少条目。

## 为Dictionary添加和替换条目
`Dictionary`只能逐条填充，且每个item只能有唯一的key。

- item可以是各种类型的数据(numbers, strings, dates, arrays, ranges, variables, collections, dictionaries, an empty string, nothing 和 objects)。这个概念上，一个`Dictionary`与R语言中的List有些类似。
- key可以是number单个, string, date 或者 object, 或者是包含单个number, string, date 或者 object的变量, 但不能是array，哪怕是1维数组。

添加和修改item有4种方法：

### Method `.Add`
上一节里已经提到，最基本的给`Dictionary`添加item的方法。
要注意的是，在这种添加Item的方法下，如果一个key已经被添加，则无法再添加相同的key。如下面的这段代码就会报错：

``` VB
With CreateObject("scripting.dictionary")
    .Add "Name", "Alex Ruffle"
    .Add "Name", "John Doe"
End With
```

### Method `.Item()=`

``` VB
With CreateObject("scripting.dictionary")
    .Item("Name") = "Alex Ruffle"
End With
```
`.Item()`的括号中接key的名字(即"Name")，等号后面跟着这个item的内容。
从这个用法也可以管中窥豹，一探Dictionary中的索引方式。
通过`.Item(key)`来搜索对应key下的item内容，如果key不存在，则等号赋值添加一对新的key和item；如果key存在，那么等号可以重新赋值。如：

``` VB
With CreateObject("scripting.dictionary")
    .Item("Name") = "Alex Ruffle"
    MsgBox .Item("Name")
    .Item("Name") = "John Doe"
    MsgBox .Item("Name")
End With
```

### Method `=.Item()`

``` VB
With CreateObject("scripting.dictionary")
    x0 = .Item("Name")
End With
```
这是一个比较奇葩的方式。将key为Name的item赋值给变量x0。如果这个key不存在，则该key会自动添加到`Dictionary`中，但是item内容为空。
此种方法无法用于修改`Dictionary`中已存在的Item的数据。

### 调用对象变量(object variable)
`Dictionary`是个对象，所以也可以将它赋值给对象变量(object variable)。

``` VB
Set dict_team = CreateObject("scripting.dictionary")
dict_team("Name") = "Alex Ruffle"
dict_team.Item("Gender") = "Male"
```
上面最后两行实现同样的效果，分别创建了条目`Name`及`Gender`。
此处修改指定item的逻辑与第二种方式无异。

## 键(Keys)

如上所述，Key可以是除了数组以外的许多种对象或数据格式，但是key的长度只能为1。

### 键的类型
目前为止，本文代码中用到的均为string key。除此之外，我们还可以使用多种类型的数据或对象作键。

``` VB
With CreateObject("scripting.dictionary")
    .Add 1, "ID0000001"	                                    'Number key
    .Item("Name") = "Alex Ruffle"                           'String key
    .Item(CDate("11/11/2015")) = "Spent 92.5 billion RMB"   'Date key

    MsgBox .Item(1) & vbNewLine & _
           .Item("Name") & vbNewLine & _
           .Item(CDate("11/11/2015"))

    'Object key
    For Each i in Worksheets
        .Item(i) = i.Name
        MsgBox .Item(i)
    Next
End With
```

以上是可能会常用的key的类型，有兴趣的话还可以继续探索其他类型的key。

### 键的唯一性
需要注意的是，当我们使用string key的时候，字母的大小写也需要注意区别。
VBA提供了两种模式，用`.CompareMode`属性来区分。
此属性默认值为0，表示区别大小写。如果赋值为1，则不区分大小写。

``` VB
With CreateObject("scripting.dictionary")
    For Each it In Array("aa1", "AA1", "Aa1", "aA1", "bb1", "BB1", "Bb1", "bB1")
        y = .Item(it)
    Next
    
    MsgBox .Count ' 8 unique keys
    MsgBox Join(.Keys, vbNewLine)
End With
```

以上返回8个排它的key，而下面的代码只返回两个key，且返回值均默认为小写。

``` VB
With CreateObject("scripting.dictionary")
    .CompareMode = 1
    For Each it In Array("aa1", "AA1", "Aa1", "aA1", "bb1", "BB1", "Bb1", "bB1")
        y = .Item(it)
    Next
    
    MsgBox .Count ' 8 unique keys
    MsgBox Join(.Keys, vbNewLine)
End With
```

## 条目(Items)
条目可以为空集，也可以是上文提到的任意类型数据
``` VB
With CreateObject("scripting.dictionary")
    x0 = .Item("aa1")                       'Nothing
    .Item("x1") = vbNullString              'An empty string
    .Item("x2") = ""                        'An empty string
    .Item("x3") = "vbNullString"            'A string
    .Add "x4", vbTab                        'A non-printable character
    .Add "x5", 12321423                     'A number
    .Add "x6", DateSerial(2015,11,12)       'A Date
    .Item("x7") = Array("a1","a2","a3")	    'A 1-dimension array
    .Item("x8") = Range("A1:D4").Formula    'A multi-dimensional array
    .Add "x9", Range("A1:D4")               'An object
    Set .Item("x10") = Range("A5:D8")       'An object (use Set method)

    'Worksheets
    For Each sht in Sheets
        Set .Item(sht.Name) = sht
    Next
End With
```

## `.Items`及`.Keys`属性
两者以array的形式分别返回所有的item和key。
`Dictionary`提供`.Count`方式返回其中元素的数量。同样我们也可以用`UBound()`函数去计算数组的上限，从而返回元素的数量。在数组中，第一个元素下标为0，所以`UBound(array) + 1`才是一个数组的元素总量。

``` VB
With CreateObject("scripting.dictionary")
    For Each it In Array("x1", "x2", "x3", "x4", "x2")
        y = .Item(it)
    Next
    
    MsgBox Join(.Keys, vbNewLine)
    MsgBox IsArray(.Keys) & " " & IsArray(.Items)
    MsgBox .Count
    MsgBox UBound(.Keys) + 1
    MsgBox UBound(.Items) + 1
End With
```

## `.Exists(key)`查看键是否存在
为避免`Dictionary`的录入错误，我们可以在添加新的key之前先行检查被添加项是否已经存在。这里要用到的方法是`.Exists(key)`。

``` VB
Set dict = CreateObject("scripting.dictionary")

'Create a dictionary
With dict
    For Each it In Array("x1", "x2", "x3", "x4", "x5")
        .Item(it) = it & "_content"
    Next
End With

MsgBox Join(dict.Keys, vbNewLine)
MsgBox Join(dict.Items, vbNewLine)

With dict
    For Each it In Array("x4", "x5", "x6", "x7")
        If Not .Exists(it) Then
            .Item(it) = it & "_new_content"
        Else
            .Item(it) = it & "_changed_content"
	    End If
    Next
End With

MsgBox Join(dict.Keys, vbNewLine)
MsgBox Join(dict.Items, vbNewLine)
```

下一篇Dictionary将主要介绍Dicitonary中元素的提取、删除、合并以及实例的应用。

---------------------

## Reference
- [VBA for Smarties: Dictionary](http://www.snb-vba.eu/VBA_Dictionary_en.html)

