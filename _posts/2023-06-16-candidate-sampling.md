---
  layout: post
  title: candidate sampling
  categories: MachineLearning
  tags:
--- 

本文是对[Candidate Sampling](https://www.tensorflow.org/extras/candidate_sampling.pdf)的翻译和整理

假设我们面临一个多分类或者多标签问题，类别集合为L。每个训练样本表示为$$(x_i, T_i)$$, $$x_i$$表示上下文(特征)向量，$$T_i$$为样本所属的类别（类别可以是一个集合set或者multiset）。比如给定句子前面的单词去预测下一个单词是什么(或者接下来的单词集合)。

我们希望可以学习一个函数$$F(x,y)$$，该函数表示在给定上下文向量$$x$$时类别y的概率。

穷举式的训练方法是LR或者softmax，这需要我们对L中的所有类别都要计算$$F(x,y)$$，当L数量很大时，计算成本会很高。

Candidate Sampling 解决该问题的方式是构造一个训练任务，对于每个训练样本$$(x_i, T_i)$$，我们只对一小部分候选类别$$C_i \in L$$计算$$F(x,y)$$。通常,候选类别$$C_i$$除了目标类别$$T_i$$之外，还包括随机采样的其他类别集合$$S_i \in L$$，即 $$C_i = T_i \cup S_i$$。$$S_i$$的随机选取可能依赖也可能不依赖$$x_i$$和$$T_i$$。

我们使用神经网络作为训练模型，代表$$F(x,y)$$的那一层可以通过对损失函数的反向传播来训练。

![Table of Candidate Sampling Algorithms](https://user-images.githubusercontent.com/1762074/246267828-2427921e-e8b1-4cab-a893-1f7235510bbd.png)

说明如下:

 $$ Q(y\vert x) $$是给定上下文x的情况下类别y在根据采样算法采样出来的类别集合中的概率（或者叫expect count）。 

$$K(x)$$是不依赖候选类别的任意函数。因为Softmax包含了一个归一化项，所以加入这一个函数并不影响计算出来的概率。

logistic training loss

$$
\begin{aligned}
&-\sum_{i}(\sum_{y \in POS_i} log(\sigma(G(x_i, y)))  +\sum_{y \in NEG_i}  log(1-\sigma(G(x_i, y)))) = \\ 
 &\sum_{i}(\sum_{y \in POS_i} log(1+exp(-G(x_i,y)) )+\sum_{y \in NEG_i} log(1+exp(G(x_i,y)) ))
\end{aligned}
$$

这是当y取1和-1时的logistic损失函数。

softmax training loss

$$
\begin{aligned}
&\sum_{i}(-log(\frac{exp(G(x_i,t_i))}{\sum_{y \in POS_i \cup NEG_i} exp(G(x_i,y))})) = \\
&\sum_{i}(-G(x_i,t_i) + log {\sum_{y \in POS_i \cup NEG_i} exp(G(x_i,y))})

\end{aligned}
$$

NCE和负采样都可以扩展到T是一个multiset的场景。此时,$$P(y\vert x)$$表示$$T_i$$中y的期望数量（expected count）。类似地，NCE，负采样和Sampled Logistic 也可以扩展到
$$S_i$$是一个multiset的场景。此时，$$Q(y\vert x)$$表示$$S_i$$中y的期望数量。

问题:
multiset是啥，在实际实现时有什么区别

Sampled Softmax
====

假设我们有一个单标签(single-label)问题。每个训练样本$$(x_i,\{t_i\})$$包含一个上下文向量和一个目标label。$$P(y\vert x)$$表示给定上下文是x时目标类别是y的概率。


我们要训练一个函数$$F(x,y)$$来输出softmax logits，也就是给定上下文后目标类别的相对对数概率(relative log probabilities)。

$$
F(x,y) = log(P(y|x)) + K(x)
$$

$$K(x)$$是不依赖y的任意函数。

在完整的softmax训练中，对每个训练样本$$(x_i,\{t_i\})$$,我们需要计算L中所有类别$$y$$的logits $$F(x, y)$$。 当L中类别很多时这个计算成本会很高。

在Sampled Softmax中，每个训练样本$$(x_i,\{t_i\})$$都根据采样函数$$Q(y\vert x)$$从L中采样一个负类集合$$S_i$$。L中的每个类别$$y$$都以概率$$Q(y\vert x)$$相互独立地包含在$$S_i$$中。

$$
P(S_i = S | x_i) = \prod_{y\in S} Q(y|x_i) \prod_{y\in {L-S}}(1-Q(y|x_i))
$$

假设候选类别集合$$C_i$$包含了正类和所有采样得到的负类

$$
C_i = S_i \cup \{t_i\}
$$

我们的任务就变成了在给定候选类别集合$$C_i$$的情况下，$$C_i$$中的哪个类别是目标类别。

对$$ C_i$$ 中的每个类别$$y$$ ,我们需要计算在给定$$(x_i,C_i)$$的条件下，y是目标类别的后验概率,记为$$P(t_i = y\vert x, C_i) $$。

根据贝叶斯公式:

$$

\begin{aligned}
P(t_i = y|x_i, C_i) &= \frac{P(t_i = y, C_i|x_i)} { P(C_i\vert x_i)}\\
&= \frac{P(t_i = y|x_i) P(C_i|t_i= y, x_i)}{P(C_i\vert x_i)}\\
&= \frac{P(y|x_i) P(C_i|t_i= y, x_i)}{P(C_i\vert x_i)}\\
\end{aligned}
$$

现在来计算$$P(C_i\vert t_i = y, x_i) $$， $$S_i$$可以包含也可以不包含y，但是必须包含$$C_i$$中的其他类别，必须不包含不在$$C_i$$中的其他类别。

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

这个就是输入给softmax 函数的logits,该softmax函数预测C中的哪个候选类别是正类。

softmax:

$$
P(t_i=y |x_i, C_i) = softmax(\theta_i) = \frac{exp ( \theta_i)} {\sum_{j=1}^V exp(\theta_j)}
$$

log softmax:

$$
log\ P(t_i=y |x_i, C_i) = log\ softmax(\theta_i) =  \theta_i - log {\sum_{j=1}^V exp(\theta_j)}
$$

$$\theta_i$$就是输入给softmax函数的logits，也就是$$log(P(y\vert\ x_i)) - log(Q(y\vert\ x_i))+ K'(x_i, C_i)$$，后面的$$ K'(x_i, C_i)$$由于和目标类别无关，是一个公共项，可以从softmax中提取出去，所以输入给softmax
的logits也就变成了$$log(P(y\vert\ x_i)) - log(Q(y\vert\ x_i))$$。

因为我们训练函数$$F(x,y)$$来近似$$log(P(y\vert x))$$,我们用DNN中的一层隐藏层来表示$$F(x,y)$$,减去$$log(Q(y\vert x))$$,把结果作为softmax函数的输入。

即:

$$
Training\ Softmax\ Input = F(x, y) − log(Q(y|x))
$$



Noise Contrastive Estimation(NCE)
====

每个训练样本$$(x_i,T_i)$$包含一个上下文向量和一个目标类别集合。$$T_i$$可能只包含一个类别，也可能是一个集合(set)，为了一般性这里使用multiset。

$$P(y\vert x)$$还是表示在给定上下文向量的条件下目标类别集合中某个类别的expected count。如果目标类别集合中没有重复元素，那么$$P(y\vert x)$$是一个概率。

$$
P(y\vert x) = E(T(y) \vert x)
$$

我们需要训练一个函数$$F(x,y)$$来近似给定上下文向量时目标类别集合中某个类别的expected count，当目标类别集合没有重复元素时，近似的就是log概率$$log(P(y\vert x))$$。

$$
F(x,y) = log(P(y|x))
$$

对每个样本$$(x_i, T_i)$$，我们挑选一个采样类别集合$$S_i$$，这个类别集合是一个multiset，也就是集合中的元素存在重复。一般情况下集合中的类别是无重复的，这里为了一般性集合中的元素可重复。采样算法依赖不依赖$$x_i$$都可以，但是不会依赖$$T_i$$。
我们构造一个multiset集合$$C_i = T_i + S_i $$，包含了所有的正类和采样得到的负类集合。

我们的训练任务是从所有候选采样类别中区分出真正的类别。对于正类集合$$T_i$$中的每个类我们都有一个正样本，对于采样的负类集合$$S_i$$中的每个类我们都有一个负样本。

我们用$$Q(y\vert x)$$表示采样后的类别集合中某个类别的期望次数。如果$$S$$不包含重复元素，那么$$Q(y\vert x)$$就是一个概率。

$$
Q(y\vert x) := E(S(y)|x)
$$

对数几率如下：

$$
log\ odds (y 来自 T\ vs\ S\ \vert x) = log \frac{P(y|x)}{Q(y|x)} = log(P(y|x)) - log(Q(y|x))
$$

$$log(P(y\vert x))$$我们通过训练$$F(x,y)$$来近似，DNN中的一个隐层表示$$F(x,y)$$。加上第二项$$-log(Q(y\vert x))$$，这一项可以有解析解，然后把结果作为logistic函数的输入，label表示y来自于T还是S。

$$
Logistic\ Regression\ Input = F(x, y) − log(Q(y|x))
$$

也是通过DNN的反向传播来训练$$F(x,y)$$。

这里说下为什么用log odds，因为在LR中，log odds得到的就是线性部分的值，也就是LR的输入。

这里也可以看到，NCE中负采样中是可以包含正类的。

Negative Sampling
====

这个是NCE的简化版本，在训练时忽略掉$$-logQ(y\vert x)$$这一项。所以F(x,y)用来近似$$log(E(y\vert x) - log(Q(y\vert x)))$$。

值得注意的是，这里我们优化$$F(x,y)$$用来近似依赖于采样分布$$Q(y\vert x)$$的某个量，会导致结果高度依赖采样分布的选择，而其他算法并没有这个问题。

Sampled Logistic
====

这个是NCE的一个变体，区别在于采样过程中丢弃了刚好是目标类别的采样。(这里的without replacement 是不放回采样，但是不太明白这里的影响是啥,这里应该是说采样时是无放回的，所以采样到的类别是不会重复的)，这就要求$$T_i$$是一个集合，而不是multiset，当然$$S_i$$可以是
一个multiset。这样，我们学习到的其实是一个类别的log odds，而不是log-probability(区别是啥？)。具体的数学表达式也就从NCE的形式变成了如下形式:

$$
log\ odds(y\ 来自\ T\ vs\ (S − T) | x) = log \frac{P(y|x)}{Q(y|x)(1−P(y|x)}) = log(\frac{P(y|x)}{1-P(y|x)}) - log(Q(y|x))
$$

分母中的$$Q(y\vert x)$$表示从S中采样得到类别y的概率，$$1-P(y\vert x)$$表示S中是不不包含类别$$y$$的。

此时，F(x,y)要预估的量就变成了$$log\frac{P(y\vert x)}{1-P(y\vert x)}$$。

$$
 Logistic\ Regression\ Input = F(x, y) − log(Q(y|x))
$$

Context‐Specific vs. Generic Sampling
====

上面讨论的采样算法可以依赖于上下文。对于某些模型来说，上下文相关的采样可能会很有用，这样可以产生依赖上下文的hard negative，提供更有用的训练信息。到目前位置，这些算法的作者只关注使用通用采样算法比如均匀采样或者unigram采样，没有考虑利用上下文信息。

Batchwise Sampling
====

上述算法对于一个batch内的样本都使用相同的采样结果。这有点反直觉，因为按照直觉来说每个样本使用不同的采样结果会收敛更快。而现在整个batch使用相同的采样结果的原因主要是计算效率上的考虑。

在我们的模型中，F(x,y)是通过上下文的特征向量（神经网络的最后一个隐层）和类别的embedding向量的内积来计算的。多个特征向量和多个embedding向量之间的内积是通过矩阵乘法计算的，
在现在的硬件尤其是GPU上速度很快。在batch维度执行这个操作即使使用成百上千个采样类别也不会有显著的速度下降。

另外一个原因是不同设备之间拉取类别的embedding向量的开销比计算成百上千的特征向量的内积要高。
参考：

https://www.tensorflow.org/extras/candidate_sampling.pdf

https://www.zhihu.com/question/50043438


