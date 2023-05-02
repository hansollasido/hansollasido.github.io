---
title: "Object Detection-Based Video Retargeting With Spatial-Temporal Consistency 리뷰"
excerpt: "Video Retargeting 기술"

categories:
  - 컴퓨터 비전
tags:
  - [딥러닝, 논문 리뷰, Object Detection, Video Retargeting]

permalink: /categories9/review6/

use_math: true
toc: true
toc_sticky: true

date: 2023-05-01
last_modified_at: 2023-05-01
---

# 작성중(4/21)

# Object Detection-Based Video Retargeting With Spatial-Temporal Consistency

---

### 간단한 소개

본 논문은 Object Detection (사물 인지)를 기반으로 한, deep neural network를 통해 [video retargeting (1)](#추가설명) 하는 기법을 얘기하고 있습니다. 

보통 Video Retargeting을 진행하면 distortion(왜곡)이 일어납니다. 왜곡되어야 하는 부분과 왜곡 되지 않아야 하는 부분을 구별하는 것이 video retargeting 기술의 핵심 성능입니다.

이전 연구에서는 **단순한 Resizing**, **에너지 최적화 모델 기반 방법**, **DNN 기반 방법**으로 video retargeting하였습니다. **단순한 Resizing** 기법은 선형 scaling, cropping, inserting이어서 중요한 요소가 왜곡되고 제거되기도 하였습니다. **에너지 최적화 모델**에서는 Region of Interests(RoIs)를 고를 때 수 많은 경우의 수를 평가하는 용도로 [Salinency (2)](#추가설명)를 평가합니다. 이미지 픽셀의 밝기값, 색상의 강도 또는 주파수에 따라 유망한 region을 유도하는데 Saliency가 계산할 때 쓰입니다. 이렇게 단순히 RoI를 픽셀 단위와 저레벨에서 계산하는 것은 이미지를 보는 사람에게 인식되는 객체를 고려하지 않게 됩니다. 추가로 RoI와 RoI가 되지 않는 부분을 이분화할 수 없어서 방대한 왜곡이 생기게 됩니다. 그러므로 이미지 구조적인 부분에서 왜곡이 일어날 수밖에 없습니다. 

이러한 왜곡을 해결하기 위하여 전처리 등의 기술이 나왔지만, saliency dtection의 근본적인 문제를 해결하기에는 어려움이 있었습니다. 

**DNN 기반** 방법은 이미지 구조적 특징들은 유지한채, 중요한 부분의 왜곡을 최소화하는 식으로 진행되어 왔습니다. 하지만 ground truth이 없는 데이터 셋에는 지도학습이 불가능해지는 문제가 있었습니다. 

[InGAN (3)](#추가설명)은 [GAN (4)](#추가설명)을 사용하여 단순히 한 이미지에 있는 image patch의 특징 분포를 매칭시키는 학습을 진행합니다. GAN을 사용하면 patch가 재정돈되어 다른 aspect ratio에서도 이미지를 잘 구성할 수 있습니다. 하지만 이러한 deep learning 기법은 한 이미지를 target으로 한 모델이기 때문에 비디오처럼 연속적인 이미지에는 고려되지 않는 모델입니다.

이러한 기존의 문제점 때문에 새로운 기법이 필요하였습니다. 

본 논문은 object detection을 사용하여 video retargeting 방법을 제안합니다. 

### 간단한 구조

<p align="center"><img src="../../assets/images/050202.jpg" width="700px" height="700px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 1. Object Detection기반 Video Retargeting 구조></center>

간단한 구조는 위와 같습니다. object detection을 통해 RoI를 결정하고 저레벨 object detection보다도 DNN 기반의 object detection을 사용했습니다. object detection하고 난 다음, bounding box은 [Siamese networks (5)](#추가설명)기반한 object-tracking network를 거쳐 진행됩니다. 

Saliency 기법과 달리 Object Detection을 기반으로 한 기법은 object와 non-object를 정확하게 구분한다는 장점이 있습니다. RoI에 있는 부분은 변형없이 background area에 있는 부분을 resize합니다. background를 다른 ratio로 scaling할 경우, 아래 <그림 2>의 (b)와 같이 구조적, 시간적 왜곡이 생기게 됩니다. 

<p align="center"><img src="../../assets/images/050203.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 2. retargeting 기법 비교></center>

RoI는 object detection을 통해 이뤄지기 때문에, 에너지 최적화 기법처럼 추가적인 전처리가 필요 없게 됩니다. 

--- 

## 참고자료

[GAN](https://process-mining.tistory.com/169)

[Siamese Network](https://www.boostcourse.org/ai218/lecture/410030?isDesc=false)

---

## 추가설명

**(1) Video Retargeting**
- 비디오 콘텐츠의 크기와 비율을 자동으로 조정하여, 다양한 화면 크기와 비율의 기기에서 콘텐츠를 최적화하는 기술

**(2) Saliency**
- 사물의 중요도, 중심성이며 시각적 자극 중 어떤 부분이 인지 상태에서 더욱 주목받는 더욱 중요한 영역인지를 나타내는 지표

**(3) InGAN**
- Invertible Generative Adversarial Networks의 약자로, 생성적 적대 신경망(GAN)을 역함수 가능한 모델로 변환하는 방법론 

**(4) GAN**
- Generative Adversarial Network의 줄임말로, generative model의 한 종류

<p align="center"><img src="../../assets/images/050201.png" width="500px" height="300px" title="OP code 예시" alt="OP code" ><img></p>

- 실제 처럼 보이는 데이터를 생성하는 Generator와 실제 데이터와 만들어진 가짜 데이터를 구별하려는 Discriminator로 이루어져 있음

**(5) Siamese Network**
- 비교 분석을 위해 두 개의 입력을 받는 뉴럴 네트워크. 두 입력 사이의 유사성을 계산하기 위해 설계됨.