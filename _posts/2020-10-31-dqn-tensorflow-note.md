---
  layout: post
  title: dqn-tensorflow笔记
  categories: MachineLearning
  tags:
---

打算学习下DQN，在github上找了[一份代码](https://github.com/devsisters/DQN-tensorflow)，代码质量挺高。下面是阅读笔记，期望通过对该代码的学习加深对DQN的理解。

代码结构如下：

````
├── README.md
├── config.py
├── dqn
│   ├── __init__.py
│   ├── agent.py
│   ├── base.py
│   ├── environment.py
│   ├── history.py
│   ├── ops.py
│   ├── replay_memory.py
│   ├── utils.py
└── main.py
````

程序的入口为main.py

强化学习中涉及了两个实体，一个是智能体，一个是环境，分别对应agent.py和environment.py。

智能体通过与环境交互来实现累积回报最大化，交互的动作即action，获得的奖励即reward。同时，智能体也从一个状态(state)到达了另外一个state。

根据config.py中的配置，history_length=4,即一个样本中包含四帧图像。
样本中包含如下信息:

当前时刻的四帧图像、最后一帧图像对应的action、reward、是否终止态，四帧图像对应的下一帧组成的四帧图像

图像格式:[batch_size,history_length,screen_width,screen_height]

根据取值，实际格式为[batch_size,4,84,84]

在build_dqn中为网络结构。

对于q网络，包含3层卷积层和2层全连接层。

目标网络也是包含3层卷积层和2层全连接层

q网络:
三层卷积的配置如下:

1. 第一层:32个feature_map,每个feature_map大小[8,8],步长[4,4]
2. 第二层:64个feature_map,每个feature_map大小[4,4],步长[2,2]
3. 第三层:64个feature_map,每个feature_map大小[3,3],步长[1,1]

当不使用GPU 时，cnn_format为NHWC,dqn网络过程如下：
0. 输入:[N,84,84,4]

1. 第一层CNN
stride:[1,4,4,1],kernel:[8,8,4,32] 输出: 根据 (x-kernel)/stride+1,得到输出维度:[N,20,20,32]
2. 第二层CNN
stride:[1,2,2,1],kernel:[4,4,32,64] 输出:[N,9,9,64]
3. 第三层CNN
stride:[1,1,1,1],kernel:[3,3,64,64] 输出:[N,7,7,64]
4. 两层MLP
reshape成[N,64*7*7]维度后接两层MLP
5. 决策
最终输出[N,action_size],表示每个样本上每个action的未来累积期望的分布, 选择最大的动作记录到q_action中。

target 网络结构都一样

计算出每个样本的目标q值 target_q

````
  self.target_q_idx = tf.placeholder('int32', [None, None], 'outputs_idx')
  self.target_q_with_idx = tf.gather_nd(self.target_q, self.target_q_idx)
````

这个是啥？

看代码在double dqn中用到


将预测q网络的参数复制到目标q网络
````
    with tf.variable_scope('pred_to_target'):
      self.t_w_input = {}
      self.t_w_assign_op = {}
      #将q网络的参数复制到目标q网络
      for name in self.w.keys():
        self.t_w_input[name] = tf.placeholder('float32', self.t_w[name].get_shape().as_list(), name=name)
        self.t_w_assign_op[name] = self.t_w[name].assign(self.t_w_input[name])
````

loss 
````
def clipped_error(x):
  # Huber loss
  try:
    return tf.select(tf.abs(x) < 1.0, 0.5 * tf.square(x), tf.abs(x) - 0.5)
  except:
    return tf.where(tf.abs(x) < 1.0, 0.5 * tf.square(x), tf.abs(x) - 0.5)

````

训练：

先使用同一个screen 组成四帧 放到history中

在每一个step中：

1.基于当前history（只存储了最新的四个screen），选择action
  基于epsilon greedy选择，如果随机数小于epsilon，则随机选择action，否则，使用**预测q网络**预测当前状态（四帧图像）下 q值最大的action，并采用该action。
  epsilon的值需要注意一下：
  在测试阶段 ep值是固定的，即随机的概率是一定的
  在训练阶段，如果当前step 小于learn_start,那么就全部进行随机选择
  如果 step 大于learn_start,那么此时探索的概率就线性下降。
  
2.采用该action得到reward、screen和terminal（表示该轮游戏是否终止）
  采用该action时重复了action_repeat次，不清楚是为啥。reward是重复这么多次的累积的reward。
  screen 为重复数次后达到的状态。
  
3.将screen 放到history中，将screen、action、reward、terminal放到memory中
  **表示在采取该action后，即时回报为reward，到达的状态为screen，是否终止存储在terminal中。**
4.如果当前step正好到了训练频次（step>learn_start且step % train_frequency == 0）那么从memory中采样一个batch进行学习
5.如果到了更新目标网络的频次（step%target_q_update_step==target_q_update_step-1),那么用预测q网络的参数更新target q网络。
6.如果一局游戏结束，则重新开始下一轮游戏，同时，重置该轮（episode）游戏的reward为0；否则更新该轮游戏reward。


从memory中采样batch：

先从[self.history_length, self.count - 1](闭区间）中随机生成待采样的样本的index，

index需要满足如下条件：

1.index >=current & index -history_length < current

2.\[index-history_length, index-1\](闭区间)之间不能包含终止state

第一个条件是说选择的样本不能包含current，因为memory中的样本都是顺序添加的，这样的话选出来的样本包含的帧就不是连续的了
第二个条件比较好理解，是指样本中不能包含终止状态的帧。但是下标为index的screen（即后文的poststates) 可以是终止状态的。

如果index 没有命中上面两个条件，那么 
以**index-1** 为最后一个元素，继续向前取history_length-1 个元素，共history_length个元素 作为prestates。
以index 为最后一个元素，继续向前取history_length-1个元素，共history_length个元素，作为poststates。

同时，将index加入到列表中，该列表中记录了历史上所有选中的screend的下标。

最后，将这些下标对应的action、reward、terminals，连同prestate和poststate都返回，作为一个batch。

**这些信息是说当状态是prestate时，采取动作为action，立即回报为reward，是否到达终止状态为terminals，转移到的state为poststate。即poststate的前一个state为poststate。**

采样完batch后：

如果是double dqn：

1.使用预测q网络预估poststate的未来累积回报最大的action，并将最大的action的下标作为该样本采取的目标action。

2.然后使用目标q网络预估poststate的未来累积回报的分布即target_q;

最后，使用1预估的最优action从2中的target_q中选择对应的q值,记为q_t_plus_1_with_pred_action

label如下：

````
      target_q_t = (1. - terminal) * self.discount * q_t_plus_1_with_pred_action + reward

````

如果只是一般的dqn：
则使用目标q网络预测poststates的目标q值 target_q，维度为(batch_size,action_size)
这个target_q 值是在采取动作action后观察到状态poststate的q值分布。
选择每个样本在poststate下target_q中最大的q值，表示该状态下的最优action。然后计算该状态下最优的未来累积回报。

````
    target_q_t = (1. - terminal) * self.discount * max_q_t_plus_1 + reward
````

这个值就是label。

同时，网络以prestate为输入，得到该状态下的q值分布，然后乘以action，表示在该状态下采取action 得到的未来累积回报（q值），这个就是预估值了，记做q_acted。

````
self.delta = self.target_q_t - q_acted
````
二者之差就是预估偏差。然后就是根据这个计算loss，并根据loss更新q网络。


可以看到，target网络是用来算目标值的，预测网络是用来算预估值的。


double dqn的思想:








