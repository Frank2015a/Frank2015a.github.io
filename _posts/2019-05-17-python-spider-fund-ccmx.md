---
layout:     post
title:      Python|爬取基金持仓明细数据
subtitle:   爬虫技术
date:       2018-06-05
author:     马骋
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - 数据挖掘 
    - 爬虫
---



天天基金网的基金持仓数据：http://fundf10.eastmoney.com/ccmx_163402.html

![image_1dcoao4rvd9u1rrpk8g1f9g107d9.png-101.6kB][1]

直接用软件爬取有一些问题：

- 后裔爬虫软件直接解析表格数据，其他的软件尚未尝试
- 静态网页默认只显示2019年（当年）的持仓数据，其他季度的需要手动查询才会显示

# 解决办法

## 获取年度持仓数据查询页面的链接

经过F12后台分析，发现各年度的基金持仓数据，需要通过一个查询页面访问：

http://fundf10.eastmoney.com/FundArchivesDatas.aspx?type=jjcc&code=163402&topline=20&year=2018

![image_1dcobgk9h1gj71nqo10tor7g14qs1j.png-112.6kB][2]

注意到网址的格式非常标准化，可以直接指定基金的代码、年度和显示数量：

- code=163402
- year=2018
- topline=20

因此可以批量构造所有查询的网址

原计划直接用构造好的网址，导入后裔软件爬取，发现直接爬取的质量很差，后处理非常麻烦，故弃用，直接编程实现。

### 不同年份页面格式的差别

注意到网页格式有变化，导致字段提取的代码需要分情况设置。

- 2018年及以前
![image_1dcoboc0k15lt1kog10651cp5eqj2d.png-32.6kB][3]
- 2019年
![image_1dcobnqe11r081k8fr0v8b54m20.png-44.2kB][4]

### 可查询年份的确认

直接输入一个不存在年份，网页会返回可查询年份的列表：

http://fundf10.eastmoney.com/FundArchivesDatas.aspx?type=jjcc&code=163402&topline=20&year=2050

```
var apidata={ content:"",arryear:[2019,2018,2017,2016,2015,2014,2013,2012,2011,2010,2009,2008,2007,2006,2005],curyear:2050};
```

即可直接解析可查询的年份list。

## 静态网页中表格数据的爬取

这是一个标准的数据爬取问题，直接解析网页即可。类似于爬取天气网站的数据：

```python
import requests
from bs4 import BeautifulSoup

try:
    r = requests.get('http://tianqi.2345.com/wea_history/60024.htm')
    r.encoding = r.apparent_encoding
except:
    print('网页源代码获取失败')

soup = BeautifulSoup(r.text,'html.parser')
soup.find('div','data-table')
```

关键是找到网页中的数据存储在'div','data-table'的结构中，通过F12工具即可查到：

![image_1dcob686loireb1c0h1cffhcfp.png-70.8kB][5]

解析对应的html代码段即可得到数据：

```
<div class="data-table" id="weather_tab"><table><colgroup><col width="20%"><col width="12%"><col width="12%"><col width="20%"><col width="20%"><col width="16%"></colgroup><thead><tr><th>日期</th><th class="center">最高气温</th><th class="center">最低气温</th><th>天气</th><th>风向风力</th><th>空气质量指数</th></tr></thead><tbody><tr onmouseout="this.style.backgroundColor=''" onmouseover="this.style.backgroundColor='#f1f1f1'"
```

解析网页中的tr、td，记录后，转换成dataframe即可。

```python
data = {'date':[],'high_temp':[]}

for tr in soup.find_all('tr')[1:]:
    date = tr('td')[0].text
    high_temp = tr('td')[1].text
    
    data['date'].append(date)
    data['high_temp'].append(high_temp)
    
import pandas as pd
pd.DataFrame(data)
```

得到结果如下：

![image_1dcobcerr1iao7di19f01l6o1u6v16.png-22.2kB][6]

# 其他编程技术

用reduce 实现dataframe的批量append:

```python
from functools import reduce
df_ccmx = reduce(lambda x, y: x.append(y) , df_by_year)
```



# 数据爬取完整代码

