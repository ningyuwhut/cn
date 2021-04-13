---
  layout: post
  title: DeepCTR 笔记
  categories: MachineLearning
  tags:
---

inputs.py 定义了几种常用的特征

由于对python的一些用法不熟悉，所以笔记中就会介绍一些python的知识


__slots__ 显式声明一个类的实例包含那些属性，这样有两个优点：

1.可以更快的访问对象的成员；
2.节省内存

__new__ 用于创建新的对象实例

super() 函数是用于调用父类(超类)的一个方法。

__hash__ 是自定义的hash函数

@property 

是一种设置类成员属性的方式，使用该注解可以不用再定义该方法的get或者set方法

在keras中自定义Layer

使用 **build(inputs_shape)** 定义该层中的参数,**call**在第一次被调用的时候会自动调用**build**方法


chain.from_iterable

用于将多个可遍历序列连接成一个序列

Make an iterator that returns elements from the first iterable until it is exhausted, then proceeds to the next iterable, until all of the iterables are exhausted. Used for treating consecutive sequences as a single sequence. Roughly equivalent to:

````
def chain(*iterables):
    # chain('ABC', 'DEF') --> A B C D E F
    for it in iterables:
        for element in it:
            yield element
````





1. https://stackoverflow.com/questions/472000/usage-of-slots
2. https://www.programiz.com/python-programming/property
3. https://tensorflow.google.cn/guide/keras/custom_layers_and_models?hl=zh-cn
