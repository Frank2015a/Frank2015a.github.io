---
layout:     post
title:      Python|Pandas数据处理常用技术
subtitle:   datafame
date:       2018-06-05
author:     马骋
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - 数据挖掘 
    - python
---



# 行列操作

## 获取访问元素

- 获取df行列数
```c
df.shape        # get both
len(df.index)   # get col
```

- 获取column
```c
s = df['loan_amnt']     # Series
s = df.loc[:,'loan_amnt']
s2 = df[['loan_amnt']]  # DataFrame
```
注意：获取的结果是Series还是DataFrame。


- 获取row
```
df.iloc[0:2]  # 等价于df.iloc[0:2,:]
```

- 获取单元格
iloc的colname要用下标，loc支持字符串名。
```c
df.iloc[0:2,1]
df.loc[0:1,'loan_amnt']
```

- 转置
```
df = df.T 
df = df.transpose()
```

## rename

- rename columns
用map 的方式rename columns。
```
df = df.rename(columns={"loan_amnt": "loan_amount", 
                        "annual_inc": "annual_income"})
```
- rename index
```
frame3.index.name = 'year'; frame3.columns.name = 'state'
```


## drop行列

- drop列
一般用于清除无效特征，axis=1即drop列。
```c
frame2.drop('east', axis=1)
frame2.drop(['east','west'], axis=1)
del frame2['east']
```

- drop行
```
df.dropna()
```


## df拼接


- df上下合并
```
X_train_bal = pd.concat([X_pos, X_neg])  # df
y_train_bal = pd.concat([y_pos, y_neg])  # seriers
```
- 左右合并
设置`axis=1`

此处不涉及join操作。


## df统计变量频数

最有效的方法：

- value_counts
```c
df.fe1.value_counts()
pd.value_counts(y_train)
0    880095
1     10910
Name: label, dtype: int64
```

- groupby
```c
f_10w.groupby(by = 'label')['id'].count()

label
-1      479
 0    98335
 1     1185
```
输出清晰，指定对'id'统计，不用对其他列操作。

- Counter
```c
from collections import Counter
Counter(y)
Counter({2: 4674, 1: 262, 0: 64})
```

# dtype操作

## 统计dtype

用于确认数值型特征和非数值型特征的数量，以便后续的特征工程：

```c
df_train.dtypes.value_counts()
# output
dtypes of train data:
float64    65
int64      41
object     16
dtype: int64

```

通过df的info函数，也能得到类型统计：

```c
df_train.info()
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 307511 entries, 0 to 307510
Columns: 122 entries, SK_ID_CURR to AMT_REQ_CREDIT_BUREAU_YEAR
dtypes: float64(65), int64(41), object(16)
memory usage: 286.2+ MB

```

## dtypes 筛选col

可能筛选出数值型特征和非数值型的特征分别处理：

```c
df.select_dtypes(include='object').columns
df.select_dtypes(include=['int64', 'float64']).columns
```
https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.select_dtypes.html

获取col的dtype

```python
if df[col].dtypes == 'object':
    do
```

# map

> 这一部分的理解还不够。

## apply

apply**广泛**应用于元素的变换，经常与lambda函数组合。

```python
# 直接对元素apply
df.fe1.apply(lambda x: any([kw in x.lower() for kw in v]))

# 先定义单独的lambda函数
f = lambda x: len(re.findall(kw, x, re.IGNORECASE)) > 0
df[fe_name] = df.title.apply(f) & df.desc.apply(f) 
df['issue_d'].apply(lambda x: pd.to_datetime(x).year)
```

当apply 直接作用于df时，对行列变换：

## map

map起到了映射编码的效果，相当于labelencoder：
```
df.loan_status.map({'Fully Paid':0, 'Charged Off':1})
```

也可以用于异常值的替换。

## applymap

应用于df每个元素，不分行列：

```c
format = lambda x: '%.2f' % x
frame.applymap(format)
frame[0:1].applymap(format)
```

series不能直接用applymap，要用map。


# df存储压缩

很大的df数据加载后非常占用内存，数据压缩的思路：

- 64位转32位
- bool转int
- str转cate

```c
def convert_types(df, print_info = False):
    original_memory = df.memory_usage().sum()
    # Iterate through each column
    for c in df:        
        # Convert ids and booleans to integers
        if ('SK_ID' in c):
            df[c] = df[c].fillna(0).astype(np.int32)            
        # Convert objects to category
        elif (df[c].dtype == 'object') and (df[c].nunique() < df.shape[0]):
            df[c] = df[c].astype('category')        
        # Booleans mapped to integers
        elif list(df[c].unique()) == [1, 0]:
            df[c] = df[c].astype(bool)        
        # Float64 to float32
        elif df[c].dtype == float:
            df[c] = df[c].astype(np.float32)            
        # Int64 to int32
        elif df[c].dtype == int:
            df[c] = df[c].astype(np.int32)
        
    new_memory = df.memory_usage().sum()    
    if print_info:
        print(f'Original Memory Usage: {round(original_memory / 1e9, 2)} gb.')
        print(f'New Memory Usage: {round(new_memory / 1e9, 2)} gb.')
    return df
```

实测可以压缩的原来的1/3。

> 
Original Memory Usage: 0.49 gb.
New Memory Usage: 0.16 gb.


# 总体信息

要看到df数据的总体信息，可用`describe`和`info`函数，

