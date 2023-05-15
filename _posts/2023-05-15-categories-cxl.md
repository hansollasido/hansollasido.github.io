---
title: "[👍] CXL"
excerpt: "CXL 스터디"

categories:
  - CXL
tags:
  - [과제]

permalink: /categories10/cxl1/

use_math: true
toc: true
toc_sticky: true

date: 2023-05-15
last_modified_at: 2023-05-15
---

# CXL

[유튜브 링크](https://www.youtube.com/@CXLConsortium)

[홈페이지](https://www.computeexpresslink.org/resource-library)

---

Cloud computing이 중요해짐에 따라 data center 구조를 개선해야할 필요성이 높아짐. Increase Compute capacity, Deliver faster data processing 관점으로 개선.

GPU, FPGAs, AI Processor, smartNICs, Computational Storage가 이미 PCI Express로 연결되어 있지만 여러 종류의 시스템을 어떻게 더 최적화 해야하는지에 관련해서는 CXL (Compute Express Link)로 개선해야 했음.

CXL은 open interconnect standard로 Increases memory capacity와 Increases bandwidth, Enables lower latencies이 특징임.

**Bandwidth가 크면 좋은 점**

1. 데이터 전송 속도 좋아짐. 특히 대용량 데이터를 처리하는 작업에서는 대역폭이 큰 메모리가 중요함.
2. 시스템 반응성 향상. 실행 속도와 시스템의 반응성이 향상됨.
3. 다중 작업 처리 능력 향상. 병목 현상을 줄이고 다중 작업 처리 능력을 향상시킬 수 있음.
4. 그래픽 성능 향상. 대역폭이 큰 메모리는 그래픽 작업에서 성능 향상에 기여할 수 있음.

CXL은 cache coherent standard로 host processor과 cxl devices가 같은 데이터를 access할 수 있도록 함. CPU host는 성능 향상과 복잡도 감소가 목적. device cost와 overhead를 줄임. 

CXL 1.1은 3개의 protocol을 가짐. (CXL.io, CXL.cache, CXL.mem)
- CXL.io는 PCle 5.0와 비슷하며 intialization, link-up, device discovery와 enumeration, register access에 사용됨.
- CXL.cahe는 host와 device의 상호작용을 정의함. 효율적인 cache host memory, low latency, Request and response approach가 특징임.
- CXL.mem은 device-attach된 memory에 load와 store command를 사용하여 access함.

Type 1, 2, 3에 따라 사용하는 분야가 다름.

