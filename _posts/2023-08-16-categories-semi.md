---
title: "SRAM"
excerpt: "SRAM 구조 및 동작원리"

categories:
  - 반도체
tags:
  - [SRAM]

permalink: /categories16/semi/

use_math: true
toc: true
toc_sticky: true

date: 2023-08-16
last_modified_at: 2023-08-16
---

## SRAM

<p align="center"><img src="../../assets/images/081601.jpg" width="500px" height="500px" title="SRAM" alt="SRAM" ><img></p>

SRAM은 Static Random Access Memory의 줄임말로, 휘발성 메모리로 동작함.

DRAM(Dynamic Random Access Memory)보다 빠르지만, 그만큼 overhead가 있음. 

집적도 낮고, 가격이 비싸며, 전력소비도 큼.

빠르다는 장점 때문에 L1, L2, L3 Cache에 많이 쓰임.

---

### Bitline & Wordline

SRAM Cell 구조에서 Bitline과 Wordline이 있음. 

메모리에서 매트릭스의 행을 제어하는 선이 WL(Wordline)이고 열을 제어하는 선이 BL(Bitline)임. 

메모리 cell은 이러한 line을 공유하여 사용하고 있음. 

---

### Hold mode 

WL = 0(Low)가 되면 Access TR가 OFF가 되어 안에 있는 inverter 회로가 현재 값을 계속 유지함.

DRAM에 비해 Refresh할 필요가 없어 자연스레 이전 값을 유지할 수 있음. 

---

### Read mode

WL = 1(High)가 되고 Access TR가 ON이 됨. BL과 /BL의 값을 Vdd로 pre-charge 시킴.

Pre-charge되면, Q와 /Q에 있는 값에 따라 BL과 /BL의 voltage값이 미세하게 변함. 

이 미세한 변화를 Sense Amplifier가 증폭하여 data를 read함. 

---

### Write mode

WL = 1(High)가 되고 Access TR가 ON이 됨. BL과 /BL을 반대되는 데이터를 주어 write할 수 있음. 

주기적으로 Refresh할 필요 없어, 정적(Static) Random Access Memory로 불림. 


