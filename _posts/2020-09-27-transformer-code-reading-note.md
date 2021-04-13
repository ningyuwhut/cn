---
  layout: post
  title: Transformer 源码阅读
  categories: MachineLearning
  tags:
---

源码地址：https://github.com/Kyubyong/transformer

带着问题阅读：

1.如何实现multi-head-attention，如何理解 multi-head的作用

2.self-attention如何理解，其中的Q、K、V的含义是啥

3.如何做mask？在encoder 和decoder 部分 mask 的具体区别和实现是怎样的

在decoder 中要实现两个mask
第一个mask 是屏蔽decoder 的输入序列中padding的部分。这个比较好理解

第二个mask 是屏蔽未来的输入。即假设当前处理到第N个位置，那从N+1往后的信息需要都屏蔽掉。

具体的实现方式是 使用一个下三角矩阵，生成mask， 上三角对应的部分置为0， 其余部分保持不变。

4.为啥要scaled

5.layer norm 的实现 及 用ln的原因, 和batch normalization的区别

6.在decoder部分两个multi-head的具体过程是啥

7.transformer 的encoder 的输入和decoder 的输入和输出分别是什么

8.position embedding 是咋实现的，怎么理解


download.sh 下载数据集

iwslt2016 de-en 数据集是一个德语到英语的机器翻译数据集。 

prepro.py 对数据集进行预处理，从中分别生成训练集、验证集和测试集。同时，使用BPE模型对训练集中的德语文件和对应的英文文件进行分词，得到分词模型和词典。
BPE 可以参考：https://zhuanlan.zhihu.com/p/86965595

接下来就是训练了，实现在脚本train.py中。

首先是生成流式输入:

````
  def get_batch(fpath1, fpath2, maxlen1, maxlen2, vocab_fpath, batch_size, shuffle=False):
````
该函数读取分词后的文件，然后生成batch

````
  shapes = (([None], (), ()),
              ([None], [None], (), ()))
    types = ((tf.int32, tf.int32, tf.string),
             (tf.int32, tf.int32, tf.int32, tf.string))
    paddings = ((0, 0, ''),
                (0, 0, 0, ''))
 dataset = tf.data.Dataset.from_generator(
        generator_fn,
        output_shapes=shapes,
        output_types=types,
        args=(sents1, sents2, vocab_fpath))  # <- arguments for generator_fn. converted to np string arrays
````

核心是generator_fn的实现


````
def generator_fn(sents1, sents2, vocab_fpath):
    '''Generates training / evaluation data
    sents1: list of source sents
    sents2: list of target sents
    vocab_fpath: string. vocabulary file path.

    yields
    xs: tuple of
        x: list of source token ids in a sent
        x_seqlen: int. sequence length of x
        sent1: str. raw source (=input) sentence
    labels: tuple of
        decoder_input: decoder_input: list of encoded decoder inputs
        y: list of target token ids in a sent
        y_seqlen: int. sequence length of y
        sent2: str. target sentence
    '''
    token2idx, _ = load_vocab(vocab_fpath)
    for sent1, sent2 in zip(sents1, sents2):
        # 输入序列sent1编码时加了一个结束符 </s>
        # 输出序列sent2编码时加了一个<s> </s>
        x = encode(sent1, "x", token2idx)
        y = encode(sent2, "y", token2idx)
        decoder_input, y = y[:-1], y[1:]

        x_seqlen, y_seqlen = len(x), len(y)
        yield (x, x_seqlen, sent1), (decoder_input, y, y_seqlen, sent2)
````

首先，加载词典，并返回token到词典索引的map。
然后，利用词典将德语句子和对应的英文翻译转换为id序列。
其中，德语句子最后加了一个</s>,英文句子外面分别套上了一层<s></s>。
不在词典中的token则使用unk的索引作为默认值。

注意返回的数据结构:

分词两部分：
输入部分包括编码后的token id 序列x，序列长度和对应的原始序列。
输出部分包括已经编码的输出序列decoder_input,目标token id 序列y,目标token id序列的长度和原始输出序列sent2。

decoder_input 表示在翻译过程中已经翻译出来的部分。

关于from_generator的output_shapes参数的说明参考:
https://stackoverflow.com/questions/48769142/tensorflow-how-to-use-dataset-from-generator-in-estimator

