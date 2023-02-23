---
title: "[Chap 3]"
excerpt: "분류"

categories:
  - 핸즈온
tags:
  - [공부, 머신러닝]

use_math: true

permalink: /categories2/handson4/

toc: true
toc_sticky: true

date: 2023-02-23
last_modified_at: 2023-02-23
---

# 분류

## MNIST

데이터 불러오기

```python
from sklearn.datasets import fetch_openml
mnist= fetch_openml("mnist_784", version=1)
```

## 이진 분류기 훈련

두 개의 클래스를 구분할 수 있는 분류기

**확률적 경사 하강법(SGD) 분류기**
- SGD가 한 번에 하나씩 훈련 샘플을 독립적으로 처리하기 때문에 큰 데이터셋을 효율적으로 처리함

```python
from sklearn.linear_model import SGDClassifier

sgd_clf = SGDClassifier(random_state=42)
sgd_clf.fit(X_train, y_train_5)
sgd_clf.predict([some_digit])
```

## 성능 측정

### 교차 검증을 사용한 정확도 측정

k-겹 교차 검증을 사용해 모델 평가함

교차 검증 과정을 더 많이 제어할 필요가 있으면 `StratifiedKFold` 사용

```python
from sklearn.model_selection import StratifiedKFold
from sklearn.base import clone

skfolds = StratifiedKFold(n_splits=3, random_state=42, shuffle=True)

for train_index, test_index in skfolds.split(X_train, y_train_5):
  clone_clf = clone(sgd_clf)
  # print(np.array(X_train.iloc[0]))
  X_train_folds = np.array(X_train.iloc[train_index])
  y_train_folds = y_train_5[train_index]
  X_test_fold = np.array(X_train.iloc[test_index])
  y_test_fold = y_train_5[test_index]

  clone_clf.fit(X_train_folds, y_train_folds)
  y_pred = clone_clf.predict(X_test_fold)
  n_correct = sum(y_pred == y_test_fold) 
  print(n_correct / len(y_pred)) # 0.9502, 0.96565, 0.96495 출력
```

그냥 cross_val_score() 함수로 교차 검증 사용 가능함

```python
from sklearn.model_selection import cross_val_score

cross_val_score(sgd_clf, X_train, y_train_5, cv=3, scoring="accuracy")
```

### 오차 행렬

<mark style='background-color: #fff5b1'>정확도를 분류기의 성능 측정 지표로 선호하지 않음 (특히 불균형 데이터셋)</mark>

더 좋은 성능 평가 지표는 **오차 행렬**을 조사하는 것임
- 클래스 A의 샘플이 클래스 B로 분류된 횟수를 세는 것임

`cross_val_predict`은 `cross_val_score`처럼 교차 검증을 수행하지만 점수가 아닌 예측 값을 반환함

```python
from sklearn.model_selection import cross_val_predict

y_train_pred = cross_val_predict(sgd_clf, X_train, y_train_5, cv=3)
```

오차 행렬 확인

```python
from sklearn.metrics import confusion_matrix
confusion_matrix(y_train_5, y_train_pred)
```
`array([[53892,   687],
       [ 1891,  3530]])`


정확한 음성 : **진짜 음성**

정확하지 않은 음성 : **가짜 음성**

정확한 양성 : **진짜 양성**

정확하지 않은 양성 : **가짜 양성**

```python
y_train_perfect_predictions = y_train_5 # 완벽한 분류기일 경우
confusion_matrix(y_train_5, y_train_perfect_predictions)
```

`array([[54579,     0],
       [    0,  5421]])`

- 거짓 양성, 음성이 없음을 확인할 수 있음

---
**정밀도**

정밀도 = $\frac{TP}{TP+FP}$

TP는 진짜 양성의 수, FP는 거짓 양성의 수

_다른 모든 양성 샘플을 무시하기 때문에 유용하지 않음_

---
**재현율 (민감도 또는 진짜 양성 비율)**

재현율 = $\frac{TP}{TP+FN}$

FN는 거짓 음성의 수

### 정밀도와 재현율

```python
from sklearn.metrics import precision_score, recall_score
precision_score(y_train_5, y_train_pred) # == 4096 / (4096 + 1522), 정밀도
recall_score(y_train_5, y_train_pred) # == 4096 / (4096 + 1325), 재현율
```

---

**F1 점수**

- 정밀도와 재현율을 하나의 숫자로 만듦. 정밀도와 재현율의 **조화 평균** 

$F_1 = \frac{2}{\frac{1}{정밀도}+\frac{1}{재현율}} = 2 \times\frac{정밀도 \times 재현율}{정밀도 + 재현율} = \frac{TP}{TP+\frac{FN+FP}{2}}$

