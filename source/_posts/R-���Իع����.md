title: R - 多元线性回归检验
date: 2015/07/06
tags: [机器学习, 回归检验, R]

---

由天池大数据竞赛带着我步入了数据挖掘的道路，现在有点停不下来了，感觉前面是一座巨大的 Pizza 山，从上往下看是平囊，但是，如果你从侧边看确实有胸的，突的还是很高的，也就是从不同的维度有不同的信息，哈。

这次写的是上次线性回归的后半部分。这次主要写多元回归的检验，对于多元，我们要确定我们选择的多个自变量的相关性，数据的异常值等。进而可以更改自变量的个数或者修改数据。


<!-- more -->

###R 中回归相关函数概述###

>下面只是介绍一些回归中可能用到的函数，如果碍眼，可以直接跳过。

```
cor.text()   #相关性检验
lm()         #线性回归
summary()    #提取模型计算结果
confint()    #求参数置信区间
anova()      #方差分析
predict()    #求预测值和预测区间
step()       #多元，逐步回归法选择变量
residual()   #计算残差
update()     #更新模型
influence.measure() #计算模型中强影响点
vif（）       #检查共线性，方差膨胀因子
cook.distance(),diffits(),covration() #影响分析
gqtest()    #检验异方差
bptest()    #检验异方差
cor(),之后 kappa() #检查解释变量共线性
eigen()     #寻找共线性强的解释变
```

###回归过程一般性概述###

1. 模型假设：确定自变量和因变量有哪些
2. 模型参数：确定回归方程的参数，即为回归系数
3. 模型求解：利用样本数据，采用最小二乘法求出模型参数，统计上称为参数估计。然后进行回归方程的显著性检验，就是对假设来做假设检验，检验结果拒绝就认为方程是显著的。有三种方法：t 检验、F 检验、相关系数检验法。最后得到区间估计。

###R 多元线性回归###

####数据介绍####

对比与一元线性回归，多元的区别在于显著性检验要做两个：一个是方程的显著性检验，一个是系数的显著性检验。前者考察与所有的自变量之间是否存在相关关系；后者的假设是，关注的是每个自变量对因变量的影响是不是可有可无的。需要指出的是一元线性回归也需要或者也可以进行回归检验，但是由于它的自变量个数太少，想到对于多元的情况有差异。

还有一个区别是，在对系数做显著性检验后可能要剔除一些弱的变量，这就是变量选择与最优回归。

