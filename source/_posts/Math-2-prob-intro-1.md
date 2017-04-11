title: 概率复习 - Part I
date: 2016-3-6 20:34:43
categories: Mathématiques|数字游戏
tags: [数学, 概率, MIT]
---

以下内容取自MIT的在线课程[Introduction to Probability - The Science of Uncertainty](https://courses.edx.org/courses/course-v1:MITx+6.041x_3+2T2016/)。
Exam1 完成前，对前四个单元的复习。
<!--more-->

## 概率模型与公理(Probability Models and Axioms)
### 样本空间(Sample Space)
所有可能性的集合。这个集合必须满足三个条件：
* 集合中的因素互相排斥
* 集合穷尽所有的可能性
* 有正确的颗粒度

### 古典方法(Uniform Probability Law)
均匀分布(Uniform Distribution)的特征模型里，每个观测点出现的几率都均等。对应到古典方法里：
* 离散事件: $P(A) = \frac{k}{n}$, $\Omega$包含$n$个点，事件$A$包含$k$个点 
* 连续事件: $P(A) = \frac{Area\ of\ A}{Area\ of\ \Omega}$

### 概率公理及推论(Axioms and Consequences)
#### 基本公理及推论

公理(Axiom)                                               | 推论(Consequences)
----------------------------------------------------------|-------------
$P(A) \geq 0$                                             | $P(A) \leq 1$
$P(\Omega) = 1$                                           | $P(\emptyset) = 0$
$A \cap B = \emptyset \implies P(A \cup B) = P(A) + P(B)$ | $P(\\{s_1, s_2, \dots , s_n\\}) = \sum_1^n s_k$, for disjoint events $\\{s_k\\}$

更多的推论:
* $P(A \cup A^c) = \Omega$
* $P(A \cap A^c) = \emptyset$
* if $A \subset B$ then $P(A) \leq P(B)$
* $P(A \cup B \cup C) = P(A) + P(A^c \cap B) + P(A^c \cap B^c \cap C)$
* generally:
	* $P(A \cup B) = P(A) + P(B) - P(A \cap B)$ where $P(A \cap B) \geq 0$
	* $P(A \cup B) \leq P(A) + P(B)$

#### 可数可加性
如果$A_1, A_2, A_3, \dots$是**不相交**事件的无限**序列**，那么他们满足：
$$P(A_1 \cup A_2 \cup A_3 \cup \dots) = P(A_1) + P(A_2) + P(A_3) + \dots$$

### 概率计算的顺序
* 确定样本空间
* 确定概率方法
* 确定目标事件
* 计算目标事件概率

## 条件概率与贝叶斯公式(Conditioning and Bayes' Rule)
已知事件$B$，求事件$A$发生的概率：
$$P(A) = \frac{P(A \cap B)}{P(B)} \  (only when P(B) > 0)$$

条件概率的性质与普通概率一样。

### 乘法公式(The Multiplication Rule)
$$P(A|B) = \frac{P(A \cap B)}{P(B)}$$
那么我们可以推导出：
$
\begin{aligned}
P(A \cap B) & = P(B)P(A|B) \\\
& = P(A)P(B|A)
\end{aligned}
$

### 全概率公式(Total Probability Theorem)
将样本空间分割到$A_1, A_2, A_3, \dots$等$n$个空间中。
每个$A_i$中事件$B$的概率为$P(B|A_i)$ 。
那么，我们可以通过全概率公式来计算事件$B$在$\Omega$中的概率：
$$P(B) = \sum_{i=1}^{n} P(B|A_i)P(A_i)$$

### 贝叶斯公式(Bayes' Rule)
基于乘法公式和全概率公式我们可以得到贝叶斯公式：
$$P(A_i|B) = \frac{P(A_i)P(B|A_i)}{\sum_j P(A_j)P(B|A_j)}$$

## 独立性
两个事件是否独立最基本的验证方式就是检查$P(A|B) = P(A)$是否成立。如果等式成立，则$A$, $B$两事件相互独立。
上面这个等式等价于$P(A \cap B) = P(A)P(B)$

推而广之，如果事件$A_1, A_2, A_3, \dots, A_n$中存在任意$1 \leq i < j < k \geq n$上均满足：
$$P(A_i \cap A_j \cap \dots \cap A_k) = P(A_i)P(A_j) \dots P(A_k)$$
则以上所有事件**互相独立**。

条件概率间的独立性检验同样适用以上规则。

