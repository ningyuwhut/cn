---
  layout: post
  title: SVM损失函数的梯度推导
  categories: MachineLearning
  tags:
---

在cs231n中assignment1中有一节需要优化svm的损失函数,所以需要求解svm的损失函数的梯度。

以下符号和作业中的符号保持一致。

首先，svm的损失函数：


$$
\begin{equation}
\label{exampleone}
L_i = \sum_{j\neq y_i} \max(0, w_j^T x_i - w_{y_i}^T x_i + \Delta) 
\end{equation}
$$

$$x_i$$表示第i个样本,为行向量。假设有N个样本,特征个数为D

$$y_j$$表示该样本的label，假设有C个类

$$\Delta$$是margin

$$w_j$$为第j个类的权重，为长度为D的列向量。

$$w_j$$为我们要学习的参数，总共有$$ C*D $$ 个，用$$W$$表示.

上面的损失可以看出SVM想要样本的正确类别$$y_i$$的得分$$w_{y_i}^T x_i$$比其他类别的得分都要高至少$$\Delta$$.

因为当$$w_j^T x_i  + \Delta < w_{y_i}^T x_i$$ 时，损失才为0.


损失函数$$L_i$$关于W的梯度可以表示如下:

$$
\nabla_{w} L_i 
  =
  \begin{bmatrix}
    \frac{dL_i}{dw_1} & \frac{dL_i}{dw_2} & \cdots & \frac{dL_i}{dw_C} 
  \end{bmatrix}
  = 
  \begin{bmatrix}
    \frac{dL_i}{dw_{11}} & \frac{dL_i}{dw_{21}} & \cdots & \frac{dL_i}{dw_{y_i1}} & \cdots & \frac{dL_i}{dw_{C1}} \\
    \vdots & \ddots \\
    \frac{dL_i}{dw_{1D}} & \frac{dL_i}{dw_{2D}} & \cdots & \frac{dL_i}{dw_{y_iD}} & \cdots & \frac{dL_i}{dw_{CD}} 
  \end{bmatrix}
$$

我们分析其中的一个元素$$\frac{dL_i}{dw_{11}}$$


$$
\begin{align*}
L_i = &\max(0, x_{i1}w_{11} + x_{i2}w_{12} \ldots + x_{iD}w_{1D} - x_{i1}w_{y_i1} - x_{i2}w_{y_i2} \ldots - x_{iD}w_{y_iD} + \Delta) + \\
 &\max(0, x_{i1}w_{21} + x_{i2}w_{22} \ldots + x_{iD}w_{2D} - x_{i1}w_{y_i1} - x_{i2}w_{y_i2} \ldots - x_{iD}w_{y_iD} + \Delta) + \\
&\quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \vdots \\
&\max(0, x_{i1}w_{C1} + x_{i2}w_{C2} \ldots + x_{iD}w_{CD} - x_{i1}w_{y_i1} - x_{i2}w_{y_i2} \ldots - x_{iD}w_{y_iD} + \Delta)
\end{align*}
$$

这个表达式其实是把上面的向量表达拆分成更具体的形式。


如果$$w_1^T x_i - w_{y_i}^T x_i + \Delta >0 $$

那么有

$$
\begin{equation}
\frac{dL_i}{dw_{11}} = x_{i1}
\end{equation}
$$

借助指示函数，可以表示为

$$
\begin{equation}
\frac{dL_i}{dw_{11}} = \mathbb{1}(w_1^Tx_i - w_{y_i}^T x_i+ \Delta > 0) x_{i1}
\end{equation}
$$

类似地:

$$
\begin{equation}
\frac{dL_i}{dw_{12}} = \mathbb{1}(w_1^Tx_i - w_{y_i}^Tx_i + \Delta > 0) x_{i2} \\
\frac{dL_i}{dw_{13}} = \mathbb{1}(w_1^Tx_i - w_{y_i}^Tx_i + \Delta > 0) x_{i3} \\
\vdots \\                                           
\frac{dL_i}{dw_{1D}} = \mathbb{1}(w_1^Tx_i - w_{y_i}^Tx_i + \Delta > 0) x_{iD}
\end{equation}
$$


所以:

$$
\begin{align*}
\frac{dL_i}{dw_{j}} &= \mathbb{1}(w_j^Tx_i - w_{y_i}^Tx_i + \Delta > 0)
  \begin{bmatrix}
  x_{i1} \\
  x_{i2} \\
  \vdots \\
  x_{iD}
  \end{bmatrix}
\\
&= \mathbb{1}(w_j^Tx_i - w_{y_i}^Tx_i + \Delta > 0) x_i^T \tag{2}
\end{align*}
$$


类似地，

$$
\begin{align*}
\frac{dL_i}{dw_{y_i}} &= - \sum_{j\neq y_i} \mathbb{1}(x_iw_j - x_iw_{y_i} + \Delta > 0)
  \begin{bmatrix}
  x_{i1} \\
  x_{i2} \\
  \vdots \\
  x_{iD}
  \end{bmatrix}
\\
&= - \sum_{j\neq y_i} \mathbb{1}(x_iw_j - x_iw_{y_i} + \Delta > 0) x_i^T \tag{3}
\end{align*}
$$
注意，这里是求和的。


