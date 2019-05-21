---
layout:     post
title:      数据挖掘|HomeCredit信贷数据EDA挖掘
subtitle:   客户群特征与违约风险分析
date:       2018-06-05
author:     马骋
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - 数据挖掘 
    - 信贷风险
---

---

# 0. 数据背景

数据来源于kaggle的HomeCredit比赛，https://www.kaggle.com/c/home-credit-default-risk。

HomeCredit是一家捷克的消费信贷公司（捷信），其业务致力于普惠金融，试图覆盖传统金融机构未能覆盖服务的群体，即缺乏银行账户和信用记录的边缘群体。主要通过申请人的资料和第三方信用、交易数据进行授信。

# 1. 关键结论与建议

1. 客户群总体特征：**总体的违约率为8.07%**，用户贷款额的中位数在51.3W，年度还款额中位数在2.5W，还款年限中位数20年。
用户中女性为主占比65.8%，职业以working为主，总体教育水平偏低。
2. 贷款额度在40w-80w的客户群，违约比例最高，当贷款额超过80W，金额越大，违约比例反而越低；年度还款额的影响类似，随着额度增大，违约比例先升后降。**贷款额度最高的群体，违约率相对最低**。
3. 教育水平与收入水平有较显著的相关性，**教育水平越高的群体，收入越高、贷款额越大，违约比例越低**。教育水平高的群体是优质客户群，但在所有客户中占比最小。
4. **随着收入水平的增加，违约比例呈明显下降趋势**，当收入水平增加到一定程度（40w）后，违约比例不再下降。
5. **不同职业类型的群体，违约比例有显著的差别**，'Working' > 'Commercial associate' > 'State servant' > 'Pensioner'。商人的收入远超其他职业，其次是商业活动者，收入最低的职业为失业者和产假期的女性（违约比例最高）。
6. **随着用户年龄增大，违约比例呈现出显著的递减规律**，但老年人的收入并非最高的。因此造成不同年龄段违约率的差别的因素不一定是收入水平，更可能是风险和责任的意识。与年龄相似的工龄、注册时间等指标对于违约比例也有一定的区分度。
7. 还款-收入比值体现了一个人借款的杠杆率，当杠杆率在1.0以内，违约率没有显著差别；当**杠杆率超过1.0之后，违约率显著提高**。
8. 其他影响违约比例的因素有：婚姻家庭状况、住房类型（是否独立居住）、性别（女性违约比例低于男性）。

业务建议：

1. 从盈利的角度，尽可能扩大优质客户群的数量。教育水平高的群体贷款额度大、违约率低，但在用户中占比低，可以拓展这类客户的渠道。
2. 年龄较小的群体违约比例最高，但主要的影响因素并非收入，更可能由于信用风险意识的不足，在发放贷款时对年轻人加强宣传教育可能有助于降低违约率。年轻的客户群对于业务的长远发展是有利的。
3. 收入类型中Working占51.6%是主要的群体，这个群体的违约比例9.58%，显著高于平均值。如何控制这一群体的违约率是业务提升的重点，可能的措施有降低授信额度、缩短分期期限，对信用较低的群体进行更多人工审核调查。

# 2. 总体数据

主要的数据是application表格，分train和test。

其中训练数据的维度为(307511, 122)，122列中包含：

- `TARGET`，贷款是否违约
- `SK_ID_CURR`，用户唯一ID
- 其余121 特征列

数据概览：

![image_1db46g791107upo21sio154j14i39.png-27.3kB][1]

## 违约语义分析

TARGET：

- 1 - client with payment difficulties: he/she had late payment more than X days on at least one of the first Y installments of the loan in our sample, 
- 0 - all other cases

根据数据描述，TARGET==1，表示贷款人在已经发生的Y分期中，只要有一次逾期超过X天。注意，这是贷中状态，贷款尚未偿还完毕。因此当前的“逾期”并不等同于“违约”：一方面发生Late的用户最终可能完全偿还贷款；另一方面，当前尚未Late的用户，在未来的分期中发生逾期和违约，即潜在的违约客户。

为了方便分析，本文中将TARGET==1状况等同于**不良贷款**。

## 数据类型

各列的数据类型分布为：

```
float64    65
int64      41
object     16
```

数值类型的特征（float64，int64）可以直接应用于建模，而object（字符）类型的特征，需要编码处理之后，才能入模。重点关注object类型数据，统计得到：

