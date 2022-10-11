---
title: Pandas学习
date: 2022-10-11 20:42:18
tags: [python]
excerpt: 学习Pandas的基本操作
categories: python
index_img: /img/index_img/5.png
banner_img: /img/banner_img/background5.jpg
---


## 数据读取

```python
train_chunks = pd.read_csv(csv_path, encoding='gbk', iterator=True, chunksize=65535)
# 读取csv文件，其中iterator=True, 是为了读取大文件，chunksize=65535指定每次读取65535

list1 = list()
for it in train_chunks:
    list1.append(it)

df = pd.concat(list1, axis=0, ignore_index=False)
```

![](https://raw.githubusercontent.com/univwang/img/main/20221011214406.png)


## 数据筛选

### 数据筛选

```python
# 通过日期列筛选数据
df1_ = df1_[(df1_['日期'] >= '2018/3/1') & (df1_['日期'] <= '2018/3/31')]
```

### 数据分组和过滤

```python
# 按小区编号分类，该类别数据长度大于1
df = df.groupby('小区编号').filter(lambda x: len(x) > 1)
# 找到所有小区编号为1的数据
df = df.groupby('小区编号').get_group(1)

# 统计小区编号的种类
df['小区编号'].value_counts()
```

### 数据清洗

```python

# 删除重复的数据
df1 = df1.drop_duplicates(keep='first')
# 删除空值的数据
df1 = df1.dropna()

```

## plt画图

```python 
# 调用pd中的plot接口，指定x、y画图
df1.plot(x = 'data', y = '上行业务量GB')
plt.ylabel('up GB')
plt.xlabel('Date')
plt.show()
```

## pandas 时间戳

```python
# 将数据转化为date类型
pd.to_datetime("20110424 01:30:00.000", format='%Y%m%d %H:%M:%S.%f')

# 按照时间戳筛选数据
s_date = datetime.datetime.strptime('20050606', '%Y%m%d').date()
e_date = datetime.datetime.strptime('20071016', '%Y%m%d').date()
df = df[(df['tra_date'] >= s_date) & (df['tra_date'] <= e_date)]

```