根据上面的公式，实现非向量化的svm还是比较简单的，其实只要两层训练即可，

第一层循环遍历每个样本；第二层循环遍历每个类，

在第二层循环中需要完成两件事:
1.计算当前类的loss，
2.同时计算对当前类和样本正确类别的梯度。

```python
def svm_loss_naive(W, X, y, reg):

  """
  Structured SVM loss function, naive implementation (with loops).

  Inputs have dimension D, there are C classes, and we operate on minibatches
  of N examples.

  Inputs:
  - W: A numpy array of shape (D, C) containing weights.
  - X: A numpy array of shape (N, D) containing a minibatch of data.
  - y: A numpy array of shape (N,) containing training labels; y[i] = c means
    that X[i] has label c, where 0 <= c < C.
  - reg: (float) regularization strength

  Returns a tuple of:
  - loss as single float
  - gradient with respect to weights W; an array of same shape as W
  """

  dW = np.zeros(W.shape) # initialize the gradient as zero

  # compute the loss and the gradient
  num_classes = W.shape[1]
  num_train = X.shape[0]
  loss = 0.0
  for i in xrange(num_train):
    scores = X[i].dot(W) #该样本每个类的得分
    correct_class_score = scores[y[i]]
    for j in xrange(num_classes):
      if j == y[i]:
        continue
      margin = scores[j] - correct_class_score + 1 # note delta = 1
      if margin > 0:
        loss += margin
        dW[:,y[i]]+= -X[i] 
        dW[:, j] +=  X[i]

  # Right now the loss is a sum over all training examples, but we want it
  # to be an average instead so we divide by num_train.
  loss /= num_train
  dW /= num_train

  dW += reg * 2 * W

```

下面我们看看对应的矢量化的SVM该怎么写.

首先，我们可以通过矩阵乘法计算所有样本属于每个类的得分

$$
XW
$$

为了求得Loss，每个样本的每个类得分都要前去该样本正确类别的得分，同时还要加上$$\Delta$$.

最后，因为每个样本的Loss都是针对非正确样本进行求和的，所以还要把每个样本的正确类别的loss置为0.

实现上述功能可以借助numpy的索引功能:


下面看下怎么求梯度:

根据上面的公式，对于类别j的参数$$w_j$$,我们需要考虑两种情况:

一是对于所有不属于类别j的样本，这些样本中类别j的损失不为0 的样本都会参与到对$$w_j$$的更新
即公式2

二是对于所有属于类别j的样本，该样本每个损失不为0的类别都会更新一次$$w_j$$。

综上，我们需要一个矩阵，矩阵中的每一列对应该类下每个样本的指示变量：
如果该样本不属于该类且该类的损失不为0，则该元素为1，如果该类的损失为0则该元素为1；

如果该样本属于该类，那么该元素需要设置为该样本中损失不为0的类的个数，并且取负。

```
def svm_loss_vectorized(W, X, y, reg):$
  """$
  Structured SVM loss function, vectorized implementation.$
  Inputs and outputs are the same as svm_loss_naive.$
  """$
  loss = 0.0$
  dW = np.zeros(W.shape) # initialize the gradient as zero$
  num_train=X.shape[0]$
  #############################################################################$
  # TODO:                                                                     #$
  # Implement a vectorized version of the structured SVM loss, storing the    #$
  # result in loss.                                                           #$
  #############################################################################$
  pass$
  score=X.dot(W)$
  true_score=score[range(num_train), y] #这里还可以使用choose函数，$
  #参考https://stackoverflow.com/questions/17074422/select-one-element-in-each-row-of-a-numpy-array-by-column-indices$
  print (true_score.shape, true_score.reshape(-1,1).shape, np.matrix(true_score).shape)$
  score -= (true_score.reshape(-1,1) - 1) #delta=1, 每个样本每个类别的得分都要减去正确类别的得分$
  margin=np.maximum(0, score) #np.maxinum 对两个矩阵做element-wise的比较，生成一个同样维度的矩阵$
  margin[range(num_train),y] = 0  #置真实类的score为0$
  loss = np.mean( np.sum(margin, axis=1) )$
  loss += reg * np.sum(W * W)$
  #############################################################################$
  #                             END OF YOUR CODE                              #$
  #############################################################################$
  #############################################################################$
  # TODO:                                                                     #$
  # Implement a vectorized version of the gradient for the structured SVM     #$
  # loss, storing the result in dW.                                           #$
  #                                                                           #$
  # Hint: Instead of computing the gradient from scratch, it may be easier    #$
  # to reuse some of the intermediate values that you used to compute the     #$
  # loss.                                                                     #$
  #############################################################################$
  mask = margin$
  mask[margin>0] = 1$
  row_num = np.sum( mask, axis=1)$
  mask[ range(num_train), y] -= row_num.T $
  dW=np.dot(X.T, mask)$
  dW/=num_train$
  dW += 2*reg*W$
  #############################################################################$
  #                             END OF YOUR CODE                              #$
  #############################################################################$
  return loss, dW$
```

参考:

1.[https://mlxai.github.io/2017/01/06/vectorized-implementation-of-svm-loss-and-gradient-update.html]
