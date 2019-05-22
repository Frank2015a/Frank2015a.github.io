---
layout:     post
title:      数据挖掘|Lending Club P2P风险策略分析
subtitle:   信贷数据EDA+风险分析
date:       2018-06-05
author:     马骋
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - 数据挖掘
    - 信贷风控
---

---

# 引言

Lending Club（以下简称LC）是美国的P2P互联网消费借贷平台，可以看做P2P行业的鼻祖。LC创始于2007年，2008金融危机期间抓住信贷缺失的机遇，快速发展，目前是美国最大的P2P平台。关于LC的业务模式参见[《Lending Club简史》读书笔记--美国P2P行业发展浅析][1]
本文结合kaggle的Lending Club数据集，分析了LC的业务状况、信贷违约，并提出一些业务提升的建议。

# 0. 关键结论与建议

**关键结论**：

1. LC的P2P贷款业务发展迅速，从2007到2015年年均增长率超过100%，2015年的贷款总量超过60亿美元；
用户的单笔贷款金额逐年增加，贷款金额主要集中在5000-20000美元的区间；
2. 贷款的额度大、分期数多（36期、60期）、贷款频率低，主要用途是**集合债务、换信用卡**，用于消费场景的比例较小。贷款利率主要由用户的信用分和分期数决定，利率集中在10%到20%的区间。
3. LC平台的风控能力较差，在2007-2015年之间，贷款的坏账率高达17%，实际收回还款总额是贷款本金的108%，收益率并不高。
4. 风险因素中，违约率随着贷款信用水平下降而单调增加，可见美国的FICO信用评分就能较准确地评估违约风险。

**业务提升建议**：

1. LC平台的贷款利率已经较高，提升业务的关键是加强风控，减少坏账率，并吸引更多优质客户。
2. 信用水平最低的F、G级别贷款违约风险高、总量少，且还款的回报率显著低于级别贷款，建议停止发放此类贷款；
3. 高收入群体的违约率低、贷款金额大，是LC平台的优质客户群。
建议主动吸引更多**高收入用户**以LC作为贷款融资渠道，通过提升客户质量来降低贷款不良风险
4. 发放的贷款的主要分布在B/C/D等级，通过机器学习模型，寻找信用分之外的违约因素，**控制主要客户群的违约风险**水平；
5. LC当前的分期模式只支持36期和60期，建议设置12/24月分期的贷款产品，满足更多**短期、高频的消费贷款**需求，提升用户复用率，扩大业务量；


# 1. 数据描述

Lending Club(LC)的数据见：https://www.kaggle.com/wendykan/lending-club-loan-data 
kaggle提供的数据包含从2007年到2015年的发放的887k贷款记录，每个贷款记录有75个维度的特征。

![image_1d64mijhm1m291tis11bl1o3resh9.png-32.9kB][2]

主要的数据特征及语义：

| 特征 | 语义 | 备注 |
| ------ | ------ | ------ |
| **issue_d** | 贷款发放时间 | 年-月 |
| **grade** | LC的信用评级 | 主要根据FICO分和贷款数额、期限确定 |
| **int_rate** | LC发放贷款利率 | 主要由grade评级决定 |
| **annual_inc** | 年收入 | 体现还款能力 |
| **loan_amnt** | 贷款数额 |  |
| **term** | 分期期数 | 36期（3年），60期（5年） |
| **loan_status** | 贷款状态 | 正常、逾期、违约 |
| delinq_2yrs | 2年内的钱款 | 最近的信用记录 |
| inq_last_6mths | 过去6个月，信用记录被查询次数 | 可能意味着向其他机构申请贷款未批准 |
| home_ownership | 住房状态 | 自有、租房、按揭 |
| desc | 借款描述 | 文本信息，需要NLP处理 |
| dti | LC以外需要还款的数额 |  |
| emp_title | 职业职位 | 可以考虑对职业分群 |
| emp_length | 职业年限 | 数值型 |

以上数据维度较为丰富，数据应用需要注意的问题：

- 一些重要信息没有包括，如年龄、性别、婚姻状态、FICO信用分数，可能考虑隐私保护；
这些信息在实际风控中非常重要；
- 贷款状态描述当前的情况，已经放款的贷中项目，其最终是否违约不确定的，因此直接计算得到的违约率是偏低的；

