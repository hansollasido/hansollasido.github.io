---
title: "ViT"
excerpt: "An Image is worth 16x16 words: Transformers for image recognition at scale 리뷰"

categories:
  - 컴퓨터 비전
tags:
  - [딥러닝, 논문 리뷰, Transformer]

permalink: /categories9/review12/

use_math: true
toc: true
toc_sticky: true

date: 2023-07-31
last_modified_at: 2023-07-31
---

# 배경

Transformer 구조는 NLP에서 많이 사용되어 왔습니다. 하지만 Transformer를 Computer vision에 사용하기란 제한이 있었죠. Vision에서는 attention이 CNN과 결합되어 적용되었거나, 전체적인 구조를 유지한체 CNN을 대체하는 데에 사용되어 왔습니다. 

본 논문은 CNN의 불필요성에 대해 얘기하고, image patch의 연속된 집합에 직접적으로 적용된 transformer가 image classification에 얼마나 잘 작동하는 지를 보여줍니다. 본 논문의 **Vision Transformer (ViT)**는CNN의 state of the art에 비교하여 성능이 뛰어나고, 학습하는 데에 computational resource가 덜 요구된다는 장점이 있습니다.

오늘은 ViT에 대해 알아보겠습니다.

### Introduction

Self-attention-based 구조는 NLP 분야에서 잘 쓰이지만 computer vision 분야에서는 ResNet과 같은 convolutional network가 잘 쓰입니다. 최근에 CNN 구조에 self-attention을 결합한 다양한 연구들이 있었지만, 특정 attention 패턴 사용으로 인한 HW 가속기로 인해 효율적으로 scale되지 못한다는 제한이 있었습니다. 

NLP에 사용되는 Transformer은 효율적으로 scaling합니다. 이에 영감을 받아, 표준 Transformer를 image에 적용해보고, 최소한의 수정 단계를 거쳐보는 실험을 진행하였습니다. 이렇게 하기 위해서는 image를 patch로 분할하고 이 patch의 선형 embedding 집합을 Transformer의 input으로 정의하였습니다. Image patch는 NLP에서 token과 같은 맥락으로 보시면 됩니다. 

강한 규제 없이 ImageNet과 같은 mid-size dataset에 학습을 시켰을 때, 모델은 ResNet과 비교할만한 사이즈에서 조금 더 낮은 퍼센트의 적절한 정확도를 보여줬습니다. 이러한 결과는 Transformer가 CNN에 내재되어 있는 **귀납적 편향 (Inductive Biases)**를 가지지 않는 다는 것을 의미합니다. **따라서 data의 양이 불충분할 때에 학습하면 일반화를 잘 못합니다.**

귀납적 편향 (Inductive Biases)은 무엇일까요?

편향 (Bias)과 분산 (Variance)의미 부터 보면 아래와 같습니다.

<p align="center"><img src="../../assets/images/073101.png" width="600px" height="600px" title="ViT" alt="ViT" ><img></p>

data가 퍼진 정도를 분산이라 하며, data가 target에 부터 얼마나 멀리 떨어져 있는가를 말합니다. 일반적으로 모델이 일반화되지 못하는 이유는 학습한 데이터에 편향만 생각하여, 새로운 데이터가 주어졌을 때의 편향까지 고려하지 않기 때문입니다. **귀납적 편향 (Inductive Bias)**은 주어지지 않은 입력의 출력을 예측하는 것입니다. 정확한 출력에 가까워질 수 있도록 가정을 추가합니다. CNN은 이미지가 지역적으로 얻을 정보가 많다는 것을 가정하고 만들어진 모델이고, Transformer은 Positional Embedding, Self-Attention 등과 같은 모든 정보를 활용한 모델입니다. 

**즉, Transformer은 CNN에 비해 귀납적 편향이 부족하여 데이터 양이 불충분할 경우, 일반화가 잘 되지 않는다는 문제점에 의해 한계가 있습니다.**

하지만, 14M-300M image 처럼 큰 dataset이라면 얘기가 달라집니다. 큰 dataset에 경우, Transformer가 귀납적 편향을 충분히 갖추게 됩니다. 본 논문의 **Vision Transformer (ViT)**는 적절한 규모에 전이학습되면 훌륭한 결과를 보여줍니다. 


### 방법

<p align="center"><img src="../../assets/images/073102.png" width="600px" height="600px" title="ViT" alt="ViT" ><img></p>

최대한 original Transformer(Vaswani et al., 2017)과 비슷하게 모델을 설계하였습니다.

#### VISION TRANSFORMER (ViT)