|      -                      |   count |   unique | top                           |   freq |   freq_ratio |
|:---------------------------|--------:|---------:|:------------------------------|-------:|-------------:|
| ORGANIZATION_TYPE          |  307511 |       58 | Business Entity Type 3        |  67992 |     0.221104 |
| OCCUPATION_TYPE            |  211120 |       18 | Laborers                      |  55186 |     0.261396 |
| CODE_GENDER                |  307511 |        3 | F                             | 202448 |     0.658344 |
| NAME_EDUCATION_TYPE        |  307511 |        5 | Secondary / secondary special | 218391 |     0.710189 |
| HOUSETYPE_MODE             |  153214 |        3 | block of flats                | 150503 |     0.982306 |
| EMERGENCYSTATE_MODE        |  161756 |        2 | No                            | 159428 |     0.985608 |


可以看出obj特征列中：

- 除了`ORGANIZATION_TYPE`和`OCCUPATION_TYPE`，其余特征的unique值数量在2-8，方便进行哑编码
- 大多数特征列中都存在一个频率大于0.5的取值
- `HOUSETYPE_MODE`和`EMERGENCYSTATE_MODE`中频率最高的项超过0.98，接近1，即所有记录的取值都基本一致，能贡献的区分性信息量估计不大

可以直观得到的信息：

- 性别中，女性占65.8%
- 贷款类型中，Cash loans现金贷款占90.4%（区别于循环贷款）
- 收入来源中，Working占51.6%
- 教育水平中，Secondary / secondary special占71%，总体的教育水平偏低

## 缺失项分析

结构化的表格数据中，往往存在大量的缺失项。统计得到，缺失率超过0.5的特征有41列。典型如：

|   -                       |   null_ratio |
|:-------------------------|-------------:|
| COMMONAREA_MEDI          |     0.698723 |
| COMMONAREA_AVG           |     0.698723 |
| COMMONAREA_MODE          |     0.698723 |
| NONLIVINGAPARTMENTS_MODE |     0.69433  |
| NONLIVINGAPARTMENTS_AVG  |     0.69433  |

缺失比例过高（例如超过0.5），则特征列包含的信息非常有限。在数据预处理过程中，可能排除缺失值比例过高的特征。

## 缺失的特征

以上数据维度虽然多，但依然缺失了一些值得关注的信息：

- 利率
主表中缺乏利率的信息
- 贷款申请/发放日期
一方面，不同年份的经济情况会有区别
另一方面，可以统计各年份的贷款量，分析业务增速。
- 贷款用途描述
贷款用途信息有助于分析不同用途对应的风险水平。
借贷做生意的风险可能高于房贷的方向。
- 违约用户已经偿还的资金

# 2.客户群体特征分析

## 总体违约分布

对于违约的标签`TARGET`，

- 1表示违约（发生过逾期），占比91.92%
- 0表示正常（尚未发生逾期），占比8.07%

![image_1db98o3ef1s4j1q5tduj1rlhiun9.png-7.9kB][2]

注意到，这里的“违约”并不意味着全部金额坏账，实际的总体坏账率可能低于8.07%的水平。

## 性别分布

性别有3个分类，XNA可能表示第三性别（或者不确定），占比极少（0.0013%）数量极少，在分析中可以忽略不计。

![image_1db991vfn1odmdbld2v12c643am.png-10.8kB][3]

值得注意的是，女性用户数量几乎是男性的2倍，可能的原因是女性受接受金融服务的渠道比男性少，因此更需要HomeCredit的服务。

从子群体的违约比例来看，**男性用户的违约比例显著高于女性**，男性0.101、女性0.070。

![image_1db99d48eodo11pmfk46fbthd13.png-10.8kB][4]

## 婚姻家庭状况

婚姻状态中，Married占最大比例，0.638
`Civil marriage`表示未经过宗教仪式的结婚。

![image_1db99pahnggj1q8h8v8p1t1igm1j.png-13kB][5]

各群体的违约比例：

- Married群体违约比例7.55%，略低于总体的违约比例8.07%
- 违约风险较高的群体是Civil marriage 和Single / not married  
- Widow孀妇群体的违约比例5.82%，低于平均水平
或许体现了“独立女性”在财务上具有更好的掌控力。

![image_1db99srdt19g11lsruli1phqc8r20.png-16.6kB][6]

## 住房情况

69.3%的客户拥有自己的房产，是否拥有房产对违约比例几乎没有影响。

![image_1db9a7l3eeoetbc357egglir2d.png-10.7kB][7]

