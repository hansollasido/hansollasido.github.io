---
title: "Faster R-CNN 리뷰"
excerpt: "Towards Real-Time Object Detection with Region Proposal Networks"

categories:
  - 컴퓨터 비전
tags:
  - [딥러닝, 논문 리뷰, Object Detection]

permalink: /categories9/review5/

use_math: true
toc: true
toc_sticky: true

date: 2023-04-21
last_modified_at: 2023-04-21
---

# 작성중(4/21)

# Fast R-CNN 

[R-CNN](https://hansollasido.github.io/categories9/review3/), [Fast R-CNN](https://hansollasido.github.io/categories9/review4/) 논문 리뷰를 먼저 보시는 것을 추천드립니다!!

---

### 간단한 소개

Faster R-CNN은 R-CNN의 region proposal에서의 시간을 획기적으로 줄이기 위해서 RPN(Region Proposal Networks)을 적용한 모델입니다. 

Faster R-CNN의 원리에 대해 알아보겠습니다.

---

### Introduction

Region proposal을 만들 때, selective search 방법이 있습니다. selective search 방법은 segmentation을 여러 번 진행하여 해당 segmentation에 맞는 region proposal만을 선택하는 형태로 진행합니다. R-CNN은 이런 region proposal을 2000개 만들고 각 proposal에 CNN을 적용합니다. 2000번의 CNN 연산을 진행하니 당연히 느리겠죠.

<p align="center"><img src="../../assets/images/042107.png" width="800px" height="800px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 1. selective search></center>


좀 더 빠르게 하고자 나온 모델이 Fast R-CNN, SPPnet입니다. Fast R-CNN은 RoI pooling layer로, SPPnet은 Spatial Pyramid Pooling layer로 CNN연산을 2000번에서 1번으로 줄여 연산 결과를 공유하는 식으로 모델 성능을 올렸죠. 하지만 Fast R-CNN이나 SPPnet은 항상 region proposal을 사용할 때, selective search를 이용해야 한다는 점이 있고 이는 시간적 소모가 많이 든다는 단점이 있었습니다. 

따라서 이 region proposal을 얻을 때의 방법을 개선하여 성능을 높인 모델이 Faster R-CNN입니다. 

Faster R-CNN은 Region Proposal Network(RPN)을 만들어 연산한 값을 공유하는 방식으로 성능을 향상시켰습니다. 여기서 anchor box라는 새로운 방법을 도입했는데 한 번 알아보겠습니다. 

---

### Faster R-CNN 

<p align="center"><img src="../../assets/images/042108.jpg" width="300px" height="300px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 2. Faster R-CNN 구조></center>

Faster R-CNN은 두 개의 모듈이 있는데, 하나는 region 생성을 위한 deep fully convolutional network와 다른 하나는 Fast R-CNN detector입니다. [Fast R-CNN](https://hansollasido.github.io/categories9/review4/)을 보시고 이해하시는 것을 추천드립니다.

<그림 2>에 나와있는 대로 RPN(Region Proposal Network) 모듈은 Fast R-CNN 모듈이 어디를 봐야하는지 알려주는 Network입니다. 이 때 [Attention machanism (1)](#추가설명)을 이용합니다. 

---

#### Region Proposal Networks(RPN)

<p align="center"><img src="../../assets/images/042109.png" width="800px" height="800px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 3. Faster R-CNN 구조(2)></center>

RPN은 어느 size의 image든 그 image를 input으로 받고 output으로 사각형 형태의 object proposal을 만듭니다. 이 과정을 fully convolutional network으로 진행하였습니다. 궁극적 목표는 Fast R-CNN network와 연산한 값을 공유하는 것이며 이를 위해 convolution layer를 RPN과 Fast R-CNN이 공유하게 됩니다. 

이 과정에서 5개의 공유 가능한 Convolutional layer을 가진 Zeiler와 Fergus 모델인 ZF와 13개의 공유 가능한 Convolution layer를 가진 Simonyan과 Zisserman 모델의 VGG-16을 이용합니다. 이 모델을 사용해 feature map을 생성하고 RPN의 input으로 사용합니다. ZF는 256-d, VGG는 512-d의 channel을 갖습니다. 

---

###### Anchors

<p align="center"><img src="../../assets/images/042110.png" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 4. Anchor 생성></center>

VGG 모델을 사용했다고 가정하고 h x w x 512의 input이 들어갔다고 생각해보겠습니다. 이 때 RPN에서 n x n(논문에서는 n = 3) convolution을 진행한 뒤, 두개의 sibling 1x1 convolution layer를 거칩니다. 이 때, 사용하는 bounding box 기법이 anchor 입니다. anchor은 <그림 4>처럼 scale과 비율을 정해, 해당 픽셀에서 k개의 bounding box를 만듭니다. 

<p align="center"><img src="../../assets/images/042111.png" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 5. Anchor 생성(2)></center>



---

### 참고 사이트

[selective search(1)](https://velog.io/@ym980118/%EB%94%A5%EB%9F%AC%EB%8B%9D-Object-Detection-%EA%B0%9D%EC%B2%B4-%EC%9D%B8%EC%8B%9D-selective-search)

[selective search(2)](https://better-tomorrow.tistory.com/entry/Selective-Search-%EA%B0%84%EB%8B%A8%ED%9E%88-%EC%A0%95%EB%A6%AC)

[selective search(3)](https://m.blog.naver.com/laonple/220918802749)

[Faster R-CNN 논문 리뷰](https://herbwood.tistory.com/10)

그리고 ChatGPT

---

### 추가설명

**(1) Attention machanism**
- 스퀀스(sequence) 데이터를 처리할 때, 특정 위치의 입력값에 더 많은 가중치를 부여하여 해당 위치가 출력값에 미치는 영향력을 강화하는 방법임.