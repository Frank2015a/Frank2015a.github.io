---
layout:     post
title:      数据库基础|sqlite
subtitle:   sql
date:       2018-06-05
author:     马骋
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - 数据库 
    - sql
---


# sqlite基础

sqlite是**文件型**数据库，数据直接存储在xx.db的文件中，数据的迁移直接拷贝文件即可，轻便。

# 基本操作

```sql
.mode column # 按column显示的格式
create table t1(a int, b int);
.tables
insert into t1 values (1,2);
select * from t1
.quit
```

参考的学习资源：https://www.runoob.com/sqlite/sqlite-tutorial.html

# python sqlite

```python
# 数据库连接，可以是不存在的
sqlite_con = sql.connect('./data/fund_data.db')
# df数据存储到sql
df.to_sql(name='fund_jjjz', con=sqlite_con, if_exists='replace', index=False)
# sql中查询获取数据
pd.read_sql('select * from fund_jjjz limit 5',sqlite_con)
```