```python
from sklearn.metrics import f1_score
f1_score(y_train_5, y_train_pred)
```

<font color="blue">정밀도/재현율 트레이드오프 존재함</font>

### 정밀도/재현율 트레이드오프

**결정 함수**, **결정 임곗값**
- 결정 임곗값보다 크면 양성
- 결정 임곗값보다 작으면 음성

---
적절한 임곗값을 정하도록 `cross_val_predict()`사용

```python
y_scores = cross_val_predict(sgd_clf, X_train, y_train_5, cv=3, method="decision_function")
```

이 점수로 precision_recall_curve() 함수를 사용하여 가능한 모든 임곗값에 대해 정밀도와 재현율을 계산할 수 있음

```python
from sklearn.metrics import precision_recall_curve
a
precisions, recalls, threshold = precision_recall_curve(y_train_5, y_scores)
```

맷플롯립을 사용하여 그림

```python
def plot_precision_recall_vs_threshold(precisions, recalls, thresholds):
  plt.plot(thresholds, precisions[:-1], "b--", label="precision")
  plt.plot(thresholds, recalls[:-1], "g-", label="recall")
  plt.legend()
  plt.xlabel('value')
  # plt.show()

plot_precision_recall_vs_threshold(precisions, recalls, thresholds)
plt.show()
```


<img src="../../assets/images/022302.png" width="300px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

재현율에 대한 정밀도 곡선을 그려 확인함

```python
def plot_precision_vs_recall(precisions, recalls):
    plt.plot(recalls, precisions, "b-", linewidth=2)
    plt.xlabel("Recall", fontsize=16)
    plt.ylabel("Precision", fontsize=16)
    plt.axis([0, 1, 0, 1])
    plt.grid(True)

plt.figure(figsize=(8, 6))
plot_precision_vs_recall(precisions, recalls)
plt.plot([recall_90_precision, recall_90_precision], [0., 0.9], "r:")
plt.plot([0.0, recall_90_precision], [0.9, 0.9], "r:")
plt.plot([recall_90_precision], [0.9], "ro")
# save_fig("precision_vs_recall_plot")
plt.show()
```

<img src="../../assets/images/022303.png" width="300px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

갑자기 떨어지는 지점이 있음. 이 부분으로 결정하는 것이 좋음.

### ROC 곡선

**수신기 조작 특성 (ROC)** 곡선도 좋음
- 거짓 양성 비율(FPR, 양성으로 잘못 분류된 음성 샘플의 비율)에 대한 진짜 양성 비율(TPR, 재현율)
- TNR(정확하게 분류한 음성 샘플의 비율), 특이도라고도 말함

$FPR = \frac{FP}{FP+TN} = 1-\frac{TN}{FP+TN} = 1-TNR$

FPR 곡선을 그릴 수 있음

```python
from sklearn.metrics import roc_curve

fpr, tpr, thresholds = roc_curve(y_train_5, y_score)
```
<img src="../../assets/images/022304.png" width="300px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

점선은 완전한 ROC 곡선임
- 좋은 분류기는 이 점선에서 최대한 멀리 떨어져 있어야함

**곡선 아래의 면적(AUC)**을 측정하여 분류기들을 비교할 수 있음

```python
from sklearn.metrics import roc_auc_score
roc_auc_score(y_train_5, y_scores)
```

- 완벽한 분류기는 1, 완전한 분류기는 0.5

```python

from sklearn.ensemble import RandomForestClassifier

forest_clf = RandomForestClassifier(random_state = 42)
y_probas_forest = cross_val_predict(forest_clf, X_train, y_train_5, cv=3, method="predict_proba")
```

```python
y_scores_forest = y_probas_forest[:,1] # 양성 클래스에 대한 확률을 점수로 사용함
fpr_forest, tpr_forest, thesholds_forest = roc_curve(y_train_5, y_scores_forest)
```

```python

plt.plot(fpr, tpr, "b:", label="SGD")
plot_roc_curve(fpr_forest, tpr_forest, "Random Forest")
plt.legend()
plt.show()
```
<img src="../../assets/images/022305.png" width="300px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

- 랜덤 포레스트가 좋아보임

```python
roc_auc_score(y_train_5, y_scores_forest)
```
- ROC AUC 점수 확인

---

## 다중 분류

**다중 분류기** : 둘 이상의 클래스를 구별

OvR(one-versus-the-rest)
- 이미지를 분류할 때 각 분류기의 결정 점수 중에서 가장 높은 것을 클래스로 선택

OvO(one-versus-one)
- 이진 분류기를 조합하여 만듦

```python
from sklearn.svm import SVC
svm_clf = SVC()
svm_clf.fit(X_train, y_train)
svm_clf.predict([some_digit])
```

