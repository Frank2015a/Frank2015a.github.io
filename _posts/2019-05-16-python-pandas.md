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
```
df.shape        # get both
len(df.index)   # get col
```

- 获取column
```
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
```
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
```
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
```
df.fe1.value_counts()
pd.value_counts(y_train)
0    880095
1     10910
Name: label, dtype: int64
```

- groupby
```
f_10w.groupby(by = 'label')['id'].count()

label
-1      479
 0    98335
 1     1185
```
输出清晰，指定对'id'统计，不用对其他列操作。

- Counter
```
from collections import Counter
Counter(y)
Counter({2: 4674, 1: 262, 0: 64})
```

# dtype操作

## 统计dtype

用于确认数值型特征和非数值型特征的数量，以便后续的特征工程：

```
df_train.dtypes.value_counts()
# output
dtypes of train data:
float64    65
int64      41
object     16
dtype: int64

```

通过df的info函数，也能得到类型统计：

```
df_train.info()
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 307511 entries, 0 to 307510
Columns: 122 entries, SK_ID_CURR to AMT_REQ_CREDIT_BUREAU_YEAR
dtypes: float64(65), int64(41), object(16)
memory usage: 286.2+ MB

```

## dtypes 筛选col

可能筛选出数值型特征和非数值型的特征分别处理：

```
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

```
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

```python
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


# 其他

## 相关性分析

```
returns['MSFT'].corr(returns['IBM'])  # 列vs列
returns['MSFT'].cov(returns['IBM'])
returns.corr()  # df vs self，相关性矩阵
returns.cov()
```

## 数据存储csv

注意 index=False

## 获取整体null比例

基本实现：

```
df_null = pd.DataFrame(df.isnull().sum() / df.shape[0])
df_null.sort_values(by=0, ascending=False).head(30)
```

函数实现：

```python
def df_null_stat(df, thres_null=0.5):
    df_null = pd.DataFrame(df.isnull().sum() / df.shape[0])
    df_null.rename(columns={0:'null_ratio'}, inplace=True);
    df_null = df_null.sort_values(by='null_ratio', ascending=False)
    
    num_null = df_null[df_null.null_ratio > thres_null].shape[0]
    print('num of col null ratio>{}: {}'.format(thres_null, num_null))
    cols_drop = list(df_null[df_null.null_ratio > thres_null].index)

    return df_null, cols_drop
```

