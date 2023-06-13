---
title: "EfficientDet"
excerpt: "EfficientDet: Scalable and Efficient Object Detection 리뷰"

categories:
  - 컴퓨터 비전
tags:
  - [딥러닝, 논문 리뷰, Object Detection]

permalink: /categories9/review9/

use_math: true
toc: true
toc_sticky: true

date: 2023-06-13
last_modified_at: 2023-06-13
---

# 작성중 (6/2)

# 참고 사이트

[약초의 숲으로 놀러오세요 (herbwood)](https://herbwood.tistory.com/25)

위 사이트에서 참고를 많이 했습니다. 

# EfficientDet

### 소개

Model 효율이 컴퓨터 비전에서 매우 중요한 역할을 담당합니다. 본 논문은 object detection을 위한 신경망 구조 설계에 대해 시스템적으로 연구하고 효율성을 향상시키기 위한 몇 가지 주요한 최적화 방법에 대해 공개하고자 합니다. 

첫 번째로 Bi-directional Feature Pyramid Network (BiFPN)로 쉽고 빠른 multi-scale 특성 융합에 대해 공개합니다.

두 번째로 복합 scaling 방법으로 해상도 깊이 너비를 [backbone (1)](#추가설명), 특성 네트워크, box/class 예측 네트워크를 동시간 대에 균형있게 조정해보고자 합니다.

이러한 최적화와 더 낳은 backbone으로 object detectors의 새로운 공동체: **EfficientDet**를 발전시킬 수 있습니다. 

오픈 소스는 [https://github.com/google/automl/tree/master/efficientdet](https://github.com/google/automl/tree/master/efficientdet)에 있으며 지금부터 EfficientDet에 대해 자세히 알아보겠습니다.

---

### 배경

<p align="center"><img src="../../assets/images/061301.jpg" width="300px" height="300px" title="OP code 예시" alt="OP code" ><img></p>

Sota Object Detector가 점점 expensive해집니다. 파라미터의 개수도 엄청나게 늘어나고 그만큼 정확도도 높아지죠. 하지만 이는 self-driving cars등과 같이 실시간으로 object detection해야할 필요성이 있는 것들에게는 항상 치명적입니다. 모델 사이즈나 latency가 항상 제한될 수 밖에 없죠. 

좀 더 효율적인 detector 구조를 목표로 하기 위하여 이전의 연구들은 one-stage, anchor-free, compress existing model 등으로 구현하였습니다. 이는 효율성을 높일 수는 있지만 정확도가 떨어지는 trade-off가 있었죠. 

정확도를 높이는 동시에 더 좋은 속도를 낼 수 있는 detection 구조에 대해 생각해볼 때입니다. 이를 위해서는 몇 가지 도전과제가 있죠.

도전 1: 효율적인 [multi-scale feature fusion (2)](#추가설명)

---

## 추가설명

**(1) backbone**
- 딥러닝 모델의 구조는 주로 두 부분으로 나눌 수 있는데, 하나는 backbone이라고 불리는 핵심 네트워크 구조이고, 다른 하나는 특정 작업을 위한 추가적인 레이어나 모듈임. backbone은 주로 컴퓨터 비전 관련 작업에서 사용되며, 입력 이미지나 데이터로부터 중요한 특징을 추출하는 역할을 함. 

**(2) multi-scale feature fusion**
- 다중 스케일 특징 융합은 컴퓨터 비전 및 이미지 처리 분야에서 사용되는 기술로, 다양한 크기와 해상도의 이미지에서 추출한 다중 스케일 특징들을 효과적으로 결합하여 더 강력한 특징을 얻는 것을 목표로 함. 

**(3) FPN (Feature Pyramid Network)**
- 객체 탐지 (Object Detection) 및 시맨틱 세그멘테이션 (Semantic Segmentation)과 같은 컴퓨터 비전 작업에서 주로 사용되는 네트워크 아키텍처임. 다중 스케일 특징을 효과적으로 활용하여 객체의 다양한 크기 및 해상도에 대한 정확한 탐지 및 분할을 수행할 수 있도록 도와줌. 