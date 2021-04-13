---
  layout: post
  title: ctr校准
  categories: MachineLearning
  tags:
--- 

ctr校准有两种方法，一个是Platt Scaling，一个是Isotonic Regression，中文即保序回归。


Platt Scaling 比较简单，是对模型的预估值使用sigmoid函数重新预估一遍，使用交叉熵损失进行训练。

保序回归更常见一些。该方法使用“Pool Adjacent Violators” PAV 算法 来进行校准。

该方法需要用到一个叫 reliability diagram的东西。这个东西先不提。

我们说下保序回归是怎么进行校准的。

首先，搜集reliability diagram需要的数据，即每个样本的预估值和真实值。

然后，按照样本的预估值进行排序，将样本分到不同的分桶中。统计分桶中的样本的预估均值和实际均值。

这个时候，**不同分桶中的实际均值序列可能并不是有序的，这时候我们应用PAV算法进行调整。调整后的实际均值序列就是有序的，即单调非递减。且不违背原始序列的排序。**

最后，将每个分桶的预估值和校准后的预估值（应该是真实均值）做成词表，线上先根据预估值判断落到了哪个分桶，然后使用这个分桶中的校准值作为校准后的预估值。

引用校准时需要特别注意的一点，为了不引入偏差，校准用的数据集和模型训练的数据集是不同的。


PAV算法如下:


reliability diagram



参考:

http://vividfree.github.io/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/2015/12/21/classifier-calibration-with-isotonic-regression



