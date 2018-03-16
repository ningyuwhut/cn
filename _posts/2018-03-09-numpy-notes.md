---
  layout: post
  title: numpy 笔记
  categories: numpy
  tags:
---

python 中的float是64位的，占用8字节

```
>>> import numpy as np
>>> import sys
>>> sys.getsizeof(1.0)
24
>>> a=[1.0]
>>> sys.getsizeof(a)
80
>>> b=[1.0,2.0]
>>> sys.getsizeof(b)
88
>>> c=[1.0,2.0,3.0]
>>> sys.getsizeof(c)
96
```

可以看到，数组中每增加一个元素，数组的大小增大8字节

```
>>> np_a = np.array(a)
>>> sys.getsizeof(np_a)
104
>>> np_b = np.array(b)
>>> sys.getsizeof(np_b)
112
>>> np_c = np.array(c)
>>> sys.getsizeof(np_c)
120
```

可以看出，python中的float 数组转换为numpy数组后每个元素也是占用8个字节.

即np.float64

```
>>> np_a_32 = np.array(a,dtype=np.float32)
>>> sys.getsizeof(np_a_32)
100
>>> np_b_32 = np.array(b,dtype=np.float32)
>>> sys.getsizeof(np_b_32)
104
>>> np_c_32 = np.array(c,dtype=np.float32)
>>> sys.getsizeof(np_c_32)
108
```

转换为np.float32后，可以看出每个元素的大小就变成4字节

```
>>> np_a_32_1 = np.array(a).astype(np.float32)
>>> sys.getsizeof(np_a_32_1)
100

>>> np_a = np.array(a)
>>> sys.getsizeof(np_a)
104

>>> np_c_32_1 = np.array(c).astype(np.float32)
>>> sys.getsizeof(np_c_32_1)
108
```

可以看到，使用astype效果也是一样的.