# 2. LC借贷业务整体分析

数据期限从2007年6月到2015年12月，共8年半。

## 2.1 业务总量

LC在截至2015年底的8年半中，总共发放的贷款笔数为887k。逐年的贷款总量和笔数如下图：

![image_1d6557njbc01dit1iiu87t14j213.png-34.7kB][3]

可以看出，贷款笔数和贷款总量均呈现逐年快速增加的趋势，2015年发放贷款总量超过60亿美元，超过40万笔。

按月份统计的贷款量增长如下：

![image_1d65l813q1ltdlp0qom7rc9eu55.png-46.2kB][4]

注意到在2014年以前，贷款量的增长曲线非常平滑，在2014年6月之后，新增贷款量出现非常大的波动，大致呈现出每3个月出现一个贷款量高峰的模式。

![image_1d65lb68l199v1h1a1ss1oui1md75i.png-28.3kB][5]

在2014-2015的详图中，从2014年7月起，每隔1个季度，出现一次贷款高峰。在贷款高峰月，未发现美国利率和资本市场有异常情况，因此可能的解释为：由于某种政策的影响，每个季度开始的月份，是美国消费者债务整合操作的高峰时期。

在实际业务中，如果遇到这样的资金需求量波动周期，应该挖掘出背后的因素，以便提前筹备资金。

## 2.2 单笔贷款

![image_1d65p9si21p036e71f1s15qch6u7c.png-40.6kB][6]

从2007年到2015年，左图看出平均单笔贷款的额度从8254美元增加到15240美元，单笔额度逐年增加。这一趋势与贷款人年均收入逐年增加（右图）的趋势对应，显示了借款人和投资人的信心均随着金融危机后美国经济复苏而增强。

单笔贷款金额分布如：

![image_1d655ja8v1she1tutu041rf84911t.png-53.2kB][7]

可以注意到：

- 最小的单笔额度为500美元
- 最大额度在2007-2010年为25000美元，在2011年之后，最大额度调整为35000美元
- 在整数额度（5k、10k等），借贷笔数出现高峰，体现出借款用户金额**凑整**的心态
- 贷款额度主要分布在5000-20000美元的区间

## 2.3 利率与分期

LC的贷款分期数只有**2个档次**：3年分期（36月）、5年分期（60月）。贷款的利率主要由grade评级、贷款额度、分期数决定。对同一个人，申请同一额度，如果分期数增加，贷款评级会下调，即利率增大。

所有贷款项目，最低利率5.32%，最高利率28.99%，平均值在**13.25%**.

![image_1d65a2hv71617c62a9n4rq1he22a.png-45.8kB][8]
可以看出：

- 贷款项目的利率主要分布在10%到20%的区间；
- 2007年的项目最高利率不超过18%，后续利率的上限达到了29%；
- 与国内的个人贷款项目相比，利息是偏高的如，
    - 借呗日利息万2.5到万5（年化9.1%-18.25），
    - 微信微粒贷年化18.25%

贷款平均利率在不同年份有显著变化：

![image_1d65om32a2uocnv19mj1d8uupm6c.png-16kB][9]

对比不同分期数的利率设置：

- 不同分期，利率的上下限没有显著变化
- 分36期的贷款利率均12.02%，显著小于分60期的利率均值16.11%

![image_1d65akijsrg26a11fhhr3e1nsm2n.png-5.2kB][10]

对比不同评级分组的贷款利率，评级主要根据FICO信用分，将贷款项目分为A-G的等级，信用依次降低，利率依次增大。

![image_1d65b42jr1jn41hns1pt01gvb1tp03h.png-14.6kB][11]

可以看出，

- 不同grade组平均利率呈现显著的台阶；
- 但最低贷款利率除了G以为区别不大。可能较低评级的贷款在特定的情况下，也会取得较低利益；
- 从数量上的来看，A-D等级的贷款占了贷款笔数的90%以上，E-G等级越低的贷款，数量越少；
看一看出平台对申请贷款的准入门槛较高，低等级贷款发放量很少。

## 2.4 贷款用途

