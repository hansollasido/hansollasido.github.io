---
title: "AlexNet 리뷰"
excerpt: "ImageNet Classification with Deep Convolutional Neural Networks"

categories:
  - 컴퓨터 비전
tags:
  - [딥러닝, 논문 리뷰, Object Detection]

permalink: /categories9/review2/

use_math: true
toc: true
toc_sticky: true

date: 2023-03-29
last_modified_at: 2023-03-29
---

# AlexNet

#### 간단한 소개

AlexNet에 대해서 논문 리뷰를 시작해보겠습니다.

Krizhevsky et al이 고안한 AlexNet이 2012년에 *ILSVRC(ImgaeNet Large Scale Visual Recognition Challenge) ImageNet data를 기반으로 최종 우승하였습니다. 

8개의 층으로 구현된 AlexNet은 대회 최초로 Convolution Neural Network으로 구현하여 우승을 차지하였습니다.

<img src="../../assets/images/033001.jpg" width="800px" height="400px" title="OP code 예시" alt="OP code"><img><br/>

위 그림을 보시면 AlexNet 이전에는 매우 얕은 층이었지만 AlexNet에서는 8개 층으로 구현되기 시작하였고 error rate(y축)도 16.4%로 이전년도보다 약 10% 감소한 것을 볼 수 있습니다.

이런 AlexNet은 어떤 구조로 되어있으며, 획기적인 결과를 낳은 이유가 무엇인지 알아보겠습니다.

---

#### 데이터 준비

AlexNet은 ImageNet을 통해 학습을 진행하였습니다. ImageNet은 1000개 이상의 카테고리로 분류된 대규모 이미지 데이터셋으로, 약 150만장의 이미지와 이에 대한 레이블 정보를 포함하고 있습니다.

ImageNEt의 해상도가 다양해서 일정한 해상도로 만들 필요가 있었습니다. 따라서 256 x 256 사이즈로 일관되게 변경하였으며, RGB 색상 또한 구별하게 끔 설정하였습니다.

---

#### ReLU 사용

ReLU를 본격적으로 사용한 CNN입니다. 그동안 Activation function으로 tanh, sigmoid 함수를 많이 사용하였으나 deep neural network에서 발생하는 gradient vanishing 문제에 의해 ReLU가 주목받게 되었습니다.

<img src="../../assets/images/033001.png" width="400px" height="400px" title="OP code 예시" alt="OP code"><img><br/>

ReLU는 $max(0, x)$ 로 saturation이 x>0에서 발생하지 않아 좀 더 깊은 신경망층을 생성할 수 있습니다. 
([논문 참조](https://www.cs.toronto.edu/~fritz/absps/reluICML.pdf))

---

#### AlexNet의 구조

5개의 convolution layer, 3개의 fully-connected layer로 구성되어 있으며 정리하면 아래와 같습니다.

[227x227x3] INPUT 

**1st layer**
- [55x55x96] CONV1: 96 11x11 filters at stride 4, pad 0
- [27x27x96] MAX POOL1: 3x3 filters at stride 2
- [27x27x96] NORM1

**2nd layer**
- [27x27x256] CONV2: 256 5x5 filters at stride 1, pad 2
- [13x13x256] MAX POOL2: 3x3 filters at stride 2
- [13x13x256] NORM2

**3rd layer**
- [13x13x384] CONV3: 384 3x3 filters at stride 1, pad 1

**4th layer**
- [13x13x384] CONV4: 384 3x3 filters at stride 1, pad 1

**5th layer**
- [13x13x256] CONV5: 256 3x3 filters at stride 1, pad 1
- [6x6x256] MAX POOL3: 3x3 filters at stride 2

**6th layer**
- [4096] FC6: 4096 neurons

**7th layer**
- [4096] FC7: 4096 neurons

**8th layer**
- [4096] FC8: 1000 neurons (class scores)

<img src="../../assets/images/033002.jpg" width="800px" height="400px" title="OP code 예시" alt="OP code"><img><br/>

그림으로 본 AlexNet의 구조입니다. 좀 복잡해보이죠?

약간 이상한 점이 보입니다. AlexNet 구조를 보면은 위 아래 두개로 나눠져 있는 것을 볼 수 있습니다. 두 개로 나눈 이유가 무엇일까요?

AlexNet은 GPU를 하나만 사용하지 않고 두개의 GPU를 돌려 training 합니다. *GTX 580 GPU는 메모리 용량이 3GB이기 때문에 전체 network를 훈련하는데에는 제한이 됩니다. 따라서 GPU 두 개로 나누어 병렬적으로 훈련하게 만들었습니다. 이렇게 두 개의 GPU를 사용하면 메모리 전체 용량 제한에서 자유로워지고, 병렬적으로 연산하니 속도는 더 빨라지게 됩니다. 

2번째, 4번째, 5번째 convolution layer은 같은 GPU를 사용한 이전 층의 kernel map과만 연결되어 있으며, 그 외의 다른 층은 이전 층의 전체 kernel map과 연결되어 있습니다. 

---
#### 과대적합을 막기 위해

**Data Augmentation**
- 보통 과대적합을 막기 위해서는 충분한 dataset이 필요합니다. 이미지 dataset에서는 data augmentation 기법으로 이미 있는 데이터셋을 회전, 이동, 반전, 자르기 크기 변경 등으로 변형하여 데이터 양을 늘립니다. AlexNet에서도 data augmentation 기법을 적용하였습니다.

**Dropout**
- Dropout은 신경망을 인위적으로 끄는 방법으로, Dropout을 0.5으로 적용하여 과대적합을 막았습니다.

---
#### 이외의 parameter 값들

batch size 128, SGD momentum 0.9, learning rate 1e-2 (10배씩 줄어듭니다.), L2 규제 5e-4

---

#### 결과

AlexNet의 결과는 어떠할까요? 

_간단한 소개_ 목차에서 봤듯이 error rate가 약 10% 감소한 것을 볼 수 있었습니다.

<img src="../../assets/images/033003.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code"><img><br/>

위는 ILSVRC-2012 대회의 결과이며 아래는 2009년 가을 버젼의 ImageNet으로 test error rate을 포함한 결과입니다. 

<img src="../../assets/images/033004.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code"><img><br/>

정확도 부분에서 이전과 다르게 엄청난 결과를 낳았다고 봐도 무방할 것 같네요!

---
#### 정리

정리하자면 다음과 같습니다.

1. AlexNet은 ILSVRC 대회에서 최초로 쓰인 CNN 구조이다.
2. 이전 대회 우승자는 매우 얕은 층 구조를 사용하였지만 AlexNet 무려 8개의 층으로 구성되어 있다.
3. ReLU 함수를 처음으로 Activation function으로 이용한 사례이다.
4. GPU 용량 제한으로 인해 대규모 데이터셋에 문제가 있어 GPU 두 개를 사용하였다. 이를 병렬적으로 연산하여 속도와 정확도를 높였다.
5. 정확도가 이전 대회 우승자에 비해 획기적으로 올랐다.

---

지금까지 인공지능 분야에서 많이 들어봤을 법한 AlexNet 논문을 리뷰해보았습니다! VGGNet, ResNet등 학부수업 때 들은 CNN구조도 논문 리뷰도 진행해보겠습니다.
