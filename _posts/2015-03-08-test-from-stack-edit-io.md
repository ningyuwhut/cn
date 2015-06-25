---
  layout: post
  title: Test from stackedit.io
  categories: MachineLearning
  tags:
---
  
这是从[StackEdit](https://stackedit.io/)发布的一篇测试文章。

在StackEdit上可以直接发布文章到github上搭建的jekyll博客，只需要发布的时候像下面这样填写：

>Repository: cn

>Branch: gh-pages

>File path: _post/2015-03-08-test-from-stack-edit-io.md

>Format: markdown


但是发布过程还是遇到一些问题，比如发布到github后发现格式有问题，但是在stackedit上显示是没有问题的，这大概是对markdown的解析不同造成的，也可能与我对markdown不熟有关。所以对于格式问题还是少不了在本地测试一下。

另外，就是jekyll的使用问题了，下面简单记录一下。

在编辑完文件后，可以使用如下命令在本地地址**localhost:4000**上测试一下:

    jekyll serve --config=_local_config.yml

我的_local_config.yml如下：

	 markdown: rdiscount
	 permalink: /:year/:month/:title/
	 url: http://localhost:4000
	 author: 宁雨
	 pygments: true

与默认的_config.yml的区别就在于url是本地地址而已。