关于住房的另一项统计信息为**住房类型**：独立居住、租赁公寓、与父母同居等等。

![image_1db9acm9c1aq3l79o9rhp3111n2q.png-15.9kB][8]

住房类型对于违约有一定的区分度，Rented apartment 和 With parents类型的客户，违约概率显著高于平均水平。

![image_1db9afg801remb97khg1grps8837.png-18.2kB][9]

## 教育水平

客户的总体受教育水平较低，Secondary / secondary special占71.0%，其次是Higher education占比24.3%。

![image_1db9aotv1s50a3rr24a2rln33k.png-14.8kB][10]

不同教育水平群体的违约比例：

- **教育水平与违约率呈显著相关性，教育程度越低，违约的比例越高**
- 最高学历的群体`Academic degree`违约率仅为1.8%

![image_1db9as1lrl5v1smh1mcc17lmov4e.png-18.3kB][11]

从各学历群体的贷款额度中位数来看，教育水平越高，收入水平越高；总体上教育水平越高，贷款额越大，Secondary群体的贷款额度略高于Incomplete higher。

![image_1dbbpk7etm69101l1b9i109e12un6i.png-39.1kB][12]

总体来看，教育水平高的群体贷款额度高、违约风险低，是极佳的用户群体。但HomeCredit致力于普惠金融，更需要解决的问题是，如何控制中低教育水平群体的违约风险，服务更大的中低端客户群。

## 收入水平

原始收入数据中存在剧烈偏离分布的极大值，如下表中收入中位数为147150，而最大值高达1.17e+08，两者相差将近3个数量级。为了便于分析数据的分布，需要先剔除数量极少的极大值。

|  -     |   AMT_INCOME_TOTAL |
|:-------|-------------------:|
| mean   |      168798        |
| min    |       25650        |
| 50%    |      147150        |
| 99%    |      472500        |
| 99.99% |           2.25e+06 |
| max    |           1.17e+08 |

考虑到99.9%的人收入都不超过1e6，故过滤收入在1e6之内的数据进行分析，收入的分布如下图：

![image_1db9c3m8m15fp1inl18nm10ca14h15o.png-31.6kB][13]

将收入水平划分档次，分析各收入水平群体的违约比例。在各收入水平的违约比例图表中，可以看出随着**收入水平的增加，违约概率呈明显下降趋势**。当收入水平增加到一定程度（4e5）后，违约比例不再下降。

![image_1db9c8c9k36p1nu61589us1uc865.png-21.6kB][14]

## 教育、收入、违约相关性分析

教育水平的特征为字符类型，需要编码转换成数值型，才能进行分析。映射关系如：

|     教育水平     |   编码 |
|:-----------|-------------:|
|Academic degree| 0 |
|Higher education| 1 |
| Incomplete higher | 2 |
| Lower secondary | 3 |
| Secondary / secondary special | 4 |

教育与收入、违约的相关性如下：

- 在valid数据中，教育与收入的相关性达到0.243，相关性很高，符合预期
- 教育水平与违约的相关性达到0.054

|     数据   |   EDU-INCOME |   EDU-TARGET |
|:-----------|-------------:|-------------:|
| 原始数据   |   -0.0962191 |    0.0546986 |
| 去除异常收入的 |   -0.243327  |    0.0545697 |

## 职业类型

主要的职业类型有：working、commercial association 、pensioner。

![image_1dbapcog814c01d6k6cd1eil16vq9.png-15.5kB][15]

各种职业群体的收入中位数分布：

- 商人的收入远超其他职业，其次是商业活动者
- 收入最低的群体为：失业者和产假期的女性

![image_1dbapeopo2791bvna4lnjf1u6rm.png-20.3kB][16]

各群体的违约比例：

![image_1dbapjmb6crg1i2h13b21gpnm7l13.png-23.7kB][17]

其中'Unemployed', 'Student', 'Businessman', 'Maternity leave'的人数太少，**缺乏统计意义**。排除这些极少数类之后，分析结果更加清晰。不同职业类型的人，违约概率有显著的差别，违约比例：'Working' > 'Commercial associate' > 'State servant' > 'Pensioner'。

## 年龄分布

用户的年龄分布相当均衡

- 年龄分布从20-70，总体分布较为均匀
- 随着年龄增大，违约比例呈现出**显著的递减**规律

![image_1dbaq6hmq1h054ld12i31ir1gp41g.png-15.5kB][18]

