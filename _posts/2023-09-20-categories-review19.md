---
title: "Predator"
excerpt: "Predator: A Predictable SDRAM Memory Controller"

categories:
  - DRAM
tags:
  - [반도체, DRAM]

permalink: /categories9/review19/

use_math: true
toc: true
toc_sticky: true

date: 2023-09-20
last_modified_at: 2023-09-20
---

(9/20 작성중)

SDRAM Memory Controller 관련된 논문입니다. 2007년 CODES+ISSS에서 publish된 논문인데, 오래된 논문이지만 Memory Controller관련된 개념을 잘 설명하고 있는 좋은 논문입니다. 

---

### Abstract

SoC multi-processor에서 intellectual property(IP)의 memory 요구사항이 점차 증가하고 있습니다. SDRAM은 이전의 request에 매우 변칙하는 access time을 갖고 있습니다. 이는 정확하게 설계단계에서 유용한 bandwidth와 latency를 결정하기 어려움이 따릅니다. 

본 논문의 주요 내용은 memory controller가 IP의 최소 bandwidth와 최대 latency를 보장하는 것입니다. 이는 예측가능한 SDRAM sharing에 two step 새로운 방법을 사용해서 가능했습니다. 첫번째로, efficiency와 latency를 알고 있는 가운데, 미리 계산한 SDRAM commands의 sequence에 상응하는 memory access group을 정의합니다. 두번째로, 예측가능한 arbiter를 사용하여 이 group들을 동적으로 schedule할 수 있게 합니다. 이러면, bandwidth와 latency를 IP에 맞게 할당할 수 있게 되죠. 

이 방법은 SDRAM의 모든 generation에 사용가능합니다. 

---

### Introduction

Chip 설계가 복잡해지고 있습니다. 현재, SoC는 수 많은 IP를 사용하여 HW Accelerator와 shared memory와 소통이 가능한 cache가 있는 processor를 구현하기도 합니다. memory traffic이 역동적이고 request가 memory controller에 도착하는 것이 설계 단계에서 완벽히 알지 못합니다. DDR2 SDRAM과 같이 high-speed external memory를 사용하여 memory capacity를 어느정도 보장하는데, 이러한 memory들은 SoC 성능에 main이기 때문에, 효율적으로 활용되어야 합니다. 외부 memory의 사용가능한 bandwidth의 양은 traffic에 의존됩니다. 

외부 memory controller가 존재하는 것은 SoC의 복잡도가 증가하는 면에서 유연하기 다루기가 어려우며, 설계 단계에서 verification하는 것도 지원하지 않습니다. 다른 controller는 유연한 동적 scheduling을 사용하여 bandwidth를 최대화하지만 이는 request의 latency를 bounding하기가 어려워집니다. 따라서 제공된 net bandwidth는 simulation을 통해서만 추정될 수 있으며, bandwidth 할당은 requestor가 추가, 제거 또는 구성을 변경할 때마다 re-evaluated가 되어야 하는 어려운 작업입니다. 

본 논문의 주요 작업은 최소 net bandwidth와 최대 latency 경계를 지키는 memory controller를 설계하려고 합니다. two-step으로 이뤄져 있는데, 첫 번째는 efficiency와 latency를 알고 있는 SDRAM의 명령어에 상응하는 read와 write group을 정의하는 것입니다. 두 번째로는 Credit-Controlled Static-Priority (CCSP)로 run-time에서 동적으로 scheduled되는 requestor입니다. 이 CCSP는 arbiter로 rate regulator와 scheduler로 구성되어 있습니다. 

---

### Sharing SDRAM

이 Section에서 예측가능한 방법으로 sharing SDRAM에 있는 문제점을 해결해보겠습니다. 

---

#### SDRAM operation

SDRAM은 세 차원의 layout으로 구성되어 있습니다. bank, row와 column으로 구성되어 있습니다. 

<p align="center"><img src="../../assets/images/092001.jpg" width="500px" height="500px" title="Predator" alt="Predator" ><img></p>

