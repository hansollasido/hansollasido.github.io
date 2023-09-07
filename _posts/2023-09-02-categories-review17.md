---
title: "Samsung PIM"
excerpt: "Hardware Architecture and Software Stack for PIM Based on Commercial DRAM Technology"

categories:
  - PIM
tags:
  - [딥러닝, 논문 리뷰, PIM, 반도체]

permalink: /categories9/review17/

use_math: true
toc: true
toc_sticky: true

date: 2023-09-02
last_modified_at: 2023-09-02
---

(9/2 작성중)

PIM중에서도 Computer Architecture 분야에서 PIM을 동작시킨 논문입니다. 

---

### Abstract

Deep Neural Network는 고성능의 off-chip memory bandwidth을 원합니다. 하지만, 물리적인 제약으로 인해 off-chip memory의 bandwidth를 증가시키기에는 어려운 부분이 있습니다. 

게다가, memory hierarchy 전역에 data를 이동시키는 것은 시스템 에너지 소모가 매우 크며, 데이터 재사용 또한 치명적입니다. 

이러한 bandwidth와 energy 효율성을 보장하기 위해 processing-in-memory(PIM) 구조가 각광받고 있으며, 2.5D/3D stacking 기법과 같은 기술이 나오고 있습니다.

과거 PIM 구조는 memory 제조자가 쉽게 다룰 수 없는 host processor와 application code에 변화를 요구합니다.

본 논문은 이러한 PIM 구조의 문제점을 해결하고자 혁신적이고 실용적인 PIM 구조를 제안합니다. 

실용적이고, 효율적인 시스템 레벨을 적용하기 위하여 20nm DRAM 기술을 적용, 수정되지 않은 commercial processor를 통합, 필요한 software stack 설치, source code의 변경없이 application 실행 등을 하였습니다.

시스템 레벨에서의 본 논문의 검증은 PIM이 memory-bound neural network kernels와 application을 11.2배, 3.5배 각각 성능을 증가시키는 것을 보여줬습니다. 

---

### Introduction

Deep Neural Network (DNN)은 computational characteristic에 의하 off-chip memory bandwidth가 높아야 합니다. 반면에 system의 에너지 효율성은 또한 연속적으로 상승해야하죠. data 이동간의 에너지 소모가 줄어들어야 합니다. 실제로 계산하는 것보다도, off-chip에서 data를 transferring하는 것이 에너지 소모가 극심하죠. **Processor execution units과 DRAM devices간의 data 이동이 각 layer의 성능과 에너지 효율성을 가장 큰 비중으로 담당하고 있습니다.**

**High Bandwidth Memory (HBM)**이 high bandwidth, bit 이동간 low energy을 보장하기 위하여 소개되었습니다. HBM은 bit 이동간의 energy를 더 낮추며, 더 높은 대역폭을 제공합니다. package 안에 common substrate에 processor die와 통합시켰기 때문에 가능했습니다. 

processor와 DRAM 사이의 I/O interconnect을 10~13배 더 많아졌으며, 기존 DDR4 DRAM보다 2~2.4배 energy가 줄어들었습니다. 그럼에도 최근 연구들은 model size가 급격하게 커지고, 계산량이 HBM 사용에도 메모리 병목현상이 발생하도록 만들고 있습니다. 

많은 PIM 구조가 나왔지만, 실제로 동작하는 증명 개념 (Proof-Of-Concept) 반도체조차 개발한 곳이 없을 뿐더러, 제품화한 곳 또한 없습니다. 
주요 이유는 두 가지 입니다. 첫 번째로 3D, 2.5D 통합은 그 자체로 기존 DRAM보다 processor를 위한 고대역폭을 자동으로 제공하지 않습니다. 이는 각 DRAM bank의 대역폭이 I/O device와 맞게 설계되었기 때문입니다. processor를 위한 고대역폭을 제공하기 위해, DRAM을 customization하는 것이 요구되는데 이는 비용을 증가시킵니다. 두 번째로, host와 in-memory processor간의 communication을 조직하기 위하여 이전 PIM 구조들은 주로 host processor와 application code의 눈에 띄는 수정을 요구하는데, 이는 memory 제조사가 쉽게 다룰 수 없습니다. 

본 논문은 이와 같은 challenge을 극복하고자, 혁신적이면서도 실용적인 PIM architecture를 제안합니다. 첫 번째로, DRAM의 주요 구성원 (sub-array, bank)를 건드리지 않습니다. 대신에 bank-level 병렬화를 적용하여 고대역폭, bit간 이동할 때의 energy 절약을 제공합니다. 즉, DRAM commodity와 쉽고 균일하게 통합할 수 있습니다. 두 번째로, 현대 commercial processor의 어떤 구성원(DRAM controller 포함)도 수정을 요구하지 않습니다. 

