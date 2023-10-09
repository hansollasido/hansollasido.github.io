---
title: "CBAM"
excerpt: "CBAM: Convolutional Block Attention Module"

categories:
  - 컴퓨터 비전
tags:
  - [딥러닝, 논문 리뷰, Transformer]

permalink: /categories9/review20/

use_math: true
toc: true
toc_sticky: true

date: 2023-10-09
last_modified_at: 2023-10-09
---

### Abstract

Convolutional Block Attention Module (CBAM)에 대해서 소개합니다. CBAM은 feed-forward convolutional neural network를 위한 효과적인 attention module입니다. 

Intermediate feature map이 있다면, 본 논문의 모듈은 두 개의 분리된 차원, 채널, 공간에 따라 attention map을 연속적으로 만들어내고, attention map은 input feature map과 곱해져 수정됩니다. CBAM이 가볍고 일반적인 모듈이기 때문에, 적은 overhead로 CNN 구조에 통합될 수 있으며 base CNN으로 end-to-end 학습이 가능합니다. 

---

### Introduction

<p align="center"><img src="../../assets/images/100907.png" width="500px" height="500px" title="CBAM" alt="CBAM" ><img></p>

CNN에서는 width, depth, cardinality를 보며 발전해왔습니다. VGGNet, ResNet, ResNeXt 등이 있었죠. 하지만 본 논문은 좀 다른 관점에서 CNN을 보려고 합니다. 바로 **attention**이죠. 

Attention은 이전까지 광범위하게 연구되었습니다. Attention은 어디에 집중해야하는 지 뿐만 아니라, 관심있는 곳을 표현하는 데에도 개선해줍니다. 본 논문의 목적은 attention 매커니즘을 활용하여, representation power를 증가시키는 것입니다. 중요한 특징과 불필요한 것을 줄이면서요. 

본 논문에서 새로운 network 모듈인 "Convolutional Block Attention Module" (CBAM)을 소개합니다. convolution으로 cross-channel과 spatial information을 통합하여 informative feature을 추출할 수 있기에, 본 논문의 모듈은 channel과 spatial axes라는 두 주된 차원으로 의미있는 feature을 강조합니다. <figure 1>에서 보이는 것 처럼, channel과 spatial attention module을 연속으로 적용합니다. 이러면 channel과 spatial 축 각각에서 '무엇을', '어디에서' 라는 요소를 학습할 수 있게 됩니다. 결과적으로 본 논문의 모듈은 강조(emphasize)나 축소(suppress)로 information flow를 효율적으로 만듭니다. 

ImageNet-1K dataset으로 다양한 baseline network를 통해 정확도 향상을 얻었습니다. 또한 본 논문의 모듈은 가벼워서 parameter와 computation의 overhead는 작았습니다.

---

**Contribution**

- 본 논문은 3개의 contribution이 있습니다.

1. 본 논문은 간단하고 효과적인 attention module (CBAM)을 소개합니다. CBAM은 CNN의 representation power을 증가시킬 수 있습니다. 

2. 본 논문의 attention 모듈의 효율성을 검증하였습니다.

3. 본 논문은 다양한 network의 성능이 CBAM 모듈을 통하여 향상되었다는 것을 검증하였습니다.

---

### Convolutional Block Attention Module

intermediate feature map $F \in \mathbb{R}^{C\times H \times W}$를 input으로 주어진다면, CBAM은 1D channel attention map $M_c \in \mathbb{R}^{C\times 1 \times 1}$과 2D spatial attention map $M_s \in \mathbb{1 \times H \times W}$를 연속으로 취합니다. 즉 아래와 같이 정리할 수 있습니다.

<center>

$F'=M_c(F)\otimes F,$

<p>

</p>

