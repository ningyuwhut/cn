---
  layout: post
  title: 逻辑斯蒂损失函数的两种形式
  categories: MachineLearning
  tags:
---

这两天看了下libFM的代码，对于分类问题，libFM使用的损失函数是logloss，跟常见的LR的损失函数的形式略有不同。于是整理了下两种形式的LR的异同。

在二分类问题中，样本的label一般有两种形式: $$y=\pm1$$或者$$y=0,1$$.

如果$$y=0,1$$,那么有:

$$
\begin{equation}
\begin{aligned}
\mathbb{P}(y=1|z) & =\sigma(z)=\frac{1}{1+e^{-z}}\\
\mathbb{P}(y=0|z) & =1-\sigma(z)=\frac{1}{1+e^{z}}\\
\end{aligned}
\end{equation}
$$

上面的等式可以表示成:

$$
\mathbb{P}(y|z)  =\sigma(z)^y(1-\sigma(z))^{1-y}
$$

对应的对数损失为:

$$
\begin{equation}
\begin{aligned}
l(z)=-\log\big(\prod_i^m\mathbb{P}(y_i|z_i)\big)=-\sum_i^m\log\big(\mathbb{P}(y_i|z_i)\big)=\sum_i^m-y_iz_i+\log(1+e^{z_i})
\end{aligned}
\end{equation}
$$

而如果我们选择$$y=\pm1$$,则y的概率为

$$
\begin{equation}
\begin{aligned}
\mathbb{P}(y|z)&=\sigma(yz)\\
&=\frac{1}{1+e^{-yz}}
\end{aligned}
\end{equation}
$$

将$$y=\pm1$$ 分开表示则有:

$$
\begin{equation}
\begin{aligned}
\mathbb{P}(y=1|z) & =\sigma(z)=\frac{1}{1+e^{-z}}\\
\mathbb{P}(y=-1|z) & =\sigma(-z)=\frac{1}{1+e^{z}}\\
\end{aligned}
\end{equation}
$$

可以看出，跟$$y=0,1$$其实是一样的。

由于logstic 函数有如下性质:

$$
\sigma(-z)=1-\sigma(z)
$$

那么，也就是:

$$
\mathbb{P}(y=0|z)=\mathbb{P}(y=-1|z)=\sigma(-z)
$$


和上面的推导一样:

$$
\begin{equation}
\begin{aligned}
L(z)=-\log\big(\prod_j^m\mathbb{P}(y_j|z_j)\big)=-\sum_j^m\log\big(\mathbb{P}(y_j|z_j)\big)=\sum_j^m\log(1+e^{-y_jz_j})
\end{aligned}
\end{equation}
$$

从上面的推导可以看出，两种损失函数其实是等价的。但是两种形式还是有些不同的。

第一种形式其实是从y服从伯努利分布推导出来的。想想伯努利分布的定义:

$$
P(Y = y\ |\ p) = \mathcal L(p; y) = p^y\ (1-p)^{1-y} = \begin{cases}1-p &y=0 \\ p &y=1 \end{cases}
$$

和第一种损失的表示形式是一样的。

这种形式其实也是广义线性模型的一种。

第二种形式的好处是和hinge loss，0-1 loss 比较起来比较方便，因为都是定义在$$y=\pm1$$上的。

参考:

1.https://stats.stackexchange.com/questions/250937/which-loss-function-is-correct-for-logistic-regression

2.https://stats.stackexchange.com/questions/229645/why-there-are-two-different-logistic-loss-formulation-notations?noredirect=1&lq=1
