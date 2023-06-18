---
  layout: post
  title: candidate sampling
  categories: MachineLearning
  tags:
--- 

本文是对[Candidate Sampling](https://www.tensorflow.org/extras/candidate_sampling.pdf)的翻译和整理

假设我们面临一个多分类或者多标签问题，类别集合为L。每个训练样本表示为$$(x_i, T_i)$$, $$x_i$$表示上下文(特征)，$$T_i$$为样本所属的类别（类别可以为1个或者多个）。比如给定句子前面的单词去预测接下来的单词是什么。

我们希望可以学习一个函数$$F(x,y)$$，该函数表示在给定上下文向量$$x$$时类别y的概率。

最直接的方法是LR或者softmax，需要我们对L中的所有类别都要计算$$F(x,y)$$，当L数量很大时，计算成本会很高。


Candidate Sampling 解决该问题的方式是构造一个训练任务，对于每个训练样本$$(x_i, T_i)$$，我们只对一小部分候选类别$$C_i \in L$$计算$$F(x,y)$$。通常,候选类别$$C_i$$除了目标类别$$T_i$$之外，还包括随机采样的其他类别集合$$S_i \in L$$，即 $$C_i = T_i \cup S_i$$。

我们使用神经网络进行训练，代表$$F(x,y)$$的那一层可以通过对损失函数的反向传播来训练。

![Table of Candidate Sampling Algorithms](https://user-images.githubusercontent.com/1762074/246267828-2427921e-e8b1-4cab-a893-1f7235510bbd.png)

说明如下:

 $$ Q(y\vert x) $$是给定x的情况下类别y在根据采样算法采样出来的类别集合中的概率（或者叫expect count）。 

$$K(x)$$是不依赖候选类别的任意函数。因为Softmax包含了一个归一化项，所以加入这一个函数并不影响计算出来的概率。

logistic training loss

$$
\begin{aligned}
&-\sum_{i}(\sum_{y \in POS_i} log(\sigma(G(x_i, y)))  +\sum_{y \in NEG_i}  log(1-\sigma(G(x_i, y)))) = \\ 
 &\sum_{i}(\sum_{y \in POS_i} log(1+exp(-G(x_i,y)) )+\sum_{y \in NEG_i} log(1+exp(G(x_i,y)) ))
\end{aligned}
$$

softmax training loss

$$
\begin{aligned}
&\sum_{i}(-log(\frac{exp(G(x_i,t_i))}{\sum_{y \in POS_i \cup NEG_i} exp(G(x_i,y))})) = \\
&\sum_{i}(-G(x_i,t_i) + log {\sum_{y \in POS_i \cup NEG_i} exp(G(x_i,y))})

\end{aligned}
$$

NCE和负采样都可以扩展到T包含多个类别的情况。此时,$$P(y\vert x)$$表示$$T_i$$中y的期望数量（expected count）。类似地，NCE，负采样和Sampled Logistic 也可以扩展到
$$S_i$$包含多个类别的情况。此时，$$Q(y\vert x)$$表示$$S_i$$中y的期望数量。


Sampled Softmax
====

假设我们有一个单标签(single-label)问题。每个训练样本$$x_i,{t_i}$$包含一个上下文向量和一个目标label。$$P(y\vert x)$$表示给定上下文是x时目标类别是y的概率。


我们要训练一个函数$$F(x,y)$$来输出softmax logits，也就是给定上下文后正负类别之间的log概率的比值。

$$
F(x,y) = log(P(y|x)) + K(x)
$$

$$K(x)$$是不依赖y的任意函数。

在完整的softmax训练中，对每个训练样本$$x_i,{t_i}$$,我们需要计算所有类别$$y\in L$$的logits $$F(x, y)$$。 当类别很多时这个计算成本会很高。

在Sampled Softmax 中，每个训练样本$$x_i,{t_i}$$都根据采样函数$$Q(y\vert x)$$采样一个负类集合$$S_i \in L$$。 $$S_i$$中的每个类别$$y\in L$$都和$$Q(y\vert x)$$相互独立。

$$
P(S_i = S | x_i) = \prod_{y\in S} Q(y|x) \prod_{y\in {L-S}}(1-Q(y|x))
$$

假设$$C_i$$包含了正类和所有采样得到的负类

$$
C_i = S_i \cup {t_i}
$$

我们的任务就变成了在给定集合$$C_i$$的情况下，$$C_i$$中的哪个类别是目标类别。

对$$ C_i$$ 中的每个类别，$$y \in C_i$$ ,我们需要计算在给定$$x_i,C_i$$的条件下，y是目标类别的后验概率,记为$$P(t_i = y\vert x, C_i) $$。

根据贝叶斯公式:

$$

\begin{aligned}
P(t_i = y|x_i, C_i) &= \frac{P(t_i = y, C_i|x_i)} { P(C_i,x_i)}\\
&= \frac{P(t_i = y|x_i) P(C_i|t_i= y, x_i)}{P(C_i, x_i)}\\
&= \frac{P(y|x_i) P(C_i|t_i= y, x_i)}{P(C_i, x_i)}\\
\end{aligned}
$$

现在来计算$$P(C_i\vert t_i = y, x_i) $$， $$S_i$$可以包含也可以不包含y，必须包含$$C_i$$中的其他元素，必须不包含不在$$C_i$$中的其他类别。


所以有

$$
\begin{aligned}
P(t_i = y|x_i, C_i) &=  \frac{P(y|x_i) \prod_{y'\in{C_i-{y}}}Q(y'|x_i) \prod_{y'\in(L-C_i)}(1-Q(y'|x_i))}{P(C_i|x_i)} \\
&=\frac{\frac{P(y|x_i)}{Q(y|x_i)} \prod_{y'\in{C_i}}Q(y'|x_i) \prod_{y'\in(L-C_i)}(1-Q(y'|x_i))}{P(C_i|x_i)} \\
&=\frac{\frac{P(y|x_i)}{Q(y|x_i)}}{K(x_i, C_i)}
\end{aligned}
$$

