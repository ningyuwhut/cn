---
  layout: post
  title: LightGBM:A Highly Efficient Gradient Boosting Decision Tree
  categories: MachineLearning
  tags:
--- 

已有的GBDT实现在效率和可扩展性上仍然无法令人满意，主要原因是每次在选择特征分裂点时，都要遍历所有的样本并计算每个可能的分裂点的增益，这个过程十分耗时。为了解决该问题，文中提出两个算法：Gradient-based One-Side Sampling (GOSS) and Exclusive Feature Bundling (EFB).
使用GOSS，可以排除掉大量梯度较小的样本，只使用剩余的样本计算增益。文中证明，在计算信息增益时，梯度大的样本作用更大，所以GOSS可以使用更少的样本得到相当准确的增益值。通过EFB，将互斥的特征捆绑在一起，降低特征数量。互斥特征是指两个特征很少同时取非0值。作者证明，寻找互斥特征的最优捆绑是NP-hard的，但是使用一个贪心算法可以取得不错的近似，所以可以在不过多损失分裂点精度的条件下有效减少特征个数。作者把实现了GOSS和EFB算法的GBDT称之为LightGBM。

GBDT 中寻找最优分裂点最常见的算法是pre-sorted算法。即先对特征值进行排序，然后遍历所有可能的特征值计算最优的分裂点。另外一个比较常见的是直方图算法,如下图所示：

