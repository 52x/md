title: 阿里推荐大赛season1--(2)
date: 2014-07-31 22:31:31
tags: [AliBigData, Python, Recommender]
categories: [AliBigData]
description: Season1使用Python这把利器！用它数据处理，数据分析；用它画图；用它写算法。用它就意味着效率！

---
早在去年暑假在[无锡为立][weili]实习的时候，我就接触到了*推荐系统*相关知识，那时候把[《推荐系统实践》][ref1]一书研究了下，并照着实现了大部分算法，尤其是其中的*协同过滤*跟*LFM*模型，也是在那个时候接触到了Python。

这次比赛开始，我们就确定使用Python作为第一阶段的开发语言，因为方便直观，上手快，开发效率高。Season1前前后后，算上一些垃圾代码，一万行代码还是有的。仅以此文纪念我们曾经写过的Python代码，由于代码贴的有点多，我就分成了两篇文章。

# Teamwork
比赛开始第一天，我就兴冲冲地构建了本地代码仓库，又因为酷酷宇在东南大学，为了远程协作方便，我在[Bitbucket][]搭建了远程代码库，这些都是基于[Git][]。后来由于另外两个人对这些不太熟悉，就退而求其次地使用[Dropbox][]作为团队合作的工具了。仅以下图纪念下我维护过的远程代码仓库：![Bitbucket_pic](/image/bitbucket.png)

# 轻松开好头
不管是Season1还是Season2我们基本在开始前两天就搞定本地准备工作，其中最重要的当属构建本地训练、预测框架。
## 简单处理
原始数据的时间列是中文日期格式，利用正则先转成可处理的日期格式，假设数据是2013年的。
```python
def get_date(origin_date):
    month_day = re.findall(r"[\d|.]+", origin_date)
    month = int(month_day[0])
    day = int(month_day[1])
    return date(2013, month, day)
```
Season1开始两次线上测试是一个星期测试一次，后来改为了一天一测。在线上测试机会不多的时候，我们就得做好线下测试的工作。这次比赛很明显跟时间挂钩的，我们假定4个月的数据是5/6/7/8四个月，那么本地测试的时候，就用8月份（当时是用［7-15，8-15］）有购买行为的(u,b)来测试本地训练预测的效果。下面代码可以获取8月份有购买行为的(u,b)对，并保存在字典`test`中。
```python
def load_test_set(data_path):
    reader = csv.reader(open(data_path))    # 使用python的csv模块
    test = {}
    for user,brand,action,r_date in reader:
        now = get_date(r_date)
        if now >= date(2013,7,15) and now <= date(2013,8,15) and action = '1':
            test.setdefault(user, set())
            test.get(user).add(brand)
    return test
```
## 本地评估
有了本地测试集`test`后，我们就很容易计算本地的准确率、召回率以及F1评分了。
```python
def precision(predict, test):
    hit = 0
    total = 0
    for user,p_items in predict.items():
        test_items = test.get(user, '')
        if test_items:
            for item in p_items:
            if item in test_items:
                hit += 1
        total += len(p_items)
    return hit / (total * 1.0)
```
```python
def recall(predict, test):
    hit = 0
    total = 0
    for user,t_items in test.items():
        pred_items = predict.get(user, '')
        if pred_items:
            for item in t_items:
                if item in pred_items:
                    hit += 1
        total += len(t_items)
    return hit / (total * 1.0)
```
```python
def f1_score(P, R):
    if P == 0 and R == 0:
        return 0
    return (2 * P * R) / (P + R)
```
其中，`predict`跟`test`是一样的数据结构：`{u1: set(), u2: set(), ...}`，用集合可以方便去除重复。

## 结果提交
最终提交结果时，是需要写到一个txt文件里面，并且遵循[官方要求格式][ref2]，下面函数将上面的`predict`写入到指定文件`file_path`中。
```python
def write_into_file(predict, file_path):
    out = file(file_path, 'w')
    for user,buy_set in predict.items():
        row = '%s\t' % user
        row += (','.join(buy_set))
        out.write('%s\n' % row)
    out.close()
```

# 画图分析
在进行数据分析时，数据可视化无疑可以提高效率，帮助发现蕴含在数据内部的规律。为了保持语言的一致，我们使用了python的*Matplotlib*模块。后来发现，我们当时有点耿直了，完全可以使用其他画图软件，而且画图没有给我们带来实质性的帮助，使用*Matplotlib*画图对我们来说是一次性价比不高的选择。Anyway，贴上几个当时我们参考的教程：
* [官方手册][ref3]
* [快速入门（英文）][ref4]
* [快速入门（中文）][ref5]

我们当时对四种行为每个月的分布、每个用户123天的行为分布、本地能命中的(u,b)的历史行为分布等等都画图进行了分析。比如下图：![hit_picture](/image/hit.png)就是某个本地能命中的(u,b)，红圈表示点击，蓝方块表示购买，紫三角表示收藏。为了批量画图方便，我们没有统一坐标轴比例跟长度，也没有画上图标解释。

# 未完待续
这篇简单介绍一些准备工作，下一篇记录下我们曾经做过的*策略*跟*LR*。



[weili]:http://58.214.255.186:3000/
[ref1]:http://book.douban.com/subject/10769749/
[ref2]:http://102.alibaba.com/competition/addDiscovery/gameTopic.htm
[ref3]:http://matplotlib.org/users/pyplot_tutorial.html
[ref4]:http://www.loria.fr/~rougier/teaching/matplotlib/#quick-references
[ref5]:http://reverland.org/python/2012/09/07/matplotlib-tutorial/
[Bitbucket]: http://bitbucket.org
[Git]: http://git-scm.com/
[Dropbox]:https://www.dropbox.com
