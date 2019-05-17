---
layout:     post
title:      Python|Plotly
subtitle:   Plotly交互式可视化
date:       2018-06-05
author:     马骋
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - 数据挖掘 
    - python
---

# 引言

plotly是一种基于网络服务的**交互式**python绘图工具。

为什么要配置plotly？Kaggle上使用plot的开发者越来越多，如果不能使用plotly，难以吸收其他人的成果。

offline配置没有走通，以后主要就用online方式了。

# installl & import

```
import plotly.plotly as py
import plotly.figure_factory as ff
```

# online mode


## 账户设置

需要在plotly网站注册账户

> macheng2019/mypwd

直接在NB中设置账户的方法：

```
import plotly 
plotly.tools.set_credentials_file(username='DemoAccount', api_key='lr1c37zw81')
```

运行set_credentials_file后，会在home路径下生成`~/.plotly/.credentials`，以后就不用重复登录了。

```
{
    "username": "username",
    "api_key": 'your_key'
    "proxy_username": "",
    "proxy_password": "",
    "stream_ids": []
}
```

## 绘图

如此设置之后，可以正常使用，并在plotly网站上查看绘图。

```
table = ff.create_table(df.head())
py.iplot(table)
```

绘图显示在https://plot.ly/~macheng2019/6/#/ 


![xx](http://static.zybuluo.com/frank0449/pxqd6njai9pm3nuzh414gzoc/image_1db1i0vs21tb48fbnna52frgf9.png)


# offline mode


pylab中 offline方法没有走通，放弃。

据说要安装pylab extension：https://github.com/jupyterlab/jupyter-renderers/tree/master/packages/plotly-extension

以下尝试是失败的：

```
jupyter labextension install @jupyterlab/plotly-extension
conda install nodejs

import plotly.offline as pyo
import plotly.graph_objs as go
# Set notebook mode to work in offline
pyo.init_notebook_mode()
```

