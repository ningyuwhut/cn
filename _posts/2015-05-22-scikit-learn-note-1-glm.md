<hr>

<p>layout: post <br>
  title: scikit-learn 笔记（一）：GLM <br>
  categories: <br>
  tags:</p>

<hr>

<p>今天开始学习scikit-learn,把遇到的问题记录在这里。</p>

<pre><code>The following are a set of methods intended for regression in which the target value is expected to be a linear combination of the input variables. 
</code></pre>

<p>第一句话就让我有点疑问，线性模型的定义不应该是参数<script type="math/tex" id="MathJax-Element-642">w</script>的线性组合吗，最简单的线性回归也是关于输入变量<script type="math/tex" id="MathJax-Element-643">x</script> 的线性组合，难不成下面都是这种最简单的线性组合。</p>

<p>线性回归学习的目标是拟合参数<script type="math/tex" id="MathJax-Element-644">w</script>的线性模型，从而最小化目标变量的观测值和预测值之间的残差平方和（这个定义是不是有点狭隘？）。</p>

<h1 id="ordinary-least-squares">Ordinary Least Squares</h1>

<p>首先是Ordinary Least Squares。该方法就是最小二乘。不过该方法有一个局限性：输入变量之间必须是独立的。如果输入变量之间具有线性关系，那么<script type="math/tex" id="MathJax-Element-476">design matrix X</script>会近似于一个奇异（sigular）矩阵，从而使结果对观测值中的随机误差十分敏感，从而具有较大的方差（ <code>the least-squares estimate becomes highly sensitive to random errors in the observed response, producing a large variance</code>）(不是很理解） <br>
对于design matrix的定义，<a href="http://stats.stackexchange.com/questions/66516/meaning-of-design-in-design-matrix">这篇文章</a>解释的比较清楚。 <br>
输入变量之间的线性关系称为多重共线性（collinearity）。</p>

<h2 id="ridge-regression">Ridge Regression</h2>

<p>ridge regression 是在OLS的基础上加了一个L2 norm正则项。</p>



<p><script type="math/tex; mode=display" id="MathJax-Element-526">
min_{w}||Xw-y||_{2}^{2}+\alpha||w||_{2}^{2}
</script></p>

<pre><code>$\alpha$ controls the amount of shrinkage: the larger the value of \alpha, the greater the amount of shrinkage and thus the coefficients become more robust to collinearity.
</code></pre>



<h1 id="lasso">LASSO</h1>

<p>lasso算法产生稀疏解，减少结果所依赖的变量，所以Lasso及其变体是压缩感知的基础。 <br>
Lasso算法其实就是使用L1先验作为正则项。</p>



<p><script type="math/tex; mode=display" id="MathJax-Element-554">
min_{w}\frac{1}{2n_{samples}}||Xw-y||_{2}^{2}+\alpha||w||_{1}
</script></p>

<p>算法使用坐标下降和least angle regression进行求解。</p>

<pre><code>As the Lasso regression yields sparse models, it can thus be used to perform feature selection
</code></pre>

<p>Lasso可以用于特征选择。</p>

<blockquote>
  <p>Written with <a href="https://stackedit.io/">StackEdit</a>.</p>
</blockquote>