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

<p align="center"><img src="../../assets/images/052503.jpg" width="300px" height="300px" title="OP code 예시" alt="OP code" ><img></p>
<center><표 1, LH-Cache De-Optimizing></center>

<표 1>는 speed up, hit-rate, 평균 hit latency를 LH-Cache 다양한 방법으로 실험한 결과입니다. LH-cache의 direct-mapped는 set-associative 보다 더 효과적인 반면에, Tag Serialization Latency와 [Predictor Serialization Latency (6)](#추가설명)에서 여전히 문제가 있습니다. 하지만 본 논문에서 제시하는 모델은 serialization latencies를 제거하고 IDEAL-LO에 비슷한 성능을 가집니다. 실험적 방법으로 저희 Solution을 말씀드리겠습니다.

---

## 실험

### 환경 설정

<p align="center"><img src="../../assets/images/052504.jpg" width="300px" height="300px" title="OP code 예시" alt="OP code" ><img></p>
<center><표 2, 환경설정 Baseline></center>

Pin-based x86 시뮬레이터를 사용하며, <표 2>로 Baseline 환경설정을 하였습니다. L3 cache와 DRAM paramter는 LH-Cache 논문과 같고, LH-Cache와 SRAM-Tag는 [LRU-based DIP replacement (7)](#추가설명)를 사용합니다. 


---

### workload

<p align="center"><img src="../../assets/images/052505.jpg" width="300px" height="300px" title="OP code 예시" alt="OP code" ><img></p>
<center><표 3, Benchmark Characteristics></center>

<표 3>을 보면 workload를 볼 수 있는데, MPKI는 Misses Per 1000 Instructions이고, foot-print는 linesize만큼 특정 line에 배로 곱한 개 수입니다. 

---

## <font color="red"> Latency-Optimized Cache Architecture</font>

<p align="center"><img src="../../assets/images/052506.jpg" width="700px" height="700px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 4, 구조></center>

이제 핵심부분입니다. 구조적으로 어떻게 구현했길래 IDEAL-LO에 가까운 성능을 내는지 확인해봅시다. 

LH-Cache를 29-way에서 direct-mapped 구조로 설정함에 따라 성능이 8%에서 15%로 늘었지만 (<표 1> 참고), latency-optimized에는 아직 해결할 길이 많아 보입니다 (38%). tag lookup 의한 serialization latency로 차이가 발생하는 것이 주요 원인입니다. LH-cache가 이전 전통적인 cache처럼 tag-store와 data-store를 DRAM cache에서 분리한다고 생각합시다. 물리적으로나 분리된 구조이기 때문에 tag-store와 data-store는 이전 cache처럼 만드는 것이 말이 되어 보입니다. tag-store은 latency를 최적화하여 빠른 lookup이 되도록 만들어주고, data-store가 최적화되는 반면에도 여러 port를 가질 수 있습니다. 

**우리는 인접한 tag-store를 분리하는 것이 tag와 data가 같은 DRAM array에 공존할 때에 불필요하다는 중요한 관점을 가져야 합니다.**

---

### Alloy Cache

tag-store와 data-store의 분리를 정확히 하는 것이 TSL overhead를 피하도록 도와줍니다. 이것이 Alloy Cache라 부르는 우리의 주요 cache 구조의 key point입니다. Alloy cache는 tag와 data를 TAD (Tag and Data)라고 불리는 하나의 entity로 합칩니다. Alloy Cache에 access할 때, 하나의 TAD를 제공합니다. 만일 tag가 TAD으로 부터 받은 tag와 일치한다면 cache hit라 생각하고 TAD에 있는 data line을 제공합니다. 만일 tag가 없다면 cache miss라 생각합니다. 그러므로 tag-store과 data-store이라는 각각의 분리된 access를 가지는 것이 아닌, Alloy Cache는 두 access를 하나의 융합된 access로 통합합니다. <그림 4> 처럼요. Cache miss에서, 사용되지 않은 data line을 이동시킬 때 bandwidth가 소모되는 적은 cost가 발생합니다. 이 overhead는 LH-Cache보다 상당히 적은 overhead입니다. 

각 TAD는 directed-mapped된 Alloy Cache의 한 set를 대표합니다. Alloy Cache가 두 개의 power가 없는 set를 가지고 있따면, set를 확인하기 위해 address bit를 단순히 사용할 수 없습니다. 

<그림 4>처럼 구조가 설계되어 있으며, Alloy Cache의 data transfer 크기는 DRAM cache의 물리적 제약에 의해 영향을 많이 받습니다. 5개의 data transfer를 진행하면 80bytes인데 TAD는 72bytes입니다. 때문에 8 bytes는 버려야 하는데 만일 set가 홀수이면 앞의 8bytes를, 짝수면 뒤의 8bytes를 버립니다. 

---

### Impact on Effective Bandwidth

<p align="center"><img src="../../assets/images/052507.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><표 4, Bandwidth 비교></center>

<표 4>를 보면, Alloy Cache는 6.4x배의 bandwidth를 제공하지만 LH-Cache는 1.8x배로 Alloy Cache가 훨씬 더 Bandwidth 부분에서 효과가 좋다는 것을 볼 수 있습니다. 

---

### Latency and Performance Impact

<p align="center"><img src="../../assets/images/052508.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 5, Alloy Cache speed up></center>

<그림 5>를 보면 latency와 성능 부분에서도 긍정적인 영향을 미치는 것을 볼 수 있습니다. 이상적인 SRAM-Tag와 성능이 비슷하게 Alloy Cache는 21%의 성능 향상을 보여주는 것을 볼 수 있습니다. 특히 perfect predictor(100% accuracy, zero-cycle latency)이면 Alloy Cache의 성능은 37%까지 상승하는 것을 볼 수 있습니다. 

---

## Low-Latency Memory Access Prediction

MissMap 방법은 DRAM cache의 정보를 완벽하게 가져오는 것에 집중합니다. 그러므로, line 마다 정보를 계속 tracking할 필요성이 있습니다. 이것이 line당 one-bit 의 storage를 발생시킨다 해도, line이 수백만 개나 된다면, MissMap의 크기는 megabyte단위로 빠르게 접근합니다. MissMap의 크기가 클 때, storage와 L3 cache처럼 [on-chip 구조 (8)](#추가설명)에 저장하는 것을 피하는 것이 좋습니다. 이에 따라 L3 cache access latency(24 cycles)이 발생하게 됩니다. 이에 무시할만한 storage와 delay를 유발하는 정확한 predictor를 개발해야 합니다.

---

### Serial Access vs. Parallel Access

<p align="center"><img src="../../assets/images/052601.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 6, Cache Access 하는 방법: 직렬과 병렬></center>

LH-Cache에서 만든 암묵적 가정은 메모리 접근하기 전에 DRAM cache miss가 있으면 시스템이 이를 보장해야 한다는 것입니다. 이 가정은 전통적인 cache가 어떻게 작동하는지와 유사합니다. 울리는 이를 *Serial Access Model (SAM)*이라 부르며, cache access와 memory access가 직렬적으로 처리됩니다. SAM 모델은 cache miss를 main memory로 보낼 때에 bandwidth가 효율적으로 동작합니다. 

대안적으로, cache와 memory를 병렬적으로 관측하는 bandwidth가 덜 효율적인 모델을 사용할 수 있습니다. 우리는 이런 모델을 *Parallel Access Model (PAM)*이라 부릅니다. PAM은 memory access path에서 cache-miss detection latency를 제거한다는 장점이 있습니다. 

PAM을 적절히 넣기 위해 memory content보다 cache content에 우선권을 줘야만 합니다. tag를 확인한 결과를 cache가 반환하기 전에 memory system이 data를 반환한다면, cache에서 dirty state로 line을 배정하여 data를 사용하기 전에 기다릴 수 있도록 만들어야 합니다. 

DRAM cache miss인 경우 DRAM cache에 access하는 것이 낭비일지 모릅니다. 하지만 LH Cache와 Alloy Cache에서 tag는 DRAM에 위치하고 있습니다. 그렇기 때문에 DRAM cache miss가 나더라도 victim line을 선택하고 victim이 사용되었는지 확인하도록 tag를 읽어야만 합니다. 그래서 PAM은 perfect predictor에 비교했을 때, [cache utilization (9)](#추가설명)에 중대한 영향을 미치지는 않습니다. 

---

### To Wait or Not to Wait

cache에 위치하는지 아닌지를 추정해서, SAM과 PAM을 동적으로 선택하는 것이 최우선 같아보입니다. 이 것을 *Dynamic Access Model (DAM)* 이라고 부릅니다. line이 cache에 있다고 판별되면, DAM은 SAM을 사용하여 memory bandwidth를 절약합니다. line이 cache에 없다면, DAM은 PAM을 사용하여 latency를 줄입니다. DAM은 SAM과 PAM 사이에 완벽한 정보를 요구하지는 않습니다만 좋은 추정치이어야만 합니다. 이 추정을 돕기 위해, 우리는 *Memory Access Predictor (MAP)* 기반의 하드웨어를 공개합니다. latency를 최소화 하기 위해 예측기를 간단한 것으로 고려해야 합니다. 

---

### Memory Access Predictor

PAM의 latency 절감과 SAM의 bandwidth 절감은 cache hit rate에 따라 달라집니다. 이 cache miss와 hit는 이전 결과의 좋은 상관관계를 드러내며, 단순히 hit-rate를 사용하는 것 보다 더 효과적인 예측을 상관관계 결과를 추출함에 따라 가져올 수 있습니다. 이전에 hit이냐 miss냐에 의존해서 예측하는 예측기를 *History-Based Memory-Access Predictors*라 합니다. 

---

#### Global-History Based MAP (MAP-G)

*MAP Global(MAP-G)*는 *Memory Access Counter (MAC)*이라 불리는 counter를 사용하여 memory access 또는 DRAM cache의 hit를 초래하는 최근의 L3 miss를 추적합니다. 만일 L3 miss가 memory access로 이어졌을 대, MAC는 증가하고 아니면 MAC는 감소합니다. 이러한 예측으로 MAP-G는 MAC의 MSB를 사용하여 L3 miss가 SAM (MSB=0) 또는 PAM (MSB=1)을 사용하는 지를 결정합니다. 

---

#### Instruction-Based MAP (MAP-I)

cache access를 유발하는 instruction address로 hit/miss 정보를 잘 아는 observation을 추출하여 MAP-G의 효율성을 향상시킬 수 있습니다. 이 것을 *Instruction-Based MAP (MAP-I)*이라고 합니다. MAC을 사용하는 대신, MAC의 table을 사용하는데 이를 *Memory Access Counter Table (MACT)*라 합니다. 

---

## 성능 결과

<p align="center"><img src="../../assets/images/052602.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 7, 성능 비교></center>

MAP-I방법이 제일 좋고 그 다음 MAP-G 순으로 좋네요. 

<p align="center"><img src="../../assets/images/052603.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><표 5, 정확도></center>

정확도도 MAP-I이 높은 것을 볼 수 있습니다. 

<p align="center"><img src="../../assets/images/052604.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 8></center>

Cache size에 따른 speed up도 Alloy-Cache가 IDEAL-LO에 가까운 것을 볼 수 있네요.

<p align="center"><img src="../../assets/images/052605.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 9></center>

hit-latency도 Alloy-Cache에서 가장 낮은 것을 볼 수 있습니다. 

<p align="center"><img src="../../assets/images/052606.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><표 6></center>

<p align="center"><img src="../../assets/images/052607.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><그림 10></center>

<p align="center"><img src="../../assets/images/052608.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>
<center><표 7></center>

Room 또한 향상된 것을 볼 수 있네요.

## Conclusion

DRAM cache의 구조에 대하여 trade-off를 분석해보았습니다. 최근에 발표된 설계 디자인 LH-cache와 latency 최적화된 이상적인 SRAM-based Tag-Store(SRAM-Tags)와 비교하여 성능을 보았으며 latency를 최적화하는 것은 단순히 hit-rate를 최적화하는 것보다 더 효과적인 DRAM cache를 제공한다는 것 또한 보았습니다. 

정리해서 보면

1. DRAM cache를 높은 associativity에서 direct mapped로 단순히 변경하여 그것 자체만으로도 좋은 성능 향상을 보여줬습니다. 예를 들어, LH-Cache를 29-way에서 1-way로 변경한 것은 8.7%에서 15%의 성능향상을 보여줬습니다. 이는 direct-mapped cache가 buffer hit를 이용하는 능력 뿐만 아니라 latency가 낮기 때문에 이뤄졌습니다. 

2. direct-mapped로 단순히 구조를 바꾸는 것은 충분하지 않습니다. tag-store과 data-store로 나누는 것이 tag-serialization latency를 유발할 수 있기 때문에, Alloy Cache 구조를 제안합니다. data와 tag를 함께 storage entity로 합쳐줍니다. 이에 따라 tag와 data가 하나의 access로 가능해졌고 이는 21%나 되는 Alloy Cache의 성능향상을 볼 수 있었습니다. 

3. Alloy Cache의 성능은 miss를 빠르게 처리를 하므로 향상될 수 있습니다. DRAM cache에 있는 tag를 확인해보기 전에 miss를 memory로 빠르게 보냄에 따라서죠. 그러나 MissMap으로 하는 것은 Megabytes나 되는 storage overhead와 latency가 발생하기 때문에, 성능이 낮아졌습니다. 대신 low-latency, low-storage overhead, 높은 정확도를 가진 hardware-based *Memory Access Predictor*를 사용하여 Alloy Cache를 35%나 성능 향상할 수 있었습니다. 

---

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

**(8) On-Chip Architecture**
- 반도체 칩 내부에서 구성되는 하드웨어 아키텍처를 의미. CPU, 캐시 등의 요소를 포함

**(9) Cache utilization**
- cache의 활용도나 이용률을 나타내는 개념. 

**(10) MissMap**
- cache 동작과 관련된 용어 중 하나. cache의 특정 위치(주소)에 대한 cache miss가 발생하였을 때, 해당 위치를 기록하는 map임. 