$$K(x_i, C_i)$$是不依赖y的函数。所以

$$
log(P(t_i=y |x_i, C_i)) = log(P(y|x_i)) - log(Q(y|x_i)) + K'(x_i, C_i)
$$

这个就是输入给softmax 函数的logits,该softmax函数预测C中的那个候选类别是正类。

因为我们训练函数$$F(x,y)$$来近似$$log(P(y\vert x))$$,我们把深层网络中的一层来表示$$F(x,y)$$,减去$$log(Q(y\vert x))$$,把结果作为softmax函数的输入。

即

$$
Training Softmax Input = F(x, y) − log(Q(y|x)
$$

NCE
====

假设我们有一个单标签(single-label)问题。每个训练样本$$x_i,{T_i}$$包含一个上下文向量和一个目标类别集合。$$T_i$$可能只包含一个类别，也可能包含多个类别，为了一般性这里使用包含多个类别的情况。

$$P(y\vert x)$$还是表示在给定上下文向量的条件下目标类别集合的概率（或者叫expected count）。我们需要训练一个函数$$F(x,y)$$来近似log(P(y\vert x))。

$$
F(x,y) = log(P(y|x))
$$

对每个样本$$(x_i, T_i)$$，我们挑选一个采样类别集合$$S_i$$。采样算法依赖不依赖$$x_i$$都可以，但是不会依赖$$T_i$$。我们构造一个集合$$C_i = T_i + S_i $$，包含了所有的正类和采样得到的负类集合。


$$Q(y\vert x)$$表示采样后的类别中某个类别的期望次数。

$$
log\ odds (y 来自 T\ vs\ S|x) = log \frac{P(y|x)}{Q(y|x)} = log(P(y|x)) - log(Q(y|x))
$$


$$
Logistic\ Regression\ Input = F(x, y) − log(Q(y|x))
$$

Negative Sampling
====

这个是NCE的简化版本，在训练时 忽略$$logQ(y\vert x)$$。所以F(x,y)用来近似$$log(E(y\vert x) - log(Q(y\vert x)))$$

值得注意的是，这里我们优化$$F(x,y)$$用来近似依赖于采样分布$$Q(y\vert x)$$的某个量。会导致结果高度依赖采样分布的选择，其他算法并没有这个问题。

Sampled Logistic
====

这个是NCE的一个变体，区别在于采样过程中丢弃了刚好是目标类别的采样。


$$
log\ odds(y\ came\ from\ T\ vs\ (S − T) | x) = log \frac{P(y|x)}{Q(y|x)(1−P(y|x)}) = log(\frac{P(y|x)}{1-P(y|x)}) - log(Q(y|x))
$$

$$log\frac{P(y|x)}{1-P(y|x)}$$ 就是F(x,y)要预估的量。

$$
 Logistic\ Regression\ Input = F(x, y) − log(Q(y|x))
$$

Context‐Specific vs. Generic Sampling
====

采样算法可以依赖于上下文。对于某些模型来说，上下文相关的采样可能会很有用。这样可以产生依赖上下文的hard negative，提供更有用的训练信息。通用采样算法比如均匀采样或者unigram采样没有利用上下文信息。

Batchwise Sampling
====

上述算法对于一个batch内的样本都使用相同的采样结果。按照直觉来说每个样本使用不同的采样结果会收敛更快。而这么做的原因主要是计算量上的考虑。

在我们的实现中，F(x,y) 是通过上下文的特征向量（神经网络的最后一个隐层）和类别的embedding向量的内积来计算的。多个特征向量和多个embedding向量之间的内积通过矩阵乘法计算，在现在的硬件尤其是GPU上速度很快。在batch维度执行这个操作
即使使用成百上千个采样类别也不会有显著的速度下降。

另外一个原因是不同设备之间拉取类别的embedding向量的开销比计算成百上千的特征向量的内积要高。
参考：

https://www.tensorflow.org/extras/candidate_sampling.pdf

https://www.zhihu.com/question/50043438


