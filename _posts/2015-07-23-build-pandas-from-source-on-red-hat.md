---
  layout: post
  title: build pandas form source on red hat
  categories: MachineLearning
  tags:
---

pandas是基于python的一个数据分析库,经常和numpy、scipy等库结合起来使用。本文记录我在red hat上的编译过程。

pandas依赖比如numpy和scipy、python-dateutil、cython等库, python-dateutil又要用setuptools进行安装，所以我们先安装setuptools和python-dateutil。

因为这里所有的安装都包括下载压缩包，解压，切换到代码目录这三个步骤，所以下面在叙述时将这三个步骤省略。

####setuptools

执行 `python setup.py install` 即完成setuptools的安装， 

####python-dateutil

执行`python setup.py install`即完成安装。

####cython

执行`python setup.py build`,然后执行 `python setup.py install`

#### pytz 

执行`python setup.py install`

###pandas
    
执行` python setup.py build`,然后执行`python setup.py install`

测试:

    python
    >>>import pandas
    >>> print pandas.__version__

参考:

1.http://bommaritollc.com/2012/06/building-python-pandas-from-development-source/ 
