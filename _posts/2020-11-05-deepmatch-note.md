---
  layout: post
  title: DeepMatch
  categories: MachineLearning
  tags:
---

run_sdm.py

使用movielens_sample.txt作为样本文件

特征如下：

````
'user_id', 'movie_id', 'gender', 'age', 'occupation', 'zip', 'genres'
````

代码中先用LabelEncoder将这些特征进行编码并转换为id。

并将特征分为用户特征和item特征。

用户特征：

````
["user_id", "gender", "age", "occupation", "zip", "genres"]
````

item特征：

````
["movie_id"]
````

然后对数据集进行处理,格式如下:

````
userid，
短序列的movie_id(按时间由近及远排列）、 
当前序列(hist)中除去短序列后剩余的movie_id序列作为长序列id、
最近的movie_id(作为当前待排序的movie_id)、
1(作为label),
短序列长度、
长序列长度、
最近的movie_id的评分、
短序列的genres、
长序列的genres
````

数据集中每一条是用户的一条观看记录，对用户聚类即得到该用户的观看序列。

假设序列完整长度为N，那么可以生成N个子序列，每个子序列都是完整序列的前K个元素（K>=1 and K < N)。

假设子序列为S，那么前S-1个子序列加入到训练集，最后一个子序列加入到测试集。

序列的label是当前序列S的最后一个movie，最后一个即最近的一个

将当前序列划分为长序列和短序列。

短序列是前seq_short_len个元素，后面的都是长序列。

如果当前序列长度不足seq_short_len，那么长序列就以0填充。

代码中根据当前序列长度和短序列的长度seq_short_len之间的关系，分成四种情况处理：

当前序列长度小于seq_short_len且不是完整序列的最后一个元素

当前序列长度大于seq_short_len且不是完整序列的最后一个元素

当前序列长度小于seq_short_len且是完整序列的最后一个元素

当前序列长度大于seq_short_len且是完整序列的最后一个元素


前两种情况加入到训练集，后两种情况加入到测试集。

最后，生成模型的输入格式。


输入特征分成两大块，一块是用户特征，一块是item特征。

用户特征又细分为用户的画像特征和行为序列特征。

user_id、gender、age、occupation、zip为画像特征

short_movie_id、prefer_movie_id、short_genres、prefer_genres为序列特征。

movie_id为item特征。


SDM.py


先针对各个特征建立embedding矩阵。

然后各个特征分别进行embedding_lookup。


得到用户特征的embedding user_emb_list

长序列的prefer_emb_list，短序列的prefer_emb_list。

然后

1.对用户的embedding输入 经过一个Dense层，得到输出user_emb_output。


2.调用AttentionSequencePoolingLayer 计算长序列和当前用户特征的attention（即DIN），输出都加入到列表prefer_att_outputs中。

然后prefer_att_outputs 经过一个Dense层，得到长序列的表示prefer_output。


3.短序列表示short_emb_list先经过一个Dense层，然后经过一个DynamicMultiRNN 层，最后再经过一个SelfMultiHeadAttention层，得到短序列的表示short_att_output。以用户的表示user_emb_output、短序列的表示short_att_output为输入，调用UserAttention 计算attention，得到短序列的表示short_output。


4.最后，以 用户的表示user_emb_output、短序列的表示short_output、长序列的表示prefer_output, 作为gate门的输入，得到融合后的向量。


gate门就是一个Dense层，得到一个融合后的向量，然后分别对short_output和prefer_output 进行融合。得到最后的输出gate_output。


最后，调用SampledSoftmaxLayer 计算item的表示和gate_output 之间的负采样的softmax 损失。

被采样的类别从哪来的呢？

DynamicMultiRNN

该类实现多层RNN，支持LSTM和GRU。包括dropout、残差两个组件。


num_residual_layers 表示需要残差连接的层数。

总层数为num_layers，那么i>=num_layers-num_residual_layers的层即需要进行残差连接的层。


SelfMultiHeadAttention

实现多头注意力机制，就是transformer中的encoder层。


DotAttention:
#内积attention，即Q*K/\sqrt(d_k)

SoftmaxWeightedSum:

#对attention 权重进行mask之后进行softmax，然后对value进行加权求和，得到加权后的序列输出

mask逻辑需要注意下。

key_mask 是key的掩码，有值的地方为1，否则为0。对于key_masks 为1的地方，直接使用权重。否则，填充一个巨小的值（这里是-2 ** 32 + 1），这样后面计算softmax的时候这种被padding的地方的权重就是0了。
计算出来的权重再对value进行加权，得到最终的输出结果。

代码中也实现了decoder阶段的mask。

DotAttention和SoftmaxWeightedSum合在一起就是多头注意力中的attention。



LayerNormalization:
实现LayerNorm

UserAttention

计算用户和key序列的attention。

attention 方法为DotAttention和SoftmaxWeightedSum。


ConcatAttention

实现了DIN，只负责计算权重。

AttentionSequencePoolingLayer

调用ConcatAttention 计算query和key直接的权重。然后调用SoftmaxWeightedSum 基于权重对keys进行加权。


EmbeddingIndex

不清楚这个有啥用

SampledSoftmaxLayer

调用sampled_softmax_loss计算采样后的softmax损失。

权重矩阵就是embeddings，


















