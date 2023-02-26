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

### 계산 복잡도

역행렬을 계산하는 계산 복잡도는 일반적으로 $O(n^{2.4})$에서 $O(n^3)$사이임

## 경사 하강법

**경사 하강법(GD)** 는 최적의 해법을 찾을 수 있는 일반적인 최적화 알고리즘임

그레이디언트가 감소하는 방향으로 진행함

$\theta$가 임의의 값으로 시작해서(**무작위 초기화**) 비용 함수가 감소되는 방향으로 진행하여 최솟값에 수렴할 때까지 점진적으로 향상시킴

**학습률**이라는 중요한 하이퍼파라미터로 결정됨

<mark style='background-color: #fff5b1'>경사 하강법의 문제점</mark>
- 무작위 초기화로 인해 **전역 최솟값**보다 **지역 최솟값**에 수렴할 수 있음
  - 비용 함수 MSE는 **볼록 함수**이기 때문에 전역 최솟값만 있음
- 비용 함수를 최소화하는 모델 파라미터의 조합을 찾는 일이 모델 훈련임
  - 이를 모델의 **파라미터 공간**에서 찾는 다고 말함
  - 모델이 가진 파라미터가 많을 수록 검색이 더 어려워짐

### 배치 경사 하강법

$\theta_j$가 조금 변경될 때 비용 함수가 얼마나 바뀌는지 계산해야 함
- 이를 **편도함수**라고 함

**비용 함수의 편도함수**

$\frac{\partial}{\partial\theta_j}MSE(\theta)=\frac{2}{m}\sum^m_{i=1}(\theta^Tx^{(i)}-y^{(i)})x^{(i)}_j$

**비용 함수의 그레이디언트 벡터**

<math xmlns="http://www.w3.org/1998/Math/MathML" display="block">
  <msub>
    <mi mathvariant="normal">&#x2207;<!-- ∇ --></mi>
    <mi>&#x03B8;<!-- θ --></mi>
  </msub>
  <mi>M</mi>
  <mi>S</mi>
  <mi>E</mi>
  <mo stretchy="false">(</mo>
  <mi>&#x03B8;<!-- θ --></mi>
  <mo stretchy="false">)</mo>
  <mo>=</mo>
  <mrow>
    <mo>(</mo>
    <mtable rowspacing="4pt" columnspacing="1em">
      <mtr>
        <mtd>
          <mfrac>
            <mi mathvariant="normal">&#x2202;<!-- ∂ --></mi>
            <mrow>
              <mi mathvariant="normal">&#x2202;<!-- ∂ --></mi>
              <msub>
                <mi>&#x03B8;<!-- θ --></mi>
                <mn>0</mn>
              </msub>
            </mrow>
          </mfrac>
          <mi>M</mi>
          <mi>S</mi>
          <mi>E</mi>
          <mo stretchy="false">(</mo>
          <mi>&#x03B8;<!-- θ --></mi>
          <mo stretchy="false">)</mo>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <mfrac>
            <mi mathvariant="normal">&#x2202;<!-- ∂ --></mi>
            <mrow>
              <mi mathvariant="normal">&#x2202;<!-- ∂ --></mi>
              <msub>
                <mi>&#x03B8;<!-- θ --></mi>
                <mn>1</mn>
              </msub>
            </mrow>
          </mfrac>
          <mi>M</mi>
          <mi>S</mi>
          <mi>E</mi>
          <mo stretchy="false">(</mo>
          <mi>&#x03B8;<!-- θ --></mi>
          <mo stretchy="false">)</mo>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <mfrac>
            <mi mathvariant="normal">&#x2202;<!-- ∂ --></mi>
            <mrow>
              <mi mathvariant="normal">&#x2202;<!-- ∂ --></mi>
              <msub>
                <mi>&#x03B8;<!-- θ --></mi>
                <mn>2</mn>
              </msub>
            </mrow>
          </mfrac>
          <mi>M</mi>
          <mi>S</mi>
          <mi>E</mi>
          <mo stretchy="false">(</mo>
          <mi>&#x03B8;<!-- θ --></mi>
          <mo stretchy="false">)</mo>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <mo>&#x22EF;<!-- ⋯ --></mo>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <mfrac>
            <mi mathvariant="normal">&#x2202;<!-- ∂ --></mi>
            <mrow>
              <mi mathvariant="normal">&#x2202;<!-- ∂ --></mi>
              <msub>
                <mi>&#x03B8;<!-- θ --></mi>
                <mi>n</mi>
              </msub>
            </mrow>
          </mfrac>
          <mi>M</mi>
          <mi>S</mi>
          <mi>E</mi>
          <mo stretchy="false">(</mo>
          <mi>&#x03B8;<!-- θ --></mi>
          <mo stretchy="false">)</mo>
        </mtd>
      </mtr>
    </mtable>
    <mo>)</mo>
  </mrow>
  <mo>=</mo>
  <mfrac>
    <mn>2</mn>
    <mi>m</mi>
  </mfrac>
  <msup>
    <mi>X</mi>
    <mi>T</mi>
  </msup>
  <mo stretchy="false">(</mo>
  <mi>X</mi>
  <mi>&#x03B8;<!-- θ --></mi>
  <mo>&#x2212;<!-- − --></mo>
  <mi>y</mi>
  <mo stretchy="false">)</mo>