full software stack을 실행할 때에 system level의 실용성과 효율성을 설명하기 위하여 본 논문은 HBM2 설계를 기반으로 한 PIM-HBM을 제안합니다. 즉, 본 논문은 DRAM 제조사와 full software stack support와 수정되지 않은 commercial processor로 만들어진 최초의 2.5D-integrated HBM-based PIM 구조입니다. 

---

### Background

#### A. Characteristics of Modern DNN Applications

DNN이 발전되고 적용됨에 따라, level-2 BLAS (matrix-vector operations) 처럼 적은 연산에서도 off-chip memory access 또한 많아지게 되었습니다. 이는 **memory-bound**라는 대역폭 문제를 만들게 되었죠. 반면에, level-3 BLAS (matrix-matrix operations)은 on-chip cache에서 data reuse를 할 수 있습니다. 즉, compute capability (**compute-bound**)에 따라 성능이 달라지죠. 

초창기 DNN application은 compute-bound 특징이 있었지만 memory-bound layer의 수가 최근 유명한 DNN application에 점차 증가하였습니다. 예를 들면, RNN의 주요 함수는 matrix-vector multiplication인데 main memory bandwidth가 성능을 좌지우지합니다. 추가로 batch normalization (BN)과 skip connection layer은 low data reuse 특징으로 인해 memory-bound입니다. 


#### B. High Bandwidth Memory 

processor의 capability가 상당히 증가하면서, off-chip memory 대역폭 또한 요구되어 왔습니다. 이러한 요구를 만족하기 위해 HBM이 소개되었습니다. HBM 3D-stack은 DRAM die를 I/O 회로, memory built-in-self-test (MBIST), testing과 debugging을 하는 것들로 구성된 buffer die와 같이 쌓여져 있습니다. 고대역폭을 제공하기 위하여, DRAM die는 through silicon vias (TSVs)를 사용하여 buffer die와 communicate하고 buffer die는 package 안에 silicon interposer로 host processor과 연결되어 있습니다. 

하나 또는 그 이상의 HBM device와 host processor가 하나의 package에서 통합된 것은 종종 SiP(System-in-Package)라고 불립니다. <Fig 2>가 전형적인 ASIC와 HBM device와 HBM DRAM die 조직을 구성한 SiP를 보여주고 있습니다. 

<p align="center"><img src="../../assets/images/090501.png" width="500px" height="500px" title="HBM-PIM" alt="HBM-PIM" ><img></p>

HBM2 DRAM die는 4개의 pseudo channel (pCHs)를 구성하고 각각 4개의 bank group (BGs), 중간에 control logic, data bus, TSV I/O 회로를 구성하고 있습니다. Bank Group은 4개의 bank으로 구성되어 있으며, 4개의 bank는 datapath resource와 bank group I/O (BGIO)를 공유하고 있습니다. 중앙에 control logic은 DRAM command와 address pair (CA)를 받고 CA를 decode하여 bank control signal과 address를 생성하고 생성된 signal을 bank control line과 BGIO를 통하여 target bank에 보냅니다. 또한 data bus와 TSV I/O 회로를 CA 기반으로 control합니다. bank는 DRAM cell array, row, column decoder, I/O sense amplifier, write driver를 구성하고 있습니다. DRAM cell array는 sub-array로 구성되며, sub-array는 sub-wordline drivers (SWD), bit-line sense amplifiers (BLSAs), local I/O (LIO) lines을 구성하고 있습니다. sequence of data access operation은 DDR DRAM과 거의 같습니다. (activation (ACT), read (RD), write (WR), precharge (PRE)) 하지만 data access size는 다릅니다. 4 DRAM dies group은 rank를 구성하고 이 rank는 16개의 pCHs를 제공합니다. HBM Device는 더 큰 capacity를 위한 rank를 제공할 수 있지만, rank간 16pCHs가 공유됨에 따라 고대역폭을 제공하지는 않습니다. 

심지어 HBM으로도 현대 DNN에서 제한된 memory bandwidth로도 고생을 받고 있습니다. 한 가지 해결방법으로는 HBM device를 processor와 통합시키는 방법인데, 이는 power, 열, 뿐만아니라 I/O connection에 의해 현실적으로 만들기에 어려움이 따릅니다. 

<p align="center"><img src="../../assets/images/090502.png" width="700px" height="700px" title="HBM-PIM" alt="HBM-PIM" ><img></p>

---

### PIM-DRAM Architecture

bank-level 병렬성을 적용하여 고대역폭 on-chip을 실용적으로 사용하기 위하여 PIM 구조를 제안합니다. 비록 HBM2를 기반으로 설명되었지만, 기존 standard DRAM (DDR, LPDDR, GDDR)에서도 적용가능합니다.

---

#### Overview

<!-- 그림 1이 본 논문의 PIM 구조입니다. (a)에서 PIM-DRAM die, PIM-HBM DRAM die, (b)에서 PIM execution과 -->

