---
  layout: post
  title: AdamW 
  categories: MachineLearning
  tags:
--- 


在模型训练中，我们已经会遇到过拟合问题，而解决过拟合的常用方法是在损失函数上增长正则项，防止学习到的参数绝对值过大。正则项一般是模型权重的模的和，一般常用的是L1正则和L2正则。这里我们只讨论L2正则。

假设增加了L2正则的损失函数如下:

$$
\begin{equation}
\widetilde{E}(\mathbf{w})=E(\mathbf{w})+\frac{\lambda}{2}\mathbf{w}^2
\end{equation}
$$

当使用SGD对参数进行更新时，

$$
\begin{equation}
w_i \leftarrow w_i-\eta\frac{\partial E}{\partial w_i}-\eta\lambda w_i
\end{equation}
$$

$$
\begin{equation}
w_i \leftarrow (1-\eta\lambda) w_i-\eta\frac{\partial E}{\partial w_i}
\end{equation}
$$

可以看到，损失函数加入L2正则之后，参数在更新的时候会减小$$-\eta\lambda w_i$$，这也是L2正则又被叫做weight decay的原因。

但是weight decay是否就等价于L2正则呢？

答案是否定的。

weight decay的表达式是这样的

$$
\begin{equation}
w_i \leftarrow (1-\lambda^\prime) w_i-\eta\frac{\partial E}{\partial w_i}
\end{equation}
$$

可以看到，weight decay 是在更新参数时直接减去一部分权重值。

当$$\lambda = \frac{\lambda^\prime}{\eta}$$时，使用SGD的L2正则和weight decay是等价的。

也就是说，如果有一个全局最优的weight decay权重$$\lambda^{\prime}$$,那么正则化系数
$$\lambda$$ 的最优值就和学习率$$\eta$$绑定在一起了。为了分离(decouple)这两个超参数的影响，[DECOUPLED WEIGHT DECAY REGULARIZATION](https://arxiv.org/pdf/1711.05101.pdf)提出decouple(分离的) weight decay 正则方法。

下图是在SGD上使用了该方法的SGDW算法

![sgdw](https://user-images.githubusercontent.com/1762074/103174156-c27bf880-489a-11eb-94d6-76e193ea5f27.png)

紫色部分是L2正则，绿色部分是decoupled weight decay。（注意，程序中多了一个$$\eta_t$$参数）

可以看到，L2正则方法在第6行梯度计算时加入了正则项部分的梯度，也就引入了正则项系数$$\lambda$$。然后在第8行计算动量时又引入了学习率$$\alpha$$，这时，正则项系数和学习率就耦合在一起了。但是，在decoupled weight decay中，第6行没有引入正则化系数，第8行也就没有没有将二者耦合在一起，在第9行时才执行weight decay操作。

但是，当使用其他自适应学习率的优化算法比如adam时，L2正则和weight decay就不是等价的了。

首先，L2正则中损失函数的梯度和正则项的梯度都会在计算学习率时被调整，而weight decay中只有损失函数的梯度才会被调整，weight decay操作和自适应学习率计算操作是分开的。
另外，在adam中，更新权重时会除以历史梯度的平方的加权平均,如果使用L2正则，那么梯度中会加入正则项梯度。而值比较大的参数，它的正则项的梯度也会比较大，此时它的历史梯度的平方的加权平均值会更大，所以相比梯度较小的参数，它的更新值反而会相对来说小一些。这显然是不合理的，因为越大的权重应该惩罚越大，更新值也会越大。但是，在weight decay中，所有参数的更新系数是一样的，所以权重值越大它的惩罚也就越大。

上面从两个方面说明了自适应学习率算法的L2正则和weight decay是不一样的。下面给出adam 在decouple weight decay下的更新算法

![adamw](https://user-images.githubusercontent.com/1762074/103175043-07eff400-48a2-11eb-8cb8-6e07358035fb.png)


下面是bert中实现的该算法

````
class AdamWeightDecayOptimizer(tf.train.Optimizer):
  """A basic Adam optimizer that includes "correct" L2 weight decay."""

  def __init__(self,
               learning_rate,
               weight_decay_rate=0.0,
               beta_1=0.9,
               beta_2=0.999,
               epsilon=1e-6,
               exclude_from_weight_decay=None,
               name="AdamWeightDecayOptimizer"):
    """Constructs a AdamWeightDecayOptimizer."""
    super(AdamWeightDecayOptimizer, self).__init__(False, name)

    self.learning_rate = learning_rate
    self.weight_decay_rate = weight_decay_rate
    self.beta_1 = beta_1
    self.beta_2 = beta_2
    self.epsilon = epsilon
    self.exclude_from_weight_decay = exclude_from_weight_decay

  def apply_gradients(self, grads_and_vars, global_step=None, name=None):
    """See base class."""
    assignments = []
    for (grad, param) in grads_and_vars:
      if grad is None or param is None:
        continue

      param_name = self._get_variable_name(param.name)

      m = tf.get_variable(
          name=param_name + "/adam_m",
          shape=param.shape.as_list(),
          dtype=tf.float32,
          trainable=False,
          initializer=tf.zeros_initializer())
      v = tf.get_variable(
          name=param_name + "/adam_v",
          shape=param.shape.as_list(),
          dtype=tf.float32,
          trainable=False,
          initializer=tf.zeros_initializer())

      # Standard Adam update.
      next_m = (
          tf.multiply(self.beta_1, m) + tf.multiply(1.0 - self.beta_1, grad))
      next_v = (
          tf.multiply(self.beta_2, v) + tf.multiply(1.0 - self.beta_2,
                                                    tf.square(grad)))

      update = next_m / (tf.sqrt(next_v) + self.epsilon)

      # Just adding the square of the weights to the loss function is *not*
      # the correct way of using L2 regularization/weight decay with Adam,
      # since that will interact with the m and v parameters in strange ways.
      #
      # Instead we want ot decay the weights in a manner that doesn't interact
      # with the m/v parameters. This is equivalent to adding the square
      # of the weights to the loss with plain (non-momentum) SGD.
      if self._do_use_weight_decay(param_name):
        update += self.weight_decay_rate * param

      update_with_lr = self.learning_rate * update

      next_param = param - update_with_lr

      assignments.extend(
          [param.assign(next_param),
           m.assign(next_m),
           v.assign(next_v)])
    return tf.group(*assignments, name=name)

  def _do_use_weight_decay(self, param_name):
    """Whether to use L2 weight decay for `param_name`."""
    if not self.weight_decay_rate:
      return False
    if self.exclude_from_weight_decay:
      for r in self.exclude_from_weight_decay:
        if re.search(r, param_name) is not None:
          return False
    return True

  def _get_variable_name(self, param_name):
    """Get the variable name from the tensor name."""
    m = re.match("^(.*):\\d+$", param_name)
    if m is not None:
      param_name = m.group(1)
    return param_name
````





参考:

1.https://zhuanlan.zhihu.com/p/40814046

2.https://arxiv.org/pdf/1711.05101.pdf

3.https://stats.stackexchange.com/questions/29130/difference-between-neural-net-weight-decay-and-learning-rate