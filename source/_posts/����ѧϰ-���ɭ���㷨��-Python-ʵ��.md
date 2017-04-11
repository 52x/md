---
title: (机器学习) 随机森林算法与 Python 实现
date: 2016-11-27 09:04:54
tags: [Machine Learning, Math]
---

## 算法

整体思想：将样本和特征随机化，构建多个树。预测结果：所有树预测结果的投票或均值。从偏差-方差分解的角度看，随机森林关注降低方差，因此在不过度剪枝的决策树、神经网络等易受样本干扰的学习器上效用更为明显。

(1) 训练集 $D = \{(x_1,y_1),...,(x_m,y_m)\}$ 的特征数量为 $n$ 。对于训练轮数 $t = 1,2,...,T$

(a) 对原样本bootstrap有放回抽样，产生新的样本，用于训练决策树 $h_t(x)$ 

(b) 在每一步划分，从全体 $n$ 个特征中随机选取 $p$ 个特征作为候选，并选取其中最优的一个作为划分条件

(c) 训练的树不必剪枝，不过流行的算法库内有控制最大深度的选项；候选特征数量的通常为 $p = \sqrt n$ 

(2) 输出模型表达式：

- 判别问题 $H(x) = \underset{y \in \mathcal{Y}}{arg max} \sum_{t=1}^T I(h_t(x) = y)$
- 回归问题 $H(x) = \frac{1}{T} \sum_{t=1}^T h_t(x)$ 

<!-- more -->

## 衍生

在随机森林中，当 $p = n$ 时，算法又称为`Bagging`，即每棵树的每一次划分从所有特征中选择。

在随机森林的每棵树的每一次划分，对每个特征随机选取划分点，然后在随机划分的结果中选择最优的特征，称为`Extremely Randomized Trees`，详情参见 [sklearn 相关文档](http://scikit-learn.org/stable/modules/ensemble.html#extremely-randomized-trees)。这是进一步减少方差的做法，在变量较多情况下的结果往往优于随机森林。

## Python 实现 

> 注：依赖自行开发的决策树算法库`ml_DTS`。

代码参见 [GitHub](https://github.com/ferris-wufei/algorithm_ml/blob/master/ml_RF.py)。
