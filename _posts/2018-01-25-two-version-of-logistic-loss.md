---
  layout: post
  title: 逻辑斯蒂损失函数的两种形式
  categories: MachineLearning
  tags:
---

这两天看了下libFM的代码，对于分类问题，libFM使用的损失函数是logloss，跟常见的LR的损失函数的形式略有不同。于是整理了下两种形式的LR的异同。

在二分类问题中，样本的label一般有两种形式： $$y=\pm1$$或者$$y=0，1$$。

如果$$y=0，1$$，那么有：

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

最后一步展开如下：

$$
\begin{equation}
\begin{aligned}
\log\big(\mathbb{P}(y|z)\big) &=
\log(\sigma(z)^y(1-\sigma(z))^{1-y})\\
&= y log(\sigma(z)) + (1-y) log(1-\sigma(z))\\
&= y log(\frac{1}{1+e^{-z}}) + (1-y) log(\frac{1}{1+e^{z}})\\
&= y (log(\frac{1}{1+e^{-z}}) -log(\frac{1}{1+e^{z}})) + log(\frac{1}{1+e^{z}}) \\
&= y (log(\frac{1+e^{z}}{1+e^{-z}})) - log(1+e^{z})\\
&= y (log(\frac{e^{z}(1+e^{z})}{1+e^{z}})) - log(1+e^{z})\\
&= yz - log(1+e^{z})\\
\end{aligned}
\end{equation}
$$


而如果我们选择$$y=\pm1$$，则y的概率为

$$
\begin{equation}
\begin{aligned}
\mathbb{P}(y|z)&=\sigma(yz)
=\frac{1}{1+e^{-yz}}
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

可以看出，跟$$y=0，1$$其实是一样的。


从这里也可以看到，$$\mathbb{P}(y=0|z)$$和$$\mathbb{P}(y=-1|z)$$的表达式是相同的，
也就是有如下等式成立:

$$
\sigma(-z)=1-\sigma(z)
$$


和上面的推导一样:

$$
\begin{equation}
\begin{aligned}
L(z)=-\log\big(\prod_j^m\mathbb{P}(y_j|z_j)\big)=-\sum_j^m\log\big(\mathbb{P}(y_j|z_j)\big)=\sum_j^m\log(1+e^{-y_jz_j})
\end{aligned}
\end{equation}
$$

也就是下面两种表达式是等价的:

$$
\begin{equation}
\begin{aligned}
-y_iz_i+\log(1+e^{z_i})\equiv \log(1+e^{-y_iz_i})
\end{aligned}
\end{equation}
$$

但是两种形式还是有些不同的。

第一种形式其实是从y服从伯努利分布推导出来的。想想伯努利分布的定义:

$$
P(Y = y\ |\ p) = \mathcal L(p; y) = p^y\ (1-p)^{1-y} = \begin{cases}1-p &y=0 \\ p &y=1 \end{cases}
$$

和第一种损失的表示形式是一样的。

这种形式其实也是广义线性模型的一种。

第二种形式的好处是和hinge loss，0-1 loss 比较起来比较方便，因为都是定义在$$y=\pm1$$上的。


下面补充下sigmoid函数的梯度。

$$

\begin{equation}
\begin{aligned}
\sigma'(z) &= -\frac{1}{(1+e^{-z})^2} * e^{-z} * (-1) \\
&= \frac{e^{-z}}{(1+e^{-z})^2} \\
&= \frac{1 + e^{-z} - 1}{(1+e^{-z})^2} \\
&= \frac{1}{1 + e^{-z}} - \frac{1}{(1+e^{-z})^2} \\
&= \frac{1}{1 + e^{-z}} (1 - \frac{1}{1 + e^{-z}}) \\
&= \sigma(z)(1-\sigma(z))
\end{aligned}
\end{equation}
$$

即,sigmoid函数的梯度有如下关系成立:

$$
\sigma'(z) = \sigma(z)(1-\sigma(z))
$$

似然函数关于z的梯度:

第一种形式:
$$
\begin{equation}
\begin{aligned}
\frac{\partial log \mathbb{P}(y|z)}{\partial{z}} &= 
\frac{\partial y log(\sigma(z)) + (1-y) log(1-\sigma(z))}{\partial{z}}  \\
&= y \frac{1}{\sigma(z)}\frac{\partial \sigma(z)}{\partial {z}} +
(1-y)\frac{1}{\sigma(-z)} *(-1) * \frac{\partial \sigma(-z)}{\partial {z}} \\
&= 
y \frac{1}{\sigma(z)} \sigma(z)(1-\sigma(z)) +
(1-y)\frac{1}{\sigma(-z)} *(-1) * \sigma(-z)(1-\sigma(-z))\\
&= y (1-\sigma(z)) + (y-1) *(1-\sigma(-z)) \\
&= y (1-\sigma(z)) + (y-1) *(1-(1 - \sigma(z))) \\
&= y (1-\sigma(z)) + (y-1) *\sigma(z) \\
&= y - \sigma(z) \\
\end{aligned}
\end{equation}
$$

最终梯度形式就是label和预估值之间的残差。
而似然函数和损失函数之间差了一个负号，所以损失函数关于z的梯度形式应该是$$\sigma(z)-y$$。

推导时用到了$$\sigma(-z)=1-\sigma(z)$$ 和 $$\sigma(z)$$的梯度表达式

第二种形式:

$$
\begin{equation}
\begin{aligned}
\frac{\partial log \mathbb{P}(y|z)}{\partial{z}} &= 
\frac{\partial log \sigma(yz)}{\partial z}  \\
&= \frac{1}{\sigma(yz)} \frac{\partial \sigma(yz)}{\partial {z}}\\
&= \frac{1}{\sigma(yz)} \sigma(yz)(1-\sigma(yz)) * y \\
&= y(1-\sigma(yz))
\end{aligned}
\end{equation}
$$

梯度为$$y(1-\sigma(yz))$$。
和第一种形式一样，这里也是使用的似然函数，如果是损失函数则需要再加一个负号。

y = 1时就是$$1-\sigma(z)$$，y = -1 时是$$ \sigma(-z) -1 = 1 - \sigma(z) - 1 = -\sigma(z) $$, 

和第一种形式的梯度是一样的。

参考:

1.https://stats.stackexchange.com/questions/250937/which-loss-function-is-correct-for-logistic-regression

2.https://stats.stackexchange.com/questions/229645/why-there-are-two-different-logistic-loss-formulation-notations?noredirect=1&lq=1
