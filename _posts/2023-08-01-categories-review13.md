---
title: "CLIP"
excerpt: "Learning Transferable Visual Models From Natural Language Supervision 리뷰"

categories:
  - 컴퓨터 비전
tags:
  - [딥러닝, 논문 리뷰, Transformer]

permalink: /categories9/review13/

use_math: true
toc: true
toc_sticky: true

date: 2023-08-01
last_modified_at: 2023-08-01
---

# 배경

computer vision system의 sota(State-of-the-art)는 object categories가 미리 결정되어 있는 고정된 set로 학습합니다. 즉 예측되는 class가 미리 고정되어 있죠. 이러한 형태는 일반성과 사용성을 제한합니다. 추가적인 labeled된 data가 들어올 수 있는 computer vision 컨셉에 맞지 않는 것이죠. 이미지에 대한 정보를 직접 학습하는 것이 유망한 대안으로 소개되지만, 본 논문이서는 어떤 이미지와 어떤 캡션(설명)이 일치하는 지 예측하는 간단한 사전학습 작업이 인터넷에서 수집한 4억 쌍의 데이터셋에서 SOTA 이미지 표현을 효율적이고 확장 가능하게 학습하는 방법임을 보여줍니다. 

사전학습 뒤에는, 자연어가 학습된 시각적 개념을 reference하기 위해 사용되며 이는 모델의 zero-shot transfer을 가능하게 합니다. zero shot transfer은 모델이 학습 단계에서 보지 못했던 새로운 task나 class에 대해 예측을 수행하는 능력을 말합니다. 본 논문에서는 30개가 넘는 다른 computer vision datasets에 성능을 비교해보았습니다. 이 모델은 dataset specific training의 필요성 없는 완전한 지도 학습 baseline과 비교될 만하며, 대부분의 task에 대해 비교적 복잡한 방식으로 적용될 수 있습니다. 

예를 들면, 학습되었던 128만개의 학습 예시를 사용하는 것 없이, ImageNet zero-shot에 ResNet-50의 정확도와 본 논문의 모델이 같게 됩니다. 

본 논문의 코드는 아래에 있으며 오늘은 이 모델 (CLIP)에 대해 알아보겠습니다.

[CLIP code](https://github.com/OpenAI/CLIP)

### 방법

<p align="center"><img src="../../assets/images/080103.png" width="400px" height="400px" title="CLIP" alt="CLIP" ><img></p>

CLIP은 Contrastive learning을 수행합니다. 두 개의 input에 대해서 각각 encoder를 적용해서 나온 2개의 embedding의 유사도를 계산하는 방식으로 이뤄집니다. 

<p align="center"><img src="../../assets/images/080104.png" width="400px" height="400px" title="CLIP" alt="CLIP" ><img></p>

test에 대한 encoder 값을 만들고, 여러 label에 대한 text embedding을 각각 만들어냅니다. 학습에 사용되지 않은 image를 넣어서 embedding을 해주고, text embedding과의 유사도를 비교해서 가장 유사도가 높은 항목을 text label로 선정합니다. 이렇게 학습에 이요되지 않은 image가 들어와도 label prediction이 이뤄질 수 있기에 zero-shot learning이 가능합니다.

### 결과

<p align="center"><img src="../../assets/images/080105.png" width="400px" height="400px" title="CLIP" alt="CLIP" ><img></p>

<p align="center"><img src="../../assets/images/080106.png" width="400px" height="400px" title="CLIP" alt="CLIP" ><img></p>

결과는 매우 낙관적입니다. zero-shot CLIP이 다른 CNN이나 BiT-M에 비해 정확도와 속도가 빠르네요. 