</math>

매 경사 하강법 스텝에서 전체 훈련 세트 X에 대해 계산함 $\Longrightarrow$ **배치 경사 하강법**
- 매 스텝에서 훈련 데이터 전체를 사용함

위로 향하는 그레이디언트 벡터가 구해지면 반대 방향인 아래로 가야함. $\theta$에서 $\nabla_\theta MSE(\theta)$를 빼야한다는 뜻임

**경사 하강법 스텝**

$\theta^{(next step)}=\theta-\eta\nabla_\theta MSE(\theta)$

```python
eta = 0.1 # 학습률
n_iterations = 1000
m = 100

theta = np.random.randn(2,1) # 무작위 초기화

for iteration in range(n_iterations):
  gradients = 2/m * X_b.T.dot(X_b.dot(theta) - y)
  theta = theta-eta*gradients
```

<img src="../../assets/images/022602.png" width="400px" height="400px" title="OP code 예시" alt="OP code"><img><br/>

왼쪽은 학습률이 낮고 오른쪽은 학습률이 너무 큼

그리드 탐색을 통해 적절한 학습률을 찾음

### 확률적 경사 하강법

배치 경사 하강법의 가장 큰 문제는 매 스텝에서 전체 훈련 세트를 사용해 그레이디언트를 계산한다는 것임. 훈련 세트가 커지면 매우 느려지게 됨

**확률적 경사 하강법**은 매 스텝에서 한 개의 샘플을 무작위로 선택하고 그 하나의 샘플에 대한 그레이디언트를 계산함

배치 경사 하강법보다 훨씬 불안정함. 그러나 비용 함수가 매우 불규칙할 때 알고리즘이 지역 최솟값을 건너뛰도록 도와주므로 전역 최솟값을 찾을 가능성이 높음

전역 최솟값에 다다르지 못한다는 점이 좋지 않아 처음에 학습률을 크게 하고 점차 줄이는 방식으로 알고리즘을 설계함. 이는 어닐링 과정에서 영감을 얻은 **담금질 기법** 알고리즘과 유사함

매 반복에서 학습률을 결정하는 함수를 **학습 스케줄**이라고 부름

```python
n_epochs = 50
t0, t1 = 5, 50 # 학습 스케줄 하이퍼파라미터

def learning_schedule(t):
    return t0 / (t + t1)

theta = np.random.randn(2, 1) # 무작위 초기화

for epoch in range(n_epochs):
    for i in range(m):
        random_index = np.random.randint(m)
        xi = X_b[random_index:random_index+1]
        yi = y[random_index:random_index+1]
        gradients = 2 * xi.T.dot(xi.dot(theta) - yi)
        eta = learning_schedule(epoch * m + i)
        theta = theta - eta * gradients

```
반복되는 횟수를 **에포크**라고 함

```python
from sklearn.linear_model import SGDRegressor
sgd_reg = SGDRegressor(max_iter=1000, tol=1e-3, penalty=None, eta0=0.1)
sgd_reg.fit(X, y.ravel())
sgd_reg.intercept_, sgd_reg.coef_
```

### 미니배치 경사 하강법

**미니배치**라 부르는 임의의 작은 샘플 세트에 대해 그레이디언트를 계산함
- 최적화된 하드웨어, 특히 GPU를 사용해서 얻는 성능 향상임

SGD보다 덜 불규칙하게 움직임. 최솟값에 더 가까이 도달하게 될 것이지만 빠져나오기는 더 힘듦

<img src="../../assets/images/022603.png" width="400px" height="400px" title="OP code 예시" alt="OP code"><img><br/>