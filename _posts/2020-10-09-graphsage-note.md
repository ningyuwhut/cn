
---
  layout: post
  title: GAT
  categories: MachineLearning
  tags:
---

代码地址：https://github.com/williamleif/GraphSAGE

## 数据集介绍

以protein-protein interaction(PPI)数据集为例。

该数据集共有24张图，其中20张作为训练，2张作为验证，2张作为测试，每张图对应不同的人体组织，该数据是为了从系统的角度研究疾病分子机制、发现新药靶点等等。平均每张图有2372个结点，每个结点特征长度为50，其中包含位置基因集，基序集和免疫学特征。基因本体集作为labels（总共121个），labels不是one-hot编码，所以这是一个多分类问题。

代码中的数据集并不完整，包含3张训练图，1张验证图，1张测试图，总共包含14755个节点,每个节点的特征为50。

example_data 目录下有该数据集,文件说明如下:

toy-ppi-G.json 描述图结构 每个节点有一个val和test属性表示属于验证集还是测试集。
toy-ppi-id_map.json 对图节点进行编码
toy-ppi-class_map.json 每个节点的类别label，一个节点可能属于多个类
toy-ppi-feats.npy 预训练好的节点embedding,顺序和id_map一致。
toy-ppi-walks.txt 图上的随机游走序列，每行有两个节点，表示从第一个节点游走到第二个节点。只用在无监督任务中。

## 日志

--base_log_dir 指定了日志目录，默认是当期目录。有监督模型输出F1得分，无监督模型输出节点的embedding，存储在npy格式文件val.npy 中，同时，使用val.txt 指定embedding的顺序，一行一个节点id。

## 运行

有两个shell文件:

example_supervised.sh 和example_unsupervised.sh 


无监督版本学习图中节点的embedding，有监督是对节点进行分类？

以example_unsupervised.sh 为例：

````
python -m graphsage.unsupervised_train --train_prefix ./example_data/toy-ppi --model graphsage_mean --max_total_steps 1000 --validate_iter 10
````
该命令中的train_prefix做了修改，原始代码里面是./example_data/ppi

## unsupervised_train.py

### load_data

加载数据，数据文件在上面简单说过了。
id_map.json是一个map结构，key是节点在图中的编号， value是新的编号
class_map.json也是一个map结构，key是节点在图中的编号，value是label向量。

加载完各个文件之后做了预处理。

1.对于图中的所有边，如果边的起点或者终点在验证集或者测试集中，则把边的train_removed属性置为True，表示不能用于训练。否则，置为False。

2.如果需要则对特征进行归一化

使用训练集中的所有节点特征计算均值方差，然后对图中所有节点进行Z-Score归一化。

图中的节点都被会被标注为属于训练集、验证集还是测试集。
图中的节点的特征向量会被归一化。


### train

首先，构建以边为单位的minbatch。

minbatch的构建是在EdgeMinibatchIterator类中实现的。

下面介绍该类。

首先，构建训练集的临接矩阵和出度列表,测试集的临接矩阵

两个数据集的临接矩阵都包含了图中所有的节点。

下面介绍训练集的临接矩阵的构造过程

临接矩阵的维度是[图中节点数量+1， 最大出度]，使用节点数量作为临接矩阵的初始值。

**行加1是为啥？**

每个节点采样的邻居数量为"最大出度”

**使用节点数量作为邻居矩阵的初始值是因为啥**

节点的出度都初始为0。

遍历所有节点，
  如果节点不在训练集中则跳过该节点，
  否则，遍历该节点的所有邻居，
    如果该邻居不在训练集中，则跳过该邻居；
    否则，将该邻居加入到当前节点的邻居列表中;
将该节点的出度设置为邻居列表的长度。
假设该节点符合条件的邻居数为N
如果N为0，则跳过该节点；
如果N大于最大出度，则进行无放回采样，保证邻居数等于最大出度
如果小于最大出度，则进行有放回采样，保证邻居数等于最大出度
最后，将该节点的采样到的邻居填充到临接矩阵中。

