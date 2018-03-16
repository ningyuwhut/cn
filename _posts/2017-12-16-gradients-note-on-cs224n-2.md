---
  layout: post
  title: 神经网络中的常用梯度推导(2)
  categories: MachineLearning,tensorflow
  tags:
---


本文以一个简单的神经网络为例，应用上次的公式求解该神经网络中的各个参数的梯度

神经网络的前向传播过程如下:

$$
\begin{align}
x &= [L_{w_0}, L_{w_1}, ..., L_{w_{m - 1} }]  \\
	z &= x W + b_1 \\
	h &= relu(z) \\
	\theta &= h U + b_2 \\
	\hat y &= softmax(\theta) \\
	J &= CE(y, \hat y)
\end{align}
$$

各个参数的维度如下:

$$
L \in \mathbb{R}^{|V| \times d} \quad\quad
b_1 \in \mathbb{R}^{1 \times D_h} \quad\quad
W \in \mathbb{R}^{md \times D_h} \quad\quad
b_2 \in \mathbb{R}^{1 \times N_c} \quad\quad
U \in \mathbb{R}^{D_h \times N_c} \quad\quad
$$


其中,|V| 是词汇表的大小，d是embedding词向量的长度，m是特征数目，$$D_h$$是隐含层的神经元数目，$$N_c$$是输出类的个数


需要计算的参数梯度如下:

$$
\frac{\partial J}{\partial U} \quad\quad
\frac{\partial J}{\partial b_2} \quad\quad
\frac{\partial J}{\partial W} \quad\quad
\frac{\partial J}{\partial b_1} \quad\quad
\frac{\partial J}{\partial L_{w_i}} \quad\quad
$$

推导前，我们先定义下ReLu函数的梯度:

$$
\begin{align}
ReLU(x)&=max(x,0) \\
ReLU'(x) &= \begin{cases}
	1 \phantom{abc} \text{if $x > 0$} \\
	0 \phantom{abc} \text{if otherwise}
	\end{cases}
	= sgn(ReLU(x))
\end{align}
$$

$$sgn$$表示函数的符号


在推导前，我们再定义下两个比较重要的中间变量$$\frac{\partial J}{\partial \theta}$$和 $$\frac{\partial J}{\partial z}$$,这两个可以看做输出层和中间隐层的误差信号:

$$
\begin{align}
\delta_1 &= \frac{\partial J}{\partial \theta} = \hat y - y & \text{this is just identity (7)}\\
\delta_2 &= \frac{\partial J}{\partial z} = \frac{\partial J}{\partial \theta}\frac{\partial \theta}{\partial h}\frac{\partial h}{\partial z} & \text{using the chain rule}\\
&= \delta_1\frac{\partial \theta}{\partial h}\frac{\partial h}{\partial z} & \text{substituting in $\delta_1$}\\
&= \delta_1\ U^T \frac{\partial h}{\partial z} & \text{using identity (2)} \\
&= \delta_1\ U^T \circ relu'(z) & \text{using identity (4)} \\
&= \delta_1\ U^T \circ sgn(h) & \text{we computed this earlier} \\
\end{align}
$$

检查我们的推导是否正确的一个常用的方法就是看梯度中各个项的维度是否匹配.以上面的为例:

$$
\begin{align}
&\frac{\partial J}{\partial z} \qquad = \qquad \delta_1 \qquad\qquad\qquad U^T \qquad \circ \qquad sgn(h) \\
(1 &\times D_h) \hspace{15mm} (1 \times N_c) \hspace{18mm} (N_c \times D_h) \hspace{18mm} (D_h)
\end{align}
$$


下面依次推导上面的梯度

1.$$\frac{\partial J}{\partial U} $$

$$
\begin{align}
\frac{\partial J}{\partial U} = \frac{\partial J}{\partial \theta}\frac{\partial \theta}{\partial U} = \delta_1\frac{\partial \theta}{\partial U} = h^T \delta_1 & \text{using identity (6)}
\end{align}
$$

2.$$\frac{\partial J}{\partial b_2} $$

$$
\begin{align}
\frac{\partial J}{\partial b_2} = \frac{\partial J}{\partial \theta}\frac{\partial \theta}{\partial b_2} = \delta_1\frac{\partial \theta}{\partial b_2} =\delta_1 & \text{using identity (3)}
\end{align}
$$

3.$$\frac{\partial J}{\partial W} $$

$$
\begin{align}
\frac{\partial J}{\partial W} = \frac{\partial J}{\partial \theta}\frac{\partial z}{\partial W} = \delta_2\frac{\partial z}{\partial W} = x^T \delta_2 & \text{using identity (6)}
\end{align}
$$

4.$$\frac{\partial J}{\partial b_1} $$

$$
\begin{align}
\frac{\partial J}{\partial b_1} = \frac{\partial J}{\partial \theta}\frac{\partial z}{\partial b_1} = \delta_2\frac{\partial z}{\partial b_1} = \delta_2 & \text{using identity (3)} 
\end{align}
$$

5.$$\frac{\partial J}{\partial L_{w_i}} $$


$$
\begin{align}
\frac{\partial J}{\partial L_{w_i}} &= \frac{\partial J}{\partial z}\frac{\partial z}{\partial L_{w_i}} = \delta_2 \frac{\partial z}{\partial L_{w_i}}
\end{align}
$$

下面求解$$\frac{\partial z}{\partial L_{w_i}}$$.

$$
\begin{align}
x W &= [L_{w_0}, L_{w_1}, ..., L_{w_{m - 1} }] W
	= [L_{w_0}, L_{w_1}, ..., L_{w_{m - 1} }]
	\begin{bmatrix} W_{0:d} \\ W_{d:2d} \\ \vdots \\ W_{(m - 1)d:md} \end{bmatrix} \\
	&= L_{w_0} W_{0:d}  + L_{w_1} W_{d:2d} + \dots + L_{w_{m - 1}}W_{(m - 1)d:md}
	=\sum_{j = 0}^{m - 1} L_{w_j} W_{dj:d(j + 1)}
\end{align}
$$

$$L_{w_0}$$是x中第1个单词的embedding向量，长度为d

从上面可以看出，$$\frac{\partial z}{\partial L_{w_i}}$$只有第i项是非零的,
所以

$$
\frac{\partial z}{\partial L_{w_i}} = \frac{\partial }{\partial L_{w_i}} L_{w_i } W_{di:d(i + 1)} =  (W_{di:d(i + 1)})^T
$$