**<fig 1>**을 보면 한 bank는 row와 column으로 구성되어 있습니다. SDRAM access할 시, request의 address는 bank, row, column address로 디코딩됩니다. Bank는 두 state를 가지는데, idle과 active입니다. *activate (ACT)* command를 받으면 idle state에서 activated되는데, row buffer에 requested된 row를 load합니다. Bank가 activated되면, row buffer에 있는 column으로 access될 수 있는 *read (RD)*와 *write (WR)* burst을 받습니다. 이 burst들은 4나 8 Word로 *burst length (BL)*를 프로그램화가 가능합니다. 마지막으로 *precharge (PRE)* 명령어를 bank에 주어 idle state로 만듭니다. Read와 Write 명령어는 auto-precharge flag를 사용하여 transfer가 된 이후에 자동으로 precharge하게 만듭니다. 그다음 *refresh (REF)* 명령어로 주기적으로 refresh하게 만들죠. 만일 clock cycle 동안 command가 들어오지 않을 경우 *no-operation (NOP)* 명령어를 보냅니다. 

<p align="center"><img src="../../assets/images/092002.jpg" width="500px" height="500px" title="Predator" alt="Predator" ><img></p>

서로 다른 command간의 time delay가 있습니다. 이는 **<table 1>**에 있습니다. multi-bank 구조의 장점은 다른 bank들이 pipeline을 이룰 수 있다는 점입니다. 하나의 bank에서 data가 이동되는 동안, 다른 bank에서는 precharge와 이후에 들어올 request에 activated될 수 있죠. 이러한 과정을 **bank preparation**이라 부르며, 때때로 precharge와 activate delay를 없애기도 합니다. 

---

#### Memory efficiency

SDRAM은 memory timing 제한이나 memory controller policy로 인해 stall cycle이 발생하게 되면 완전히 사용되기가 어렵습니다. 이러한 concept을 *memory efficiency*라고 정의하죠. memory efficiency는 data가 이동될 때 clock cycle로 정의됩니다. 그러므로 net bandwidth는 peak bandwidth와 memory efficiency의 산물입니다. 

memory efficiency는 5 가지의 category로 나뉠 수 있습니다. 1) refresh efficiency, 2) read/write efficiency, 3) bank efficiency, 4) command efficiency, 5) data efficiency 입니다. refresh efficiency를 제외하고는 traffic 의존적이며, 그러므로 설계 단계에서 결정하기가 매우 어렵습니다. 

---

##### Refresh efficiency

SDRAM은 data를 계속 유지하는 것이 필요하기 때문에, 주기적으로 refresh를 해야합니다. refresh 명령어는 *tRFC* cycle이 완료되면 실행되어 평균적으로 매 *tREFI* cycle를  만듭니다. refresh 명령어가 나오기 전에, 모든 bank는 precharged되어야 합니다. refresh efficiency는 traffic과 독립적이며, 설계 단계에서 계산될 수 있습니다. refresh efficiency는 memory size에 의존되고 DDR2 memory에 95~99%를 차지합니다. 

---

##### Read/write efficiency 

SDRAM은 bi-directional data bus를가지고 있어, read to write나 write to read의 스위치 타임이 필요합니다. 이는 data bus의 방향이 바뀔 때, lost cycle를 초래합니다. DDR SDRAM의 data bus를 최대한 효율적으로 사용하기 위하여, read나 write 명령어는 *BL/2* cycle로 실행됩니다. 방향을 스위칭하는데 read나 write 명령어가 나오기 전에 추가적인 cycle의 수를 수적으로 표현하였습니다. read/write efficiency는 read/write 스위치 횟수에 의존되며, read/write를 알지 못하는 이상, 설계 단계에서 고려될 수 없습니다. 가장 안좋은 경우의 read/write efficiency는 70%의 read와 30% write인데 전체적으로 93.8%와 같습니다. 

---

##### Bank efficiency 

