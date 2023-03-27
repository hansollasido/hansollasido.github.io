---
title: "DETR 리뷰"
excerpt: "End-to-End Object Detection with Transformers"

categories:
  - 논문 리뷰
tags:
  - [Object Detection]

permalink: /categories9/review1/

toc: true
toc_sticky: true

date: 2023-03-23
last_modified_at: 2023-03-23
---

## DETR

DETR (End-to-End Object Detection with Transformers) 논문은 ECCV (유럽 컴퓨터 비전 학회)에 2020년 발표된 논문 입니다. SOTA (State of the Art) 들이 추후 DETR을 기반으로 한 만큼 중요한 알고리즘이라고 볼 수 있습니다.

### Abstract

이 논문에서는 직접 손으로 디자인해야하는 component들의 필요성을 효과적으로 줄여서 detection pipeline을 간소화하였습니다. DETR은 DEtection TRansformer의 약자로 bipartite matching (이분 매칭: 두 개의 집단의 정점이 서로 연결되며 같은 그룹 내의 정점은 서로 연결되지 못함)을 통한 Set-based global loss(객체 감지 모델에서 사용되는 손실 함수 중 하나)이며 encoder-decoder transformer 구조를 가지고 있습니다.

개념적으로 쉽고 다른 특이한 detector처럼 특별한 라이브러리가 필요없습니다.

DETR은 정확도나 run-time에서도 높은 성능을 보여주며 COCO dataset을 대상으로 한 Faster-R-CNN와 대등한 수준을 이룹니다.

