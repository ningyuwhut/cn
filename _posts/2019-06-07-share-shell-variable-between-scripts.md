---
  layout: post
  title: 脚本之间共享变量
  categories: 开发
  tags:
---


本文记录最近遇到的一个问题，所有资料都是参考自网上，如果存在错误，敬请指正。

最近在写一个shell流程，用于模型的天级别更新，流程的结构如下：

    scripts:
        step1.sh
        step2.sh
        main.sh
    config
        task.conf

task.conf 定义了所有的配置，这些配置需要在所有脚本中使用。大概如下:

    a=1
    b=2

main.sh 作为脚本的入口，如下:

    source ../config/task.conf 

    sh step1.sh
    sh step2.sh

此时,task.conf中的变量`a,b`只能在main.sh中使用，不能在step1.sh, step2.sh中使用。但是既然task.conf中定义了所有的配置，那么step1.sh、step2.sh自然需要用到其中的变量。刚开始，我用了一个笨办法，就是把变量作为参数进行传递， 但是这样有一个缺点，就是每个脚本都需要传递变量，这样有可能需要传递的变量会很多，那么参数列表会很长,看起来很不优雅，也很难维护。比较好的方法是什么呢？答案是使用export。即在task.conf中将变量定义全部export,如下：
    export a=1
    export b=2

这样,step1.sh 、step2.sh中都可以访问a、b了。

那么，其中的原因是什么呢

这其中涉及到shell中的几个概念:子shell/局部变量/全局变量/环境变量,下面分别说下这几个概念。

### 子shell

### 局部变量

在shell中,函数中定义的变量默认是全局变量， 只有加上local关键字时，才是全局变量。

````
function func() {
   local a=1
}

func
echo $a
````

可以去掉local关键字来验证

### 全局变量

所谓全局，是指在当前shell进程中都可以使用。但是不同的进程中的变量是不可以相互访问的。

那怎么区分是否是当前shell进程呢? 这又牵扯到shell脚本的两种执行方式。

一种是在新的进程中执行，比如作为可执行程序来运行或者通过bash解释器运行

作为可执行程序执行，需要给脚本增加可执行权限

````
chmod a+x a.sh
./a.sh
````

另外一种就是下面这种最常见的执行方式

````
sh a.sh
````

这两种方式执行shell脚本都会创建新的进程，也就是在子shell中执行的。但是有一种方式是在当前进程中执行的，不需要子shell，那就是source命令。

source命令其实是在当前shell中执行脚本，不会创建新的子shell(subshell)， source 还有一个等价的写法，就是点号.

````
source a.sh
. a.sh
````

这两种方式是等价的。


### 环境变量


上面说到，全局变量只能在当前shell中使用，但是有时候变量需要在脚本之间共享。比如文中开头提到的各种配置变量。那么怎么才能让变量在脚本之间共享呢？

export 命令就是来完成这个任务的。export 命令可以将全局变量导出成环境变量，然后所有的子shell都可以使用这个变量的。因为子shell会复制父shell中的所有环境变量， 所以子shell可以访问这个变量。但是由于子shell的变量是从父shell中复制而来的，所以改变子shell中的变量并不会影响父shell中的变量。

本文开头即使使用export命令将各种配置导出成环境变量，才能在整个流程中使用这些变量的。


话说这些都是shell的基础知识，但是之前都没怎么好好看过，导致写的脚本太丑。shell还是需要好好学习,遇到问题需要及时记录下解决方案，不然都是用完就忘了，无法积累。

参考:

1.http://c.biancheng.net/view/739.html

2.http://c.biancheng.net/view/739.html

3.https://www.jellythink.com/archives/128
