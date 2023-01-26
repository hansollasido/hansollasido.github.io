---
title: "[핸즈온 머신러닝]"
excerpt: "핸즈온 머신러닝 독학"

categories:
  - 핸즈온
tags:
  - [공부, 머신러닝]

permalink: /categories2/handson/

toc: true
toc_sticky: true

date: 2023-01-15
last_modified_at: 2023-01-15
---

## 핸즈온 머신러닝 독학

핸즈온 머신러닝 독학을 진행하며... 

<img src="../../assets/images/handson.png" width="200px" height="200px" title="핸즈온" alt="핸즈온"><img><br/>

---

### 노션링크

[노션링크](https://thoracic-asiago-663.notion.site/be8148cb1c09438daa68c0ea17eb8c9b)

---

## 배우는 내용

1. 데이터 처리, 머신러닝의 기초, 변환기, 원핫인코딩, 훈련/검증/테스트 분할 방법, 특성 조합 등을 학습함 (Chap1, Chap2)

<font color = 'red'>scikit learn으로 선형회귀 학습</font>
```python
from sklearn.linear_model import LinearRegression

lin_reg = LinearRegression()
lin_reg.fit(housing_prepared, housing_labels)
```

<font color = 'red'>학습된 것을 평가</font>
```python
from sklearn.metrics import mean_squared_error
housing_predictions = lin_reg.predict(housing_prepared)
lin_mse = mean_squared_error(housing_labels,housing_predictions)
lin_rmse = np.sqrt(lin_mse)
lin_rmse
```