最后，返回临接矩阵和出度列表。

注意，邻接矩阵和出度列表都包含了图中所有的节点，
**只不过 只有训练集中的节点才真实记录了采样到的邻居节点，测试集、验证集中的节点和虽然在训练集中，但是邻居都不在训练集中的节点的邻居全部是初始化值，即节点数量。这个值超出了节点下标，所以可以认为是没有邻居。**
出度列表中也只有训练集中的节点真实记录了出度值，其余都为初始值0.

测试集构造临接矩阵的逻辑和训练集基本相同，只不过不再限制节点和邻居边都在训练集中，即节点和边都可以是训练集中出现过的。

测试集中不再计算节点的出度（原因是啥）

第二步，构造训练集的边集合和验证集的边集合。

这一部分好几个地方没明白。。。

random_context 表示是否使用random walk生成的边集合作为训练集，如果为false，则使用图中的边进行训练。

n2v_retrain 表示是否用来在node2vec模型中学习新的节点的embedding。

如果n2v_retrain为false，则去掉满足如下条件的边：
1.边的起点或者终点不在图的节点集合中；
2.起点或者终点的出度为0，且(起点不在测试集中或者在验证集中)和（终点不在测试集中或者在验证集中）(**这个是为啥？**)

将满足这两种条件的边去掉,剩余的边作为训练集中的边。
将train_removed标记为true的边作为验证集中的边,即起点或者终点不在训练集中的边都作为验证集的边。

如果n2v_retrain 为true，分为两种情况：
fixed_n2v 表示是否在当前节点集合基础上重新训练node2vec。
1.fixed_n2v也为true，表示在已有节点基础上重新训练n2v模型。
此时，去掉终点在测试集或者验证集中的边（？**这里是为啥**）

2.如果为false，表示在新节点上学习embedding，此时，训练集和验证集中的边不再做任何限制，都使用所有的边。


第三步，对邻居进行采样，并构建模型

对邻居采样在UniformNeighborSampler类中实现，该类对batch中的每个节点采样指定数量的邻居。

无监督的Graphsage模型在SampleAndAggregate中实现。

identity_dim 表示是否使用节点id作为特征。
在静态图中，可以使用节点id表示特征，即每个节点id都有一个embedding。此时，训练可能会变慢，但是效果会有提升。在动态图中，由于有新节点，所以此时就不能使用节点id作为特征了。

在模型中，inputs1表示边的起点，inputs2表示边的终点。

如果features为空，则只使用节点id作为特征。如果features不为空，则使用features作为节点特征。如果identity_dim也大于0，那么将节点id的embedding也加入到特征中。

构建模型

无监督学习中，边的终点是label,负采样的节点是负样本,负样本数量为neg_sample_size。

第一步，给batch中的每条边的**起点和终点**采样邻居节点。

采样信息都记录在layer_infos中的SAGEInfo对象中。

以边的起点为例：
第一跳的输入是batch中的边的起点,记录到samples中。
support_size 记录每个起始节点累计采样到的邻居节点数量。
起点节点的support_size为1.
要采样的邻居数量为layer_infos的最后一层中的num_samples。
把采样到的邻居也加入到samples中,同时，support_size 变为上一层的采样数量(1)*该层的采样数量。
第二跳的输入是第一跳采样到的邻居。
然后对这些邻居节点的邻居进行采样，采样邻居数量为layer_infos的倒数第二层的num_samples。
support_size 变为上一跳的采样数量*该跳的采样数量。

最后，返回每一跳采样到的节点（samples）和每个输入节点在每一跳采样的邻居数量

假设有两层，第一层的邻居数量为25， 第二层邻居数量为10

那么第一次在采样时，每个节点采样10个邻居，得到节点数为10*batch_size.
第二次在采样时，每个节点采样25个邻居，得到节点数为25*10*batch_size.

**为什么要倒着来呢？**

第二步，对采样到的邻居节点信息进行聚合。

还是上面的例子。 特征维度为50.

第一层聚合

输入:[batch*50, 10*batch*50, 10*25*batch*50]
aggregator的维度是[50, 128]，输入维度50，生成128

