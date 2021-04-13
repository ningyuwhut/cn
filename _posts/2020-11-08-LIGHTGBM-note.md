---
  layout: post
  title: LightGBM
  categories: MachineLearning
  tags:
--- 


## LightGBM: A Highly Efficient Gradient Boosting Decision Tree


已有的GBDT实现在效率和可扩展性上仍然无法令人满意，主要原因是每次在选择特征分裂点时，都要遍历所有的样本并计算每个可能的分裂点的增益，这个过程十分耗时。为了解决该问题，文中提出两个算法：Gradient-based One-Side Sampling (GOSS) and Exclusive Feature Bundling (EFB).
使用GOSS，可以排除掉梯度较小的样本，只使用剩余的样本计算增益。文中证明，在计算信息增益时，梯度大的样本作用更大，所以GOSS可以使用更少的样本获得相当准确的增益值。通过EFB，将互斥的特征捆绑在一起，降低特征数量。互斥特征是指两个特征很少同时取非0值。作者证明，寻找互斥特征的最优捆绑是NP-hard的，但是使用一个贪心算法可以取得不错的近似，所以可以在不过多损失分裂点精度的条件下有效减少特征个数。作者把实现了GOSS和EFB算法的GBDT称之为LightGBM。


作者将EFB转换为一个图着色问题，特征是顶点，如果两个特征之间不互斥，则增加一条边。


GBDT 中寻找最优分裂点最常见的算法是pre-sorted算法。即先对特征值进行排序，然后遍历所有可能的特征值计算最优的分裂点。另外一个比较常见的是直方图算法![图片](../../ningyuwhut.github.io/pic/histogram-based-algo.png)。

直方图算法对连续特征进行离散化分桶（bin），使用bins来构建特征直方图。由于该算法在内存和训练速度上的优势，LightGBM也采用该算法。根据直方图算法，构建直方图的复杂度为O(#data*#feature)，分裂点复杂度为O(#bin *#feature)，


### Gradient-based One-Side Sampling

GOSS 首先根据梯度绝对值对样本进行排序，然后选择top a\*100% ，剩下的数据随机选择b\*100%(b是指占全量数据的比例),然后在计算增益时，将小梯度的数据的权重放大$\frac{1-a}{b}$，这样可以保证数据分布不变。

这里解释一下，原始数据中，大梯度样本比例为a，小样本比例为1-a,但是采样之后，大梯度和小梯度样本的比例为a:b,为了维持分布不变，小梯度样本乘以权$\frac{1-a}{b}$,这样，大梯度和小梯度样本比例还是a:1-a。


### Exclusive Feature Bundling

通过EFB，直方图构建复杂度由O(#data * #feature)下降到 O(#data * #bundle)，#bundle << #feature。

有两个问题：

1.将哪些特征bundle在一起

2.如何构建bundle

定理：

**The problem of partitioning features into a smallest number of exclusive bundles is NP-hard.**

由于第一个问题是NP-hard的，所以作者将其转换为图着色问题，以期得到一个比较好的近似。具体来说，将特征转换为图中的节点，如果两个特征不互斥，则在两个特征之间增加一条边。然后用一个贪心算法来生成bundle。另外，有一些特征，虽然不是完全互斥，但是也很少同时取非零值。如果算法允许少量冲突，我们就可以进一步减少bundle数，同时提高计算效率。通过计算，随机污染一小部分特征对准确率的影响有一个上限。

1.构建带权图，权重是两个特征之间的冲突。
2.根据节点的度对节点进行降序排列。
3.遍历排序后的特征，要么分配一个新的bundle，要么分配到一个已有的bundle。

该算法复杂度为O(#feature2)，且只在训练前处理一次。如果特征数不大时复杂度还可以接受，但是如果有数百万特征时还是有些问题。为了进一步提高效率，作者提出一个不需要建图的方法：根据非零值数量进行排序，类似于按度进行排序，因为非零值越多冲突的概率越高。

第二个问题，如何合并同一个bundle中的特征，关键是确保原始特征值在特征bundle中能够识别。由于使用直方图算法，所以可以将同一个bundle中的特征放在不同的bin中来构建bundle。文中给出了一个例子

````
suppose we have two features in a feature bundle. Originally, feature A takes value 
from [0; 10) and feature B takes value [0; 20). We then add an offset of 10 to the values of
feature B so that the refined feature takes 
values from [10; 30). After that, it is safe to merge features A and B, and use a feature bundle with range [0; 30] to replace the 
original features A and B
````

参考：

1.http://www.csuldw.com/2019/07/24/2019-07-24-an-introduction-tolightGBM-explained/

2.https://yunglinchang.blogspot.com/2019/07/lightgbm-efb.html




阅读lightgbm 源码


直方图算法


https://blog.csdn.net/anshuai_aw1/article/details/83040541

关于sklearn中的决策树是否应该用one-hot编码？


https://www.zhihu.com/question/266195966


https://blog.csdn.net/weixin_42001089/article/details/85343332

https://datascience.stackexchange.com/questions/41907/lightgbm-why-exclusive-feature-bundling-efb


vscode 编译c++程序

http://gitlinux.net/assets/LightGBM.pdf

