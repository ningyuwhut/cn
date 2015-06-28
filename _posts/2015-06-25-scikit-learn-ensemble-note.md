---
  layout: post
  title: scikit-learn ensemble note
  categories: MachineLearning
  tags:
---

> As they provide a way to reduce overfitting, bagging methods work best with strong and complex models (e.g., fully developed decision trees), in contrast with boosting methods which usually work best with weak models (e.g., shallow decision trees).

bagging方法可以减少overfitting，所以bagging方法在复杂模型下性能最好；相反，boosting方法在弱模型下（shallow decision tree）下性能最好。

Bagging方法的主要不同在于各自抽取子集的方式不同。

> When random subsets of the dataset are drawn as random subsets of the samples, then this algorithm is known as Pasting [B1999].

>When samples are drawn with replacement, then the method is known as Bagging [B1996].

>When random subsets of the dataset are drawn as random subsets of the features, then the method is known as Random Subspaces [H1998].

> Finally, when base estimators are built on subsets of both samples and features, then the method is known as Random Patches [LG2012].

随机抽取样本称为Pasting

有放回地抽取样本称为bagging

随机抽取样本特征称为Random Subspaces

样本和特征都进行随机抽取称为Random Patches

sklearn对Bagging 方法的支持如下：

    >>> from sklearn.ensemble import BaggingClassifier
    >>> from sklearn.neighbors import KNeighborsClassifier
    >>> bagging = BaggingClassifier(KNeighborsClassifier(),max_samples=0.5, max_features=0.5)


Forests of randomized trees
--

## Random Forests

random forests在实现时样本是有放回抽样的。

>In addition, when splitting a node during the construction of the tree, the split that is chosen is no longer the best split among all features. Instead, the split that is picked is the best split among a random subset of the features. As a result of this randomness, the bias of the forest usually slightly increases (with respect to the bias of a single non-random tree) but, due to averaging, its variance also decreases, usually more than compensating for the increase in bias, hence yielding an overall better model.


##Extremely Randomized Trees

>As in random forests, a random subset of candidate features is used, but** instead of looking for the most discriminative thresholds, thresholds are drawn at random for each candidate feature and the best of these randomly-generated thresholds is picked as the splitting rule**. This usually allows to reduce the variance of the model a bit more, at the expense of a slightly greater increase in bias:

和随机森林一样都使用随机抽取的特征，但是不同的是，随机森林根据某一标准(比如熵)选择最有区分性(discriminative)的阈值，而在Extremely Randomized Trees中，每个特征的阈值是随机选取的，然后从这些随机选取的阈值中选择最好的特征进行分割。这样，模型的variance会更低，同时bias会升高。

##参数

需要调整的参数有两个： n_estimators and max_features.

n_estimators是树的数量。越多越好，但是同时训练时间越大。另外，算法的性能在一个树的数量的临界点上性能会停止变的更好。The latter is the size of the random subsets of features to consider when splitting a node. 越小方差的减少越大，同时偏置的增大越大（The lower the greater the reduction of variance, but also the greater the increase in bias. ）

>Empirical good default values are max_features=n_features for regression problems, and max_features=sqrt(n_features) for classification tasks (where n_features is the number of features in the data). Good results are often achieved when setting max_depth=None in combination with min_samples_split=1 (i.e., when fully developing the trees).Bear in mind though that these values are usually not optimal, and might result in models that consume a lot of ram. The best parameter values should always be cross-validated. In addition, note that in random forests, bootstrap samples are used by default (bootstrap=True) while the default strategy for extra-trees is to use the whole dataset (bootstrap=False).

根据经验，分类问题中max_features设为sqrt(n_features),回归问题设置为n_features。同时，决策树为充分生长的树(深度不限，每个节点的最少样本数不限),但是，永远记住
**经验值是没有最优的，最优的参数应该总是从cross-validated得到。**


##Feature importance evaluation

>The relative rank (i.e. depth) of a feature used as a decision node in a tree can be used to assess the relative importance of that feature with respect to the predictability of the target variable. Features used at the top of the tree are used contribute to the final prediction decision of a larger fraction of the input samples. The expected fraction of the samples they contribute to can thus be used as an estimate of the relative importance of the features.
By averaging those expected activity rates over several randomized trees one can reduce the variance of such an estimate and use it for feature selection.

这个没有看明白

AdaBoost
---

