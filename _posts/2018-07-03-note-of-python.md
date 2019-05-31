---
  layout: post
  title: python 笔记
  categories: python
  tags:
---

zip
=====
     zip(*[('a', 1), ('b', 2), ('c', 3), ('d', 4)])

解释:

    https://stackoverflow.com/questions/34658436/python-why-is-zip-used-instead-of-unzip

    https://stackoverflow.com/questions/2921847/what-does-the-star-operator-mean


xarange
====

https://stackoverflow.com/questions/94935/what-is-the-difference-between-range-and-xrange-functions-in-python-2-x


Memory_profiler
====

通过pip install安装

在函数中加入@profile 注解

    @profile
    def a()
      print "x"



运行脚本时需传入-m memory_profiler参数

    python -m memory_profiler example.py

然后程序运行完成后就会给出监控的函数每行代码对内存的影响；
