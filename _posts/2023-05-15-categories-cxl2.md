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

---

## 정리

CXL (Compute Express Link)는 host processor과 가속기, memory buffer, I/O devices와 같은 devices 사이에 높은 bandwidth와 낮은 latency connectivity를 제공하는 open industry standard이다. AI, Machine Learning, 데이터 분석 등의 applications에서의 computational workloads을 해결해준다. 이런 applications들을 통합하면서도 data를 처리해야하는 중요성 때문에, CPU, GPU, FPGA, smart NICs와 다른 가속기에서 scalar, vector, matrix 그리고 spatial architectures의 다양한 혼합이 요구된다. 

<p align="center"><img src="../../assets/images/051601.jpg" width="700px" height="700px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 1. CXL version 성능></center>

CXL 3.0은 PCle 6.0 기술을 기반으로 추가적인 latency 없이 64GT/s의 속도를 가지고 있다. 또한 Coherency를 이전 버젼보다 높였는데, Host caches를 back invalidate하므로 가능했다. 

2.0 버젼에서 memory pooling을 도입했었는데 memory pooling으로 인해 다른 서버에서의 유연한 allocating과 deallocating이 가능했다. 

<p align="center"><img src="../../assets/images/051602.jpg" width="700px" height="700px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 2. 2.0 버젼에서의 Pooling></center>

3.0 버젼에서는 memory sharing을 도입하려고 한다. memory sharing은 CXL에 붙인 memory 역할이다. 하나의 host보다 더 많은 memory 공간을 즉시 줄 수 있어 memory pooling보다 성능이 좋다. 이 시스템은 memory 구조를 공유한 넓은 문제를 해결할 수 있다. 

<p align="center"><img src="../../assets/images/051603.jpg" width="700px" height="700px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 3. Fabric></center>

3.0 버젼에서는 fabric을 도입하는데, PCle와 이전 CXL 세대의 트리 기반 구조를 넘어선 모델이다. CXL fabric은 *Port Based Routing (PBR)*이라 불리는 새로운 scalable addressing mechanism을 사용하여 서로 소통하도록 4096까지의 노드를 지원한다. 

<p align="center"><img src="../../assets/images/051604.jpg" width="700px" height="700px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 4. GFAM, Shared memory and NIC></center>

