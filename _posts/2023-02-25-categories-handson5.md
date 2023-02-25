---
title: "[Chap 4]"
excerpt: "모델 훈련"

categories:
  - 핸즈온
tags:
  - [공부, 머신러닝]

use_math: true

permalink: /categories2/handson5/

toc: true
toc_sticky: true

date: 2023-02-25
last_modified_at: 2023-02-25
---

# 모델 훈련

**선형 회귀**
- 직접 계산할 수 있는 공식을 사용하여 훈련 세트에 가장 잘 맞는 모델 파라미터를 해석적으로 구함
- 경사 하강법(SG)이라 불리는 반복적인 최적화 방식을 사용하여 모델 파라미터를 조금씩 바꾸면서 비용 함수를 훈련 세트에 대해 최소화함
- 배치 경사 하강법, 미니배치 경사 하강법, 확률적 경사 하강법(SGD)

## 선형 회귀

**선형 회귀 모델의 예측**

$\hat{y}=\theta_0+\theta_1x_1+\theta_2x_2+\cdots+\theta_nx_n$

- $\hat{y}$는 예측값
- n은 특성의 수
- $x_i$는 i번째 특성값
- $\theta_j$는 j번째 모델 파라미터

**선형 회귀 모델의 예측(벡터 형태)**

$\hat{y}=h_\theta(x)=\theta \cdot x$

- $\theta$는 편향 $\theta_0$과 $\theta_1$에서 $\theta_n$까지의 특성 가중치를 담은 모델의 파라미터 벡터임
- x는 $x_0$에서 $x_n$까지 담은 샘플의 **특성 벡터**임. $x_0$는 항상 1임
- $\theta\cdot$는 벡터 $\theta$와 x의 점곱임. 이는 $\theta_0x_0+\theta_1x_1+\theta_2x_2+\cdots+\theta_nx_n$와 같음
- $h_\theta$는 모델 파라미터 $\theta$를 사용한 가설 함수임

회귀에서 가장 널리 사용되는 성능지표는 RMSE. RMSE를 최소화하는 $\theta$를 찾아야함. MSE를 최소화하는 것이 간단함

**선형 회귀 모델의 MSE 비용 함수**

$MSE(X,h_\theta)=\frac{1}{m}\sum^m_{i=1}(\theta^Tx^{(i)}-y^{(i)})^2$

### 정규방정식

비용 함수를 최소화하는 $\theta$값을 찾기 위한 해석적인 방법

**정규방정식**

$\hat{\theta}=(X^TX)^{-1}X^Ty$

- $\hat{\theta}$는 비용 함수를 최소화하는 $\theta$값
- y는 $y^{(1)}$부터 $y^{(m)}$까지 포함하는 타깃 벡터임

```python
import numpy as np

X = 2*np.random.rand(100, 1)
y = 4 + 3 * X + np.random.randn(100, 1)

X_b = np.c_[np.ones((100, 1)), X] # 모든 샘플에 x0 = 1을 추가함
theta_best = np.linalg.inv(X_b.T.dot(X_b)).dot(X_b.T).dot(y)
theta_best
```

`array([[4.21509616],[2.77011339]])`

가우시안 잡음을 추가했기 때문에 정확하게 재현하지 못함

```python
X_new = np.array([[0], [2]])
X_new_b = np.c_[np.ones((2,1)), X_new] # 모든 샘플에 x0 = 1을 추가함
y_predict = X_new_b.dot(theta_best)
plt.plot(X_new, y_predict, "r-", label = "예측")
plt.plot(X, y, "b.")
plt.axis([0, 2, 0, 15])
plt.xlabel('X_1')
plt.ylabel('y')
plt.legend()
plt.show()
```

<img src="../../assets/images/022501.png" width="300px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

사이킷런에서 선형 회귀 수행하기

```python
from sklearn.linear_model import LinearRegression
lin_reg = LinearRegression()
lin_reg.fit(X, y)
```

```python
# 기중치와 편향 볼 수 있음
lin_reg.intercept_, lin_reg.coef_
```

LinearRegression은 scipy.linalg.lstsq() 함수를 기반으로 함

```python
theta_best_svd, residuals, rank, s = np.linalg.lstsq(X_b, y, rcond=1e-6)
```

이 함수는 $\hat{\theta}=X^+y$를 계산함. 여기세어 $X^+$는 $X$의 **유사역행렬**임

```python
# 유사역행렬을 직접 구할 수 있음
np.linalg.pinv(X_b).dot(y)
```

유사역행렬은 **특잇값 분해 (singular value decomposition : SVD)** 표준 행렬 분해 기법을 사용해 계산됨. $X=U\Sigma V^T$ $\Longrightarrow$ $X^+=V\Sigma^+U^T$ 

