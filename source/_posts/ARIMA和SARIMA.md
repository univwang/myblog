---
title: ARIMA和SARIMA
date: 2022-10-12 11:42:18
tags: [python,机器学习]
excerpt: 时间序列预测常用模型
categories: 机器学习
index_img: /img/index_img/6.png
banner_img: /img/banner_img/background6.jpg
---



## ARMA和SARIMA

- AR：  自回归。一种模型，它使用观察值和一些滞后观察值之间的依赖关系。
- I：  综合。为了使时间序列平稳，使用原始观测值的差异（例如，从上一个时间步长的观测值中减去观测值）。
- MA：  移动平均。一种模型，该模型使用观察值与应用于滞后观察值的移动平均模型的残差之间的依赖关系。

SARIMA（Seasonal ARIMA）：ARIMA的扩展版本，可以支持带有季节性成分的时间序列数据。在ARIMA(p,d,q)基础上又增加了3个超参数(P,D,Q)，以及一个额外的季节性周期参数 s。

SARIMA(p,d,q)(P,D,Q,s)总共7个参数，可以分成2类，3个非季节参数(p,d,q)，和4个季节参数(P,D,Q,s)。


### 非季节参数

![](https://raw.githubusercontent.com/univwang/img/main/20221012113450.png)

### 季节参数

![](https://raw.githubusercontent.com/univwang/img/main/20221012113524.png)



## 参数选择

`(p,d,q) x (P,D,Q,S)`

其中d一般为1，S为数据的周期。

通过看AFC和PAFC判断p、q。
```
fig = plt.figure(figsize=(12,8))
ax1 = fig.add_subplot(211)
fig = sm.graphics.tsa.plot_acf(monthly_train['Temp'].iloc[13:],lags=40,ax=ax1)
ax2 = fig.add_subplot(212)
fig = sm.graphics.tsa.plot_pacf(monthly_train['Temp'].iloc[13:],lags=40,ax=ax2)

```
![](https://raw.githubusercontent.com/univwang/img/main/20221012225229.png)

### 使用auto_arima

```python
def auto_parameters(data, s_num):

    # s_sum是传入的季节性周期参数
    # 主要调整auto_arima中的pqd参数
    
    kpss_diff = arima.ndiffs(data, alpha=0.05, test='kpss', max_d=s_num)
    adf_diff = arima.ndiffs(data, alpha=0.05, test='adf', max_d=s_num)
    d = max(kpss_diff, adf_diff)
    D = arima.nsdiffs(data, s_num)

    stepwise_model = auto_arima(data, start_p=0, start_q=0,
                                max_p=4, max_q=2, max_d=2, m=s_num,
                                seasonal=True, d=d, D=D, trace=True,
                                error_action='ignore',
                                suppress_warnings=True,
                                stepwise=True)
    print("AIC: ", stepwise_model.aic())
    print(stepwise_model.order)		# (p,d,q)
    print(stepwise_model.seasonal_order)	# (P,D,Q,S)
    print(stepwise_model.summary())		# 详细模型
    return stepwise_model.order, stepwise_model.seasonal_order

```

## 判断模型的好坏

测试数据稳定性的方法
```python
def test_stationarity(timeseries):
  dftest = adfuller(timeseries, autolag='AIC')
  dfoutput = pd.Series(dftest[0:4], index=['Test Statistic','p-value','#Lags Used','Number of Observations Used'])

  for key,value in dftest[4].items():
    dfoutput['Critical Value (%s)'%key] = value
  return dfoutput['p-value']
# p-value < 0.05时较好，否则需要通过差分使数据稳定
```


```python
results.plot_diagnostics(figsize=(15, 12))
plt.show()
```
![](https://raw.githubusercontent.com/univwang/img/main/20221012113719.png)

我们的主要关切是确保我们的模型的残差是不相关的，并且平均分布为零。 如果季节性ARIMA模型不能满足这些特性，这是一个很好的迹象，可以进一步改善。

在这种情况下，我们的模型诊断表明，模型残差正常分布如下：

- 在右上图中，我们看到红色KDE线与N(0,1)行（其中N(0,1) ）是正态分布的标准符号，平均值0 ，标准偏差为1 ） 。 这是残留物正常分布的良好指示。

- 左下角的qq图显示，残差（蓝点）的有序分布遵循采用N(0, 1)的标准正态分布采样的线性趋势。 同样，这是残留物正常分布的强烈指示。

- 随着时间的推移（左上图）的残差不会显示任何明显的季节性，似乎是白噪声。 这通过右下角的自相关（即相关图）来证实，这表明时间序列残差与其本身的滞后版本具有低相关性。

这些观察结果使我们得出结论，我们的模型产生了令人满意的合适性，可以帮助我们了解我们的时间序列数据和预测未来价值。


经常地，对一堆数据进行建模的时候，特别是分类和回归模型，我们有很多的变量可供使用，选择不同的变量组合可以得到不同的模型，例如我们有5个变量，2的5次方，我们将有32个变量组合，可以训练出32个模型。但是哪个模型更加的好呢？目前常用有如下方法：
AIC=-2 ln(L) + 2 k 中文名字：赤池信息量 akaike information criterion
BIC=-2 ln(L) + ln(n)*k 中文名字：贝叶斯信息量 bayesian information criterion
HQ=-2 ln(L) + ln(ln(n))*k hannan-quinn criterion
L是在该模型下的最大似然，n是数据数量，k是模型的变量个数
三个模型A, B, C，在通过这些规则计算后，我们知道B模型是三个模型中最好的，但是不能保证B这个模型就能够很好地刻画数据，因为很有可能这三个模型都是非常糟糕的，B只是烂苹果中的相对好的苹果而已。



## 模型预测

```python
# 模型静态数据预测
predict = results.predict()
# 预测数据外的24个点
predict_forecast = results.forecast(24)

# 计算模型的误差
mse = mean_squared_error(predict,df[predict.index])
```