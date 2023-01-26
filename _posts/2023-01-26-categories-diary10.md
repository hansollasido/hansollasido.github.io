---
title: "2023.01.26"
excerpt: "Dacon 회의 및 Hands-on Machine Learning 공부"

categories:
  - 일기
tags:
  - [일상]

permalink: /categories7/diary10/

toc: true
toc_sticky: true

date: 2023-01-26
last_modified_at: 2023-01-26
---

## [2023-01-26]

오늘은 Dacon 회의를 진행하였다. EfficientNet으로 Conv3d을 구현하는데에는 실패하였다. input size가 워낙 다르기 때문이다.

하지만 팀원 중에 conv3d로 만들어진 densenet을 불러와 대회 input size에 맞게끔 수정하면서 학습시키어 좋은 점수를 얻게 되었다.

무려 0.95점!! 대회 30위에 해당하는 점수이다.

더 놀라운 점은 아직 전처리를 하지 않았다는 점이다. mediapipe에서 손인식 전처리를 진행한 뒤, 이 모델을 사용하면 더 좋은 결과를 얻을 수 있을 것이라 사료된다. 

Hands-on Machine Learning도 공부하였다. 

오늘은 **다항 회귀**와 **학습 곡선**에 대해 배웠다.

특성이 많을 때 선형 회귀를 이용하여 차수를 높이는 방법과 더 좋은 모델을 찾기 위해 학습 곡선을 그려 과소, 과대적합을 파악할 수 있는 방법까지 알게 되었다. 

다음에는 규제가 있는 선형 모델(릿지 회귀) 등을 배울 예정이다. 

아래는 다항 회귀의 차수에 따라 과소적합, 과대적합이 얼마나 다른지 확인할 수 있는 그래프이다. 

```python
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline

for style, width, degree in (("g-", 1, 300), ("b--", 2, 2), ("r-+", 2, 1)):
    polybig_features = PolynomialFeatures(degree=degree, include_bias=False)
    std_scaler = StandardScaler()
    lin_reg = LinearRegression()
    polynomial_regression = Pipeline([
            ("poly_features", polybig_features),
            ("std_scaler", std_scaler),
            ("lin_reg", lin_reg),
        ])
    polynomial_regression.fit(X, y)
    y_newbig = polynomial_regression.predict(X_new)
    plt.plot(X_new, y_newbig, style, label=str(degree), linewidth=width)

plt.plot(X, y, "b.", linewidth=3)
plt.legend(loc="upper left")
plt.xlabel("$x_1$", fontsize=18)
plt.ylabel("$y$", rotation=0, fontsize=18)
plt.axis([-3, 3, 0, 10])
plt.show()
```

<img src="../../assets/images/012701.png" width="300px" height="200px" title="OP code 예시" alt="OP code"><img><br/>

[관련 노션 링크](https://thoracic-asiago-663.notion.site/Chap4-81704d03482e4c4fae5d14e27c3a2b1e)