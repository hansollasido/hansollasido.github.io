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

---

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

전형적인 cache 최적화(high associativity, better replacement)는 40% 가까이나 되는 높은 miss 감소를 보여주지는 않습니다. 오히려 이전의 DRAM cache 구조에서 1.4배나 더 높은 hit latency overhead를 가져올 수 있습니다. 

---

## Background and Motivation

DRAM cache는 적어도 다음 4개의 goal에 도달할수록 효과적인 설계가 됩니다. 

첫 번째로는 [non-DRAM storage (1)](#추가설명)를 최소화 해야합니다. 
두 번째로는 hit latency를 최소화 해야합니다.
세 번째로는 miss 발생시, 메모리에 빨리 보내야하므로 miss latency를 최소화 해야합니다.
네 번째로는 좋은 hit-rate를 가져야 합니다. 

좋은 설계는 이런 것들을 균형있게 잡아 성능을 최대한으로 끌어낼 수 있습니다. 

main memory의 bandwidth 소비를 최소화하고 cache 용량을 효율적으로 사용하기 위해서 cache line의 [granularity (2)](#추가설명)에 DRAM cache를 조직하는 것이 바람직합니다. 

DRAM cache 설계하는데에 하나의 주요 도전과제가 있다면 tag store입니다. 몇 백 megabytes의 [오버헤드 (3)](#추가설명)가 발생할 수 있기 때문입니다. 

---

## SRAM-Tag Design

<p align="center"><img src="../../assets/images/052501.jpg" width="700px" height="700px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 2, DRAM cache 조직></center>

<그림 2>의 (a)를 보면SRAM 구조가 나눠져 있는 것을 볼 수 있습니다. 허용할 수 없는 매우 큰 overhead(24MB for 256MB DRAM cache)가 있다고 가정합니다. 32-way cache로 DRAM cache를 설정하고 cache의 한 행으로 전체 set를 저장합니다. data를 얻기 위해서 tag-store로 먼저 접근합니다. [serialization of tag (4)](#추가설명)으로 인해 발생하는 latency를 "Tag Serialization Latency (TSL)"이라 부르겠습니다. TSL은 cache hit latency에 직접적인 영향을 미치기 때문에 반드시 최소화해야 합니다. 

---

### Tags-in-DRAM : The LH-Cache

DRAM에 tag를 배치하여 SRAM이 overhead되는 것을 막습니다. 하지만 단순히 이렇게만 하면 DRAM cache가 access를 tag에 한 번, data에 한 번 하기 때문에 두 번의 access에 대한 latency가 발생하여 hit latency가 높아집니다. Loh와 Hill의 최근 연구를 보면 같은 행에 있는 전체 set에 대해 co-locating tag와 data로 DRAM tag의 access penalty를 줄입니다. tag store을 위해 한 행에 3개의 line을 저장하고, 29 line들은 data line으로 사용할 수 있게 만듭니다. 그러므로 29-way cache가 되는 것이죠. cache access는 반드시 먼저 tag를 얻고 그 다음 data line을 얻어야 합니다. 하지만 두 번째 access는 여전히 첫 번째 latency의 반 정도되는 latency가 발생하는데 그래서 이 설계 design은 TSL overhead를 여전히 유발합니다. 

tag check이 full DRAM access를 유발한다고 가정할 때, cache miss에 따른 latency가 상당히 증가합니다. cache miss를 빠르게 서비스하기 위해, 작가는 *Miss Map* 구조를 제안합니다. 만일 MissMap 에서 miss가 발견될 경우, tag check를 기다릴 필요 없이 memory에 직접 접근할 수 있습니다. 불행히도 MissMap 구조는 multi-megabyte의 storage overhead를 발생시킵니다. 효과적으로 MissMap을 적용하기 위해서 작가는 L3 cache에 MissMap을 넣었습니다. MissMap이 L3 miss를 조회하는데 이는 MissMap의 추가적인 latency를 의미하고 우리는 이것을 Predictor Serialization Latency (PSL)이라 합니다. 따라서 TSL과 PSL 둘다 hit latency는 악화됩니다. 본 논문을 통해, Loh와 Hill이 설계했던 것이 항상 MissMap이 설치되었다고 가정하며, 그 Cache를 간단히 *LH-Cache*라고 부르겠습니다. 

---

### 이상적인 Latency 최적화 설계

<p align="center"><img src="../../assets/images/052502.jpg" width="700px" height="700px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 3, Latency></center>

TSL로 인해 SRAM-Tag와 LH-cache에는 hit latency가 있습니다. [conflict miss (5)](#추가설명)를 줄이기 위해, 두 설계는 기존의 set-associative cache와 비슷하게 설계됩니다. 

conflict miss를 감소하기 위해 한 행의 전체 set를 배치하는데, 이럴 경우 cache access에서 row-buffer를 죽이곤 합니다. 게다가 high associativity를 지원하는 LH-Cache같은 경우, tag line의 개수가 너무 많기 때문에 더 높은 latency를 초래합니다. 또한 replacement update와 victim selection 의해 소비한 bandwidth는 hit latency를 더 악화시킵니다. 

DRAM cache는 hit latency를 최소화해야한다고 전에 말했습니다. 이상적으로는 TSL과 PSL을 0으로 만들 수 있습니다. 우리는 이러한 디자인을 IDEAL-LO (Latency-Optimized)라고 부릅니다. <그림 3>의 (c)를 보면 latency overhead가 발생하지 않음을 볼 수 있습니다. 

---

### Performance

<p align="center"><img src="../../assets/images/052503.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><표 1, LH-Cache De-Optimizing></center>

<표 1>는 speed up, hit-rate, 평균 hit latency를 LH-Cache 다양한 방법으로 실험한 결과입니다. LH-cache의 direct-mapped는 set-associative 보다 더 효과적인 반면에, Tag Serialization Latency와 [Predictor Serialization Latency (6)](#추가설명)에서 여전히 문제가 있습니다. 하지만 본 논문에서 제시하는 모델은 serialization latencies를 제거하고 IDEAL-LO에 비슷한 성능을 가집니다. 실험적 방법으로 저희 Solution을 말씀드리겠습니다.

---

## 실험

### 환경 설정

<p align="center"><img src="../../assets/images/052504.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><표 2, 환경설정 Baseline></center>

Pin-based x86 시뮬레이터를 사용하며, <표 2>로 Baseline 환경설정을 하였습니다. L3 cache와 DRAM paramter는 LH-Cache 논문과 같고, LH-Cache와 SRAM-Tag는 [LRU-based DIP replacement (7)](#추가설명)를 사용합니다. 


---

### workload

<p align="center"><img src="../../assets/images/052505.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><표 3, Benchmark Characteristics></center>

<표 3>을 보면 workload를 볼 수 있는데, MPKI는 Misses Per 1000 Instructions이고, foot-print는 linesize만큼 특정 line에 배로 곱한 개 수입니다. 

---

## <font color="red"> Latency-Optimized Cache Architecture</font>

<p align="center"><img src="../../assets/images/052506.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 4, 구조></center>





## 추가설명

**(1) non-DRAM storage**
- DRAM이 아닌 다른 형태의 저장소를 가리킴. DRAM은 Dynamic Random Access Memory의 약자로, 주로 컴퓨터의 주 메모리에 사용되는 휘발성 메모리이지만 non-DRAM은 DRAM과 다른 특성을 가진 비휘발성 메모리 형태를 말함. 데이터를 장기적으로 보관하거나 영구 저장할 목적으로 사용되며, 전원이 꺼져도 데이터를 유지할 수 있음. 

**(2) Cache의 세분성(granularity)**
- cache가 데이터를 얼마나 작은 단위로 저장하고 관리하는지를 나타냄. 

**(3) Overhead**
- 컴퓨터 구조에서 overhead는 어떤 작업을 수행하기 위해 추가적으로 필요한 비용, 시간, 자원 또는 작업으로 정의됨. 일반적으로 컴퓨터 시스템의 성능을 저하시키거나 추가적인 부담을 주는 작업이나 처리 과정을 가리킴.

**(4) Serialization of tag**
- cache system에서 사용되는 용어. tag를 처리하는 과정 중 하나로, 병렬 처리를 위해 동시에 여러 태그를 처리하는 것이 아니라 하나의 태그를 순차적으로 처리하는 것을 의미함. 

**(5) Conflict miss**
- cache memory에서 발생하는 miss의 한 종류임. set 구조에 기인한 현상으로, 동일한 set 내에서 충돌이 발생하여 data를 cache에서 찾지 못하는 상황을 의미함. 

**(6) Predictor Serailization Latency**
- 예측기 (Predictor)의 직렬화 지연을 의미. 예측기는 병렬 처리와 성능 향상을 위해 여러 예측 작업을 동시에 처리함.

**(7) LRU-based DIP replacement**
- 메모리에서 데이터를 교체하는 정책 중 하나임. Least Recently Used 약자 LRU와 Data-in-Place의 약자 DIP로 가장 오래 사용되지 않은 데이터를 교체하면서 교체된 데이터를 다른 위치에 이동시키지 않고 그 자리에 새로운 데이터를 저장하는 방식을 말함. 