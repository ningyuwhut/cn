---
  layout: post
  title: xgboost 调参
  categories: MachineLearning
  tags:
--- 


在[xgboost的官方文档](https://xgboost.readthedocs.io/en/latest/parameter.html)中，参数分成了三类,下面对三类参数做简要的介绍，由于参数太多，所以只介绍了比较常见的几个，具体可以参考官方文档。

1.通用参数:

    booster [default = gbtree] 还有gblinear。

    nthread

    num_pbuffer

    num_feature

2.学习任务参数

  这里参数比较少，放到前面来。

  objective

    reg : linear，线性回归

    reg : logistic，逻辑回归

    binary : logistic，使用LR二分类，输出概率

    binary : logitraw，使用LR二分类，输出Logistic转换之前的分类得分 。

  eval_metric

    验证集上的评价指标，比如rmse、mae、error、logloss、auc等

3.booster参数

  这里的参数是调参的重点

  比较重要的有

    eta：学习率
    gamma：特征分割时损失函数下降的阈值
    max_depth:树的最大深度
    min_child_weight:叶子节点的最小权重和
    subsample:行采样
    colsample_bytree, colsample_bylevel, colsample_bynode:列采样
    colsample_bytree:每棵树采样的特征比重
    colsample_bylevel:每棵树的每一层分裂时采样的特征比重
    colsample_bynode:树的每个节点分裂时采样的特征比重
    lambda:L2正则系数
    alpha:L1正则系数
    tree_method: exact 表示精确查找,aprrox表示近似，hist表示使用直方图，Lightgbm同样使用该方法
    grow_policy:按层生长或者按叶子生长

调参

下面的内容主要翻译自[complete-guide-parameter-tuning-xgboost-with-codes-python](https://www.analyticsvidhya.com/blog/2016/03/complete-guide-parameter-tuning-xgboost-with-codes-python/
)

    1.刚开始选择一个相对较高的学习率，比如通常设置为0.1就可以，可以根据问题的不同在0.05到3之间进行调整。然后确定在当前学习率下的最优弱分类器数量。
    xgboost有一个cv函数在每次迭代时进行交叉验证，最终返回最优的决策树数量
    2.确定学习率和决策树数量后，调整boosting参数（包括max_depth, min_child_weight, gamma, subsample, colsample_bytree),
    3.调整正则化参数（lambda、alpha）
    4.降低学习率并确定最佳参数

下面是摘自文中的调参示例:

    1.根据默认参数确定最优迭代次数
    2.确定最大深度和叶子节点最小权重
    3.确定gamma（分裂时损失函数下降的最小值）
    4.确定采样相关参数
    5.确定正则化参数
    6.减小学习率，提高最大迭代次数

参考:

https://xgboost.readthedocs.io/en/latest/parameter.html

https://www.analyticsvidhya.com/blog/2016/03/complete-guide-parameter-tuning-xgboost-with-codes-python/

https://ask.hellobi.com/blog/ai_shequ/33676

https://snaildove.github.io/2018/12/18/get_started_feature-engineering/

https://zhuanlan.zhihu.com/p/196213786