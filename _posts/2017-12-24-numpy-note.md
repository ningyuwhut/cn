---
  layout: post
  title: numpy笔记
  categories: numpy
  tags:
---

1.numpy中的一维向量即可以看做行向量，也可以看做列向量，具体要看你怎么用了

```
>>> x=np.array([[1,2,3],[4,5,6]])
>>> y=np.array([1,2,3])
>>> z=np.array([1,2])
>>> x.shape
(2, 3)
>>> y.shape
(3,)
>>> z.shape
(2,)
>>> x.dot(y)
array([14, 32])
>>> z.dot(x)
array([ 9, 12, 15])
```

2.numpy.random.choice

从一个array中随机抽样出一个子集，可以设置抽样是否有放回，同时可以指定每个entry的概率(带权抽样)

