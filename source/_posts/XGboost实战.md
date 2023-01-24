---
title: XGboost实战
date: 2022-12-24 14:42:18
tags: [实验]
excerpt: 使用XGboost模型预测
categories: 课程相关
index_img: /img/index_img/4.png
banner_img: /img/banner_img/background2.jpg
---

## 奖学金预测

```python

import pandas as pd
import xgboost as xgb
import numpy as np
from sklearn.model_selection import StratifiedKFold
import warnings
warnings.filterwarnings('ignore')
from sklearn.model_selection import train_test_split
from sklearn.model_selection import cross_val_score

def changev(x):
    if x == '学三':
        return 1
    elif x == '学二':
        return 2
    elif x == '学一':
        return 3
    elif x == '国励':
        return 4
    elif x == '国奖':
        return 5
    else:
        return 0

data_path='./data2.xlsx'
data = pd.read_excel(data_path)
data['性别'] = data['性别'].apply(lambda x:1 if x == '男'  else 0)
data['奖学金'] = data['奖学金'].apply(changev)
data.head()
print(data.columns)
X=data.iloc[:,1:25]
Y=data.iloc[:,25]

Y.head()

## 交叉验证
X_train, X_test, y_train, y_test = train_test_split(X, Y, test_size=0.25, random_state=100)
x_train, x_test, y_train, y_test = X, X, Y, Y
xgb_train=xgb.DMatrix(X_train,label=y_train)
xgb_test=xgb.DMatrix(X_test,label=y_test)


import sklearn.metrics as metrics
def f1(preds, dtrain):
    y_train = dtrain.get_label() # 'numpy.ndarray'
    y_pred = [np.argmax(d) for d in preds]
    return 'f1', metrics.f1_score(y_train, y_pred, average='weighted')


# params={
#     'objective':'multi:softmax',
#     'eta':0.1,
#     'max_depth':7,
#     # 'n_estimators':500,
#     'num_class':6,
#     'alpha': 0.6,
#     'lambda': 1.4,
#     'colsample_bytree':0.97,
#     'min_child_weight':0.5,
#     'gamma':0.015,
# }
from xgboost import XGBClassifier
bst = XGBClassifier(
    max_depth=7,
    min_child_weight=4,
    gamma= 0,
    subsample=0.8,
    colsample_bytree=0.8,
    # scale_pos_weight=1,
    learning_rate=0.1, 
    n_estimators=500,
    seed=1)
# watchlist=[(xgb_train,'train'),(xgb_test,'test')]
# num_round=100
# grid = GridSearchCV(bst, param_grid=params, cv=10, scoring='f1_weighted')
# grid.fit(x_train, y_train)

# print(grid.best_params_)
# print(grid.best_score_)
# bst=xgb.train(params,xgb_train,num_round,watchlist,feval=f1)
kfold = StratifiedKFold(n_splits=10, random_state=7,shuffle=True)
results = cross_val_score(bst, X_train, y_train, cv=kfold,scoring='f1_weighted')#对数据进行十折交叉验证--9份训练，一份测试
print(results)
print(results.mean())
```


## kpi异常检测

