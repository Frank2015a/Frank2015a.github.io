---
layout:     post
title:      Python|常用编程技术
subtitle:   datafame
date:       2018-06-05
author:     马骋
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - 数据挖掘 
    - python
---

---

# 编程技术

## 自定义module

- 在nb中正常写代码
- 完成代码后, 调用命令转换成py文件
```
os.system('jupyter nbconvert --to script myutil.ipynb')
```

加载module：

- 设置sys路径
`sys.path.append('/root/cma/machine_learning_practice/01_ml')`
- 正常直接import即可，但由于自定义module需要经常修改，需要reload
```
from importlib import reload
reload(myutil)
```

## 函数help

定义函数的时候，按标准的方式定义Docstring：

```
def df_null_stat(df, thres_null=0.5):
    """
    args:
        - thres_null, thres to filter cols to drop
    return:
        - df_null, df of null ratios 
    useage:
    df_null, col_drop = df_null_stat(df, 0.5)
    """ 
    do
    return 
```

调用help的方式与标准函数相同：

```
myutil.df_null_stat?
Signature: myutil.df_null_stat(df, thres_null=0.5)
Docstring:
args:
    - df
    - thres_null, thres to filter cols to drop
return:
    - df_null, df of null ratios 
    - cols_drop, cols pass thres
useage:
df_null, col_drop = df_null_stat(df, 0.5)
df.drop(columns=col_drop, axis=1,inplace=True)
File:      ~/cma/machine_learning_practice/01_ml/myutil.py
Type:      function
```




