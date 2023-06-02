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

date: 2023-06-02
last_modified_at: 2023-06-02
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

## 추가설명

**(1) backbone**
- 딥러닝 모델의 구조는 주로 두 부분으로 나눌 수 있는데, 하나는 backbone이라고 불리는 핵심 네트워크 구조이고, 다른 하나는 특정 작업을 위한 추가적인 레이어나 모듈임. backbone은 주로 컴퓨터 비전 관련 작업에서 사용되며, 입력 이미지나 데이터로부터 중요한 특징을 추출하는 역할을 함. 