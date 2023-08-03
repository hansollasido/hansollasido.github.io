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