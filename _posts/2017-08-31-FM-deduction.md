---
  layout: post
  title: FM推导
  categories: MachineLearning
  tags:
---

二阶FM的表达式如下:

$$
y=w_0+\sum_{i=1}^n w_i x_i + \sum_{i=1}^{n-1} \sum_{j=i+1}^n  <v_i,v_j> x_i x_j
$$

其中,$$w_o$$是偏置项,$$w$$为n维特征向量,$$V$$为N*K 维矩阵，每个$$w_i$$对应一个$$v_i$$向量。在该表达式中,最后一项刻画的是特征之间的相互关系,也可以扩展到多阶。

对于第三项有如下推导

$$
\begin{align*}
&\sum_{i=1}^{n-1}\sum_{j=i+1}^n <v_i,v_j> x_ix_j \\
&= \frac{1}{2}\sum_{i=1}^n\sum_{j=1}^n <v_i,v_j> x_ix_j - \frac{1}{2}\sum_{i=1}^n<v_i,v_i>x_ix_i \\
&=\frac{1}{2}\left( \sum_{i=1}^n\sum_{j=1}^n \sum_{f=1}^k v_{if}v_{jf} x_i x_j - \sum_{i=1}^n \sum_{f=1}^k v_{if}^2 x_i^2 \right) \\
&=\frac{1}{2}\sum_{f=1}^k\left( \left( \sum_{i=1}^n v_{if}x_i \right) \left( \sum_{j=1}^n v_{jf} x_j\right) -\sum_{i=1}^n v_{if}^2 x_i^2 \right) \\
&=\frac{1}{2}\sum_{f=1}^k \left( \left( \sum_{i=1}^n v_{if}x_i\right)^2 -\sum_{i=1}^n v_{if}^2 x_i^2  \right)
\end{align*}
$$

这样，表达式变成:

$$
\begin{align*}
y&=w_0+\sum_{i=1}^n w_i x_i + \sum_{i=1}^{n-1} \sum_{j=i+1}^n  <v_i,v_j> x_i x_j \\
&=w_0+\sum_{i=1}^n w_i x_i + \frac{1}{2}\sum_{f=1}^k \left( \left( \sum_{i=1}^n v_{if}x_i\right)^2 -\sum_{i=1}^n v_{if}^2 x_i^2  \right)
\end{align*}
$$


参数集合为$$\Theta=(w_0, w_1, w_2,...,w_n, v_{11},v_{12},...v_{nn})$$
$$y$$关于各个参数的偏导如下:

$$
\frac{\partial y}{\partial \theta} = 
\begin{cases}
1, &\theta=w_0\\
x_i, &\theta=w_i, i=1,2,...,n \\
x_i\sum_{s=1, s \neq i} v_{sf}x_s, &\theta=v_{if} ,i=1,2,...,n; f=1,2,...,k
\end{cases}
$$

$$y$$关于$$v_{if}$$的推导如下:

$$
\begin{align*}
\frac{\partial y}{\partial \theta} &= 
\frac {1}{2} \left( 2( \sum_{s=1}^n v_{sf} x_s)  x_i - 2 v_{if}x_i^2 \right) \\
&=\left( \sum_{s=1, s\neq i}^n v_{sf} x_s + v_{if} x_i \right) x_i -v_{if} x_i^2 \\
&=\left( \sum_{s=1, s\neq i}^n v_{sf} x_s \right)x_i + v_{if} x_i^2 - v_{if} x_i^2  \\
&=\left( \sum_{s=1, s\neq i}^n v_{sf} x_s \right)x_i
\end{align*}
$$

剩下的就是结合具体的损失函数和优化方法进行迭代了
