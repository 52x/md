title: Shiny包初探 - Part I
date: 2015-12-02 23:14:36
categories: Codage|编程
tags: [R, shiny, visualization, interactive]
---

作为RStudio官网上享有独立域名的一个包，它的名字都那么响亮——shiny。这是据我所知到目前为止功能最好，编写最完备的一个交互可视化包。[详细教程](http://shiny.rstudio.com/tutorial/)在官网上有介绍，这里只做一个回顾。

<!-- more -->

## 基本框架
编写一个Shiny的应用需要两个基本R Script: ui.R和server.R。前者用于界面交互，一方面从用户处获取动作指令和input，一方面对接server.R处理过后的数据，将图表呈现出来; 后者读取input，处理并计算，生成计算结果和图形，储存在内存中。

另外有两个非必须的元素，一是同一个工作路径下的data文件夹，其中包含需要从本地读取的数据(如果数据从网络获取，则此文件夹非必须)；二是helpers.R文件，其中包含帮助实现数据处理和计算的自定义函数(亦非必须)。

### ui.R
ui.R中的代码必须包含在下面的框架内：

``` r
shinyUI(fluidPage(
    # code
))
```

`fuildPage()`可以自适应用户的浏览器尺寸并调整shiny应用中元素的位置。
在此函数内，还有几个主要的函数构成：

``` r
shinyUI(fluidPage(
    # 设定app的主题
    titlePanel("title panel"),
    
    # 设定app的主体和侧边栏构成
    sidebarLayout(
        sidebarPanel("sidebar panel"),
        mainPanel("main panel")
    )
))
```

### server.R
server.R中的代码框架如下:

``` r
shinyServer(function(input, output){
    # code
})
```

## 交互及视觉效果
### HTML5函数
具体效果可见[官方教程第二课](http://shiny.rstudio.com/tutorial/lesson2/)。
既然ui.R负责浏览器页面上的交互，必不可少的也具备转换HTML5语言的函数。

*表一*

shiny函数      | HTML5对照        | 创建项目
---------------|------------------|----------------------
p              | `<p>`            | A paragraph of text
h1             | `<h1>`           | A first level header
h2             | `<h2>`           | A second level header
h3             | `<h3>`           | A third level header
h4             | `<h4>`           | A fourth level header
h5             | `<h5>`           | A fifth level header
h6             | `<h6>`           | A sixth level header
a              | `<a>`            | A hyper link
br             | `<br>`           | A line break (e.g. a blank line)
div            | `<div>`          | A division of text with a uniform style
span           | `<span>`         | An in-line division of text with a uniform style
pre            | `<pre>`          | Text ‘as is’ in a fixed width font
code           | `<code>`         | A formatted block of code
img            | `<img>`          | An image
strong         | `<strong>`       | Bold text
em             | `<em>`           | Italicized text
HTML           |                  | Directly passes a character string as HTML code

这些函数通常可用于`siderbarPanel()`和`mainPanel()`函数中，相邻函数之间用","隔开。

*更多的细节设置可参见[Customize your UI with HTML](http://shiny.rstudio.com/articles/html-tags.html)及[Shiny HTML Tag Glorssary](http://shiny.rstudio.com/articles/tag-glossary.html)。*

### 控件
控件用于帮助用户收集数据，是ui.R输入的一种主要方式。和HTML函数一样，控件代码主要放置在`sidebarPanel()`和`mainPanel()`中。

*表二*

函数               | 控件
-------------------|-------------------------
actionButton       | Action Button
checkboxGroupInput | A group of check boxes
checkboxInput      | A single check box
dateInput          | A calendar to aid date selection
dateRangeInput     | A pair of calendars for selecting a date range
fileInput          | A file upload control wizard
helpText           | Help text that can be added to an input form
numericInput       | A field to enter numbers
radioButtons       | A set of radio buttons
selectInput        | A box with choices to select from
sliderInput        | A slider bar
submitButton       | A submit button
textInput          | A field to enter text

每个控件的头两个参数都是一样的，都要求用字符串的数据类型作为参数值。第一个是控件的代码名称，这个代码用户看不到，但是会被server.R调用，要求非空；第二个是控件的名称，会作为控件的title显示出来，提示用户这是一个什么类型的控件，但是这个提示不是必须的，可以是个`""`空字符串。
各个不同的控件函数还有相应的定制的参数，有的需要指定数值或日期范围，有的是设定一个按钮，视具体的使用情况而定。
以滑条控件为例

``` r
sliderInput("range", 
            label = "Range of interest:",
            min = 0, max = 100, value = c(0, 100))
```

这段代码在app的页面中生成一个最小值为0,、最大值为100的滑条，默认滑条范围也是0到100。用户会在滑条上方看到一个小标题"Range of interest"，但是不会看到"range"字样。"range"的储存控件稍后会提到。

*更多的信息可见[Shiny Widgets Gallery](http://shiny.rstudio.com/gallery/widget-gallery.html)*

## 输入输出
### 数据流向
server.R的代码主体是这个样子：

``` r
shinyServer(function(input, output){
    # code
})
```

可以看到自定义函数的两个参数分别为input和output。
实际上，在RStudio读取ui.R中的控件部分时，就将控件函数中的第一个参数作为变量名储存在了`input`中。这里，可以将`input`视作一个environment，交互后用户设定的值会覆盖对应变量名下的默认值。
与`input`环境相对的是`output`，后者在server.R中生成，接着上面的code为例：

``` r
# server.R

shinyServer(
  function(input, output) {
    output$text1 <- renderText({ 
      paste("You have chosen a range that goes from", input$range[1], "to", input$range[2])
    })
  }
)
```

`input`和`output`两个环境可同时被ui.R和server.R读写。
`input`环境中的变量range即来自于前文中提到使用的`sliderInput()`函数。`renderText()`函数处理完输入数据以后，将运行结果赋值给`output`环境中的变量`text1`。
再回到ui.R中：

``` r
# ui.R

shinyUI(fluidPage(
  titlePanel("Demo"),
  
  sidebarLayout(
    sidebarPanel(
      sliderInput("range", 
        label = "Range of interest:",
        min = 0, max = 100, value = c(0, 100))
    ),
    
    mainPanel(
      textOutput("text1"))
  )
))

```

在`mainPanel()`函数中，我们添加了一段代码`textOutput("text1")`，`textOutput()`函数直接读取`output`环境中的`text1`变量(字符串格式调用)，并将其显示在浏览器页面的main panel对应区域中。

至此，shiny两个代码文件中最基本的数据流向已经很明确:

1. ui.R文件：控件代码获取用户的输入，并将数据存储在`input`环境中；
2. server.R文件：读取`input`环境中的输入变量，处理、运算，然后将运算结果存入`output`环境；
3. ui.R文件：读取`output`中的运算结果，转换成浏览器页面可用的图片和文字格式。

**ui.R中在两个环境的读写均通过函数和相应的字符串形式的变量名来执行，而server.R中，需要按找正常的R语言中的环境及变量的读写格式执行，如`input$var1`, `output$var2`**

另外要注意的是，如果需要调用helpers.R(`source("helpers.R")`)和同工作路径下的data文件夹及所属文件，也只能在server.R文件中调用。

### 输入输出的转换函数

也许你注意到了上一节的两段代码中提到的函数`renderText()`和`textOutput()`。前者在server.R中使用，统一用`render*()`的函数名；后者在ui.R中使用，统一用`*Output()`的函数名，就像一个从数据端到显示端压缩释放的过程。

*表三*

render 函数 | 创建
------------|-------------
renderImage | images (saved as a link to a source file)
renderPlot  | plots
renderPrint | any printed output
renderTable | data frame, matrix, other table like structures
renderText  | character strings
renderUI    | a Shiny tag object or HTML


*表四*

*Output函数        | 显示 
-------------------|-------------
htmlOutput         | raw HTML 
imageOutput        | image 
plotOutput         | plot 
tableOutput        | table
textOutput         | text
uiOutput           | raw HTML
verbatimTextOutput | text

两类函数均取用一个参数，`render*()`系用`{}`套用R中自带函数或者自定义函数，`*Output()系`调用`output`环境中的变量名(字符串形式)。

shiny的思路、构造基本如此。
接下来要研究的是如何搭建shiny的公司平台，如何让shiny应用的运算更快速、界面更优雅。

-------------------

## Reference

* [Shiny Tutorial](http://shiny.rstudio.com/tutorial/)