>By default, weak learners are decision stumps. Different weak learners can be specified through the base_estimator parameter. The main parameters to tune to obtain good results are n_estimators and the complexity of the base estimators (e.g., its depth max_depth or minimum required number of samples at a leaf min_samples_leaf in case of decision trees).
 需要调整的参数主要是n_estimators和base estimator的复杂度（在决策树中，树的最大深度和叶子节点中的最少样本数）

GBDT
---

>The advantages of GBRT are:

Natural handling of data of mixed type (= heterogeneous features)

Predictive power

Robustness to outliers in output space (via robust loss functions)

>The disadvantages of GBRT are:

Scalability, due to the sequential nature of boosting it can hardly be parallelized.


>The number of weak learners (i.e. regression trees) is controlled by the parameter n_estimators; The size of each tree can be controlled either by setting the tree depth via max_depth or by setting the number of leaf nodes via max_leaf_nodes. The learning_rate is a hyper-parameter in the range (0.0, 1.0] that controls overfitting via shrinkage .


>Classification with more than 2 classes requires the induction of n_classes regression trees at each at each iteration, thus, the total number of induced trees equals n_classes * n_estimators. For datasets with a large number of classes we strongly recommend to use RandomForestClassifier as an alternative to GradientBoostingClassifier .

多分类时每次迭代需要n_classes个树，所以总共需要n_classes*n_estimators个树，所以对于有大量类的数据集，推荐使用RandomForestClassifier.

##Controlling the tree size

>The size of the regression tree base learners defines the level of variable interactions that can be captured by the gradient boosting model. In general, a tree of depth h can capture interactions of order h . There are two ways in which the size of the individual regression trees can be controlled.

决策树的大小决定了gbdt模型可以捕捉到的变量之间关系。一般深度为h的树可以捕捉到h阶的特征关联。有两种方法可以控制树的大小:树的最大深度和树的最多叶子节点。

>We found that max_leaf_nodes=k gives comparable results to max_depth=k-1 but is significantly faster to train at the expense of a slightly higher training error. The parameter max_leaf_nodes corresponds to the variable J in the chapter on gradient boosting in [F2001] and is related to the parameter interaction.depth in R’s gbm package where max_leaf_nodes == interaction.depth + 1 .
最大叶子节点为k和最大深度为k-1的模型性能相当，除了训练误差略高。


##Mathematical formulation

>Gradient Tree Boosting uses decision trees of fixed size as weak learners. Decision trees have a number of abilities that make them valuable for boosting, namely the ability to handle data of mixed type and the ability to model complex functions.
决策树可以处理具有不同数据类型的数据，同时可以对复杂函数建模。

Loss Functions
---

###Regression

1.Least squares ('ls') 	初值为均值

2.Least absolute deviation ('lad') 初值为median

3.Huber ('huber')

4.Quantile ('quantile')

###Classification

Binomial deviance ('deviance')

Multinomial deviance 

Exponential loss 即Adaboost。 can only be used for binary classification

##Regularization

###Shrinkage

>The parameter learning_rate strongly interacts with the parameter n_estimators, the number of weak learners to fit. Smaller values of learning_rate require larger numbers of weak learners to maintain a constant training error. Empirical evidence suggests that small values of learning_rate favor better test error. [HTF2009] recommend to set the learning rate to a small constant (e.g. learning_rate <= 0.1) and choose n_estimators by early stopping. 

###Subsampling

>Stochastic gradient boosting allows to compute out-of-bag estimates of the test deviance by computing the improvement in deviance on the examples that are not included in the bootstrap sample (i.e. the out-of-bag examples). The improvements are stored in the attribute oob_improvement_. oob_improvement_[i] holds the improvement in terms of the loss on the OOB samples if you add the i-th stage to the current predictions. Out-of-bag estimates can be used for model selection, for example to determine the optimal number of iterations. OOB estimates are usually very pessimistic thus we recommend to use cross-validation instead and only use OOB if cross-validation is too time consuming.

OOB estimate没看明白

##Interpretation

###Feature importance

>Individual decision trees intrinsically perform feature selection by selecting appropriate split points. This information can be used to measure the importance of each feature; the basic idea is: the more often a feature is used in the split points of a tree the more important that feature is. This notion of importance can be extended to decision tree ensembles by simply averaging the feature importance of each tree 

一个特征被用来分裂的次数越多这个特征越重要。在ensemble中可以通过对每颗树中特征的重要性进行平均来平均特征的重要性。

###Partial dependence

