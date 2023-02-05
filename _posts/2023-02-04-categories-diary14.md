---
title: "2023.02.04"
excerpt: "소프트맥스 함수"

categories:
  - 일기
tags:
  - [일상]

permalink: /categories7/diary14/

toc: true
toc_sticky: true

date: 2023-02-04
last_modified_at: 2023-02-04
---

## [2023-02-04]

오늘 핸즈온 머신러닝 공부와 데이콘 대회 준비를 하였다.

#### 핸즈온 머신러닝

핸즈온 머신러닝에서는 **소프트맥스 회귀**에 대해서 배웠다.

소프트맥스 회귀는 각 클래스 k에 대한 점수를 계산하고 확률을 추정한다.

이 때 지수함수를 이용하여 확률을 계산하는데, 지수함수로 계산하기 때문에 각 클래스에 대한 확률이 매우 높거나 매우 낮게 계산된다. 물론 모든 확률을 더하면 1로 나온다.

사이킷런에 있는 LogisticRegression으로 multi_class, solver등을 설정하여 소프트맥스 회귀를 적용해보는 시간도 가졌다.
```python
X = iris["data"][:, (2,3)]
y = iris["target"]

softmax_reg = LogisticRegression(multi_class="multinomial", solver="lbfgs", C=10)
softmax_reg.fit(X, y)
```

---


```python
x0, x1 = np.meshgrid(
        np.linspace(0, 8, 500).reshape(-1, 1),
        np.linspace(0, 3.5, 200).reshape(-1, 1),
    )
X_new = np.c_[x0.ravel(), x1.ravel()]


y_proba = softmax_reg.predict_proba(X_new)
y_predict = softmax_reg.predict(X_new)

zz1 = y_proba[:, 1].reshape(x0.shape)
zz = y_predict.reshape(x0.shape)

plt.figure(figsize=(10, 4))
plt.plot(X[y==2, 0], X[y==2, 1], "g^", label="Iris virginica")
plt.plot(X[y==1, 0], X[y==1, 1], "bs", label="Iris versicolor")
plt.plot(X[y==0, 0], X[y==0, 1], "yo", label="Iris setosa")

from matplotlib.colors import ListedColormap
custom_cmap = ListedColormap(['#fafab0','#9898ff','#a0faa0'])

plt.contourf(x0, x1, zz, cmap=custom_cmap)
contour = plt.contour(x0, x1, zz1, cmap=plt.cm.brg)
plt.clabel(contour, inline=1, fontsize=12)
plt.xlabel("Petal length", fontsize=14)
plt.ylabel("Petal width", fontsize=14)
plt.legend(loc="center left", fontsize=14)
plt.axis([0, 7, 0, 3.5])
plt.show()
```

<img src="../../assets/images/020401.png" width="500px" height="500px" title="OP code 예시" alt="OP code"><img><br/>