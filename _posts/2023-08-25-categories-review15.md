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

SRAM은 가장 이해하기 쉬운 동작과 제조 기술이 오랜 기간동안 발전된 반도체 모델입니다. 이는 PIM을 설계하는 어려움을 줄여주고, 인공 신경망 가속기를 제조하는데 가장 인기있는 후보군이 되었죠. 

하지만 PIM 구조를 만들기에는 몇 가지 trade-off가 있습니다. 

SRAM-based PIM은 Analog PIM과 Digital PIM이 있습니다. Analog PIM은 인공 신경망의 유연성과 분류 정확도에 한계가 있는 반면에 높은 energy/area 효율성을 보여줍니다. 반면에 Digital PIM은 낮은 효율성과 throughput을 보여주는데, 대신 물리적 변화에 대응하기가 좋습니다. 

sota SRAM-based PIM은 아래 <Table I>에 있습니다. 

<p align="center"><img src="../../assets/images/082801.png" width="500px" height="500px" title="PIM Overview" alt="PIM Overview" ><img></p>

---

#### SRAM 동작과 구조 Background

<p align="center"><img src="../../assets/images/082802.png" width="500px" height="500px" title="PIM Overview" alt="PIM Overview" ><img></p>

위 **<Fig 1(a)>**를 보면 전통적인 6T SRAM cell의  write와 read을 볼 수 있습니다. 여섯 개의 transistor로 구성되어 있으며, 두 개의 inverter가 cross-coupled된 모형으로 되어 있습니다. 쓰여질 data는 wordline이 켜지기 전에 bitline에 load됩니다. 그리고 나서, bitline pair에 있는 data를 SRAM cell의 storage node에 갈 수 있도록 access transistor을 킵니다. 

0을 writing할 때, NMOS access transistor가 높은 전압보다 낮은 전압에서 passing하는 능력이 좋기 때문에, SRAM write 동작에 한계가 있습니다. 그러므로, access transistor은 SRAM cell안에 inverter의 threshold voltage보다 더 낮은 Q를 위하여 PMOS transistor보다 강해야 합니다. 이렇게 되면 0을 writing하기에 적절하죠. 

