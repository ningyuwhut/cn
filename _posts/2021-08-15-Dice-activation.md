---
  layout: post
  title: Dice 激活函数
  categories: MachineLearning
  tags:
--- 

由于relu激活函数在输出值小于0时梯度为零，此时参数就不会得到更新，导致神经元dead问题，为了解决该问题，出现了Leaky ReLU激活函数。改动点是将小于0的部分给予一个很小的斜率，保证梯度不为0。但是这个斜率是固定的，属于超参。PReLu 相比Leaky ReLu的改动点是自适应的学习这个斜率。 三种激活函数的图形如下：

![prelu](https://user-images.githubusercontent.com/1762074/129469649-2163ce48-1a29-4aed-bd2d-c80e19cc2fba.png)

prelu 公式如下:

![prelu](https://user-images.githubusercontent.com/1762074/129469663-7330d8ff-ec6c-4b28-8e43-e2b85c8f2d99.png)

以上三种激活函数都在原点作为分界点。这一点其实也是可以进行优化的，这个分界点其实也应该是根据数据自适应调整的。在DIN模型中，Dice激活函数就是为了解决这个问题提出来的。

Dice的表达式如下：

![image](https://user-images.githubusercontent.com/1762074/129471966-5f435e04-9c30-44fc-ad1d-fef80f40c70d.png)


$$f(s)$$就是Dice激活函数了。和PReLu相比，多了一个$$p(s)$$，这是一个概率值，对于大于0和小于0两种情况下的输出值进行加权。而这个概率值就是通过batch normalization学习到的。
