---
  layout: post
  title: CNN反向推导
  categories: MachineLearning
  tags:
---


## 前向传播
先简单介绍一下CNN的前向传播。

主要是两种情况，一是从输入层或者pooling层到卷积层的前向传播，另外一种是从卷积层到pooling层的前向传播。

### 1.卷积层


假设当前层是卷积层，那么令$$X_j^l$$表示第l层的第j个feature map。feature map是前一层的输入与卷积核（kernel或filter）卷积后的结果。
$$M_j^l$$表示与$$X_j^l$$相关联的l-1层的feature map的集合。那么有如下:

$$
    X_j^l=f(\sum_{i\in M_j^l} K_{i,j}^l * X_i^{l-1} + b_j^l )
$$

其中，$$*$$为卷积运算,$$b_j^l$$为bias,是一个标量。$$K_j^l$$为卷积核矩阵,大小为$$d*d$$ 。假设卷积结果为F，那么卷积的计算公式如下:

$$
    F_{u,v}=\sum_h \sum_j K_{d-h,d-j} \cdot X_{u+h, v+j}
$$

需要强调一点，在卷积运算中，不是简单地把输入矩阵和卷积核进行点乘，而是把卷积核K旋转180度之后再算点乘。卷积核k旋转180度相当于把矩阵元素先左右互换，再上下互换。
这样，元素$$K_{h,j}$$旋转180度之后就变成了$$K_{d-h-1, d-j-1}$$。$$X_{u+h,v+j}$$表示输入feature map中与卷积核进行卷积的对应元素的值。

### 2. pooling层

令$$X_j^l$$表示第l层Pooling层的第j个特征图,那么有:

$$
X_j^l = f(\beta_j^l \cdot down(X_j^{l-1}) + b_j^l)
$$

$$\beta_j^l$$和$$b_j^l$$分别为采样尺度和偏置，均为实数。


## 反向传播

令$$\delta_j^l$$表示第l层第j个feature map$$X_j^l$$对应的误差矩阵,那么有:

$$
\delta_j^l=\frac{\partial E}{\partial net_j^l} \\
(\delta_j^l)_{u,v}=\frac{\partial E}{( \partial (net_j^l)_{u,v} )} \\
$$

