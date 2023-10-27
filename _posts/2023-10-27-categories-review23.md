---
title: "Visual Attention Network"
excerpt: "VAN"

categories:
  - 컴퓨터 비전
tags:
  - [딥러닝, 논문 리뷰]

permalink: /categories9/review23/

use_math: true
toc: true
toc_sticky: true

date: 2023-10-27
last_modified_at: 2023-10-27
---

(작성중 10/27)

### Abstract

self-attention은 원래 NLP를 위해서 설계되었지만 폭풍처럼 Computer Vision에서도 최근 사용되고 있습니다. 그러나 self-attention을 computer vision에 사용하면 세 가지 challenge가 있게 됩니다. (1) Image를 1D sequence로 취급하는 것은 2D 구조를 무시하는 것이며, (2) 2차 복잡도는 고해상도 image에서 매우 비싸다는 문제, (3) channel adaptability를 무시하면서 spatial adaptability를 가지는 문제가 있습니다. 본 논문은 새로운 linear attention인 **large kernel attention (LKA)**를 사용하여 단점을 피하면서 self-attention에서 self-adaptive와 long-range 상관관계를 가능케 하였습니다. 추가로, 본 논문은 LKA 기반으로 한 neural network (Visual Attention Network(VAN))을 제안합니다. 극도로 간단하지만, VAN은 비슷한 크기의 vision transformers(ViT)보다 Convolutional Neural Network (CNNs)을 image classification, object detection, semantic segmentation, panoptic segmentation, pose estimation의 다양한 task에서 성능이 좋습니다. 예를 들어, VAN-B6는 ImageNet benchmark에서 87.8%의 정확도를 달성하였습니다. Code는 [https://github.com/Visual-Attention-Network](https://github.com/Visual-Attention-Network)에 있습니다. 

---

### Introduction 

Computer Vision에서 AlexNet이 나오면서 deep learning 세계가 다시 펼쳐졌습니다. 더 깊은 신경망을 만드는 것이 유리해졌으며, 굉장히 powerful한 vision backbone이 만들어졌죠. 이후 NLP 분야에서 attention model, 특히 Transformer의 self-attention model이 나오면서 이 것을 vision 쪽에 사용하는 연구가 진행되었습니다. ViT (Vision Transformer)은 NLP에 사용했었던 Transformer을 Vision쪽에 사용한 model입니다. 이 ViT는 CNN보다 높은 성능을 보여주면서 Transformer 모델이 powerful한 모델이 됩니다. 

기억할만한 성공에도, convolution operation과 self-attention은 단점이 있습니다. Convolution operation은 static weight를 가지고 있다는 점과 adaptability가 부족하다는 점입니다. 원래 1D NLP에서 사용되었던 self-attention을 사용하여 2D image를 1D sequence로 취급하는 것 또한 2D 구조인 image의 정보를 파괴할 수 있다는 점도 단점입니다. 게다가 고해상도 image에서는 2차 복잡도와 memory overhead를 가지고 있죠. self-attention은 spatial dimension에서 adaptability만을 고려하는 특별한 attention이지만, channel dimension에서의 adaptability는 무시합니다. 하지만 channel dimension의 adaptability는 visual task에서는 중요한 것이죠. 

본 논문은 새로운 linear attention mechanism인, **large kernel attention(LKA)**를 제안합니다. LKA는 convolution과 self-attention의 local struction information, long-range dependence, 그리고 adaptability를 포함한 유익한 점을 이어갔습니다. 반면에, channel dimension에서의 adaptability를 무시하는 등의 안 좋은 점은 피하게 했습니다. LKA에 기반하여, 새로운 vision backbone인 Visual Attention Network (VAN)을 제안하며, 이는 잘 알려진 CNN-based와 transformer-based backbone을 능가하는 성능을 보여줍니다. 

<p align="center"><img src="../../assets/images/102701.png" width="700px" height="500px" title="VAN" alt="VAN" ><img></p>

---
 
### Method

#### Large Kernel Attention



---


### 추가설명

**(1) spatial dimension & channel dimension**

- spatial dimension은 이미지나 데이터의 폭과 높이와 같은 공간적 차원을 말함. 
- channel dimension은 데이터의 깊이를 나타내며, 예를 들어 RGB 이미지는 3개의 채널로 구성됨. 
