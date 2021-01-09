---
  layout: post
  title: GBDT 为什么使用负梯度作为拟合目标
  categories: MachineLearning
  tags:
--- 

1.首先，从泰勒展开的角度说明一下为啥梯度是函数增长速度最快的方法。

2.然后，介绍下GBDT为什么使用负梯度作为拟合目标.


以函数$$f:R^d->R$$为例。假设该函数是连续可导的，那么在$$x=x_0$$经过泰勒展开后,有如下等式:

$$
f(x)=f(x_0)+f'(x_0)\Delta x + f''(x_0)\frac{(\Delta x)^2}{2!}+...+ f^{(k)}(x_0)\frac{(\Delta x)^k}{k!}+...
$$

其中，$$\Delta x = x- x_0 $$ 必须足够小，上式才成立。


假设进行一阶泰勒展开，那么

$$
f(x) \approx f(x_0)+f'(x_0)\Delta x
$$

假设 $$\Delta x = \lambda d$$ , $$d$$是单位向量，那么

$$
f(x) \approx f(x_0)+f'(x_0) \lambda d 
$$

其中

$$
f'(x_0) \lambda d  = \lambda |d| * |f'(x_0)| *cos(\theta)
$$

$$\theta$$是两个向量的夹角。

由于我们是最小化$$f(x)$$,所以就是最小化$$f'(x_0) \lambda d$$,而当两个向量方向相反时，$$f'(x_0) \lambda d $$ 的值最小。

此时，

$$d= -\frac{f'(x_0)}{||f'(x_0)||}$$

$$x = x_0 - \lambda \frac{f'(x_0)}{||f'(x_0)||} $$

$$\lambda \frac{1}{||f'(x_0)||}$$ 可以认为是学习率

这就是我们梯度下降的更新公式了。

从上面可以看出，根据一阶泰勒展开，函数沿着负梯度方向更新时 函数下降最快。


在GBDT 中，对损失函数进行泰勒展开

$$
L(y,F_m(x)) = L(y,F_{m-1}(x)) + \frac{\partial L(y,F_{m-1}(x))} {\partial F_{m-1}(x)} (F_m(x)-F_{m-1}(x))
$$

和梯度下降的推导过程一样，

$$
F_m(x) = F_{m-1}(x) - \lambda \frac{\partial L(y,F_{m-1}(x))} {\partial F_{m-1}(x)}
$$

而在GBDT中,第m棵树就是

$$
F_m(x)-F_{m-1}(x)
$$

所以，GBDT 为了最小化损失函数，第m棵树拟合的就是损失函数的负梯度。



第1个问题是在参数空间下优化函数，第2个问题是在函数空间下优化损失函数。

参考:

1.https://www.zhihu.com/question/63560633

2.https://zhuanlan.zhihu.com/p/38525412

