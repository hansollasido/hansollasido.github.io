---
title: "[Chap 2]"
excerpt: "머신러닝 프로젝트 처음부터 끝까지"

categories:
  - 핸즈온
tags:
  - [공부, 머신러닝]

use_math: true

permalink: /categories2/handson3/

toc: true
toc_sticky: true

date: 2023-02-17
last_modified_at: 2023-02-17
---

# 머신러닝 프로젝트 처음부터 끝까지

---

## 큰 그림 보기

### 문제 정의

**파이프라인**
- 머신러닝 시스템은 데이터를 조작하고 변환할 일이 많아 파이프라인을 사용하는 일이 매우 흔함

**레이블**, **다중 회귀** : 예측에 사용할 특성이 여러 개, **단변량 회귀** : 하나의 값을 예측, **다변량 회귀** : 여러 값을 예측

문제 확인 -> 데이터 분석 -> 어떤 알고리즘에 적합할 지 생각 -> 적용

---

### 성능 측정 지표 선택

회귀 문제의 전형적인 성능 지표는 **평균 제곱근 오차(RMSE)**

식 : $RMSE(X,h)=\sqrt{\frac{1}{m}\sum^m_{i=1}(h(x^{(i)})-y^{(i)})^2}$

- $m$ : RMSE를 측정할 데이터셋에 있는 샘플 수

- $x^{(i)}$는 i번째 샘플의 전체 특성값의 벡터

- $y^{(i)}$는 해당 레이블(해당 샘플의 기대 출력값)

- $X$는 데이터셋에 있는 모든 특성값을 포함하는 행렬

- $h$는 시스템의 예측 함수며 **가설**이라고 함, 기댓값 $\hat{y}^{(i)}=h(x^{(i)})$를 출력함

**평균 절대 오차(MAE)**

식 : $MAE(X,h)=\frac{1}{m}\sum^m_{i=1} \left \vert h(x^{(i)})-y^{(i)} \right \vert$

<font color="red">RMSE와 MAE 모두 예측값의 벡터와 타깃값의 벡터 사이의 거리를 재는 방법임, 즉 노름이 가능</font>

---

### 가정 검사

지금까지 만든 가정을 나열하고 검사함

---

**데이터 확인 부분은 skip함 (너무 기본적인 내용임)**

---

## 데이터 이해를 위한 탐색과 시각화

시각화하여 데이터 탐색을 하는 것이 좋음

### 지리적 데이터 시각화

```python
# alpha는 밀집된 지역을 잘 보여줌
housing.plot(kind="scatter",x="longitude",y="latitude", alpha=0.1)
```
- 위도와 경도에 따라 데이터가 어떻게 밀집되었는지 볼 수 있음

```python
# s는 원의 반지름, c는 색상, cmap의 jet은 파란색부터 빨간색까지의 범위
# sharex는 x축을 공유하도록 설정함
housing.plot(kind="scatter", x="longitude",y="latitude",alpha=0.4,
  s=housing["population"]/100, label="population", figsize=(10, 7),
  c="median_house_value", cmap=plt.get_cmap("jet"), colorbar=True,
  sharex=False)
plt.legend()
```

<img src="../../assets/images/022301.png" width="500px" height="500px" title="OP code 예시" alt="OP code"><img><br/>

- 해안가에는 인구 밀도가 높고 이는 주택가격으로 연결됨을 볼 수 있음

### 상관관계 조사

`corr()` 메서드를 사용하여 상관계수를 쉽게 계산 가능
- 양수면 양의 상관관계
- 음수면 음의 상관관계
- 계수가 0이면 선형적인 상관관계가 없다는 뜻임

### 특성 조합으로 실험

정제해야할 이상한 데이터를 확인했고 이런 데이터를 변형해야함
- 특성 조합을 실행해 보는 것임
- 특성 조합을 실행하고 상관관계를 확인해보면 선형적인 관계를 볼 수도 있을 것임

## 머신러닝 알고리즘을 위한 데이터 준비

### 데이터 정제

<mark style="background-color: #fff5b1">결측치 처리가 필요함</mark>

1. 해당 구역을 제거
2. 전체 특성을 삭제
3. 어떤 값으로 채움(0, 평균, 중간값)

