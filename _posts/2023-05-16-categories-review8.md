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

<p align="center"><img src="../../assets/images/052801.jpg" width="700px" height="700px" title="OP code 예시" alt="OP code" ><img></p>

이러한 문제점을 해결하고자, end-to-end 기법으로 image pixel이 들어오면 bounding box 좌표와 class 확률까지 직렬적으로 수행하는 object detection을 재구성하였습니다. 

YOLO(You Only Look Once)는 간단한데 Figure 1을 보시면 됩니다. 하나의 convolutional network가 즉시 다수의 bounding box와 class probabilities를 예측합니다. YOLO는 full 이미지로 훈련하며 바로 detection 성능을 최적화합니다. 이러한 통합된 모델은 예전 방법보다 훨씬 더 유익한 점을 가지고 있습니다. 

먼저, **YOLO는 매우 빠릅니다.** 복잡한 파이프라인이 필요없어 빠른 detection이 가능합니다. neural network를 실행하여 새로운 이미지에 대해 detection을 진행합니다. 

또한, **YOLO는 real-time 시스템에서 평균 정확도가 두 배 정도 뛰어납니다.** sliding window나 region proposal과는 달리, YOLO는 전체 이미지를 보기 때문에 외적인 출현 여부 뿐만아니라 class에 대한 복잡한 정보를 함축적으로 인코딩할 수 있습니다. 또한 Fast R-CNN에 비교하여 배경에 대한 에러를 훨씬 줄입니다. 

세 번째로, **YOLO는 object의 일반화할 수 있는 대표점을 학습합니다.** 자연 이미지로 학습하고 예술에 테스트할 때, YOLO는 DPM과 R-CNN보다 더 좋은 성능을 보여줍니다. 일반화를 잘하기 때문에, 새로운 domain이나 예상치 못한 입력이 들어왔을 때 쉽게 무너지지 않습니다.

YOLO는 정확도 부분에서 [(1) sota(state-of-the-art)](#추가설명) detection 시스템보다 뒤떨어집니다. image에 있는 object를 빠르게 확인할 수 있는 반면에 몇 사물을 정확히 지역화하기에는 (특히 작은 사물) 어려움이 있습니다. 몇 가지 실험을 통해 trade-off를 알려드리겠습니다.

---

### Unified Detection

object detection의 분리된 구성들을 하나의 신경망으로 통합했습니다.  

---

## 추가설명

**(1) State Of The Art**
- 어떤 분야에서 현재 가장 최신이고 최고 수준의 기술, 방법, 제품 또는 연구를 가리킴.

