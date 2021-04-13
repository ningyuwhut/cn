---
  layout: post
  title: 多分类gbdt推导
  categories: MachineLearning
  tags:
--- 

多分类gbdt的算法流程如下:

![多分类gbdt](https://user-images.githubusercontent.com/1762074/102714147-aeb71c00-4307-11eb-9dea-0bc083836890.png)

该算法的损失函数为交叉熵损失。

具体推导如下:

$$
p_k(x)=\frac{exp(F_k(x))}{\sum_{l=1}^K exp(F_l(x))}
$$

$$
L=-\sum_{k=1}^K y_k log p_k \\
 = -\sum_{k=1}^K y_k log\frac{exp(F_k(x))}{\sum_{l=1}^K exp(F_l(x))}  \\
$$

$$
L=-\sum_{k=1}^K y_k F_k(x)+ \sum_{k=1}^K y_k  log \sum_{l=1}^K exp(F_l(x)
$$

$$
L=-\sum_{k=1}^K y_k F_k(x) + log \sum_{l=1}^K exp(F_l(x))
$$

$$
\frac{\partial L}{\partial F_{k,m-1}(x)} =
-y_k + \frac{exp(F_k(x))}{\sum_{l=1}^K exp(F_l(x)) }\\
=-y_k + p_k(x)
$$

所以负梯度就是$$y_k - p_k(x)$$
