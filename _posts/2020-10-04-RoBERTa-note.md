---
  layout: post
  title: RoBERTa
  categories: MachineLearning
  tags:
---


该论文研究了不同超参数和数据集大小对bert效果的影响。并发现bert在充分调优后可以超过所有已发布的预训练模型。文中的模型在GLUE, RACE and SQuAD都取得了STOA。

作者提出的改进如下：
1.在更多数据上使用更大的batch size 训练更长时间
2.去掉NSP任务
3.在更长的序列上进行训练
4.动态改变mask方式
作者也发布了一个数据集CC-NEWS 以便更好的测试数据集的大小对效果的影响。

作者也强调MLM方法在正确的设计下和其他最近发布的方法相比仍然很有竞争力。

### Bert 

模型输入格式如下：
[CLS], x1, . . . , xN, [SEP], y1, . . . , yM, [EOS].

其中，X_i 是第一个segment。Y_i是第2个segment。M+N<T。T是预定义的序列的最长长度。

Bert 使用MLM 和NSP 两个任务进行预训练。

MLM 使用交叉熵损失，其中15%的token进行mask。其中，又有80%使用[MASK]填充。10%不变，10% 随机替换。
NSP 是个二分类任务。

Bert 使用Adam 优化，相关参数如下：

    β1 = 0.9,β2 = 0.999, ǫ = 1e-6 and L2 weight decay of 0.01

lr 在前10,000 step进行warm up，直到1e-4，然后linear decay。

所有隐层和attention权重都使用0.1的dropout，激活函数为GELU。

总步数S = 1,000,000，batch size 256, 序列最大长度为512。


### 文中的实验设置

使用FAIRSEQ实现了Bert。设置和原始bert 有如下不同：

1.lr峰值不同，warm up step 不同
2.训练对Adam 的epsilon 值比较敏感，
3.训练时没有随机插入短序列，
4.前90%的更新没有使用reduced sequence length，而是全部都使用full length 序列进行训练



### 数据

BOOKCORPUS
CC-NEWS
OPENWEBTEXT
STORIES


### 评估

GLUE
General Language Understanding Evaluation benchmark 包含9个数据集评估NLU。任务都是单句分类或者句对分类。

SQuAD
Stanford Question Answering Dataset (SQuAD) 任务 是从文章中抽取答案来回答问题。

RACE
ReAding Comprehension from Examinations 是一个阅读理解任务。每个文章有多个问题，每个问题都需要从候选中选择一个正确答案。

### 训练过程分析

该部分探索并量化哪部分对BERT更重要。实验过程中模型结构不变：
BERTBASE (L =12, H = 768, A = 12, 110M params).


#### Static vs. Dynamic Masking


原始bert 中只有在数据预处理时对序列生成一次mask。训练时每个epoch 同一个序列的mask都是不变的。为了避免这个问题。训练数据重复10次，这样在40个epoch的训练过程中一个序列就会有10个不同的mask，即每个序列的每个mask在训练过程中会遇到4次。
作者将上面的mask策略和dynamic mask 策略进行对比。
dynamic的做法是每次遇到一个序列时都生成mask。



#### Model Input Format and Next Sentence Prediction

不同的研究对NSP loss 有不同的结论。 作者对该问题进行了研究

1.SEGMENT-PAIR+NSP

即原始bert 的设置

每个segment 可以包含多个句子

2.SENTENCE-PAIR+NSP

是一个句子对。由于两个句子长度一般远小于512，所以要增大batch size 以保证和1 中的token 数大致相同。

3.FULL-SENTENCES

每个样本包含从一个document或者多个document中采样出来的连续的句子，最大长度仍然是512。当到达一个文档截尾时，接着从下一个文档采样句子。文档之间增加一个分隔符。去掉NSP损失。

4.DOC-SENTENCES

和FULL-SENTENCES类似，就是不跨文档。如果到达文档截尾时不到512 token，那么提高 batch size 保证和FULL-SENTENCES的长度差不多。去掉了NSP损失。

1和2相比发现使用单句对下游任务是不利的，作者猜测原因是此时模型无法学习长距离依赖

1和4相比发现4由于Bert_{base}的结果，说明去掉NSP 对下游任务没有明显影响。和原始Bert论文中的结果不一致，可能是因为原始bert实现只去掉了NSP损失，而保留了segment-pair的输入格式。

3和4相比DOC-SENTENCES比FULL-SENTENCES效果会好一点。但是因为DOC-SENTENCES格式中batch size 会不同，所以剩余实验都使用FULL-SENTENCES格式。

#### Training with large batches


原始bert 使用256 sequence的batch size 训练1M 步。等价于2K的batch size 训练125K步，或者8K的batch size 31K步。


实验发现提高batch size 有助于提高效果,同时易于并行训练。剩余实验使用8K的batch size。

#### Text Encoding

Byte-Pair Encoding (BPE)是character-level 和word-level之间的一种表示方法,可以处理较大的词汇表。
BPE 依赖于sub-word。一般BPE 的词汇表大小在10K-100K之间。但是，在建模较大的数据集时，Unicode字符会占据词汇表相当大的比例。
使用byte可以学习一个中等大小（50K左右）的subword 词汇表。这个词汇表可以在不引入任务UNKNOWN字符的情况下编码任意输入文本。
原始bert 使用的character-level的BPE 词汇表大小为30K，词汇表是分词之后得到的。文中byte-level BPE词汇表，大小约为50K，不需要预处理或者分词。这样相对BERTBASE和 BERTLARGE大概增加了15M和20M的参数。


剩余实验都使用这个编码规则。

### RoBERTa

将上面的优化点都杂糅在一起，即RoBERTa for
Robustly optimized BERT approach。
使用dynamic mask、FULL-SENTENCES 、去掉NSP、更大的batch size、更大的byte-level BPE。
同时探索了其他两点：
1.预训练数据
2.训练轮数

最近的XLNET 使用了10倍于原始bert的数据，有一半step使用了8倍于bert的batch size。

训练RoBERTa 使用BERTLARGE的配置（L = 24,
H = 1024, A = 16, 355M parameters）。
实验证明 相同数据上RoBERTa效果优于BERTLARGE。证明这几点优化的效果。

同时，使用更多的数据、更多的训练步数 都可以提升效果。

接下来都使用效果最好的RoBERTa模型在三个benchmark上及进行实验：GLUE, SQuaD and RACE。





论文：

https://arxiv.org/pdf/1907.11692.pdf

https://github.com/brightmart/roberta_zh