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

### 방법


<p align="center"><img src="../../assets/images/050204.jpg" width="700px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 3. 방법></center>

첫 번째로, object detection으로 input video에 대한 사물 인지를 진행합니다. 이 때 object detection은 yolov3로 진행하며 output feature map으로는 location, size, class가 있습니다. 이 bounding box는 RoI로 동작하며 합니다. 

<그림 3>과 같이 object detection으로 RoI를 지정하였으면 이 RoI값을 1-D 수직으로 영사하여 값을 얻습니다. <그림 3>의 (c)를 보면 수직으로 투영하여 RoI가 있는 x 위치는 1로 없는 x위치는 0으로 만들었습니다.


<center>$ m[x] = \begin{cases} 1 & \mbox{if} \ x \in x_{object} 
\\ 0 & \mbox{else}
\end{cases} , x = 1, 2, \cdots, W, $</center>

여기서 $W$는 input video의 너비 사이즈입니다. 위의 식처럼, 중요한 사물이 있다라고 생각되는 region은 1로 아닌 구역은 0으로 만듭니다. 중요한 사물은 일반적으로 사람, 동물 등이 있죠. 

이제 scaling ratio map인 $S[x]$를 정의할 차례입니다. 

<center>$ S[x] = \begin{cases} 1 & \mbox{if} \ x \in x_{object}
\\ \alpha & \mbox{else} \end{cases} , x = 1,2,\cdots,W,W'=\sum^W_{x=1}S[x],$
</center>


