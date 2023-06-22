---
title: "PRIME: A Novel Processing-in-Memory Architecture for Neural Network Computation in ReRAM-based Main Memory"
excerpt: "새로운 PIM 구조"

categories:
  - AI 가속기
tags:
  - [딥러닝, 논문 리뷰, AI 가속기]

permalink: /categories11/accel2/

use_math: true
toc: true
toc_sticky: true

date: 2023-06-21
last_modified_at: 2023-06-21
---

# 논문

[논문 링크](https://ieeexplore.ieee.org/document/7551380)

## Abstract

Processing-in-memory(PIM)이 "[memory wall (1)](#추가설명)"을 해결할 수 있는 유망한 해결책입니다. 이전 PIM 구조들은 메모리 근처에 computation logic을 붙여서 구현했습니다. [metal-oxide resistive random access memory (ReRAM) (2)](#추가설명)은 matrix-vector multiplication을 효율적으로 수행할 수 있고, Neural Network(NN)을 가속하는데에 널리 쓰이고 있습니다. 본 논문은 새로운 PIM 구조인 **PRIME**에 대해 설명할 것이며, 이 구조는 ReRAM 기반 메인 메모리에서 NN을 가속화 하는데 좋은 성능을 보여줍니다. 

ReRAM과 PIM 구조의 효율성을 가지고 PRIME은 이전의 NN 가속기보다 속도, 에너지 측면에서 더 좋은 성능을 보여줍니다. PRIME은 machine learning 벤치마크를 평가하는데, ~2360x배의 성능 향상과 ~895x배의 에너지 소비 감소를 볼 수 있었습니다.

PRIME 구조에 대해 본격적으로 알아보겠습니다.

---

## PRIME 구조 

<p align="center"><img src="../../assets/images/062101.png" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

Figure 3(c)가 PRIME 구조입니다. 이전 NN 가속기는 추가적인 Processing Units (PU)를 부착했지만, PRIME은 직접 ReRAM cell을 활용하여 추가적인 PU없이 수행하였습니다. 

PRIME구조에는 Memory subarrays, Full Function(FF) subarrarys, Buffer subarrays가 있습니다. 

Mem subarrays는 data storage 수용성만 가지고 있으며, 최근에 설계된 성능 최적화된 ReRAM 메인 메모리와 비슷하게 구성되어 있습니다. 

FF subarrays는 computation과 data storage 수용성을 둘다 가지고 있으며 두 개의 모드를 동작할 수 있습니다. 메모리 모드에서 FF subarrays는 이전과 같은 메모리와 같이 동작하고, Computation 모드에서는 NN 계산하는 데에 동작합니다. PRIME controller가 있어 operation과 FF subarrays의 재구성을 컨트롤할 수 있습니다. 

Buffer subarrays는 FF subarrays에 data buffer 역할을 수행합니다. FF subarrays와는 따로 있는 data port를 사용하기 때문에, Mem subarrays의 bandwidth를 차지하지는 않습니다. Buffer subarrays가 data buffer로 사용되지 않을 때에는 일반적인 메모리로 사용됩니다. 

Figure 3(c)를 보면 FF subarrays가 in-memory data 이동의 높은 bandwidth를 가지고 있으며, Buffer subarrays 덕분에 CPU와 병렬적으로 동작할 수 있습니다. 

---

### FF Subarray Design

FF subarray는 storage와 computation을 최소한의 area overhead로 지원하는 것입니다. 이러한 목표를 달성하기 위하여 본 논문은 주변 회로를 storage와 computation를 위하여 최대한 재사용했습니다. 

<p align="center"><img src="../../assets/images/062102.png" width="900px" height="900px" title="OP code 예시" alt="OP code" ><img></p>

---

#### Decoder and Driver

Figure 4(A)를 보면 밝은 파란색으로 마킹된 decoder와 driver를 볼 수 있습니다. 

우선 multi-level voltage sources를 wordlines에 부착합니다. 이렇게 하면 정확한 input voltage를 공급할 수 있죠. NN computation은 모든 input data를 즉시 해당하는 wordline에 넣어야만 합니다. 그러므로 input voltage를 control하는 latch가 추가됩니다. control signal은 voltage source의 조합으로 결정되며, 요구되는 input voltage를 공급합니다.

두 번째로는 아날로그 신호를 wordline으로 이송하기 위해 각 wordline에 분리된 amplifier를 부착하였습니다.

세 번째로는 메모리 모드에서 사용되는 두 voltage level보다 NN computation은 $2^{P_{in}}$ level의 input voltage를 필요로 합니다. 

마지막으로 두 개의 crossbar array를 사용하여 positive와 negative weights를 각각 저장하고, 같은 input port를 공유할 수 있게끔 만들어줍니다. 

---

#### Column Nultiplexer

NN computation을 지원하기 위하여, figure 4 (B)에서 밝은 파랑색으로 마크된 구성원들을 추가함에 따라 ReRAM에 column multiplexer를 수정하였습니다. 수정된 multiplexer는 analog subtraction unit과 non-linear threshold (sigmoid) unit인 두 processing units을 결합하였습니다. 큰 NN이 여러 crossbar arrays에 매핑될 때 sigmoid unit은 우회합니다. 추가로, FF subarrary가 memory 모드와 computation 모드 사이에 bitline을 스위치 하기 위하여 multiplexer를 각 bitline에 달아 스위치를 control할 수 있게끔 만들어 줍니다. 

---

#### Sense Amplifier

figure 4 (C)는 SA 설계를 보여줍니다. 

---


## 추가설명

**(1) memory wall**
- CPU와 메인 메모리 사이의 성능 격차를 가리키는 용어. CPU는 처리 속도가 높은 반면, 메인 메모리는 느림. 

**(2) ReRAM**
- Resistive Random Access Memory의 약자로, 비휘발성 메모리이며, 전압을 가하면 저항이 가리는 것을 활용하여 0과 1을 저장함. flash memory보다 낮은 전력 소모, 더 높은 속도, 충분한 수명과 내구성, 더 높은 밀도를 가지고 있음. 