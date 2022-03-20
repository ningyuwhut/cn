---
  layout: post
  title: 神经网络中的常用梯度推导
  categories: MachineLearning tensorflow
  tags:
---

cs224n中有一篇[note][1]讲解了神经网络中梯度推导用到的几个公式,总结得很不错，感觉弄清这几个公式神经网络的推导也就差不多了。

下面将这几个公式简要记录下。

对向量求导梯度的基础是雅克比矩阵（Jacobian Matrix），该矩阵的定义如下:

假设 $$f:R ^n \rightarrow R^m$$ 是一个将长度为n的向量映射到长度为m的向量的函数 $$f = f_1(x_1,...,x_n), f_2(x_1,...,x_n),...,f_m(x_1,...,x_n) $$

该函数的雅克比矩阵形式如下:

$$
	\frac{\partial f}{\partial x} = \begin{bmatrix}
		\frac{\partial f_1}{\partial x_1}  &  \dots   & \frac{\partial f_1}{\partial x_n} \\
		\vdots             & \ddots  & \vdots            \\
		\frac{\partial f_m}{\partial x_1} & \dots    & \frac{\partial f_m}{\partial x_n}            \\
	\end{bmatrix}
$$

亦即：

$$
(\frac{\partial f}{\partial x})_{ij} = \frac{\partial f_i } {\partial x_j}
$$


下面介绍如何计算几种函数的雅克比矩阵，这几个等式在神经网络推导中也经常会用到。

1.矩阵乘以列向量,关于列向量的雅克比矩阵?
===

即已知 $$ z= Wx $$,求解$$\frac{\partial z}{\partial x}$$
===

假设$$W\in R^{n * m } $$。我们可以把z看做一个将m维向量映射为n维向量的函数。所以它的雅克比矩阵维度是$$n*m$$。

因为 $$z_i = \sum_{k=1}^m W_{ik} x_{k} $$,所以 雅克比矩阵的第ij个元素为

$$
\begin{align}
	(\frac{\partial z}{\partial x})_{ij} = \frac{\partial z_i}{\partial x_j} &=  \frac{\partial }{\partial x_j}\sum_{k=1}^m W_{ik} x_{k} = \sum_{k=1}^m W_{ik} \frac{\partial }{\partial x_j}x_{k} = W_{ij}
\end{align}
$$

所以 

$$
\frac{\partial z} {\partial x} = W 
$$

2.行向量乘以矩阵，关于行向量的雅克比矩阵
===

即已知$$\mathbf z =  \mathbf x \mathbf W$$, 求解 $$\frac{\partial z}{\partial x} $$
===

假设$$W\in R^{m * n } $$。我们可以把z看做一个将m维向量映射为n维向量的函数。所以它的雅克比矩阵维度是$$n*m$$。

因为 $$z_i = \sum_{k=1}^m x_{k} W_{ki}  $$,所以 雅克比矩阵的第ij个元素为

$$
\begin{align}
	(\frac{\partial z}{\partial x})_{ij} = \frac{\partial z_i}{\partial x_j} &=  \frac{\partial }{\partial x_j}\sum_{k=1}^m x_{k} W_{ki}  = \sum_{k=1}^m  \frac{\partial }{\partial x_j}x_{k} W_{ki}= W_{ji}
\end{align}
$$

所以 

$$
\frac{\partial z} {\partial x} = W^T
$$


3.向量关于自身的雅克比矩阵
===

即 已知 $$z=x$$,求解 $$\frac{\partial z}{\partial x}$$
===

因为 $$z_i = x_i$$. 所以

$$
(\frac{\partial z}{\partial x})_{ij} = \frac{\partial z_i}{\partial x_j} = \frac{\partial}{\partial x_j}x_i = \begin{cases}
	1 \phantom{abc} \text{if $i = j$} \\
	0 \phantom{abc} \text{if otherwise}
	\end{cases}
$$

所以，$$\frac{\partial z}{\partial x}$$的雅克比矩阵是一个对角阵，对角线上的元素都是1,即单位阵.

$$
\frac{\partial z}{\partial x}=I
$$

因为矩阵乘以单位阵时得到的还是矩阵本身，所以在链式法则中应用该等式时该项可以直接消去

4.应用在一个向量上的element-wise函数的雅克比矩阵
===

即已知 $$ z= f(x) $$,求解 $$\frac{\partial z}{\partial x} $$
===

$$
\begin{align}
	(\frac{\partial z}{\partial x})_{ij} = \frac{\partial z_i}{\partial x_j} &=  \frac{\partial }{\partial x_j} f(x_i)  = \begin{cases}
	f'(x_i) \phantom{abc} \text{if $i = j$} \\
	0 \phantom{abc} \text{if otherwise}
	\end{cases}

\end{align}
$$

与3类似，此时$$\frac{\partial z}{\partial x}$$的雅克比矩阵也是一个对角阵，只不过对角线上的元素不再是1,而是$$f'(x_i)$$.可以写成如下形式:

$$
\frac{\partial z}{\partial x}=diag(f'(x))
$$

由于乘以一个对角阵相当于对角阵中的每个元素和矩阵的对应列的所有元素做element-wise乘法，所以在链式法则中应用该等式时可以写成 $$[\circ f'(x_)] $$

5.矩阵乘以列向量，关于矩阵的雅克比矩阵
===

已知$$z = W x, \delta = \frac{\partial J}{\partial z}$$,求解 $$\frac{\partial J}{\partial W} = \frac{\partial J}{\partial z} \frac{\partial z}{\partial W} = \delta \frac{\partial z}{\partial W}$$?
===

这条规则应该是note中最复杂的一条了,因为还要用到损失函数$$J$$。

假设我们计算损失函数J关于矩阵$$W\in R^{n*m}$$的梯度.我们可以把J看做将一个长度为$$n*m$$的向量映射为一个标量的函数，那么$$\frac{\partial J}{\partial W}$$就是一个长度为长度为$$n*m$$的行向量。但是在实际应用中这种排列方式并不是很有用，一个更优雅的方式是排列成$$n*m$$的矩阵:

$$
	\frac{\partial J}{\partial W} = \begin{bmatrix}
		\frac{\partial J}{\partial W_{11}}  &  \dots   & \frac{\partial J}{\partial W_{1m}} \\
		\vdots             & \ddots  & \vdots            \\
		\frac{\partial J}{\partial W_{n1}} & \dots    & \frac{\partial J}{\partial W_{nm}}            \\
	\end{bmatrix}

$$

这个矩阵和$$W$$的维度相同，所以在做梯度下降时我们可以直接从$$W$$中减去这个矩阵(还要乘以学习率)

所以我们转而求解$$\frac{\partial J}{\partial W}$$

按照上面的方式排列梯度使得计算$$\frac{\partial z}{\partial W}$$变得复杂，因为$$z$$是一个向量，这样如果像刚才那样排列的话$$\frac{\partial z}{\partial W}$$就是一个$$n\times m \times n$$的一个张量。不过，我们可以通过计算$$z$$对$$W_{ij}$$的梯度来避免这个问题。

因为$$\frac{\partial z}{\partial W_{ij}}$$是一个向量，处理起来就方便多了.

$$
\begin{align}
z_k &= \sum_{l=1}^m  W_{kl} x_{l} \\
\frac{\partial z_k}{\partial W_{ij}} &=  \sum_{l=1}^m  \frac{\partial }{\partial W_{ij}}W_{kl} x_l 
\end{align}
$$

重点来了。

$$\frac{\partial }{\partial W_{ij}}W_{kl}$$只有在i=k且j=l时才会为1，其他所有情况为0.所以$$k\neq i$$时 $$\frac{\partial }{\partial W_{ij}}W_{kl}$$ 为0，$$\frac{\partial z_k}{\partial W_{ij}}$$也为0.
而当$$k= i$$时 ，$$\frac{\partial z_k}{\partial W_{ij}}$$中只有$$l=j$$时得到一个非零项$$x_j$$,所以

$$
\frac{\partial z_k}{\partial W_{ij}} = 
\begin{cases}
	x_j \phantom{abc} \text{if $k = i$} \\
	0 \phantom{abc} \text{if otherwise}
	\end{cases}
$$

换成向量形式:

$$
\frac{\partial z}{\partial W_{ij}} = \begin{bmatrix} 0 \\ \vdots \\ 0 \\ x_j \\ 0 \\ \vdots \\ 0 \end{bmatrix}
    \gets \text{$i$th element}
$$


$$
\frac{\partial J}{\partial W_{ij}} = \frac{\partial J}{\partial z}\frac{\partial z}{\partial W_{ij}} = \delta \frac{\partial z}{\partial W_{ij}} = \sum_{k=1}^n \delta_k \frac{\partial z_k}{\partial W_{ij}} = \delta_i x_j
$$

所以,$$\frac{\partial J}{\partial W} $$的(i,j)元素值是$$\delta_i x_j $$,这个矩阵可以写成如下外积形式:

$$
\boxed{\frac{\partial J}{\partial W} = \delta^T x}
$$

**$$\delta$$是一个行向量**


6.行向量乘以矩阵，关于矩阵的雅克比矩阵
===

已知$$z =  xW, \delta = \frac{\partial J}{\partial z}$$,求解 $$\frac{\partial J}{\partial W} = \frac{\partial J}{\partial z} \frac{\partial z}{\partial W} = \delta \frac{\partial z}{\partial W}$$?
===

和上面的思路类似，我们转而求解$$\frac{\partial J}{\partial W}$$

$$W∈R^{m∗n}$$

$$
\begin{align}
z_k &= \sum_{l=1}^m x_{l} W_{lk}  \\
\frac{\partial z_k}{\partial W_{ij}} &=  \sum_{l=1}^m  x_l \frac{\partial }{\partial W_{ij}}W_{lk}
\end{align}
$$

分析上面这个式子，可以看出，只有在j=k且l=i时$$\frac{\partial }{\partial W_{ij}}W_{lk}$$才会为1.所以,$$k\neq j$$时,$$\frac{\partial z_k}{\partial W_{ij}}$$ 为0,当$$k=j$$时,$$\frac{\partial }{\partial W_{ij}}W_{lk}$$只有在$$l=i$$时才有一个非零项$$x_i$$,即

$$
\frac{\partial z}{\partial W_{ij}} = [  0  \dots  0 \dots x_i \dots 0 \dots  0  ],  \text{$j$th element}
$$

$$
\frac{\partial J}{\partial W_{ij}} = \frac{\partial J}{\partial z}\frac{\partial z}{\partial W_{ij}} = \delta \frac{\partial z}{\partial W_{ij}} = \sum_{k=1}^n \delta_k \frac{\partial z_k}{\partial W_{ij}} = \delta_j x_i
$$

$$
\boxed{\frac{\partial J}{\partial W} =  x^T\delta}
$$


[1]:http://web.stanford.edu/class/cs224n/lecture_notes/cs224n-2017-gradient-notes.pdf
[2]:https://zh.wikipedia.org/wiki/外积
