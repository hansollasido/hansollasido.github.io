---
title: "DETR 리뷰"
excerpt: "End-to-End Object Detection with Transformers"

categories:
  - 컴퓨터 비전
tags:
  - [딥러닝, 논문 리뷰, Object Detection]

permalink: /categories9/review1/

use_math: true
toc: true
toc_sticky: true

date: 2023-03-23
last_modified_at: 2023-04-14
---

## 논문리뷰를 하기 앞서

reference를 많이한 블로그가 있는데 링크 올려드립니다!

[herbwood 블로그](https://herbwood.tistory.com/26)

[oi.readthedocs.io](https://oi.readthedocs.io/en/latest/computer_vision/object_detection/object_detection.html)

## DETR

DETR (End-to-End Object Detection with Transformers) 논문은 ECCV (유럽 컴퓨터 비전 학회)에 2020년 발표된 논문 입니다. SOTA (State of the Art) 들이 추후 DETR을 기반으로 한 만큼 중요한 알고리즘이라고 볼 수 있습니다.

---

#### 배경

<center><img src="../../assets/images/041401.png" width="600px" height="400px" title="OP code 예시" alt="OP code"><img><br/></center>
<center><그림 1. window-sliding 방법></center>

초창기 window-sliding 방식으로 localization을 진행하였습니다. 사물의 위치가 아닌 분류만 하는 image classification과 달리 object detection은 분류와 사물의 위치 설정인 localization이 중요했죠. 따라서 window-sliding 기법으로 window를 이동시켜가며 window에 사물이 인식될 시, 출력시키는 방법을 채택했죠.

이렇게 될 경우, 각 [(1) grid](#추가설명)에서는 자신이 object의 중심점이라고 판단할 수 있습니다. 즉 한 사물에 여러 bounding box가 만들어질 수 있다는 뜻이죠. 이를 위해서 전처리 기법인 **Non-max suppression** 방법을 사용하였습니다. bounding box의 신뢰도가 낮은 것들은 끄고, 가장 신뢰도가 높은 bounding box를 선택하여 이와 IoU값이 0.5이상인 bounding box들 또한 끄는 기법입니다. 

<center><img src="../../assets/images/041403.png" width="600px" height="400px" title="OP code 예시" alt="OP code"><img><br/></center>
<center><그림 2. Non-max suppression></center>

전체적인 object detection의 알고리즘은 다음과 같이 정리됩니다.

<center><img src="../../assets/images/041404.png" width="600px" height="400px" title="OP code 예시" alt="OP code"><img><br/></center>
<center><그림 3. 전체적인 알고리즘></center>

하지만 window-sliding 기법으로는 한계가 있었습니다. 특히 동일한 window에 사물이 2개가 있을시, 하나밖에 분류하지 못하는 기술적 문제가 있었죠. 각 grid가 오직 하나만의 object를 detection할 수밖에 없는 문제점입니다. 하나의 grid에 두 개의 object를 감지하려면 어떻게 해야할까요?

이래서 나온 기법이 **anchor box**입니다. anchor box로 여러 box들을 선언하여 해당 object를 감지하도록 만듭니다. 

<center><img src="../../assets/images/041405.png" width="600px" height="400px" title="OP code 예시" alt="OP code"><img><br/></center>
<center><그림 4. anchor box></center>

$p_c$는 해당 object가 검출되었는지에 대한 true, false값이며 $b_x$는 bounding box의 center point x 위치, $b_y$는 bounding box의 center point y 위치, $b_h$는 bounding box의 높이, $b_w$는 bounding box의 너비를 말하며 c0~c2는 class입니다. 하나의 grid에서 두 개의 object를 동시에 detection할 수 있게 된 것이죠.

<center><img src="../../assets/images/041406.png" width="600px" height="400px" title="OP code 예시" alt="OP code"><img><br/></center>
<center><그림 5. anchor box 적용></center>

이 anchor box를 사용하여 train하고 predict하는 방법은 faster-rcnn, yolo 논문 리뷰에서 자세히 살펴보겠습니다.

<img src="../../assets/images/040301.png" width="800px" height="400px" title="OP code 예시" alt="OP code"><img><br/>
<center><그림 6. 이전 object detection 과정></center>

정리하자면 이런 anchor box를 생성하고 NMS(Non-Maximum Suppression)으로 전처리하는 과정을 통해 object detection을 합니다. NMS라는 전처리 과정을 통해 object 하나에 표현되었던 많은 box들이 하나의 단순히 몇 개의 box로 줄어들게 되죠. 

즉 예측한 bounding box와 ground truth의 관계가 many to one이 되어 전처리가 꼭 필요하게 됩니다. 처음부터 one to one으로 bounding box를 하나의 object에 그려낸다면 이런 과정을 진행할 필요가 없게 되죠. 그래서 나온 방법이 DEtection TRansformer(DETR)입니다. DETR은 처음부터 ground truth에 상호되는 bounding box가 하나밖에 나오지 않아 NMS 과정을 겪지 않아도 됩니다. 

<center><img src="../../assets/images/041407.png" width="400px" height="400px" title="OP code 예시" alt="OP code"><img><br/></center>
<center><그림 7. DETR></center>

#### DETR 원리



### 추가설명