Figure (1)에서 묘사된 전체적인 모델을 살펴보겠습니다. 기존 Transformer는 embedding token을 1D 집합으로 input을 정의합니다. 2D image를 다루기 위해서, 본 논문은 이미지를 $x \in \mathbb{R}^{H \times W \times C}$ 에서 $x_p \in \mathbb{R}^{N \times (P^2 \cdot C)}$로 변경하였습니다. $(H,W)$는 원래 이미지의 크기를 말하며, $C$는 채널의 개수, $(P,P)$는 image patch의 크기를 말하며, $N = HW/P^2$로 patch의 개수를 의미합니다. 또 Transformer는 $D$라는 vector size를 나타내는 상수를 사용하는데, 이에 본 논문은 patch를 1D로 펴서 훈련가능한 linear projection으로 D차원을 맵핑하였습니다. 

NLP의 BERT의 class token과 비슷하게, 본 논문은 embedded patch 집합에 학습가능한 embedding을 추가하였습니다. 

<p align="center"><img src="../../assets/images/073103.png" width="600px" height="600px" title="ViT" alt="ViT" ><img></p>

결론적으로는 위의 식 (4)와 같이 y라는 클래스가 나오게 되는 것이죠. 

**Inductive bias**
- Vision Transformer는 CNN보다 더 적은 이미지 특정 귀납적 편향을 가지고 있습니다. CNN에서는 지역성과 translation equivariance가 전체 모델, 각각 층에 반영되어 있습니다. ViT에서는 오직 MLP 층에서만 지역성을 갖췄으며, translation equivariance합니다. 하지만 self-attention 층은 global하죠. 2D 이웃한 구조는 아주 조금 사용됩니다. 모델 시작할 때 image를 patch로 cutting하는 부분과 다른 크기의 이미지를 position embedding할 때 조정하기 위한 fine-tuning 시간 때에만 사용되죠. 그것 외에는 patch의 2D position에 관련한 어떠한 정보도 position embedding이 가지지 않습니다. 

**Hybrid Architecture**
- Raw image patch의 대안으로, input sequence는 CNN의 특징 맵으로부터 만들 수 있습니다. 이러한 hybrid model에서, patch embedding porjection E는 CNN 특징 맵으로 부터 추출된 patch에 적용됩니다. 

#### Fine-tuning and higher resolution

전형적으로, ViT를 큰 dataset에 대하여 전이학습을 하고, downstream task에 fine-tune을 진행합니다. 사전 학습된 모델의 prediction head부분을 제거하고 zero-initialized된 $D \times K$ feedforward 층을 추가합니다. 여기서 $K$는 downstream class의 개수를 의미합니다. pre-training하는 것보다 더 큰 이미지 크기에서 fine-tune하는 것이 종종 유익합니다. 더 큰 이미지 크기의 image가 주어졌을 때, 같은 사이즈의 patch를 유지합니다. 이를 통해 효과적으로 큰 sequence 길이를 만들 수 있죠. 

하지만, Vision Transformer는 보조 sequence 길이를 조정할 수 있어서 전이학습된 position embedding을 더이상 유의미하지 않게 만듭니다. 본 논문은 전이학습된 position embedding의 2D interpolation을 진행하여 수행하였습니다. 

### 실험

ResNet과 Vision Transformer(ViT), hybrid를 실험하고 평가하였습니다. 각 모델의 data 요구사항을 이해하기 위하여, 본 논문은 다양한 크기의 dataset에 사전학습을 시행하였고 수많은 benchmark를 통해 실험을 진행하였습니다. 

<p align="center"><img src="../../assets/images/080101.png" width="600px" height="600px" title="ViT" alt="ViT" ><img></p>

ViT가 다른 CNN보다도 정확도가 높은 것을 볼 수 있습니다. 

<p align="center"><img src="../../assets/images/080102.png" width="600px" height="600px" title="ViT" alt="ViT" ><img></p>

# 결론

본 논문은 NLP에서 쓰였던 Transformer를 image classification에 적용한 방법에 대해 서술하고 있습니다. 이전 연구들은 computer vision에서 self-attention을 사용했던 것과는 달리, 본 논문은 image 특징적인 귀납적 편향을 첫 번째 patch 추출 단계를 제외하고는 모델 구조에 적용하지 않았습니다. 대신에 NLP에서 사용되었던 표준 Transformer encoder로 patch를 만들었으며, 이 patch 집합으로 이미지를 분할하였습니다. 이 간단한 전략은 큰 dataset에 사전학습되었을 때 놀라운 결과를 보여줍니다. Vision Transformer은 image classification dataset의 sota와 같거나 능가하는 결과를 보여주고 있습니다. 

아직 많은 challenge가 남아 있습니다. 그 중 하나는, ViT를 다른 computer vision task에 적용하는 일입니다. object detection이나 segmentation과 같은 작업 말이죠. 또 다른 challenge로는 self-supervised pre-training 방법을 계속 이어나가야 한다는 점입니다. 본 논문의 첫번째 실험은 self-supervised pre-training으로 부터 좋은 결과를 보여줬지만, 아직 self-supervised와 large-scale supervised pre-training과의 큰 격차가 있습니다. 