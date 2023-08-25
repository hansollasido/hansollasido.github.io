---
title: "PIM Circuits Overview"
excerpt: "An Overview of Processing-In-Memory Circuits for Artificial Intelligence and Machine Learning"

categories:
  - PIM
tags:
  - [딥러닝, 논문 리뷰, PIM, 반도체]

permalink: /categories9/review15/

use_math: true
toc: true
toc_sticky: true

date: 2023-08-25
last_modified_at: 2023-08-25
---

(8/25 작성중)

PIM에 대한 전체적인 OVERVIEW를 보여주는 논문입니다. 개인적으로 정말 도움이 많이 되었습니다.

오늘은 PIM의 전체적인 OVERVIEW를 확인해보겠습니다.

---

### Abstract

AI와 Machine Learning(ML)이 주목받는 가운데, 기존의 폰 노이만 기법으로는 Computing 성능을 개선하기에는 한계가 있습니다. 병목현상으로 인해 data가 이동할 때에 많은 energy와 time을 소모하기 때문입니다. 특히 AI, ML는 data 이동과 연산이 많기 때문에 폰 노이만 구조가 적절한 구조는 아닙니다. 

<p align="center"><img src="../../assets/images/082501.png" width="500px" height="500px" title="PIM Overview" alt="PIM Overview" ><img></p>

CPU가 Memory에 data를 요청하고 Memory는 data를 CPU에 보냅니다. 이 때, 시간 delay가 많이 발생하게 되는데 아무리 CPU가 빨라도 memory가 느리면 data가 온전히 받을 때까지 기다려야 합니다. 이 같은 현상을 **bottle neck(병목)**현상이라 합니다. 때문에 memory에서 data를 access하는 횟수가 적으면 적을수록 chip 속도는 빠르게 되는 것이죠.

이에 비교적 빠른 cache memory를 cpu 근처에 두어, 자주 쓰이는 data는 memory access 횟수가 적게 만듭니다. 병목현상으로 인해 전체적으로 chip 구조가 짜여지게 되는 것이죠. 

AI와 Machine Learning(ML)은 data 이동이 굉장히 많은 알고리즘입니다. 따라서 폰 노이만 구조를 따를 시, cache를 아무리 둬도 memory access 횟수가 많을 수밖에 없습니다. 이러한 data 이동 issue를 해결하기 위하여, 연산을 하는 module(processor)을 memory안에 둬서 data 이동을 최소화 하는 구조가 고안되었습니다. 그게 바로 **Process-In-Memory**인 것이죠.

AI와 ML은 굳이 CPU에서 data 연산을 진행할 필요가 없었습니다. ADD처럼 간단한 연산을 수행하기 위해서 폰 노이만 구조는 CPU가 memory에 data를 가져오고 그다음 더해서 memory로 보내는 형식이었는데, PIM은 CPU가 memory에 ADD 요청을 보내고 memory는 이 요청을 받아 memory내에 있는 data를 가지고 ADD를 연산한 뒤, memory에 저장합니다. 훨씬 간단하게 연산을 수행하게 되죠. 

PIM은 더 효율적인 알고리즘이 되며 energy-efficient하게 됩니다. 최신 PIM 논문들을 보면 **static-random-access-memory (SRAM)**, **dynamic-random-access-memory (DRAM)**, **resistive memory (ReRAM)**에 각각 PIM 구조를 적용하였는데, SRAM, DRAM, ReRAM 별로 원리를 파악하고 PIM구조가 어떤 것이 있는지 알아보겠습니다.

---

### Introduction

신경망 구조에서 **Multiple-and-Accumulate (MAC) operation**은 아주 중요한 연산입니다. 따라서 MAC 연산을 하기에 energy-efficient한 구조가 필요한데 그 중에 하나가 PIM 구조입니다. memory type에 따라 PIM구조는 SRAM-based, DRAM-based, ReRAM-based로 나뉠 수 있습니다. 

- SRAM은 간단한 동작 모드와 기술로 인해 가장 인기 있는 PIM 구조입니다. 하지만 SRAM cell은 DRAM, ReRAM보다 크다는 단점이 있어 memory density가 작다는 단점이 있죠.

- DRAM-based PIM은 큰 크기의 ML 모델을 큰 메모리 용량으로 가속하기에 적합한 구조입니다. DRAM자체가 density가 높기 때문이죠. 하지만 PIM 구조를 적용시키면 이러한 density가 낮아지는 문제가 발생하게 됩니다.

- ReRAM-based PIM은 요즘 핫한 energy-efficient 가속기 인데, ReRAM 기술은 아직 많이 연구되지 않았으며, ReRAM-based PIM은 다양한 설계 issues를 해결해야하는 필요성이 있습니다. 

---

### SRAM PIM