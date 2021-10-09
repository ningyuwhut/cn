---
  layout: post
  title: 几种特征处理方法
  categories: MachineLearning
  tags:
--- 


1.coec特征

coec即click over expected click，可以用来处理某些bias问题,比如最简单的位置偏置。
以解决位置偏置的coec为例，假设计算广告商家的coec，先统计每个位置上的点击率，然后广告商家在每个位置上的曝光乘以各个位置上的点击率，得到广告商家的期望点击数，最后，广告商家的实际点击数除以期望点击数就是coec的ctr了。通过这种方式计算出来的ctr，去除了位置的影响，之前因为位置排名靠前而导致ctr虚高的广告商家的coec特征就会变低，而因为位置排名靠后而导致ctr偏低的广告商家的coec特征就会变高，更能反映真实的广告质量。

coec的分母是期望点击数，其实是一个基于总体统计出来的点击数均值，而coec就是衡量当前商家的点击数相对总体点击均值的比值。该值越大，说明该商家在总体上的排名越靠前。

2.特征平滑

主要是贝叶斯平滑

贝叶斯平滑的公式如下:

$$
\hat \gamma = \frac{C+\alpha}{I+\alpha + \beta}
$$


$$
\alpha = \overline X(\frac{\overline X(1-\overline X)}{S^2}-1)
$$

$$
\beta =  (1-\overline X)(\frac{\overline X(1-\overline X)}{S^2}-1)
$$

$$\overline X$$为均值，$$S^2$$为样本方差


看了一下，


参考:

http://www.cs.cmu.edu/~xuerui/papers/ctr.pdf

https://blog.csdn.net/jinping_shi/article/details/78334362