## describe

describe函数，排除df中的nan后，给出特征列的统计信息：

- 数值型：count/mean/std/min/max，以及分位值
- 字符对象：unique/top/freq

通过字符特征的unique统计，可以快速看出哪些特征分别适合什么形式的编码。

**数值特征**

直接调用describe，默认值统计数值型特征。直接统计的结果，col与原df一致，在特征数量很多的时候，并不方便查看，转置后更方便。

```c
df_desc = df_train.describe().T
df_desc.head()
```

输出：

|          -       |   count |           mean |           std |    min |    25% |    50% |    75% |           max |
|:-----------------|--------:|---------------:|--------------:|-------:|-------:|-------:|-------:|--------------:|
| SK_ID_CURR       |  307511 | 278181         | 102790        | 100002 | 189146 | 278202 | 367142 | 456255        |
| TARGET           |  307511 |      0.0807288 |      0.272419 |      0 |      0 |      0 |      0 |      1        |
| CNT_CHILDREN     |  307511 |      0.417052  |      0.722121 |      0 |      0 |      0 |      1 |     19        |
| AMT_INCOME_TOTAL |  307511 | 168798         | 237123        |  25650 | 112500 | 147150 | 202500 |      1.17e+08 |
| AMT_CREDIT       |  307511 | 599026         | 402491        |  45000 | 270000 | 513531 | 808650 |      4.05e+06 |

如果不需要25%和75%的数值，可以在percentile中设置：

```c
df_desc = df_train.describe(percentiles=[0.5]).T
df_desc['null_ratio'] = 1 - df_desc['count'] / df_train.shape[0]
df_desc.sort_values(by = 'null_ratio', inplace=True)
```

|             -          |   count |             mean |            std |    min |    50% |    max |   null_ratio |
|:-----------------------|--------:|-----------------:|---------------:|-------:|-------:|-------:|-------------:|
| SK_ID_CURR             |  307511 | 278181           | 102790         | 100002 | 278202 | 456255 |            0 |
| REG_CITY_NOT_WORK_CITY |  307511 |      0.230454    |      0.421124  |      0 |      0 |      1 |            0 |
| FLAG_DOCUMENT_21       |  307511 |      0.000334947 |      0.0182985 |      0 |      0 |      1 |            0 |
| FLAG_DOCUMENT_20       |  307511 |      0.000507299 |      0.0225176 |      0 |      0 |      1 |            0 |
| FLAG_DOCUMENT_19       |  307511 |      0.000595101 |      0.0243875 |      0 |      0 |      1 |            0 |



**字符特征**

要统计字符特征，需显式指定类型：

```c
df_desc_obj = df_train.describe(include=[np.object]).T
# 统计频繁项的占比
df_desc_obj['freq_ratio'] = df_desc_obj['freq'] / df_desc_obj['count']
df_desc_obj.head(5)
```

统计结果可以得到unique数量，频繁项，以及频繁项的占比分布：

|       -            |   count |   unique | top           |   freq |   freq_ratio |
|:-------------------|--------:|---------:|:--------------|-------:|-------------:|
| NAME_CONTRACT_TYPE |  307511 |        2 | Cash loans    | 278232 |     0.904787 |
| CODE_GENDER        |  307511 |        3 | F             | 202448 |     0.658344 |
| FLAG_OWN_CAR       |  307511 |        2 | N             | 202924 |     0.659892 |
| FLAG_OWN_REALTY    |  307511 |        2 | Y             | 213312 |     0.693673 |
| NAME_TYPE_SUITE    |  306219 |        7 | Unaccompanied | 248526 |     0.811596 |


## info

info给出df的summary信息：

- 简洁版
给出dtypes的统计。

```c
df_train.info()

RangeIndex: 307511 entries, 0 to 307510
Columns: 122 entries, SK_ID_CURR to AMT_REQ_CREDIT_BUREAU_YEAR
dtypes: float64(65), int64(41), object(16)
memory usage: 286.2+ MB
```

- 详细版
除了简洁版的信息，还可以给出各个特征的数据类型，null统计。

```c
df_train.info(verbose=True,null_counts=True)
# verbose summary
SK_ID_CURR                      307511 non-null int64
TARGET                          307511 non-null int64
NAME_CONTRACT_TYPE              307511 non-null object
CODE_GENDER                     307511 non-null object
```


# 其他

## 相关性分析

```c
returns['MSFT'].corr(returns['IBM'])  # 列vs列
returns['MSFT'].cov(returns['IBM'])
returns.corr()  # df vs self，相关性矩阵
returns.cov()
```

## 数据存储csv

注意 index=False

## 获取整体null比例

基本实现：

```c
df_null = pd.DataFrame(df.isnull().sum() / df.shape[0])
df_null.sort_values(by=0, ascending=False).head(30)
```

函数实现：

```c
def df_null_stat(df, thres_null=0.5):
    df_null = pd.DataFrame(df.isnull().sum() / df.shape[0])
    df_null.rename(columns={0:'null_ratio'}, inplace=True);
    df_null = df_null.sort_values(by='null_ratio', ascending=False)
    
    num_null = df_null[df_null.null_ratio > thres_null].shape[0]
    print('num of col null ratio>{}: {}'.format(thres_null, num_null))
    cols_drop = list(df_null[df_null.null_ratio > thres_null].index)

    return df_null, cols_drop
```