![image](https://user-images.githubusercontent.com/1762074/118365100-e2a02000-b5cd-11eb-8d97-0bcc3516a053.png)

直方图算法对连续特征进行离散化分桶（bin），使用bins来构建特征直方图。由于该算法在内存和训练速度上的优势，LightGBM也采用该算法。根据直方图算法，构建直方图的复杂度为$$O(\#data*\#feature)$$，分裂点复杂度为$$O(\#bin *\#feature)$$， 由于bin的数量远小于feature数量，所以整体复杂度主要取决于建立直方图的复杂度。Lightgbm主要从降低样本量和特征数两个角度来优化GBDT。

### Gradient-based One-Side Sampling

![image](https://user-images.githubusercontent.com/1762074/118394780-551a0a00-b679-11eb-9d28-d654bff5ecc9.png)


由于GBDT原生不支持样本权重，所以AdaBoost上的采样方法不能直接应用到GBDT上。不过，GBDT中样本的梯度可以用来做采样。换句话说，如果一个样本的梯度很小，那么这个样本的训练误差也会比较小，说明已经训练得不错了。一个比较直接的想法就是丢掉这些小梯度的样本。但是，这样会改变样本分布，影响模型效果。 GOSS会保留所有大梯度的样本，对小梯度的样本进行采样，为了弥补对数据分布的影响，在计算增益时，GOSS对小梯度的样本引入一个常数乘子。具体如下：

1.首先根据梯度绝对值对样本进行排序，并选择top a\*100% ，剩下的数据随机选择b\*100%(b是指占全量数据的比例),

2.在计算增益时，将小梯度的数据的权重放大为$$\frac{1-a}{b}$$，这样可以保证数据分布不变。

这里解释一下，原始数据中，大梯度样本比例为a，小样本比例为1-a,但是采样之后，大梯度和小梯度样本的比例为a:b,为了维持分布不变，小梯度样本乘以权$$\frac{1-a}{b}$$,这样，大梯度和小梯度样本比例还是a:1-a。

GBDT中，信息增益经常使用节点分裂后的方差来衡量：

![image](https://user-images.githubusercontent.com/1762074/118395401-e2ab2900-b67c-11eb-868f-bf373a9d602d.png)

![image](https://user-images.githubusercontent.com/1762074/118395415-f5256280-b67c-11eb-9b1d-8874022912df.png)

在GOSS中，信息增益如下：

![image](https://user-images.githubusercontent.com/1762074/118395457-2140e380-b67d-11eb-8b3b-178efb3f1142.png)

文中证明，GOSS不会过多损失训练精度，同时也优于随机采样。证明过程没有看，跳过。


### Exclusive Feature Bundling

通过EFB，直方图构建复杂度由$$O(\#data * \#feature)$$下降到 $$O(\#data * \#bundle)$$，且$$\#bundle << \#feature$$。

有两个问题需要解决：

1.将哪些特征bundle在一起

2.如何构建bundle

定理：

**将特征划分到最小数量的互斥bundle问题 是NP-hard的**

由于第一个问题是NP-hard的，所以作者将其转换为图着色问题，以期得到一个比较好的近似解。具体来说，将特征转换为图中的节点，如果两个特征不互斥，则在两个特征之间增加一条边。然后用一个图着色问题上的贪心算法来生成bundle。另外，有一些特征，虽然不是完全互斥，但是也很少同时取非零值。如果算法允许少量冲突，我们就可以进一步减少bundle数，提高计算效率。通过计算，随机污染一小部分特征对准确率的影响有一个上限$$O([(1 − γ)n]−2/3)$$,$$\gamma$$ 是每个bundle中的最大冲突率。所以如果该值足够小，那么我们就可以在效果和速度上有一个比较好的权衡。

bundle生成算法如下：

1.构建带权图，权重是两个特征之间的冲突。
2.根据节点的度对节点进行降序排列。
3.遍历排序后的特征，要么分配一个新的bundle，要么分配到一个已有的bundle。

![image](https://user-images.githubusercontent.com/1762074/118396070-4d119880-b680-11eb-803a-c6ecfa1322c3.png)

该算法复杂度为$$O(\#feature^2)$$，且只在训练前处理一次。如果特征数不大时复杂度还可以接受，但是如果有数百万特征时还是有些问题。为了进一步提高效率，文中提出一个不需要建图的方法：根据非零值数量进行排序，类似于按度进行排序，因为非零值越多冲突的概率越高。

第二个问题，如何合并同一个bundle中的特征，关键是确保原始特征值在特征bundle中能够识别。直方图算法中将连续特征转换为离散的bin值进行存储，所以可以通过将同一个bundle中的特征放在不同的bin中的方式来构建bundle。文中给出了一个例子，假设一个bundle中有两个特征，特征A取值区间为[0;10),特征B取值区间为[0;20),那么可以给特征B的值增加为10的偏置，这样特征B取值区间为[10;30)。然后就可以合并A和B了，并使用取值区间为[0;30)的bundle来代替A和B。

算法如下:

![image](https://user-images.githubusercontent.com/1762074/118396314-8696d380-b681-11eb-9c7c-fadf0d0394d9.png)

文中最后提到直方图构图时可以使用一个table记录每个特征的非0的样本，这样直方图建立的复杂度由$$O(\#data)$$变成$$O(\#non\_zero\_data)$$。但是，这个方法需要在树的生长过程中对每个特征维护一个非零样本的表，额外增加了内存和计算消耗。LightGBM中实现了该优化。


在Lightgbm的主页，介绍了在实现上的优化。但是却没有提GOSS和EFB。

优化主要有以下几个方面:

1.速度和内存优化

很多GBDT实现默认使用预排序算法，LightGBM使用直方图算法，有如下优点：

a.降低计算每个分裂点的收益的开销

预排序算法的时间复杂度是$$O(\#data)$$
直方图建立的时间复杂度是$$O(\#data)$$,但是这是一次性的。一旦建立，基于直方图查找分裂点的复杂度是$$O(#bins)$$,而且bins的数量远小于data的数量。

b.直方图做差加速

父节点的直方图减去邻居的直方图就可以得到当前叶节点的直方图

所以只需要计算一个叶子节点的直方图，另外一个叶子节点通过做差即可。

c.减小内存

使用离散的bin值代替连续特征。如果bin的数量小，还可以使用占用空间更小的数据类型，比如 uint8_t来存储训练数据。

无须存储预排序时需要的其他信息

d.减小分布式学习时的通信消耗。

2.稀疏优化

  只需要$$ O(2 * #non_zero_data)$$ 构建直方图。

3.准确度优化

  Leaf-wise (Best-first) Tree Growth

  大部分算法使用level-wise的方式来构建树。
  ![image](https://user-images.githubusercontent.com/1762074/118399862-bfd73f80-b691-11eb-89e3-51ad82ea2a1a.png)

LightGBM使用leaf-wise的方式，每次迭代都会选择loss下降最大的叶子节点进行分裂。在叶子节点数量相同的情况下，leaf-wise方式比level-wise方式的损失更低。

但是，这种方法在数据量较小时容易过拟合，LightGBM 使用$$max_depth$$这种方式来限制树的深度。
![image](https://user-images.githubusercontent.com/1762074/118399941-2a887b00-b692-11eb-893a-179462bbdb92.png)

Optimal Split for Categorical Features

一般使用one-hot表示离散特征，但是这种方式不适合树模型，尤其是离散特征基数很大时，建成的树容易不平衡，且需要深度很大才能取得较好的效果。相比one-hot，一种更优的方式是把离散特征分成两个集合。如果有k个取值，那么$$2^{k-1}-1$$种可能的划分，对于回归树来说，更有效的算法可以达到$$O(k * log(k))$$的复杂度。做法是 根据累积统计值$$sum_gradient/sum_hession$$对直方图进行排序,然后在排序后的直方图中找到最优分裂点。

4.网络通信优化

在分布式环境下，只需要"All reduce”, “All gather” and “Reduce scatter这种聚合通信算法，比点对点通信性能更好。

5.分布式优化

特征并行

特征并行是指在决策树中并行寻找最优分裂点。传统做法如下：

1.纵向分割数据（不同机器使用不同的特征集合

2.worker在本地的特征集合中找到最优分裂点
3.互相将本地最优分裂点通知给其他worker节点，从而得到最优的分裂点
4.拥有最优分裂点的worker完成分裂操作，并把分裂结果发送给其他节点
5.其他worker根据获取到的数据完成分裂。

缺点：
有通信开销，时间复杂度是$$O(#data)$$,所以在样本数量大时特征并行加速不太好

需要传递分裂结果，复杂度为$$O(#data / 8)$$,一个数据一个bit。

LightGBM的做法是每个worker拥有整个样本数据，这样，就不需要传递分裂结果，因为每个worker都知道怎么分裂。
具体过程如下：

Workers find local best split point {feature, threshold} on local feature set.

本地特征集上找到最优分裂点{特征，阈值}

Communicate local best splits with each other and get the best one.

将本地最优的分裂结果发送给其他节点，从中找到最优的。

Perform best split.


完成最优切分。

Data Parallel

Traditional Algorithm


Partition data horizontally.

平行切分数据

Workers use local data to construct local histograms.

worker使用本地数据构建直方图

Merge global histograms from all local histograms.

从所有的本地直方图合并出全局直方图

Find best split from merged global histograms, then perform splits.

从全局直方图中找到最优分裂点并完成分裂。

缺点

High communication cost

如果用点对点通信，单台机器的复杂度为$$ O(\#machine * \#feature * \#bin)$$

如果用allreduce这种聚合通信，复杂度为$$O(2 * \#feature * \#bin)$$。


Data Parallel in LightGBM

使用Reduce Scatter 来合并直方图，

Instead of “Merge global histograms from all local histograms”, LightGBM uses “Reduce Scatter” to merge histograms of different (non-overlapping) features for different workers. Then workers find the local best split on local merged histograms and sync up the global best split.

As aforementioned, LightGBM uses histogram subtraction to speed up training. Based on this, we can communicate histograms only for one leaf, and get its neighbor’s histograms by subtraction as well.


Voting Parallel

Voting parallel further reduces the communication cost in Data Parallel to constant cost. It uses two-stage voting to reduce the communication cost of feature histograms[10].


参考：

1.http://www.csuldw.com/2019/07/24/2019-07-24-an-introduction-tolightGBM-explained/
2.https://yunglinchang.blogspot.com/2019/07/lightgbm-efb.html
3.https://blog.csdn.net/anshuai_aw1/article/details/83040541
4.https://www.zhihu.com/question/266195966
5.https://blog.csdn.net/weixin_42001089/article/details/85343332
6.https://datascience.stackexchange.com/questions/41907/lightgbm-why-exclusive-feature-bundling-efb
7.http://gitlinux.net/assets/LightGBM.pdf
8.https://lightgbm.readthedocs.io/en/latest/Features.html