工龄数据存在大量（5W+）数值超过1000年的异常值，统一重置为-10。

![image_1dbaqf3361vmc193n1gd816es1g1s1t.png-16.7kB][19]

从工龄bin与违约的关系上，可以看出：

- 对于正常工龄数据，随着工龄增加，违约率显著下降
- 工龄异常值的群体，违约率低于平均水平

![image_1dbaqfqjl1an8kqm10fp1fb1s5r2a.png-12.5kB][20]

# 4. 贷款信息分析

## 贷款还款金额

贷款总额、年度还款额、分期年数的分布如下图所示：

![image_1dbaqptamcgveng7as631hga2n.png-26.2kB][21]

- 贷款总额偏态分布，中位数在5.13e5
- 年度还款额的中位数在2.49e4
- 还款年限的中位数在20年
> 此处的还款年限是直接用总贷款额除以年度还款额得到的，不准确。

疑问：还款年限非常长，最大的达到45年，看起来像是房贷。但从用户的住房拥有程度来看，不应该是房贷。

**各个特征与违约比例直接的关系**

- 贷款额
贷款额度在40w-80w的客户，违约比例最高，当贷款额超过80W，**金额越大，违约比例反而越低**。
![image_1dbark0tc16qhbn41a5iuufge934.png-19.8kB][22]
- 年度还款额
与贷款额的影响类似，随着额度增大，违约比例先升后降
![image_1dbarpik3a4u1f5e1u2t182c1r7r3k.png-15.5kB][23]
- 还款期数
还款期数与违约比例没有显著的相关性。
![image_1dbart9cu1atsj8u11h3ii01kmk4e.png-14.8kB][24]

## 还款/收入比分析

还款收入比=年度还款额/年度收入，体现了借款人的**杠杆比例**。还款收入比主要分布在0.1-0.4的区间，体现出大多数借款人量入为出的心态。

![image_1dbasodptas1m5r12961l31cum4r.png-12kB][25]

还款收入比与违约比例的关系：
比值在1之内的情况下，违约比例没有显著的变化，超过1之后，违约比例显著增大。但由于还款收入比大于1的用户数量极少，因此统计意义不大。

![image_1dbasrjac1n3k1lso1dj41si71v8s58.png-16.8kB][26]

How many days before the application did client change his registration

## 申请前的注册时间

DAYS_REGISTRATION, 即用户从更改“注册”到申请贷款的时间，“注册”的含义在数据集中没有说明。

> How many days before the application did client change his registration

![image_1dbat9kjucta1p15uka18gg94m5o.png-14.8kB][27]

注册年数越长，违约比例越低，呈显著的趋势。

![image_1dbatc4872fg1t8n1oob1l1649o65.png-12.6kB][28]


# 外部信用数据分析

归一化到0-1的信用分数：

![image_1dbbugqtb19l11vih1o8ibh15gq7f.png-10.4kB][29]

