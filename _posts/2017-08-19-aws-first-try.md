---
  layout: post
  title: aws初体验
  categories: MachineLearning
  tags:
---


其实算是第二次用了，只是距离第一次摸索的时间比较长，怎么用都已经忘了。今天打算把使用过程记录下来，备忘

首先需要明确的一点是，aws默认会给新用户一年的免费套餐，但是套餐中的机器配置比较低，没法做深度学习，所以刚开始幻想有一年的免费机器可以用来做机器学习的想法是不实际的。

所以还是老老实实使用付费的吧。


我用的是aws的竞价请求，因为据说比较便宜。配置一个请求的教程就不贴了，网上还是挺多的图文教程的。简单提几点：

实例选择p2.xlarge,除了这个是gpu机器外，还有一个g2系列，不过据网上的信息，p2.xlarge比g2.xlarge快上两三倍.


1.ami
我选的是https://github.com/ritchieng/dlami,这个ami包含了当前几个主流的深度学习框架，不用自己再配置了。
但是这个ami有两点需要注意：

1.只在俄勒冈才有
2.ssh登录时 用户名是ec2-user，否则会有"Permission denied (publickey).”错误

2.密钥对
在创建密钥对的时候密钥对文件会下载到我们本地，不过需要注意的是这个只能下载一次，所以我在这里踩了个坑。因为我换了一台电脑，然后把之前的pem文件内容复制到当前的电脑，结果ssh的时候提示
```
Enter passphrase for key 'ningyu-dlami.pem':
```

一脸懵逼，记得第一次没要输这玩意啊，在网上也没找到解决办法。索性直接把文件拷贝过来，然后再ssh就可以了


ec2上配置jupyter


在ec2上运行gan脚本时遇到```No module named tkinter```错误，查了一下原因大概是要使用这个包就需要有一个显示器才行，所以就不指望能用matplotlib了。
不过貌似可以配置jupyter notebook，在jupyter 上写脚本，这样就可以使用matplotlib了。

下面记录一下配置过程。

1.首先是在创建竞价请求时需要在安全组中配置8888端口（入站和出站我都配了8888端口)

2.在远程实例上设置ssl证书

```
mkdir ssl
cd ssl
#创建新的ssl证书
sudo openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout "cert.key" -out "cert.pem" -batch 
```

3.在远程实例上配置jupyter

```
jupyter notebook --generate-config
```

可以通过ipython设置jupyter notebook的密码

```
from IPython.lib import *
passwd()

```
然后输入密码即可，并会生成一个输出字符串:
```
Out[5]: 'sha1:046629f3ad04:817785f7911cc9dd9e10d004ea265abee27c0df4'
```
该字符串需要在下面的配置中用到

也可以不用输入密码就可以登录，不过需要使用下面的连接地址来登录
```
Copy/paste this URL into your browser when you connect for the first time,
to login with a token:
https://localhost:8888/?token=b263338edf269c19b8a69aae2c24642f87d9289409c3d1d8
```

4.然后编辑~/.jupyter/jupyter_notebook_config.py

配置如下：

```
c = get_config()  # get the config object
c.NotebookApp.certfile = u'/home/ubuntu/ssl/cert.pem' # path to the certificate we generated
c.NotebookApp.keyfile = u'/home/ubuntu/ssl/cert.key' # path to the certificate key we generated
c.IPKernelApp.pylab = 'inline'  # in-line figure when using Matplotlib
c.NotebookApp.ip = '*'  # serve the notebooks locally
c.NotebookApp.open_browser = False  # do not open a browser window by default when using notebooks
c.NotebookApp.password = 'sha1:b592a9cf2ec6:b99edb2fd3d0727e336185a0b0eab561aa533a43'  # this is the password hash that we generated earlier.
```
password那里用的就是根据上面的密码生成的

配置需要根据自己的情况略作修改

5.然后，在远程实例上创建自己的notebook文件夹
```
mkdir notebooks
cd notebooks
```

6.在远程实例上在notebook文件夹下启动notebook

```
ipython notebook
```

7.启动本地jupyter，然后在本地浏览器中通过`远程ip:8888`访问远程实例
然后就可以新建notebook了


账单

大概0.23美元一小时，第一次用了三个小时花了0.79美元，还算比较便宜吧


```
mkdir ssl
cd ssl
#创建新的ssl证书
sudo openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout "cert.key" -out "cert.pem" -batch 


```

参考:
1.https://stackoverflow.com/questions/18705042/install-and-make-tkinter-work-on-aws-ec2-instance
2.https://www.jiqizhixin.com/articles/2017-04-06-5
3.http://www.cnblogs.com/meelo/p/5994766.html
4.http://blog.bitfusion.io/2016/11/03/quick-comparison-of-tensorflow-gpu-performance-on-aws-p2-and-g2-instances
