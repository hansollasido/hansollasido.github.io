---
title: "Swin Transformer"
excerpt: "Swin Transformer: Hierarchical Vision Transformer using Shifted Windows 리뷰"

categories:
  - 컴퓨터 비전
tags:
  - [딥러닝, 논문 리뷰, Transformer]

permalink: /categories9/review14/

use_math: true
toc: true
toc_sticky: true

date: 2023-08-03
last_modified_at: 2023-08-03
---

# Abstract

NLP에서 사용되던 Transformer를 Vision에 사용되는 데에는 몇 가지 Challenge들이 존재합니다. NLP와 Vision은 다른 도메인이기 때문이죠. NLP는 text word를 input으로 취하지만 Vision은 수많은 pixel 개수를 input으로 취합니다. 이러한 차이점에 의한 Challenge들을 해결하기 위해 본 논문에서는 Swin Transformer를 만들었습니다. Swin Transformer은 Shifted Windows Transformer의 약자로, Shifted Window를 사용하여 계산하는 계층적 Transformer입니다. 

오늘은 이런 Swin Transformer에 대해 알아보겠습니다. 

---

### Introduction

Computer vision은 CNN이 거의 주로 다뤄졌습니다. NLP는 반대로 Transformer가 요즘 많이 핫하죠. Transformer가 굉장히 효과가 좋으니 이것을 Computer vision에 어떻게 적용할지에 대한 논문들이 끊임없이 쏟아져 나왔습니다. 하지만 도메인이 다른 NLP의 Transformer를 Computer Vision에 적용하는 것이란 매우 어려운 일이었죠. 특히나 다른 점은 크기입니다. NLP의 Transformer은 word token을 사용했기 때문에, object detection과 같은 task에서는 시각적 요소가 규모차원에서 굉장히 다를 수 있습니다. 기존 Transformer-based 모델은 고정된 크기의 token이기 때문에, vision application에 적합하지는 않습니다. 다른 차이점으로는 image의 높은 pixel 해상도입니다. 해상도가 높으면 Transformer는 매우 다루기가 힘들어집니다. 특히 image는 2차원 사이즈이기 때문에 computational complexity가 굉장히 높은 편이죠.

본 논문은 Transformer의 적용성을 확대하여 Computer Vision의 general-purpose backbone을 만드려 합니다. 

<p align="center"><img src="../../assets/images/080301.png" width="400px" height="400px" title="Swin Transformer" alt="Swin Transformer" ><img></p>

Figure (1)를 보면 Swin Transformer은 작은 사이즈의 patch로 부터 시작하여 계층적으로 올라가는 것을 볼 수 있습니다. 점차 깊은 층에 가면갈수록 이웃하는 patch들과 합쳐지게 되죠. computational complexity가 self-attention을 계산하면 선형적으로 됩니다. patch의 개수가 각 window에 고정되었기 때문이죠. 이러한 계층적 구조는 Swin Transformer를 general-purpose backbone으로 적합하게 만들어주며, single resolution feature map을 생산하거나 2차 복잡도를 가진 구조에 대해서도 잘 동작하게 됩니다. 

Swin Transformer의 주요 설계 요소는 Figure (2) 처럼 연속적인 self-attention layer사이에 window partition이 shift된다는 점입니다.

<p align="center"><img src="../../assets/images/080302.png" width="400px" height="400px" title="Swin Transformer" alt="Swin Transformer" ><img></p>

이전 layer에서 shift된 window는 modeling power를 높입니다. 이러한 전략은 한 window 내에 있는 모든 query patch가 같은 key set을 공유하기 때문에 실제 latency에서도 효과적입니다. 대조적으로 이전의 sliding window로의 self-attention 방법은 다른 query pixel에 대한 다른 key set으로 인해 genearl hardware에서의 low latency가 문제였습니다. 본 논문은 slinding window 방법보다 shifted window가 lower latency인 것을 증명하였습니다. 심지어 모든 multi-layer 구조에서도 효과적인 것을 볼 수 있었습니다.

Swin Transformer은 ViT/DeiT, ResNe(X)t와 같은 모델보다도 성능이 뛰어난 것을 볼 수 있었으며, image classification, object detection, semantic segmentation과 같은 여러 분야에서도 높은 성능을 볼 수 있었습니다. 

---

### Method

#### Overall Architecture

<p align="center"><img src="../../assets/images/080303.jpg" width="800px" height="800px" title="Swin Transformer" alt="Swin Transformer" ><img></p>

전체적인 구조는 figure 3과 같습니다. tiny version인 Swin-T라는 것만 참고해주세요. 먼저 RGB image를 non-overlapping patch로 분할합니다. ViT처럼요. 각 patch는 token으로 정의되며 pixel RGB 값의 연속된 집합이라는 특징을 가지고 있습니다. 

Self-attention computation으로 수정된 여러 Transformer block들은 이 patch token에 적용됩니다. $\frac{H}{4} \times \frac{W}{4}$를 유지하며 linear embedding을 거칩니다.

여기서 $\frac{H}{4} \times \frac{W}{4}$는 token의 개수를 의미합니다. token의 개수는 network가 깊어지면 깊어질수록 patch merging layer에 의해 감소됩니다. token의 개수는 $2 \times 2 = 4$씩 감소하며, output dimension은 2C로 setting됩니다. 여기까지가 figure (3)의 Stage 2까지의 내용이었고, Stage 3과 4는 Stage 2와 같이 반복해서 진행됩니다. 최종적으로 $\frac{W}{32}\times\frac{H}{32}\times 8C$가 되겠네요. 

