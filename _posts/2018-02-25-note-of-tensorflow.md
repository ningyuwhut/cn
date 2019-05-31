---
  layout: post
  title: tensorflow笔记
  categories: MachineLearning
  tags:
---

计算图
===

计算图是Tensoflow的计算模型。

TensorFlow中所有计算(Operation)都会转换为计算图上的一个节点，节点之间的边描述了计算之间的依赖关系。

TensorFlow程序一般分为两个阶段，
第一个阶段是定义计算图中的所有计算，第二个阶段为执行计算。

在TensorFlow中，系统会自动维护一个默认的计算图，通过tf.get_default_graph 可以获取当前默认的计算图。

```
import tensorflow as tf

g1 = tf.Graph()
with g1.as_default():
    v = tf.get_variable("v", [1], initializer = tf.zeros_initializer()) # 设置初始值为0

g2 = tf.Graph()
with g2.as_default():
    v = tf.get_variable("v", [1], initializer = tf.ones_initializer())  # 设置初始值为1
    
with tf.Session(graph = g1) as sess:
    tf.global_variables_initializer().run()
    with tf.variable_scope("", reuse=True):
        print(sess.run(tf.get_variable("v")))

with tf.Session(graph = g2) as sess:
    tf.global_variables_initializer().run()
    with tf.variable_scope("", reuse=True):
        print(sess.run(tf.get_variable("v")))
```

上面的代码定义了两个计算图，每个计算图中都定义了一个v张量，且初始化了不同的值。

当需要在单个文件中定义多个数据流图时，最佳实践是不使用默认数据流图，或为其立即分配句柄。不应将默认数据流图和用户创建的数据流图混合使用。

张量
===

张量是Tensorflow的数据模型。

在tensorflow中，所有的数据都通过张量的形式来表示。从功能的角度来说，张量可以理解为多维数组，0阶张量即标量，1阶张量为向量，n阶张量为n维数组。但是张量
在tensorflow中的实现并不是采用数组的形式，它只是对tensorflow中运算结果的引用。在张量中并没有真正保存数字，它保存的是如何得到这些数字的计算过程。

一个张量主要包含3个属性:name,shape,type.

由于张量是计算结果的引用，而tensorflow中计算图中的每个节点都表示一个计算，所以张量和计算图上节点所代表的计算结果是一一对应的。这样张量的命名可以通过"node:src_output"的形式来给出。
node为节点的名称，src_output表示当前张量来自节点的第几个输出。


张量使用可以分为两大类

1.对中间结果的引用
2.当计算图构造完成之后，张量可以用来获得计算结果，也就是得到真实的数字。比如，可以通过tf.Session.run(result)来得到result张量的具体数值。

会话
===

会话是Tensorflow的运算模型。

会话拥有并管理tensorflow程序运行时的所有资源。当所有计算完成之后需要关闭会话来帮助系统回收资源，否则可能资源泄露。
Tensorflow中使用会话一般有两种模式

第一种需要明确调用会话生成和关闭函数，流程如下:

```
# 创建一个会话。
sess = tf.Session()

# 使用会话得到之前计算的结果。
print(sess.run(result))

# 关闭会话使得本次运行中使用到的资源可以被释放。
sess.close()
```



为了解决程序异常退出时资源回收的问题，我们可以使用python的with 语句:

```
with tf.Session() as sess:
    print(sess.run(result))
```

tensorflow不会自动生成默认会话，需要手动指定。指定了默认会话之后，可以通过tf.Tensor.eval函数来计算张量的取值。


```
sess = tf.Session()

with sess.as_default():
     print(result.eval())
```

```
sess = tf.Session()

# 下面的两个命令有相同的功能。
print(sess.run(result))
print(result.eval(session=sess))
```

Tensorflow提供了在交互式环境下直接构建默认会话的函数tf.InteractiveSession。使用该函数会自动将生成的会话注册为默认会话。

```
sess = tf.InteractiveSession ()
print(result.eval())
sess.close()
```

tf.Session的构造方法接收3个可选参数:

target/graph/config

target 指定了所要使用的执行引擎。大多数时候取空字符即可。在分布式设置中该参数用于连接不同server的tf.train.Server实例。

graph指定了Session对象需要加载的graph对象，默认值为None，表示将使用默认数据流图。

config参数允许用户指定配置Session对象所需的选项，如限制CPU或者GPU的使用数据，为数据流图设置优化参数等。


Session.run()函数会执行result需要的所有计算来得到该张量对应的值。

该方法接收一个参数fetches，以及其他3个可选参数：feed_dict,options和run_metadata.


fetches参数接收任意的数据流图元素（计算或者Tensor对象）。如果为Tensor对象，run的输出为一个Numpy数组。如果为一个计算，输出为None。

以上面的sess.run(result))为例。
该行代码表示，TensorFlow中的Session对象会找到计算result的值所需的全部节点，顺序执行这些节点，然后将b的值输出。

我们还可以传入一个数据流图的列表。当fetches为一个列表时，run的输出是一个与输入列表对应的列表。

有时，也会把某些计算的句柄传给fetches参数。tf.initialize_all_variables()便是这样的一个例子。它会准备将要使用所有TensorFlow Variable对象。

```
#执行初始化Variable对象所需的计算，但返回值是None
sess.run(tf.initialize_all_variables())
```

feed_dict 参数需要python字典对象作为输入，字典的key是Tensor对象的句柄.

Variable
===
Tensor对象和Op对象都是不可变的，但是Variable是可变的。

使用Variable对象时必须首先进行初始化。通常是将initialize_all_variables()传给Sess.run()完成


也可以通过tf.initialize_variables()对数据流图定义的一个或多个Variable对象进行初始化。

```
init = tf.initialize_variables([var1], name='init_val1')
```



TensorBoard
===

为了使用tensorboard,我们需要在代码中加入语句:

```
writer =tf.train.SummaryWriter('./my_graph', sess.graph)
#第一个参数表示数据流图的描述的保存路径，第二个是Session对象的graph属性，这样构造出来的SummaryWriter对象会将该数据流图的描述输出到my_graph路径下。
```

SummaryWriter对象初始化完成后会立即写入这些数据。一旦执行完成这行代码，便可以启动TensorBoard。



调试
==


https://wookayin.github.io/tensorflow-talk-debugging

tfdbg


广播机制
===


timeline
==

    import tensorflow as tf
    from tensorflow.python.client import timeline

    run_options = tf.RunOptions(trace_level=tf.RunOptions.FULL_TRACE)
    run_metadata = tf.RunMetadata()
    with tf.Session() as sess:
        sess.run(fetches, feed_dict=feed, options=run_options, run_metadata=run_metadata) #指定options和run_metadata参数,fetches和feed_dict自定义

    #只打印特定的几个batch时的timeline，保存到json文件中
    if batch_num  >= 10 and batch_num < 14:
        from tensorflow.python.client import timeline
        fetched_timeline = timeline.Timeline(run_metadata.step_stats)
        chrome_trace = fetched_timeline.generate_chrome_trace_format()
        with open('timeline_02.json', 'w') as f:
            f.write(chrome_trace)

