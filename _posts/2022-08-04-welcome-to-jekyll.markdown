---
title:  "2022-08-04"
date:   2022-08-04 15:04:23
categories: [AI]
tags: [SW]
---

## Kaggle 스터디 진행

[쇼핑몰 지점별 매출액 예측 경진 대회 - DACON][dacon] 
대회 참가하여 EDA(데이터 분석) 및 모델 구축하였다.

아쉽게도 100위권 안으로 들지는 못했다...ㅠ 하루 최대 3개 제출 제한 때문에 새로운 모델 구축하기가 어려웠다.
그래도 어느정도 성공은 했다. 여태 Kaggle 스터디 때 배웠던 데이터 분석과 GridSearch를 적용해봤기 때문이다.

### 데이터 분석

``` python
import seaborn as sns
import numpy as np
import pandas as pd
import matplotlib
import matplotlib.pyplot as plt
matplotlib.rcParams['font.family'] = 'Malgun Gothic' # Windows
matplotlib.rcParams['font.size'] = 15 # 글자 크기
matplotlib.rcParams['axes.unicode_minus'] = False # 한글 폰트 사용시, 마이너스 글자가 깨지는 현상을 해결
train = pd.read_csv('./dataset/dataset/train.csv')
test = pd.read_csv('./dataset/dataset/test.csv')
sample_submission = pd.read_csv('./dataset/dataset/sample_submission.csv')
```

데이터 분석을 위한 plot을 진행해보았다.

``` python
import matplotlib.pyplot as plt

# 이번엔 예측하고자 하는 값인 지점별 Weekly_Sales를 확인해봅니다.
plt.hist(train[train.Store==1].Weekly_Sales, bins=50)
plt.hist(train[train.Store==2].Weekly_Sales, bins=50)
plt.hist(train[train.Store==3].Weekly_Sales, bins=50)
plt.legend(['Store 1','Store 2','Store 3'])
plt.show()
```

특히 상관계수를 scatter plot으로 확인해보았고, box plot으로 데이터 비율이 어느정도 되는지 확인해 보았다.
![image.png](../../images/box1.png)

지점별로 판매량이 매우 다르다는 것을 확인할 수 있었다.

``` python
plt.plot(year2010_train['Weekly_Sales'])
plt.title("2010년도")
plt.ylabel("Weekly_Sales")
ax = plt.gca()
ax.get_xaxis().set_visible(False)
plt.show()
```

지점별로 데이터 분석을 진행해보았다.

``` python
plt.figure(figsize=(15,10))
plt.plot(store_1['Date'],store_1['Weekly_Sales'])
plt.title("지점1")
plt.ylabel("Weekly_Sales")
plt.xticks(rotation=90,size=5)
# ax = plt.gca()
# ax.get_xaxis().set_visible(False)
plt.show()
```

def를 통하여 해당하는 데이터 분석 그래프를 만들었다.
Weekly_Sales, Promotion, Temperature, Unemployment 등을 년도, 월별로 boxplot해보았다.

Weekly_Sales
``` python
def mon(data, i):
    data2 = data[data['Year']==2010]
    data3 = data[data['Year']==2011]
    data4 = data[data['Year']==2012]
    
    fig,axs = plt.subplots(3,1,figsize=(10,12))
    fig.suptitle("월별 박스 그래프")
    g = sns.boxplot(x='Month', y='Weekly_Sales', data=data2, showfliers=False, ax=axs[0])
    axs[0].set_title('지점 {} 2010 월별 박스 그래프'.format(i), size = 10)

    g = sns.boxplot(x='Month', y='Weekly_Sales', data=data3, showfliers=False, ax=axs[1])
    axs[1].set_title('지점 {} 2011 월별 박스 그래프'.format(i), size = 10)
    
    g = sns.boxplot(x='Month', y='Weekly_Sales', data=data4, showfliers=False, ax=axs[2])
    axs[2].set_title('지점 {} 2012 월별 박스 그래프'.format(i), size = 10)
    
    plt.tight_layout()
    plt.show()
```

온도 
``` python
def tem_mon(data, i):
    data2 = data[data['Year']==2010]
    data3 = data[data['Year']==2011]
    data4 = data[data['Year']==2012]
    
    fig,axs = plt.subplots(3,1,figsize=(10,12))
    fig.suptitle("월별 온도 박스 그래프")
    g = sns.boxplot(x='Month', y='Temperature', data=data2, showfliers=False, ax=axs[0])
    axs[0].set_title('지점 {} 2010 월별 박스 그래프'.format(i), size = 10)

    g = sns.boxplot(x='Month', y='Temperature', data=data3, showfliers=False, ax=axs[1])
    axs[1].set_title('지점 {} 2011 월별 박스 그래프'.format(i), size = 10)
    
    g = sns.boxplot(x='Month', y='Temperature', data=data4, showfliers=False, ax=axs[2])
    axs[2].set_title('지점 {} 2012 월별 박스 그래프'.format(i), size = 10)
    
    plt.tight_layout()
    plt.show()
```

