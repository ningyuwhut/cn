---
  layout: post
  title: softmax函数梯度 
  categories: MachineLearning
  tags:
---


softmax 函数形式:

$$
\hat y_j = softmax(\theta_j) = \frac{exp ( \theta_j)} {\sum_{i=1}^V exp(\theta_i)}
$$

交叉熵损失（cross-entropy loss）的定义如下

$$
J=-\sum_{i=1}^V y_i log (\hat y_i)
$$

首先，求解softmax函数的梯度$$\frac{ \partial \hat y_j}{\partial \theta_k}$$。

当$$k=j$$时:

$$
\begin{align*}
\frac{ \partial \hat y_j}{\partial \theta_k} 
&=\frac{exp(\theta_j)\sum_{i=1}^V exp(\theta_i)-exp(\theta_j) exp(\theta_j)}{(\sum_{i=1}^V exp(\theta_i))^2}\\
&=\frac{exp(\theta_j)}{\sum_{i=1}^V exp(\theta_i)}-(\frac{exp(\theta_j)}{\sum_{i=1}^V exp(\theta_i)})^2\\
&=\hat y_j - (\hat y_j)^2 \\
&=\hat y_j(1-\hat y_j)
\end{align*}
$$

当$$k\ne j$$时

$$
\begin{align*}
\frac{ \partial \hat y_j}{\partial \theta_k}
&=-\frac{exp(\theta_j) exp(\theta_k) }{(\sum_{i=1}^V exp(\theta_i))^2}\\
&=-\hat y_j * \hat y_k
\end{align*}
$$

所以:

$$
\frac{\partial \hat y_j}{\partial \theta_k} = \begin{cases} \hat y_j(1-\hat y_j),&\quad k = j \\ -\hat y_j * \hat y_k,&\quad k \neq j\end{cases}
$$

交叉熵损失的梯度如下:

$$
\begin{split} \frac{\partial J}{\partial \theta_k}
&=-\sum_{i=1}^V y_i\frac{\partial \log \hat y_i}{\partial \theta_k} \\ 
&=-\sum_{i=1}^V y_i \frac{1}{\hat y_i}\frac{\partial \hat y_i}{\partial \theta_k} \\ 
&=-y_k \frac{1}{\hat y_k}  \hat y_k (1-\hat y_k)-\sum_{i\neq k}y_i\frac{1}{\hat y_i}({-\hat y_i\hat y_k}) \\ 
&=-y_k(1-\hat y_k)+\sum_{i\neq k}y_i ({\hat y_k}) \\ 
&=-y_k+{y_k \hat y_k+\sum_{i\neq k}y_i({\hat y_k})} \\ 
&={\hat y_k\left(\sum_i y_i\right)}-y_k \\ 
&=\hat y_k-y_k\end{split}
$$

所以:

$$
\frac{\partial J}{\partial \theta_k} = \hat y_k-y_k
$$