- 서브벡터머신은 OvO 전략을 사용해 이진 분류기를 훈련시키고 결정 점수를 얻어 점수가 가장 높은 클래스를 선택함

```python
some_digit_scores = svm_clf.decision_function([some_digit])
some_digit_scores
```

- 강제로 OvO나 OvR을 사용할 수 있음
```python
from sklearn.multiclass import OneVsRestClassifier
ovr_clf = OneVsRestClassifier(SVC())
ovr_clf.fit(X_train, y_train)
ovr_clf.predict([some_digit])
```

- SGD 분류기는 자동으로 다중 클래스로 분류할 수 있음

---
# 에러 분석

```python
y_train_pred = cross_val_predict(sgd_clf, X_train_scaled, y_train, cv=3)
conf_mx = confusion_matrix(y_train, y_train_pred)
conf_mx
```

오차 행렬로 분석함

```python
plt.matshow(conf_mx, cmap=plt.cm.gray)
plt.show()
```

<img src="../../assets/images/022306.png" width="300px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

주대각선이 밝아야함

- 주대각선을 0으로 만들고 그래프를 그려보겠음

```python

np.fill_diagonal(norm_conf_mx, 0)
plt.matshow(norm_conf_mx, cmap=plt.cm.gray)
plt.show()
     
```

<img src="../../assets/images/022307.png" width="300px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

8열이 밝지만 8행이 어두워 실제 잘 분류한다는 것을 볼 수 있음

- 3과 5가 혼동되는 것을 볼 수 있음

맷플롯립으로 출력

```python
cl_a, cl_b = 3, 5
X_aa = X_train[(y_train == cl_a) & (y_train_pred == cl_a)]
X_ab = X_train[(y_train == cl_a) & (y_train_pred == cl_b)]
X_ba = X_train[(y_train == cl_b) & (y_train_pred == cl_a)]
X_bb = X_train[(y_train == cl_b) & (y_train_pred == cl_b)]

plt.figure(figsize=(8,8))
plt.subplot(221); plot_digits(X_aa[:25], images_per_row=5)
plt.subplot(222); plot_digits(X_ab[:25], images_per_row=5)
plt.subplot(223); plot_digits(X_ba[:25], images_per_row=5)
plt.subplot(224); plot_digits(X_bb[:25], images_per_row=5)
# save_fig("error_analysis_digits_plot")
plt.show()
```

<img src="../../assets/images/022308.png" width="300px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

- 원인은 선형모델인 SGDClassifier를 사용했기 때문임
- 픽셀에 가중치를 할당하고 새로운 이미지에 대해 단순히 가중치 합을 클래스의 점수로 계산했기 때문임

## 다중 레이블 분류

**다중 레이블 분류** : 여러 개의 이진 꼬리표를 출력하는 분류 시스템

```python
from sklearn.neighbors import KNeighborsClassifier

y_train_large = (y_train >= 7)
y_train_odd = (y_train % 2 == 1)
y_multilabel = np.c_[y_train_large, y_train_odd]

knn_clf = KNeighborsClassifier()
knn_clf.fit(X_train, y_multilabel)
```

```python
knn_clf.predict([some_digit])
```

`array([[False,  True]])`

평가 지표는 많으나 F1 점수를 구하고 평균 점수를 계산하는 방법이 있음

```python
y_train_knn_pred = cross_val_predict(knn_clf, X_train, y_multilabel, cv=3)
f1_score(y_multilabel, y_train_knn_pred, average="macro")
```

- 레이블에 클래스의 **지지도**를 가중치로 주는 것도 방법임
- average="weighted"로 설정하면 됨

## 다중 출력 분류

잡음 잡기 처럼 한 레이블이 다중 클래스가 될 수 있도록 일반화한 것임

```python
noise = np.random.randint(0, 100, (len(X_train), 784))
X_train_mod = X_train + noise
noise = np.random.randint(0, 100, (len(X_test), 784))
X_test_mod = X_test + noise
y_train_mod = X_train
y_test_mod = X_test
```

```python
some_index = 0
plt.subplot(121); plot_digit(X_test_mod[some_index])
plt.subplot(122); plot_digit(y_test_mod[some_index])
# save_fig("noisy_digit_example_plot")
plt.show()
```
<img src="../../assets/images/022309.png" width="300px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

```python
knn_clf.fit(X_train_mod, y_train_mod)
clean_digit = knn_clf.predict([X_test_mod[some_index]])
plot_digit(clean_digit)
save_fig("cleaned_digit_example_plot")
```
<img src="../../assets/images/022310.png" width="300px" height="300px" title="OP code 예시" alt="OP code"><img><br/>
