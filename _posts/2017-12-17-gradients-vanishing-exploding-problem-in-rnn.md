---
  layout: post
  title: RNN中的梯度消失和梯度爆炸
  categories: MachineLearning
  tags:
---


本文是cs231n中关于梯度消失和爆炸现象的一篇笔记。

RNN 比较难训练，原因就是容易出现梯度消失和梯度爆炸问题。
本文从梯度的计算中分析梯度消失和梯度爆炸出现的原因。


先定义下RNN的结构:

$$
\begin{equation}
	h_t = \sigma (W^{(hh)}h_{t-1} + W^{(hx)}x_{[t]})
	\label{eqn:h_t}
\end{equation}
$$

$$
\begin{equation}
	\hat{y}_t = softmax(W^{(S)}h_t)
	\label{eqn:y}
\end{equation}
$$

$$x_1, ..., x_{t-1}, x_t, x_{t+1}, ... x_{T}$$: 词向量 \\
$$W^{hx} \in \mathbb{R}^{D_h \times d}$$: 输入层和隐含层的权重矩阵 \\
$$W^{hh} \in \mathbb{R}^{D_h \times D_h}$$: 隐含层之间的权重矩阵 \\
$$h_{t-1}  \in \mathbb{R}^{D_h}$$:  $$t-1$$时刻的隐层输出. $$h_0 \in \mathbb{R}^{D_h}$$ 是 $$t = 0$$ 时刻的隐层的初始化向量. \\
$$\sigma ()$$: 隐含层的激活函数（这里是sigmoid) \\
$$\hat{y}_t = softmax (W^{(S)}h_t)$$: $$t$$时刻输出层的概率分布. $$\hat{y}_t$$ 是给定文档的上下文向量$$h_{t-1}$$ 和当前时刻的单词 $$x^{(t)}$$情况下的下一个单词. 这里,$$W^{(S)} \in \mathbb{R}^{|V| \times D_h}$$ and $$\hat{y} \in \mathbb{R}^{|V|}$$ $$|V|$$ 是词汇表大小

损失函数

$$
\begin{equation}
	J^{(t)}(\theta) = - \sum_{j=1}^{|V|} y_{t,j} \times log (\hat{y}_{t,j})
	\label {eqn:rnn_loss}
\end{equation}


$$

$$
\begin{equation}
J = \dfrac{1}{T} \sum_{t=1}^{T} J^{(t)}(\theta) = - \dfrac{1}{T} \sum_{t=1}^{T} \sum_{j=1}^{|V|} y_{t,j} \times log (\hat{y}_{t,j})
	\label {eqn:rnn_loss_T}
\end{equation}
$$

为了计算损失函数关于参数矩阵的梯度$$dE/dW$$，我们需要将每个时间步骤的梯度求和


$$
\begin{equation}
	\dfrac{\partial E}{\partial W} = \sum_{t=1}^{T}\dfrac{\partial E_t}{\partial W}
	\label{eqn:bp_rnn_error}
\end{equation}
$$

而每个时刻的梯度可以通过链式法则进行求解:

$$
\begin{equation}
	\dfrac{\partial E_t}{\partial W} = \sum_{k=1}^{t} \dfrac{\partial E_t}{\partial y_t} \dfrac{\partial y_t}{\partial h_t} \dfrac{\partial h_t}{\partial h_k} \dfrac{\partial h_k}{\partial W}
	\label{eqn:bp_rnn_chain}
\end{equation}
$$

在上面的链式法则中，我们着重关注$$\dfrac{\partial h_t}{\partial h_k}$$,因为这里才是与之前的时刻发生联系的关键.

$$
\begin{equation}
	\dfrac{\partial h_t}{\partial h_k} = \prod_{j=k+1}^{t}\dfrac{\partial h_j}{\partial h_{j-1}} = \prod_{j=k+1}^{t}W^T \times diag [f'(j_{j-1})]
	\label{eqn:bp_rnn_k}
\end{equation}
$$

这里应用了等式4和等式1,但是为啥反过来了

因为$$h \in \mathbb{R}^{D_n}$$,$$$\partial h_j/\partial h_{j-1}$$是如下的雅克比矩阵:

$$
\begin{equation}
	\dfrac{\partial h_j}{\partial h_{j-1}} = {[\dfrac{\partial h_{j}}{\partial h_{j-1,1}} ...  \dfrac{\partial h_{j}}{\partial h_{j-1,D_n}}]} =
	\begin{bmatrix}
	\dfrac{\partial h_{j,1}}{\partial h_{j-1,1}} & . & . & . & \dfrac{\partial h_{j,1}}{\partial h_{j-1,D_n}} \\
	. & . & & & . \\
	. & & . & & . \\
	. & & & . & . \\
	\dfrac{\partial h_{j,D_n}}{\partial h_{j-1,1}} & . & . & . & \dfrac{\partial h_{j,D_n}}{\partial h_{j-1,D_n}} \\
	\end{bmatrix}
	\label{eqn:bp_rnn_jaocb}
\end{equation}
$$

综合以上几个公式，我们可以得到:

$$
\begin{equation}
	\dfrac{\partial E}{\partial W} = \sum_{t=1}^{T}\sum_{k=1}^{t} \dfrac{\partial E_t}{\partial y_t} \dfrac{\partial y_t}{\partial h_t} (\prod_{j=k+1}^{t}\dfrac{\partial h_j}{\partial h_{j-1}}) \dfrac{\partial h_k}{\partial W}
\end{equation}
$$

而每个时刻的雅克比矩阵的模又有如下不等式成立:

$$
\begin {equation}
	\parallel \dfrac{\partial h_j}{\partial h_{j-1}} \parallel \leq \parallel W^T\parallel  \parallel diag [f'(h_{j-1})]\parallel \leq \beta_W \beta_h
	\label{eqn:bp_rnn_k_norm}
\end {equation}
$$

其中, $$\beta_W$$ and $$\beta_h$$分别是对应矩阵的模的上界.
而当激活函数使用sigmoid函数时$$f'(h_{j-1})$$的上界为1,所以有如下等式成立:

$$
\begin {equation}
	\parallel \dfrac{\partial h_t}{\partial h_k} \parallel = \parallel \prod_{j=k+1}^{t} \dfrac{\partial h_j}{\partial h_{j-1}}\parallel \leq (\beta_W \beta_h)^{t-k}
	\label{eqn:bp_rnn_k_norm_total}
\end {equation}
$$

可以看出,当$$\beta_W \beta_h$$ 远大于或远小于1且$$t-k$$足够大时$$(\beta_W \beta_h)^{t-k}$$会变成一个非常大或者非常小的数，而一个非常大的$$t-k$$表示距离比较远的单词对损失的贡献。

**当梯度消失时，远距离的单词对预测当前单词的贡献就基本上没有了**
