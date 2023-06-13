---
title: "Feature Pyramid Networks"
excerpt: "Feature Pyramid Networks for Object Detection"

categories:
  - 컴퓨터 비전
tags:
  - [딥러닝, 논문 리뷰, Object Detection]

permalink: /categories9/review10/

use_math: true
toc: true
toc_sticky: true

date: 2023-06-12
last_modified_at: 2023-06-12
---

# 작성중 (6/13)

### Abstract

FPN (Feature Pyramid Networks)를 소개합니다. pyramid 구조는 compute 과정이 많고 memory intensive한 특징이 있어 최근 deep learning 부분에서 꺼려하고 있는 구조입니다. 본 논문은 multi-scale, 피라미드 구조의 deep convolutional network를 소개합니다. 모든 크기에서 high-level semantic feature maps를 만드는 lateral connection을 가지고 top-down 구조를 개발하였습니다. 본 논문은 이 구조를 Feature Pyramid Network(FPN)이라 부릅니다. FPN은 Faster RCNN 시스템을 사용하였고 GPU에 6 FPS의 속도로 동작하는 것을 확인하였습니다. 그러므로 multi-scale object detection에 실용적이고 정확한 해결책이 될 수 있습니다. 

---

### 소개





