---
title: "使用JXL组件操作Excel和导出文件"
date: 2015-01-29 16:29:57 +0800
tags:
comments: true
categories: java
keywords: jxl, excel, java
---

这段时间参与的项目要求做几张Excel报表，由于项目框架使用了jxl组件，所以把jxl组件的详细用法归纳总结一下。本文主要讲述了以下内容：

* JXL及相关工具简介
* 如何安装JXL
* JXL的基本操作
	* 创建文件
	* 单元格操作
		* 合并单元格
		* 行高和列宽
	* 数据格式化
		* 字符串格式化
		* 对齐方式
	* 读取文件
	* 修改文件
* 导出文件实例

<!--more-->

原文链接：<http://tianweili.github.io/blog/2015/01/29/use-jxl-produce-excel/>

## 简介

jxl是一个韩国人写的java操作excel的工具, 在开源世界中，有两套比较有影响的API可供使用，一个是POI，一个是jExcelAPI。其中jExcelAPI功能相对POI比较弱一点。但jExcelAPI对中文支持非常好，API是纯Java的，并不依赖Windows系统，即使运行在Linux下，它同样能够正确的处理Excel文件。另外需要说明的是，这套API对图形和图表的支持很有限，而且仅仅识别PNG格式。

## 搭建环境

网上下载jxl.jar包，然后导入工程项目lib中，即可使用。

## 基本操作

### 创建文件

以下实例是生成一个名为“test.xls”的Excel文件，其中第一个工作表被命名为“第一页”。编译执行后，会产生一个Excel文件。

```java
// 生成Excel的类 
import java.io.File;

import jxl.Workbook;
import jxl.write.Label;
import jxl.write.WritableSheet;
import jxl.write.WritableWorkbook;

public class CreateExcel {
    public static void main(String args[]) {
        try {
            // 打开文件
            WritableWorkbook book = Workbook.createWorkbook(new File("c:/test.xls"));
            // 生成名为“第一页”的工作表，参数0表示这是第一页
            WritableSheet sheet = book.createSheet(" 第一页 ", 0);
            // 在Label对象的构造子中指名单元格位置是第一列第一行(0,0)
            // 以及单元格内容为test
            Label label = new Label(0, 0, " test ");
            // 将定义好的单元格添加到工作表中
            sheet.addCell(label);

            // 生成一个保存数字的单元格，必须使用Number的完整包路径，否则有语法歧义。
            //单元格位置是第二列，第一行，值为123.456
            jxl.write.Number number = new jxl.write.Number(1, 0, 123.456);
            sheet.addCell(number);

            // 写入数据并关闭文件
            book.write();
            book.close();
        } catch (Exception e) {
            System.out.println(e);
        }
    }
}
```

### 单元格操作
Excel中很重要的一部分是对单元格的操作，比如行高、列宽、单元格合并等，所幸jExcelAPI提供了这些支持。这些操作相对比较简单，下面只介绍一下相关的API。

#### 合并单元格

合并既可以是横向的，也可以是纵向的。合并后的单元格不能再次进行合并，否则会触发异常。

```java
// 方法作用是从(m,n)到(p,q)的单元格全部合并
WritableSheet.mergeCells( int m, int n, int p, int q);

// 合并第1列第1行到第3列第4行的所有单元格
sheet.mergeCells(0, 0, 2, 3);

// 先合并单元格，再添加内容。并且定义的列行方位在合并的单元格第一个列行方位，否则添加不上内容，如下所示：
Label label = new Label(0, 0, " 测试 ");
sheet.addCell(label);
```

#### 行高和列宽

```java
// 作用是指定第i+1行的高度
WritableSheet.setRowView( int i, int height);

// 将第一行的高度设为200
sheet.setRowView(0, 200);

// 作用是指定第i+1列的宽度
WritableSheet.setColumnView( int i, int width);

// 将第一列的宽度设为30
sheet.setColumnView(0, 30);
```

### 数据格式化

#### 字串格式化

字符串的格式化涉及到的是字体、粗细、字号等元素，这些功能主要由WritableFont和WritableCellFormat类来负责。

WritableFont有非常丰富的构造子方法，供不同情况下使用，jExcelAPI的java-doc中有详细列表，这里不再列出。

WritableCellFormat类非常重要，通过它可以指定单元格的各种属性，后面的单元格格式化中会有更多描述。

```java
//字体样式：宋体；11号；粗体
WritableFont font1 = new WritableFont(WritableFont.createFont("宋体"), 11, WritableFont.BOLD);
WritableCellFormat format1 = new  WritableCellFormat(font1);
Label label = new  Label( 0 , 0 , "test", format1);
```

