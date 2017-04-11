title: 阿里推荐大赛Season1--(3)
date: 2014-08-04 15:29:29
tags: [AliBigData, Python, Recommender]
categories: [AliBigData]
description: 接着上一篇文章，继续我们Season1的python之旅！其实我就是再次code review了。

---
接上篇，整理下*策略*跟*LR*部分的代码。
# 策略预测
今天在写这篇总结的时候，刚好数据心跳发了一篇他的心路历程[《成也solo，败也solo》][ref1]。他里面提到Season1的时候他靠纯规则刷到`7.45%`，我还是很佩服的，想想当时我们刷了个半死，才刷到`7.11%`。对于这个比赛，如果靠规则，那么比的就是天赋、分析能力；如果靠机器学习算法，那么比的就是理论基础、耐心和学习能力。

## 暴力存储
Season1的数据量不大，基本可以暴力读到内存操作，在使用策略进行预测的时候，我们使用了一种比较暴力、直观的数据结构。
```python
def get_date_of_all_action(path, start_date, end_date): # 可以指定时间段
    reader = csv.reader(open(path))
    user_brand = {}
    for user,brand,action,v_date in reader:
        now = get_date(v_date)
        if now >= start_date and now < end_date:
            delta_days = (now - date(2013,4,15)).days
            user_brand.setdefault(user, {})
            user_brand[user].setdefault(brand, {'click': [],
                                                'buy': [],
                                                'fav': [],
                                                'cart': []})
            if action == '0':
                user_brand[user][brand]['click'].append(delta_days)
            elif action == '1':
                user_brand[user][brand]['buy'].append(delta_days)
            elif action == '2':
                user_brand[user][brand]['fav'].append(delta_days)
            elif action == '3':
                user_brand[user][brand]['cart'].append(delta_days)
    return user_brand
```
```python
def get_date_of_all_action_to_brand(user_brand):
  brand_user = {}
  for user in user_brand:
    for item in user_brand[user]:
      brand_user.setdefault(item,{})
      brand_user[item][user] = user_brand[user][item]
  return brand_user
```
大体结构：`{u1: {b1: {'click': [t11,t12,...], 'buy': [t21,t22,...], ...}, ...}, ...}`，这样可以任意操作某个(u,b)的历史行为的时间序列（List存储），这里我们把时间转为了距离4月15日那天的天数，天数区间[0, 122]。函数`get_date_of_all_action_to_brand`仅仅是将u和b的键值关系对调一下，因为某些策略需要着重考虑品牌，这样的结构相对方便一些。

## 策略实例
Season1几乎都在围绕策略做，想了很多策略，效果最好的就是给四种行为一定的权重，再考虑时间衰减因素。除此以外，考虑很多特殊情况，用打补贴的方式添加到最终的预测结果中。比如下面这个看着让人捉急的策略：
```python
def get_brand_buy_and_click(user_brand):      # 传入上面的暴力结构
    user_buy = {}                             # 存放预测结果
    for u,b_dict in user_brand.items():
        for b,actions in b_dict.items():
            buy_arr = actions['buy']
            click_arr = actions['click']
            if buy_arr:
                last_buy = max(buy_arr)
                if last_buy > 105:            # 最后一次购买在最近半月
                    user_buy.setdefault(u, set())
                    user_buy[u].add(b)
                if last_buy > 45:
                    if len(set(buy_arr)) >= 2:                 # 在至少两个不同日期购买过
                        buy_np = np.array(buy_arr, float)
                        if buy_np.var() > 6:                   # 购买的时间序列的方差至少为6
                            user_buy.setdefault(u, set())
                            user_buy[u].add(b)
                if click_arr:                 # 考虑有购买行为(u,b)的点击行为
                    last_click = max(click_arr)
                    if last_click > last_buy and last_click > 110:
                    # 最后一次购买之后还有点击，并且点击行为的时间是在最近13天
                        user_buy.setdefault(u, set())
                        user_buy[u].add(b)
                    after_buy = [i for i in click_arr if i > last_buy]
                    if len(set(after_buy)) >= 3:
                    # 最后一次购买后，在至少3个日期再次发生点击行为
                        after_buy_np = np.array(after_buy, float)
                        if after_buy_np.var() > 4:
                            user_buy.setdefault(u, set())
                            user_buy[u].add(b)
    return user_buy
```
现在想想这些策略，感觉当时的我们想的有点复杂了。

# 浅谈LR
跟LR的狗血在之前Season1流水账部分就已经扯过了，这里主要记录下方法。这里再次感谢[oilbeater][]大神的[LR入门][ref2]，关于LR的简单介绍以及使用的算法包可以参考它。

在知道原理，又有算法包后，最关键的就是构建训练集跟测试集了。在这个数据集中，训练集由两部分组成：特征＋label，其中label是最后一个月有没有买，买了就标记为1，否则为0。测试集只需要相应的特征就行，根据训练的模型，可以预测某个(u,b)买还是不买。下面我们主要以构造训练集为例：
```python
def construct_train(out_path, feature, label, user_buy_count, brand_bought_count):
    writer = csv.writer(file(out_path, 'wb'))
    n_count = 0                       # 标记负样本(label=0)的序号
    writer.writerow(('label','click_score','fav_score','cart_score',
                     'buy_score','u_count','b_count'))
    for u,b_dict in feature.items():
        u_count = user_buy_count.get(u, 0)
        for b,f_arr in b_dict.items():
            b_count = brand_bought_count.get(b, 0)
            if (u,b) in label:
                writer.writerow((1, f_arr[0], f_arr[1], f_arr[2], f_arr[3],
                                 u_count, b_count))
            else:
                n_count += 1
                if n_count % 120 == 0:        # 类似于对负样本抽样，按模抽样
                    writer.writerow((0, f_arr[0], f_arr[1], f_arr[2], f_arr[3],
                                     u_count, b_count))
```
这里，feature是一个字典，结构：`{u1: {b1: [v1,v2,v3,v4], ...}, ...}`，List`[v1,v2,v3,v4]`表示四种行为的时间衰减评分（代码省略），这样有了feature后，也就有了有行为(u,b)的基本的四个特征。label是一个元组集合，即最后一个月有购买行为的(u,b)对集合。`user_buy_count`表示用户购买的品牌数，结构：`{u1: {b1: b1_count, ...}, ...}`；`brand_bought_count`表示品牌被购买的用户数，结构类似于`user_buy_count`，提取这两个特征的代码也省略。

Season1的LR中，我们就用了6个特征配合欠采样。当时由于时间关系，对LR的理解也不够深入，加上对特征也没放开手脚去提取，Season1的成绩也没有更进一步的提高，不过为Season2一开始琢磨LR铺平了道路。

# 写在Season1最后
官方设置Season1的目的应该是让我们入门、让我们熟悉数据，并非让我们尝试牛逼的算法，从这层目的来说，我们基本达到了要求。到Season2时，数据量大了，算法的威力基本就会显现出来，当时我们握着仅有的算法利器*LR*......


[oilbeater]:http://oilbeater.com/
[ref1]:http://www.yumumu.me/Ali_BigData/
[ref2]:http://oilbeater.com/%E9%98%BF%E9%87%8C%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%AF%94%E8%B5%9B/2014/04/04/the-bigdata-race-3.html
