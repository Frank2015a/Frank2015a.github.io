---
layout:     post
title:      Python|Pandas爬虫
subtitle:   爬虫
date:       2018-06-05
author:     马骋
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - 数据挖掘 
    - python
---


# 模拟浏览器的爬虫设置

参考https://blog.csdn.net/chenlibao0823/article/details/81989002 中的爬虫设置。

```
from selenium import webdriver
from bs4 import BeautifulSoup

opt = webdriver.ChromeOptions()
opt.set_headless()
driver = webdriver.Chrome(options=opt)
```

利用chrome后台进行数据爬取，直接调用会报错，问题是chrome后台没有设置好。

处理办法：

- 服务器端安装chrome浏览器与chromedriver
- chrome安装并检查version
安装chrome的问题主要是dependency安装比较麻烦，处理办法：
```
sudo dpkg -i google-chrome-stable_current_i386.deb
sudo apt-get install -f
```
安装完成后，通过locate查找到安装的路径：/opt/google/chrome/chrome
查询chrome版本号：
```
./chrome --version
Google Chrome 74.0.3729.169 unknown
```
- 安装与chrome匹配的chromedriver
在官网下载对应版本的驱动，拷贝到chrome的安装目录下,确认版本对应
```
cd /opt/google/chrome
./chromedriver --version
ChromeDriver 74.0.3729.6 (255758eccf3d244491b8a1317aa76e1ce10d57e9-refs/branch-heads/3729@{#29})
```

- 将路径添加到PATH
```
export PATH=/opt/google/chrome:$PATH
```

- python调用
注意设置no-sandbox，以免报错。
```
sys.path.append('../machine_learning_practice/01_ml/')
options = webdriver.ChromeOptions() 
options.add_argument('--headless') 
options.add_argument('--no-sandbox')
driver = webdriver.Chrome(executable_path="/opt/google/chrome/chromedriver",
                          chrome_options=options)
```

其余使用方法正常。