연료비
``` python
def fuel_mon(data, i):
    data2 = data[data['Year']==2010]
    data3 = data[data['Year']==2011]
    data4 = data[data['Year']==2012]
    
    fig,axs = plt.subplots(3,1,figsize=(10,12))
    fig.suptitle("월별 연료비 박스 그래프")
    g = sns.boxplot(x='Month', y='Fuel_Price', data=data2, showfliers=False, ax=axs[0])
    axs[0].set_title('지점 {} 2010 월별 박스 그래프'.format(i), size = 10)

    g = sns.boxplot(x='Month', y='Fuel_Price', data=data3, showfliers=False, ax=axs[1])
    axs[1].set_title('지점 {} 2011 월별 박스 그래프'.format(i), size = 10)
    
    g = sns.boxplot(x='Month', y='Fuel_Price', data=data4, showfliers=False, ax=axs[2])
    axs[2].set_title('지점 {} 2012 월별 박스 그래프'.format(i), size = 10)
    
    plt.tight_layout()
    plt.show()
```

실업률
``` python
def unem_mon(data, i):
    data2 = data[data['Year']==2010]
    data3 = data[data['Year']==2011]
    data4 = data[data['Year']==2012]
    
    fig,axs = plt.subplots(3,1,figsize=(10,12))
    fig.suptitle("월별 실업률 그래프")
    
    axs[0].plot(data2['Month'],data2['Unemployment'])
#     g = sns.boxplot(x='Month', y='Unemployment', data=data2, showfliers=False, ax=axs[0])
    axs[0].set_title('지점 {} 2010 월별 그래프'.format(i), size = 10)

    axs[1].plot(data3['Month'],data3['Unemployment'])
#     g = sns.boxplot(x='Month', y='Unemployment', data=data3, showfliers=False, ax=axs[1])
    axs[1].set_title('지점 {} 2011 월별 그래프'.format(i), size = 10)
    
    axs[2].plot(data4['Month'],data4['Unemployment'])
#     g = sns.boxplot(x='Month', y='Unemployment', data=data4, showfliers=False, ax=axs[2])
    axs[2].set_title('지점 {} 2012 월별 그래프'.format(i), size = 10)
    
    plt.tight_layout()
    plt.show()
```

휴일
``` python
def holi_mon(data, i):
    data2 = data[data['Year']==2010]
    data3 = data[data['Year']==2011]
    data4 = data[data['Year']==2012]
    
    fig,axs = plt.subplots(3,1,figsize=(10,12))
    fig.suptitle("월별 휴일 박스 그래프")
    
    g = sns.boxplot(x='NumberHoliday', y='Weekly_Sales', data=data2, showfliers=False, ax=axs[0])
    axs[0].set_title('지점 {} 2010 월별 그래프'.format(i), size = 10)


    g = sns.boxplot(x='NumberHoliday', y='Weekly_Sales', data=data3, showfliers=False, ax=axs[1])
    axs[1].set_title('지점 {} 2011 월별 그래프'.format(i), size = 10)
    
    
    g = sns.boxplot(x='NumberHoliday', y='Weekly_Sales', data=data4, showfliers=False, ax=axs[2])
    axs[2].set_title('지점 {} 2012 월별 그래프'.format(i), size = 10)
    
    plt.tight_layout()
    plt.show()
```

휴일 갯수
``` python
def holi_counts(data,i):
    data2 = data[data['Month']==i]['NumberHoliday'].value_counts()
    if len(data2) == 2:
        return data2.iloc[1]
    else :
        return 0

def holi2_mon(data, i):
    data2 = data[data['Year']==2010]
    data3 = data[data['Year']==2011]
    data4 = data[data['Year']==2012]
    
    data2_2 = []
    data3_2 = []
    data4_2 = []
    for j in (range(1,13)):
        data2_2.append(holi_counts(data2,j))
        data3_2.append(holi_counts(data2,j))
        data4_2.append(holi_counts(data2,j))
    
    fig,axs = plt.subplots(3,1,figsize=(10,12))
    fig.suptitle("월별 휴일 개수 그래프")
    
    month = [1,2,3,4,5,6,7,8,9,10,11,12]
    axs[0].plot(month,data2_2)
    axs[0].set_title('지점 {} 2010 월별 그래프'.format(i), size = 10)

    axs[1].plot(month,data3_2)
    axs[1].set_title('지점 {} 2011 월별 그래프'.format(i), size = 10)
    
    axs[2].plot(month,data4_2)
    axs[2].set_title('지점 {} 2012 월별 그래프'.format(i), size = 10)
    
    plt.tight_layout()
    plt.show()
```

