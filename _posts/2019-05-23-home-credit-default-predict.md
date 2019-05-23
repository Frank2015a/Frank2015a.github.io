---
layout:     post
title:      数据挖掘|HomeCredit信贷违约预测模型
subtitle:   LGB建模分析
date:       2018-06-05
author:     马骋
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - 数据挖掘 
    - 信贷风控
---

# 0. 问题描述

数据来源于kaggle的HomeCredit比赛，https://www.kaggle.com/c/home-credit-default-risk。

HomeCredit是一家捷克的消费信贷公司（捷信），其业务致力于普惠金融，试图覆盖传统金融机构未能覆盖服务的群体，即缺乏银行账户和信用记录的边缘群体。主要通过申请人的资料和第三方信用、交易数据进行授信。

本文通过数据建模，预测客户的违约概率。

# 1. 整体结论与建议

- 通过决策树集成算法建模，取得较好的违约预测能力，auc指标达到0.792
- 结合信贷业务特点分析了采用各个阈值下预测结果带来的收益损失，确定了使得综合收益最大的fpr为0.11，tpr=0.474


# 2. 建模

## 数据划分

建模采用经过特征工程处理的数据：https://www.kaggle.com/willkoehrsen/home-credit-simple-featuers

- simple_features_train，维度307507, 718
- simple_features_test，维度48744, 718

为了验证模型的性能，在原始的训练数据中划分出1W的数据作为验证集，用于评价性能。划分后，各数据集的数量：

| 数据集 | 样本数 |
|--|--|
| train | 297507 |
| val | 10000 |
| test | 48744 |

## 建模与调参

模型采用lightgbm模型，采用贝叶斯调参方法，经过1000轮调参，得到最优参数。

模型的参数如：

```
{'learning_rate': 0.07218374731817535,
 'reg_lambda': 0.7364934411848395,
 'verbose': 1,
 'subsample': 0.6195545022366721,
 'subsample_for_bin': 60000,
 'boosting_type': 'dart',
 'is_unbalance': True,
 'num_leaves': 47,
 'colsample_bytree': 0.6001712855022151,
 'reg_alpha': 0.5969339070590824,
 'min_child_samples': 485,
 'metric': 'auc'}
```

由于`dart`模式下无法early stop，需要手动设置好停止的轮数1000。如果超过了最佳轮数，继续训练反而会使得模型过拟合，导致性能更差。


# 3. 模型评测


## K折交叉验证

在完整的训练集上采用K-折交叉验证，

```
import lightgbm as lgb
train_set = lgb.Dataset(train, label = train_labels)
cv_results = lgb.cv(hyperparameters, train_set,
                    num_boost_round = 1000, 
                    early_stopping_rounds = 100, 
                    metrics = 'auc', 
                    nfold = 3, 
                    verbose_eval=10)
```

得到auc score=**0.78639**。

## 竞赛模型评测

在竞赛官网提交结果，得到的成绩如：

![](http://static.zybuluo.com/frank0449/rmvsbo7h91qf95k58g14e69j/image_1dbgvf6e5db61n1n1vlvcol1tor9.png)

top成绩榜单如下，可以看到与最高成绩仍有差距，但0.7928已经是一个不错的成绩。

![](http://static.zybuluo.com/frank0449/mqae8r2y1m6918ie5nz6y8zb/image_1dbgvja7vuf313l0ai8hdi7om.png)


## 留出法性能验证

为了得到更详细的模型性能，采用留出法，即采用划分之后的train做为训练数据，val作为验证数据，评测模型的性能。

```
X_train, \
X_val, \
y_train, \
y_val = train_test_split(train, train_labels, test_size = 10000, random_state = 42)

lgbm = lgb.LGBMClassifier(n_estimators = 1000, **hyperparameters)
lgbm.fit(X_train, y_train, verbose=50)

y_true = y_val
y_prob = lgbm.predict_proba(X_val)[:, 1]
fpr, tpr, thresholds = metrics.roc_curve(y_true, y_prob, pos_label=1)
auc = metrics.auc(fpr, tpr)
```

模型评价的基础指标为：

- fpr(False Positive Rate)表示假阳性率，即非违约客户被错误预测为违约的概率，
- tpr（True Positive Rate）表示当前预测的违约用户占真正违约用户的比例；

模型对样本的预测结果是数值在0到1之间的概率值，例如张三违约的概率的0.456，要给出是否违约的判断，就需要确定一条及格线--阈值。取不同的阈值，可以得到不同的fpr、tpr数值，绘制成ROC曲线如下：

![](http://static.zybuluo.com/frank0449/1sdxea285gcforl60ua37ps4/image_1dbh07sgj14bkrdjf7eo3i1bdv1g.png)

对于预测模型，ROC曲线在坐标轴上包围的面积（auc）越大，模型的识别性能越好，当前模型的指标为auc score=0.79119。 

更直观的指标为tpr@fpr，即在误识率为fpr的水平下被识别出的真实违约用户的比例。


| -   |   fpr |       tpr |
|---:|------:|----------:|
|  0 | 0.001 | 0.0160494 |
|  1 | 0.005 | 0.0580247 |
|  2 | 0.01  | 0.114815  |
|  3 | 0.05  | 0.32963   |
|  4 | 0.1   | 0.449383  |
|  5 | 0.2   | 0.619753  |

在应用中，要排除一些潜在的违约用户，必然会误伤一些实际并不会违约的用户。例如以上表格中，fpr=0.1、tpr=0.449意味着要识别出44.9%的违约用户，就会错误的拒绝10%的正常用户。

## 收益分析

显然，不论阈值确定在什么水平，都是有代价的：

- 拒绝真正的违约用户避免了损失
- 拒绝正常用户意味着收益的损失

从用户统计可以看出，违约用户占总数的8%，正常用户比例是92%。
假设贷款年利率在8%的水平，而违约用户平均会造成50%贷款金额无法收回。

如此可以确定收益分析的系数：

- 总体比例系数
    - 违约：0.08
    - 正常：0.92
- 收益损失系数：
    - 违约：-0.5
    - 正常：0.08
    
那么按一个阈值确定是否通过贷款，带来的综合收益为：

```
S = -fpr*0.92*0.08 + tpr*0.08*0.5
```

按以上假设，得到各个水平下的综合收益：

![](http://static.zybuluo.com/frank0449/d5bfy2qs6vhd6xnty6yhfe3x/image_1dbh27bl51ih5r0u9nfcjg17a31t.png)

可以看出，随着fpr水平的提高，综合收益先增加，后降低。使得综合收益最大的fpr为0.11。

|   - |   fpr |      tpr |   overall_gain |
|---:|------:|---------:|---------------:|
| 11 |  0.11 | 0.474074 |      0.010867  |
|  9 |  0.09 | 0.437037 |      0.0108575 |
| 12 |  0.12 | 0.490123 |      0.0107729 |
|  7 |  0.07 | 0.395062 |      0.0106505 |
| 13 |  0.13 | 0.504938 |      0.0106295 |

在实际的业务中，应当根据实际的正常客户收益率和违约客户损失率来确定最优的fpr水平，以实现业务的优化。

