---
title: "Multimodal Deep Learning for Computer Vision"
excerpt: "Multimodal Deep Learning for Robust RGB-D Object Recognition"

categories:
  - 컴퓨터 비전
tags:
  - [딥러닝, 논문 리뷰, 멀티모달, Computer Vision]

permalink: /categories9/review18/

use_math: true
toc: true
toc_sticky: true

date: 2023-09-08
last_modified_at: 2023-09-08
---

Object Recognition관련된 논문이자, CNN 뿐만아니라 encoding도 적절하게 섞은 Multimodal Deep Learning입니다. 

---

### Abstract

본 논문은 RGB-D 구조의 recognition을 위한 방법을 제안합니다. 두 개의 나눠진 CNN 진행 과정을 거치는데요. 나중에 fusion network를 통하여 합쳐집니다. 본 논문은 real-world robotics task에서 전형적인 문제에 집중해보았습니다. 정확한 학습을 위해 multi-stage 학습 방법과 CNN에서 depth data를 다루기 위한 두 가지 중요한 요소를 소개합니다. 첫 번째로, large depth dataset의 필요가 없는 학습이 가능하게 한 depth 정보를 효율적으로 encoding한 것입니다. 두 번째로는, 실제 노이즈 패턴으로 depth 이미지를 손상시켜 강인한 학습을 위한 data augmentation 방식입니다. 

본 논문은 RGB-D object dataset에 대하여 sota result를 만들었으며, 실제 노이즈가 있는 RGB-D에서도 recognition이 가능한 것을 보여줍니다. 

---

### Introduction

RGB-D object recognition은 robotics나 indoor, outdoor에 많은 application의 핵심 역할을 담당하고 있습니다. 소프트웨어 지원이 충분하여 비싸지 않고, 복잡한 하드웨어를 요구하지 않고, 특별한 sensing 용량을 제공합니다. 출현과 문맥에 대한 정보를 제공하는 RGB data에 비교해보았을 때, RGB-D는 depth data를 포함하고 있어 object shape의 추가 정보를 제공하고, 빛이나 색깔 변동에도 변동성이 적습니다. 

본 논문은, RGB-D data에 관련한 object recognition에 대해 새로운 방법을 제안합니다. 특히, 완벽하지 않은 센서 데이터에 강한 recognition을 만드려 노력했습니다. 많은 robotic task의 전형적인 시나리오죠. machine learning과 computer vision community에서의 최근 approach에 기반하여 설계했습니다. 특히, CNN을 확장하여 RGB image를 RGB-D image의 영역으로 넓혔습니다. 본 논문이 제안하는 구조는, **<Fig 1>**에 있습니다. 

<p align="center"><img src="../../assets/images/091101.png" width="300px" height="300px" title="Multimodal" alt="Multimodal" ><img></p>

두 개의 Convolutional network 줄기가 color와 depth 정보를 각각 동작하고 있는 것을 볼 수 있습니다. 나중에 이 네트워크는 서로 합쳐지게 됩니다. 이 구조는 다른 최근 multi-stream 방법에 유사한 구조를 가지고 있습니다. 각각의 줄기 네트워크 학습 뿐만 아니라 통합된 구조도 stage-wise 방법을 따릅니다. 분리된 채로 학습하는 네트워크는 fine-tuned되어 있으며 마지막 fusion 네트워크는 classification을 수행합니다. RGB와 depth stream 네트워크는 ImageNet dataset으로 사전 학습된 네트워크입니다. RGB 네트워크를 사전 학습된 ImageNet 네트워크에 시작하는 것은 간단하지만, depth 네트워크를 처리하는 데 사용하는 것은 쉽지 않습니다. 이상적으로는, 사람들은 depth 데이터로부터 recognition 네트워크를 다른 모드에서 사전 학습하지 않고 직접 학습하고 싶어합니다. 하지만, 대규모로 labeling된 depth dataset의 부족으로 인하여, 이것은 실행하기 어렵습니다. 

labeled된 학습 data의 부족으로, depth-modality를 위한 사전 학습 pahse가 중요합니다. 그러므로, **본 논문은 ImageNet으로 학습된 CNN을 재사용할 수 있도록 depth data를 encoding하는 방법을 제안**합니다. 주어진 RGB image로 depth image를 encode하여 세 개의 RGB channel 모두에 depth data가 포함된 정보를 전달하고, 이후 recognition을 위한 표준 CNN을 사용합니다. 

실제 환경에서, 사물은 종종 가려짐과 센서 잡음의 영향을 받습니다. **본 논문은 depth data를 위한 data augmentation 기술을 사용하여 강인한 학습이 될 수 있게** 만듭니다. 실 세계 환경에서 샘플링된 놓친 data pattern을 활용하여 depth data를 붕괴해서 사용가능한 학습 예제를 증강시킵니다. 이러한 두 기술을 사용하여, 본 논문의 시스템은 **강인한 depth feature**와 두 양식의 중요성을 함축적으로 내포하는 **가중치**를 학습할 수 있게 됩니다. 