Promotion
``` python
def Promotion_mon(data, i, j):
    data2 = data[data['Year']==2010]
    data3 = data[data['Year']==2011]
    data4 = data[data['Year']==2012]
    
    fig,axs = plt.subplots(3,1,figsize=(10,12))
    fig.suptitle("월별 Promotion{} 박스 그래프".format(j))
    g = sns.boxplot(x='Month', y='Promotion{}'.format(j), data=data2, showfliers=False, ax=axs[0])
    axs[0].set_title('지점 {} 2010 월별 박스 그래프'.format(i), size = 10)

    g = sns.boxplot(x='Month', y='Promotion{}'.format(j), data=data3, showfliers=False, ax=axs[1])
    axs[1].set_title('지점 {} 2011 월별 박스 그래프'.format(i), size = 10)
    
    g = sns.boxplot(x='Month', y='Promotion{}'.format(j), data=data4, showfliers=False, ax=axs[2])
    axs[2].set_title('지점 {} 2012 월별 박스 그래프'.format(i), size = 10)
    
    plt.tight_layout()
    plt.show()
```

이와 같이 def로 box plot을 진행하였고 년도별, 월별로 Weekly_Sales가 크게 차이 난다는 것을 확인할 수 있었다.

### 모델 학습
비선형적인 데이터 분포를 띠고 있어 선형회귀가 아닌 비선형 회귀 즉 결정트리 (Random Forest, XGBoost)를 적용해보았다.


랜덤포레스트
``` python
import time
from sklearn.ensemble import RandomForestRegressor
start = time.time() # 시작 시간 저장

# 랜덤 포레스트의 parameter 범위를 정의한다.
RF_params = {
    'n_estimators' : [50,100,150,200,300,500,1000],
    'max_features' : ['auto','sqrt'],
    'max_depth' : [8,10,12,14,16],
    'min_samples_leaf' : [1,2,4,8],
    'min_samples_split' : [2,3,5,10]}

# GridSearchCV를 이용하여 dict에Randomforest 모델을 저장한다. 
RF_models = {
    'RF': GridSearchCV(
    RandomForestRegressor(random_state=42), param_grid=RF_params, n_jobs=-1
    ).fit(train_input, train_target).best_estimator_}

print(f'걸린시간 : {np.round(time.time()-start, 3)}초') # 현재시간 - 시작시간(단위 초)
```

XGBoost
``` python
import xgboost as xgb
start = time.time() # 시작 시간 저장

# xgboost parameter space를 정의한다.
XGB_params = {
    'min_child_weight' : [1,3,5,10],
    'gamma' : [0.3,0.5,1,1.5,2.5],
    'subsample' : [0.6,0.8,1.0,1.2],
    'colsample_bytree' : [0.6,0.8,1.0],
    'max_depth' : [3,4,5,7,10]}


# GridSearchCV를 통해 parameter를 탐색하게 정의한다.
XGB_gridsearch = GridSearchCV(xgb.XGBRegressor(random_state=42),
                                 param_grid=XGB_params, n_jobs=-1)

# 모델 학습
XGB_gridsearch.fit(train_input, train_target)

print(f'걸린시간 : {np.round(time.time()-start,3)}초') # 현재시간 - 시작시간(단위 초)
```


**좋았던 점**

- 데이터 분석을 통하여 비선형적 데이터 분포를 띠고 있음을 확인하여 알고리즘을 RF, XGBoost으로 적용해보았던 것이 인상깊었다.
- 가장 영향을 끼치는 데이터가 무엇인지 box plot을 통하여 확인할 수 있었다.

**아쉬웠던 점**

- 데이터 전처리를 더 해서 예측값을 더 정확하게 할 수 있었을텐데 아쉬웠다. ㅠㅠ
- 1일 최대 제출 제한이 3개라는 것이 너무 아쉬웠다. 계속 시도해보면 좋았을 텐데...

다음부터는 모델 구축에 좀더 투자해야할 것 같다!!

[dacon]: https://dacon.io/competitions/official/235942/overview/description