申请贷款必须填写贷款的用途，最常见的用途是**集合债务**贷款，或者偿还高利息的**信用卡**贷款。
其余的用途有： 房屋装修、婚礼、汽车、小生意、教育培训。

![image_1d65nld9o16aqnql5sd5313e75v.png-29kB][12]

可以看出集合债务和信用卡是贷款的主要用途，其余的选项占比很少。

## 2.5 用户行为分析

尝试分析LC贷款用户的重复贷款行为，检查数据集中的member_id，发现没有重复的。这意味着所有的用户在2007-2015年，在LC平台上只有一次贷款。这一结论不符合预期，在《Lending Club简史》中，存在一些用户长期在LC上融资。猜测数据可能存在的问题是：

- LC放出公开数据时，只保留了一个用户ID的一次贷款行为；
- 处于隐私保护的考虑，修改了同一个用户在不同贷款项目的member_id;

从贷款额度和分期的角度来看，LC平台的贷款度平均在1万美元以上，分期至少36期。尤其在分期上缺乏弹性，可以推测贷款用户只有融资需求较大时，才会在LC平台贷款。因此用户行为总体是：**大额、低频**。这种模式可能造成老用户的活跃度不高。

# 3. 贷款违约分析

对于各种贷款状态，划分为：

- Bad，不良/坏账
- Late，逾期
- Good，完成还款
- Cur，贷中

| 贷款状态 | 语义 | Bad | Late| Good | Cur |
| ------ |------ | ---- | ---- |---- | ---- |
| Current | 贷中 | - | -| - | Y |
| Fully Paid | 全额还款 | - | -| Y | -|
| **Default** | 违约 | Y | - | - | -|
| **Charged Off** | 坏账 | Y | - |  - | -|
| Issued  | 发布 | - | - |  - | Y |
| In Grace Period | 宽限期（逾期15天之内） | - | Y | - | -|
| Late (16-30 days) | 逾期16-30天 | - | Y| - | -|
| Late (31-120 days) | 逾期31-120天 | _ | Y | - | -|
| Does not meet the credit policy. Status:Fully Paid | 全额还款（不符合信用） | - | -|  Y | -|
| Does not meet the credit policy. Status:Charged Off | 坏账（不符合信用） | Y | - | - | -|


## 3.1 贷款风险与资损情况分析

各年度贷款统计如下：

- bad_ratio，除去贷中（Current）之后的坏账比例
- cur_ratio，贷中的比例
- ratio_pymnt，总还款额率=（本金+利息）/总贷款额
- ratio_prncp，本金偿还率= 偿还本金/总贷款额

![image_1d8ofochkodi1it61k11iv1dmv1p.png-18.3kB][13]

可以看出：

- 2007-2009年的贷款没有Current
    - 2007-2008，贷款不良率**超过20%**，本金回收率只有0.78和0.80，总体上有资金损失；
    - 2009年，贷款不良率在14%，总体还款/本金比率为1.08，即总体上是有收益的。
- 2010-2012年，current状态的贷款，比例不超过10%
    - 总体还款/本金比率为1.08
    - 贷款不良率稳定在0.14-0.16，本金收回率在0.87 
- 2013-2015年，大量的贷款处于Current，因此回收资金的比例很低。
贷中项目最终可能发展为Bad/Good/Late中的一类，当前的贷款不良率没有反应真实的违约状况。

需要注意的是，Bad贷款并不意味着资金全部损失，实际上贷款用户会在分期还款的一定阶段出现逾期，最终坏账不再还款。统计Bad贷款可以收回的本金比例在43.7%，即平均损失月60%的本金。

坏账贷款停止还款的期数分布如下，平均在14分期是发生违约。

![image_1d8og3geg1anp1ah0jpmgjfgfa36.png-58.8kB][14]

## 3.2 贷款风险与信用等级

### 贷款评级分析

LC贷款数据中，贷款评级grade直接体现了信用等级。不同等级的贷款不良率如下：

![image_1d8ogactr34g1ila1mndng5313j.png-21.5kB][15]

随着grade降级贷款的不良率单调增加，这说明了美国的FICO信用评价是非常可靠的。E-G级别贷款的不良率很高，但从贷款总量上来看，占比并不大。因此要控制总体的贷款不良率，主要还是应该从B、C级别的贷款入手。

