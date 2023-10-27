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

### Abstract

self-attention은 원래 NLP를 위해서 설계되었지만 폭풍처럼 Computer Vision에서도 최근 사용되고 있습니다. 그러나 self-attention을 computer vision에 사용하면 세 가지 challenge가 있게 됩니다. (1) Image를 1D sequence로 취급하는 것은 2D 구조를 무시하는 것이며, (2) 2차 복잡도는 고해상도 image에서 매우 비싸다는 문제, (3) channel adaptability를 무시하면서 spatial adaptability를 가지는 문제가 있습니다. 본 논문은 새로운 linear attention인 **large kernel attention (LKA)**를 사용하여 단점을 피하면서 self-attention에서 self-adaptive와 long-range 상관관계를 가능케 하였습니다. 추가로, 본 논문은 LKA 기반으로 한 neural network (Visual Attention Network(VAN))을 제안합니다. 극도로 간단하지만, VAN은 비슷한 크기의 vision transformers(ViT)보다 Convolutional Neural Network (CNNs)을 image classification, object detection, semantic segmentation, panoptic segmentation, pose estimation의 다양한 task에서 성능이 좋습니다. 예를 들어, VAN-B6는 ImageNet benchmark에서 87.8%의 정확도를 달성하였습니다. Code는 [https://github.com/Visual-Attention-Network](https://github.com/Visual-Attention-Network)에 있습니다. 