#### 对齐方式

在WritableCellFormat类中，还有一个很重要的方法是指定数据的对齐方式，比如针对我们上面的实例，可以指定：

```java
// 把水平对齐方式指定为居中 
format1.setAlignment(jxl.format.Alignment.CENTRE);
// 把垂直对齐方式指定为居中 
format1.setVerticalAlignment(jxl.format.VerticalAlignment.CENTRE);
```

### 读取文件

```java
// 读取Excel的类 
import java.io.File;

import jxl.Cell;
import jxl.Sheet;
import jxl.Workbook;

public class ReadExcel {
    public static void main(String args[]) {
        try {
            Workbook book = Workbook.getWorkbook(new File("c:/test.xls"));
            // 获得第一个工作表对象
            Sheet sheet = book.getSheet(0);
            // 得到第一列第一行的单元格
            Cell cell = sheet.getCell(0, 0);
            String contents = cell.getContents();//得到单元格内容
            System.out.println(contents);
            book.close();
        } catch (Exception e) {
            System.out.println(e);
        }
    }
}
```

程序的输出结果是：

```text
    test
```

Cell接口的方法还可以获取单元格行、列位置，单元格是否隐藏等属性。具体的参考jxl的API。

### 修改文件

修改Excel文件除了打开文件的方式不同之外，其他与创建Excel是一样的。

```java
// 修改Excel的类 
import java.io.File;

import jxl.Workbook;
import jxl.write.Label;
import jxl.write.WritableSheet;
import jxl.write.WritableWorkbook;

public class UpdateExcel {
    public static void main(String args[]) {
        try {
            // 获得Excel文件
            Workbook wb = Workbook.getWorkbook(new File("c:/test.xls"));
            // 打开一个文件的副本，并且指定数据写回到原文件
            WritableWorkbook book = Workbook.createWorkbook(new File("c:/test.xls"), wb);
            
            //修改原工作表数据
            WritableSheet sheet1 = book.getSheet(0);
            sheet1.addCell(new Label(0, 0, "覆盖原来的test"));
            
            // 添加一个新工作表
            WritableSheet sheet2 = book.createSheet(" 第二页 ", 1);
            sheet2.addCell(new Label(0, 0, " 第二页的测试数据 "));
            
            book.write();
            book.close();
        } catch (Exception e) {
            System.out.println(e);
        }
    }
}
```

## 导出文件

附上一个导出文件例子。

```javascript
$( function() {
    /** 报表导出按钮 */
    $( '#exportBtn' ).click( function() {
        if( !$( '#frm' ).validationEngine( 'validate' ) )
            return false;
        $( '#frm' )[ 0 ].action = '${ctx }/exportAction.do?m=exportExcel';
        $( '#frm' )[ 0 ].submit();
        $( this ).attr( 'disabled', true );
        window.setTimeout( function() {
            document.getElementById( 'exportBtn' ).disabled = false;
        }, 5000 );
    } );
} );
```


```java
/** 根据浏览器类型，转换为当前浏览器支持的中文*/
String fileName = "Excel工作表";

/** header 浏览器key */
String userAgent = request.getHeader("USER-AGENT").toUpperCase();
if (userAgent != null && userAgent.length() != 0 && fileName != null && fileName.length() != 0) {
    /** header IE */
    if ( -1 != userAgent.indexOf("MSIE") )
        fileName = URLEncoder.encode(fileName, "UTF-8");
    /** header Mozilla */
    else if ( -1 != userAgent.indexOf("MOZILLA") )
        fileName = new String(fileName.getBytes(), "ISO8859-1");
    /** header Safari */
    else if ( -1 != userAgent.indexOf("SAFARI") )
        fileName = new String( fileName.getBytes(), "ISO8859-1");
    /** header Opera */
    else if ( -1 != userAgent.indexOf("OPERA") )
        fileName = new String( fileName.getBytes(), "ISO8859-1");
    /** header 其它内核浏览器 */
    else
        fileName = new String( fileName.getBytes(), "ISO8859-1");
}

response.setCharacterEncoding("UTF-8");
response.setContentType("application/vnd.ms-excel");
response.setHeader("Content-disposition", new StringBuffer("attachment").append( ";filename=" ).append( fileName ).append(".xls").toString() );

WritableWorkbook book = Workbook.createWorkbook(response.getOutputStream());
WritableSheet sheet = book.createSheet("Excel工作表", 0);

//...

book.write();
book.close();
```
---

作者：[李天炜](http://tianweili.github.io/)

原文链接：<http://tianweili.github.io/blog/2015/01/29/use-jxl-produce-excel/>