$F''=M_s(F')\otimes F',$

</center>

여기서 $\otimes$는 element-wise multiplication입니다. nuultiplication 동안, attention 값은 broadcasted 됩니다. channel attention value가 spatial dimension 등으로 broadcast 됩니다. <figure 2>를 보면 attention map의 computation 과정이 묘사되어 있습니다. 

<p align="center"><img src="../../assets/images/100908.jpg" width="500px" height="500px" title="CBAM" alt="CBAM" ><img></p>

---

#### Channel attention module

inter-channel 관계를 추출하여 channel attention module을 만들었습니다. feature detector로서 feature map의 각 channel이 사용되었는데, channel attention은 input image의 '무엇'이라는 의미적인 것에 집중합니다. channel attention을 효과적으로 계산하기 위하여, input feature map의 spatial dimension을 줄입니다. spatial information을 모으기 위해, average-pooling이 많이 사용되었습니다. 

먼저 feature map의 spatial information을 average-pooling과 max-pooling operation을 동시에 사용하고, $F^C_{avg}$와 $F^C_{max}$를 generating하여 모읍니다. 두 descriptor는 shared network에 들어가 channel attention map인 $M_c \in \mathbb{R}^{C\times 1 \times 1}$를 만듭니다. Shared Network는 MLP입니다. parameter overhead를 줄이기 위하여, hidden activation 크기를 $\mathbb{R}^{C/r\times1\times1}$ (여기서 $r$은 reduction ratio입니다.)로 설정하였습니다. shared network 뒤에는 element wise summation을 사용하여 output feature vecotr를 통합합니다. 정리하자면, channel attention은 아래와 같이 계산됩니다. 

<center>

$M_c(F)=\sigma(MLP(AvgPool(F))+MLP(MaxPool(F)))$

<p>

</p>

$\sigma(W_1(W_0(F^c_{avg}))+W_1(W_0(F^c_{max}))),$

</center>

여기서 $\sigma$는 sigmoid function이고, $W_0\in\mathbb{R}^{C/r\times C}$, $W_1\in\mathbb{R}^{C\times C/r}$입니다. MLP 가중치와 $W_0$, $W_1$은 두 input을 공유하며, ReLU 활성화 함수가 $W_0$ 이후에 있습니다.

---

#### Spatial attention module

본 논문은 spatial attention map을 feature의 inter-spatial relationship을 활용하여 만들었습니다. channel attention과 달리, spatial attention은 '어디'라는 informative part에 집중합니다. spatial attention을 계산하기 위하여, 먼저 average-pooling과 max-pooling을 channel 축에 따라 적용하고 효율적인 feature descriptor를 만들어내기 위하여 합칩니다. pooling을 channel 축에 따라 적용하면, informative region을 하이라이팅하는데에 효과적입니다. feature descriptor를 합치면, convolution layer를 활용하여 강조(emphasize)할 부분과 축소(suppress)할 부분을 encode하는 spatial attention map $M_S(F)\in R^{H\times W}$를 생성합니다. 

feature map의 channel information을 두 pooling operation을 활용하여 모으고, 두 2D maps: $F^s_{avg} \in \mathbb{R}^{1\times H \times W}$와 $F^s_{max} \in \mathbb{R}^{1\times H \times W}$를 생성합니다. 각각은 average-pooled feature와 max-pooled feature을 의미합니다. 두 feature은 나중에 합쳐지고 convolved되며 2D spatial attention map을 생성합니다. 즉, 간단히 말하면 아래와 같이 계산됩니다.

<center>

$M_s(F)=\sigma(f^{7\times7}([AvgPool(F);MaxPool(F)]))$

<p>

</p>

$=\sigma(f^{7\times7}([F^s_{avg};F^s_{max}])),$

</center>

여기서 $\sigma$는 sigmoid function을 의미하며 $f^{7\times7}$은 7 x 7 크기의 filter로 convolution operation한 것을 의미합니다. 

---

#### Arrange of attention modules

input image가 주어지면, 두 attention module인 channel, spatial은 attention을 계산하여 '무엇'과 '어디'라는 point에 집중합니다. 이것을 고려하여, 두 모듈은 병렬적으로나 직렬적인 방법으로 위치합니다. 본 논문은 직렬적인 위치가 병렬적인 위치보다 더 나은 결과를 낳는다는 것을 발견하였습니다. 또한 channel을 먼저 두는 것이 spatial을 먼저 두는 것보다 더 좋은 결과를 낳는 것을 발견하였습니다. 

---

### Experiments

Benchmark인 ImageNet-1k를 classification으로 사용하였습니다. 

<p align="center"><img src="../../assets/images/100909.jpg" width="500px" height="500px" title="CBAM" alt="CBAM" ><img></p>

<p align="center"><img src="../../assets/images/100910.jpg" width="500px" height="500px" title="CBAM" alt="CBAM" ><img></p>

<p align="center"><img src="../../assets/images/100911.jpg" width="500px" height="500px" title="CBAM" alt="CBAM" ><img></p>

<p align="center"><img src="../../assets/images/100912.jpg" width="500px" height="500px" title="CBAM" alt="CBAM" ><img></p>

<p align="center"><img src="../../assets/images/100913.jpg" width="500px" height="500px" title="CBAM" alt="CBAM" ><img></p>

<p align="center"><img src="../../assets/images/100914.jpg" width="500px" height="500px" title="CBAM" alt="CBAM" ><img></p>

---

### Conclusion

CBAM이라는 convolutional bottleneck attention module을 만들어, CNN network의 representation power를 향상시켰습니다. Attention-based feature인 두 모듈, channel과 spatial을 가지고 overhead는 적게 유지하면서 성능을 향상시켰습니다. '무엇'과 '어디'라는 초점을 강조시키고 축소시켜 효율적으로 intermediate feature을 조정하도록 CBAM은 학습합니다. CBAM이 여러 benchmark 보다 성능이 뛰어나다는 것을 실험을 통해 확인하였습니다. 