最后，对batch进行padding。

Transformer.py

encode

将输入序列转换为embedding，并进行了scale()
然后加上position embedding。

按照论文中的公式：

$$
PE(pos,2i) = sin(pos/10000^{2i/dmodel} ) \\
PE(pos,2i+1) = cos(pos/10000^{2i/dmodel} )
$$

实现如下：

````
  # First part of the PE function: sin and cos argument
        position_enc = np.array([
            [pos / np.power(10000, (i-i%2)/E) for i in range(E)]
            for pos in range(maxlen)])

        # Second part, apply the cosine to even columns and sin to odds.
        position_enc[:, 0::2] = np.sin(position_enc[:, 0::2])  # dim 2i
        position_enc[:, 1::2] = np.cos(position_enc[:, 1::2])  # dim 2i+1
````

然后进行dropout，作为接下来的输入，记为enc。

接下来就是multi-head attention结构

在encode阶段，multi-head attention的Q、K、V都是输入序列enc。

multi-head attention 有多层，前一层的输出作为后一层的输入。

````
def multihead_attention(queries, keys, values, key_masks,
                        num_heads=8, 
                        dropout_rate=0,
                        training=True,
                        causality=False,
                        scope="multihead_attention"):
    '''Applies multihead attention. See 3.2.2
    queries: A 3d tensor with shape of [N, T_q, d_model].
    keys: A 3d tensor with shape of [N, T_k, d_model].
    values: A 3d tensor with shape of [N, T_k, d_model].
    key_masks: A 2d tensor with shape of [N, key_seqlen]
    num_heads: An int. Number of heads.
    dropout_rate: A floating point number.
    training: Boolean. Controller of mechanism for dropout.
    causality: Boolean. If true, units that reference the future are masked.
    scope: Optional scope for `variable_scope`.
        
    Returns
      A 3d tensor with shape of (N, T_q, C)  
    '''
    d_model = queries.get_shape().as_list()[-1]
    with tf.variable_scope(scope, reuse=tf.AUTO_REUSE):
        # Linear projections
        Q = tf.layers.dense(queries, d_model, use_bias=True) # (N, T_q, d_model)
        K = tf.layers.dense(keys, d_model, use_bias=True) # (N, T_k, d_model)
        V = tf.layers.dense(values, d_model, use_bias=True) # (N, T_k, d_model)
        
        # Split and concat
        Q_ = tf.concat(tf.split(Q, num_heads, axis=2), axis=0) # (h*N, T_q, d_model/h)
        K_ = tf.concat(tf.split(K, num_heads, axis=2), axis=0) # (h*N, T_k, d_model/h)
        V_ = tf.concat(tf.split(V, num_heads, axis=2), axis=0) # (h*N, T_k, d_model/h)

        # Attention
        outputs = scaled_dot_product_attention(Q_, K_, V_, key_masks, causality, dropout_rate, training)

        # Restore shape
        outputs = tf.concat(tf.split(outputs, num_heads, axis=0), axis=2 ) # (N, T_q, d_model)
              
        # Residual connection
        outputs += queries
              
        # Normalize
        outputs = ln(outputs)
 
    return outputs
````

````
def scaled_dot_product_attention(Q, K, V, key_masks,
                                 causality=False, dropout_rate=0.,
                                 training=True,
                                 scope="scaled_dot_product_attention"):
    '''See 3.2.1.
    Q: Packed queries. 3d tensor. [N, T_q, d_k].
    K: Packed keys. 3d tensor. [N, T_k, d_k].
    V: Packed values. 3d tensor. [N, T_k, d_v].
    key_masks: A 2d tensor with shape of [N, key_seqlen]
    causality: If True, applies masking for future blinding
    dropout_rate: A floating point number of [0, 1].
    training: boolean for controlling droput
    scope: Optional scope for `variable_scope`.
    '''
    with tf.variable_scope(scope, reuse=tf.AUTO_REUSE):
        d_k = Q.get_shape().as_list()[-1]

        # dot product
        outputs = tf.matmul(Q, tf.transpose(K, [0, 2, 1]))  # (N, T_q, T_k)

        # scale
        outputs /= d_k ** 0.5

        # key masking
        outputs = mask(outputs, key_masks=key_masks, type="key")

        # causality or future blinding masking
        if causality:
            outputs = mask(outputs, type="future")

        # softmax
        outputs = tf.nn.softmax(outputs)
        attention = tf.transpose(outputs, [0, 2, 1])
        tf.summary.image("attention", tf.expand_dims(attention[:1], -1))

        # # query masking
        # outputs = mask(outputs, Q, K, type="query")

        # dropout
        outputs = tf.layers.dropout(outputs, rate=dropout_rate, training=training)

        # weighted sum (context vectors)
        outputs = tf.matmul(outputs, V)  # (N, T_q, d_v)

    return outputs
