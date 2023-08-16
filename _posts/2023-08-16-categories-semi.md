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

### Bitline & Wordline

SRAM Cell 구조에서 Bitline과 Wordline이 있음. 

메모리에서 매트릭스의 행을 제어하는 선이 WL(Wordline)이고 열을 제어하는 선이 BL(Bitline)임. 

메모리 cell은 이러한 line을 공유하여 사용하고 있음. 