```python
# fillna로 결측치 넣기
housing["total_bedrooms"].fillna(median, inplace=True)
```

```python
# SimpleImputer로 손쉽게 결측치를 중간값으로 대체하기
from sklearn.impute import SimpleImputer

imputer = SimpleImputer(strategy="median")
housing_num = house.drop("ocean_proximity", axis=1)
X = imputer.transform(housing_num)
```

### 텍스트와 범주형 특성 다루기

텍스트형 데이터가 있다면 Encoder로 숫자형으로 변환함
```python
from sklearn.preprocessing import OrdinalEncoder

ordinal_encoder = OrdinalEncoder()
housing_cat_encoded = ordinal_encoder.fit_transform(housing_cat)
housing_cat_encoded[:10]
```
```python
# 변환된 리스트를 확인할 수 있음
ordinal_encoder.categories_
```
원 핫 인코딩으로 만들 수 있음
```python
from sklearn.preprocessing import OneHotEncoder
cat_encoder = OneHotEncoder()
housing_cat_1hot = cat_encoder.fit_transform(housing_cat)
housing_cat_1hot
```
```python
# 원핫 인코딩 데이터 확인 가능
housing_cat_1hot.toarray()
# 카테고리 리스트 읽을 수 있음
cat_encoder.categories_
```

### 나만의 변환기

사이킷런은 나만의 변환기와 부드럽게 연계 가능

```python
from sklearn.base import BaseEstimator, TransformerMixin

rooms_ix, bedrooms_ix, population_ix, households_ix = 3, 4, 5, 6

class CombinedAttributesAdder(BaseEstimator, TransformerMixin):
    def __init__(self, add_bedrooms_per_room = True): # *arg나 **kargs가 아님
        self.add_bedrooms_per_room = add_bedrooms_per_room 
    def fit(self, X, y=None):
        return self # 더 할 일이 없음
    def transform(self, X):
        rooms_per_household = X[:, rooms_ix] / X[:, households_ix]
        population_per_household = X[:, population_ix] / X[:, households_ix]
        if self.add_bedrooms_per_room:
            bedrooms_per_room = X[:,bedrooms_ix] / X[:,rooms_ix]
            return np.c_[X, rooms_per_household, population_per_household, bedrooms_per_room]
        
        else :
            return np.c_[X, rooms_per_household, population_per_household]
        
attr_adder = CombinedAttributesAdder(add_bedrooms_per_room=False)
housing_extra_attribs = attr_adder.transform(housing.values)
```

### 특성 스케일링

**특성 스케일링**으로 데이터 범위를 같도록 만들어줘서 학습이 잘 되게 만듦

1. **min-max 스케일링**
- 0 ~ 1 범위에 들도록 값을 이동하고 스케일을 조정함
- 데이터에서 최솟값을 뺀 후 최댓값과 최솟값의 차이로 나눔
- `MinMaxScaler`

2. **표준화**
- 평균을 뺀 후 표준편차로 나누어 결과 분포의 분산이 1이 되도록 만듦
- 상한과 하한이 없어 문제가 될 수 있으나 이상치에는 영향을 덜 받음
- `StandardScaler`

### 변환 파이프라인
변환 단계가 많아 정확한 순서대로 실행되어야 하는데 이에 순서대로 처리할 수 있도록 도와주는 Pipeline 클래스가 있음

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler

num_pipeline = Pipeline([
    ('imputer', SimpleImputer(strategy='median')),
    ('attribs_adder',CombinedAttributesAdder()),
    ('std_scaler',StandardScaler())
])

housing_num_tr = num_pipeline.fit_transform(housing_num)
```

각 열마다 적절한 변환을 적용할 수 있도록 처리할 수 있는 ColumnTransformer가 있음

```python
from sklearn.compose import ColumnTransformer

num_attribs = list(housing_num)
cat_attribs = ['ocean_proximity']

full_pipeline = ColumnTransformer([
    ('num', num_pipeline, num_attribs),
    ('cat', OneHotEncoder(), cat_attribs),
])

housing_prepared = full_pipeline.fit_transform(housing)
```

## 모델 선택과 훈련

### 훈련 세트에서 훈련하고 평가하기

선형 회귀 만듦

```python
from sklearn.linear_model import LinearRegression

