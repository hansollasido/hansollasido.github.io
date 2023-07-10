---
title: "DETR"
excerpt: "End-to-End Object Detection with Transformers 리뷰"

categories:
  - 컴퓨터 비전
tags:
  - [딥러닝, 논문 리뷰, Object Detection]

permalink: /categories9/review10/

use_math: true
toc: true
toc_sticky: true

date: 2023-07-10
last_modified_at: 2023-07-10
---

# 참고 사이트

[약초의 숲으로 놀러오세요 (herbwood)](https://herbwood.tistory.com/25)

위 사이트에서 참고를 많이 했습니다. 

---

## 논문 소개

이전의 많은 object detection 논문들은 non-maximum suppression(NMS)와 anchor generation 등의 추가적인 hand-design 작업이 필요했습니다. 하지만 본 논문은 detection pipeline을 간소화하여 이와 같은 전처리 작업의 필요성을 제거하였습니다. 본 논문의 새로운 프레임워크인 DEtection TRansformer(DETR)에 대해 설명해보겠습니다.

---

## 배경

DETR이 나온 배경은 다음과 같습니다.

이전의 detector들은 간접적인 방법으로 detection을 진행했습니다. proposal, anchor, window center등의 방법으로 box와 class를 예측해왔습니다. 하지만 이러한 작업은 불필요한 중복 예측에 전처리 단계가 꼭 있어야하며, anchor 설계와 target box를 anchor에 할당하는 휴리스틱을 설계해야한다는 문제가 있었죠. 

이러한 문제점들을 해결하기 위하여 end-to-end 철학을 적용한 DETR 기법을 소개해드리려고 합니다. 

<p align="center"><img src="../../assets/images/071001.png" width="600px" height="600px" title="DETR 예시" alt="OP code" ><img></p>

DETR은 encoder-decoder 구조를 적용하였는데요, prediction과 ground-truth간의 관계에 bipartite matching을 적용한 구조입니다. DETR은 이전의 detection 방법과는 다르게 customized layer가 필요하지 않다는 장점을 가지게 됩니다.

---

## DETR 모델

detection하기 위해 두 가지를 명심해야합니다. 

1. a set of prediction loss that forces unique matching between predicted and ground truth
2. architecture that predicts a set of objects and models their relation

---

### Object detection set prediction loss

고정된 N 개의 예측을 진행할 때, N은 이미지에 있는 object 개수보다 훨씬 많은 수를 가집니다. 

학습할 때 가장 어려운 점은 ground-truth에 대하여 predicted된 object 점수(class, position, size)입니다. 본 논문은 predicted object와 ground-truth object 사이의 최적의 bipartite matching을 만들어내고, bounding box loss 또한 최적화하는 loss를 정의합니다.  

<p align="center"><img src="../../assets/images/071002.png" width="300px" height="300px" title="cost 식" alt="OP code" ><img></p>

<center><식 1></center>

<p align="center"><img src="../../assets/images/071003.png" width="300px" height="300px" title="cost 중간 식" alt="OP code" ><img></p>

<center><식 2></center>

여기서 loss function을 계산하기 위하여 적용한 방법이 *Hungarian algorithm*입니다. 

<p align="center"><img src="../../assets/images/071005.png" width="600px" height="600px" title="Hungarian algorithm 1" alt="OP code" ><img></p>

Hungarian algorithm은 어떠한 집합 I와 matching 집합 J가 있으면 matching하는데 드는 비용을 c(i,j)라고 할 때, 일대일 대응 중에서 가장 적은 cost가 드는 matching에 대한 permutation $\sigma$를 찾는 것입니다. 예를 들어 위 figure를 보면, 오른쪽 행렬이 cost에 대한 행렬일 때 permutation이 [1,2,3,4,5]이라면 Mathing score는 32입니다. 

<p align="center"><img src="../../assets/images/071006.png" width="600px" height="600px" title="Hungarian algorithm 2" alt="OP code" ><img></p>

하지만 위 figure와 같이 permuation이 [3,4,1,5,2]라면 matching score는 12로 [1,2,3,4,5]보다 cost가 낮게 나와 더 나은 결과를 가질 수 있습니다. 이처럼 matching algorithm에 *Hungarian algorithm*을 적용하면 최소의 cost를 가지는 prediction을 얻을 수 있습니다. 

<p align="center"><img src="../../assets/images/071004.png" width="600px" height="600px" title="식 2 내용" alt="OP code" ><img></p>

이 *Hungarian algorithm*을 적용하여 L_matching을 구합니다. 

---

### Bounding box loss

<p align="center"><img src="../../assets/images/071007.png" width="600px" height="600px" title="GIoU" alt="OP code" ><img></p>

이전 detector과 달리 bounding box를 바로 predict하기 때문에 initial guess가 없어 예측하는 값의 범주가 상대적으로 큽니다. 이에 l1 loss만 쓰는 것이 아닌 generalized IoU(GIoU) loss를 함께 사용합니다. 

<p align="center"><img src="../../assets/images/071008.png" width="300px" height="300px" title="GIoU 식" alt="OP code" ><img></p>

이 l1 loss와 GIoU loss는 batch안에 있는 object 개수에 따라 정규화 됩니다. 

---

## DETR 구조

이제 DETR 구조입니다. 전체적인 구조는 아래 figure와 같습니다.

<p align="center"><img src="../../assets/images/071009.png" width="600px" height="600px" title="DETR 구조" alt="OP code" ><img></p>

총 다섯 가지의 BIG Network로 구성되어 있습니다.

1. Backbone
2. Transformer encoder
3. Transformer decoder
4. Prediction feed-forward networks (FFNs)
5. Auxiliary decoding losses

---

### Backbone

initial image가 $3 \times H_0 \times W_0$로 되어 있을 때 이를 $C \times H \times W$로 변환합니다. 본 논문은 $C$를 $2048$, $H$,$W$를 각각 $\frac{H_0}{32}$, $\frac{W_0}{32}$로 변환합니다. 

---

### Transformer encoder

1x1 convolution을 진행하여 차원을 C에서 d로 변환합니다. 그리고 $d \times H \times W$를 $d \times HW$로 변환합니다. 

각 encoder layer는 multi-head self-attention module과 feed forward network(FFN)으로 구성되어 있습니다. transformer 구조가 permutation-invariant이기 때문에, 고정된 positional encodings를 각 attention layer의 input에 더합니다. 

---

### Transformer decoder

decoder은 transformer의 standard 구조를 따랐으며, multi-headed self-와 encoder-decoder attention 메커니즘을 사용하여 N 개의 embedding을 d로 변환합니다. 기존의 transformer과의 차이점은 N개의 object를 각 decoder layer에 병렬적으로 decode한다는 점입니다. 

decoder는 permutation-invariant하기 때문에 N개의 input embedding은 반드시 다른 결과를 내야합니다. 이러한 input embedding은 learnt positional encoding입니다. positional encoding은 *object queries*라고도 언급되는데, encoder과 비슷하게 각 attention layer의 input에 이를 더합니다. 

N개의 object queries는 decoder를 통해 output embedding로 변환되는데, 이후 N개의 최종적인 prediction을 낳기 위하여 feed forward network를 통해 독립적으로 box coordinate와 class label로 decoding됩니다.

---

### Prediction feed-forward networks (FFNs)

최종 prediction은 ReLU 활성화 함수가 있는 3개의 layer preceptron과 hidden dimension d, linear projection layer로 계산됩니다. FFN은 정규화된 center 좌표와 높이 너비를 예측하고 linear layer은 softmax 함수를 사용하여 class를 예측합니다. 

N개의 bounding box의 고정된 사이즈 set를 예측하기 때문에, N은 공집합을 포함하여 image의 실제 object 수보다 훨씬 클 수밖에 없습니다. 이 공집합은 기존 object detection의 background class와 비슷한 역할을 수행합니다. 

---

### Auxiliary decoding losses

보조적인 loss를 추가하는 것이 더 효율적임을 발견해서, 각 decoder layer 후에 FFNS 예측과 Hungarian loss를 추가합니다. 모든 predictions FFN는 파라미터를 공유합니다. 

---

## 실험 결과

COCO 2017 detection, segmentation Dataset에 대하여 실험을 진행하였습니다. 

<p align="center"><img src="../../assets/images/071010.png" width="600px" height="600px" title="실험 결과" alt="OP code" ><img></p>

여기서 DC5는 Dilated C5 stage를 의미하며 R101은 ResNet 101을 의미합니다. FPN은 Feature Pyramid Network의 약자입니다. 

top section에는 Detectron2에서 Faster RCNN를, middle section에서는 GIoU와 Faster R-CNN 모델을 사용했을 때의 결과를 보여줍니다. Faster R-CNN에 비교하였을 때, 비슷한 정확도를 보이면서 end-to-end 학습이 가능한 것을 볼 수 있습니다. 

<p align="center"><img src="../../assets/images/071011.png" width="600px" height="600px" title="실험 결과" alt="OP code" ><img></p>

<p align="center"><img src="../../assets/images/071012.png" width="600px" height="600px" title="실험 결과" alt="OP code" ><img></p>

<p align="center"><img src="../../assets/images/071013.png" width="600px" height="600px" title="실험 결과" alt="OP code" ><img></p>

<p align="center"><img src="../../assets/images/071014.png" width="600px" height="600px" title="실험 결과" alt="OP code" ><img></p>

spatial position encoding 없이 나온 AP와 적용한 AP의 비교 결과 입니다. 확실히 spatial position encoding을 적용하면 AP가 높은 것을 볼 수 있네요. 

<p align="center"><img src="../../assets/images/071015.png" width="600px" height="600px" title="실험 결과" alt="OP code" ><img></p>

loss function을 l1과 GIoU 둘다 사용했을 때 AP가 높은 것을 볼 수 있습니다.

---

## 결론

transformer와 bipartite matching loss를 기반으로 한 DETR이라는 새로운 object detection 시스템을 설계하였습니다. Faster R-CNN 와 비교했을 때 비교할만한 성능이 나왔습니다. DETR은 실행할 때 straightforward 방법으로 진행되며, segmentation으로 확장하기도 쉬운 유연한 구조로 설계되었습니다. 추가적으로 self-attention 으로 수행되는 global information processing 덕분에 Faster R-CNN보다 더 큰 object에 좋은 성능을 보여줍니다.

이러한 DETR은 작은 object에 대하여 학습, 최적화, 성능부분에서 새로운 도전과제를 남습니다. 이러한 도전과제를 해결하기 위하여 future work 또한 기대합니다. 