![image_1d8ogbfom1b85st1aog1oet1s3i50.png-31.6kB][16]

### 收入水平分析

对借款人的**收入水平分级**：

- 年收入<10w美元：**low**
- 年收入在10w和20w美元之间：**medium**
- 年收入大于20w美元：**high**

![image_1d664qegl1hhs19ul1m3sof25lbat.png-64kB][17]

可以看出：

- 在统计意义上，高收入群体的贷款不良率更低；
- 从贷款数额上看，高收入群体的贷款额度也更大；
在LC的用户群中，高收入群体可以作为优质用户。

需要注意的是，高收入并不等价于高信用，依然存在E-G低信用评级的贷款。

![image_1d6723k75agk166d20n1rfb1hd0ba.png-16.5kB][18]

## 3.3 贷款风险与分期数

LC的贷款分期数只有36月和60月分期两种，分析两种分期模式下贷款的不良率：

![image_1d672k1ci12j71fg144114dk1lv2bn.png-14.2kB][19]

从统计上来看，60月分期的贷款不良率显著高于36月分期的贷款。综合分期数和贷款等级，如下：

![image_1d67308941j7hlr21lnb1ubh15gsc4.png-19.2kB][20]

注意到：

- A-D评级较高的贷款中，分期36的数量多于分期60
- E-G评级较低的贷款中，分期60月的贷款比例很高

即由于更多的高风险贷款选择了60月分期，使得60月分期贷款的总体不良率更高。对比各等级贷款，在不同分期数下的不良率，注意到各等级的贷款，采用60月分期的贷款总体比36月分期的不良率更低。

![image_1d673ei0bedk7b91brb1s3ug6kch.png-23.9kB][21]

## 3.4 贷款用途与使用目的

最常见的用途是**集合债务**贷款，或者偿还高利息的**信用卡**贷款。其余的用途有： 房屋装修、婚礼、汽车、小生意、教育培训。

不同目的贷款的不良率如：

![image_1d673rrkj1djd1e00ghf1gpvca7cu.png-30.6kB][22]

注意到：

- 贷款不良率最高的类别是教育（20.8%）和小生意（17.16%）
- 集合债务、信用卡是贷款量最大的业务，不良率相对较低，即**主营业务**风险可控性强

为何用于教育培训的贷款不良率是最高的呢？教育类贷款总共423笔，其中88笔不良，发放年份集中在2007-2010年，2010年之后几乎没有再发放教育类贷款。可能的原因分析：

- 2007-2010年是金融危机之后就业形势总体较差的时期，在这一阶段贷款用于教育，很可能投入的资金难以取得回报
- 可能由于教育类贷款总体风险太高，LC在2010年之后，基本停止了教育类贷款的发放

## 3.5 其他因素的分析

对其他因素的分析没有发现与贷款不良率有较强的相关性。

- 贷款额度
![image_1d674luib188313eg1utbknba1mdb.png-19.3kB][23]
- 贷款人住址地域
此处的地域分类为美国各州的东南、东北等大区域
![image_1d674t47d1443qd4p1q2fsjerf5.png-18.6kB][24]

可以看出，贷款额度、贷款人地域等因素对贷款不良率几乎没有影响。

# 4. 贷款收益比分析

信用等级高的贷款，风险低，利率收益低；反之，信用等级低的贷款，风险高，利息收益也高。那么实际运行中，哪些群体的贷款最终收益更高呢？以最终收回的还款比例衡量：

- 不同等级贷款，/C/D等级贷款的收益最高，而低信用的F/G等级贷款，收益最低；
![image_1d8oggd14da11tj4130c1e7g110l5d.png-11.7kB][25]
- 不同收入群体，高收入群体的贷款收益最好
![image_1d8ogjk3e16qdeabinj1v4q9o5q.png-12.9kB][26]

