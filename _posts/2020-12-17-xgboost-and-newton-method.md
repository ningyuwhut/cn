---
  layout: post
  title: 牛顿法和Xgboost
  categories: MachineLearning
  tags:
--- 

1.首先，介绍一下牛顿法

2.然后，介绍下Xgboost的推导及和牛顿法的关系


以函数$$f:R->R$$为例。假设该函数是连续可导的，那么在$$x=x_0$$经过泰勒展开后,有如下等式:

$$
f(x)=f(x_0)+f'(x_0)\Delta x + f''(x_0)\frac{(\Delta x)^2}{2!}+...+ f^{(k)}(x_0)\frac{(\Delta x)^k}{k!}+...
$$

其中，$$\Delta x = x- x_0 $$ 必须足够小，上式才成立。


假设进行二阶泰勒展开，那么

$$
f(x) \approx f(x_0)+f'(x_0)\Delta x + f''(x_0)\frac{(\Delta x)^2}{2!}
$$


由于函数$$f(x)$$有极值的必要条件是在极值点处一阶导数为0，即梯度向量为0。

那么，两边进行求导

$$
f'(x)\approx f'(x_0)+ f''(x_0)\Delta x 
$$

令上式为0，有如下等式成立

$$
 f'(x_0)+ f''(x_0)(x-x_0) = 0 
$$

$$
x=x_0 - \frac{f'(x_0)}{f''(x_0)}
$$

上式便是牛顿法的迭代公式了。

下面开始Xgboost的推导。

在Xgboost中，损失函数如下:

$$
Obj^{(t)} = \sum_{i=1}^n l(y_i,\hat y_i^{(t)})+\sum_{i=1}^t \Omega(f_i) \\
	  = \sum_{i=1}^n l(y_i, \hat y_i^{t-1} + f_t(x_i) ) + \Omega(f_t) + constant
$$

分别定义一阶导和二阶导

$$
g_i = \partial _{\hat y^{(t-1)}} \quad  l(y_i,\hat y^{(t-1)}) \\
h_i = \partial_{\hat y^{(t-1)}}^2 \quad  l(y_i,\hat y^{(t-1)}) \\
$$

对其进行二阶泰勒展开:

$$
Obj^{(t)} = \sum_{i=1}^n [l(y_i, \hat y_i^{(t-1)}) + g_i f_t(x_i) + 1/2 h_i f_t^2(x_i)  ] +\Omega(f_t)+constant
$$

$$l(y_i, \hat y_i^{(t-1)}) $$也可以归入到常数项中。

其实，如果不考虑正则项，我们现在就可以得到第t棵树的拟合目标:

$$
f_t(x)=-\frac{g}{h}
$$

但是，因为正则项的存在，这时，就不可以简单直接的应用牛顿法中的推导了。

此时，我们引入正则项定义:

$$
\Omega(f_t)=\gamma T + 1/2 \lambda \sum_{j=1}^T w_j^2
$$

T 是叶子节点的个数，$$w_j$$为第j个叶子节点的权重。

我们将损失函数按照树的叶子节点来组织,有

$$
Obj^t = \sum_{i=1}^n [ g_i f_t(x_i) + 1/2 h_i f_t^2(x_i)  ] +\Omega(f_t) \\
       = \sum_{i=1}^n [ g_i w_q(x_i) + 1/2 h_i w_q^2(x_i)  ] + \gamma T + 1/2 \lambda \sum_{j=1}^T w_j^2 \\
       = \sum_{j=1}^T [ (\sum_{i\in I_j }g_i) w_j + 1/2(\sum_{i\in I_j} h_i + \lambda)w_j^2]  + \gamma T
$$

上面的表达形式是不是又和损失函数的二阶泰勒展开又一样了呢

为了表示方便，我们将叶子节点上所有样本的梯度和 和 二阶梯度 和表示如下:

$$
G_j=\sum_{i \in I_j}g_i, H_j=\sum_{i\in I_j}h_i
$$

这个时候，我们对$$w_j$$进行求导，得到如下等式:


$$
w_j^{\star}=-\frac{G_j}{H_j+\lambda}
$$

这个表示式和牛顿法的表达式基本相同，除了分母上多了一个$$\lambda$$,这个是正则项系数


从上面的推导过程可以看出，xgboost的推导过程和牛顿法的推导基本一致，不同点在于，xgboost中引入了正则项，无法直接进行牛顿法推导，需要将损失函数按树的结构进行组织，这样可以把正则项中的内容也引入到表达式中进行统一表示。

有一点需要注意的是，上面的推导有一个前提是树的结构已经确定了。

参考:

1.https://www.zhihu.com/question/63560633

2.https://zhuanlan.zhihu.com/p/38525412

