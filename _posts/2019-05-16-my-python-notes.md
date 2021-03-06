---
layout:     post
title:      Python|常用编程技巧
subtitle:   通用
date:       2018-06-05
author:     马骋
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - python
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

Ipython的auto-reload更方便，修改module后，可以自动加载

>  IPython comes with automatic reloading magic. 
You can reload all changed modules before executing a new line.

```
%load_ext autoreload
%autoreload 2
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

# embed调试

在code中插入embed，运行到断点处，即可进入Ipython 模式。
以便交互式地查看变量，调试代码。

```Python
from IPython import embed
a = 10
b = 20
embed
# embed(header='First time')
c = 30
d = 40
```

注意：

- 只能在单线程的模式下使用embed
- 退出的时候Ctrl+C即可。

## matplotlib中文显示

linux 系统中python绘图中的中文显示往往有问题，显示成方块。 
一个靠谱的解决办法是：https://blog.csdn.net/jeff_liu_sky_/article/details/54023745 

- 下载中文字体
http://fontzone.net/download/simhei 
- 找到matplotlib的字体路径
```
locate -b '\mpl-data'
```
并将字体copy到fonts/ttf这个目录
- 清除matplotlib的缓存路径，即重设
- python中设置字体
    - coding:utf-8 说明文件编码格式
    - 用simhei 字体显示中文
    - `mpl.rcParams['axes.unicode_minus'] = False` 这个用来正常显示负号
```
import matplotlib as  mpl
from matplotlib  import pyplot as plt
mpl.rcParams[u'font.sans-serif'] = ['simhei']
mpl.rcParams['axes.unicode_minus'] = False
```


