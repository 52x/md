layout: post
title: "利用 Python 解决问题"
toc: true
date: 2015-04-06 09:58:21
tags: 知识管理
category: Python
description: 前几天接了一个活，需要把 Excel 表格中的数据做成可以在微信内置浏览器中访问并且体验要好。我的选择是利用 Hexo 和 coding 搭一个博客用来展示。
---
Update:
今天发现，对 Excel 的读写可以利用 xlrd 这个库，做事之前真应该查一查有没有相关的库啊！！！
另外， Python 的库真是丰富多彩啊！
18:06 2015年4月10日

********
前几天接了一个活，需要把 Excel 表格中的数据做成可以在微信内置浏览器中访问并且体验要好。我的选择是利用 Hexo 和 coding 搭一个博客用来展示。

拿到表格后我就震惊了，尼玛，这么多的数据怎么才能放到小小的手机屏幕上。经过反复的挑选主题，最终选择了跟本博客相同的主题。这个主题在移动端的体验非常好。
![表格一](http://7u2qla.com1.z0.glb.clouddn.com/000.jpg)          

拿到的表格如图。为了让小小的屏幕装下这么长这么宽的表格，我将表格重新排列，原来的列变成行，原来的行变成列，并且每行只展示五个省份的数据。那么问题来了。这么多数据让我怎么写到文件里，简单的复制粘贴还不行，要命的是还需要遵守 MarkDown 的语法才行。

算了，光想有啥用，赶紧开干吧。于是我就一边是 Excel 一边是 MarkDown ，终于花了两个多小时把 Excel 中的数据搬到了 MarkDown 里。拿到手机里看一下效果，嗯，效果还不错。干了这么久，还是休息一下吧，于是我自然地去看了个电影。心里还想着就这个速度很快就能干完了吧。

第二天起来准备继续干昨天的活。等等！为什么昨天那张表有十几个专业，今天这张表有五十个专业。你他妈在逗我！

于是我再也不能淡定了，这要是全部手动去弄是要干到猴年马月啊。于是我想起了刚刚学的半斤八两的 Python ，东西学来不就是用的吗。于是就决定用 Python 来处理这些数据。于是就有了如下的解决方法。

首先把数据复制到一个文件里，像这样。

![数据文件](http://7u2qla.com1.z0.glb.clouddn.com/003.jpg)       

然后是这样的代码

```
with open('input.txt', encoding='utf-8') as f:  # 打开文件是要注意编码问题，编码问题是个大坑(-｡-;)
    s = f.readlines()
major = s[0].split()  # 把第一行的专业放到列表里


for i in range(0, len(s)):  # 把每的成绩包括省份放到列表里，同时为了保证数据不发生错位用 '\t' 分割字符
    if i != 0:
        s[i] = s[i].split('\t')
        s[i][len(s[i])-1] = s[i][len(s[i])-1].replace('\n', '')  # 去掉最后的换行符


for i in range(1, len(s)):  # 为了最后效果看起来美观一些把成绩里的空白的位置替换为 '000'
    for j in range(1, len(s[i])):
        if s[i][j] == '':
            s[i][j] = '000'
        elif s[i][j] == ' ':  # 处理第一个表格没有遇到空格的问题，第二三各表格就有空格问题了，所以加了这么一行
            s[i][j] = '000'


max = 0
for i in range(1, len(s)):  # 在下面的处理数据是老是发现列表下标越界，最后发现有些行最后一列没有数据就会导致这些没有数据的数据丢失
    if len(s[i]) > max:     # 所以就查找出最长的列表，所有小于这个长度的列表都应该补上少的 '000 000'
        max = len(s[i])
max = (max - 1)//2


scours = []
for i in range(1, len(s)):  # 取出数据中的最高最低分，并补上缺少的数据，以 '最高分 最低分'的格式存储在列表中
    scour = []
    scour.append(s[i][0])
    for j in range(1, len(s[i]), 2):  # 控制取出数据的那行代码
        scour.append(s[i][j] + ' ' + s[i][j+1])
    if len(scour) < max+1:  # 补上缺少的
        for k in range(0, max-len(scour)+1):
            scour.append('000 000')
    scours.append(scour)

for i in range(1, len(s)):  # 产生符合 MarkDown 语法的数据，并 print ，此处利用捕获异常并处理的方法处理最后一组数据的长度不确定的情况
    try:
        if i in range(1, len(s), 5):
            str = '|最高分 最低分' + '|' + s[i][0] + '|' + s[i+1][0] + '|' + s[i+2][0] + '|' + s[i+3][0] + '|' + s[i+4][0] + '|'
            str1 = '| - | - | - | - | - | - |'
            print(str)
            print(str1)
            for j in range(len(major)):
                str2 = '|' + major[j] + '|' + scours[i-1][j+1] + '|' + scours[i][j+1] + '|' + scours[i+1][j+1] + '|' + scours[i+2][j+1] + '|' + scours[i+3][j+1] + '|'
                print(str2)
    except:
            try:
                str = '|最高分 最低分' + '|' + s[i][0] + '|' + s[i+1][0] + '|' + s[i+2][0] + '|' + s[i+3][0] + '|'
                str1 = '| - | - | - | - | - |'
                print(str)
                print(str1)
                for j in range(len(major)):
                    str2 = '|' + major[j] + '|' + scours[i-1][j+1] + '|' + scours[i][j+1] + '|' + scours[i+1][j+1] + '|' + scours[i+2][j+1] + '|'
                    print(str2)
            except:
                try:
                    str = '|最高分 最低分' + '|' + s[i][0] + '|' + s[i+1][0] + '|' + s[i+2][0] + '|'
                    str1 = '| - | - | - | - |'
                    print(str)
                    print(str1)
                    for j in range(len(major)):
                        str2 = '|' + major[j] + '|' + scours[i-1][j+1] + '|' + scours[i][j+1] + '|' + scours[i+1][j+1] + '|'
                        print(str2)
                except:
                    try:
                        str = '|最高分 最低分' + '|' + s[i][0] + '|' + s[i+1][0] + '|'
                        str1 = '| - | - | - |'
                        print(str)
                        print(str1)
                        for j in range(len(major)):
                            str2 = '|' + major[j] + '|' + scours[i-1][j+1] + '|' + scours[i][j+1] + '|'
                            print(str2)
                    except:
                        str = '|最高分 最低分' + '|' + s[i][0] + '|'
                        str1 = '| - | - |'
                        print(str)
                        print(str1)
                        for j in range(len(major)):
                            str2 = '|' + major[j] + '|' + scours[i-1][j+1] + '|'
                            print(str2)

```
处理后的数据就是下面这个样子。

![处理后](http://7u2qla.com1.z0.glb.clouddn.com/004.jpg)                               

至于其他那两个表格也只是取出其最高最低分数。只需要把控制取出数据的那行代码改成如下就可以了。

```
    for j in range(1, len(s[i]), 3):  # 控制取出数据的那行代码
        scour.append(s[i][j] + ' ' + s[i][j+2])
```
和
```
    for j in range(1, len(s[i]), 4):  # 控制取出数据的那行代码
        scour.append(s[i][j] + ' ' + s[i][j+1])
```

![表格二](http://7u2qla.com1.z0.glb.clouddn.com/001.jpg)
![表格三](http://7u2qla.com1.z0.glb.clouddn.com/002.jpg)
到此为止，数据就处理完了。利用程序处理后的数据去跟手动打的数据一比较，尼玛果然手动不行有那么多地方都有差错。

总结一下，利用所学解决问题这真是好爽啊啊啊。
另外，代码写得丑望见谅。还有一点写文件没有搞定，当时赶时间就没有弄，得找个时间解决一下。