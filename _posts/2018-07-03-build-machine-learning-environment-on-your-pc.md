---
  layout: post
  title: 搭建开发环境
  categories: MachineLearning
  tags:
---

使用anaconda 进行环境搭建

下载安装anaconda

如果遇到:

    ERROR: The install method you used for conda--probably either `pip install conda` or `easy_install conda`--is not compatible with using conda as an application.
    If your intention is to install conda as a standalone application, currently
    supported install methods include the Anaconda installer and the miniconda
    installer.  You can download the miniconda installer from
    https://conda.io/miniconda.html.

那么可以将conda的可执行文件所在目录放到$PATH中,即在~/.zshrc(也可以是~/.bashrc)中加入如下语句
    
    export PATH="/anaconda2/bin:$PATH"
    
这个目录根据你的安装情况来定。

然后 source ~/.zshrc 即可。

因为conda的官方源速度很慢， 所以一定要先配置清华源:

    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/

安装anaconda后scipy/numpy/pandas等都已经安装好了，可以使用如下脚本进行校验:

    # scipy
    import scipy
    print('scipy: %s' % scipy.__version__)
    # numpy
    import numpy
    print('numpy: %s' % numpy.__version__)
    # matplotlib
    import matplotlib
    print('matplotlib: %s' % matplotlib.__version__)
    # pandas
    import pandas
    print('pandas: %s' % pandas.__version__)
    # statsmodels
    import statsmodels
    print('statsmodels: %s' % statsmodels.__version__)
    # scikit-learn
    import sklearn
    print('sklearn: %s' % sklearn.__version__)


输出如下:

    scipy: 1.1.0
    numpy: 1.14.3
    matplotlib: 2.2.2
    pandas: 0.23.0
    statsmodels: 0.9.0
    sklearn: 0.19.1

然后再安装深度学习的包
    
    conda install theano
    conda install tensorflow
    conda install keras

使用如下脚本检测:

    # theano
    import theano
    print('theano: %s' % theano.__version__)
    # tensorflow
    import tensorflow
    print('tensorflow: %s' % tensorflow.__version__)
    # keras
    import keras
    print('keras: %s' % keras.__version__)

输出如下:

    theano: 0.9.0.dev-c697eeab84e5b8a74908da654b66ec9eca4f1291
    tensorflow: 1.1.0
    Using TensorFlow backend.
    keras: 2.2.0



参考:

https://machinelearningmastery.com/setup-python-environment-machine-learning-deep-learning-anaconda/
