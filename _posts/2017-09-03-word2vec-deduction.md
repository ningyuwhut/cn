---
  layout: post
  title: word2vec推导
  categories: MachineLearning
  tags:
---

之前整理过word2vec的推导过程，整理后放在这。


统计语言模型
===
所谓统计语言模型就是来计算一个句子的概率的概率模型

假设句子$$W=(w_1, w_2,...,w_T)$$,该句子的概率:

$$
P(W)=P(w_1)P(w_2|w_1)P(w_3|w_1^2)...P(w_T|w_1^{T-1})
$$

其中,$$P(w_1^{T-1})$$表示单词序列$$W=(w_1,...w_{T-1})$$的概率

假设句子长度为T，词典大小为N，那么参数个数为$$TN^T$$

如何计算这些参数?
===

n_gram模型
====

$$
p(w_k|w_1^{k-1})=\frac{p(w_1^k)}{p(w_1^{k-1})}
$$
可以近似为
$$
p(w_k|w_1^{k-1})=\frac{count(w_1^k)}{count(w_1^{k-1})}
$$

n-gram的思想：

**一个词出现的概率只和它前面的n-1个词相关**。
$$
p(w_k|w_1^{k-1})=\frac{count(w_{k-n+1}^k)}{count(w_{k-n+1}^{k-1})}
$$

假设词典大小为20万，那么这里需要计算的参数个数与$$n$$的关系如下:

