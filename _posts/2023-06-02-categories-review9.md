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

# 참고 사이트

[약초의 숲으로 놀러오세요 (herbwood)](https://herbwood.tistory.com/25)

위 사이트에서 참고를 많이 했습니다. 

# EfficientDet

### Abstract

Model 효율이 컴퓨터 비전에서 매우 중요한 역할을 담당합니다. 본 논문은 object detection을 위한 신경망 구조 설계에 대해 시스템적으로 연구하고 효율성을 향상시키기 위한 몇 가지 주요한 최적화 방법에 대해 공개하고자 합니다. 

첫 번째로 Bi-directional Feature Pyramid Network (BiFPN)로 쉽고 빠른 multi-scale 특성 융합에 대해 공개합니다.

두 번째로 복합 scaling 방법으로 해상도 깊이 너비를 [backbone (1)](#추가설명), 특성 네트워크, box/class 예측 네트워크를 동시간 대에 균형있게 조정해보고자 합니다.

이러한 최적화와 더 낳은 backbone으로 object detectors의 새로운 공동체: **EfficientDet**를 발전시킬 수 있습니다. 

오픈 소스는 [https://github.com/google/automl/tree/master/efficientdet](https://github.com/google/automl/tree/master/efficientdet)에 있으며 지금부터 EfficientDet에 대해 자세히 알아보겠습니다.

---

### Introduction

<p align="center"><img src="../../assets/images/061301.jpg" width="300px" height="300px" title="OP code 예시" alt="OP code" ><img></p>

Sota Object Detector가 점점 expensive해집니다. 파라미터의 개수도 엄청나게 늘어나고 그만큼 정확도도 높아지죠. 하지만 이는 self-driving cars등과 같이 실시간으로 object detection해야할 필요성이 있는 것들에게는 항상 치명적입니다. 모델 사이즈나 latency가 항상 제한될 수 밖에 없죠. 

좀 더 효율적인 detector 구조를 목표로 하기 위하여 이전의 연구들은 one-stage, anchor-free, compress existing model 등으로 구현하였습니다. 이는 효율성을 높일 수는 있지만 정확도가 떨어지는 trade-off가 있었죠. 

정확도를 높이는 동시에 더 좋은 속도를 낼 수 있는 detection 구조에 대해 생각해볼 때입니다. 이를 위해서는 몇 가지 도전과제가 있죠.

도전 1: 효율적인 [multi-scale feature fusion (2)](#추가설명)
- 단순히 feature fusion으로 network 구조를 개발하는 것은 확실한 차이를 만들어 내지 못합니다. input feature가 다르기 때문에 다른 해상도를 만들어내는데, output feature가 동일하지 않게 fusion하는 것을 볼 수 있습니다. 이러한 문제를 해결하기 위하여 더 효율적인 bi-directional feature pyramid network(BiFPN)을 공개합니다. BiFPN은 다른 input feature의 중요성을 학습하여 학습할 수 있는 가중치를 도입한 network입니다.

도전 2: model scaling
- 이전 연구들은 좋은 정확도를 위하여 더 큰 backbone network나 더 큰 input image size에 주로 의존하였습니다. feature network 사이즈를 높이는 것과 box/class prediction network의 사이즈를 높이는 것은 정확도와 속도면 두 부분에서 중요하다는 것을 관찰하였습니다.

이전 공통되게 사용된 backbone보다 더 좋은 속도를 얻기 위하여 EfficientNets를 최근 도입하였습니다. EfficientNet backbone과 BiFPN을 결합하여 새로운 object detection을 구현하였으니, 이름은 EfficientDet입니다. 

이 EfficientDet은 파라미터 개수는 크게 줄였으며 정확도는 4 AP나 올랐습니다. 

---

### 관련 연구들

**One-Stage Detectors**
- 기존 object detector은 대부분 proposal을 내기 위한 two-stage이거나 one-stage입니다. two-stage는 더 flexible하고 accurate할 수 있지만 one-stage는 간단하고 사전에 정의된 anchor로 인해 더 효율적입니다. 최근, one-stage detector는 효율성, 간단성으로 인해 주목을 받고 있습니다. 본 논문은 one-stage detector 설계를 따라 속도와 더 좋은 정확성을 얻을 수 있도록 네트워크 구조를 최적화해보겠습니다.

**Multi-Scale Feature**
- object detection에서 가장 어려운 점 중 하나는 mult-scale feature를 효과적으로 표현하고 처리하는 것입니다. 본 논문에서는 multi-scale feature fusion을 직관적이고 합리적인 방법으로 최적화하는 것이 목표입니다.

**Model Scaling**
- 더 좋은 정확도를 얻기 위해, 더 큰 backbone 네트워크를 기용하여 baseline detector를 크게 만드는 것이 일반적이었습니다. 네트워크 너비, 깊이, 해상도의 크기를 늘려서 모델의 정확도를 올렸죠. 본 논문은 object detection을 위한 통합적인 scaling method를 **EfficientNet**에 영감을 받아 개발하였습니다.

---

### BiFPN

multi-scale feature fusion 문제를 수식화하였습니다. 그 다음 본 논문의 주요 아이디어인 BiFPN(Efficient bidirectional cross-cale connections and weighted feature fusion)을 제안합니다. 

#### Problem Formulation

<p align="center"><img src="../../assets/images/061402.jpg" width="700px" height="700px" title="OP code 예시" alt="OP code" ><img></p>

BiFPN이 나오기 전의 Network를 수식화하고 도형화해서 간단하게 표현해보면 위 그림과 같습니다. 

multi-scale feature list를 $\vec{P}^{in}=(P^{in}_{l_1}, ...)$ 이라 할 때, output을 $\vec{P}^{out}=f(\vec{P}^{in})$이라 합시다. 

단순한 FPN 같은 경우

<p align="center">$P^{out}_7=Conv(P^{in}_7)$</p>

<p align="center">$P^{out}_6=Conv(P^{in}_6+Resize(P^{out}_6))$</p>

<p align="center">...</p>

<p align="center">$P^{out}_3=Conv(P^{in}_3+Resize(P^{out}_3))$</p>

이 됩니다. FPN에 대한 자세한 내용은 [FPN 논문](https://arxiv.org/pdf/1612.03144.pdf)을 참고하시길 바랍니다. 

FPN은 한방향으로 이뤄져 있는 information flow이기 때문에, 내부적으로 제한되어 있습니다. 이러한 문제를 해결하기 위하여 PANet은 추가적인 bottom-up path를 붙여 network를 구성하였습니다. 최근에는 NAS-FPN을 적용하여 cross-scale feature network를 만들었는데 GPU 시간을 갉아먹고, network가 불규칙적이어서 수정하거나 해석하기가 어렵다는 단점이 있습니다. 

FPN과 NAS-FPN보다 PANet이 정확도 부분에서 좋은 결과를 낳는다는 것을 볼 수 있었으나 문제는 파라미터의 개수가 많고 계산량이 많다는 부분이었습니다. 이러한 속도, 효율성 면에서 개선하기 위하여 본 논문은 cross-scale connection을 최적화하는 여러가지 방법을 공개하려고 합니다. 

첫 번째로, one input edge를 제거합니다. 이 one input edge는 feature fusion이 없기 때문에 network 대한 영향력이 적습니다. 이것을 제거하면 간단한 bi-directional network가 만들어 집니다. 

두 번째로, 원래의 input에서부터 추가적인 edge를 넣습니다. 

세 번째로, 하나의 top-down path와 bottom-up path를 가진 PANet과는 다르게 bidirectional path(top-down & bottom-up)을 사용하고 같은 layer은 여러번 반복하여 고레벨 feature fusion을 가능하게 만듭니다. [Compound Scaling Method](#추가설명)을 사용하여 layer 수를 결정하는 방법에 대해서 나중에 설명해보겠습니다. 

이러한 최적화를 이용하여 새로운 네트워크를 만들었는데 이를 bidrectional feature pyramid network(BiFPN)이라 명명하겠습니다.

<p align="center"><img src="../../assets/images/061404.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>

fusion하기 위하여 만들어진 수식은 위와 같습니다. $P^{out}_6$은 $P^{in}_7$과 $P^{td}_6$로 결정됩니다. 이와 같은 수식으로 fusion하여 bi-directional 한 네트워크를 구성합니다. 

---

### EfficientDet 구조

BiFPN을 기반으로 EfficientDet이라는 새로운 detection model을 개발하였습니다. 

<p align="center"><img src="../../assets/images/061403.jpg" width="700px" height="700px" title="OP code 예시" alt="OP code" ><img></p>

BiFPN을 사용하여 output feature를 만들면 이를 class와 box network로 보내어 object class, bounding box prediction을 각각 수행하도록 만듭니다. 

---

### Compound Scaling

정확도와 속도를 최적화 하기 위하여, resource 제한을 넓게 포괄하는 모델이 필요했습니다. 주요한 challenge는 EfficientDet 모델을 어떻게 크기를 키우는지 입니다. 

기존의 방법들은 그리드 서치가 불리한 scaling method이기 때문에, width와 depth, input resolution을 한번에 scaling up 할 수 있는 파라미터인 $\phi$을 적용하여 compound scaling을 진행하였습니다.

**Backbone network**
- width/depth는 EfficientNet-B0부터 B6의 coefficient를 재사용하였습니다.

**BiFPN network**

<p align="center">$W_{bifpn}=64\cdot(1.35^{\phi}), \ D_{bifpn}=3+\phi$ </p>

- depth(#layers)는 선형적으로 width(#channel)는 지수형태로 증가하도록 수식을 만들었습니다. 그리드 서치를 위하여 $\phi$는 {1.2, 1.25, 1.3, 1.35, 1.4, 1.45} 중 하나로 수행합니다. 

**Box/class prediction network**
- BiFPN처럼 항상 너비가 고정되지만 선형적으로 증가하도록 depth (#layers)를 다음과 같이 만들었습니다.

<p align="center">$D_{box}=D_{class}=3+[\phi/3]$</p>

**Input image resolution**
- BiFPN에서 사용되는 3-7층은 128로 나뉠수 있는 input resolution이어야 합니다. 이에 우리는 다음과 같은 수식을 만들었습니다. 

<p align="center">$R_{input}=512+\phi\cdot 128$</p>

<p align="center"><img src="../../assets/images/061405.jpg" width="700px" height="700px" title="OP code 예시" alt="OP code" ><img></p>

다음 도표를 통해 $Dn$를 정의하고 결과를 확인해보겠습니다.

---

### Experiments

<p align="center"><img src="../../assets/images/061406.jpg" width="700px" height="700px" title="OP code 예시" alt="OP code" ><img></p>

전체적인 결과는 위의 표와 같습니다. 기존의 FPN, NAS-FPN보다 성능이 뛰어나는 것을 볼 수 있네요. 

<p align="center"><img src="../../assets/images/061407.jpg" width="700px" height="700px" title="OP code 예시" alt="OP code" ><img></p>

제일 중요한 결과 그림이 위에 있습니다. 파라미터의 개수는 적은데 정확도는 높고 속도는 빠른 것을 볼 수 있습니다. 확실히 Bi-FPN이 NAS-FPN과 보다 성능이 좋은 것을 볼 수 있습니다. 

<p align="center"><img src="../../assets/images/061409.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>

FPN과 BiFPN을 비교하였을 때, 정확도는 4 AP 정도, 파라미터 개수는 9M, FLOPs는 51B 정도 줄어든 것을 볼 수 있습니다. 확실히 정확도는 높은데 속도는 개선이 되었네요. 

---

## 추가설명

**(1) backbone**
- 딥러닝 모델의 구조는 주로 두 부분으로 나눌 수 있는데, 하나는 backbone이라고 불리는 핵심 네트워크 구조이고, 다른 하나는 특정 작업을 위한 추가적인 레이어나 모듈임. backbone은 주로 컴퓨터 비전 관련 작업에서 사용되며, 입력 이미지나 데이터로부터 중요한 특징을 추출하는 역할을 함. 

**(2) multi-scale feature fusion**
- 다중 스케일 특징 융합은 컴퓨터 비전 및 이미지 처리 분야에서 사용되는 기술로, 다양한 크기와 해상도의 이미지에서 추출한 다중 스케일 특징들을 효과적으로 결합하여 더 강력한 특징을 얻는 것을 목표로 함. 

**(3) FPN (Feature Pyramid Network)**

<p align="center"><img src="../../assets/images/061401.jpg" width="300px" height="300px" title="OP code 예시" alt="OP code" ><img></p>

- 객체 탐지 (Object Detection) 및 시맨틱 세그멘테이션 (Semantic Segmentation)과 같은 컴퓨터 비전 작업에서 주로 사용되는 네트워크 아키텍처임. 다중 스케일 특징을 효과적으로 활용하여 객체의 다양한 크기 및 해상도에 대한 정확한 탐지 및 분할을 수행할 수 있도록 도와줌. 

**(4) Compound SCaling Method**
- 네트워크 디자인 기술 중에 하나로, CNN에서 많이 사용됨. Depth, Width, Resolution Scaling을 동시에 진행하여 성능과 효율성을 높이는 목적으로 사용됨. 