第一跳：
输入为[batch*50,10*batch*50]

邻居被reshape为[batch,10,50]
假设是MeanAggregator 
邻居特征矩阵的维度为[batch_size, 10,50],那么，
先对邻居求均值，得到聚合后的均值向量[batch_size,50].

然后分别对邻居的聚合向量和自身的特征向量进行线性映射。 
  Ws*self + Wn*neigh

最后，将两部分结果 concat或者相加 作为第一跳的聚合结果,如果是concat，输出维度为[batch_size,256]

第二跳

输入为[10*batch*50,10*25*batch*50]

邻居reshape为[10*batch,25,50]

先对邻居求均值，得到聚合后的均值向量[10*batch_size,50]

然后分别对邻居的聚合向量和自身的特征向量进行线性映射。 
  Ws*self + Wn*neigh

最后，将两部分结果concat 或者相加作为第二跳的聚合结果，如果是concat，输出维度为[10*batch_size,256]

第二层：
aggregator 的维度是[256,128],

第一跳：
输入是[batch*256,10*batch*256]

聚合过程和第一层一样，得到的输出维度[batch_size,256]

这也是起始节点最终聚合后的结果。

分别得到边的起点的表示和终点的表示(维度和起点相同）。

最后，对采样的负样本同样进行负采样和聚合，得到负样本的表示。

这样，起点、终点和负样本都得到表示了。

最后，计算损失函数。

函数的输入就是起点、终点和负样本的表示。

默认是交叉熵损失：

````
 def _xent_loss(self, inputs1, inputs2, neg_samples, hard_neg_samples=None):
        aff = self.affinity(inputs1, inputs2) #维度为[batch_size] #计算input1和input2的内积
        neg_aff = self.neg_cost(inputs1, neg_samples, hard_neg_samples)
        #维度为[batch_size ,20],每个样本都和20个负样本计算内积作为相似度
        true_xent = tf.nn.sigmoid_cross_entropy_with_logits(
                labels=tf.ones_like(aff), logits=aff) #正样本的交叉熵
        negative_xent = tf.nn.sigmoid_cross_entropy_with_logits( #负样本的交叉熵
                labels=tf.zeros_like(neg_aff), logits=neg_aff)
        loss = tf.reduce_sum(true_xent) + self.neg_sample_weights * tf.reduce_sum(negative_xent)
        return loss
````
affinity计算起点和终点的内积作为正样本得分，neg_cost计算起点和负样本的内积作为负样本得分。

然后分别计算正样本的损失和负样本的损失，最后对两部分进行加权求和，得到最终的loss。


到这里模型结构就搭建完了。

下面就要开始训练了。

每个epoch 刚开始先shuffle 训练集的边和节点，并且重置batch_num=0

在每一步，从训练集的边中按顺序取一个batch，并把batch中的边的起点feed给batch1，终点feed给batch2,起点和终点 就是模型的输入了。有了输入就可以计算损失了。

并且，每隔一定的step就在验证集上选择一个batch进行验证。

迭代，直至训练集中所有边都处理过了。

````
  def next_minibatch_feed_dict(self):
        start_idx = self.batch_num * self.batch_size
        self.batch_num += 1
        end_idx = min(start_idx + self.batch_size, len(self.train_edges))
        batch_edges = self.train_edges[start_idx : end_idx]
        return self.batch_feed_dict(batch_edges)

     def batch_feed_dict(self, batch_edges):
        batch1 = []
        batch2 = []
        for node1, node2 in batch_edges:
            batch1.append(self.id2idx[node1])
            batch2.append(self.id2idx[node2])

        feed_dict = dict()
        feed_dict.update({self.placeholders['batch_size'] : len(batch_edges)})
        feed_dict.update({self.placeholders['batch1']: batch1})
        feed_dict.update({self.placeholders['batch2']: batch2})
````

监督学习脚本是一个节点分类任务，节点的label 在数据中应该有。


参考：
https://blog.csdn.net/yyl424525/article/details/102966617