```python
import numpy as np
import pandas as pd
from  datetime import datetime
import matplotlib.pyplot as plt
from sklearn.preprocessing import LabelEncoder
%config InlineBackend.figure_format = 'svg'
import warnings
warnings.filterwarnings("ignore")

data = pd.read_csv("./train.csv")

data.info()

data["timestamp"] = data["timestamp"].apply(lambda x: datetime.fromtimestamp(x))
data["timestamp"] = pd.to_datetime(data["timestamp"])
le=LabelEncoder()
data['kpi_id']=le.fit_transform(data['KPI ID'])

data.head()


data.sort_values(['KPI ID','timestamp'],inplace=True)
groups = data.groupby('KPI ID')




from collections import Counter
from sklearn.model_selection import train_test_split

# x_train, x_test, y_train, y_test = None, None, None, None
data = groups.head(200000)
# for name,group in groups:
#     df = group.head(10000).reset_index(drop=True)
#     x = df.drop('label', axis=1)
#     y = df['label']
#     x_train_t, x_test_t, y_train_t, y_test_t = train_test_split(x, y, test_size=0.2, shuffle=False, random_state=40)
#     x_train, x_test, y_train, y_test = pd.concat([x_train_t, x_train]), pd.concat([x_test_t, x_test]),pd.concat([y_train_t, y_train]),pd.concat([y_test_t, y_test])


data.head()

data['kpi_id'].value_counts().plot.bar()


data.sort_values(['KPI ID','timestamp'],inplace=True)
data['weekday']=data['timestamp'].dt.weekday
data['hour']=data['timestamp'].dt.hour
data['weekend']=(data['weekday']>5).astype(int)
data['daylight']=((data['hour'] >= 20) | (data['hour'] <= 7)).astype(int)
data['value']=pd.to_numeric(data['value'],errors='coerce')
data['value']=data['value'].fillna(method='bfill')


import math
data.sort_values(['KPI ID','timestamp'],inplace=True)
data["value"] = data["value"].apply(lambda x:  math.log(x + 10))
data['value'].plot(kind='kde')

#分组滑动窗口'
df = data
cols_win = ['value']
for col in cols_win:
    df['win_2_mean_{}'.format(col)]= df.groupby("kpi_id")[col].rolling(window=2,min_periods=1).mean().reset_index(0,drop=True)
    df['win_3_mean_{}'.format(col)]= df.groupby("kpi_id")[col].rolling(window=3,min_periods=2).mean().reset_index(0,drop=True)

    
    df['win_24_mean_{}'.format(col)]= df.groupby("kpi_id")[col].rolling(window=24,min_periods=6).mean().reset_index(0,drop=True)
    df['win_24_std_{}'.format(col)]= df.groupby("kpi_id")[col].rolling(window=24,min_periods=6).std().reset_index(0,drop=True)
    df['win_24_skew_{}'.format(col)]= df.groupby("kpi_id")[col].rolling(window=24,min_periods=6).skew().reset_index(0,drop=True)
    df['win_24_kurt_{}'.format(col)]= df.groupby("kpi_id")[col].rolling(window=24,min_periods=6).kurt().reset_index(0,drop=True)
    df['win_24_q25_{}'.format(col)]= df.groupby("kpi_id")[col].rolling(window=24,min_periods=6).quantile(0.25).reset_index(0,drop=True)
    df['win_24_q50_{}'.format(col)]= df.groupby("kpi_id")[col].rolling(window=24,min_periods=6).quantile(0.50).reset_index(0,drop=True)
    df['win_24_q75_{}'.format(col)]= df.groupby("kpi_id")[col].rolling(window=24,min_periods=6).quantile(0.75).reset_index(0,drop=True)
    
    df['win_168_mean_{}'.format(col)]= df.groupby("kpi_id")[col].rolling(window=168,min_periods=84).mean().reset_index(0,drop=True)
    df['win_168_std_{}'.format(col)]= df.groupby("kpi_id")[col].rolling(window=168,min_periods=84).std().reset_index(0,drop=True)


cols_shift=['value','win_2_mean_value','win_3_mean_value']
#计算差分
for col in cols_shift:
    for i in [1,24,168]:
        cc="shift_{}_{}".format(col,i)
        df[cc] = df.groupby('kpi_id')[col].shift(i)
        df['x_y_{}_{}'.format(col,i)]=np.abs(df[col]-df[cc])
        df['xy_{}_{}'.format(col,i)]=df[col]/df[cc]
        df.drop(cc,axis=1,inplace=True)

df['expa_mean']=df.groupby(["kpi_id"])['value'].expanding().mean().reset_index(drop=True)
df['expa_std']=df.groupby(["kpi_id"])['value'].expanding().std().reset_index(drop=True)
df['win_3_mean']= df.groupby("kpi_id")['value'].rolling(window=3,min_periods=3).mean().reset_index(0,drop=True)
df['trend_win3']=df['value']-df['win_3_mean']
df['z_score']=(df['win_3_mean']-(df.groupby("kpi_id")['value'].transform('mean')))/df.groupby("kpi_id")['value'].transform('std')
df['3_24_score'] = (df['win_3_mean'] - df['win_24_mean_value']) / df["win_24_std_value"]
df['24_168_score'] = (df['win_24_mean_value'] - df['win_168_mean_value']) / df["win_168_std_value"]


#删除较多的缺失值行
df=df[df['x_y_value_168'].notnull()]
train = df[df['label'].notnull()]
train.fillna(0,inplace=True)

train.head()


train = train.drop("timestamp", axis=1)
train = train.drop("KPI ID", axis=1)
train.head()

train = train.drop('weekend', axis=1)
train = train.drop('expa_mean', axis=1)
train = train.drop('expa_std', axis=1)
train = train.drop('trend_win3', axis=1)
train = train.drop('daylight', axis=1)
train = train.drop('win_3_mean', axis=1)
# train = train.drop('xy_win_2_mean_value_1', axis=1)
# train = train.drop('xy_value_1', axis=1)
# train = train.drop('x_y_value_24', axis=1)

# 分割数据集
groups = train.groupby('kpi_id')
x_train, x_test, y_train, y_test = None, None, None, None
for name,group in groups:
    df = group.reset_index(drop=True)
    x = df.drop('label', axis=1)
    y = df['label']
    x_train_t, x_test_t, y_train_t, y_test_t = train_test_split(x, y, test_size=0.2, shuffle=False, random_state=40)
    x_train, x_test, y_train, y_test = pd.concat([x_train_t, x_train]), pd.concat([x_test_t, x_test]),pd.concat([y_train_t, y_train]),pd.concat([y_test_t, y_test])

x_train.head()


x_test.head()


from imblearn.over_sampling import SMOTE
smo = SMOTE(random_state=42)
 
train_y = np.nan_to_num(y_train)
train_x = np.nan_to_num(x_train)
X_smo, y_smo = smo.fit_resample(train_x, train_y)


y_train.value_counts().plot.bar()

X_smo = pd.DataFrame(X_smo)
y_smo = pd.Series(y_smo)

X_smo.columns = x_train.columns

train_x=X_smo.reset_index(drop=True)
train_y=y_smo.reset_index(drop=True)

y_smo.value_counts().plot.bar()

X_smo.head()

from sklearn import preprocessing
from sklearn.metrics import confusion_matrix, roc_auc_score
from sklearn.model_selection import StratifiedKFold, cross_val_score, KFold,TimeSeriesSplit
from xgboost import XGBClassifier
import xgboost as xgb
from xgboost import plot_importance
from sklearn.metrics import make_scorer


from sklearn.metrics import f1_score
xgb_parrams={
    'booster':'gbtree',
    'objective':'binary:logistic',
    'eval_metric':'auc',
     'colsample_bytree': 0.30801698649621306,
     'gamma': 0.47969503858326235,
     'learning_rate': 0.04341140878653162,
     'max_depth': 10,
     'reg_alpha': 0.28161740307145836,
     'reg_lambda': 0.2109582304218081,
     'subsample': 0.9
            }
def f1_score_vail(pred, data_vail):
    labels = data_vail.get_label()
    pred = pred > 0.51
    score_vail = f1_score(y_true=labels, y_pred=pred, average='macro')      # xgb的predict输出即为对应的label
    return '1-f1_score', 1-score_vail   # xgb目标是将目标指标降低

X_tr, X_vl = train_x, x_test
y_tr, y_vl = train_y, y_test
train_data = xgb.DMatrix(X_tr, y_tr)
validation_data = xgb.DMatrix(X_vl, y_vl)
watchlist = [(train_data, 'train_F1'),(validation_data, 'test_F1')]

evals_result = {}
xgb_clf = xgb.train(
                xgb_parrams,
                train_data,
                num_boost_round=600, 
                early_stopping_rounds=200,
                feval=f1_score_vail, 
                evals_result=evals_result,
                evals=watchlist, 
                verbose_eval=10,
                )
X_valid = xgb.DMatrix(X_vl)  # 转为xgb需要的格式
plt.rcParams["figure.figsize"] = (20, 10)
xgb.plot_importance(xgb_clf)

test_AUC=list(evals_result['test_F1'].values())[0]
test_F1=list(evals_result['test_F1'].values())[1]
x_scale=[i for i in range(len(test_F1))]
plt.figure(figsize=(6,4))
plt.title('F1')
plt.plot(x_scale,test_AUC,label='AUC',color='r')
plt.legend()
plt.show()
plt.figure(figsize=(6,4))
plt.plot(x_scale,test_F1,label='1-F1',color='b')
plt.legend()
plt.show()
```