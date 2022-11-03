---
title: Xgboost集成学习
date: 2022-11-01 21:35:18
tags: [机器学习]
excerpt: 集成学习的常用模型
categories: 机器学习
index_img: /img/index_img/4.png
banner_img: /img/banner_img/background4.jpg
---


## 简介

XGBoost（eXtreme Gradient Boosting）极致梯度提升，是一种基于GBDT的算法或者说工程实现。

XGBoost的基本思想和GBDT相同，但是做了一些优化，比如二阶导数使损失函数更精准；正则项避免树过拟合；Block存储可以并行计算等。

XGBoost具有高效、灵活和轻便的特点，在数据挖掘、推荐系统等领域得到广泛的应用。

`XGBoost可用于分类问题、回归问题

## XGBoost模型参数

### 不需要调优的参数


-booster:  [default=gbtree] 决定类型 ，gbtree : 树模型 gblinear :线性模型。
- objective:
    - binary:logistic 用来二分类
    - multi:softmax 用来多分类
    - reg:linear 用来回归任务
- silent [default=0]:
  - 设为1 则不打印执行信息
  - 设为0打印信息
- nthread ：控制线程数目

### 需要调优的参数

- max_depth: 决定树的最大深度，比较重要 常用值4-6 ，深度越深越容易过拟合。
- n_estimators: 构建多少颗数 ，树越多越容易过拟合。
- learning_rate/eta : 学习率 ，范围0-1 。默认值 为0.3 , 常用值0.01-0.2
- subsample: 每次迭代用多少数据集 。
- colsample_bytree: 每次用多少特征 。可以控制过拟合。
- min_child_weight :树的最小权重 ，越小越容易过拟合。
- gamma：如果分裂能够使loss函数减小的值大于gamma，则这个节点才分裂。gamma设置了这个减小的最低阈值。如果gamma设置为0，表示只要使得loss函数减少，就分裂。
- alpha：l1正则，默认为0
- lambda ：l2正则，默认为1

## 实战一：波士顿房价预测
```python
from  sklearn import datasets 
import pandas as pd 
import xgboost as xgb
import numpy as np
from sklearn.metrics import mean_squared_error
from sklearn.model_selection import train_test_split
import matplotlib.pyplot as plt


boston = datasets.load_boston()
data = pd.DataFrame(boston.data)
data.columns = boston.feature_names
data['price'] = boston.target


y = data.pop('price')

data_matrix = xgb.DMatrix(data,y)


xg_reg= xgb.XGBRegressor(
    objective='reg:linear',
    colsample_bytree=0.3,
    learning_rate=0.1,
    max_depth=5,
    n_estimators=10,
    alpha=10
)
x_train,x_test,y_train  ,y_test = train_test_split(data,y,test_size=0.2)
xg_reg.fit(x_train,y_train)
pred = xg_reg.predict(x_test)
mean_squared_error(pred,y_test)


params = {"objective":"reg:linear",'colsample_bytree': 0.3,'learning_rate': 0.1,
                'max_depth': 5, 'alpha': 10}
cv_results = xgb.cv(dtrain=data_matrix, params=params, nfold=3,
                    num_boost_round=50,early_stopping_rounds=10,metrics="rmse", as_pandas=True, seed=123)
cv_results.head()

#打印最后一次结果
print((cv_results["test-rmse-mean"]).tail(1))

#查看模型权重

xgb.plot_importance(xg_reg)
plt.rcParams['figure.figsize'] = [3, 3]
plt.show()
```
## 实战二：小麦类型预测

```python
import pandas as pd
import xgboost as xgb
import numpy as np
import warnings
warnings.filterwarnings('ignore')
from sklearn.model_selection import train_test_split

data_path='./seeds_dataset.txt'
data=pd.read_csv(data_path,header=None,sep='\s+',converters={7:lambda x:int(x)-1})
data.rename(columns={7:'lable'},inplace=True)
data.head()

X=data.iloc[:,:6]
Y=data.iloc[:,7]
X_train, X_test, y_train, y_test = train_test_split(X, Y, test_size=0.25, random_state=100)

xgb_train=xgb.DMatrix(X_train,label=y_train)
xgb_test=xgb.DMatrix(X_test,label=y_test)

params={
    'objective':'multi:softmax',
    'eta':0.1,
    'max_depth':7,
    'n_estimators':500,
    'num_class':3,
    'alpha': 0.6,
    'lambda': 1.4,
    'colsample_bytree':0.97,
    'min_child_weight':0.5,
    'gamma':0.015,
}

watchlist=[(xgb_train,'train'),(xgb_test,'test')]
# 设置训练轮次，这里设置60轮
num_round=100
bst=xgb.train(params,xgb_train,num_round,watchlist)

pred=bst.predict(xgb_test)
print(pred)

#模型评估

# error_rate=np.sum(pred!=test.lable)/test.lable.shape[0]
error_rate=np.sum(pred!=y_test)/y_test.shape[0]

print('测试集错误率(softmax):{}'.format(error_rate))

accuray=1-error_rate
print('测试集准确率：%.4f' %accuray)


import matplotlib.pyplot as plt
xgb.plot_importance(bst)
plt.rcParams['figure.figsize'] = [8, 4]
plt.show()
```