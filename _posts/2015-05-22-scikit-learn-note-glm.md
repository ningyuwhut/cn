---
    layout: post 
    title: scikit-learn note GLM 
    categories: MachineLearning
    tags:
---

今天开始学习scikit-learn,把遇到的问题记录在这里。

The following are a set of methods intended for regression in which the target value is expected to be a linear combination of the input variables. 
第一句话就让我有点疑问，线性模型的定义不应该是参数的线性组合吗，最简单的线性回归也是关于输入变量 的线性组合，难不成下面都是这种最简单的线性组合。

线性回归学习的目标是拟合参数的线性模型，从而最小化目标变量的观测值和预测值之间的残差平方和（这个定义是不是有点狭隘？）。

Ordinary Least Squares
---

首先是Ordinary Least Squares。该方法就是最小二乘。不过该方法有一个局限性：输入变量之间必须是独立的。如果输入变量之间具有线性关系，那么会近似于一个奇异（sigular）矩阵，从而使结果对观测值中的随机误差十分敏感，从而具有较大的方差（ the least-squares estimate becomes highly sensitive to random errors in the observed response, producing a large variance）(不是很理解） 
对于design matrix的定义，这篇文章解释的比较清楚。 
输入变量之间的线性关系称为多重共线性（collinearity）。

Ridge Regression
---


ridge regression 是在OLS的基础上加了一个L2 norm正则项。

$\alpha$ controls the amount of shrinkage: the larger the value of \alpha, the greater the amount of shrinkage and thus the coefficients become more robust to collinearity.

LASSO
---

lasso算法产生稀疏解，减少结果所依赖的变量，所以Lasso及其变体是压缩感知的基础。 
Lasso算法其实就是使用L1先验作为正则项。

算法使用坐标下降和least angle regression进行求解。

As the Lasso regression yields sparse models, it can thus be used to perform feature selection
Lasso可以用于特征选择。
