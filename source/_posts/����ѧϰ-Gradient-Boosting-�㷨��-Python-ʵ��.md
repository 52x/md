---
title: (机器学习) Gradient Boosting 算法与 Python 实现
date: 2016-11-25 14:06:34
tags: [Machine Learning, Math]
---

> 算法参考：
> - 李航：《统计学习方法》 `P151`
> - The Elements of Statistical Learning `P380`

## 算法

常见的 Gradient Boosting 采用误差平方和作为损失函数，每次迭代时，对预测值与实际值的差异进行拟合。而对于一般的损失函数形式，采用**损失函数的负梯度**在当前模型的值作为残差，进行下一步拟合。从偏差-方差分解的角度看，Boosting 主要关注降低偏差，因此能基于泛化能力相当弱的学习器构建出很强的集成。

<!-- more -->

### 误差平方和损失函数

训练集 $D = \{(x_1,y_1),...,(x_m,y_m)\}$ ，训练分为 $J$ 个步骤，第 $j$ 步时的总体模型为 $f_j(x)$，以下为训练步骤：

(1) 初始化 $f_0(x) = 0$ 

(2) 对 $j = 1,2,...,J$ 

(a) 对所有样本计算残差

$$r_{ji} = y_i - f_{j-1}(x_i), \space i = 1,2,...,m$$

(b) 拟合残差 $r_{ji}$ 训练一个回归树，记为 $T(x;\Theta_j)$ 

(c) 更新模型 $f_j(x) = f_{j-1}(x) + T(x;\Theta_j)$ 

(3) 得到回归问题提升树 

$$f(x) = \sum_{j=1}^J T(x;\Theta_j)$$

### 一般损失函数

(1) 初始化 

$$f_0(x) = \underset{c}{argmin} \sum_{i=1}^m L(y_i, c))$$

这一步估计一个使损失函数极小化的常数值 $c$ 。

(2) 对 $j = 1,2,...J$ 

(a) 以损失函数的负梯度作为残差估计值

$$r_{ji} = -[\frac{\partial L(y_i, f(x_i))}{\partial f(x_i)}]_{f(x) = f_{j-1}(x)}, \space i = 1,2,...,m$$

*例如：当 $L$ 为平方损失函数，上式等价于通常意义上的残差。当 $L$ 为3次方损失函数，上式等价于残差的平方。*

(b) 拟合 $r_{ji}$ 训练一个回归树，得到第 $j$ 棵树的叶结点区域 $R_{jh}, \space h = 1,2,...,H$

(c) 对 $h = 1,2,...,H$ 计算各个叶节点内使损失函数最小的常数值

$$c_{jh} = \underset{c}{argmin} \sum_{x_i \in R_{jh}} L(y_i, f_{j-1}(x_i) +c)$$

(d) 更新 $f_j(x) = f_{j-1}(x) + \sum_{h=1}^H c_{jh} I(x \in R_{jh})$ 

(3) 得到回归树

$$\hat f (x) = f_k(x) = \sum_{j=1}^J \sum_{h=1}^H c_{jh} I(x \in R_{jh})$$

### 应用到分类问题

由于Boosting可以选择损失函数，因此可以扩展到判别问题，例如可以将 $L$ 改为 logloss，而训练过程**仍采用回归树**作为弱学习器，例如

$$L = - \frac{1}{m} \sum_{i=1}^m [y_i log(f_{j-1}(x_i)) + (1 - y_i) log(1 - f_{j-1}(x_i))]$$
$$r_{ij} = \frac{\partial L}{\partial f_{j-1}(x_i)} = \frac{1}{m} (\frac{1 - y_i}{1 - f_{j-1}(x_i)} - \frac{y_i}{f_{j-1}(x_i)})$$

每次迭代时，使用回归树拟合上式的残差 $r_{ij}$ ，则最终的回归树输出为0到1之间的数值，可以理解为正例的概率。

## Python 实现

> 注：依赖自行开发的决策树算法库`ml_DTS`。

此处的实现以决策树作为弱分类器，拟合误差平方和损失函数，即 Gradient Boosting Decision Trees。代码参见 [GitHub](https://github.com/ferris-wufei/algorithm_ml/blob/master/ml_GBDT.py)。
