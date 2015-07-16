---
  layout: post
  title: how to use mathjax in jekyll
  categories: software
  tags: jekyll
---

首先安装kramdown

    gem install kramdown

在执行这个命令时我遇到如下错误:

>ERROR:  While executing gem ... (Gem::RemoteFetcher::FetchError)
    Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://api.rubygems.org/quick/Marshal.4.8/kramdown-1.8.0.gemspec.rz)

大概是因为rubygems.org在国内访问有问题吧，把源替换成淘宝的就可以了。

    gem sources --remove https://rubygems.org/
    gem sources -a http://ruby.taobao.org/

然后在default.html中加入如下代码:

    <script type="text/javascript"
     src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
     </script>

下面测试一下是否可以:

$$
a^2+b^2=c^2
$$


公式要用`$$`括起来才行,而且前后要有空行，不知道使用起来会不会有什么问题

参考

1.http://cyukang.com/2013/03/03/try-mathjax.html