![这里写图片描述](http://img.blog.csdn.net/20170324143023866?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFyY2hfb24=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

可以看出,参数数量与$$n$$成指数关系，一般在应用时使用$$n=3$$居多。

n-gram模型在应用时首先统计语料中各个词串的次数，经过平滑处理将计算出的各种概率值存储下来。需要计算一个句子的概率时直接找到对应的概率相乘就得到了句子的概率。

但是更常见的套路是对问题建模，然后计算损失函数，最后优化损失函数得到最优的参数。然后在预测时使用最优的参数进行预测。比如，在统计语言模型中，我们首先定义一个
最大似然函数$$\prod_{w \in C} p(w\mid Context(w))$$ ,一般为了避免最大似然函数连乘导致的下越界,会对似然函数取对数，即对数似然函数$$L=\sum_{w\in C} log p(w\mid Context(w))$$,
这就是我们的目标函数。

现在的问题有两个:

1.如何对$$p(w\mid Context(w))$$建模;

2.如何最大化对数似然函数
 
那么如何计算p(w|Context(w))?

神经概率语言模型
====
bengio等人在03年提出了一种神经概率语言模型,该模型中用到了词向量。

所谓词向量，是指词典中的任何一个词，都有一个长度为m的向量表示$$v(w)\in R^m$$ ,这个向量即w的词向量

下图是bengio提出了神经概率语言模型网络图:

![这里写图片描述](http://img.blog.csdn.net/20170317134647791?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFyY2hfb24=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

输入层是单词$$w$$的context中的$$n-1$$个单词的词向量，投影层将输入层中的$$n-1$$个词向量连接起来,大小是$$(n-1)*m$$.

$$W$$和$$p$$分别是投影层和隐藏层之间的连接权重矩阵和偏置向量;$$U$$和$$q$$分别是隐藏层和输出层之间的连接权重矩阵和偏置向量;

前向传播过程如下：

$$
z_w= tanh(Wx_w+p) \\
y_w=softmax(Uz_w+q)
$$

$$
p(w|Context(w))=\frac{e^{y_{w,i_w}}}{\sum_{i=1}^N e^{y_{w,i}}}
$$
,$$i_w$$表示词w在词典D中的索引。

这里需要学习的参数包括:

1.词向量：$$v(w)\in R^m, w\in D $$ 以及填充向量

2.神经网络参数:

$$
W\in R^{n_h * (n-1)*m},p\in R^{n_h},U\in R^{N*n_h}, q\in R^N
$$

注意，权值矩阵的维度与各个层的维度相对应。

在这些维度中，context中的n一般不超过5，词向量的长度$$m$$一般在$$10^2$$量级，$$n_h$$一般也在$$10^2$$量级,$$N$$是词库的大小，一般在$$10^4~10^5$$这个量级。
所以,可以看出计算量比较大的应该都和$$N$$相关,主要集中在隐藏层和输出层的矩阵$$U$$和输出层的归一化运算。

与n-gram模型的优势:

1.词语之间的相似性可以通过词向量表示

2.基于词向量的模型自带平滑功能，无须额外处理

词向量的理解:

one-hot representation vs distributed representation.

最简单的词向量应该就是one-hot表示了，但是one-hot表示将语义全部集中到非零元素上，向量极其稀疏，且无法进行相关性的比较，维度还特别大.

下面介绍word2vec的两种模型，分别是CBOW和Skip-gram

CBOW  Skip-gram 
===

**CBOW 模型**
![这里写图片描述](http://img.blog.csdn.net/20170317140010889?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFyY2hfb24=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
               
**Skip-gram模型**
![这里写图片描述](http://img.blog.csdn.net/20170317135950828?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFyY2hfb24=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


对着两种模型，word2vec又分别使用了两种优化方法：Hierarchical Softmax和Negative Sampling。

Hierarchical Softmax
====

基于神经网络的语言模型的目标函数

CBOW：
$$
L=\sum_{w\in C} log p(w|Context(w))
$$

Skip-gram:
$$
L=\sum_{w\in C} log p(Context(w)|w)
$$

重点在于条件概率的计算


CBOW
===

样本（Context(w),w),其中Context(w)取w前后各c个词

1.输入层

包含Context中2c个词(前后各c个)的词向量。

2.投影层

将输入层的2c个向量累加，即
$$
x_w=\sum_{i=1}^{2c} v(Context(w)_i)\in R^m
$$

![这里写图片描述](http://img.blog.csdn.net/20170317142148808?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFyY2hfb24=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

3.输出层

输出层是一棵二叉树，它以语料中出现过的词为叶子节点，以词在语料中出现的次数构造出huffman树。在这棵树中，叶子节点共N个，分别对应词典D中的词，非叶子节点N-1个。

不同
--

与前面的神经语言网络不同:

1.输入层到投影层

拼接 vs 累加求和

2.有/无隐藏层

3.线性结构 vs 树形结构

下面对基于Hierarchical Softmax的CBOW模型进行推导

![这里写图片描述](http://img.blog.csdn.net/20170317143034868?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFyY2hfb24=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


![这里写图片描述](http://img.blog.csdn.net/20170317150307414?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFyY2hfb24=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


$$
p(w|Context(w)) = \prod_{j=2}^{l^w} p(d_j^w|x_w,\theta_{j-1}^w) \\
p(d_j^w|x_w,\theta_{j-1}^w)=\left\{
\begin{aligned}
\sigma(x_w^T\theta_{j-1}^w), d_j^w = 0 ;\\
1-\sigma(x_w^T \theta_{j-1}^w), d_j^w =1,  \\
\end{aligned}
\right.
$$
$$
p(d_j^w|x_w,\theta_{j-1}^w)=\sigma(x_w^T\theta_{j-1}^w)^{1-d_j^w} (1-\sigma(x_w^T \theta_{j-1}^w))^{d_j^w}
$$

$$
\begin{align*}
L&=\sum_{w \in C} log \prod_{j=2}^{l^w} p(d_j^w|x_w,\theta_{j-1}^w)  \\
 &=\sum_{w \in C} \sum_{j=2}^{l^w} log \left( \sigma(x_w^T\theta_{j-1}^w)^{1-d_j^w} (1-\sigma(x_w^T \theta_{j-1}^w))^{d_j^w} \right) \\
 &=\sum_{w \in C} \sum_{j=2}^{l^w} \left( (1-d_j^w) log \sigma(x_w^T\theta_{j-1}^w) +  (d_j^w) log (1-\sigma(x_w^T \theta_{j-1}^w)) \right) \\
\end{align*}
$$

然后进行梯度求导

参数包括:

输入向量$$x$$，树的每个非叶子节点的向量$$\theta_{j}^w$$，context中每个词的向量$$v$$。

$$
L(w,j)=(1-d_j^w)log[\sigma(x_w^T\theta_{j-1}^w)]+d_j^wlog[1-\sigma(x_w^T\theta_{j-1}^w)]
$$

$$
[log\sigma(x)]^{'}=1-\sigma(x)\\
[log(1-\sigma(x)]^{'}=-\sigma(x)
$$

利用上述公式进行如下推导:

$$
\begin{align*}
\frac{ \partial L(w,j) }{ \theta_{j}^w} &=\frac{\partial \left( (1-d_j^w)log[\sigma(x_w^T\theta_{j-1}^w)]+d_j^wlog[1-\sigma(x_w^T\theta_{j-1}^w)] \right) }{ \theta_{j}^w}  \\
&=(1-d_j^w) ( 1- \sigma(x_w^T\theta_{j-1}^w ) )x_w  - d_j^w\sigma(x_w^T\theta_{j-1}^w)x_w \\
&=\{ (1-d_j^w) ( 1- \sigma(x_w^T\theta_{j-1}^w ) ) - d_j^w\sigma(x_w^T\theta_{j-1}^w)\} x_w \\
&= \left[1-d_j^w -\sigma(x_w^T\theta_{j-1}^w) \right] x_w
\end{align*}
$$

所以,$$\theta_{j}^w$$的更新公式如下:

$$
\theta_{j}^w = \theta_{j-1}^w+\eta[1-d_j^w-\sigma(x_w^T\theta_{j-1}^w)]x_w
$$

关于$$x_w$$的偏导如下:

$$
\frac{\partial L(w,j) } {\partial{x_w}} = [1-d_j^w-\sigma(x_w^T\theta_{j-1}^w)]\theta_{j-1}^w. \\
$$

那么如何更新context中的词向量呢?word2vec中的做法如下：

$$
v(\tilde w) +\eta \sum_{j=2}^{l^w} \frac{\partial L(w,j)}{x_w}, \tilde w \in Context(w),表示context（w）中词的个数。
$$

即把$$\frac{\partial L(w,j) } {\partial{x_w}}$$贡献到$$context(w)$$中的每个词向量上

代码流程图如下:

![这里写图片描述](http://ww1.sinaimg.cn/large/6cbb8645gw1f5wqgz0elqj20pm0qaq5a.jpg)

Skip-gram 模型
===

![这里写图片描述](http://img.blog.csdn.net/20170317161718938?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFyY2hfb24=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

$$
L=\sum_{w\in C} log p(Context(w)|w) \\
p(Context(w)|w)=\prod_{\mu\in Context(w) } p(\mu|w) \\
p(u|w)=\prod_{j=2}^{l^u} p(d_j^u|v(w), \theta_{j-1}^u),
$$

$$
p(d_j^u|v(w), \theta_{j-1}^u)=[\sigma(v(w)^T \theta_{j-1}^u)]^{1-d_j^u}. [1-\sigma(v(w)^T \theta_{j-1}^u)]^{d_j^u}
$$

损失函数

$$
\begin{align*}
L&=\sum_{w \in C} log  \prod_{\mu\in Context(w) } \prod_{j=2}^{l^u} [\sigma(v(w)^T \theta_{j-1}^u)]^{1-d_j^u}. [1-\sigma(v(w)^T \theta_{j-1}^u)]^{d_j^u} \\
 &= \sum_{w \in C} \sum_{\mu\in Context(w) }\sum_{j=2}^{l^u} (1-d_j^u) log ( \sigma(v(w)^T \theta_{j-1}^u) ) + d_j^u log ( 1-\sigma(v(w)^T \theta_{j-1}^u) )
\end{align*}

$$

记
$$
L(w,u,j)=(1-d_j^u)log[\sigma(v(w)^T\theta_{j-1}^u)]+d_j^ulog[1-\sigma(v(w)^T\theta_{j-1}^u)]
$$

$$
\begin{align*}
\frac{\partial L(w,u,j)}{\theta_{j-1}^u} &= \frac{\partial } { \theta_{j-1}^u}  (1-d_j^u)log[\sigma(v(w)^T\theta_{j-1}^u)]+d_j^ulog[1-\sigma(v(w)^T\theta_{j-1}^u)] \\
&=(1-d_j^u)[ 1- \sigma(v(w)^T\theta_{j-1}^u) ]v(w) + d_j^u \sigma(v(w)^T\theta_{j-1}^u )v(w) \\
&=(1-d_j^u-\sigma(v(w)^T\theta_{j-1}^u ) ) v(w)
\end{align*}
$$

$$\theta_{j-1}^u$$的更新公式如下:

$$
\theta_{j-1}^u = \theta_{j-1}^u +\eta [1-d_j^u-\sigma(v(w)^T\theta_{j-1}^u)]v(w)
$$

$$v_w$$的更新如下:

$$
\frac{\partial L(w,u,j) } {\partial{v_w}} = [1-d_j^u-\sigma(v_w^T\theta_{j-1}^u)]\theta_{j-1}^u. \\
$$

$$
v(w)=v(w)+\eta \sum_{u \in Context(w) } \sum_{j=2}^{l^u}\frac{\partial L(w,u,j) }{\partial{v_w}}
$$

代码流程如下:

![这里写图片描述](http://img.blog.csdn.net/20170323173516678?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFyY2hfb24=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

以上是基于层次softmax的CBOW和Skip-Gram模型

Negative Sampling
==

假定一个单词w，对于Context（w）来说，w是正样本，它的负样本子集记做NEG(w)，且正样本的标签为1，负样本的标签为0.

CBOW
===

对于一个给定的正样本(Context(w), w),我们最大化

$$
g(w)=\prod_{u \in {w} \cup NEG(w)} p(u|Context(w)).
$$

其中,

$$
p(u|Context(w))=\left\{
\begin{aligned}
\sigma(x_w^T\theta^u), L^w(u) =1 ;\\
1-\sigma(x_w^T \theta^u), L^w(u) =0,  \\
\end{aligned}
\right.
$$

即

$$
p(u|Context(w))=[\sigma(x_w^T \theta^u)]^{L^w(u)}. [1-\sigma(x_w^T \theta^u)]^{1-L^w(u)}
$$

$$x_w$$表示Context(w)中各词的词向量之和,$$\theta^u \in R^m$$ 表示词u对应的辅助词向量。

最大化$$g(w)$$即最大化词为$$w$$的概率，同时最小化词为$$u,u \in NEG(w)$$的概率。

整个语料的似然函数:

$$
G=\prod_{w\in C} g(w)
$$

$$
\begin{align*}
L&=logG=log\prod_{w\in C} g(w) =\sum_{w\in C} log g(w) \\
 &=\sum_{w \in C} log \prod_{u\in {w} \cup NEG(w) } \{  [\sigma(x_w^T \theta^u)]^{L^w(u)}. [1-\sigma(x_w^T \theta^u)]^{1-L^w(u)} \} \\
 &=\sum_{w \in C} \sum_{u \in {w} \cup NEG(w) }\left( L^w(u) log \sigma(x_w^T \theta^u) + (1-L^w(u)) log [ 1-\sigma(x_w^T \theta^u )] \right) \\
 &=\sum_{w \in C} \left( log \sigma(x_w^T \theta^w) + \sum_{u \in NEG(w) } log [ 1-\sigma(x_w^T \theta^u )] \right) \\
 &=\sum_{w \in C} \left( log \sigma(x_w^T \theta^w) + \sum_{u \in NEG(w) } log [ \sigma( -x_w^T \theta^u )] \right)
\end{align*}
$$

最后一步用了$$1-\sigma(x)=\sigma(-x)$$这个推导

记
$$
L(w,u)=L^w(u).log[\sigma(x_w^T \theta^u)]+[1-L^w(u)]log[1-\sigma(x_w^T\theta^u)]
$$

对$$\theta^u$$求偏导:

$$
\begin{align*}
\frac{\partial L(w,u)}{\partial \theta^u} &= \frac{\partial} {\partial \theta^u } L^w(u).log[\sigma(x_w^T \theta^u)]+[1-L^w(u)]log[1-\sigma(x_w^T\theta^u)] \\
&= \left( L^w(u) ( 1- \sigma(x_w^T \theta^u ) ) - [1-L^w(u)] \sigma(x_w^T\theta^u ) \right) x_w \\
&= \left( L^w(u) - \sigma(x_w^T \theta^u ) \right) x_w
\end{align*}
$$

$$\theta^u$$的更新如下:

$$
\theta^u = \theta^u + \eta [ L^w(u)-\sigma(x_w^T\theta^u)]x_w\\
$$


对$$x_w$$的偏导如下:

$$
\frac{\partial L(w,u) } {\partial{x_w}} = [L^w(u)-\sigma(x_w^T\theta^u)]\theta^u. \\
$$

按如下公式更新上下文中的每个单词的词向量。

$$
v(\tilde w) = v(\tilde w) +\eta \sum_{u\in {w}\cup NEG(w)}\frac{\partial L(w,u) } {\partial{x_w}} , \tilde w \in Context(w)
$$


基于negative sampling的CBOW模型的伪代码如下:
![这里写图片描述](http://img.blog.csdn.net/20170324100120048?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFyY2hfb24=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

Skip-gram 模型
--
样本是(w,Context(w)),注意跟CBOW的区别

$$
G=\prod_{w\in C } g(w) \\
=\prod_{w\in C } \prod_{u\in Context(w) }g(u) \\
=\prod_{w\in C } \prod_{u\in Context(w) } \prod_{z \in {u} \cup NEG(u) }p(z|w)
$$

$$NEG(u)$$表示w的上下文单词u的负样本子集

$$
p(z|w)=\left\{
\begin{aligned}
\sigma(v(w)^T\theta^z), L^u(z) =1 ;\\
1-\sigma(v(w)^T \theta^z), L^u(z) =0,  \\
\end{aligned}
\right.
$$

似然函数如下:

$$
\begin{align*}
L&=log G=log \prod_{w\in C } \prod_{u\in Context(w) }g(u) = \sum_{w\in C} \sum_{u \in Context(w)} log g(u) \\
 &= \sum_{w\in C} \sum_{u \in Context(w)} log  \prod_{z \in {u} \cup NEG(u) }p(z|w)\\
 &= \sum_{w\in C} \sum_{u \in Context(w)} \sum_{z \in {u} \cup NEG(u) } log  p(z|w)\\
 &= \sum_{w\in C} \sum_{u \in Context(w)} \sum_{z \in {u} \cup NEG(u) } log \left( \sigma(v(w)^T\theta^z)^{L^u(z)} (1-\sigma(v(w)^T \theta^z))^{(1-L^u(z))}\right) \\
 &=\sum_{w\in C} \sum_{u \in Context(w)} \sum_{z \in {u} \cup NEG(u) } L^u(z) log\sigma(v(w)^T\theta^z) +(1-L^u(z)) log(1-\sigma(v(w)^T\theta^z)) 
\end{align*}
$$

实际采用的方法
==


上述推导需要对样本（w,Context(w))，针对Context(w)中的每一个词进行负采样。

源码上对w进行了|Context(w)|次负采样。
之前CBOW通过将上下文的词向量求和来使用，这里将Context（w）中的每一个词单独处理。

$$
g(w)=\prod_{\tilde w \in Context(w)} \prod_{u\in {w} U NEG^{\tilde w} (w)} p(u|\tilde w)
$$

$$
p(u|\tilde w)=\left\{
\begin{aligned}
\sigma(v(\tilde w)^T\theta^u), L^w(u) =1 ;\\
1-\sigma(v(\tilde w)^T \theta^u), L^w(u) =0,  \\
\end{aligned}
\right.
$$

损失函数如下:

$$
\begin{align*}
L&=log G = log \prod_{w\in C} g(w) = \sum_{w\in C} log g(w) \\
 &=\sum_{w\in C} log \prod_{\tilde{w} \in Context(w)} \prod_{\mu \in {w} \cup NEG^{\tilde{w}}(w)} \left( [\sigma(v(\tilde w)^T \theta^u) ]^{L^w(\mu)}[ 1-\sigma(   v(\tilde{w})^T \theta^{\mu} ) ]^{1-L^w(\mu)} \right) \\
 &=\sum_{w\in C} \sum_{\tilde{w} \in Context(w)}\sum_{\mu \in {w} \cup NEG^{\tilde{w}}(w)} { L^w(\mu) log [ \sigma(v(\tilde{w})^T\theta^\mu] + [1-L^w(\mu)] log [ 1- \sigma(v(\tilde{w})^T \theta^\mu)]}
\end{align*}
$$

$$
L(w,\tilde w, u)=L^w(u).log[\sigma(v(\tilde w)^T \theta^u)]+[1-L^w(u)]log[1-\sigma(v(\tilde w)^T\theta^u)]
$$


$$
\theta^u = \theta^u + \eta [ L^w(u)-\sigma(v(\tilde w)^T\theta^u)]v(\tilde w)\\
$$

$$
\frac{\partial L(w,\tilde w,u) } {\partial{v(\tilde w）}} = [L^w(u)-\sigma(v(\tilde w)^T\theta^u)]\theta^u. \\
$$

$$
v(\tilde w) = v(\tilde w) +\eta \sum_{u\in {w}U ENG(w)}\frac{\partial L(w,\tilde w ,u) } {\partial{v(\tilde w)}} , \tilde w \in Context(w)
$$


