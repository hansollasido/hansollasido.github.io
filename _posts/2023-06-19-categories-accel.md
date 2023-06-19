---
title: "A Survey of Accelerator Architectures for Deep Neural Networks"
excerpt: "AI 가속기 전체적인 개념 및 흐름"

categories:
  - AI 가속기
tags:
  - [딥러닝, 논문 리뷰, AI 가속기]

permalink: /categories11/accel1/

use_math: true
toc: true
toc_sticky: true

date: 2023-06-19
last_modified_at: 2023-06-19
---

# (작성중)

# 논문

[논문 링크](https://www.sciencedirect.com/science/article/pii/S2095809919306356)

AI 가속기의 종류와 개념을 전체적으로 설명하고 있음.

---

# AI 가속기 

Big data 수용성과 computing power의 급성장으로 AI가 주목 받고 있음. 하지만 AI는 processing 속도와 기존 computer system의 규모에 의해 제한받고 있음. 이러한 상황에서 본 논문은 DNN(Deep Neural Network)를 위한 가속기 설계를 훑어보고, **(1) computing units**, **(2) dataflow optimization**, **(3) targeted network topologies**, **(4) architectures on emerging technologies, and accelerators for emergin applications** 시점으로 다뤄볼 예정. 또한 미래 AI 가속기 트렌드가 어떠할 것인지 논의.

---

## 배경지식


---

## On-chip accelerators

[general-purpose processing](https://dl.acm.org/doi/pdf/10.1145/2589750)을 위해 또는 [작은 neural network](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=7167250)를 위해 가속하는 on-chip 가속기

1. [NPU (Neural Processing Unit)](https://dl.acm.org/doi/pdf/10.1145/2589750)

2. [RENO: A reconfigurable NoC accelerator](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=7167250)

---

## Stand-alone DNN/convolutional neural network accelerator

1. [DianNao series](https://dl.acm.org/doi/pdf/10.1145/2996864)

2. [TPU](https://dl.acm.org/doi/pdf/10.1145/3079856.3080246)

3. **Data Flow analysis and architecture design**
- [Eyeriss (1)](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=7738524)
- [Eyeriss (2)](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=7551407)
- [GGRA-based dataflow processor](https://hc29.hotchips.org/)

---

## Accelerators with emerging memories

[ReRAM](https://www.nature.com/articles/nature06932)

[Hybird Memory Cube (HMC)](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=7477494)

ReRAM과 HMC는 PIM을 할 수 있게끔 memory 구조와 기술을 통합한 대표적 기술임.

**ReRAM-based DNN accelerators**

1. [PRIME: A Novel Processing-in-memory Architecture for Neural Network Computation in ReRAM-based Main Memory](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=7551380)
2. [ISSAC](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=7551379)
3. [PipeLayer](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=7920854)


4. [Neurocube](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=7551408)
5. [Tetris](https://arxiv.org/ftp/arxiv/papers/1811/1811.06841.pdf)

---

## Accelerators for emerging applications

1. Low precision neural network

---

## The future of DNN accelerators