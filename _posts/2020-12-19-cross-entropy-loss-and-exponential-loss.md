---
  layout: post
  title: 交叉熵损失和指数损失的关系
  categories: MachineLearning
  tags:
--- 


这是对[ESL 中文版 10.5 为什么是指数损失 ](https://esl.hohoweiya.xyz/10-Boosting-and-Additive-Trees/10.5-Why-Exponential-Loss/index.html) 一节的整理。

该节证明了指数损失函数和交叉熵损失函数的期望损失具有相同的最小值。

1.
$$ 
f^{\star}(x)=\mathrm{arg};\underset{f(x)}{\min}\mathrm{E}_{Y\mid x}(e^{-Yf(x)})\\=\frac{1}{2}\log\frac{\Pr(Y=1\mid X)}{\Pr(Y=-1\mid X)} \tag{10.16} 
$$
 或者等价地， $$ \Pr(Y=1\mid x)=\frac{1}{1+e^{-2f^(x)}} $$

 从上面可以看出来，指数损失函数的期望的最小值是1/2*对数几率。

 证明如下:

$$
E(e^{-yF(x)}|x) = P(y=1|x)e^{-F(x)}+P(y=-1|x)e^{F(x)}
$$

$$
\frac{\partial E(e^{-yF(x)}|x)  }{\partial F} = -P(y=1|x)e^{-F(x)}+P(y=-1|x)e^{F(x)}
$$

令导数为0

$$
e^{2F(x)}=\frac{P(y=1|x)}{P(y=-1|x)}
$$


得到：
$$
F(x)=\frac{1}{2}log\frac{P(y=1|x)}{P(y=-1|x)}
$$

也可以进行如下推导，得到P的表达式

$$
e^{-2F(x)}=\frac{P(y=-1|x)}{P(y=1|x)}=\frac{1-P(y=1|x)}{P(y=1|x)}
$$

$$
P(y=1|x)=\frac{1}{1+e^{-2F(x)}}
$$

也就是说，指数损失函数的期望的极小值在$$P(Y=1|x)$$处取得。

下面看下交叉熵损失的期望的推导

令
$$ 
p(x)=\Pr(Y=1\mid x)=\frac{e^{f(x)}}{e^{-f(x)}+e^{f(x)}}=\frac{1}{1+e^{-2f(x)}} 
$$ 

label $$Y\in{+1,-1}$$,

所以,$$Y'=(Y+1)/2\in \{0,1\}$$

所以，交叉熵损失为

$$ 
l(Y,p(x))=Y'\mathrm{log}p(x)+(1-Y')\mathrm{log}(1-p(x)) 
$$

按照label分开来看:


Y=1时:
$$ 
l(Y=1, p(x)) =\log p(x) = -\log(1+e^{-2f(x)})\ =-\log(1+e^{-2Yf(x)}) 
$$

Y=-1时
$$
l(Y=-1,p(x)) =\log (1-p(x)) = \log\frac{e^{-2f(x)}}{1+e^{-2f(x)}}=\log\frac{1}{1+e^{2f(x)}}=-\log(1+e^{2f(x)})\ =-\log(1+e^{-2Yf(x)})
$$

所以，交叉熵损失也可以表示为:

$$ -l(Y,f(x))=\mathrm{log}(1+e^{-2Yf(x)}) $$


对交叉熵损失求导

$$
\frac{\partial E(L(Y',p(x))  }{\partial p(x)} =
E(\frac{Y'}{p(x)} -\frac{1-Y'}{1-p(x)}|x)
$$

令上式为0，有

$$
E((1-p(x))Y'-(1-Y')p(x)|x)=0
$$

$$
E(Y'-p(x)|x)=0
$$

$$
p(x)=E(Y'|x)=Pr(Y=1|x)
$$


所以，交叉熵损失函数的期望的极小值也是在$$P(Y=1|x)$$处取得。



参考:

1.https://esl.hohoweiya.xyz/10-Boosting-and-Additive-Trees/10.5-Why-Exponential-Loss/index.html