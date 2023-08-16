---
title: "8/16"
excerpt: "SRAM, Docker"

categories:
  - 연구 일지
tags:
  - [linux, code, research, semiconductor]

permalink: /categories15/research2/

use_math: true
toc: true
toc_sticky: true

date: 2023-08-16
last_modified_at: 2023-08-16
---

SRAM과 Docker에 대해 잠시 공부했음.

PIM관련 개념이 자세히 나와있는 논문을 읽음. 아직 많이 읽어야 함.

<p align="center"><img src="../../assets/images/081603.png" width="500px" height="500px" title="research" alt="research" ><img></p>

CPU가 2개 일 때의 gem5 구조

<p align="center"><img src="../../assets/images/081604.png" width="500px" height="500px" title="research" alt="research" ><img></p>

CPU가 4개 일 때의 gem5 구조

현재 L2 Cache는 공유되고 있음. 이를 local하게 만들어주기 위해 변경이 필요함. 

[Tistory](https://ryotta-205.tistory.com/103)

위 블로그를 활용하여 L2 private cache를 만들 수 있을 것 같음. 