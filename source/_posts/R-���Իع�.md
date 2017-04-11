title: R 线性回归(Linear Regression)
date: 2015/07/01
tags: [机器学习, 线性回归, R]

---
线性回归是利用数理统计中的回归分析，来确定两种或者两种以上变量相互依赖的定量关系的一种统计分析方法。分析按照自变量和因变量之间的关系类型，可分为线性回归分析和非线性回归分析。

回归分析中，只包括一个自变量和一个因变量，且二者的关系可用一条直线近似表示，这种回归分析称为一元线性回归分析。如果回归分析中包括两个或两个以上的自变量，且因变量和自变量之间是线性关系，则称为多元线性回归分析。

对于线性回归、逻辑回归等概念查看链接：[http://blog.csdn.net/viewcode/article/details/8794401](http://blog.csdn.net/viewcode/article/details/8794401).

<!-- more --> 

###简单线性回归###
简单的线性回归涉及到两个变量：一个是解释变量，称为 x；另外一个为被解释变量，称为 y。简单的线性模型如下所示：

![lm](/images/simple_lm_expression.gif)

其中 β0 和 β1 是回归参数，εi表示误差。求解这些参数可以利用的方法有：最小二乘法和梯度下降法。而两者思路比较相似，都是通过求导来求损失函数的最小值，即是估计值与实际值的总平方差尽量小。不同点在于最小二乘法直接对下面的式子求导找出全局最小，是非迭代的；而梯度下降是一种迭代法，先给点一个参数，然后找一个下降最快的方向调整该参数，若干次之后找到局部最小。因此，梯度下降法的缺点是最小点收敛比较快，并且对初始点的选取比较敏感，其改进的地方真是在于此。

最小二乘法来求解其中的参数：

![least_square_method](/images/least_square_method.gif)

它的思想如上面说的，让预测数据和实际的数据之间的差平方和最小，当满足这个条件时，可求得回归的参数。对着理解这个需要一点数学理论。
最小二乘法更深入的理解，请查看：
[线性回归-最小二乘法](http://sbp810050504.blog.51cto.com/2799422/1269572)。

在 R 中，你可以通过函数 lm() 进行线性回归。lm() 的函数说明如下：

```
lm(formula,data,subset,weights,na.action,method='qr',
   model=TRUE,x=FALSE,y=FALSE,qr=TRUE,singular.ok=TRUE,contrasts=NULL,offset,...)
```

参数 formula 模型公式，例如 y ~ x。公式中的波浪号（～）左侧是响应变量，右侧是预测变量。函数会估计系数 β0 和 β1，分别以截距（intercept）和 x 的系数表示。

首先来看一个简单的例子：


	#读入数据
	x<- c(0.10,0.11,0.12,0.13,0.14,0.15,0.16,0.17,0.18,0.20,0.21,0.23)
	y<-c(42.0,43.5,45.0,45.5,45.0,47.5,49,53,50,55,55,60)
	#绘出 x 与 y 的散列图
	plot(y~x)


散列图如下：

![](/images/lm_xy.png)

从上图中我们可以看出 y 和 x 存在线性相关性，可以进行线性回归分析：

>注：如下代码的起始位置有个字符 ">"，它是 R 是每行的提示符，为了防止将我们输入的和它输出的弄混淆，所有有些地方的代码会以 ">" 开始。

```
	>model<-lm(y~1+x) #1 可以不写，代表截距项;如果写 -1 表示强制过原点
	>summary(model) #如下是此句输出
	Call:
	lm(formula = y ~ 1 + x)

	Residuals:
    Min      1Q  Median      3Q     Max 
	-2.0431 -0.7056  0.1694  0.6633  2.2653 

	Coefficients: #估计值	  #标准差  #t 值	 #p 值
            	Estimate Std. Error t value Pr(>|t|)    
	(Intercept)   28.493      1.580   18.04 5.88e-09 ***
	x            130.835      9.683   13.51 9.50e-08 ***
	---
	Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

	Residual standard error: 1.319 on 10 degrees of freedom
	Multiple R-squared:  0.9481,    Adjusted R-squared:  0.9429 
	F-statistic: 182.6 on 1 and 10 DF,  p-value: 9.505e-08
```

我们通过 P 值(就是上面的 pr 那一列)来查看对应的解释变量 x 的显著性，通过将 p 值与 0.05 进行比较，若改值小于 0.05，就可以说该变量与被解释变量存在显著的相关性。

Multiple R-squared 和 Adjusted R-squared 这两个值，就是我们常称为”拟合优度“和”修正的拟合优度“，是指回归方程对样本的拟合程度，这里我们可以看到，修正的拟合优度为 0.9429，表示拟合程度超过五成，这个值越高越好。

最后，看下 F-statistic，也就是常说的 F 统计量，也称为 F 检验，常用语判断方程整体的显著性实验，其 p 值为 9.505e-08，显然小于 0.05，我们可以认为方程在 P=0.05 的水平上是通过显著性检验的。

总结起来，我们检验得出的线性回归的性能的参数的步骤如下：
1. T 检验中查看解释变量的显著性；
2. R-squared 查看方程的拟合程度；
3. R 检验查看方程的整体显著性。

关于这个更多的可以查看：
[http://f.dataguru.cn/thread-14689-1-1.html](http://f.dataguru.cn/thread-14689-1-1.html)，评论中也有更多的解释。

从上面我们看出我们的线性回归效果不错，那么我们可以利用拟合方程进行分类，或者预测。

	>newX<data.frame(x=0.16) #这里是输入数据，必须是frame类型
	>predict(model,newdata=newX,interval="prediction",level=0.95)#interval=”prediction“ level指定预测的置信区间
	   fit      lwr      upr
	1 49.42639 46.36621 52.48657

lm 中 formular 的语法：

	语法                    模型                              备注    
	Y ~ A                  Y = β0 + β1A                      带y截距的直线    
	Y ~ -1 + A             Y = β1A                           没有截距的直线，即强制通过原点    
	Y ~ A + I(A^2)         Y = β0 + β1A + β2A2               多项式模型。注意函数I()允许常规数学符号    
	Y ~ A + B              Y = β0 + β1A + β2B                没有交互项的A和B一阶模型    
	Y ~ A:B                Y = β0 + β1AB                     仅有交互项的A和B一阶模型    
	Y ~ A*B                Y = β0 + β1A + β2B + β3AB         A和B全一阶模型，等同于：Y ~ A + B + A:B    
	Y ~ (A+B+C) ^2         Y = β0+ β1A + β2B + β3C + β4AB + β5BC + β6AC

###多元线性回归###

多元线性回归一般是一个因变量对应多个自变量，其一般形式如下所示：
 
![](/images/multiple_lm_expression.gif)

其中 β0 是截距项，ε 称为噪音。通常 ε～n(0,σ^2)。R 中进行多元回归的方式如下：

	> cement <- data.frame(
	+ x1=c(7,1,11,11,7,11,3,1,2,21,1,11,10),
	+ x2=c(26,29,56,31,52,55,71,31,54,47,40,66,68),
	+ x3=c(6,15,8,8,6,9,17,22,18,4,23,9,8),
	+ x4=c(60,52,20,47,33,22,6,44,22,26,34,12,12),
	+ y=c(78.5,74.3,104.3,87.6,95.9,109.2,102.7,72.5,93.1,115.9,83.8,113.3,109.4))
 
	> lm.sol <- lm(y ~ x1 + x2 + x3 + x4,data=cement)
	> summary(lm.sol)
 
	Call:
	lm(formula = y ~ x1 + x2 + x3 + x4, data = cement)
 
	Residuals:
    Min      1Q  Median      3Q     Max 
	-3.1750 -1.6709  0.2508  1.3783  3.9254 
 
	Coefficients:
                  Estimate  Std.Error t value Pr(>|t|)  
	(Intercept)  62.4054    70.0710   0.891   0.3991  
	x1            1.5511     0.7448   2.083   0.0708 .
	x2            0.5102     0.7238   0.705   0.5009  
	x3            0.1019     0.7547   0.135   0.8959  
	x4           -0.1441     0.7091  -0.203   0.8441  
	---
	Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
 
	Residual standard error: 2.446 on 8 degrees of freedom
	Multiple R-squared:  0.9824,    Adjusted R-squared:  0.9736 
	F-statistic: 111.5 on 4 and 8 DF,  p-value: 4.756e-07

代码中我们在 lm 函数中通过 y ~ x1 + x2 + x3 + x4 公式就可以指定多元线性回归。

从上面的 T 检验的 P 值可以看出，自变量和因变量之间的相关性不是很显著，我们需要剔除不显著项，此处数据有些不好，我们就以剔除 x3 为例。

这里涉及到一个问题，如何进行回归诊断。这个下一篇再写。


###逻辑回归###

逻辑回归的模型是一个非线性模型，利用了 sigmod 函数。又称为逻辑回归函数。除了 sigmoid 映射函数关系，其他的步骤，算法都是线性回归。逻辑回归利用非线性 sigmoid 函数可以处理 0/1 分类问题。

在 R 中逻辑回归使用函数 glm(),你可以在 R 控制台中输入 help(glm) 查看关于该函数的说明文档。

R 实现的一个实例：

	counts <- c(18,17,15,20,10,20,25,13,12)
	outcome <- gl(3,1,9)
	treatment <- gl(3,3)
	print(d.AD <- data.frame(treatment, outcome, counts))
	glm.D93 <- glm(counts ~ outcome + treatment, family = poisson())
	anova(glm.D93)
	summary(glm.D93)