SDRAM의 access time은 꽤나 변칙적입니다. read나 write 명령어가 즉시 active row에 실행됩니다. 하지만 만일 명령어가 inactive row를 target으로 한다면 row miss가 되어, activate 명령어 다음으로 나오는 precharge를 먼저 해줘야합니다. 이는 추가적으로 *tRP*+*tRCD* cycle이 read나 write 명령어가 실행되기 전에 필요하게 되죠. *tRC* cycle은 같은 bank에 있는 또 다른 activate 명령어와 분리되어야 되기 때문에,  이 penalty는 커질 수 있습니다. 이 overhead는 bank efficiency로 인해 정의되며, request의 target address와 memory map으로 memory bank를 어떻게 mapping하는 지에 따라 크게 의존합니다. 

---

##### Command efficiency 

DDR device가 rising과 falling edge에서 data를 이동한다해도, 명령어는 매 clock cycle에만 실행될 수 있습니다. 때때로 activate나 precharge 명령어는 delayed될 수 있는데, clock cycle에 이미 다른 명령어가 실행되고 있다면 그럴 수 있습니다. 이는 row miss로 인해 read나 write 명령어가 delayed될 때 lost cycle를 초래합니다. 이러한 영향은 burst length에 연결되어 있습니다. 작은 burst일 수록 더 작은 efficiency를 낳죠. Command efficiency는 traffic 의존적이라 설계 단계에서 일반적으로 계산되지 않습니다. 하지만 95~100%사이로 측정되죠. 

---

##### Data efficiency 

Data efficiency는 request된 data를 포함하는 memory access의 한 일부분으로 정의될 수 있습니다. 외부 memory가 최소한의 burst length로 access될 때 100%보다 낮을 수 있습니다. 이 문제는 fine-grained request에 연관될 뿐만아니라, memory burst측면에서 어떻게 data를 할당할 것인지에 연관되어 있습니다. 이는 burst 길이로 균등하게 나눠질 수 있는 address에서 *BL* words를 access할 때 burst가 사용되기 때문입니다. data efficiency는 traffic 의존적이라, 설계 단계에서 계산될 수 없습니다. 

---

memory efficiency를 세분화하여 분석해보았습니다. 

<p align="center"><img src="../../assets/images/092003.jpg" width="500px" height="500px" title="Predator" alt="Predator" ><img></p>

**<fig 2>**는 worst-case일 때의 memory efficiency를 나타냅니다. 

---

### Proposed Solution

#### Memory access groups

본 논문의 memory controller 설계는 세 memory access groups인 read group, write group, refresh group이 있습니다. 

read와 write group은 모든 bank에서 연속적으로 하나의 read나 write burst로 구성되어 있습니다. 이는 bank preparation의 사용을 최대화하기 위해 memory access하는 최적의 방법입니다. 

이 access 방식의 문제점은 memory가 $BL \times n_{banks}\times = w_{mem}$ B의 고정된 세분화로 access된다는 것입니다. 여기서 $n_{banks}$는 bank의 수를 말하고, $w_{mem}$은 memory interface의 width를 나타냅니다. 4 bank와 8 words 길이의 burst를 가진 예시 memory는 64B와 동일하죠. 몇 SoC는 SDRAM의 효율성으로 설계되어 가능하다면 IP 사이즈를 조정할 수 있습니다. 이 세분화된 access는 전형적인 L2 cache 사이즈에 잘 맞으며, TriMedia 3270 같은 특정 processor의 L1 cache 라인에서도 잘 맞습니다. 

read나 write mask를 적용하여 더 작은 세분화된 access를 할 수 있고, data efficiency에 영향을 주는 원치않는 data를 버릴 수 있습니다. 

Read와 Write group은 requestor 사이의 interface를 제거할 수 있도록 설계되었습니다. 

<!-- 4 bank와 8 word의 burst를 가진 본 논문의 예시 memory는 64B입니다. 몇 SoC는 SDRAM의 효율성을 갖고 설계되며, 가능한  -->