자세한 내용은 [링크](https://boaaaang.tistory.com/entry/SRAM%EC%9D%98-%EB%8F%99%EC%9E%91-%EC%9C%A0%EC%A7%80-%EC%9D%BD%EA%B8%B0-%EC%93%B0%EA%B8%B0-%EB%8F%99%EC%9E%91)를 참고하시길 바랍니다. 

이러한 SRAM cell이 연결되어 그림 **<Fig 1(b)>**처럼 구성되어 있습니다. Memory Arrary, Row Decoders, Column Multiplexers, Sense Amplifiers, Write Drivers, Controller로 구성되어 있습니다. 

Write 동작할 동안 write driver가 쓰여질 data를 선택된 bitline pair에 보내고, 선택되지 않은 bitline들은 wordline이 활성화되기 전에 높은 voltage로 pre-charged됩니다. 오직 하나의 cell이 read 동작동안 선택되며, 이에 대응되는 bitline pair가 column multiplexer를 통해 sense amplifier로 연결됩니다. 

---

#### Analog SRAM-Based PIM

<p align="center"><img src="../../assets/images/082803.png" width="500px" height="500px" title="PIM Overview" alt="PIM Overview" ><img></p>

**<Fig 2(a)>**에서의 기본적인 6T SRAM cell로도 MAC operation이 가능한 memory macro를 만들 수 있습니다. binary input이 wordline(WL)에 들어오고, standard 6T SRAM이 MAC 모드로 동작할 때는 binary weight가 SRAM 내부 node에 저장됩니다. Input 신호가 WL에 적용되자마자 하나의 SRAM cell에서 One binary multiplication이 동작합니다. 이에 상응하는 결과는 bitline pair (BL & BLb)에 반영되죠. 하나의 column에 있는 모든 bitcell의 multiplication 결과가 축적되고 bitline의 voltage drop을 만듭니다. Standard 6T SRAM cell은 write 와 read path를 공유한다는 부분에서 문제점이 있습니다. 

MAC 동작을 진행할 때, 전통적인 6T SRAM의 고유한 문제점을 해결하기 위하여 최근에는 몇 가지 다른 bitcell들을 제안하였습니다. foundry 8T SRAM cell은 read 문제점을 성공적으로 해결하였습니다. 두 개의 NMOS transistor가 standard 6T에 추가되어 독립적인 read bit-line (RBL)을 생성하고, ground에 discharging path를 두었습니다. multiplication 결과는 SRAM 내부의 노드 (Qb)에 저장된 weight value과 read wordline (RWL)의 voltage level로 표현된 input value에 의존됩니다. 하지만 foundry 8T는 큰 bitcell area를 가지는데, 오직 single-ended read 동작만 지원합니다. 

Custom 8T SRAM cell은 foundry 8T(i.e., weight '1/0' multiply input '1/0')에 비교하여 MAC operation(i.e., weight '+-1' multiply input '1/0')의 다양성을 향상시키기 위해 개발되었습니다. **<Fig 2(c)>**처럼요. 하지만 이런 설계도 여전히 동적 range와, current-based accumulation scheme로 인한 이상적이지 않는 특성에 문제를 가지고 있습니다. 

Differential voltage-mode SRAM-based PIM cell은 두 CMOS inverter와 하나의 XNOR을 standard 6T SRAM에 사용하여 동적 range를 향상시켰습니다. 하지만, 더 큰 cell size와 비이상적인 특성 때문에 여전히 문제가 있었습니다. 

Metal-oxide-metal capacitor (MOMCAP)과 8T SRAM은 passive capacitor 덕분에 analog 비이상적 특성을 최소화하합니다. **<Fig 2(d)>**가 그 구조입니다. MAC 동작을 분리시켜 SRAM 문제점을 제거하였습니다. 하지만 bitcell의 area overhead가 두 transistor와 하나의 capacitor 때문에 크게 늘어나죠. 

<p align="center"><img src="../../assets/images/082804.png" width="500px" height="500px" title="PIM Overview" alt="PIM Overview" ><img></p>

**<Fig 3>**은 analog SRAM-based PIM macro의 인기있는 동작 type을 설명하고 있습니다. 각 current-mode, voltage-mode, charge-sharing, 그리고 capacitor-coupling이 있습니다. current-mode (**<Fig 3(a)>**)에서는 같은 열에서의 각 6T SRAM cell에서 부터 나온 element-wise binary multiplication 결과를 discharge current를 생성하느냐 않느냐의 표현으로 축적하고, bitline pair에 voltage drop을 유도합니다. 제한된 동적 range는 축적 결과의 선형성을 보장할 필요성이 있습니다. 

Voltage-mode SRAM-based MAC operation scheme은 이런 동적 range (i.e., rail-to-rail)를 향상시키기 위해 고안되었습니다. 공유된 RBL은 pull-up 또는 pull-down을 각 bitcell의 multiplication 결과에 의해 병렬적으로 유도됩니다. **<Fig 3(b)>**처럼요. 하지만 voltage-mode 방법은 잔여 비선형성과 변동 문제점이 있습니다. 

Passive capacitor은 analog 계산에 의해 비선형성과 변동 문제를 최소화하기 위한 capacitive-coupling, charge-sharing을 적용하는데에 사용되었습니다. **<Fig 3(c)>**와 **<fig 3(d)>**를 보면, charge-sharing scheme은 각 cell에 두 개 이상의 switch가 필요하고 capacitive-coupling scheme에 비교하여 accumulation operation을 할 때 사이클이 더 걸리게 됩니다. 비록 charge-domain은 더 높은 energy 효율성을 제공하고 throughput performance를 제공하지만, 여전히 capacitor에 의한 area overhead가 문제점으로 지적되고 있습니다. 게다가 [Charge Injection (1)](#추가설명) 문제는 charge-domain PIM macro를 설계할 때, 중요하게 고려되어야 하는 상항입니다. 

---

#### Digital SRAM-Based PIM

Digital PIM 구조는 analog PIM에서의 문제, analog-to-digital/digital-to-analog converter (ADC/DAC) overhead와 비선형성 계산으로 인한 PVT 변동 문제를 해결하기 위하여 개발되었습니다. digital PIM 구조에서 계산은 전체적으로 digital이라 물리적 변동의 영향을 제거하고 data 전환이 더이상 필요하지 않게 만듭니다. 

---

### 추가설명

(1) Charge Injection
- MOSFET의 게이트를 통해 전하가 드레인 또는 소스로 주입될 때 발생함. 원치 않은 신호 변조, 회로의 정밀도와 성능 저하, 노이즈 및 다른 아날로그 아티팩트의 생성 등의 문제를 만듦
