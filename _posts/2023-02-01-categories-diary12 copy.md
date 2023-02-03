---
title: "2023.02.01"
excerpt: "핸즈온 머신러닝"

categories:
  - 일기
tags:
  - [일상]

permalink: /categories7/diary12/

toc: true
toc_sticky: true

date: 2023-02-01
last_modified_at: 2023-02-01
---

## [2023-02-01]

오랜만에 핸즈온 머신러닝 공부를 진행하였다. 

특히나 로지스틱 회귀에 대해서 자세히 배웠다.

**로지스틱 회귀**란 샘플이 특정 클래스에 속할 확률을 추정하는 데 널리 사용되는 회귀이다. 

1. 확률 추정
2. 훈련과 비용 함수
3. 결정 경계
4. 소프트맥스 회귀(다항 로지스틱 회귀)

로지스틱 회귀에서 확률 추정 함수와 비용 함수가 어떻게 되는지, 결정 경계가 어떻게 구현되는지를 배울 수 있었던 시간이었다.

```python
from sklearn.linear_model import LogisticRegression

X = iris["data"][:, (2, 3)]  # petal length, petal width
y = (iris["target"] == 2).astype(np.int)

log_reg = LogisticRegression(solver="lbfgs", C=10**10, random_state=42)
log_reg.fit(X, y)

x0, x1 = np.meshgrid(
        np.linspace(2.9, 7, 500).reshape(-1, 1),
        np.linspace(0.8, 2.7, 200).reshape(-1, 1),
    )
X_new = np.c_[x0.ravel(), x1.ravel()]

y_proba = log_reg.predict_proba(X_new)

plt.figure(figsize=(10, 4))
plt.plot(X[y==0, 0], X[y==0, 1], "bs")
plt.plot(X[y==1, 0], X[y==1, 1], "g^")

zz = y_proba[:, 1].reshape(x0.shape)
contour = plt.contour(x0, x1, zz, cmap=plt.cm.brg)


left_right = np.array([2.9, 7])
boundary = -(log_reg.coef_[0][0] * left_right + log_reg.intercept_[0]) / log_reg.coef_[0][1]

plt.clabel(contour, inline=1, fontsize=12)
plt.plot(left_right, boundary, "k--", linewidth=3)
plt.text(3.5, 1.5, "Not Iris virginica", fontsize=14, color="b", ha="center")
plt.text(6.5, 2.3, "Iris virginica", fontsize=14, color="g", ha="center")
plt.xlabel("Petal length", fontsize=14)
plt.ylabel("Petal width", fontsize=14)
plt.axis([2.9, 7, 0.8, 2.7])
plt.show()
```

<img src="../../assets/images/020201.png" width="500px" height="500px" title="OP code 예시" alt="OP code"><img><br/>
위 그림은 sklearn에서 Iris-Verginica 데이터셋으로 꽃잎 너비와 꽃잎 길이에 따른 Iris-Verginica 분류 확률 모델이다. 

가운데 점선이 결정 결계인 것을 확인할 수 있었다. 

내일은 여러가지를 더 해봐야한다. 데이콘 대회, All-in study 동시에 해야한다. 병원도 가야하는데 할일이 많네...ㅎ