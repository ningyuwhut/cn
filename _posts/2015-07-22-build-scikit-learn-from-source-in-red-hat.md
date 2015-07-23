---
  layout: post
  title: build scikit-learn form source on red hat
  categories: MachineLearning
  tags:
---

由于公司虚拟机无法上网，所以安装scikit-learn只能从源码开始编译。下面记录了我的安装过程。

由于scikit-learn依赖numpy和scipy，同时，scipy依赖lapack和blas两个线性代数函数库，我们又要安装lapack和atlas两个库。另外，还要安装单元测试框架nose。
所以，我们先要安装nose、lapack、atlas、numpy和scipy这五个库。
scikit-learn需要python至少是python2.6，所以在安装这些依赖库之前我先安装python2.7。

###安装python2.7。

1.下载，解压文件

2.切换到源代码目录

3.执行如下命令

    ./configure --prefix=/usr/local
    make
    make install

安装完成后，python被安装在PREFIX/bin/python，我这里是 /usr/local/bin/python2.7。然后设置一下PATH环境变量，把刚才安装的python路径放到最前面，取代系统默认的python。

    PATH=/usr/local/bin/:$PATH

###安装nose

1.下载，解压

2.切换到解压后的目录

    tar xzvf 
    cd 
    python setup.py install

3. 测试
    
    python
    >>>import nose
    >>>nose
   
###安装lapack

1.下载,解压

2.切换到解压后的目录

3.执行如下指令

    cp INSTALL/make.inc.gfortran make.inc

4.修改make.inc

    在OPTS和NOOPT后面加上选项-fPIC

>    OPTS     = -O2 -frecursive -fPIC
     NOOPT    = -O0 -frecursive -fPIC

-frecursive选项可能会导致错误，所以可以去掉。

5.

    cd SRC
    make

###安装atlas

1.下载，解压

2.切换到解压后生成的目录

3.创建一个atlas_rhel目录（目录名自定),并执行configure

    mkdir atlas_rhel
    ../configure -Fa alg -fPIC --with-netlib-lapack-tarfile=../../lapack-3.5.0.tgz 

(我这里使用的相对路径，你需要根据自己的情况进行修改）

4. 
    make
（这里可能会花很长时间，几个小时也说不定）

5. 完成后，执行如下命令

    export LD_LIBRARY_PATH=/search/songwei/software/ATLAS/atlas_rhel/lib:$LD_LIBRARY_PATH

###安装numpy

1.下载，解压

2.切换到生成的目录

3.修改site.cfg

    cp site.cfg.example site.cfg

修改site.cfg

>   [DEFAULT]
    library_dirs = /search/songwei/software/ATLAS/atlas_rhel/lib
    include_dirs = /search/songwei/software/ATLAS/atlas_rhel/lib 
    [atlas]
    atlas_libs = lapack, f77blas, cblas, atlas

    #这里的路径应该是包含生成的.a文件的路径

4.安装

    python setup.py build
    python setup.py install

5.测试是否成功：

    python

    >>>import numpy
    >>>numpy.test()

###安装scipy

1.下载，解压

2.切换到生成的目录。

3.把numpy目录下的site.cfg复制过来(貌似不执行也可以）

4.安装
    
    python setup.py build
    python setup.py install

5.测试是否成功:

    python
    >>>import scipy
    >>>scipy.test()

###安装scikit-learn
1.下载，解压
2.切换到生成的目录。
3.安装
    
    python setup.py build
    python setup.py install

参考:

1.http://pages.physics.cornell.edu/~myers/teaching/ComputationalMethods/python/WorldPyLinux_old.html
2.http://blog.mimvp.com/2014/04/linux-install-numpy-and-scipy/
3.http://idolinux.blogspot.com/2011/02/atlas-numpy-scipy-build-on-rhel-6.html
4.http://www.scipy.org/scipylib/building/linux.html#building-atlas-by-hand
5.http://www.douban.com/note/203828349/
