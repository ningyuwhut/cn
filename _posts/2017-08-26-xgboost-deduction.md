---
  layout: post
  title: xgboost推导
  categories: MachineLearning
  tags: MachineLearning
---

该文为阅读陈天奇的xgboost tutorial “Introduction to Boosted Trees ”的笔记，比较简略。这篇tutorial写得很详细，也比较好懂，推荐阅读。

xgboost的目标函数:

$$
\sum_{i=1}^n l(y_i,\hat y_i)+\sum_k \Omega(f_k), f_k \in F
$$

因为我们这里的f是一棵棵树，所以我们没法使用SGD这种优化算法。解决方法称为加性训练（Additive Training）

具体就是从一个常函数开始，每次学习一个新的函数（即一棵新的树）

$$
\hat y_i^{(0)} = 0 \\
\hat y_i^{(1)} = f_1(x_i) = \hat y_i^{(0)} +f_1(x_i) \\
\hat y_i^{(2)} = f_1(x_i) + f_2(x_i) = \hat y_i^{(1)} +f_2(x_i) \\
\hat y_i^{(t)} = \sum_{k=1}^t f_k(x_i) = \hat y_i^{(t-1)} +f_t(x_i) \\
$$

那么每次我们怎么来学习新函数f呢
当然是优化目标函数了

$$
Obj^{(t)} = \sum_{i=1}^n l(y_i,\hat y_i^{(t)})+\sum_{i=1}^t \Omega(f_i) \\
	  = \sum_{i=1}^n l(y_i, \hat y_i^{t-1} + f_t(x_i) ) + \Omega(f_t) + constant
$$

以平方损失函数来说:

$$
Obj^{(t)} = \sum_{i=1}^n (y_i - (\hat y_i^{t-1} + f_t(x_i) ))^2 + \Omega(f_t) + constant \\
	  = \sum_{i=1}^n [2(\hat y_i^{t-1} - y_i)f_t(x_i) + f_t(x_i)^2 ] + \Omega(f_t) + constant \\
$$

注意从第一步到第二步的推导，这里$$(y_i-\hat y_i^{t-1})^2$$这一项因为跟$$f_t(x)$$无关，所以被纳入了constant项

这是平方损失的情况，推导起来还比较简单，但是对于更一般的损失函数就不那么好推了。
那么对于更一般的损失函数使用什么方法比较好呢？答案就是`泰勒公式`

$$
f(x+\Delta x)=f(x)+f'(x)\Delta x + 1/2 f{''}(x)\Delta x^2
$$

分别定义一阶导和二阶导如下:

$$
g_i = \partial _{\hat y^{(t-1)}} \quad  l(y_i,\hat y^{(t-1)}) \\
h_i = \partial_{\hat y^{(t-1)}}^2 \quad  l(y_i,\hat y^{(t-1)}) \\
$$

所以目标函数变成了

$$
Obj^{(t)} = \sum_{i=1}^n [l(y_i, \hat y_i^{(t-1)}) + g_i f_t(x_i) + 1/2 h_i f_t^2(x_i)  ] +\Omega(f_t)+constant
$$

当损失函数是平方损失时,有

$$
g_i = 2(\hat y_i^{(t-1)} - y_i) \\
h_i = 2
$$

如果去掉常数项，那么损失函数就是如下形式：

$$
Obj^{(t)} = \sum_{i=1}^n [ g_i f_t(x_i) + 1/2 h_i f_t^2(x_i)  ] +\Omega(f_t) \\
g_i = \partial _{\hat y^{(t-1)}} \quad  l(y_i,\hat y^{(t-1)}) \\
h_i = \partial_{\hat y^{(t-1)}}^2 \quad  l(y_i,\hat y^{(t-1)}) \\
$$

推导完损失函数之后，我们再回到树的定义。

树可以定义成一个叶子节点下标到叶子节点得分的这么一个映射函数:

$$
f_t(x)=w_q(x), w\in R^T, q:R^d \to {1,2,...,T}
$$

其中，T是叶子节点个数，d是样本的维度，w即叶子节点的得分,q(x)是将样本映射到叶子节点下标的映射函数

树的定义有了，树的复杂度该怎么定义呢？
一种方法就是考虑叶子节点个数和所有叶子节点的得分的L2范数

$$
\Omega(f_t)=\gamma T + 1/2 \lambda \sum_{j=1}^T w_j^2
$$

将刚才约定的树的定义和复杂度的定义与之前的目标函数结合起来,我们可以按照叶子节点来重新组织目标函数

$$
Obj^t = \sum_{i=1}^n [ g_i f_t(x_i) + 1/2 h_i f_t^2(x_i)  ] +\Omega(f_t) \\
       = \sum_{i=1}^n [ g_i w_q(x_i) + 1/2 h_i w_q^2(x_i)  ] + \gamma T + 1/2 \lambda \sum_{j=1}^T w_j^2 \\
       = \sum_{j=1}^T [ (\sum_{i\in I_j }g_i) w_j + 1/2(\sum_{i\in I_j} h_i + \lambda)w_j^2]  + \gamma T
$$


下面先介绍两个结论

$$
argmin_x Gx+1/2Hx^2 = -G/H, H>0 \\
min_x Gx+1/2Hx^2 = -1/2G^2/H
$$

结合我们上面的损失函数，我们给出G和H的定义:

$$
G_j=\sum_{i \in I_j}g_i, H_j=\sum_{i\in I_j}h_i
$$

目标函数变成:

$$
Obj^{(t)}= \sum_{j=1}^T [ (\sum_{i\in I_j }g_i) w_j + 1/2(\sum_{i\in I_j} h_i + \lambda)w_j^2]  + \gamma T \\
	 = \sum_{j=1}^T [ G_j w_j + 1/2 (H_j +\lambda)w_j^2] + \gamma T
$$

利用上面的两个结论，可以得出：
如果树的结构已经确定的情况下，每个叶子节点的最优权重为

$$
w_j^{\star}=-\frac{G_j}{H_j+\lambda}
$$

此时，损失函数为
$$
Obj=-1/2\sum_{j=1}^T \frac{G_j^2}{H_j+\lambda}+\gamma T
$$

该值越小表明树的结构更优。

那么，在实际应用中，树可以有很多种结构，怎么选择一个最优的呢？
我们可以采用贪心的选择:
在对节点进行分裂时，我们获得的gain如下:

$$
    Gain=1/2[\frac{G_L^2}{H_L+\lambda} + \frac{G_R^2}{H_R+\lambda} - \frac{(G_L+G_R)^2}{H_L+H_R+\lambda}]-\lambda
$$

选择Gain最大的值作为分裂点

具体点，就是找到每个特征的最佳分裂点（gain最大的点），然后再找出所有特征的最佳分裂点。在寻找特征的最佳分裂点时需要对特征按取值进行排序。

对于深度为K的树，这一过程的复杂度为$$O(ndklon n)$$,其中,$nlog n$是排序的时间复杂度，d是特征数，k是深度

在tutorial中作者还特意提到了离散特征的处理方法，那就是将离散特征进行one-hot，这样就可以和连续特征一样对待了.

那么，如何对树进行剪枝呢？

注意，上面的Gain的计算公式中需要减去$\lambda$，这样，Gain就有可能为负数。
那么，就有了以下的剪枝策略:
1. Pre-Stoping

如果最优分裂点的Gain为负数则停止进行分裂

但是这种分裂有可能对将来的分裂有益

2.Post-Pruning
先将树增长到最大深度，然后递归地对Gain为负数的叶子节点进行剪枝




参考

1.https://homes.cs.washington.edu/~tqchen/pdf/BoostedTree.pdf