<center>$\alpha=\frac{W'-W+n(x_{object})}{n(x_{object})}, $</center>

여기서 $W'$는 target width로 변경할 이미지 너비이고, n(A)는 A의 구성요소 개수입니다. $n(x_{object})$는 그러면 object가 있는 x의 개수를 의미하겠네요. object가 있는 x 위치에는 1, 아니면 $\alpha$로 scaling ratio를 정의합니다. 각 component에 scaling ratio를 더하면 image는 재구성되어 위치하게 됩니다. <그림 3>의 (e)처럼요. 하지만 그림을 보시면 hole이 생겨 검은색 선이 생기게 됩니다. 이런 경우에는 [directional interpolation (6)](#추가설명)을 적용하여 content를 채웁니다. 

새로운 이미지에 재구성된 index값인 $\tau$는 다음과 같습니다.

<center>$\tau=\left[\sum_{n=1}^x S[n]\right],x=1,2,\cdots,W,$</center>

위 식을 통해 $\tau$를 계산하고 원본 이미지의 index에 해당하는 값들을 재위치 시킵니다. 그 다음 hole을 interpolation으로 보간하여 채우죠. 이런식으로 적용하면 <그림 3>의 (f)처럼 완전한 모양을 갖출 수 있습니다.

---

### 문제점

해당 방법으로 video retargeting을 진행할 시 문제점이 발생합니다. 첫 번째로는 **연속적인 프레임에서 bounding box가 달라질 경우 일시적인 distortion**이 발생할 수 있습니다. 두 번째로는 **object detection 네트워크에 bounding box가 detect 되지 않을 경우 예외처리가 힘든 점**입니다. 세 번째로는 **object detection이 있는 네트워크로서 처리 속도가 생명인 video retargeting 관점에서는 비효율적**일 수 있습니다. 이러한 세 가지 문제를 본 논문에서는 다음과 같이 해결했습니다.

---

#### 해결과정

Siamese network를 사용해서 연속적인 프레임의 상관관계를 유추하였습니다. 먼저 연속적인 프레임에 L2 norm을 적용하여 나온 값이 특정 값을 초과한다면 scene이 바뀐 것으로 인지합니다. 이 scene이 바뀌는 change point를 기억해서 scene들을 여러 set으로 모읍니다. set에 있는 첫 번째 이미지를 object detection하여 bounding box가 생기면 그 bounding box를 이용해 retargeting을 진행합니다. 그 다음 object detection 네트워크 대신, Siamese network를 사용하여 연속적인 bounding box를 유추합니다. 

<p align="center"><img src="../../assets/images/050401.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 4. Siamese network></center>

Siamese network는 깊은 네트워크가 아닌 얕은 네트워크로 구성되어 있습니다. 이전 프레임 이미지인 $F_{t-1}$의 bounding box $B_{t-1}$와 현재의 전체 이미지인 $F_t$를 같은 네트워크 $\psi$의 input으로 정의합니다. 이전 이미지에 있는 object가 그 다음 이미지에 있을 가능성이 높기 때문에 해당 네트워크를 사용해서 상관도를 분석한 뒤, 이전 이미지의 $Obj_{t-1}$의 bounding box를 이용하여 현재 이미지의 $Obj_t$의 위치를 예측합니다. t+1시점에서도 마찬가지로 t에서의 $Obj_t$를 이용하여 bounding box를 예측합니다. 이런 식으로 object detection 네트워크 사용없이 Siamese network를 사용할 수 있으며, 이 같은 방법을 사용할 경우 계산 복잡도가 낮아져 불필요한 operation들을 제거할 수 있습니다. 

<p align="center"><img src="../../assets/images/050403.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 5. Object tracking></center>

이러한 Object tracking 기술은 일시적인 distortion에서도 합리적인 해결방법이 될 수 있으나, bounding box의 사이즈나 위치 차이가 연속적인 프레임에서 이질성이 증폭될 수 있습니다. 그러므로 이러한 점을 해결하기 위하여 일관성을 고려해야합니다. 다른 프레임이라도 똑같은 scene에서 1D RoI는 [exponential average filter (7)](#추가설명)를 사용하여 이와 같은 문제를 해결하였습니다. 

<center>$S_t=\begin{cases}Y_1,  & t=1 \\
\gamma\cdot Y_t+(1-\gamma)\cdot S_{t-1},  & t>1 \end{cases}$</center>

여기서 $Y_t$는 t번째 이미지의 RoI를 의미하고, $\gamma$는 하이퍼파라미터, $S_t$는 t번째 이미지에서 새롭게 계산한 RoI을 의미합니다. $\gamma$가 0.1일 때 가장 자연스러운 결과를 얻었습니다. 때로는 부정확한 object detection와 object tracking으로 인해 RoI가 retargeted video에서 왜곡될 수 있습니다. 이와 같은 문제를 해결하기 위해서 추론된 RoI와 축적된 RoI 사이의 Intersection Over Union(IOU)를 사용하여 $\gamma$를 정의합니다. 


<center>$\gamma=0.10\cdot\frac{Y_t\cap S_{t-1}}{Y_t\cup S_{t-1}}$</center>

이렇게 하면 부정확한 object detection, object tracking에서도 깔끔한 retargeted image가 만들어집니다. 하지만 RoI가 이미지에서 매우 크게 차지하는 문제가 발생하는 경우가 가끔 발생하게 됩니다. 이러한 경우, 큰 RoI를 제외한 작은 배경만이 stretch되기 때문에 매우 이질감이 듭니다. 따라서 object size에 따른 적절한 scaling을 계산하여 stretching 비율을 조절하였습니다. 

<center>$\lambda_{sr}=1+(\frac{AP_{\mbox{target}}}{AP_{\mbox{original}}}-1)\cdot\frac{W_{\mbox{RoIs}}}{W_{\mbox{image}}}$</center>

<p align="center"><img src="../../assets/images/050402.jpg" width="700px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 6. stretching 비율 조절></center>

---

### 결과

<p align="center"><img src="../../assets/images/050404.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 6. stretching 비율 조절></center>

위 결과 사진을 보면 정말 깔끔한 retargeting image가 만들어집니다. object detection을 매 프레임마다 하는 것이 아닌 Siamese network로 bounding box를 매 프레임마다 예측해도 깔끔한 이미지를 만들어주네요!

---

### 정리 

정리하면 다음과 같습니다.

1. object detection으로 RoI를 지정하여 해당 RoI의 x축은 그대로 나머지 부분만 stretching 함.
2. stretching할 때는 directional interpolation을 사용.
3. object detection을 매번 할 경우 계산 복잡도가 매우 높기 때문에, scene set 별로 첫 번째 프레임만 object detection 적용함. 그 다음 Siamese network를 통해 bounding box를 첫 번째 프레임의 object detection으로 예측함. 이와 같은 방법으로 복잡도 개선과 부정확한 retarget을 해결함.
4. 일시적인 왜곡이 생길 수 있는 경우는 filter를 사용하여 개선하였고, RoI가 매우 큰 이미지 같은 경우에는 stretching 비율을 조정하여 이질감을 줄였음.
5. 결과적으로 중요한 사물은 보존, 중요하지 않는 배경은 늘려 retargeted 이미지를 생성함.

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

**(6) Directional interpolation (방향성 보간)**
- 객체의 움직임 방향을 기반으로 보간을 진행. 

**(7) Exponential average filter**
- 신호처리에서 사용되는 필터 중 하나, 이전 데이터 포인트들의 가중평균값을 사용하여 현재 데이터 포인트를 평활화하는 필터임.이전 데이터 포인트들의 영향력이 지수적으로 감소하도록 설계되어 있음.