**这里的数据使用的是天池大数据竞赛的数据。**
关于数据的描述，请查看： [资金流入流出赛题与数据](http://tianchi.aliyun.com/competition/information.htm?spm=5176.100067.5678.2.X4lUty&raceId=3)。

由于此次决赛是在阿里的云平台 ODPS 上进行数据处理和提交，于是我用 SQL 语句处理下数据之后，将其复制到本地进行调优。数据如下所示：

![purchase_redeem_amt](/images/purchase_redeem_amt.png)

简单说下第一列是日期，官方提供的是 20130701 至 20140831 的数据，但是由于 2013 年那个时候余额宝才刚兴起不久，超高的收益吸引大量的资金流入，从而会导致 2013 年的数据波动非常大。所以我这里选择的是 2014 年 4 月份到 8 月份的数据，这几个月是数据比较稳定，用户沉淀到了一定的时期。中间的好几列都是我自己构造特征，与时间日期相关的特征，很明显这个资金的流入流出是有一定周期性。而最后的两列 `purchase_amt` 和 `redeem_amt` 分别表示的是每一天十万个用户总的申购和总的赎回量。单位是分。

###回归实现###

R 中实现多元回归的方式和上一篇一样，利用 lm() 函数。如下是实现多元回归的方法：

	#读入数据
	purchase_amt<-read.csv("C:/Users/lemon/Desktop/tianchi-second/lm/R_test_lm_purchase.csv")

	#线性回归方程公式
	purchase_formulaStr<-"purchase_amt~Monday+Tuesday+Wednesday+Thursday+Friday+Saturday+
						Sunday+head_one_week_month+head_two_week_month+tail_one_week_month
						+tail_two_week_month+holiday+workday+weekend"
	#回归函数
	purchase_model<-lm(as.formula(purchase_formulaStr), purchase_amt, interval="prediction")

执行完上面的代码之后，我们同样可以使用 summary() 函数查看回归方程：


	>summary(purchase_model)
	Call:
	lm(formula = as.formula(purchase_formulaStr), data = purchase_amt, 
    interval = "prediction")

	Residuals:
       Min         1Q     Median         3Q        Max 
	-276014530  -70950754   -3172133   63908552  334026127 

	Coefficients: (1 not defined because of singularities)
                      Estimate Std. Error t value Pr(>|t|)    
	(Intercept)          634934831   72543614   8.752 6.30e-15 ***
	Monday                83401543   77376745   1.078   0.2830    
	Tuesday              140072015   82285055   1.702   0.0909 .  
	Wednesday             71065064   82285055   0.864   0.3893    
	Thursday              26484419   81339807   0.326   0.7452    
	Friday              -101994686   81339807  -1.254   0.2120    
	Saturday             -41097748   34131003  -1.204   0.2306    
	Sunday                      NA         NA      NA       NA    
	head_one_week_month   66114099   28362979   2.331   0.0212 *  
	head_two_week_month  -69447159   36164290  -1.920   0.0569 .  
	tail_one_week_month   20723557   26627773   0.778   0.4377    
	tail_two_week_month -147115082   36164290  -4.068 7.91e-05 ***
	holiday              -17983886   53696807  -0.335   0.7382    
	workday              369252505   67565708   5.465 2.08e-07 ***
	weekend               51990045   60130437   0.865   0.3887    
	---
	Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

	Residual standard error: 111200000 on 139 degrees of freedom
	Multiple R-squared:  0.7772,    Adjusted R-squared:  0.7564 
	F-statistic:  37.3 on 13 and 139 DF,  p-value: < 2.2e-16

从上面的输出结果我们看出，有几个解释变量的 P 值超过了 0.05，，但是回归方程的检验统计量 F 对应的 P 值远远小于 0.05，修正的可决系数也较高，说明回归效果较显著。这似乎是矛盾的，产生这个矛盾的原因是这几个解释变量之间存在共线性。下面说明如何检验回归方程和选择最优解释变量组合。

###回归检验###

异常值往往会使回归模型不稳定，回归诊断主要内容有：残差分析、影响分析、异方差检验和共线性检验。

####残差分析####

得到的回归模型，进行显著性检验后，还要在做残差分析(预测值和实际值之间的差)，看假设是否成立。画出残差图。理论上，它是相互独立，近似服从 N(0,1) 分布。故残差散点应该随机分布在 -2 到 +2 的带状区域内。如下图：

![residuals](/images/residuals.jpg)

方差非齐性时，有时可以通过对因变量做变换来改善。常用：开方变换，对数变换，倒数变换和 BOX-COX 变换。

如下是检验残差的齐性：

	>y.rst<-rstandard(purchase_model)
	>y.fit<-predict(purchase_model)
	>plot(y.fit~y.rst)

输出的结果如下：

![plot_residual](/images/plot_residual.png)

显示的残差图较好的符合齐性。否则就要进一步处理。

####影响分析####

找出对回归结果影响很大的观测点就是影响分析。有四种度量方法：lm.influence()、cook.distance()、diffs()、covration()、influence.measures()。
当 Cook 距离 > 4/n 的点认为是强影响点，n 为样本容量。或者 covration 离 1 越远则影响越强。

这里我们以 influence.measure() 函数为例：

	>influcen.measures(purchase_model)

输出结果中以 "*" 标识的数据点为强影响点。

![influence_point](/images/influence_point.png)

####异方差检验####

对于多元线性回归模型，如果随机扰动项的方差并非是不变常数，则称为存在异方差。也就是说噪音的方差不是常数。如果存在异方差，则用传统的最小二乘法估计模型，得到的参数估计量不是有效估计量，甚至也不是渐近有效的估计量，此时也无法对模型参数进行有关显著性检验。

我们利用 bptest() 进行检验,使用前要先导入 lmtest 包。
	>library(lmtest)
	>bptest(purchase_model)
	  studentized Breusch-Pagan test

	data:  purchase_model
	BP = 25.403, df = 14, p-value = 0.03079	

p 值很小时我们认为存在异方差，此时该回归模型存在异方差。

修正异方差的一种方法：

	>lm.test<-lm(log(resid(purchase_model)^2)~Monday+Tuesday+Wednesday+Thursday+Friday+Saturday+Sunday+head_one_week_month+head_two_week_month+tail_one_week_month+tail_two_week_month+holiday+workday+weekend,data=purchase_amt)
	>lm.test2<-lm(purchase_amt~Monday+Tuesday+Wednesday+Thursday+Friday+Saturday+Sunday+head_one_week_month+head_two_week_month+tail_one_week_month+tail_two_week_month+holiday+workday+weekend,weights=1/exp(fitted(lm.test)),data=purchase_amt)
	>summary(lm.test2)
之后输出的结果可以和前一次的对比，同时可以再次检查异方差存在与否。

####共线性分析####

计算解释变量相关稀疏矩阵的条件数 k，k < 100 多重共线性程度很小，100 < k < 1000 较强， k > 1000 严重

	>xx<-cor(purchase_amt[,2:14]
	>kappa(xx)
	[1] 1.031462e+17

说明共线性比较强，此时我们就要进一步处理，要么手动的更改变量组合来看效果，或者利用函数 eigen() 寻找共线性强的解释变量组合，或者利用函数 step() 逐步回归法修正多重共线性。这里利用 step()。

	>purchase_model<-step(purchase_model)
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             

###补充###

异常点检测
利用 plot 函数，画出四张图，从中查看效果：

![residual_fitted](/images/resid_fitted.png)

![qqtest](/images/qqtest.png)

![scale_location](/images/scale_location.png)

![cook_distance](/images/cook_distance.png)

四幅图依次为：
1. 普通残差与拟合值的残差图
2. 正态 QQ 的残差图(若残差是来自正态总体分布的样本，则 QQ 图中的点应该在一条直线上)
3. 标准化残差开方与拟合值的残差图(对于近似服从正态分布的标准化残差，应该有 95% 的样本点落在 [-2,2] 区间内，这也好似判断异常点的直观方法)
4. cook 统计量的残差图(cook 统计量值越大的点越可能是异常值，但具体阈值较难判别)


###参考资料###

1. [回归分析入门](http://mayvi.blog.163.com/blog/static/200368352201310226952544/)
2. [如何衡量多元线性回归模型优劣](http://datakung.com/?p=50)
3.  [统计与 R 入门 || 回归分析](http://www.ryanzhang.info/archives/2014)
4. [R 语言 线性回归](http://my.oschina.net/u/1047640/blog/198956)
5. [R 语言进行简单多元回归的基本步骤](http://blog.sina.com.cn/s/blog_6ee39c3901017fpd.html)



                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            