**Swin Transformer block**
- Swin Transformer은 multi-head self attention (MSA) 모듈이 shifted window 기반 모듈로 대체되며 만들어집니다. Figure (3)에서 보면, Swin Transformer block은 shifted window based MSA 모듈로 구성되어 있으며, 2개의 MLP와, 그 사이에 GELU라는 비선형 함수가 있습니다. LayerNorm (LN) layer은 각 MSA 모듈과 각 MLP 모듈 이전에 적용되며, 각 모듈 이후에는 residual connection이 적용됩니다. 

---

#### Shifted Window based Self-Attention

기존 Transformer 구조와 image classification을 위한 Transformer은 둘 다 global self-attention을 만듭니다. 이러한 global computation은 token 개수와 관련하여 2차 복잡도를 만듭니다. prediction을 위한 막대한 token set이 필요하고, 고 해상도 image를 표현하기 위해서나 prediction을 하기 위해서나 막대한 token set을 필요로 해서 많은 vision 분야에 적용하기 어렵습니다.

**Self-attention in non-overlapped windows**
- 효율적인 모델링을 하기 위해서 본 논문은 local window 내에 self-attention을 계산하는 방법을 공개합니다. 겹치지 않게 window를 만들고 image를 분할합니다. 

<center>
$\Omega(\mbox{MSA})=4hwC^2+2(hw)^2C$
</center>
<center>
$\Omega(\mbox{W-MSA})=4hwC^2+2M^2hwC$
</center>
- 위 식의 계산 복잡도를 가집니다. 이전의 MSA는 $hw$의 2차 함수 형태로 복잡도를 가졌던 반면, window-based MSA는 hw의 선형적 함수 형태로 복잡도를 가지는 것을 볼 수 있습니다. Global self-attention computation은 큰 hw에 대해서는 일반적으로 계산하기 어렵지만 window-based self-attention은 크기가 작아 계산하기 쉽습니다. 

**Shifted window partitioning in successive blocks**

<p align="center"><img src="../../assets/images/080401.png" width="500px" height="500px" title="Swin Transformer" alt="Swin Transformer" ><img></p>

위와 같은 동작으로 figure (3)의 (b)를 수행합니다. 이 때 이전에 말씀했던 것 처럼 shifted window partitioning을 진행합니다. 

<p align="center"><img src="../../assets/images/080402.png" width="500px" height="500px" title="Swin Transformer" alt="Swin Transformer" ><img></p>

Shifted window partitioning을 진행할 경우, 더 많은 window가 생겨날 수 있습니다. $\lceil\frac{h}{M}\rceil\times\lceil\frac{w}{M}\rceil$에서 $(\lceil\frac{h}{M}\rceil + 1)\times(\lceil\frac{w}{M}\rceil + 1)$ 으로 바뀝니다. 그리고 몇몇 window는 $M \times M$ 사이즈보다 더 작을 수 있습니다. 이러한 문제를 해결하고자, cyclic shift를 진행합니다. 위 그림 figure (4) 처럼 cyclic shift를 진행하면, 인접하지 않은 patch까지 window에 담을 수 있으면서 window 수와 크기가 regular window와 같아집니다. 이는 매우 효과적입니다. 

---

#### Architecture Variants

Swin-B라는 base model을 설계하였습니다. 이는 ViT-B/DeiT-B와 비슷한 computation complexity와 model size를 가지고 있습니다. Swin-T, Swin-S, Swin-L은 각각 0.25배, 0.5배, 2배의 model size와 computation complexity를 가지고 있습니다. 본 논문에서는 아래와 같이 모델 variant를 정의하였습니다.

<p align="center"><img src="../../assets/images/080403.png" width="500px" height="500px" title="Swin Transformer" alt="Swin Transformer" ><img></p>

여기서 C는 channel 수를 말합니다. 

---

### 실험 결과

<p align="center"><img src="../../assets/images/080404.png" width="300px" height="300px" title="Swin Transformer" alt="Swin Transformer" ><img></p>

ImageNet에 대하여 classification을 진행한 결과, Swin Transformer가 비슷한 image size에서 가장 정확도가 높은 것을 볼 수 있었습니다. 

<p align="center"><img src="../../assets/images/080405.png" width="300px" height="300px" title="Swin Transformer" alt="Swin Transformer" ><img></p>

Object Detection 분야에서도 ResNe(X)t backbone보다 Swin backbone이 정확도가 높은 것을 볼 수 있었습니다. 

<p align="center"><img src="../../assets/images/080406.png" width="300px" height="300px" title="Swin Transformer" alt="Swin Transformer" ><img></p>

Semantic segmentation에서도 마찬가지로 Swin Transformer가 정확도 부분에서 높은 것을 볼 수 있었습니다. 

<p align="center"><img src="../../assets/images/080407.png" width="300px" height="300px" title="Swin Transformer" alt="Swin Transformer" ><img></p>

<p align="center"><img src="../../assets/images/080408.png" width="300px" height="300px" title="Swin Transformer" alt="Swin Transformer" ><img></p>

Shifted Window가 있고 없고의 차이를 보여주는 결과입니다. 확실히 shifted window가 좋아보이네요. 

---

### 결론

본 논문은 Swin Transformer에 대해 소개하고 있습니다. 새로운 Vision Transformer이면서 계층적 feature를 가지고 있습니다. 또한 input image size에 대해 선형적인 computational complexity를 가지고 있어 2차 computational complexity였던 기존 Transformer과는 다른 방법입니다. Swin Transformer은 object detection, semantic segmentation 등등 여러 분야에서 state-of-art performance를 지니고 있으며, 이러한 Swin Transformer가 language, vision을 혼합한 모델을 촉진할 것으로 기대됩니다. 

Swin Transformer의 주요 구성요소로 *shifted window-based self-attention*가 있는데요, vision problem에 대하여 효과적이고 효율적으로 접근하고 있습니다. 또한 NLP에서도 잘 적용될 것이라 기대됩니다. 