---

### Multimodal Architecture for RGB-D Object Recognition

<p align="center"><img src="../../assets/images/091102.png" width="700px" height="700px" title="Multimodal" alt="Multimodal" ><img></p>

RGB stream과 depth stream은 깊은 CNN으로 구성되며, 이 CNN은 ImageNet database의 object classification을 사전학습한 네트워크입니다. 사전 학습된 네트워크로 시작하는 주된 이유는 제한된 RGB-D 학습 데이터를 사용하여 백만개의 파라미터가 있는 큰 CNN을 학습시키려하기 때문입니다. 먼저 두 모형으로 부터 data를 전처리한 다음, stage-wise 방법으로 학습합니다. target data의 classification을 위한 각 stream 네트워크 파라미터를 fine-tune합니다. 이후에 fusion 네트워크로 함께 학습되죠. 

---

#### Input Preprocessing

ImageNet으로 사전학습된 CNN을 온전히 활용하기 위하여, RGB와 depth input data를 원래 ImageNet input의 종류에 호완될 수 있도록 전처리합니다. 특히, CaffeNet을 사용하는데, CaffeNet은 더 큰 256x256 RGB 이미지로 부터 무작위로 crop하여 227x227 RGB image를 input으로 취합니다. 

처음으로 전처리하는 단계는 image를 적절한 image 크기로 scaling하는 단계입니다. 가장 간단한 방법은 original image를 요구된 image 차원으로 warping하는 방법입니다. **<Fig 3>**처럼요. object recognition 성능이 이 전처리 과정에서 나빠지는 것을 알 수 있었습니다. shape 정보가 이 과정에서 잃어버리기 때문이죠. 

그러므로 다른 전처리 방법을 사용하였습니다. 원래 image의 가장 긴 부분을 256 pixel로 scale하여 256 x N 또는 N x 256 사이즈로 만드는 방법입니다. 그다음, 가장 긴 쪽을 짧은 쪽 축에 따라 tile을 만듭니다. RGB나 depth 이미지는 object border 주위의 인위적인 context을 보여줍니다. (**<Fig 3>**참고). RGB와 depth image 둘다 같은 scaling operation을 적용하였습니다. 

<p align="center"><img src="../../assets/images/091202.png" width="300px" height="300px" title="Multimodal" alt="Multimodal" ><img></p>

RGB 이미지는 이 전처리 단계를 지나고 다음 CNN의 input으로 직접 쓰이게 되는 반면, rescaled된 depth data는 추가적인 단계를 거칩니다. 이를 실현하기 위해, ImageNet에서 훈련된 네트워크는 특정 input distribution에 따라 학습되었다는 것을 기억해야합니다. 즉, depth sensor와는 호환되지 않죠. 그럼에도 불구하고 **<Fig 4>**를 보면, RGB 이미지에 많은 특징들을 정의할 수 있습니다. depth data도 볼 수 있죠. 이러한 것은 ImageNet으로 학습된 CNN을 위해 input으로 depth data의 rendered 버젼으로 사용하므로 만들어낼 수 있습니다. 본 논문의 실험에서 depth 에서 image로 만드는 것은 encoding 방법으로 할 수 있습니다. 두 encoding 방법이 있는데 (1)는  depth data를 grayscale로 만들고 grayscale 값을 세 channel로 만듭니다. (2)는 각 노멀 벡터의 차원이 결과 이미지의 한 채널에 해당하는 표면 노멀을 사용합니다. 더 복잡한 방법으로, HHA 인코딩이라 불리는 것은 세 channel에 지상 위의 높이, 수평 차이, 그리고 표면 노멀과 중력 방향 사이의 픽셀별 각도를 인코딩합니다. 

본 논문의 encoding은 HHA encoding보다도 훨씬 뛰어난 성능을 보여주는 것을 확인하였습니다. 모든 depth 값을 0~255사이의 값으로 정규화하고, 다음 jet colormap를 적용하여 주어진 image를 한 개의 channel에서 세 개의 channel로 변경시킵니다. 

distance에 map하여 color 값을 적용하였습니다. 가까운 것은 빨강, 먼 것은 파랑으로 depth 정보를 넣어 RGB channel로 만들었습니다. 네트워크가 RGB 이미지를 위하여 설계되었기 때문에, depth와 RGB image 사이의 흔한 구조를 제공하여 적절한 특징을 학습하도록 만듭니다. 

---

#### 실험 결과

<p align="center"><img src="../../assets/images/091203.png" width="300px" height="300px" title="Multimodal" alt="Multimodal" ><img></p>

depth를 encoding하여 RGB channel에 합하니 굉장한 성능을 보여주네요. 