$$
net_j^l = \left\{ 
\begin{aligned} 
& \sum_{i\in M_j^l}K^l_{i,j}*X_i^{l-1} + b_j^l （卷积层） \\
& \beta_j^l \cdot down(X_j^{l-1}) + b_j^l   （池化层）\\
\end{aligned}
\right.
$$

其中，$$net_j^l$$表示神经网络层的输入
## 卷积层

卷积层的参数包括卷积核和偏置。

令$$\delta_j^l$$表示卷积层第j个feature map的误差矩阵

$$
\begin{aligned}
    \delta_j^l & =\frac{\partial E}{\partial net_j^l} \\
	       & = \frac{\partial E}{\partial net_j^{l+1}} \frac{\partial net_j^{l+1} }{ X_j^l } \frac { \partial X_j^l }{ \partial net_j^l} \\
	       & = \beta_j^{l+1} (up(\delta_j^{l+1}) \cdot f'(net_j^l))
\end{aligned}
$$



有了误差项之后，我们开始计算卷积核和偏置的导数。
因为卷积层的每个输出feature map都对应一个偏置$$b_j^l$$，且该偏置为标量，那么该输出特征图中的每个误差项都对$$b_j^l$$有所贡献,即

$$
b_j^l = \sum_u \sum_v \frac{\partial E }{ \partial (net_j^l)_{u,v} } \frac { \partial (net_j^l)_{u,v} } {\partial b_j^l } = \sum_u \sum_v (\delta_j^l)_{u,v}
$$

$$
\begin{aligned}
\frac{\partial E}{\partial ( (K_{i,j}^l )_{r,s} } & = \sum_u \sum_v  \frac { \partial E  } {\partial (net_j^l)_{u,v} } \frac {\partial (net_j^l)_{u,v} } { \partial  (K_{i,j}^l)_{r,s} } \\
						  & = \sum_u \sum_v (\delta_j^l)_{u,v} \frac{ (X_i^{l-1} * K_{i,j}^l + b_j^l)_{u,v} } { (K_{i,j}^l)_{r,s} }
\end{aligned}
$$

其中,

$$
(X_i^{l-1} * K_{i,j}^l)_{u,v} = \sum_r \sum_s (K_{i,j}^l)_{r,s}  * (X_i^{l-1})_{u+r,v+s}
$$

所以，

$$
\begin{aligned}
\frac{\partial E}{\partial ( (K_{i,j}^l )_{r,s} } & = \sum_u \sum_v (\delta_j^l)_{u,v} \frac{ (X_i^{l-1} * K_{i,j}^l + b_j^l)_{u,v} } { (K_{i,j}^l)_{r,s} } \\
						  & = \sum_u \sum_v (\delta_j^l)_{u,v} (X_i^{l-1})_{u+r,v+s}
\end{aligned}
$$

举个例子.

$$
\begin{matrix}
X=\begin{bmatrix}
x_{11} &  x_{12} &  x_{13}\\
x_{21} &  x_{22} &  x_{23} \\
x_{31} &  x_{32} &  x_{33} \\
\end{bmatrix} &  K = \begin{bmatrix}
k_{11} &  k_{12} \\
k_{21} & k_{22}
\end{bmatrix} & P = \begin{bmatrix}
a_{11} &  a_{12} \\
a_{21} & a_{22}
\end{bmatrix}  \\ 
\end{matrix}
$$

$$
\begin{align}
a_{11}  = x_{11}k_{22}+x_{12}k_{21} +x_{21}  k_{12}+x_{22}k_{11}  \\
a_{12}  = x_{12}k_{22}+x_{13}k_{21} +x_{22}  k_{12}+x_{23}k_{11}  \\
a_{21}  = x_{21}k_{22}+x_{22}k_{21} +x_{31}  k_{12}+x_{32}k_{11}  \\
a_{22}  = x_{22}k_{22}+x_{23}k_{21} +x_{32}  k_{12}+x_{33}k_{11}
\end{align}
$$

即

$$
P = X * K 
$$

所以

$$
\begin{aligned}
\frac{\partial E}{\partial ( (K_{i,j}^l )_{r,s} }  &= \sum_u \sum_v  \frac { \partial E  } {\partial (net_j^l)_{u,v} } \frac {(net_j^l)_{u,v} } { (K_{i,j}^l)_{r,s} } \\ 
						   &= \sum_u \sum_v \frac {\partial E} { \partial P_{u,v} } \frac{ \partial P_{u,v} }{ \partial (K_{i,j}^l)_{r,s} }
\end{aligned}
$$

所以

$$
\begin{align}
\frac{\partial E}{\partial k_{11}}  &=\frac{\partial E}{\partial a_{11}} x_{22} + \frac{\partial E}{\partial a_{12}} x_{23} + \frac{\partial E}{\partial a_{21}} x_{32} + \frac{\partial E}{\partial a_{22}} x_{33}  \\
\frac{\partial E}{\partial k_{12}}  &=\frac{\partial E}{\partial a_{11}} x_{21} + \frac{\partial E}{\partial a_{12}} x_{22} + \frac{\partial E}{\partial a_{21}} x_{31} + \frac{\partial E}{\partial a_{22}} x_{32}  \\
\frac{\partial E}{\partial k_{21}}  &=\frac{\partial E}{\partial a_{11}} x_{12} + \frac{\partial E}{\partial a_{12}} x_{13} + \frac{\partial E}{\partial a_{21}} x_{22} + \frac{\partial E}{\partial a_{22}} x_{23}  \\
\frac{\partial E}{\partial k_{22}}  &=\frac{\partial E}{\partial a_{11}} x_{11} + \frac{\partial E}{\partial a_{12}} x_{12} + \frac{\partial E}{\partial a_{21}} x_{21} + \frac{\partial E}{\partial a_{22}} x_{22}  \\
\end{align}
$$


$$
\begin{matrix}
TMP = \begin{bmatrix}
\frac{\partial E}{\partial k_{22}} & \frac{\partial E}{\partial k_{21}}\\
\frac{\partial E}{\partial k_{12}} & \frac{\partial E}{\partial k_{11}}\\
\end{bmatrix}  = \begin{bmatrix}
x_{11} &  x_{12} &  x_{13}\\
x_{21} &  x_{22} &  x_{23} \\
x_{31} &  x_{32} &  x_{33} \\
\end{bmatrix} * 
\begin{bmatrix}
\frac{\partial E}{\partial a_{22}} & \frac{\partial E}{\partial a_{21}}\\
\frac{\partial E}{\partial a_{12}} & \frac{\partial E}{\partial a_{11}}\\
\end{bmatrix} 
\end{matrix}
$$

### 池化层

池化层的参数包括采样尺度和偏置，令$$\delta_i^l$$表示pooling层的误差矩阵，有

$$
\begin{aligned}
    \delta_i^l & = \frac {\partial E} {\partial net_i^l } \\
	       & = \frac {\partial E } {x_i^l} \frac { \partial x_i^l } { \partial net_i^l }  \\
	       & = \frac {\partial x_i^l } {\partial net_i^l } \sum_{j}\frac { \partial E } { \partial net_j^{l+1} } \frac { \partial net_j^{l+1} } { x_i^l } \\
	       & = \frac { \partial x_i^l } { \partial net_i^l } \sum_j \delta_j^{l+1} \frac { \partial (K^{l+1}_{i,j}*X_i^l + b_j^{l+1}) } { x_i^l } \\ 
	       & = f'(net_i^l) * \sum_j conv2( \delta_j^{l+1},K_{i,j}^{l+1},full)
\end{aligned}
$$

最后一步我也不太明白，不过可以通过一个例子来帮助理解。

$$
\begin{matrix}
x_i^l=\begin{bmatrix}
x_{11} &  x_{12} & x_{13} \\
x_{21} &  x_{22} &  x_{23}\\
x_{31} &  x_{32} &  x_{33}
\end{bmatrix} &  K_{ij} = \begin{bmatrix}
k_{11} & k_{12} \\
k_{21} & k_{22}
\end{bmatrix}
\end{matrix}
$$

令$$A_i^{l+1}=x_i^l * K_{ij} $$,那么 $$net_j^{l+1} =\sum_i A_i^{l+1} + b_j^{l+1} $$

$$
\begin{matrix}
A_i^{l+1}=\begin{bmatrix}
a_{11} &  a_{12}  \\
a_{21} &  a_{22} 
\end{bmatrix}  &
\begin{align}
a_{11}  = x_{11}k_{22}+x_{12}k_{21} +x_{21}  k_{12}+x_{22}k_{11}  \\
a_{12}  = x_{12}k_{22}+x_{13}k_{21} +x_{22}  k_{12}+x_{23}k_{11}  \\
a_{21}  = x_{21}k_{22}+x_{22}k_{21} +x_{31}  k_{12}+x_{32}k_{11}  \\
a_{22}  = x_{22}k_{22}+x_{23}k_{21} +x_{32}  k_{12}+x_{33}k_{11}
\end{align}
\end{matrix}
$$

可以看到，$$x_{11}$$只参与到$$a_{11}$$的计算，那么也只对$$(net_j^{l+1})_{11}$$有贡献。其他可以类推

$$
\begin{matrix}
\frac {\partial E}{\partial x_i^l} & =\begin{bmatrix}
\frac {\partial E}{\partial x_{11}} & \frac {\partial E}{\partial x_{12}}  & \frac {\partial E}{\partial x_{13}}  \\
\frac {\partial E}{\partial x_{21}}  & \frac {\partial E}{\partial x_{22}}  & \frac {\partial E}{\partial x_{23}} \\
\frac {\partial E}{\partial x_{31}}  & \frac {\partial E}{\partial x_{32}}  & \frac {\partial E}{\partial x_{33}} 
\end{bmatrix}  
\end{matrix}
$$

$$
\begin{align}
\frac {\partial E}{\partial x_{11}} &= \frac {\partial E}{\partial a_{11}} \frac {\partial a_{11}}{\partial x_{11}} =\frac {\partial E}{\partial ((net^{l+1}_j)_{11})}\frac{\partial ((net^{l+1}_j)_{11})}{\partial a_{11}} k_{22} = (\delta_{j}^{l+1})_{11} k_{22} \\
\frac {\partial E}{\partial x_{12}}& = \frac {\partial E}{\partial a_{11}} \frac {\partial a_{11}}{\partial x_{12}} + \frac {\partial E}{\partial a_{12}} \frac {\partial a_{12}}{\partial x_{12}} = (\delta_{j}^{l+1})_{11} k_{21}+ (\delta_{j}^{l+1})_{12} k_{22} \\
\frac {\partial E}{\partial x_{13}} & = (\delta_{j}^{l+1})_{12} k_{21} \\
\frac {\partial E}{\partial x_{21}} & =  (\delta_{j}^{l+1})_{11} k_{12}   +(\delta_{j}^{l+1})_{21} k_{22} \\
\frac {\partial E}{\partial x_{22}} & =  (\delta_{j}^{l+1})_{11} k_{11}   + (\delta_{j}^{l+1})_{12} k_{12} + (\delta_{j}^{l+1})_{21} k_{21}   + (\delta_{j}^{l+1})_{22} k_{22} \\
\frac {\partial E}{\partial x_{23}} & =  (\delta_{j}^{l+1})_{12} k_{11}   + (\delta_{j}^{l+1})_{22} k_{21}\\
\frac {\partial E}{\partial x_{31}} & =  (\delta_{j}^{l+1})_{21} k_{12}   \\
\frac {\partial E}{\partial x_{32}} & =  (\delta_{j}^{l+1})_{21} k_{11} +(\delta_{j}^{l+1})_{22} k_{12}  \\
\frac {\partial E}{\partial x_{33}} & =  (\delta_{j}^{l+1})_{22} k_{11}    \\
\end{align} \\ 
$$

$$
\begin{matrix}
\frac {\partial E}{\partial x_i^l} & =\begin{bmatrix}
0 & 0 & 0 & 0 \\
0 &  (\delta_{j}^{l+1})_{11} &  (\delta_{j}^{l+1})_{12} & 0 \\
0 &  (\delta_{j}^{l+1})_{21} &  (\delta_{j}^{l+1})_{22} & 0 \\
0 & 0 & 0 & 0 \\  
\end{bmatrix}  
  *   
  \begin{bmatrix}
   k11,k12 \\
   k21,k22
  \end{bmatrix}

\end{matrix}
$$

$$
\begin{align}
\frac {\partial E}{\partial \beta_j^l} & = \sum_u\sum_v \frac {\partial E}{\partial ((net_j^l)_{u,v})} \frac {\partial ((net_j^l)_{u,v})}{\partial \beta_j^l} \\
&= \sum_u\sum_v (\delta_j^l)_{u,v} down(X_j^{l-1})_{u,v} \\
&= \sum_u\sum_v (\delta_j^l \circ down(X_j^{l-1}))_{u,v} \\
\frac {\partial E}{\partial b_i^l} & =  \sum_u\sum_v \frac {\partial E}{\partial ((net_j^l)_{u,v})} \frac {\partial ((net_j^l)_{u,v})}{\partial b_j^l} \\
&= \sum_{u,v} (\delta_i^j)_{u,v} \\
\end{align}
$$

$$\circ$$表示矩阵点乘

参考:
1.http://blog.csdn.net/kymowind/article/details/75206173
