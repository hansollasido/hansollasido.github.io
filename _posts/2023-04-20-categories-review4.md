---
title: "Fast R-CNN 리뷰"
excerpt: "Fast R-CNN"

categories:
  - 컴퓨터 비전
tags:
  - [딥러닝]

permalink: /categories9/review4/

use_math: true
toc: true
toc_sticky: true

date: 2023-04-20
last_modified_at: 2023-04-20
---

# 작성중(4/20)

# Fast R-CNN

R-CNN 논문 리뷰를 먼저 보시는 것을 추천드립니다!!

#### 간단한 소개

object detection을 위한 Convolutional Network method에 대해서 얘기하고 있습니다. 학습과 test하는 속도, 그리고 정확도 부분에서 성능이 향상되는 몇 가지 혁신적인 부분을 담고 있습니다.

VGG16 네트워크 구조에서 Fast R-CNN은 R-CNN보다 9배 빠르며, 테스트 속도에서는 213배 빠릅니다. 오늘 논문 이름에서도 볼 수 있듯이 Fast R-CNN이 왜 빠른지 알아보겠습니다.

---

#### Instroduction

deep [ConvNets (1)](#추가설명)는 image classification과 object detection 정확도를 향상시킨 모델입니다. image classification과 달리 object detection은 localization에 의해 complexity가 매우 높습니다. 이러한 complexity 때문에, 현재의 학습 모델은 [multi-stage (2)](#추가설명)이라 속도가 느리고 매력적이지 못합니다. 

이런 복잡성은 localization에 의해 발생하게 되는데, 이는 두 가지 challenge를 낳습니다. 첫 번째는 무수히 많은 후보 proposal를 처리해야한다는 점, 두 번째로는 이 후보군을 정확한 localization을 수행해야 한다는 점입니다. 이 문제를 해결하기 위해서는 속도, 정확성, 간단함이 동반되어야 합니다. 

이에 본 논문은 single-stage 학습 알고리즘을 도입하여 proposal을 학습해보았습니다. 

---

##### R-CNN과 SPPnet

Region-based Convolutional Network(R-CNN)은 object detection에서 Convolution Network을 사용하여 높은 정확도를 얻었습니다. 하지만 R-CNN에 몇 가지 단점이 있습니다.

1. multi-stage pipeline
2. 학습이 공간상, 시간상 많은 소모가 필요하다는 점
3. detection하는 속도가 느림

R-CNN은 Convolution Network가 계산한 값을 공유하지 않고 진행하기 때문에, 느립니다. 이에 [SPPnets (3)](#추가설명)이 도입되어 계산한 값을 공유하여 속도를 높였습니다. 

SPPnet은 CNN 구조가 고정된 입력 이미지 크기를 입력으로 취하는 데에서 발생한 문제점을 개선하기 위해 고안되었습니다. 특히 R-CNN에서는 227x227 input 사이즈를 위해 warp시켜 이미지를 휘게 만드는데, 이와 같은 warp를 진행하면 전체 이미지 정보가 손실되어 계산한 값이 공유가 안되고 정보가 변형이 된다는 문제가 발생합니다. 

<p align="center"><img src="../../assets/images/042001.png" width="500px" height="300px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 1. crop/warp 제거></center>

고정된 input 사이즈가 필요한 부분은 사실상 Convolution layer가 아닌 Fully connected layer입니다. 따라서 SPPnet은 Spatial Pyramid Pooling layer를 추가하여 임의의 사이즈로 입력을 취할 수 있게 만들어줍니다. 

<p align="center"><img src="../../assets/images/042002.png" width="500px" height="300px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 2. R-CNN, SPP-net 비교></center>

R-CNN은 각 proposal에 CNN을 적용하기 때문에, 무수히 많은 proposal을 CNN에 적용시켜 많은 시간이 소요됩니다. 하지만 SPPnet은 CNN을 이미지에 한 번만 적용하여 속도가 빠릅니다. 즉, input image 전체에 대한 Convolution을 진행하기 때문에 계산한 값이 공유된다는 점이 SPPnet의 장점입니다. 

하지만 이러한 SPPnet도 단점이 있습니다. R-CNN처럼 multi-stage라는 점입니다.(log loss, SVM, bounding-box regressor 등을 사용) 또한 R-CNN과는 다르게 spatial pyramid pooling을 update할 수 없다는 점도 단점이 될 수 있습니다. 이러한 제약 때문에, deep network의 정확성 또한 낮아지게 됩니다. 

이에 본 논문에서는 R-CNN과 SPPnet의 단점을 개선한 새로운 학습 알고리즘을 개발하였습니다. 바로 **Fast R-CNN**입니다. Fast R-CNN은 몇 가지 장점이 있습니다.

1. R-CNN, SPPnet보다 더 높은 mAP를 가집니다. 
2. multi-task loss를 사용하여 single-stage 학습을 진행합니다.
3. 모든 network layer를 update할 수 있습니다.
4. [feature caching (4)](#추가설명)하기 위한 disk 저장이 필요 없어집니다.

Python과 C++로 구현된 Fast R-CNN 코드를 첨부합니다. [깃허브 링크](https://github.com/rbgirshick/fast-rcnn)

---

#### Fast R-CNN 구조와 학습방법

전체 image와 proposal set을 input으로 정의합니다. 그 다음 전체 image를 여러 convolution과 pooling layer를 거쳐 conv feature map을 형성합니다. 이후 RoI pooling layer로 feature map으로 부터 feature vector을 추출합니다. 각 feature vector은 fully connected layer로 들어가게 되며 두 개의 output을 산출합니다. 

output 중 하나는 object class에 대한 softmax 확률과 [catch-all background class (5)](#추가설명)을 더한 output이고 다른 하나는 네 개의 실수형 수인데, 이 수는 bounding-box의 위치를 표현합니다. 

<p align="center"><img src="../../assets/images/042003.jpg" width="500px" height="300px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 3. Fast R-CNN 구조></center>


---

### 참고 사이트

[SPPnet](https://deep-learning-study.tistory.com/445)


---


### 추가설명

**(1) ConvNets**
- Convolutional Neural Network의 약자. 합성곱 연산을 사용하여 입력 데이터에서 특징 맵을 생성함.

**(2) multi-stage**
- 여러 단계로 이루어진 모델링 접근 방법을 의미함. 복잡한 작업을 수행하는 데 필요한 다양한 기능을 각각 다른 모델로 분리하여 처리하는 것을 의미함. 이러한 다단계 접근 방식은 높은 정확성을 달성하는 데 도움이 될 수 있음. 이전 단계의 추출된 특징이 이후 단계에서 재사용될 수 있으며 이를 통해 예측 속도가 향상될 수 있음. 

**(3) SPPnets**
- Spatial Pyramid Pooling Network의 약자로, 기존의 CNN에서 발생하는 문제점 중 하나인 입력 크기가 일정해야 한다는 문제를 해결하기 위해 제안됨. SPPnet은 고정된 크기의 입력 이미지에서 Spatial Pyramid Pooling을 사용하여, 다양한 크기의 이미지에 대해서도 효과적으로 작동할 수 있도록 설계됨. 
  - Spatial Pyramid Pooling은 입력 이미지를 여러 개의 크기와 비율로 나누어 각 영역에서 최대 풀링 연산을 수행하여, 각 영역에서 추출된 특징을 결합함.

**(4) Feature caching**
- 모델의 특징 맵(feature map)을 미리 계산하여 저장해놓는 기법. 입력 이미지를 전체적으로 처리하기 보다는 작은 영역들로 나누어서 처리함. 각 영역에 대한 특징 맵을 미리 계산해서 저장해놓기 때문에, disk storage가 필요하게 됨. 

**(5) catch-all background class**
- object detection에서 사용되는 클래스 중 하나. 단순한 background class와 다르게 모든 객체가 아닌 부분을 나타냄. 보통 객체 검출 작업에서 입력 이미지에 존재하는 모든 객체를 각각의 클래스로 지정하고, 이외의 부분은 배경 클래스로 지정함. 하지만 모든 객체를 미리 지정하지 않고 모르는 객체를 일괄적으로 처리하기 위해 catch-all background class를 사용함. 