- EXT1
![image_1dbbuhjnh155giv61adsqt819qf7s.png-16.4kB][30]
- EXT2
![image_1dbbuhu8o1lc61mutq11hopvr489.png-15.6kB][31]
- EXT3
![image_1dbbuib3t17am19itmtd1o0q1hmh8m.png-16.2kB][32]


  [1]: http://static.zybuluo.com/frank0449/506uqis3cb87vsbnw7zf7gtw/image_1db46g791107upo21sio154j14i39.png
  [2]: http://static.zybuluo.com/frank0449/0rm7uzeklc9g3phmccqwn1w5/image_1db98o3ef1s4j1q5tduj1rlhiun9.png
  [3]: http://static.zybuluo.com/frank0449/obu7jd8pvcddnw1l7oeojtl0/image_1db991vfn1odmdbld2v12c643am.png
  [4]: http://static.zybuluo.com/frank0449/2mw0k7wfxjzjg16ikoakygie/image_1db99d48eodo11pmfk46fbthd13.png
  [5]: http://static.zybuluo.com/frank0449/b7tacl7w1568dbt1wd7kwlls/image_1db99pahnggj1q8h8v8p1t1igm1j.png
  [6]: http://static.zybuluo.com/frank0449/rifzv2litplc08c8t9gl0hf8/image_1db99srdt19g11lsruli1phqc8r20.png
  [7]: http://static.zybuluo.com/frank0449/d48820d3u8kshb31cq2o4mhz/image_1db9a7l3eeoetbc357egglir2d.png
  [8]: http://static.zybuluo.com/frank0449/iah3sqyt5s4cmsesexpf6kmo/image_1db9acm9c1aq3l79o9rhp3111n2q.png
  [9]: http://static.zybuluo.com/frank0449/njfb3zm2ii6e69ertans8abn/image_1db9afg801remb97khg1grps8837.png
  [10]: http://static.zybuluo.com/frank0449/hbxo9sswl3i5y5wq6gafj7cn/image_1db9aotv1s50a3rr24a2rln33k.png
  [11]: http://static.zybuluo.com/frank0449/ugrg9s9alt16qr7o9bhot9g6/image_1db9as1lrl5v1smh1mcc17lmov4e.png
  [12]: http://static.zybuluo.com/frank0449/rki0yd9xod2tpss5ctum37bs/image_1dbbpk7etm69101l1b9i109e12un6i.png
  [13]: http://static.zybuluo.com/frank0449/xp3eakh7rs6iqrekeb4xyfzz/image_1db9c3m8m15fp1inl18nm10ca14h15o.png
  [14]: http://static.zybuluo.com/frank0449/x4u9os6irc4e4wudcz5xop4l/image_1db9c8c9k36p1nu61589us1uc865.png
  [15]: http://static.zybuluo.com/frank0449/xrcm8k5y84124rke2ic0m5lq/image_1dbapcog814c01d6k6cd1eil16vq9.png
  [16]: http://static.zybuluo.com/frank0449/ujb7xog8i1mk5ubjzy40kxk0/image_1dbapeopo2791bvna4lnjf1u6rm.png
  [17]: http://static.zybuluo.com/frank0449/439o6w5gz0okdln8bj8588ou/image_1dbapjmb6crg1i2h13b21gpnm7l13.png
  [18]: http://static.zybuluo.com/frank0449/mltloqdo4axvomu94m8exf2t/image_1dbaq6hmq1h054ld12i31ir1gp41g.png
  [19]: http://static.zybuluo.com/frank0449/4w4fkw9qpfgxq6g1xrxagi8p/image_1dbaqf3361vmc193n1gd816es1g1s1t.png
  [20]: http://static.zybuluo.com/frank0449/wgc3o76kxjakx76wwj6b3ts3/image_1dbaqfqjl1an8kqm10fp1fb1s5r2a.png
  [21]: http://static.zybuluo.com/frank0449/ycmwwyocs57x81zji0x5gxgc/image_1dbaqptamcgveng7as631hga2n.png
  [22]: http://static.zybuluo.com/frank0449/2sw1h2b8d1t4bpt29k02kq3d/image_1dbark0tc16qhbn41a5iuufge934.png
  [23]: http://static.zybuluo.com/frank0449/3l9q471skrrzpjzemr5pj4iu/image_1dbarpik3a4u1f5e1u2t182c1r7r3k.png
  [24]: http://static.zybuluo.com/frank0449/6gvcf8sd6ogs66u71xitcxs3/image_1dbart9cu1atsj8u11h3ii01kmk4e.png
  [25]: http://static.zybuluo.com/frank0449/zjzihy1dvpzowzsfeaetrsog/image_1dbasodptas1m5r12961l31cum4r.png
  [26]: http://static.zybuluo.com/frank0449/fu9lv9ojfpkep8tafpbcxi65/image_1dbasrjac1n3k1lso1dj41si71v8s58.png
  [27]: http://static.zybuluo.com/frank0449/dahixmwqtc2bxml7xdfn7d2z/image_1dbat9kjucta1p15uka18gg94m5o.png
  [28]: http://static.zybuluo.com/frank0449/tqljt1ge8bwetdogqzsur5nk/image_1dbatc4872fg1t8n1oob1l1649o65.png
  [29]: http://static.zybuluo.com/frank0449/fmvtsjl36mcg45wgb2mhxq3q/image_1dbbugqtb19l11vih1o8ibh15gq7f.png
  [30]: http://static.zybuluo.com/frank0449/3ypbvjnp7jmq58p8m40dh1dz/image_1dbbuhjnh155giv61adsqt819qf7s.png
  [31]: http://static.zybuluo.com/frank0449/hnsuh8djijbjv6nawff4x7ry/image_1dbbuhu8o1lc61mutq11hopvr489.png
  [32]: http://static.zybuluo.com/frank0449/9u7gils0ajmzszncosyzz7lu/image_1dbbuib3t17am19itmtd1o0q1hmh8m.png