因此在业务策略上，建议停止发放低信用的F/G等级贷款，在客户群上多吸引高收入优质客户群体。

  [1]: https://www.jianshu.com/p/dc7efd39cb4b
  [2]: http://static.zybuluo.com/frank0449/8fzb0x1k5v3m7fmmu5idbv12/image_1d64mijhm1m291tis11bl1o3resh9.png
  [3]: http://static.zybuluo.com/frank0449/t4icngum015eqhdhlz5qgyc9/image_1d6557njbc01dit1iiu87t14j213.png
  [4]: http://static.zybuluo.com/frank0449/knejm6895zecpsk84i6v5fl1/image_1d65l813q1ltdlp0qom7rc9eu55.png
  [5]: http://static.zybuluo.com/frank0449/l6dkepgxp4gcucu6tiiaheam/image_1d65lb68l199v1h1a1ss1oui1md75i.png
  [6]: http://static.zybuluo.com/frank0449/20da7xmdktybac4e9m6cejc9/image_1d65p9si21p036e71f1s15qch6u7c.png
  [7]: http://static.zybuluo.com/frank0449/qkmvfwjtw9vfo57x5fu1op7e/image_1d655ja8v1she1tutu041rf84911t.png
  [8]: http://static.zybuluo.com/frank0449/pssivd1nwjlfmfvtj8bjcrt6/image_1d65a2hv71617c62a9n4rq1he22a.png
  [9]: http://static.zybuluo.com/frank0449/klu4y56usv43cfqcmhi1jdsy/image_1d65om32a2uocnv19mj1d8uupm6c.png
  [10]: http://static.zybuluo.com/frank0449/aiffqna3b7w1o1qebr023u09/image_1d65akijsrg26a11fhhr3e1nsm2n.png
  [11]: http://static.zybuluo.com/frank0449/gbc2mzidlhg5hwvqxfv5au6z/image_1d65b42jr1jn41hns1pt01gvb1tp03h.png
  [12]: http://static.zybuluo.com/frank0449/i98lqr8yfqy10dihyj6b5qyo/image_1d65nld9o16aqnql5sd5313e75v.png
  [13]: http://static.zybuluo.com/frank0449/ds32ks341fiq9wqkjsdb4mug/image_1d8ofochkodi1it61k11iv1dmv1p.png
  [14]: http://static.zybuluo.com/frank0449/m46et87xvqdpmilvrbaj6vjr/image_1d8og3geg1anp1ah0jpmgjfgfa36.png
  [15]: http://static.zybuluo.com/frank0449/h9ifn1yc2w8bang0fcx5gl2s/image_1d8ogactr34g1ila1mndng5313j.png
  [16]: http://static.zybuluo.com/frank0449/esmdddey8m6ritqlftacksxq/image_1d8ogbfom1b85st1aog1oet1s3i50.png
  [17]: http://static.zybuluo.com/frank0449/k2dxs3aa6ilbq2hw77fkluqy/image_1d664qegl1hhs19ul1m3sof25lbat.png
  [18]: http://static.zybuluo.com/frank0449/dna8alb3p4y7gs0o7hoyhoje/image_1d6723k75agk166d20n1rfb1hd0ba.png
  [19]: http://static.zybuluo.com/frank0449/7tskqvtrfbrz85mskxr0g8ht/image_1d672k1ci12j71fg144114dk1lv2bn.png
  [20]: http://static.zybuluo.com/frank0449/jajspg2y61zrcp8efayku37c/image_1d67308941j7hlr21lnb1ubh15gsc4.png
  [21]: http://static.zybuluo.com/frank0449/jhr6vw0q3wr05angv0go9oaq/image_1d673ei0bedk7b91brb1s3ug6kch.png
  [22]: http://static.zybuluo.com/frank0449/qunmhz9vr282tlmywukolog9/image_1d673rrkj1djd1e00ghf1gpvca7cu.png
  [23]: http://static.zybuluo.com/frank0449/5k0v1vc05x6bj6k7zcorpp6n/image_1d674luib188313eg1utbknba1mdb.png
  [24]: http://static.zybuluo.com/frank0449/tcuvpxzfyachqaiyh64topim/image_1d674t47d1443qd4p1q2fsjerf5.png
  [25]: http://static.zybuluo.com/frank0449/mx9ft7xk9lzds2cqg0ctnzru/image_1d8oggd14da11tj4130c1e7g110l5d.png
  [26]: http://static.zybuluo.com/frank0449/shdpfldnqo3s76hzzwpusfco/image_1d8ogjk3e16qdeabinj1v4q9o5q.png