lin_reg = LinearRegression()
lin_reg.fit(housing_prepared, housing_labels)
```

RMSE로 측정

```python
from sklearn.metrics import mean_squared_error
housing_predictions = lin_reg.predict(housing_prepared)
lin_mse = mean_squared_error(housing_labels,housing_predictions)
lin_rmse = np.sqrt(lin_mse)
lin_rmse
```

비선형 회귀(결정 트리)

```python
from sklearn.tree import DecisionTreeRegressor

tree_reg = DecisionTreeRegressor()
tree_reg.fit(housing_prepared, housing_labels)
```

RMSE로 측정
```python
housing_predictions = tree_reg.predict(housing_prepared)
tree_mse = mean_squared_error(housing_labels, housing_predictions)
tree_rmse = np.sqrt(tree_mse)
tree_rmse
```

### 교차 검증을 사용한 평가

**k-겹 교차 검증**
- 훈련세트를 **폴드(fold)**라 불리는 세브셋으로 무작위 분할함
- 매번 다른 폴드를 선택해 평가에 사용하고 나머지는 훈련에 사용함

```python
from sklearn.model_selection import cross_val_score
scores = cross_val_score(tree_reg, housing_prepared, housing_labels,
                        scoring='neg_mean_squared_error', cv=10)
tree_rmse_scores = np.sqrt(-scores)
```

랜덤포레스트 사용 
- 무작위로 특성을 선택해서 많은 결정 트리를 만들고 그 예측을 평균 내는 방식으로 작동함
- 다른 모델을 모아서 하나의 모델을 만드는 **앙상블 학습**임

```python
from sklearn.ensemble import RandomForestRegressor
forest_reg = RandomForestRegressor()
forest_reg.fit(housing_prepared, housing_labels)
```

랜덤포레스트 평가

```python
forest_scores = cross_val_score(forest_reg, housing_prepared, housing_labels,
                            scoring='neg_mean_squared_error',cv=10)
forest_rmse_scores = np.sqrt(-forest_scores)
display_scores(forest_rmse_scores)
```

모델 저장

```python
import joblib

joblib.dump(forest_reg, "my_model.pkl")
# 그리고 나중에...

#my_model_loaded = joblib.load("my_model.pkl")
```

## 모델 세부 튜닝

### 그리드 탐색

만족할 만한 하이퍼파라미터 조합을 자동으로 찾아줌

```python
from sklearn.model_selection import GridSearchCV
from sklearn.ensemble import RandomForestRegressor

param_grid=[
    {'n_estimators':[3,10,30], 'max_features':[2,4,6,8]},
    {'bootstrap':[False],'n_estimators':[3,10],'max_features':[2,3,4]},
]

forest_reg = RandomForestRegressor()

grid_search = GridSearchCV(forest_reg, param_grid, cv=5,
                          scoring='neg_mean_squared_error',
                          return_train_score=True)

grid_search.fit(housing_prepared, housing_labels)
```

베스트 파라미터 확인

```python
grid_search.best_params_
```

최적의 추정기에 적븐 가능

```python
grid_search.best_estimator_
```

평가 점수도 확인 가능

```python
cvres = grid_search.cv_results_
for mean_score, params in zip(cvres["mean_test_score"], cvres["params"]):
  print(np.sqrt(-mean_score), params)
```

### 테스트 세트로 시스템 평가하기

그리드 서치로 최적의 추정기로 훈련함

```python
final_model = grid_search.best_estimator_

X_test = strat_test_set.drop("median_house_value", axis=1)
y_test = start_test_set["median_house_value"].copy()

X_test_prepared = full_pipeline.transform(X_test)

final_predictions = final_model.predict(X_test_prepared)

final_mse = mean_squared_error(y_test, final_predictions)
final_rmse = np.sqrt(final_mse) 
```

추정값이 얼마나 정확한지 알고 싶다면 일반화 오차의 95% **신뢰 구간**을 계산함

```python
from scipy import stats
confidence = 0.95
squared_errors = (final_predictions - y_test) ** 2
np.sqrt(stats.t.interval(confidence, len(squared_errors) -1,
                        loc = squared_errors.mean(),
                        scale=stats.sem(squared_errors)))
```
