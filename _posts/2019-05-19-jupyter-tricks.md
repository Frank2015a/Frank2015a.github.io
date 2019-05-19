---
layout:     post
title:      Python|Juoyter使用技巧汇总
subtitle:   Notebook + Lab
date:       2018-06-05
author:     马骋
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - 数据挖掘 
    - python
---


# nbextention扩展

nbextention目前只支持Notepad，还不能支持lab。需要使用扩展功能的时候，只能再开一个notebook ssh。

## 安装配置

https://github.com/ipython-contrib/jupyter_contrib_nbextensions

安装方法：

```
pip install jupyter_contrib_nbextensions
jupyter contrib nbextension install --user
```

安装完成后，启动notebook，即可在配置页面进行设置：

![配置页面](http://static.zybuluo.com/frank0449/7rhclw8g30as3x8cuq0a210y/image_1db6ujq4g1m1i1mbj7741vea15pcm.png)

## 重要的扩展功能

- Table of Contents
对于大型的数据分析，toc导航对于提高效率非常重要。
![](http://static.zybuluo.com/frank0449/sb3gru13wkxask15qmyei8t2/image_1db6ur47u1t4pgj51bou1isl1bl013.png)

- 标题折叠
![](http://static.zybuluo.com/frank0449/li2rcjl5b5ieojw7hz33t0nk/image_1db6utag4s6k1m5o1l471fgf1vn91g.png)

- snippet
代码快速插入，但定义比较麻烦，先不使用

- Autopep8
自动对代码进行规范化


# jupyter lab

## open py with Notebook

用notebook的方法打开py文件，右键 `open with`->`notebook`即可。

自定义的module直接用notebook编辑即可，不用手动转换格式。可以像使用notebook一样使用py文件。



