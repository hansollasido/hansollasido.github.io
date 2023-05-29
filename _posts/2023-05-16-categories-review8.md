---
title: "yolo"
excerpt: "You Only Look Once: Unified, Real-Time Object Detection 리뷰"

categories:
  - 컴퓨터 비전
tags:
  - [딥러닝, 논문 리뷰, Object Detection]

permalink: /categories9/review8/

use_math: true
toc: true
toc_sticky: true

date: 2023-05-16
last_modified_at: 2023-05-28
---

# 참고 사이트

[약초의 숲으로 놀러오세요 (herbwood)](https://herbwood.tistory.com/13)

위 사이트에서 참고를 많이 했습니다. 

---

# yolo v1

## 소개

당시에 detection 시스템은 detection에 맞게 classifier을 재구성하였습니다. 사물을 인지하기 위해, 사물을 위한 classifier를 진행하고 다양한 위치에서 평가하고 test 이미지에 맞게 크기를 조정합니다. 

Object detection이었던 DPM (Deformable Parts Models), R-CNN은 각각 sliding window와 region proposal을 거쳐 bounding box를 만들고 그 box에 classifier를 실행합니다. 이후, 중복된 detection을 전처리, 같은 scene에 있는 다른 사물들을 기반으로 box를 다시 점수 매겨 진행하죠. 이러한 복잡한 파이프라인은 느리고 각 구성들을 분리해서 훈련시켜야 하기 때문에 최적화하기 어렵다는 문제점이 있죠.

<p align="center"><img src="../../assets/images/052801.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>

이러한 문제점을 해결하고자, end-to-end 기법으로 image pixel이 들어오면 bounding box 좌표와 class 확률까지 직렬적으로 수행하는 object detection을 재구성하였습니다. 

YOLO(You Only Look Once)는 간단한데 Figure 1을 보시면 됩니다. 하나의 convolutional network가 즉시 다수의 bounding box와 class probabilities를 예측합니다. YOLO는 full 이미지로 훈련하며 바로 detection 성능을 최적화합니다. 이러한 통합된 모델은 예전 방법보다 훨씬 더 유익한 점을 가지고 있습니다. 

먼저, **YOLO는 매우 빠릅니다.** 복잡한 파이프라인이 필요없어 빠른 detection이 가능합니다. neural network를 실행하여 새로운 이미지에 대해 detection을 진행합니다. 

또한, **YOLO는 real-time 시스템에서 평균 정확도가 두 배 정도 뛰어납니다.** sliding window나 region proposal과는 달리, YOLO는 전체 이미지를 보기 때문에 외적인 출현 여부 뿐만아니라 class에 대한 복잡한 정보를 함축적으로 인코딩할 수 있습니다. 또한 Fast R-CNN에 비교하여 배경에 대한 에러를 훨씬 줄입니다. 

세 번째로, **YOLO는 object의 일반화할 수 있는 대표점을 학습합니다.** 자연 이미지로 학습하고 예술에 테스트할 때, YOLO는 DPM과 R-CNN보다 더 좋은 성능을 보여줍니다. 일반화를 잘하기 때문에, 새로운 domain이나 예상치 못한 입력이 들어왔을 때 쉽게 무너지지 않습니다.

YOLO는 정확도 부분에서 [sota(state-of-the-art) (1)](#추가설명) detection 시스템보다 뒤떨어집니다. image에 있는 object를 빠르게 확인할 수 있는 반면에 몇 사물을 정확히 지역화하기에는 (특히 작은 사물) 어려움이 있습니다. 몇 가지 실험을 통해 trade-off를 알려드리겠습니다.

---

### Unified Detection

object detection의 분리된 구성들을 하나의 신경망으로 통합했습니다. 전체 이미지를 통해 bounding box를 예측합니다. end-to-end 훈련과 real-time 스피드를 가진 YOLO는 높은 정확도 또한 가지고 있습니다. 

YOLO는 우선 입력 이미지를 $S \times S$ 그리드로 나눕니다. 만일 사물의 중앙점이 그리드 cell에 들어갈 경우, 해당 그리드 cell은 그 사물을 detect하는데에 책임이 있습니다.

각 그리드 cell은 B bounding box와 confidence score를 가집니다. confidence score(신뢰도)는 얼마나 이 box가 사물을 담고 있고 예측을 얼마나 정확하게 했는지를 반영합니다. 

이 신뢰도를 $\mbox{Pr(Object)} * \mbox{IOU}^{\mbox{truth}}_{\mbox{pred}}$로 정의하겠습니다. 만일 해당 그리드 cell에 사물이 없으면 신뢰도는 0이 됩니다. 그렇지 않을 경우, 신뢰도는 bounding box와 ground truth의 IOU(Intersection over union)와 같게 됩니다. 

각 bounding box는 5개로 구성되어 있습니다: $x, y, w, h,$와 신뢰도 이죠. $(x,y)$는 그리드 cell bound와  관련있는 box의 center를 대표합니다. 너비와 높이는 전체 이미지에 관련하여 예측되죠. 마지막으로 신뢰도는 결과 box와 ground-truth box의 IOU로 표현됩니다. 

각 그리드 cell은 $\mbox{Pr}(\mbox{Class}_i \| \mbox{Object})$, 즉 조건부 확률을 예측합니다. 이 확률은 그리드 cell에 사물이 포함되었을 경우에만 계산됩니다. B box 개수에 상관없이 그리드 cell당 class 확률 예측합니다. 이 점수는 box에 class가 출현되었는지의 확률과 object에 얼마나 잘 맞게 box가 만들었는지에 관련한 확률로 인코딩합니다. 

<p align="center"><img src="../../assets/images/052902.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>

figure 2를 보시면 이해가 쉬울 겁니다. YOLO는 PASCAL VOC를 사용하여 평가하였으며, S=7, B=2로 설정했습니다. PASCAL VOC는 label된 class의 개수가 20개이므로 C=20이니 prediction은 $7\times7\times30$의 tensor가 됩니다. 

---

### 네트워크 설계

이 모델에 CNN을 적용하고 PASCAL VOC 데이터셋에 평가를 진행하였습니다. fully connected layer가 output probabilities와 coordinates를 예측할 때, 첫 번째 층은 image로 부터 특징을 추출합니다. 

[GoogLeNet (2)](#추가설명)을 사용하여 이미지 분류를 하였으며 24개의 convolutional layer과 2개의 fully connected layer를 사용하였습니다. GoogLeNet의 inception 모듈을 사용하는 대신, 간단히 $1\times1$ reduction layer과 $3\times3$ convolutional layer를 사용하였습니다.

더 빠른 버젼의 YOLO를 만들기 위해서 convolutional layer를 24개에서 9개로 줄여 만든 fast YOLO도 있습니다. 다른 모든 parameter 값들은 fast YOLO나 YOLO나 동일합니다. 

<p align="center"><img src="../../assets/images/052901.jpg" width="700px" height="700px" title="OP code 예시" alt="OP code" ><img></p>

---

### 학습

ImageNet 1000-class competition dataset으로 pretrain을 하였으며 이 때 사용한 네트워크는 그림 3에서 average-pooling layer와 fully connected layer 다음에 20개의 convolutional layer를 사용하였습니다. 

detection하기 위해 이 모델을 적절하게 변경하였습니다. 마지막 layer는 class 확률과 bounding box 좌표를 예측합니다. bounding box의 너비와 높이는 정규화하여 0과 1사이의 값을 가지도록 만들었습니다. bounding box의 $x, y$는 특정 그리드 cell 위치의 offset으로 설정하였으며 이 또한 0과 1사이의 값을 가집니다. 

선형 활성화 함수를 마지막 층과 다른 층에 사용하는데 leaky rectified 활성화 함수를 사용하였습니다.

<center>$\phi(x)=\begin{cases}x,& \mbox{if} \ x> 0 \\0.1x, & \mbox{otherwise}\end{cases}$</center>

총 제곱 오차를 사용하여 optimize를 진행하였습니다. 하지만 정확도를 높이는데에 한정적이었고 이상적이지 않았습니다. 또한 거의 많은 그리드 cell이 사물을 포함하고 있지 않아 신뢰도가 과도하게 0이되어 사물을 포함하는 cell의 gradient는 overpower가 된다는 문제점이 있었습니다. 이는 매우 불안정하며 일찍이 훈련을 분산시키는 요인이 됩니다. 

이를 완화하기 위하여 두 개의 parameter를 따로 적용하였습니다. 하나는 box coordinate를 위해, 또 다른 하나는 object이 없는 box를 위해 parameter를 따로 적용하여 coordinate에는 loss를 증가시키고 object가 없는 box의 loss는 감소시켰습니다. 

YOLO는 각 그리드 cell에 다수의 bounding box를 예측합니다. 하지만 우리는 각 사물을 담당하는 box 예측기를 하나만 쓰고 싶습니다. 이에 ground truth과의 IOU가 가장 높은 object를 만들어낸 예측기만 responsible 예측기로 설정합니다. 

<p align="center"><img src="../../assets/images/052903.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>

loss 함수는 위와 같습니다. i는 cell 번호이며, j는 bounding box 예측기의 번호입니다. ij는 i번째 cell에 있는 bounding box 예측기인데 그 중 responsible한 예측기를 말합니다.

(논문에 배치 사이즈, 모멘텀, 에포크, learning rate 등이 있으나 중요한 것은 아니라 넘어가겠습니다.)

---

## 한계

YOLO 논문에 한계도 적어놨네요. 좀 특이한 것 같습니다. 보통 자기 논문에 한계를 적지는 않는데....

YOLO의 한계는 각 그리드 cell이 오직 두개의 box를 예측하고 class는 한 개만 가질 수 있기 때문에 공간적으로 매우 한정적이라는 것입니다. 이 공간적 제약은 가까이 있는 사물의 수를 제한합니다. 즉, YOLO는 새 떼처럼 작은 사물에 대해서는 고전합니다. 

또한 loss 함수가 detection 성능을 근사시킨 값으로 훈련시키기 때문에, loss 함수는 작은 bounding box나 큰 bounding box나 같은 error로 취급합니다. 큰 box의 작은 error는 일반적이지만 작은 box의 작은 error는 IOU에 큰 영향을 끼칩니다. 

**즉 주요 error의 원인은 부정확한 localization입니다.**

---

## 성능비교

<p align="center"><img src="../../assets/images/052904.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>

YOLO는 매우 빠른 것이기 때문에 다른 real-time detector 부분에서 빠른 성능을 보여줍니다. Fast YOLO가 가장 높은 FPS를 보여주네요. 물론 그만큼 YOLO보다 정확도가 낮지만요.

<p align="center"><img src="../../assets/images/052905.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>

당시 나왔던 논문인 Fast R-CNN과 비교했을 때, Background부분에서 오차가 줄었지만, Localization에서 오차가 큰 것을 볼 수 있습니다.

<p align="center"><img src="../../assets/images/052906.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>

YOLO는 Fast R-CNN보다 background에 대한 오차가 적기 때문에 YOLO를 사용하여 Fast R-CNN의 background detection을 줄인다면 성능을 극대화 할 수 있습니다. YOLO와 같이 사용했을 때 mAP가 71.8%에서 75.0%로 3.2% 증가한 것을 볼 수 있습니다. 

<p align="center"><img src="../../assets/images/052907.jpg" width="700px" height="700px" title="OP code 예시" alt="OP code" ><img></p>

<p align="center"><img src="../../assets/images/052908.jpg" width="700px" height="700px" title="OP code 예시" alt="OP code" ><img></p>

<p align="center"><img src="../../assets/images/052909.jpg" width="700px" height="700px" title="OP code 예시" alt="OP code" ><img></p>

빠른 성능을 보여주며 이미지를 detection해주는 모습까지 보여주네요. 

---

## 결론

YOLO 시리즈의 첫 번째 버젼입니다. YOLO는 다른 detector과 달리 전체 이미지를 직접 가져와 학습시키고 구성하는 간단한 모델입니다. 다른 분류기로 사용하는 detector과 달리, YOLO는 직접 detection 성능에 직접 상응하는 loss 함수를 사용하여 학습하고 전체 모델은 공동으로 훈련됩니다. 

Fast YOLO는 가장 빠른 목적으로 재구성된 모델이며, YOLO는 real-time object detection에서 sota를 맡고 있습니다. 

---

## 추가설명

**(1) State Of The Art**
- 어떤 분야에서 현재 가장 최신이고 최고 수준의 기술, 방법, 제품 또는 연구를 가리킴.

**(2) GoogLeNet**
- 2014년에 개발된 딥러닝 모델로, 이미지 인식과 관련된 작업을 수행하는 데 사용됨. ImageNet Large Scale Visual Recognition Challenge 2014에서 우승한 모델로 높은 정확도와 효율성으로 유명함. 
