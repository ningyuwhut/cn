---
  layout: post
  title: batch normalization 反向传播推导
  categories: MachineLearning
  tags:
---

在cs231n中有一个作业需要完成batch normalization的反向传播，没有推导出来，不过找了一篇[不错的文章][1]，简单翻译了主要的过程。

batch normalization 的前向传播过程
====

给定一个输入batch，维度为(N,D),权重矩阵w的维度为(D,H),偏置向量b的维度为(H,1)。

即有N个样本，特征数为D，隐层大小为H

前向过程为

1.线性变换

$$
h=XW+b
$$

h的维度为(N,H)

2.batch normalization 变换

1.对h进行标准化处理，使其均值为0，方差为1

$$ \hat{h}= (h-\mu)(\sigma^2+\epsilon)^{-1/2} $$

$$\mu$$和$$\sigma^2$$分别是batch的均值和方差向量:

$$
\begin{align*}
  \mu &= \frac{1}{N} \sum_i h_i \\
  \sigma^2 &= \frac{1}{N} \sum_i (h_i - \mu)^2
\end{align*}
$$

2.进行平移和缩放

$$
y = \gamma \hat{h}+\beta
$$


3.进行非线性变换，假设激活函数为ReLU

$$
a = ReLu(y)
$$


batch normalization 的后向传播过程
====

后向传播需要计算损失函数关于参数的梯度。在这里我们的参数包括$$\gamma$$,$$\beta$$和$$w$$

为了计算关于$$w$$的梯度，我们还需要计算关于$$h$$的梯度

损失函数关于$$h$$的梯度可以写成:

$$
\begin{equation}
\frac{d\mathcal{L}}{dh} =
\begin{pmatrix}
   \frac{d\mathcal{L}}{dh_{11}} & .. & \frac{d\mathcal{L}}{dh_{1H}} \\
   .. & \frac{d\mathcal{L}}{dh_{kl}} & .. \\
   \frac{d\mathcal{L}}{dh_{N1}} & ... & \frac{d\mathcal{L}}{dh_{NH}}
\end{pmatrix}.
\end{equation}
$$

记住，反向传播的主要思想就是链式法则的运用，而损失函数对h的梯度可以表达为

$$
\begin{eqnarray}
\frac{d\mathcal{L}}{dh_{ij}}       &=&        \sum_{k,l}       \frac{d
\mathcal{L}}{dy_{kl}}\frac{dy_{kl}}{d\hat{h}_{kl}}\frac{d\hat{h}_{kl}}{d h_{ij}}.
\end{eqnarray}
$$

其中，

$$
\frac{dy_{kl}}{d\hat{h}_{kl}}=\gamma_l
$$

难点就在于第三部分了

为了简便起见，我们先只考虑平移部分,即假设

$$
\hat{h_{kl}} = h_{kl}- \mu_l
$$

即没有考虑方差向量

此时有:

$$
\frac{d\hat{h}_{kl}}{d h_{ij}} = \delta_{i,k}\delta_{j,l}-\frac{1}{N}\delta_{j,l}.
$$

其中,只有当$$i=j$$ 时$$\delta_{i,j}=1$$ ,否则全部为0

所以当$$k=i$$且$$j=l$$时第一项为0，仅当$$j=l$$时d第二项为$$1/N$$.

这里不是很理解是怎么推导出来的

现在我们考虑上方差部分:

$$
\begin{eqnarray}
\frac{d\hat{h}_{kl}}{dh_{ij}} = (\delta_{ik}\delta_{jl}-\frac{1}{N}\delta_{jl})(\sigma_l^2+\epsilon)^{-1/2}-\frac{1}{2}(h_{kl}-\mu_l)\frac{d\sigma_l^2}{dh_{ij}}(\sigma_l^2+\epsilon)^{-3/2}
\end{eqnarray}
$$

其中:

$$
\sigma_l^2 = \frac{1}{N}\sum_p \left(h_{pl}-\mu_l\right)^2
$$

所以

$$
\begin{eqnarray}
\frac{d\sigma_l^2}{dh_{ij}} &=& \frac{1}{N}\sum_p2\left(h_{pl}-\mu_l\right)\left(\delta_{ip}\delta_{jl}-\frac{1}{N}\delta_{jl}\right)\\
&=&\frac{2}{N}(h_{il}-\mu_l)\delta_{jl}-\frac{2}{N^2}\sum_p\delta_{jl}\left(h_{pl}-\mu_l\right)\\
&=&\frac{2}{N}(h_{il}-\mu_l)\delta_{jl}-\frac{2}{N}\delta_{jl}\left(\frac{1}{N}\sum_p
h_{pl}-\mu_l\right)\\
&=& \frac{2}{N}(h_{il}-\mu_l)\delta_{jl}
\end{eqnarray}
$$