```python
import re
import copy
import pandas as pd
import requests
from bs4 import BeautifulSoup

from tqdm import tqdm
from multiprocessing import Pool
from functools import reduce

data_template = {'stock_code':[],
        'stock_name':[],
        'stock_ratio_pct':[],
        'stock_num_wan':[],
        'stock_value_wan':[],
        'fund_code':[],
        'quater':[]}
        
data_template = {'stock_code':[],
        'stock_name':[],
        'stock_ratio_pct':[],
        'stock_num_wan':[],
        'stock_value_wan':[],
        'fund_code':[],
        'quater':[]}
        
def get_year_avilable(fund_code):
    """
    查询一只基金有持仓明细数据的年份，返回网页列表
    Input:
        - fund_code, 待查询的基金代码
    Output:
        - years_avilable, 可供查询年份列表
        - ccmx_urls, 可供查询的持仓明细网页列表
    """
    url_fund_test = 'http://fundf10.eastmoney.com/FundArchivesDatas.aspx?type=jjcc&code={}&topline=20&year={}'.format(fund_code, 2030)
    r = requests.get(url_fund_test)
    r.encoding = r.apparent_encoding

    years_str = re.findall('\[(.*)\]', r.text)[0]
    years_avilable = [int(y) for y in years_str.split(',')]
    
    ccmx_urls = ['http://fundf10.eastmoney.com/FundArchivesDatas.aspx?type=jjcc&code={}&topline=20&year={}'.format(fund_code, year) for year in years_avilable]
    return years_avilable,ccmx_urls

 
 def get_ccmx_from_url(url_ccmx):
    """
    根据基金代码和已经查询到的tables数据，获取基金的持仓信息
    暂缺一些防出错code
    Input:
        - url_ccmx, 基金在某一年份持仓明细的查询链接
        url形如 'http://fundf10.eastmoney.com/FundArchivesDatas.aspx?type=jjcc&code=163402&topline=20&year=2018'
    Output:
        - df_ccmx, 持仓明细的df数据
    """    
    try:
        r = requests.get(url_ccmx)
    except:
        print('request fail')
        return None
    
    r.encoding = r.apparent_encoding
    soup = BeautifulSoup(r.text,'html.parser')
    tables = soup.find_all('div','box')
    
    fund_code = re.findall(r'code=(\d+)&', url_ccmx)[0]
    year = re.findall(r'year=(\d+)',url_ccmx)[0]
    # 2019的查询页面和其他年份不同
    if year=='2019':
        idx_stock_ratio = 6
        idx_stock_num = 7
        idx_stock_value = 8
    else:
        idx_stock_ratio = 4
        idx_stock_num = 5
        idx_stock_value = 6 
    
    data = copy.deepcopy(data_template)
    for table in tables:
        # 获取table对应的季度数
        quarter_tmp = table.find('label', 'left').text.split('\xa0')[-1]
        quarter_tmp = re.findall(r'(\d+)年(\d+)季度', quarter_tmp)[0]
        quater_str = 'Q'.join(list(quarter_tmp))

        for idx, tr in enumerate(table.find_all('tr')):
            if idx==0:
                # 跳过标题行
                continue
            # 记录股票代码、简称、持仓占比、持股数、持仓市值
#             embed()
            stock_code = tr('td')[1].text
            stock_name = tr('td')[2].text
            try:
                stock_ratio_pct = float(tr('td')[idx_stock_ratio].text.replace('%', ''))
            except:
                stock_ratio_pct = None
            
            try:
                stock_num_wan = float(tr('td')[idx_stock_num].text.replace(',', ''))
            except:
                stock_num_wan = None
            
            try:
                stock_value_wan = float(tr('td')[idx_stock_value].text.replace(',', ''))
            except:
                stock_value_wan = None

            data['stock_code'].append(stock_code)
            data['stock_name'].append(stock_name)
            data['stock_ratio_pct'].append(stock_ratio_pct)
            data['stock_num_wan'].append(stock_num_wan)
            data['stock_value_wan'].append(stock_value_wan)
            data['fund_code'].append(fund_code)
            data['quater'].append(quater_str)
    df_ccmx = pd.DataFrame(data)     
    return df_ccmx
    
def get_ccmx_all_years(fund_code):
    """
    下载一个基金的历年全部持仓数据
    Input:
        - fund_code, 基金代码
    Output:
        - df, 历年个季度的持仓数据
    """
    years, urls = get_year_avilable(fund_code)
    df_by_year = []
    for url_ccmx in urls:
        # 正常直接运行，遇到挂的地方，打印出网页
        try:
            df_by_year.append(get_ccmx_from_url(url_ccmx))
        except:
            print(url_ccmx)
    df_ccmx = reduce(lambda x, y: x.append(y) , df_by_year)
    return df_ccmx

with Pool(4) as p:
    res_df = list(tqdm(p.imap(get_ccmx_all_years, fund_codes), total=len(fund_codes)))
df_ccmx_all = reduce(lambda x, y: x.append(y) , res_df)
```



  [1]: http://static.zybuluo.com/frank0449/zz9mwb0mr23zhcm6yrdhjiew/image_1dcoao4rvd9u1rrpk8g1f9g107d9.png
  [2]: http://static.zybuluo.com/frank0449/tvoz648bnpg7svkc2t6ilbuo/image_1dcobgk9h1gj71nqo10tor7g14qs1j.png
  [3]: http://static.zybuluo.com/frank0449/qexmt49w6tq2pqibg588oeyb/image_1dcoboc0k15lt1kog10651cp5eqj2d.png
  [4]: http://static.zybuluo.com/frank0449/ef1xbwa4wau9sw5b2u4sbiwg/image_1dcobnqe11r081k8fr0v8b54m20.png
  [5]: http://static.zybuluo.com/frank0449/gpxroldrl9y8jii4bt5l1asw/image_1dcob686loireb1c0h1cffhcfp.png
  [6]: http://static.zybuluo.com/frank0449/utif09p1ikw4iqbuicw1jxp9/image_1dcobcerr1iao7di19f01l6o1u6v16.png