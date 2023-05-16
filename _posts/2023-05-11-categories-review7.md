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

date: 2023-05-16
last_modified_at: 2023-05-16
---

# 참고 사이트

[약초의 숲으로 놀러오세요 (herbwood)](https://herbwood.tistory.com/20)

위 사이트에서 참고를 많이 했습니다. 

# Faster R-CNN 

[Faster R-CNN](https://hansollasido.github.io/categories9/review5/)논문 리뷰를 먼저 보시는 것을 추천드립니다!!

## 간단한 소개

본 논문은 object instance segmentation을 위한 간단한 framework를 제안합니다. object detection task보다는 instance segmentation task에 주로 사용됩니다. 

<p align="center"><img src="../../assets/images/051104.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 1. Instance segmentation></center>

Instance segmentation을 위한 framework를 Faster R-CNN에 병렬적으로 붙인 것이 Mask R-CNN인데요. 해당 Network가 어떻게 적용되는지 알아보겠습니다.
  
---

### Mask R-CNN

Mask R-CNN은 개념적으로는 간단합니다. Faster R-CNN이 object에 대한 label과 bounding box를 만들면 여기에 object mask라는 branch를 넣습니다. 그 다음으로는 Fast/Faster R-CNN이 놓쳤던 pixel to pixel alignment이라는 핵심을 도입합니다. 

<p align="center"><img src="../../assets/images/051605.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 2. Mask R-CNN framework></center>

Mask R-CNN은 segmentation task를 효과적으로 수행하기 위해 객체의 spatial location을 보존하는 RoIAlign layer를 추가했습니다. 

<p align="center"><img src="../../assets/images/051606.png" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 3. Mask Branch></center>

*<그림 3>은 정식 Mask R-CNN 구조가 아니니 참고 바랍니다.* RoI pooling을 통해 얻은 고정된 크기의 feature map을 mask branch에 입력하여 segmentation mask를 얻습니다. 

segmentation task는 픽셀 단위로 class를 분류해야 하기 때문에 [spatial layout (1)](#추가설명)를 필요합니다. class label이나 bounding box가 fc layer에서 변형되는 것과 달리 mask는 공간 정보를 효과적으로 encode할 수 있습니다. 

mask branch는 RoI에 대하여 class별로 binary mask를 출력합니다. 기존의 instance segmentation 모델은 하나의 이미지에서 여러 class를 예측한 반면, Mask R-CNN은 class 별로 mask를 생성한 후 픽셀이 해당 class에 해당하는지 여부를 표시합니다. 

<p align="center"><img src="../../assets/images/051607.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 4. Mask></center>

---

#### RoIAlign 

Faster R-CNN에서 RPN(Rgeion Pooly Network)를 진행한 뒤, RoI Pooling을 통하여 고정된 크기의 feature map을 얻습니다. 하지만 RoI Pooling은 정확한 픽셀 단위의 alignment가 불가능합니다. 이는 픽셀 단위로 진행하는 segmentation에 치명적입니다. 따라서 Mask R-CNN에서는 **RoIAlign**을 사용하여 이러한 문제를 해결했습니다. 

<p align="center"><img src="../../assets/images/051608.png" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 5. RoI Pooling></center>

<그림 5>를 보시면 RoI pooling 시 quantization이 필요한 것을 볼 수 있습니다. quantization을 진행하다보면 손실하는 데이터가 많게 됩니다. pixel-to-pixel 단위의 image segmentation에는 굉장히 치명적이게 되는 것이죠.

<p align="center"><img src="../../assets/images/051609.png" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 6. Quantization loss></center>

이러한 quantization의 단점을 보완하기 위해 RoIAlign 방법을 사용하였습니다.

<p align="center"><img src="../../assets/images/051610.png" width="700px" height="700px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 7. RoIAlign 방법></center>

이 때 사용하는 것이 Bilinear interpolation입니다. 먼저 RoI projection을 quantization없이 진행합니다. 그 다음 feature map을 분할해줍니다. Bilinear interpolation 절차에 따라 중간에 있는 값을 추정합니다. 

<center>$P\approx\frac{y_2-y}{y_2-y_1}(\frac{x_2-x}{x_2-x_1}Q_{11}+\frac{x-x_1}{x_2-x_1}Q_{21})+\frac{y-y_1}{y_2-y_1}(\frac{x_2-x}{x_2-x_1}Q_{12}+\frac{x-x_1}{x_2-x_1}Q_{22})$</center>

<p align="center"><img src="../../assets/images/051611.png" width="700px" height="700px" title="OP code 예시" alt="OP code" ><img></p>


<p align="center"><img src="../../assets/images/051612.png" width="700px" height="700px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 8. Bilinear interpolation 과정></center>

위 과정을 모든 cell에 반복하고 maxpooling을 진행합니다. 이렇게 하면 quantization으로 인해 손실되었던 data를 어느정도 복구할 수 있습니다. 

<p align="center"><img src="../../assets/images/051613.png" width="700px" height="700px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 9. Max pooling></center>

---

#### Mask Branch

<p align="center"><img src="../../assets/images/051614.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 10. Head Architecture></center>

RoIAlign 과정을 통해 얻은 7x7 크기의 feature map을 mask branch에 전달하면 14x14x(k) (여기서 k는 class의 개수)크기의 output feature을 얻을 수 있습니다. 이 output feature은 class 별로 생성된 binary mask입니다. 

---

## 추가설명

**(1) Spatial layout**
- 공간적 배치 또는 구조를 의미함. 요소나 개체들이 어떻게 공간 내에서 배열되는지에 대한 정보를 포함함. 