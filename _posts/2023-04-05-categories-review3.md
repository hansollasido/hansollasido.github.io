---
title: "RCNN 리뷰"
excerpt: "Rich feature hierarchies for accurate object detection and semantic segmentation"

categories:
  - 컴퓨터 비전
tags:
  - [딥러닝]

permalink: /categories9/review3/

use_math: true
toc: true
toc_sticky: true

date: 2023-04-05
last_modified_at: 2023-04-05
---

# R-CNN (Regions with CNN features)

#### 간단한 소개

RCNN에 대해서 논문 리뷰를 시작해보겠습니다.

보통 <U>(1) PASCAL VOC 데이터셋</U> 으로 한 사물인지(Object Detection)에서 성능 향상 방법은 복잡한 <U>(2) 앙상블(Ensemble) 시스템</U>이 대부분 이었습니다. 

이 논문에서는 간단하고 크기 조절이 가능한 detection 알고리즘을 소개하고 있으며 <U>(3) mean average precision (mAP)</U>가 이전보다 30%가까이 오른 것을 결과로 보여주고 있습니다.

특히 이 논문은 R-CNN에 대해서 다룰 터인데, R-CNN은 CNN을 위치(region) propsals와 합한 알고리즘입니다. 

간단한 소스코드는 [http://www.cs.berkeley.edu/~rbg/rcnn](http://www.cs.berkeley.edu/~rbg/rcnn) 에 있으며 R-CNN은 200-class ImageNet Large Scale Visual Recognition Challenge (ILSVRC) 2013 데이터셋에서 <U>(4) OverFeat</U>를 능가하는 성능을 보여줍니다.

이제 본격적으로 R-CNN에 대해 알아보겠습니다.

---

#### Introduction

CNN으로 2012년에 ILSVRC에서 우승한 AlexNet은 object detection이 아닌 image classification입니다. image classification은 이미지를 입력으로 받아 이미지에 대한 클래스 레이블을 예측하는 반면, object detection은 분류는 물론 위치 까지 인식해야하는 복잡한 문제를 해결해야 합니다. ILSVRC 2012년에 우승한 CNN classification 결과를 PASCAL VOC Challenge에 object detection 결과에 일반화할 수 있는 방법이 없을까요? 그래서 이 논문은 image classification과 object detection간의 차이를 연결하여 이 문제에 대한 해답을 찾고자 합니다.

image classification과 달리 object detection은 한 이미지에 여러 사물을 분류하기 때문에 localization(지역화)이 중요합니다. 이에 이전 object detection은 localization을 sliding-window 기법으로 해결하였습니다. 하지만 sliding-window에 CNN을 적용할 경우, 너무 많은 receptive fields를 가지게 되고 매우 정확한 localization이 필요하여 sliding-window에 기술적 단점이 있게 되었습니다.

그래서 이 논문은 **CNN localization 문제**를 구간(region)을 나누어 인식(recognition)하게 하여 해결하였습니다. Region Proposal, 즉 이미지에서 객체가 위치한 후보 영역(Region of Interest, ROI)을 추론합니다. 원래는 이미지 전체를 통해 객체의 위치와 종류를 파악하였으나 이미지 내에서 객체가 위치할 가능성이 높은 후보 영역을 추론하였기 때문에 해당 영역에 대해서만 객체 검출을 수행합니다. 이렇게 되면 연산량과 시간 면에서 매우 효율적으로 변합니다. 

<p align="center"><img src="../../assets/images/040601.jpg" width="500px" height="300px" title="OP code 예시" alt="OP code" ><img></p>
<center>그림 1. R-CNN 구현 과정</center>

OverFeat의 sliding-window CNN 기술은 당시까지 ILSVRC2013에 가장 좋은 기술이었지만 R-CNN은 OverFeat을 능가하면서 mAP가 R-CNN은 31.4%, OverFeat은 24.3%으로 더 좋은 수치가 나온 것을 확인했습니다. 

localization 문제 말고도 다른 문제가 있었습니다. **labeled된 data가 적고 CNN을 훈련시키기 위한 충분한 양의 data가 없다는 문제**입니다. 이전에는 이러한 문제를 비지도 사전학습(unsupervised pre-training)으로 해결하였습니다. 하지만 이 논문에서는 지도 사전학습(supervised pre-training)을 large auxiliary dataset인 ILSVRC에서 진행하여 해결하였습니다. 즉, 큰 데이터셋으로 labeled된 이미지를 지도 사전학습 시킨 뒤, 이를 작은 데이터셋에 적용하여 학습 데이터양이 부족한 문제점을 해결했다는 말입니다. 비지도 사전학습보다 지도 사전학습으로 진행한 결과 mAP는 2010년 VOC 33%에서 54%로 향상된 것을 볼 수 있었습니다.

---

#### R-CNN으로 Object Detection하는 방법



---

#### 추가설명

(1) PASCAL VOC 데이터셋
- 컴퓨터 비전 분야에서 객체 검출, 분할 이미지 분류 등의 문제를 해결하기 위해 사용되는 대표적인 데이터셋 중 하나.

(2) 앙상블 시스템
- Object Detection에서 앙상블 시스템은 여러 개의 다른 object detection 모델들을 결합하여 보다 정확한 object detection 결과를 얻는 기법을 말함. 좀 더 일반화된 모델을 얻을 수 있음.

(3) mean average precision (mAP)
- Object Detection, Semantic Segmentation 등에서 모델의 정확도를 측정하는 평가 지표 중 하나. 클래스(class)별로 Average Precision(AP)을 구한 후, 모든 클래스에 대한 AP의 평균값을 의미. AP는 Precision과 Recall을 기반으로 계산되며 Precision-Recall 곡선에서 넓이(Area Under the Curve)를 계산하여 구할 수 있음. 

$Recall = \frac{TP}{TP+FN} = \frac{TP}{ground truths}$

$Precision = \frac{TP}{TP+FP} = \frac{TP}{predictions}$

(4) OverFeat
- 2013년에 발표된 객체 검출(Object Detection)과 이미지 분류(Image Classification)모델임. CNN을 기반으로 하며, 일반적인 CNN모델과 달리 이미지 전체를 한 번에 처리하지 않고, 이미지를 여러 개의 큰 윈도우(window)로 분할하여 처리하는 방식을 사용함.