代入，得

$$
\begin{eqnarray}
\frac{d\hat{h}_{kl}}{dh_{ij}} = (\delta_{ik}\delta_{jl}-\frac{1}{N}\delta_{jl})(\sigma_l^2+\epsilon)^{-1/2}-\frac{1}{N}(h_{kl}-\mu_l)(h_{il}-\mu_l)\delta_{jl}(\sigma_l^2+\epsilon)^{-3/2}.
\end{eqnarray}
$$



$$
\begin{eqnarray}
\frac{d\mathcal{L}}{dh_{ij}} &=& \sum_{kl}\frac{d\mathcal{L}}{dy_{kl}}\frac{dy_{kl}}{d\hat{h}_{kl}}\frac{d\hat{h}_{kl}}{dh_{ij}}\\
&=& \sum_{kl}\frac{d\mathcal{L}}{dy_{kl}}\gamma_l\left((\delta_{ik}\delta_{jl}-\frac{1}{N}\delta_{jl})(\sigma_l^2+\epsilon)^{-1/2}-\frac{1}{N}(h_{kl}-\mu_l)(h_{il}-\mu_l)\delta_{jl}(\sigma_l^2+\epsilon)^{-3/2}\right)\\
&=&\sum_{kl}\frac{d\mathcal{L}}{dy_{kl}}\gamma_l\left((\delta_{ik}\delta_{jl}-\frac{1}{N}\delta_{jl})(\sigma_l^2+\epsilon)^{-1/2}\right)-\sum_{kl}\frac{d\mathcal{L}}{dy_{kl}}\gamma_l\left(\frac{1}{N}(h_{kl}-\mu_l)(h_{il}-\mu_l)\delta_{jl}(\sigma_l^2+\epsilon)^{-3/2}\right)\\
&=&\frac{d\mathcal{L}}{dy_{ij}}\gamma_j(\sigma_j^2+\epsilon)^{-1/2}-\frac{1}{N}\sum_{k}\frac{d\mathcal{L}}{dy_{kj}}\gamma_j(\sigma_j^2+\epsilon)^{-1/2}-\frac{1}{N}\sum_{k}\frac{d\mathcal{L}}{dy_{kj}}\gamma_j\left((h_{kj}-\mu_j)(h_{ij}-\mu_j)(\sigma_j^2+\epsilon)^{-3/2}\right)\\
&=&\frac{1}{N}\gamma_j(\sigma_j^2+\epsilon)^{-1/2}\left(N\frac{d\mathcal{L}}{dy_{ij}}-\sum_k\frac{d\mathcal{L}}{dy_{kj}}-(h_{ij}-\mu_j)(\sigma_j^2+\epsilon)^{-1}\sum_k\frac{d\mathcal{L}}{dy_{kj}}(h_{kj}-\mu_j)\right)
\end{eqnarray}
$$

$$\gamma$$和$$\beta$$和的梯度为

$$
\begin{eqnarray}
\frac{d\mathcal{L}}{d\gamma_j} &=& \sum_{kl}\frac{d\mathcal{L}}{dy_{kl}}\frac{dy_{kl}}{d\gamma_j}\\
&=& \sum_{kl}\frac{d\mathcal{L}}{dy_{kl}}\hat{h}_{kl}\delta_{lj}\\
&=& \sum_{k}\frac{d\mathcal{L}}{dy_{kj}}(h_{kj}- \mu_j)(\sigma^2_j+\epsilon)^{-1/2}\\
\end{eqnarray}
$$


$$
\begin{eqnarray}
\frac{d\mathcal{L}}{d\beta_j} &=& \sum_{kl}\frac{d\mathcal{L}}{dy_{kl}}\frac{dy_{kl}}{d\beta_j}\\
&=& \sum_{kl}\frac{d\mathcal{L}}{dy_{kl}}\delta_{lj}\\
&=& \sum_{k}\frac{d\mathcal{L}}{dy_{kj}}
\end{eqnarray}
$$


1.http://cthorey.github.io/backpropagation/
2.https://chrisyeh96.github.io/2017/08/28/deriving-batchnorm-backprop.html
3.https://kratzert.github.io/2016/02/12/understanding-the-gradient-flow-through-the-batch-normalization-layer.html
4.https://kevinzakka.github.io/2016/09/14/batch_normalization/
