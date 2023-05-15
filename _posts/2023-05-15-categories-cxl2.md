---
title: "[👊] CXL 3.0 Introduction"
excerpt: "CXL 스터디"

categories:
  - CXL
tags:
  - [과제]

permalink: /categories10/cxl2/

use_math: true
toc: true
toc_sticky: true

date: 2023-05-15
last_modified_at: 2023-05-15
---

# CXL
 
Bandwidth가 크면 accelerators, system memory, smartNICs, Leading-edge networking부분에서 성능이 높아짐. 

CXL 3.0은 memory-centric fabric architectures이자 capabilities가 향상됨. 이에 따라 scalability와 resource utilization이 좋아짐.

1. Fabric capabilities and management
2. Improved memory sharing and pooling
3. Enhanced coherency
4. Efficient peer-to-peer

으로 성능을 크게 나눌 수 있음

---

CXL 3.0은 data rate를 64G transfer per second로 2배 가량 늘림. 추가적인 latency와 device type없이 지원 가능함. 

CXL 3.0은 CXL 2.0와 다르게 dynamic capacity device에 지원가능함. cascade와 fanout에서 multiple switch level이 가능함. 

CXL 3.0은 CXL fabric 형태에서 지능적인 연결이 가능하고 어떤 형태로든 사용 가능함. 

<p align="center"><img src="../../assets/images/051501.png" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>

type 1, 2, 3을 어느 곳에서든 사용 가능함. Dynamic, Flexible, Intelligent한 것이 큰 특징임. 

또한 CXL 3.0은 2.0, 1.0 버젼과 호환이 잘 맞음. 즉 역호환성이 좋음. 
