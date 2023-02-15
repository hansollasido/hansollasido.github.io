---
title: "2023.02.15"
excerpt: "핸즈온 머신러닝 더 공부"

categories:
  - 일기
tags:
  - [일상]

permalink: /categories7/diary20/

toc: true
toc_sticky: true

date: 2023-02-15
last_modified_at: 2023-02-15
---

## [2023-02-15]

핸즈온 머신러닝을 오늘 많이 했다.

특히 랜덤포레스트, 부스팅에 대해 배웠다.

앙상블 개념을 이해하면서 정말 이러한 머신러닝 알고리즘을 개발한 사람들은 얼마나 똑똑할까라는 느낌이 들었다. 

특히 전에 많이 사용했던 랜덤포레스트, 에이다부스트, 그레이디언트 부스트의 개념을 확실히 알 수 있었던 계기가 되었다. 



```python
from sklearn.ensemble import AdaBoostClassifier

ada_clf = AdaBoostClassifier(
    DecisionTreeClassifier(max_depth=1), n_estimators=200,
    algorithm="SAMME.R", learning_rate=0.5, random_state=42)
ada_clf.fit(X_train, y_train)

```
```python
from matplotlib.colors import ListedColormap

def plot_decision_boundary(clf, X, y, axes=[-1.5, 2.45, -1, 1.5], alpha=0.5, contour=True):
    x1s = np.linspace(axes[0], axes[1], 100)
    x2s = np.linspace(axes[2], axes[3], 100)
    x1, x2 = np.meshgrid(x1s, x2s)
    X_new = np.c_[x1.ravel(), x2.ravel()]
    y_pred = clf.predict(X_new).reshape(x1.shape)
    custom_cmap = ListedColormap(['#fafab0','#9898ff','#a0faa0'])
    plt.contourf(x1, x2, y_pred, alpha=0.3, cmap=custom_cmap)
    if contour:
        custom_cmap2 = ListedColormap(['#7d7d58','#4c4c7f','#507d50'])
        plt.contour(x1, x2, y_pred, cmap=custom_cmap2, alpha=0.8)
    plt.plot(X[:, 0][y==0], X[:, 1][y==0], "yo", alpha=alpha)
    plt.plot(X[:, 0][y==1], X[:, 1][y==1], "bs", alpha=alpha)
    plt.axis(axes)
    plt.xlabel(r"$x_1$", fontsize=18)
    plt.ylabel(r"$x_2$", fontsize=18, rotation=0)
```

```python
plot_decision_boundary(ada_clf, X, y)
```
<img src="../../assets/images/021502.png" width="500px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

```python
m = len(X_train)

fix, axes = plt.subplots(ncols=2, figsize=(10,4), sharey=True)
for subplot, learning_rate in ((0, 1), (1, 0.5)):
    sample_weights = np.ones(m) / m
    plt.sca(axes[subplot])
    for i in range(5):
        svm_clf = SVC(kernel="rbf", C=0.2, gamma=0.6, random_state=42)
        svm_clf.fit(X_train, y_train, sample_weight=sample_weights * m)
        y_pred = svm_clf.predict(X_train)

        r = sample_weights[y_pred != y_train].sum() / sample_weights.sum() # equation 7-1
        alpha = learning_rate * np.log((1 - r) / r) # equation 7-2
        sample_weights[y_pred != y_train] *= np.exp(alpha) # equation 7-3
        sample_weights /= sample_weights.sum() # normalization step

        plot_decision_boundary(svm_clf, X, y, alpha=0.2)
        plt.title("learning_rate = {}".format(learning_rate), fontsize=16)
    if subplot == 0:
        plt.text(-0.75, -0.95, "1", fontsize=14)
        plt.text(-1.05, -0.95, "2", fontsize=14)
        plt.text(1.0, -0.95, "3", fontsize=14)
        plt.text(-1.45, -0.5, "4", fontsize=14)
        plt.text(1.36,  -0.95, "5", fontsize=14)
    else:
        plt.ylabel("")

plt.show()
```

<img src="../../assets/images/021501.png" width="700px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

위는 에이다부스트 코드이며, 앙상블할 때 가중치를 다르게 하여 예측기를 지나갈 때 마다 더 좋은 예측기로 훈련되는 것을 확인할 수 있다. 


에이다부스트는 learning_rate 매개변수를 통해 학습률을 조정할 수 있다. 