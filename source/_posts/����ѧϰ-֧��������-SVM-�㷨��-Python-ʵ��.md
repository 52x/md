---
title: (机器学习) 支持向量机 SVM 算法与 Python 实现
date: 2016-11-27 12:56:59
tags: [Machine Learning, Math]
---

## 线性核

### 几何间隔与损失函数

对任意样本 $x_i, \space i = 1,2,...,m$ ，定义**函数间隔**为 $M_i = |w \cdot x_i + b|$ ，假设在决策边界上，函数间隔的值为 $M$ ，则 

$$M_i = |w \cdot x_i + b| = y_i (w \cdot x_i + b) \ge M, \space \forall i$$

沿着决策边界的单位法向量方向，在决策边界两侧的边界上，分别取2个点 $x_a, x_b$ ，则

$$2M = w \cdot (x_a - x_b) + (b - b) = ||w|| \cdot ||x_a - x_b|| \cdot \cos \pi/2$$
$$||x_a - x_b|| = \frac{2M}{||w||}$$

上式的左侧为**几何间隔，SVM的目标是几何间隔最大化**，即

$$\max_{w,b} ||x_a - x_b||$$

由于 $M$ 可通过 $w$ 与常数相乘任意放大，为了得到有意义的 $w,b$ 这里约束 $M=1$ ，因此问题等价于

$$\min_{w,b} \frac{1}{2} ||w||^2$$
$$s.t. \space y_i (w \cdot x_i + b) \ge 1, \space \forall i$$

上式即线性可分条件下的损失函数。

<!-- more -->

### 软间隔与松弛变量

放松线性可分的约束，即允许部分样本在集合间隔内、甚至在决策边界的反方向，此时几何间隔称为**软间隔**。

引入**松弛变量** $\xi_i$ ，并对全体样本的松弛变量的1阶或2阶矩进行约束，损失函数改为

$$\min_{w, b} \frac{1}{2} ||w||^2 + C \sum_{i=1}^m \xi_i^p, \ p=1,2$$
$$s.t. \space y_i (w \cdot x_i + b) \ge 1 - \xi_i; \space \xi_i \ge 0, \space \forall i, $$

其中参数 $p$ 控制违反 $y_i (w \cdot x_i + b) \gt 1$ 约束的成本。与Lasso类似，一阶正则化可以将无关特征的系数置零，因此**SVM无需做特征选择**，参见 [Variable importance from SVM](http://stats.stackexchange.com/questions/2179/variable-importance-from-svm)。

### 对偶问题

考虑含松弛变量的SVM对偶问题

$$\max_{\alpha, \mu} \min_{w,b,\xi} L(w, b, \alpha, \xi, \mu)$$
$$L(w, b, \alpha, \xi, \mu) = \frac{1}{2} ||w||^2 + C \sum_{i=1}^m \xi_i + \sum_{i=1}^m \alpha_i (1 - \xi_i - y_i (w \cdot x_i + b)) - \sum_{i=1}^m \mu_i \xi_i$$

其中 $\alpha_i \ge 0, \mu_i \ge 0$ 是拉格朗日乘子，首先对内层最小化问题求偏导得到

$$w \sum_{i=1}^m \alpha_i y_i x_i$$
$$0 = \sum_{i=1}^m \alpha_i y_i$$
$$C = \alpha_i + \mu_i$$

代入后得到外层最大化问题

$$\max_{\alpha} \sum_{i=1}^m \alpha_i - \frac{1}{2} \sum_{i=1}^m \sum_{j=1}^m \alpha_i \alpha_j y_i y_j x_i \cdot x_j$$
$$s.t. \space \sum_{i=1}^m \alpha_i y_i = 0; \space 0 \le \alpha_i \le C, \space \forall i$$

解求解后的系数为 $\alpha^{\ast}$ ，假设一个正的分量为 $\alpha_j^{\ast}>0$ ，则 $b^{\ast}$ 可以通过以下方式求解，或对所有正分量的以下结果求均值

$$b^{\ast} = y_j - \sum_{i=1}^m \alpha_j^{\ast} y_i (x_i \cdot x_j)$$

将 $\alpha, b$ 代入决策边界表达式得到决策函数

$$f(x) = sign(\sum_{i=1}^m \alpha_i^{\ast} y_i x_i \cdot x + b^{\ast})$$

### 核函数

在决策边界与线性偏差较大的情况下，可以将原输入空间 $\mathcal{X}$ 使用非线性的方式映射到更高维度的空间，在高维空间内应用线性决策边界。假设映射函数为

$$x \rightarrow \phi (x)$$

由对偶问题的表达式可知，优化过程只需要计算样本映射后的内积 $\phi (x_i) \cdot \phi (x_j)$ 。高维空间的内积计算成本较高，我们用一个函数替代它，简化*先映射、再计算内积*的过程

$$K(x_i, x_j) = \phi (x_i) \cdot \phi (x_j)$$

$K$ 称为核函数，因此损失函数变为

$$\max_{\alpha} \sum_{i=1}^m \alpha_i - \frac{1}{2} \sum_{i=1}^m \sum_{j=1}^m \alpha_i \alpha_j y_i y_j K(x_i, x_j)$$
$$s.t. \space \sum_{i=1}^m \alpha_i y_i = 0; \space 0 \le \alpha_i \le C, \space \forall i$$

解 $\alpha^{\ast}$ 后，对一个正分量计算 $b^{\ast}$ 

$$b^{\ast} = y_j - \sum_{i=1}^m \alpha_i^{\ast} y_i K(x_i, x_j)$$

决策函数变为

$$f(x) = sign(\sum_{i=1}^m \alpha_i^{\ast} y_i K(x, x_i) + b^{\ast})$$

## Python实现

这里并没有使用 Dual 问题的 SMO 算法，而是对 Primal 问题求解二次规划。需要注意的是，`cvxpy` 对向量和矩阵运算的语法与 Numpy 并不兼容，参见 [CVXPY FUNCTIONS](http://www.cvxpy.org/en/latest/tutorial/functions/)。由于 Primal 问题的表达式解释性不强，目前的 to-do 是修改为对偶问题的二次规划。实现代码参见 [GitHub](https://github.com/ferris-wufei/algorithm_ml/blob/master/ml_SVM.py)。
