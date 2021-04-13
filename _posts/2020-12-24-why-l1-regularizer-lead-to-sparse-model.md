---
  layout: post
  title: L1 正则为什么会得到稀疏模型
  categories: MachineLearning
  tags:
--- 

在解决各种机器学习问题中，我们已经会遇到过拟合现象。此时，我们一般会使用正则化来缓解这一现象。

过拟合的现象一般是模型权重的绝对值很大。为了抑制权重的绝对值过大，我们可以在损失函数上加上一个限制

$$
𝑊 = 𝑎𝑟𝑔𝑚𝑖𝑛_W L(W,Z)
s.t. \Psi(W)<\delta
$$

根据KKT条件，这种有约束的最优化问题 在合适的参数$$\lambda$$下可以转换为无约束最优化问题

$$
W=𝑎𝑟𝑔𝑚𝑖𝑛_W[ L(W,Z) +\lambda \Psi(W) ]
$$

这就是我们常见的加了正则化的损失函数了。$$\Psi(W)$$ 称为正则化因子，一般是关于W的模的函数。（在GBDT这种非数值优化的模型中就不太一样了)。

一般常见的有两种:

1.L1 正则

$$
\Psi(W) = || W||_1 = \sum_{i=1}^N |w_i|
$$

2.L2 正则

$$
\Psi(W) = || W||_2^2 = \sum_{i=1}^N w_i^2=W^TW
$$


两个正则化方法的差异有很多种解释，比如

1.L1正则可以使得模型更稀疏，模型对内存的占用和计算复杂度都会降低，同时也有特征选择的作用


2.从计算角度看，L2正则可导，可以直接应用梯度下降等方法。L1正则不可导，需要用到次梯度等方法。

关于第1点，L1为什么可以得到更稀疏的模型，我看到有三种解释，分别介绍如下：

第一种就是下面这张比较经典的图了

![L1vsL2](https://user-images.githubusercontent.com/1762074/103151025-26c48c80-47b5-11eb-9228-7a7239b917b5.png)


这是模型只有两个参数的情况。左侧的方形是L1正则的图像，表示W只能取方形中的值。绿色的圆圈表示模型损失的等高线，等高线和方形首次相交的地方就是最优解了。从图形中可以看到，交点容易在坐标轴的角上，此时，另外一个特征维度的值就是0。而右侧的L2 正则的交点就没那么容易在坐标轴上了，所以就不容易出现为0的权重值了。


剩下两种解释是基于解析的解释

先介绍第二种。

下面是L1、L2正则的导数


$$
\begin{equation}
\frac{\partial ||w||_1}{\partial w(k)}=
\begin{cases}
\delta(w(k)) & \text{if(w(k) != 0)}\\
undefined & \text{if(w(k)) =0}
\end{cases}
\end{equation}
$$


$$
\frac{\partial ||w||_2}{\partial w(k)}=2w(k)
$$

$$\delta(x)$$ 是关于x的指示函数，x大于0时为1，小于 0 时为-1。所以当我们要最小化
$$||w||_1$$时，w往0靠近的速度是恒定的，都是1，直到为0为止。而L2的导数在离0越近的地方越小，所以向0靠近的速度是逐渐变慢的。从这个角度上来说，L1更容易得到稀疏的模型。

第三种解释也是基于解析的解释

在L1正则下，我们的目标函数是

$$
Obj = L(W,Z) +\lambda || W||_1 
$$

假设对于某个$$i \in \{1, 2, \ldots, n\}$$来说，$$w_i=0$$,那么，在下一次迭代中，$$w_i$$的值为

$$
\omega_i \gets 0 - \eta\frac{\partial \text{Obj}}{\partial \omega_i}
$$


所以，正则项上增加值为

$$
\Delta \Psi(W) ={\lambda}  \eta\frac{\partial \text{Obj}}{\partial \omega_i}
$$

损失函数降低的值为

$$
\Delta L = \eta\Bigl\lvert \frac{\partial \text{Obj}}{\partial \omega_i}\Bigr\rvert\Bigl\lvert \frac{\partial L}{\partial \omega_i}\Bigr\rvert
$$

上市就是梯度*w变化量

而如果$$\Delta L < \Delta \Psi(W) $$,即
$$\Bigl\lvert \frac{\partial \text{Obj}}{\partial \omega_i}\Bigr\rvert < \lambda_1$$,
那么目标函数总体上就变大了，这个时候就不会更新$$\omega w_i $$了，也就是$$\omega w_i $$还是为0。从这个角度来说，L1正则会得到更稀疏的模型。








参考:

1.https://liam.page/2019/08/31/a-not-so-simple-introduction-to-FTRL/
2.https://www.tianyancha.com/research/LR_intro.pdf