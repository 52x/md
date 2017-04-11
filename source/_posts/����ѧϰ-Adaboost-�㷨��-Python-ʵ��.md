---
title: (机器学习) Adaboost 算法与 Python 实现
date: 2016-11-25 16:01:45
tags: [Machine Learning, Math]
---

## 算法

> 算法参考：
>  - 李航《统计学习方法》 `P139`

Adaboost 可以看做损失函数为 $L(f(x),y) = \sum_i^m  \exp(-f(x_i) \cdot y_i)$ 的 Gradient Boosting。

训练集 $D = \{(x_1, y_1), (x_2, y_2), ..., (x_m, y_m)\}$ ，其中 $x_i \in \mathbb{R}^n$ 且 $y_i \in \{-1, +1\}$ 。

(1) 初始化训练数据的权值分布

$$D_1 = (w_{1,1}, ..., w_{1,i}, ..., w_{1,m}), \space w_{1i} = \frac{1}{m}, \space i = 1,2,...,m$$

<!-- more -->

(2) 对 $j = 1, 2, ..., J$ 循环次数：

(a) 使用具有权值分布 $D_j$ 的训练数据学习，得到基本分类器

$$G_j(x): \mathcal{X} \rightarrow \{-1, +1\}$$

(b) 计算 $G_j(x)$ 在训练数据集上的分类误差率，注意这里使用**加权误差率**

$$e_j = P(G_j(x) \ne y_i) = \sum_{i=1}^N w_{ji} I(G_j(x_i) \ne y_i)$$

如果 $e_j \ge 0.5$ 返回*步骤 (a)*。

(c) 计算分类器 $G_j(x)$ 的系数

$$\alpha_j = \frac{1}{2} log \frac{1 - e_j}{e_j}$$

错误率越低，系数越大。

(d) 更新训练数据集的权值分布

$$D_{j+1} = (w_{j+1,1}, ..., w_{j+1,i}, ..., w_{j+1,m})$$
$$w_{j+1,i} = \frac{w_{j,i}}{Z_j} exp(-\alpha_j y_i G_j(x_i)), \space i = 1,2,...,m$$

这里 $Z_j$ 是规范化因子，使得 $D_{j+1}$ 之和为1。对于预测正确的样本，减小权值，否则增大权值。

(3) 构建基本分类器的线性组合，得到最终分类器

$$G(x) = sign(\sum_{j=1}^J \alpha_j G_j(x))$$

## Python 实现 

> 注：依赖自行开发的决策树算法库`ml_DTS`。

此处的实现以决策树作为弱分类器，代码参见 [GitHub](https://github.com/ferris-wufei/algorithm_ml/blob/master/ml_Adaboost.py)。
