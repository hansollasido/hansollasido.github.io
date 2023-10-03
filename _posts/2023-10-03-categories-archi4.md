---
title: "컴퓨터구조 공부4"
excerpt: "Pipeline"

categories:
  - 컴퓨터구조
tags:
  - [컴퓨터구조]

permalink: /categories22/컴퓨터구조4/

use_math: true
toc: true
toc_sticky: true

date: 2023-10-03
last_modified_at: 2023-10-03
---

## Computer Architecture

우리가 항상 사용하는 컴퓨터는 정말 복잡한 시스템으로 구성되어 있다. 

---

### Pipeline

한 test를 끝내고 다른 test를 진행하기에는 시간 소모가 매우 크다. 

<p align="center"><img src="../../assets/images/100305.jpg" width="500px" height="500px" title="Pipeline" alt="Pipeline" ><img></p>

위 처럼 task를 여러 stage로 나누어서 stage마다 task를 진행하면 더 효율적인 결과를 낳을 수 있다. 

RISC-V에서는 5 stage로 나눈다. 

- IF: Instruction fetch from memory
- ID: Instruction decode & register read
- EX: Execute operation or calculate address
- MEM: Access memory operand
- WB: write result back to register

Pipeline을 사용하면 훨씬 빠른 속도로 연속된 instruction을 수행할 수 있다. 

$\mbox{Time between instructions}_{\mbox{(pipelined)}}=\frac{\mbox{Time between instructions}}{\mbox{Number of stages}}$

<p align="center"><img src="../../assets/images/100306.jpg" width="500px" height="500px" title="Pipeline" alt="Pipeline" ><img></p>

stage마다 register를 두어서 pipeline이 될 수 있게 설계한다. 

<p align="center"><img src="../../assets/images/100307.jpg" width="500px" height="500px" title="Pipeline" alt="Pipeline" ><img></p>

Pipeline하게 되면 위 그림처럼 stage별로 수행하게 된다. 

<p align="center"><img src="../../assets/images/100308.jpg" width="500px" height="500px" title="Pipeline" alt="Pipeline" ><img></p>

Control Signal까지 추가된 모습은 위와 같다. 

하지만 Pipeline을 적용시 해결해야할 문제가 있다. 

바로 **Hazard**다. 

