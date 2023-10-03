---
title: "컴퓨터구조 공부5"
excerpt: "Hazard"

categories:
  - 컴퓨터구조
tags:
  - [컴퓨터구조]

permalink: /categories22/컴퓨터구조5/

use_math: true
toc: true
toc_sticky: true

date: 2023-10-03
last_modified_at: 2023-10-03
---

## Computer Architecture

우리가 항상 사용하는 컴퓨터는 정말 복잡한 시스템으로 구성되어 있다. 

---

### Hazard

Hazard는 pipeline 적용시 해결해야하는 문제이다. 

종류는 다음과 같다. 

- Structure hazard
  - A required resource is busy
- Data hazard
  - Need to wait for previous instruction to complete its data read/write
- Control hazard
  - Deciding on control action depends on previous instruction

---

**Structure Hazard**

resource 사용할 때 충돌로 발생하며, 이는 singe memory로 pipeline을 적용할 때 발생한다. Fetch하는 동안 load/store하는 경우에는 memory를 하나 쓰고 있을 때, 문제가 생긴다. 따라서 instruction memory와 data memory를 따로 두어 해결한다. 

---

**Data Hazard**

이전의 instruction 결과의 data를 access하는 경우 발생한다. 

예를 들면 

- add x19, x0, x1
- sub x2, x19, x3 

일 때, x19의 값은 x0와 x1를 더하고 난 뒤의 값으로 sub를 진행해야한다. 

**Forwarding (aka Bypassing)**

Data Hazard를 해결할 수 있는 방법으로 Forwarding이 있다. 이는 추가적인 datapath를 주어서 compute가 되면 바로 사용할 수 있게 만든다. 

<p align="center"><img src="../../assets/images/100309.jpg" width="500px" height="500px" title="Hazard" alt="Hazard" ><img></p>

실제로 data hazard는 많이 발생하는 hazard이다. 위 그림처럼 forwarding을 주었지만 instruction이 아래와 같이 연속으로 오게되면, forwarding을 복잡하게 해줘야한다. 

- sub x2, x1, x3
- and x12, x2, x5
- or x13, x6, x2
- add x14, x2, x2
- sd x15, 100(x2)

forwarding할 시에는 추가적인 path를 놓아야한다. data hazard가 발생하는 지는 아래와 같이 알 수 있다. 

1a. EX/MEM.RegisterRd = ID/EX.RegisterRs1
1b. EX/MEM.RegisterRd = ID/EX.RegisterRs2
2a. MEM/WB.RegisterRd = ID/EX.RegisterRs1
2b. MEM/WB.RegisterRd = ID/EX.RegisterRs2

전부 register 사용할 때 발생한다. 

<p align="center"><img src="../../assets/images/100310.jpg" width="500px" height="500px" title="Hazard" alt="Hazard" ><img></p>

**Double data hazard**

- add x1, x1, x2
- add x1, x1, x3
- add x1, x1, x4

위와 같이 instruction에서 연속으로 하나의 register를 원할 때, double data hazard가 발생한다. 이와 같은 경우, 추가적인 forwarding condition을 준다. 

<p align="center"><img src="../../assets/images/100311.jpg" width="500px" height="500px" title="Hazard" alt="Hazard" ><img></p>

**Load-Use Data Hazard**

forwarding을 한다고 항상 stall을 피할 수 있는 건 아니다. 아래와 같은 경우에는 무조건 stall이 필요하다. 

<p align="center"><img src="../../assets/images/100312.jpg" width="500px" height="500px" title="Hazard" alt="Hazard" ><img></p>

stall은 nop signal을 보내어 bubble을 만들어 준다. 

<p align="center"><img src="../../assets/images/100313.jpg" width="500px" height="500px" title="Hazard" alt="Hazard" ><img></p>

Hazard Detection을 포함한 datapath는 위 그림과 같다. 

---

Stall은 성능을 낮추는 요인이 된다. 따라서 stall을 최소화 하는 것이 핵심이다. 

stall을 최소화하는 방법으로는 code scheduling이 있다. code scheduling으로 instruction 순서를 바꿔 결과는 같지만 stall이 발생하지 않도록 하여, cycle의 수를 줄일 수 있다. 

---

**Branch**

Branch가 있을 경우에는 어떻게 할까? Branch가 있을 경우, branch와 관련없는 instruction을 끈다. 그리고 해당하는 instruction에 가게 되는데, RISC-V pipeline에서는 branch 결과를 미리 예측하고, 미리 fetch하기 때문에 문제가 없다. 

<p align="center"><img src="../../assets/images/100314.jpg" width="500px" height="500px" title="Hazard" alt="Hazard" ><img></p>

위와 같이 branch를 판별하고 

<p align="center"><img src="../../assets/images/100315.jpg" width="500px" height="500px" title="Hazard" alt="Hazard" ><img></p>

위와 같이 branch가 있을 때, 필요없는 instruction을 bubble하여 stall을 최소화한다. 

---

**Branch Prediction**

Branch가 있는지 없는지 예측하는 것이 어떻게 보면 가장 중요할 것처럼 보인다. Branch를 예측하면 stall을 최소화할 수 있기 때문이다. 

1-bit Predictor와  2-bit Predictor가 있다. 