````

这里说明一下mask

有两种类型的mask
一种是在encode阶段计算attention时对key进行mask。


key_mask 是一个维度为[batch_size,d_head]的矩阵，经过预处理，变成[batch_size*num_head, 1, d_head]的矩阵。

其中，d_model = num_head*d_head。

然后key_mask中被padding的元素值为1，有值的地方值为0。
然后key_mask 乘以一个极小数（代码中是-2 ** 32 + 1），加回到输入inputs中，即对inputs进行了mask。
此时，inputs中有值的地方不变，被padding的地方就接近-2 ** 32 + 1。

这样，后面在经过softmax时，被padding的地方的权重值就是0了，从而实现mask。

在decode时，由于当前token只知道前面的token，所以，此时要把后面的token给mask掉。

softmax 之前， logits矩阵是一个dq*dk的矩阵。每行都是当前token作为query时 整个序列上每个token的权重。
假设对于第i行的token，此时，只有0-(i-1)的token是可见的，所以i+1之后的token 都需要被padding掉。
所以，可以生成一个dq*dk维度的下三角矩阵，对于上三角中为0的元素，logits都填充为-2 ** 32 + 1，下三角的logits保持不变。
这样，softmax 后 第i个token做query时，第i个token之后的token都会被mask掉。



decode

对

decode阶段，输入序列是decoder_inputs，这是输出序列的前n-1个token组成的序列。输出的label是y，这是输出序列的第一个到最后一个token组成的序列，是decoder_inputs对应的label。即使用前面i-1个token预测第i个token。

对decode_inputs的处理和encode阶段相同，将该阶段的输出记为dec。

在decode中，有两层multi-head attention。

第一层attention中，Q、K、V都是dec，进行self-attention。将该层的输出记为dec。

第二层attention中，Q是dec。K、V都是encode阶段的输出。即利用源序列的编码结果对输出序列进行解码。

输出矩阵记为dec，维度为[N,T2,d_model]。

此时，将dec和embeddings矩阵相乘，得到每个token处下一个token的概率分布。结果矩阵为(N, T2, vocab_size)。
这样，取 logits最大的token作为预估的类别。

这样，就得到了预估类别和真实label。

最后，计算损失。

````
 # train scheme
        y_ = label_smoothing(tf.one_hot(y, depth=self.hp.vocab_size))
        ce = tf.nn.softmax_cross_entropy_with_logits_v2(logits=logits, labels=y_)
        nonpadding = tf.to_float(tf.not_equal(y, self.token2idx["<pad>"]))  # 0: <pad>
        loss = tf.reduce_sum(ce * nonpadding) / (tf.reduce_sum(nonpadding) + 1e-7)
````

这里，对label作为label smoothing。 y的维度是[N, T2],one_hot后变成[N,T2,vocab_size]

softmax_cross_entropy_with_logits_v2 返回的是每个序列的每个token的loss矩阵。维度为[N,T2]

nonpadding 是得到有真实label的mask矩阵,被mask（值为<pad>对应的token id）的元素置为0，有token的元素值为1。

这样,ce*nonpadding 就把被mask的loss置为0了,求和之后就是整个batch的loss,最后除以非0的token数(tf.reduce_sum(nonpadding))即平均loss。


评估

评估时，由于是翻译任务，每个序列都只有一个起始token<s>，即decoder_inputs。

首先，对输入序列进行encode。
假设输出序列长度最大为maxlen2，那么依次遍历每个位置，基于当前的_decoder_inputs和输入序列的编码结果，输出_decoder_inputs中每个位置上概率最大的token id。然后将输出结果作为当前的翻译，并作为下一次迭代的输入。

````
            if tf.reduce_sum(y_hat, 1) == self.token2idx["<pad>"]: break
````
这句话怎么理解？