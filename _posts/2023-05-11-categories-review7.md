---
title: "Mask R-CNN"
excerpt: "Mask R-CNN 리뷰"

categories:
  - 컴퓨터 비전
tags:
  - [딥러닝, 논문 리뷰, Object Detection]

permalink: /categories9/review7/

use_math: true
toc: true
toc_sticky: true

date: 2023-05-11
last_modified_at: 2023-05-11
---

# 작성 중 (5/11)

# Faster R-CNN 

[Faster R-CNN](https://hansollasido.github.io/categories9/review5/)논문 리뷰를 먼저 보시는 것을 추천드립니다!!

## 간단한 소개

본 논문은 object instance segmentation을 위한 간단한 framework를 제안합니다. object detection task보다는 instance segmentation task에 주로 사용됩니다. 

<p align="center"><img src="../../assets/images/051104.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 1. Instance segmentation></center>

Instance segmentation을 위한 framework를 Faster R-CNN에 병렬적으로 붙인 것이 Mask R-CNN인데요. 해당 Network가 어떻게 적용되는지 알아보겠습니다.
  


---

## 추가 설명