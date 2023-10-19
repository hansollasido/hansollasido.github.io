---
title: "SoC"
excerpt: "SoC 구조"

categories:
  - 반도체
tags:
  - [SoC]

permalink: /categories16/semi3/

use_math: true
toc: true
toc_sticky: true

date: 2023-10-10
last_modified_at: 2023-10-10
---

## SoC

원래 System-on-board 였던 것이 Transistor가 작아짐에 따라 on-chip으로 통합할 수 있게 되었다. Chip 하나에 여러 시스템을 갖추게 되는데 이를 System-on-Chip 줄여서 SoC라고 부른다. 

---

**Architecture**

- Definition 
  - Architecture은 HW와 SW가 만들어지는 방법에 대한 것임.
  - 설계 철학에 기반한 구조임
  - system 동작에 영향을 주는 기본적인 구성원을 다루며 이에 따라 capabilities와 limitations이 정의됨. 

---

**Architecture vs Design**

- Architecture은 non-functional 결정이 되는 곳과 functional requirements가 분할되는 경우 
- Design: functional requirements가 완성되는 경우. 
- fuctional requirements은 Scalability, Availability, Safety, Security, testability, Manufacturability 등이 될 수 있음. 이것이 domain이 됨. 

-> 중요한 건 가끔 borders are blurred된다는 사실. 

---

**A system-on-chip contains Multiple Compute Engines**

- Main processor (CPU)
  - compute engine for running rich software. runs device's operating system, applications and user interface. 

- Graphics processor (GPU)
  - generating 2D/3D images and executing high-parallelized workloads such as NN

- Digital signal processors (DSPs)
  - radio control, audio and image processing

- Accelerators
  - heavily-optimized data processors for frequently-used tasks. ex : video, computer vision. 

--- 

**Computing Element Programmability**

- Dedicated HW
  - Computes pre-defined specific function for target application
  - Function defined at fabrication time (design time)

Dedicated HW는 미리 정의해 둬서 overhead가 적게 만듦. 

- GP-Processor
  - Computes "any" computable function on the fly via software programs

General Purpose이기 때문. 

