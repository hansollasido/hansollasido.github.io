---
title: "Fundamental Latency Trade-offs in Architecting DRAM Caches"
excerpt: "Outperforming Impractical SRAM-Tags with a Simple and Practical Design"

categories:
  - cache
tags:
  - [cache, 컴퓨터, 반도체]

permalink: /categories13/cache1/

use_math: true
toc: true
toc_sticky: true

date: 2023-05-18
last_modified_at: 2023-05-18
---

# 논문 요약

large-scale의 DRAM caches를 설계하는 것에 대한 trade-off를 분석하는 논문입니다. 이전 cache 디자인(tag, large associativity, replacement state)이 hit latency를 악화시켜 DRAM cache 성능을 낮춥니다. 이 논문은 hit rate와 latency를 위한 DRAM cache 구조 최적화를 진행합니다. 

latency에 최적화된 cache 구조를 Alloy Cache라고 불리는데, tag와 data를 single burst에서 함께 streaming하여 delay를 제거하였습니다. 

본 논문은 1 cycle의 latency와 core당 96 byte의 storage overhead를 유발하는 간단하고 높은 성능의 Memory Access Predictor를 제안합니다. 이는 일반적인 경우인 cache miss detection을 기다리는 것보다 cache miss를 더 빠르게 하는데 도움을 줍니다. 

본 논문는 latency가 최적화된 cache 설계가 Loh, Hill이 최근(2012) 제안한 논문보다 성능이 좋다는 것을 보여줍니다.

## Introduction

large cache로 부터 고성능을 내기 위해서는, key challenge가 있는데 바로 tag store, hit latency최적화 miss발생시 효율적 처리 방법 등이 있습니다. SRAM의 storing tag overhead를 DRAM의 tag를 사용하여 피하지만, 단순히 tag를 적용하면 DRAM cache의 latency는 두배가 됩니다. 

Loh와 Hill의 연구는 tags-in-DRAM 방법을 tag와 data를 같은 행에 배치하면서 효율적으로 만들었습니다. 하지만 예전 연구와 마찬가지로 최근에 나온 연구도 전통적인 SRAM cache와 같은 방법으로 DRAM cache를 설계합니다. tag와 data access의 직렬화, 높은 associativity와 좋은 replacement를 사용하는 방법이 그 예죠. 

우리는 cache 최적화의 성능을 기술 constraint와 parameter에 기반하여 관찰합니다. DRAM cache의 latency와 size parameter가 전통적인 cache와 매우 다르고, 기술 constraint가 이질적임을 본다면 우리는 DRAM cache의 구조에서 최적화에 대해 굉장히 조심해야할 필요성이 있습니다. 예전 cache와 현재 cache 구조가 다르니깐요!

DRAM cache는 예전 cache보다 더 느리기 때문에 이미 높은 hit latency를 악화시키는 최적화는 전체적인 성능을 하락시킵니다. 이게 간단한 개념인 것 같지만, 결과에 아주 치명적입니다. DRAM cache를 위해 예전 cache 최적화를 재점검하는 필요성에 대해 간단한 예제로 설명해보겠습니다. 

cache와 memory 시스템을 생각해봅시다. 1 unit의 latency로 memory access를 하고 cache는 0.1 unit으로 access 합니다. cache hit rate를 0%에서 100%로 증가시키는 것은 평균 latency를 1에서 0.1로 선형적이게 감소시킵니다. 

<p align="center"><img src="../../assets/images/051801.jpg" width="700px" height="700px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 1, cache hit rate & average latency></center>

<그림 1>에서 hit rate가 50%인 부분에서 Base Cache는 0.55의 average latency를 보입니다. Optimization을 진행하면 miss의 40%를 개선하여 hit rate가 70%로 변하는 것을 볼 수 있습니다. 하지만 latency가 1.4배 늘어나게 되죠. 이 논문에서는 최적화를 average latency를 줄이는 방향으로 진행하고 싶어합니다. 

이 Break-Even Hit Rate는 base cache의 hit rate에 달라지기 때문에, 만일 hit rate가 60%라면 BEHR은 100%가 되는 것이죠. 이는 cache hit latency가 DRAM cache에 매우 중요한 역할을 할 때 매우 비효율적일 수 있습니다. 
