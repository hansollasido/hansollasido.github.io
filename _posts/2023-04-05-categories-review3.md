---
title: "R-CNN 리뷰"
excerpt: "Rich feature hierarchies for accurate object detection and semantic segmentation"

categories:
  - 컴퓨터 비전
tags:
  - [딥러닝, 논문 리뷰, Object Detection]

permalink: /categories9/review3/

use_math: true
toc: true
toc_sticky: true

date: 2023-04-05
last_modified_at: 2023-04-13
---

# R-CNN (Regions with CNN features)

#### 간단한 소개

RCNN에 대해서 논문 리뷰를 시작해보겠습니다.

보통 [(1) PASCAL VOC 데이터셋](#추가설명) 으로 한 사물인지(Object Detection)에서 성능 향상 방법은 복잡한 [(2) 앙상블(Ensemble) 시스템](#추가설명)이 대부분 이었습니다. 

이 논문에서는 간단하고 크기 조절이 가능한 detection 알고리즘을 소개하고 있으며 [(3) mean average precision (mAP)](#추가설명)가 이전보다 30%가까이 오른 것을 결과로 보여주고 있습니다.

특히 이 논문은 R-CNN에 대해서 다룰 터인데, R-CNN은 CNN을 위치(region) propsals와 합한 알고리즘입니다. 

간단한 소스코드는 [http://www.cs.berkeley.edu/~rbg/rcnn](http://www.cs.berkeley.edu/~rbg/rcnn) 에 있으며 R-CNN은 200-class ImageNet Large Scale Visual Recognition Challenge (ILSVRC) 2013 데이터셋에서 [(4) OverFeat](#추가설명)를 능가하는 성능을 보여줍니다.

이제 본격적으로 R-CNN에 대해 알아보겠습니다.

---

#### Introduction

CNN으로 2012년에 ILSVRC에서 우승한 AlexNet은 object detection이 아닌 image classification입니다. image classification은 이미지를 입력으로 받아 이미지에 대한 클래스 레이블을 예측하는 반면, object detection은 분류는 물론 위치 까지 인식해야하는 복잡한 문제를 해결해야 합니다. ILSVRC 2012년에 우승한 CNN classification 결과를 PASCAL VOC Challenge에 object detection 결과에 일반화할 수 있는 방법이 없을까요? 그래서 이 논문은 image classification과 object detection간의 차이를 연결하여 이 문제에 대한 해답을 찾고자 합니다.

image classification과 달리 object detection은 한 이미지에 여러 사물을 분류하기 때문에 localization(지역화)이 중요합니다. 이에 이전 object detection은 localization을 sliding-window 기법으로 해결하였습니다. 하지만 sliding-window에 CNN을 적용할 경우, 너무 많은 receptive fields를 가지게 되고 매우 정확한 localization이 필요하여 sliding-window에 기술적 단점이 있게 되었습니다.

그래서 이 논문은 **CNN localization 문제**를 구간(region)을 나누어 인식(recognition)하게 하여 해결하였습니다. Region Proposal, 즉 이미지에서 객체가 위치한 후보 영역(Region of Interest, ROI)을 추론합니다. 원래는 이미지 전체를 통해 객체의 위치와 종류를 파악하였으나 이미지 내에서 객체가 위치할 가능성이 높은 후보 영역을 추론하였기 때문에 해당 영역에 대해서만 객체 검출을 수행합니다. 이렇게 되면 연산량과 시간 면에서 매우 효율적으로 변합니다. 

<p align="center"><img src="../../assets/images/040601.jpg" width="500px" height="300px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 1. R-CNN 구현 과정></center>

OverFeat의 sliding-window CNN 기술은 당시까지 ILSVRC2013에 가장 좋은 기술이었지만 R-CNN은 OverFeat을 능가하면서 mAP가 R-CNN은 31.4%, OverFeat은 24.3%으로 더 좋은 수치가 나온 것을 확인했습니다. 

localization 문제 말고도 다른 문제가 있었습니다. **labeled된 data가 적고 CNN을 훈련시키기 위한 충분한 양의 data가 없다는 문제**입니다. 이전에는 이러한 문제를 비지도 사전학습(unsupervised pre-training)으로 해결하였습니다. 하지만 이 논문에서는 지도 사전학습(supervised pre-training)을 large auxiliary dataset인 ILSVRC에서 진행하여 해결하였습니다. 즉, 큰 데이터셋으로 labeled된 이미지를 지도 사전학습 시킨 뒤, 이를 작은 데이터셋에 적용하여 학습 데이터양이 부족한 문제점을 해결했다는 말입니다. 비지도 사전학습보다 지도 사전학습으로 진행한 결과 mAP는 2010년 VOC 33%에서 54%로 향상된 것을 볼 수 있었습니다.

---

#### R-CNN으로 Object Detection하는 방법

크게 세 가지 모듈이 있습니다.

1. 카테고리에 의존하지 않는 region proposal을 생성합니다. 이 proposal은 detector가 탐지할 수 있는 후보군을 정의합니다.
2. 각 region에 [(5) feature vector](#추가설명)을 추출하는 CNN 구조가 있습니다.
3. 선형 SVM으로 클래스를 결정합니다.

<그림 1> 참고하시면 이해가 되실 겁니다.

일단 category-independent한 region proposals을 생성합니다. 각각의 region proposal을 이제 CNN 구조에 적용시켜야 하는데 문제는 CNN 구조의 input size가 227 x 227 RGB로 고정되어 있다는 점입니다. 

이에 R-CNN은 region proposal을 팽창시켜 227 x 227 size에 맞게끔 늘립니다. 

<p align="center"><img src="../../assets/images/040701.jpg" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 2. CNN input size를 맞추기 위해 region proposal을 변형시키는 방법들></center>

위의 <그림 2>을 보았을 때 original object proposal이 (A)와 같다면 이것을 (D)로 팽창시키는 과정이 있게 됩니다. input size가 227 x 227이 된다면 이제 CNN에 적용할 수 있습니다. 

약 2000개나 되는 region proposal을 CNN에 적용하면 feature vector가 결과값으로 나오게 됩니다. 이 feature vector을 선형 [(6) SVM](#추가설명)을 이용하여 class를 결정합니다. 

SVM을 거쳐 최종적으로 region이 나오게 되면 이 region을 [(7) greedy non-maximum suppression](#추가설명)으로 전처리합니다.

이와 같이 하면, 첫 번째로 CNN 파라미터가 모든 category 전역에 공유된다는 점과 두 번째로는 feature vector가 저차원 계산된다는 점이 있습니다. 이 두 가지 장점은 효율적인 detection이 되게끔 만듭니다. 특히 속도면에서 매우 효율적이죠. 계산해야할 feature 개수가 적어서 그렇습니다.

---

#### 결과

<p align="center"><img src="../../assets/images/041201.jpg" width="800px" height="400px" title="OP code 예시" alt="OP code" ><img></p>
<center><표 1. Detection average precision (%) on VOC 2010 test></center>

<표 1>을 보시면 PASCAL VOC 데이터셋에서 R-CNN이 높은 mAP를 보여주는 것을 볼 수 있습니다. 그 중에서도 Bounding-box regression (BB)가 가장 mAP가 높은데 이건 다음 [section](#bounding-box-regression-bb)에 자세히 설명해두었습니다.

<p align="center"><img src="../../assets/images/041202.jpg" width="800px" height="400px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 3. mAP on ILSVRC2013 detection test set></center>

다른 dataset인 ILSVRC2013 테스트 set에서도 가장 높은 mAP를 보여줍니다. 


<p align="center"><img src="../../assets/images/041203.jpg" width="800px" height="400px" title="OP code 예시" alt="OP code" ><img></p>
<center><표 2. Detection average precision (%) on VOC 2007 test></center>

<표 2>를 보시면 R-CNN 성능에 대해 세부적으로 나와있습니다. FT는 [(8) fine-tuned](#추가설명)로 FT를 적용한 R-CNN이 더 높은 성능을 보여줍니다. 그 아래 [(9) DPM](#추가설명)은 baseline으로 R-CNN 성능을 확인하기 위한 기준 모델입니다. 전체적으로 성능이 매우 좋네요. VOC 2007 test 데이터에서도 bounding-box regression이 적용된 모델이 가장 성능이 좋은 것을 볼 수 있습니다. 


<p align="center"><img src="../../assets/images/041204.jpg" width="800px" height="400px" title="OP code 예시" alt="OP code" ><img></p>
<center><표 3. Detection average precision (%) on VOC 2007 test for two different CNN architecture></center>

<표 3>을 보시면 AlexNet 구조를 사용한 R-CNN(T-Net)과, Simonyan과 Zisserman이 발표한 16층 구조를 사용한 R-CNN(O-Net)의 성능 차이를 볼 수 있습니다. 전체적으로 O-Net이 성능이 좋네요.

이와 같은 결론을 보면, R-CNN은 이전의 모델보다 훨씬 더 빠르고 정확도가 높은 모델임을 확인할 수 있습니다. 

---

#### 부록

##### Object proposal transformations 

object proposal은 크기가 제멋대로인 image 사각형이기 때문에 227 x 227 고정된 크기의 input에 맞게끔 변경시켜줘야 합니다. 방법은 다음과 같습니다.

1. 227 x 227 input 사각형에 해당 proposal을 꽉 차게 변경하여 넣습니다. 이 때 수평, 수직 비율은 변경 전과 같습니다.
2. 사물 이외의 context(배경)을 제거합니다.
3. 다시 227 x 227 input 사각형에 해당 proposal을 꽉 차게 변경합니다. 이 때에는 수평, 수직 비율 상관없이 기형적으로 팽창하여 가득 채우게 합니다.

context padding(p)가 있고 없고에 따라서도 성능 차이가 나게 됩니다. 

<p align="center"><img src="../../assets/images/040701.jpg" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 2. CNN input size를 맞추기 위해 region proposal을 변형시키는 방법들></center>

<그림 2>를 보면 각 사진별 두개의 행이 있는데 위쪽 행은 padding이 0인 경우, 아래쪽 행은 padding이 16pixel인 경우입니다. context padding은 입력 이미지 크기를 일정하게 유지하면서 객체가 이미지의 가장자리에 위치할 경우 객체가 잘리지 않도록 보장합니다. 전체적으로 context padding이 16pixel인 경우가 그렇지 않는 경웨 비해 3-5mAP 정도 성능이 좋은 것을 볼 수 있습니다. 

---

##### Bounding-box regression (BB)

Bounding-box regression은 예측된 객체의 bounding box를 정확하게 예측하는 데 사용됩니다. 일반적으로 bounding box의 위치와 크기를 수정하는데에 사용됩니다. 

<center>${(P^i,G^i)}_{i=1,...,N}$</center>

여기서 $P^i$는 입력 image 위치와 크기, $G^i$는 실제(ground-truth) bounding box의 위치와 크기를 말합니다. 

<center>$P^i=(P^i_x,P^i_y,P^i_w,P^i_h)$</center>

<center>$G^i=(G^i_x,G^i_y,G^i_w,G^i_h)$</center>

P를 G에 maping 시키는 transform을 하는 것이 목표입니다.

다음과 같은 transform 식으로 ground-truth box $\hat{G}$를 예측합니다. 

<center>$\hat{G}_x=P_wd_x(P)+P_x\qquad(1)$</center>

<center>$\hat{G}_y=P_yd_y(P)+P_y\qquad(2)$</center>

<center>$\hat{G}_w=P_w\exp (d_w(P))\qquad(3)$</center>

<center>$\hat{G}_h=P_h\exp (d_h(P))\qquad(4)$</center>

여기서 $d(P)$는 변형 함수입니다. $d(P)$를 선형 함수를 통해 구현하였는데 pooling layer 5인 pool5에서 입력 P의 feature을 $\phi_5(P)$라고 한다면 $d_\star(P)=w^T_{\star}\phi_5(P)$로 둘 수 있습니다. 여기서 $w_\star$은 학습가능한 가중치 파라미터이며 릿지 회귀로 이 가중치를 업데이트할 수 있습니다. 

<p align="center"><img src="../../assets/images/041301.jpg" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

target인 $t_\star$은 다음과 같은 식으로 표현할 수 있습니다.

<center>$t_x=(G_x-P_x)/P_w\qquad(6)$</center>

<center>$t_y=(G_y-P_y)/P_h\qquad(7)$</center>

<center>$t_w=\log(G_w/P_w)\qquad(8)$</center>

<center>$t_h=\log(G_h/P_h)\qquad(9)$</center>

---

#### 정리

중요한 R-CNN의 특징을 정리하자면 다음과 같습니다.

1. AlexNet은 image classification에만 적용했다. image classification 말고도 region이 있는 object detection에 CNN을 사용하기 위해서 CNN과 region propsal을 합쳐서 만든 알고리즘이다.  
2. object detection은 localization(지역화)가 필요하다. 이전 sliding-window으로 CNN을 적용하기에는 너무 많은 receptive field를 가지게 되고 매우 정확한 localization이 필요하여 기술적 단점이 있게 되었다. 이에 region을 나누어 인식하게 하여 해결하고 후보 영역을 추론하여 해당 영역에 대해서만 object detection이 되게끔 수행하였다. 
3. CNN의 input 값은 고정되어 있기 때문에 무분별한 크기의 region으로 적용하기 불가능하였다. 따라서 227x227 size에 맞게 끔 region을 warp시켜 주었다. warp시키는 방법은 [부록](#object-proposal-transformations)에 자세히 설명되어 있다.
4. ILSVRC2013 detection test set이나 VOC 2007 test set에 대해서 매우 높은 성능을 보여주었다. 


---

### 추가설명

**(1) PASCAL VOC 데이터셋**
- 컴퓨터 비전 분야에서 객체 검출, 분할 이미지 분류 등의 문제를 해결하기 위해 사용되는 대표적인 데이터셋 중 하나.

**(2) 앙상블 시스템**
- Object Detection에서 앙상블 시스템은 여러 개의 다른 object detection 모델들을 결합하여 보다 정확한 object detection 결과를 얻는 기법을 말함. 좀 더 일반화된 모델을 얻을 수 있음.

**(3) mean average precision (mAP)**
- Object Detection, Semantic Segmentation 등에서 모델의 정확도를 측정하는 평가 지표 중 하나. 클래스(class)별로 Average Precision(AP)을 구한 후, 모든 클래스에 대한 AP의 평균값을 의미. AP는 Precision과 Recall을 기반으로 계산되며 Precision-Recall 곡선에서 넓이(Area Under the Curve)를 계산하여 구할 수 있음. 

$Recall = \frac{TP}{TP+FN} = \frac{TP}{ground truths}$

$Precision = \frac{TP}{TP+FP} = \frac{TP}{predictions}$

**(4) OverFeat**
- 2013년에 발표된 객체 검출(Object Detection)과 이미지 분류(Image Classification)모델임. CNN을 기반으로 하며, 일반적인 CNN모델과 달리 이미지 전체를 한 번에 처리하지 않고, 이미지를 여러 개의 큰 윈도우(window)로 분할하여 처리하는 방식을 사용함.

**(5) feature vector**
- 특징 벡터로 입력 데이터를 수치화하는 과정에서 생성되는 고차원 공간상의 점을 나타내는 벡터임. 예를 들어 사람이 다른 사람을 기억할 때, 특징(피부색, 키 등)을 통해 쉽게 기억함. 머신러닝도 입력 데이터를 특징 벡터로 만들어 고차원 벡터 형태로 표현하고 이를 예측하는데 활용함.

**(6) SVM (Support Vector Machine)**
- 지도학습 알고리즘 중 하나로 분류 및 회귀 문제에 사용함. 입력 데이터를 n차원 공간상에 표현하고 각 클래스의 데이터를 가장 잘 구분할 수 있는 경계선을 찾음. 이 때 경계선은 마진(margin)이 최대화되도록 정의함. SVM 분류는 결정 경계선과 가장 가까운 데이터 샘플들 간의 거리을 최대화하여 최적의 분류 경계선을 찾음.

<center><img src="../../assets/images/040702.png" width="400px" height="200px" title="OP code 예시" alt="OP code" ></center>

<center><왼쪽 그림은 선형 분류 3개, 오른쪽 그림은 SVM 분류></center>

**(7) non-maximum suppression (NMS)**
- object detection 작업에서 중복되는 bounding box를 제거하는 알고리즘 중 하나임. bounding box들 중 가장 높은 confidence score(신뢰도 점수)를 가진 box를 선택하고 다른 box들 중 이 box와 IoU(Intersection over Union) 값이 일정 threshold 이상인 것들을 제거하는 방식임.

<center><img src="../../assets/images/040703.png" width="400px" height="200px" title="OP code 예시" alt="OP code" ></center>

**(8) Fine-Tuned**
- 미리 학습된 모델을 새로운 작업에 맞게 적용하기 위해 새로운 작은 데이터셋에서 추가로 학습하는 과정을 말함.

**(9) DPM (Deformable Parts Model)**
- 객체 검출 분야에서 사용되는 모델 중 하나. 