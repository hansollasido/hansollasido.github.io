---
title: "DMPS"
excerpt: "Memory Access Scheduling Based on Dynamic Multilevel Priority in Shared DRAM Systems"

categories:
  - DRAM
tags:
  - [반도체, DRAM]

permalink: /categories9/review22/

use_math: true
toc: true
toc_sticky: true

date: 2023-10-20
last_modified_at: 2023-10-20
---

(10/20 작성 중)

### Abstract

Shared main memory에서 Interapplication interference은 심각하게 성능을 저하하고, DRAM frequency가 증가하면 simple memory scheduler가 필요하게 됩니다. 

이전 memory scheduler는 높은 시스템 성능을 위해 각 애플리케이션별 순위 매기기 체계를 사용하거나, 낮은 하드웨어 비용을 위해 그룹별 순위 매기기 체계를 사용했습니다. 하지만 둘 사이의 균형을 제공하는 것은 많지 않습니다. 

이에 본 논문은 DMPS라는 dynamic multilevel priority에 기반한 memory scheduler를 제안합니다. 먼저 DMPS는 memory occupancy(메모리 점유율)을 사용하여 interference를 양적으로 측정합니다. 두 번째로, DMPS는 application을 그룹화하고, latency-sensitive groups을 우선시하며, 단계별 순위 매기기 체계를 사용하여 application들의 우선순위를 동적으로 정합니다. 

simulation 결과는 DMPS가 7.2% 더 좋은 성능을 보여주며, low HW complexity와 cost에 대해서 FRFCFS보다 22%보다 더 나은 [(1) 공정성](#추가설명)을 제공합니다. 

---

### Introduction

멀티 코어 시스템에서 main memory는 전형적으로 high access latency와 제한된 memory bandwidth에 의해 critical한 shared resource입니다. 동시에 실행되는 다양한 applications들은 제한된 memory bandwidth에 대해 심하게 경쟁하며 서로에게 해로운 간섭을 일으킵니다. Application에서 온 Request은 지연되고 심지어는 추가적인 row-buffer 충돌, bank 충돌, address/data bus 충돌을 일으켜 다른 application을 starving하고, application 내의 bank-level 병렬성을 파괴할 수 있습니다. Interapplication interference는 전체적인 성능과 공정성을 훼손할 수 있으며, 특히 chip에 있는 코어의 개수를 늘고, application의 data 강도가 늘어나는 트렌드에서는 더더욱 그렇습니다. 

memory interference를 완화하기 위해서는, smart resources, dumb resources, 그리고 integrating smart and dumb resources는 주요한 방법일 수 있습니다. [Mutlu 2013; Mutlu et al. 2015]. smart resourecs 방법은 memory controllers, interconnects, caches가 interapplication interference를 발견하고 불공정한 application slowdown과 성능 저하를 방지하도록 만듭니다. 반면 dumb resource 방법은 resource allocation을 cores/sources 또한 system software에서 제어하여 불공정한 slowdown과 성능 저하를 완화시킵니다. (인용된 논문이 많음) smart와 dumb resource를 합하는 방법은 interference mitigation을 효율적으로 만들 수 있는데, 이 방법들은 직교(orthogonal)하기 때문에 가능합니다. 여기서 직교라는 것은 두 접근법이 서로 독립적이라는 뜻입니다. 

Application-aware memory access scheduling은 application간 interference를 완화하기 위한 대표적 방법으로, smart memory controller 설계에 집중을 둡니다. 전형적으로 application-aware memory scheduler는 세 part로 나눠져 있습니다. (1) application의 memory access behavior 관측 : interference에 취약성을 대신하는 수단으로 사용. (2) memory access behavior에 기반한 application 순위 매겨서 interference에 취약한 application들이 더 높은 priority를 가지도록 함. (3) 제일 높은 priority를 가진 ready command를 선택하여 실행. 

memory scheduler의 HW 복잡도와 비용은 application을 순위 매기는 세부 정도와 memory access behavior를 모니터하는 복잡성에 의해 결정됩니다. 이전에 제안된 application-aware memory sheduler는 시스템 성능과 공정성 아니면 HW 복잡성과 비용 중 하나로 편향되어 있습니다. 가장 큰 차이점은 application을 우선순위화 하는데 사용되는 ranking mechanism입니다. <fig 1>처럼요. 

<p align="center"><img src="../../assets/images/102301.png" width="500px" height="500px" title="DMPS" alt="DMPS" ><img></p>

Extreme에서 system 성능은 shortest-job-first라는 원칙을 적용하여 얻을 수 있습니다. **system 성능을 최대화하기 위하여, <Fig 1(a)>에 보여진 scheduler처럼 application ranking 메커니즘과를 사용하고, [(2) memory intensity](#추가설명)로 application을 분류해서 lower memory intensity인 application이 강력하게 우선시 되도록 합니다.** Memory intensity는 다른 scheduler에서는 다른 의미를 가집니다. parallelsim-aware batch schdeuling (PARBS) [Mutlu and Moscibroda 2008]에서 max-total bank load, adaptive per-thread least-attained service memory scheduling (ATLAS) [Kim et al. 2010a]에서의 service, 그리고 thread cluster memory scheduling (TCM) [Kim et al. 2010b]에서의 misses per kilo instructions (MPKI)처럼요. 

우선 순위 규칙, 예를 들면 PARBS의 'marked requests first', ATLAS의 'age-over-threshold requests first', 또는 TCM의 'bandwidth-sensitive한 application간의 삽입 또는 무작위 섞기'와 같은 방식을 적용함으로써 공정성을 보장할 수 있습니다. **이러한 scheduler는 높은 시스템 성능과 공정성을 얻을 수 있지만, HW 복잡성과 비용이 매우 커질 수 있습니다.** 이전 연구에서 [Subramanian et al. 2014] PARBS와 TCM의 24 코어 시스템에서의 critical path latency가 DDR3-1333과 DDR4-3200에서의 minimum scheduling time $t_{CCD}$를 넘는 다고 밝혔습니다. 코어의 개수와 DRAM frequency가 꾸준히 증가하면서 이런 상황은 더 악화되고 있습니다. **그러므로, memory scheduler를 가볍게 유지하여 빠른 scheduling 행동을 할 수 있게 하는 것이 설계에 있어 중요한 고려 사항이 됩니다.** 

반대편에서, <Fig 1(b)>처럼 scheduler는 per-group ranking 메커니즘을 사용하여 HW 복잡성을 낮게 유지하고, 단순한 비교 작업을 통해 application들을 대략적으로 그룹으로 분류합니다. (i.e., nonbalcklisted group blacklisted groups in the blacklisted memory scheduler (BLISS)[ Subramanian et al. 2014, 2016]). 공정성을 확보하기 위하여 BLISS는 nonblacklisted applications을 우선순위화하고 같은 그룹에 있는 application에 동등한 우선순위를 줍니다. HW 복잡도와 비용은 매우 작고, BLISS의 critical path latency은 DDR4-3200에서 $t_{CCD}$보다 작습니다. 하지만, BLISS는 처음에 모든 application에 동등한 우선순위를 주고, 충분한 interference를 일으킨 후에 interference를 일으키는 application을 blacklist에 올립니다. BLISS는 interference를 일으키는 능력을 측정하기 위하여 threshold를 사용하고, memory access behavior에서 application의 다양성을 찾아내지 못합니다. 그러므로, BLISS는 적은 HW 복잡도와 비용을 추구하기 위하여 높은 system 성능을 희생해야합니다. 

**이번 연구의 목표는 높은 system 성능과 공정성을 얻기 위하여 간단한 memory scheduler를 설계하는 것입니다.** 본 논문은 DMPS라는 새로운 동적인 multilevel priority에 기반한 memory scheduler를 제안합니다. DMPS의 주요 아이디어는 interference을 일으킨 것에 응해 여러 level로 application을 동적으로 우선순위화한다는 것에 있습니다. DMPS의 설계는 세 가지 관점에 기반합니다. 

첫 번째로, 본 논문은 memory 점유율을 사용하여 memory interference를 발생시킬 능력을 정량적으로 측정합니다. 이는 어떤 application에서 제공되는 read request 수와 각 application에 대해 제공되는 평균 read request 수의 비율로 정의됩니다. memory interference을 측정하기 위한 대리 척도로서, memory 점유율은 제공된 read request의 수를 저장하기 위해 몇 개의 counter를 단순히 추가함으로써 HW에서 구현될 수 있습니다. memory 점유율은 memory 강도와 row-buffer 지역성을 둘 다의 interference를 고려합니다. 높은 memory 점유율을 가진 application은 전형적으로 높은 memory 강도와 높은 row=buffer 지역성을 가지고 있습니다. 이는 memory bandwidth를 더 가진다는 뜻이자, interference를 크게 발생시키죠. 

두 번째로, 이전 연구 [Multu and Moscibroda 2008; Kim et al. 2010a, 2010b; Zheng et al. 2008]에서 latency에 민감한 application을 bandwidth에 민감한 application보다 우선순위화하는 것이 system 성능을 아주 높게 향상시킨다는 것을 설명하고 있습니다. latency에 민감한 application들은 좀 처럼 memory request를 발생하지 않기 때문에, 우선 순위된다면 더 적은 memory 점유율과 다른 application에 대하여 무시가능한 interference가 됩니다. DMPS에서는 latency와 bandwidth에 민감한 group을 memory 점유율에 기반으로 분류합니다. 시스템 성능을 최대화하기 위해 latency에 민감한 application을 bandwidth에 민감한 application보다 먼저 우선순위화 합니다. 

세 번째로, 이전 연구 [Kim et al. 2010a, 2010b]에서 bandwidth에 민감한 application 간의 고정적 우선순위로 부터 오는 불공정성 문제를 보여주고 있습니다. 그러므로 DMPS는 공정성을 보장하기 위하여 application에 동적 우선순위를 사용합니다. 같은 group에 있는 application은 먼저 동등한 우선순위를 부여합니다. 어떤 application의 memory 점유율이 미리 정해진 threshold을 초과하면 해당 application은 너무 많은 memory bandwidth를 획득한 것으로 간주되며, 다른 application에게 memory bandwidth의 공정한 분배를 보장하기 위해 그 application의 우선순위를 한 단계 낮춥니다. 그러므로 같은 group에 있는 application들도 다르게 취급됩니다. 비록 DMPS는 application을 group으로 대략적으로 분류하지만, 세밀한 multilevel 우선순위로 인해 DMPS는 memory access behavior에서 application의 다양성에 민감하게 반응합니다. multilevel 우선순위는 HW 복잡도를 굉장히 줄이며, DMPS는 더 많은 core와 빠른 DRAM system에서 적용될 수 있습니다. 

정리하자면 아래와 같습니다.

- 본 논문의 아이디어는 다양한 우선순위 level로 application을 분류하여 높은 시스템 성능과 공정성을 얻는 가운데, 적은 HW 복잡도와 비용을 가집니다. MPKC에 기반한 메모리 점유율 metric을 사용하여 application의 memory interference를 양적으로 측정하는 능력을 측정합니다. Memory 점유율에 기반하여, application은 latency와 bandwidth에 민감한 group으로 나누고 latencey에 민감한 application을 먼저 높은 우선순위를 줍니다. 미리 정한 threshold와 request의 수를 비교해서 application을 동적으로 다양한 level에 우선순위화 합니다. 

- DMPS를 24 코어와 4 channel system에서 80 worklads에 대한 시스템 성능과 공정성을 이전에 발표된 다섯 개의 memory scheduling 알고리즘과 비교해보았습니다. DMPS가 7.2% 더 높은 시스템 성능을 보여줬으며 PARBS와 비슷하게 FRFCFS보다 22%나 높은 공정성을 보여줬습니다. 

- RTL에서 DMPS와 이전 5개의 scheduler를 넣어 적용해보고 critical path latency와 are를 비교함으로써 HW 복잡도를 비교해보았습니다. DMPS의 HW 복잡도는 TCM과 PARBS보다 더 적으며, BLISS와 FRFCFS-Cap보다는 약간 높은 것을 확인하였습니다.

<p align="center"><img src="../../assets/images/102302.png" width="500px" height="500px" title="DMPS" alt="DMPS" ><img></p>

---

### Background

#### DRAM-Based Memory System

<fig 2(a)>에서 channel, rank, bank로 계층적으로 조직된 멀티코어 시스템에서의 전형적인 DRAM-based memory 시스템인 것을 보여줍니다. 각 channel은 독립적인 address와 command 그리고 data bus를 갖고 있으며 최고의 병렬성을 달성할 수 있습니다. 각 channel은 여러 rank로 구성되어 있고, 각 rank는 동시에 작동하는 여러 bank로 구성되어 있습니다. Rank와 bank는 병렬적으로 동작할 수 있지만, channel 내의 모든 rank와 bank가 주소, 명령 및 데이터 bus를 공유하기 때문에 병렬성의 정도는 낮습니다. bank는 2차원 data 배열로 되어 있으며, row와 column을 가집니다. 각 bank에 있는 내부 구조로 row buffer가 있는데, data 배열과 memory controller 사이에서 interface의 역할을 하고 있습니다. row buffer의 상태에 의거하여 memory access가 세 가지 분류로 나뉩니다. : row buffer에서 현재 open되어 있는 row에 access하는 경우를 **row hit**, closed된 row buffer에 access하는 경우를 **row closed**, row buffer에 현재 open되어 있는 row의 다른 row에 접근하는 경우를 **row conflict**라고 부릅니다. <fig 2(b)>는 세 가지 분류에서의 access latency를 보여줍니다. row hit, closed, conflict의 latency는 $t_{CAS}$, $t_{RCD}+t_{CAS}$ 그리고 $t_{RP}+t_{RCD}+t_{CAS}$ 입니다. 보통 row hit는 row closed보다 대략 두 배 빠르게 처리되며, row conflict보다 거의 세 배 빠르게 처리됩니다. 따라서 start-of-the-art memory scheduler들은 DRAM throughput을 최대화하기 위하여 다른 요청들보다 row-hit request를 우선시합니다. 

---

#### [(3)Application-Unaware Memory Schedulers](#추가설명)

FRFCFS는 흔히 쓰이는 memory scheduling 알고리즘입니다. 다른 requests보다 row-hit requests로부터의 ready command를 우선시하고, 만일 row-hit 상태가 동일하면 더 오래된 request를 다른 젊은 request보다 먼저하게 합니다. FRFCFS의 목적은 memory access의 service latency의 평균을 최소화하고 DRAM throughput을 최대화하는 것입니다. FRFCFS는 단일 쓰레드 시스템에서 가장 평균적인 성능이 좋은 것을 보여줍니다. [Rixner et al. 2000] 그러나, FRFCFS는 멀티코어 system에서 application을 고려하지 않는 scheduling policy로 인해 시스템 성능과 공정성이 떨어집니다. FRFCFS는 높은 row-buffer 지역성과 높은 memory 강도를 가진 application의 request를 우대하며, 다른 application의 request를 지연시키거나 심지어 starving하기도 합니다. 

연속적인 많은 row hit가 동일한 bank에 대한 다른 application의 row-closed 또는 row-conflict request를 starving할 수 있기 때문에, FRFCFS-Cap은 동일한 bank에 대한 더 오래된 row access보다 더 어린 column access의 수를 모니터링하기 위해 bank counter를 사용합니다. counter가 Cap parameter 아래인 bank들에 대해서는 FRFCFS 정책이 적용되며, counter가 Cap parameter를 초과하는 bank에 대해서는 공정성을 위해 FCFS 정책이 적용되고 counter는 0으로 재설정됩니다. 그러므로 FRFCFS-Cap은 row-buffer 지역성이 높은 application으로 부터 온 request의 문제를 완화시킵니다. 하지만 FRFCFS-Cap은 memory를 많이 사용하지 않는 application의 request를 처벌하는 FCFS의 본질적인 문제를 해결할 수 없습니다. 



---

#### [(4) Application-Aware Memory Schedulers](#추가설명)

멀티 코어 환경에서, FRFCFS기반의 application-aware memory scheduler는 두 방향으로 발전되었습니다. : 메모리 interference의 영향을 정확하게 측정하고 제어하여 예측 가능한 성능을 제공하기 위해, 예를 들면 stall-time fair memory scheduler(STFM) [Mutlu and Moscibroda 2007], fairness via source throttle (FST) [Ebrahimi et al. 2010], per-thread cycle accounting [Eyerman and Eeckhout 2009], memory interference–induced slowdown estimation (MISE) [Subramanian et al. 2013], 그리고 the application slowdown model (ASM) [Subramanian et al. 2015]와 같은 방법을 사용합니다. HW accelerators (HWAs)를 달고 있는 heterogeneous 환경에서 DASH [Usui et al. 2016]은 HWA의 deadline을 만족하고, 높은 CPU 성능을 제공하기 위해 설계되었습니다. COTS 기반의 multicore 환경에서 Kim 등은 request와 job=driven 기법을 결합하여 가장 최악의 경우에서의 memory interference 경계를 이뤘습니다. Kim 등은 동일한 코어에서 전용 DRAM bank와 함께 memory 집약적인 작업을 함께 배치하여 작업 간의 memory interference를 줄이고 고성능 및 공정성을 위해 interference을 완화하는 memory interference 지향적인 작업 할당 알고리즘을 개발하였습니다. 예를 들면, daptive history–based memory scheduler (AHB) [Hur and Lin 2004], the RL-based memory scheduler (RLMS) [Ipek et al. 2008], the fair thread-aware memory scheduler (FTAM) [Zhu et al. 2010], the request density–aware fair memory scheduler (RDAF) [Ikeda et al. 2012], the multiobjective reconfigurable self-optimizing memory scheduler (MORSE) [Mukundan and Mart´ınez 2012], PARBS, TCM, ATLAS, BLISS 등이 있습니다. 

---

##### Application-Based Marking

PARBS은 memory request를 batch로 그룹화하고 오래된 batch를 최근 것보다 먼저 진행하여, application전반의 공정성과 request에 starvation freedom을 제공합니다. PARBS는 bank 부하를 선택하여 application의 memory access behavior를 특성화하고, max-total scheme을 사용하여 application간의 순위를 형성합니다. 시스템 성능을 최대화하기 위하여, 주로 memory 집중적이지 않은 높은 순위 application 및 row-hit request이 우선 순위로 설정됩니다. batch 내에서의 순위 기반 우선 순위 설정은 application 내 bank-level parallelism을 유지합니다. 그러나, PARBS는 여러 memory controller 간의 application 내 bank-level parallelism을 유지할 수 없습니다. appliaction의 순위는 batch 시작에서 계산되며 batch의 미세한 정밀도로 인해 PARBS는 중요한 조정이 필요합니다. 

ATLAS는 여러 memory controller에서 얻은 서비스를 선택하여 application의 memory access behavior을 특성화하고 long-term behavior를 고려합니다. ATLAS의 목표는 높은 시스템 성능과 scalability를 얻어, core와 memory controller의 개수를 늘리는 것입니다. Application은 엄격하게 우선 순위가 매겨지며 가장 낮은 서비스를 받은 application이 가장 높은 우선 순위를 받습니다. 그러나 시스템 성능의 증가는 공정성을 희생하는 대가로 옵니다. memory 사용이 적은 application이 엄격하게 우선하여 memory 집중적인 application은 높은 latency를 겪게 됩니다. 

TCM은 시스템 성능과 공정성의 목적을 분리하며, application을 두 개의 집단으로 분리하여 각 집단에서 다른 scheduling policy를 적용하여 두 가지 목적을 모두 최적화합니다. 높은 시스템 성능을 얻기 위하여, TCM은 bandwidth 민감한 application보다 latency 민감한 application에 우선순위를 두며, latency 감지 cluster에서는 가장 memory 집중적이지 않은 application이 가장 높은 우선순위를 받도록 엄격한 우선순위를 시행합니다. TCM은 *niceness*라는 새로운 metric을 도입하여 application의 memory access behavior를 특정화시켰습니다. row-buffer 지역성이 높은 application은 다른 application에 대해 덜 협조적이며, 반면 bank-level parallelism이 높은 application은 협조적입니다. 그래서 TCM은 application의 row-buffer 지역성과 bank-level parallelism을 모니터링해야 합니다. 

BLISS는 단순히 application에서 제공된 연속적인 request 수를 세어 application을 두 개의 그룹으로 동적으로 분리하기 때문에 HW 복잡도와 비용이 낮습니다. memory interference를 완화하기 위하여, BLISS는 interference에 취약한 group을 interference를 일으키는 group보다 우선시합니다. 모든 application은 처음에, interference 취약한 group에 있게 됩니다. 한 application이 interference를 심하게 일으켰을 때만 blacklist로 지정되며, interference에 취약한 application에 벌점을 부과합니다. BLISS는 interference을 측정하기 위해 단일 threshold만 사용하므로, memory access behavior의 다양성을 감지할 수 없습니다. 따라서 BLISS는 system 성능이 낮습니다. 

FATM과 RDAF는 memory interference를 측정하기 위하여 MPKC라는 metric을 사용합니다. FATM은 다른 thread로 온 memory request를 다른 queue로 유지하고, 각 queue의 첫 번째 request 중에서 가장 높은 우선순위의 request를 scheduling하여, 동일한 thread에서의 request이 순서를 어길 수 없도록 합니다. FATM은 thread의 우선순위를 계산하기 위하여 thread, arriving time, serving history를 사용합니다. thread $i$의 우선순위 계산식은 다음과 같습니다.

<center>

$P_i=\alpha\times WT_i+\beta\times AST_i+\gamma$, 

<p>

</p>

where $\alpha, \beta, \gamma$ are weights, $WT_i$ is the waiting time of thread $i$, and $AST_i$ is the accumulated serving times of thread $i$

</center>

thread는 언제든지 총 순서로 순위가 매겨집니다. 높은 AST를 가진 thread의 request는 오랜 시간 동안 차단될 수 있습니다. FATM에서 WT는 32bit이고 AST는 16bit이므로 thread 우선 순위의 값이 크게 될 수 있어 복잡도가 높아질 수 있습니다. 반면, DMPS는 memory 점유에 기반하여 thread를 동적으로 다중 level로 우선순위 지정하므로 복잡도가 낮아집니다. 

FDAF는 original TCM의 MPKI를 MPKC로 대체합니다. thread clustering을 위해 latency에 민감한 thread는 마지막 quantum에서 제공된 read request 수를 기준으로 정렬하여 결정됩니다. DMPS는 또한 마지막 quantum에서 thread의 MPKC를 사용하지만, latency에 민감한 thread는 memory 점유를 parameter MOPL과 비교하여 결정하므로 복잡한 정렬 작업을 피합니다. 세세한 우선순위 할당을 위해 DMPS는 여전히 MPKC를 사용하지만, RDAF는 read queue에서 대기 중인 read request 수인 read density를 사용합니다. RDAF의 단점은 TCM과 유사합니다 : 모든 thread가 총 순서로 순위가 매겨집니다. 높은 MPKC를 가진 thread가 낮은 MPKC를 가진 thread에 의해 오랜 시간 차단될 수 있습니다.

---

##### Request-Based Marking

Ghose 등 [2013]의 치명성 인식 scheduler는 ROB (Reorder Buffer) head를 차단하는 부하가 치명적이라는 기준을 기반으로 합니다. processor 측의 commit block predictor (CBP)는 이전에 ROB를 차단한 부하를 추적하고 부하가 발생할 때 부하의 치명성을 예측합니다. 다섯 가지 CBP table이 평가되었습니다 : Binary, BlockCount, LastStallTime, MaxStallTime 그리고 TotalStallTime. Binary Predictor는 각 memory controller의 request entry 당 추가 bit를 필요로 하며, 다른 predictor들은 request entry당 적어도 14bit가 필요하며 가장 치명성이 높은 request를 scheduling하기 위한 복잡한 논리를 필요로 합니다. Ghose 등 [2013]는 치명성을 기반으로 한 predictor가 낮은 경쟁 상태에서 잘 수행되는 것을 보여주었으며, 한편 DMPS는 TCM 및 PARBS와 같이 thread heterogeneity와 높은 경쟁 상태를 대상으로 설계되었습니다. 따라서 치명성 인식 predictor은 DMPS와 별개입니다. 또한 DMPS는 치명성 인식 scheduler의 starving problem of request를 해결할 수 있습니다. 

Hur와 Lin [2004]의 AHB는 두 가지 기본적인 history-based arbiter를 도입합니다. 하나는 read와 write의 비율을 맞추는 arbiter로, reorder queue 내에서 병목 현상을 피하는 경향이 있으며, 다른 하나는 schedule된 operation의 예상 latency을 최소화하기 위한 것입니다. 두 목표는 확률적인 방식으로 두 arbiter 사이를 주기적으로 전환함으로써 부분적으로 달성될 수 있습니다. AHB는 다른 명령 pattern을 최적화하기 위한 여러 history-based arbiter를 구현하여 알 수 없는 command pattern과 일치시킬 수 있으며, 주기적으로 가장 적합한 arbiter를 선택할 수 있습니다. AHB는 간단하며 단일 thread system의 DDR2 DRAM 처리량을 향상시킬 수 있지만, multithread system에서는 memory interference 문제를 해결할 수 없습니다. 높은 속도의 DDR3/4 환경에서 AHB의 명령 pattern arbiter는 write drain policy 때문에 효과적이지 않을 수 있습니다. 

RLMS [Ipek et al. 2008]와 MORSE [Mukundan과 Martínez 2012]는 강화 학습의 Q-learning 알고리즘을 적용하여 scheduling policy를 최적화하기 위해 학습하고 장기적인 성능을 극대화하기 위해 실시간으로 조정할 수 있습니다. 그러나 대규모 Q-값의 존재로 인해 매우 높은 HW overhead를 가집니다. 

---

### Mechanism

#### Simple Memory Occupancy to Measure Interference

본 논문은 memory interference를 유발하는 능력을 측정하기 위하여 memory occupancy를 사용했습니다. memory occupancy에 관련된 수학적 정의는 아래 식 (1)에 있습니다. 분자는 마지막 양자에서 application에 제공된 read request의 수이며, 분모는 마지막 양자에서 각 application당 제공된 평균 read request의 수입니다. 그러므로, memory occupancy는 application 자체와 동시에 실행 중인 다른 application에 의해 결정됩니다. 

<center>

$MO_i = \frac{ReqCnt_i}{\sum^{N_{core}}_{j=1}ReqCnt_j/N_{core}}=\frac{N_{core}\times MPKC_i}{\sum^{N_{core}}_{j=1}MPKC_j}$

<p>

</p>

, where $MO_i$ is the memory occupancy of application $i$, $ReqCnt_i$ is the number of read requests served from application $i$ in a quantum, $N_{core}$ is the number of currently executing applications, and $MPKC_i$ is the misses per kilo cycles of application $i$ in a quantum.
</center>

memory occupancy는 반드시 MPKC를 기반으로 합니다. interference에 취약성의 대리자로서, MPKC는 HW에서 구현하기 간단하며 다른 application과 함께 실행될 때 memory bandwidth를 점유하는 능력을 직접 반영합니다. MPKC는 MPKI, row-buffer 지역성, bank-level parallelism, 또는 다른 processor 측 behavior의 결과입니다. 반면 MPKI는 application의 memory intensity만을 나타냅니다. MPKI를 사용하는 scheduler에서 row-buffer 지역성처럼 다른 metric도 사용될 것 입니다. interference를 유발하는 application의 능력을 결정하기 위하여, 이 metric의 함수는 조심히 설계되어야 합니다. FATM에서는 $P_i=\alpha\times WT_i + \beta \times AST_i + \gamma$입니다. TCM에서 bandwidth 사용은 thread clustering에서 사용되고 MPKI, row-buffer 지역성, bank-level parallelism은 application의 우선순위를 결정하기 위해 분류됩니다. BLISS는 application에서 제공된 연속 read request의 최대 수를 모니터링하고 최대 수가 blacklist threshold 값을 초과할 때 application을 blacklist로 지정합니다. 따라서 interference을 일으키는 application이 blacklist로 지정되기 전에 제공된 read request의 수를 제어할 수 없습니다. 그러나, MPKC는 application에서 제공된 read request의 총 수를 사용하며 이러한 문제가 없습니다. 따라서 MPKC는 bandwidth 사용을 제어할 수 있으며, 복잡성을 낮추려는 scheduler에게 MPKI보다 적합합니다. 

bandwidth 사용 관점에서 보면, 높은 MPKC를 가진 application은 더 많은 interference를 유발하는 경향이 있습니다. memory interference를 완화하기 위하여, DMPS는 동적으로 application을 여러 priority level로 할당합니다. *ReqPL*은 epoch애서 level 당 평균 MPKC로, quantum 내의 모든 application의 long-term MPKC QReqCnt와 구성 가능한 parameter MOPL에 의해 결정됩니다. *EReqCnt*는 한 epoch에서 short-term MPKC를 의미합니다. application의 priority level은 EReqCnt를 미리 정해진 ReqPL의 배수인 threshold과 비교하여 결정됩니다. 

---

#### Coarse-Grain Synchronization to Group Applications

Algorithm 1은 상세하기 동기화의 과정을 보여줍니다. 동기화는 모든 quantum마다 수행되며, $QReqCnt_{i,j}$의 누적, application의 group, 그리고 level당 threshold $ReqPL$의 update를 포함합니다. 첫 번째로, $QReqCnt_{i,j}$, channel $j$로 부터 온 application $i$의 read request의 횟 수는 각 quantum의 끝에서 다양한 memory controller에서 부터 metacontroller에까지 전달됩니다. $ReqCnt_i$는 $QreqCnt_{i,j}$의 모든 channel 전반에 걸친 합이며 $TotalReqCnt$는 $ReqCnt_i$의 application 전반에 걸친 합입니다. 

두 번째로, Algorithm 1에서 $MOPL$는 priority level 당 memory occupancy 값이며 Algorithm 2에서 $PLNUM$는 priority level의 수 입니다. $MOPL$과 $PLNUM$은 사용자에 따라 configured될 수 있습니다. application을 다양한 level로 효과적으로 분리하기 위해서 $MOPL$과 $PLNUM$의 곱은 대략 1입니다. DMPS에서는 memory occupancy가 $MOPL$보다 큰 application은 bandwidth에 민감한 것으로 취급되며, 다른 application은 latency 민감한 것으로 취급됩니다. memory occupancy를 나눗셈으로 계산하는 것은 복잡하고 비용이 많이 들기 때문에 $MOPL$ 대신 read request 수를 기준으로 application을 분류하기 위해 $GroupTh$를 사용합니다. $GroupTh$는 서비스된 read request 수를 기준으로 한 해당 값입니다. 현재 quantum에서 $ReqCnt$가 $GroupTh$보다 큰 application은 bandwidth 민감한 것으로 표시되고, 그 외의 application은 latency 민감한 것으로 표시됩니다. 다음 quantum에서 application의 group 상태인 $NxtGroup$은 이전 두 가지 분류인 $PreGroup$과 $CurGroup$에 의해 예측됩니다. application은 과거 두 개의 quantum에서 bandwidth 민감하게 분류되었다면 bandwidth 만감하게 예측됩니다. 그렇지 않으면 application은 latency 민감하게 예측됩니다. 과거 두 가지 분류를 고려하는 이 two-bit predictor은 최근 분류를 직접 사용하는 것과 비교하여 더 안정적이고 정확하며, latency 민감한 application에게 latency 민감 group에 머무를 기회를 더 많이 제공합니다. 

세 번째로, $ReqPL$은 해당 quantum에서 $MOPL$과 $TotalReqCnt$로 인해 결정됩니다. $ReqPL$은 다음 quantum에서 application의 동적 priority를 update하기 위한 level당 threshold로 사용됩니다. 현재 quantum에서 많은 read request가 제공되면 memory interference가 심하며, $ReqPL$의 값이 높아집니다. 높은 $ReqPL$은 더 낮은 MPKC를 가진 application에게 높은 priority에 머무를 기회를 더 많이 제공하여 memory interference를 완화합니다. 마지막으로 metacontroller는 application의 예측된 group 상태인 $NxtGroup$과 level 당 threshold인 $ReqPL$을 각 memory controller에 보냅니다. DMPS에서 quantum의 길이는 ATLAS와 TCM과 마찬가지로 백만 cycle로 설정되며, 이는 application의 memory access behavior의 변화를 감지하기에 충분히 짧고, metacontroller와 여러 memory controller간의 통신 overhead를 최소화하기에 충분히 길게 설정됩니다. 

<p align="center"><img src="../../assets/images/102601.png" width="700px" height="500px" title="DMPS" alt="DMPS" ><img></p>

<p align="center"><img src="../../assets/images/102602.png" width="700px" height="700px" title="DMPS" alt="DMPS" ><img></p>

---

#### Fine-Grain Dynamic Multilevel Priority 

<fig 3(a)>는 DMPS의 간단한 구조를 보여줍니다. memory occupancy monitor에서 각 application은 제공된 read request 수를 저장하기 위해 두 개의 counter가 필요합니다 : quantum을 위한 long-time counter인 $QReqCnt$, 그리고 epoch를 위한 short-time counter $EReqCnt$입니다. long-time counter인 $QreqCnt$는 매 quantum의 끝에서 metacontroller로 동기화를 위해 전송되고 short-time counter인 $EReqCnt$는 매 사이클마다 multi-level comparator로 전송됩니다. multilevel comparator은 multiple priority level에 제공된 read request의 수로부터 mapping function으로 수행됩니다. <fig 3(b)>는 세 가지 priority level을 가진 DMPS의 mapping function을 보여주며, 더 높은 priority level은 더 높은 priority에 해당합니다. epoch에서 application의 priority level을 결정하려면 short-time counter인 $EReqCnt$를 $ReqPL$ 및 $2\times ReqPL$과 비교해야 합니다. 여기서 $ReqPL$은 priority level 당 제공된 read request 수입니다. $EReqCnt$가 $ReqPL$보다 작을 때, application은 초기 priority level에 있습니다. $EReqCnt$가 $2\times ReqPL$보다 작지 않을 때, application의 priority level은 1입니다. 그 외의 상황에서는 application의 priority level은 2입니다. Algorithm 2는 세부적인 과정을 보여주고 세밀한 동적 multilevel priority process를 보여줍니다. 

<p align="center"><img src="../../assets/images/102603.png" width="700px" height="700px" title="DMPS" alt="DMPS" ><img></p>

mapping function은 두 특징을 가집니다. 첫 번째로, 높은 system 성능을 제공하기 위해 낮은 MPKC를 가진 대부분의 application은 높은 MPKC를 가진 application보다 더 우선순위가 높습니다. MPKC가 낮은 application은 memory request을 거의 생성하지 않으므로 빠른 진행이 가능한 높은 잠재력을 가질 가능성이 있습니다. quantum에서 먼저 bandwidth 민감한 group의 application보다 latency 민감 group의 application에게 priority가 주어집니다. epoch에서 작은 $EReqCnt$를 가진 application은 일시적으로 latency 민감하게 처리될 수 있으며, 그 중 대부분은 동일한 group 내의 다른 application보다 priority가 부여됩니다. 두 번째로, 동일한 group 내의 application은 처음에는 동일한 priority level을 가집니다. application에서 제공된 read request 수가 미리 정해진 threshold보다 초과하면, application의 priority level이 memory interference 완화하기 위해 한 level 낮아집니다. 따라서 유사한 $EReqCnt$를 가진 application은 동일한 priority level에 있습니다. 낮은 priority level에 있는 application은 보통 bandwidth 민감하며, 동일한 priority level은 memory bandwidth의 공정한 분배를 보장할 수 있습니다. 따라서 mapping function은 DMPS가 모든 application을 상대적으로 균일하고 빠른 속도로 진행시켜 고성능 및 공정성을 제공합니다. 

epoch 길이는 kilo cycle로, quantum 길이보다 훨씬 짧습니다. 세밀한 epoch의 주요 기능 중 하나는 $ReqPL$의 값을 작게 유지하고 동적 priority 효과를 보장하는 것입니다. $ReqPL$이 너무 클 경우, application의 priority를 낮추는 데 시간이 오래 걸리며, 동적 priority가 정적 priority로 약화될 수 있어 높은 불공정성과 starving을 유발할 수 있습니다. 또한 세밀한 epoch의 다른 기능은 starving을 피하는 것입니다. 낮은 priority level의 application의 request가 현재 epoch에서 차단되면, 다음 epoch의 시작에서 application의 동적 priority가 초기 priority level로 재설정되며, request는 나이가 많기 때문에 동일한 priority level의 request 사이에서 priority level가 부여됩니다. 따라서 낮은 priority level의 request이 시간적으로 처리됩니다. 

---

#### Summary : DMPS Prioritization Rules

Algorithm 3은 application으로 부터 온 memory request를 어떻게 DMPS가 schedule하는 지에 관련하여 요약했습니다. 

<p align="center"><img src="../../assets/images/102604.png" width="700px" height="700px" title="DMPS" alt="DMPS" ><img></p>

같은 bank에 두 memory request를 위해, 높은 priority level의 application으로 부터 request가 먼저 우선 처리됩니다. 두 application이 같은 priority level로 부여될 때, row-hit request가 먼저 우선처리 됩니다. 만일 동일하다면 오래된 request가 우선처리됩니다. 

---

#### Support for Software Control

DMPS는 system software에서 할당된 weights를 지원하고 그러므로 scheduling 동안에 높은 weigth를 가진 application을 우선처리 할 수 있습니다. TCM은 Memory access behavior를 고려하지 않고 큰 가중치를 가진 application에 무작위로 우선순위를 부여하는 것이 시스템 성능과 공정성을 심각하게 저하시킬 수 있음을 보여주었습니다. 따라서, DMPS는 group의 범위 내에서 SW 가중치를 존중합니다. weight가 작으면서 latency에 민감한 application들은 큰 weight를 가진 bandwidth 민감한 application보다 더 먼저 우선처리 됩니다. 이러한 것을 하기 위해서 metacontroller는 grouping할 때, software weights를 신경쓰지 않습니다. DMPS는 SW weight를 포함시키기 위해, 서로 다른 application에 서로 다른 level 당 threshold을 할당함으로써 구현할 수 있습니다. application의 level당 threshold는 metacontroller의 level 당 threshold와 해당 SW weight의 곱입니다. 높은 level당 threshold의 뜻은 application이 우선순위 level이 낮아지기 전에 더 많은 read request를 제공할 수 있다는 것입니다. 

<center>

$ReqPL_i=ReqPL\times SWWeight_i$

</center>

---

### Implementation and hardware cost

DMPS는 memory occupancy를 관제하고, 동적인 multilevel priority를 지원하기 위하여 추가적인 logic과 storage를 FR-FCFS보다 더 만들어줘야합니다. <Table I>에서 24-core와 4-channel system에서 memory controller당 storage와 logic cost를 보여줍니다. 

<p align="center"><img src="../../assets/images/23103001.png" width="500px" height="300px" title="DMPS" alt="DMPS" ><img></p>

memory occupancy를 관제하기 위하여, 각 application은 두 개의 counter를 두어 read request가 served된 횟수를 기록합니다. : 한 epoch를 위한 short-time counter 그리고 한 quantum을 위한 long-time counter. 같은 width의 adder은 각 counter을 업데이트하는데 필요합니다. 모든 request가 row hit일 때 wort case 경우, 두 개의 counter에 대한 최대 가능한 값은 epoch와 quantum의 길이를 최소 scheduling distance $t_{CCD}$로 나눈 결과입니다. 그러므로, 두 counter의 너비는 각각 9 bits와 16 bits입니다. 동적 multilevel priority를 지원하기 위해, DMPS는 처음의 우선순위를 저장하기 위하여, application당 1-bit register가 필요하고, 동적인 우선순위를 저장하기 위해 application 당 2-bit register 그리고 threshold $ReqPL$을 저장하기 위한 9-bit register가 필요합니다. application의 동적 priority를 short-time counter를 threshold $ReqPL$와 $2\times ReqPL$과 비교해서 모든 cycle에 업데이트를 진행합니다. 그래서 두 개의 9-bit comparators 가 각 application당 필요하게 됩니다. 

---

### Methodology and metric

#### System configuration

DMPS를 이전 5개의 scheduler와 같이 평가했는데, 사용한 memory system simulator은 USIMM입니다. 

<p align="center"><img src="../../assets/images/23103002.png" width="500px" height="300px" title="DMPS" alt="DMPS" ><img></p>


<p align="center"><img src="../../assets/images/23103003.png" width="500px" height="300px" title="DMPS" alt="DMPS" ><img></p>

---

#### Evaluation Metrics

평가 지표가 필요해서 가져와봤습니다.

multiprogrammed 환경에서 흔히 사용되는 평가지표로, system performance를 측정하기 위한 weighted speedup, fairness를 측정하기 위해 maximum slowdown, fairness와 system performance의 균형을 측정하기 위한 harmonic speedup이 있습니다. 

<center>

$Weighted \ Speedup = \sum^n_{i=1} \frac{IPC^{shared}_i}{IPC^{alone}_i}$

<p>

</p>

$Harmonic Speedup = \frac{N}{\sum^n_{i=1} \frac{IPC^{alone}_i}{IPC^{shared}_i}}$

<p>

</p>

$Maximum Slowdown = max_i\frac{IPC^{alone}_i}{IPC^{shared}_i}$

</center>

여기서 $IPC^{alone}_i$는 혼자 작동할 때, application $i$에 해당하는 instructions per cycle이며, $IPC^{shared}_i$는 다른 application과 같이 동작할 때, application $i$에 해당하는 instructions per cycle을 뜻합니다. 

<p align="center"><img src="../../assets/images/23103004.png" width="700px" height="500px" title="DMPS" alt="DMPS" ><img></p>


<p align="center"><img src="../../assets/images/23103005.png" width="700px" height="500px" title="DMPS" alt="DMPS" ><img></p>

<p align="center"><img src="../../assets/images/23103006.png" width="700px" height="500px" title="DMPS" alt="DMPS" ><img></p>

<p align="center"><img src="../../assets/images/23103007.png" width="700px" height="500px" title="DMPS" alt="DMPS" ><img></p>

<p align="center"><img src="../../assets/images/23103008.png" width="700px" height="500px" title="DMPS" alt="DMPS" ><img></p>

---

### Conclusions

본 논문은 새롭고 간단한 동적 multilevel priority 기반 memory access scheduler인 DMPS를 도입했습니다. 가장 이전의 application-aware scheduler는 높은 system 성능과 fairness를 추구하고 per-application ranking mechanism을 사용하여 높은 HW 복잡도와 cost를 초래합니다. 가장 최근의 application-aware scheduler는 HW 복잡도와 비용을 낮추려고 하며, group당 ranking 방식을 사용하여 빠른 scheduling 조치를 취하도록 하고, 이로 인해 시스템 성능과 공정성이 심각하게 저하됩니다. DMPS는 system 성능, fairness 그리고 HW 복잡도 사이의 균형을 제공합니다. memory interference를 유발하는 것을 측정하기 위해 memory occupancy를 도입하여, DMPS는 application을 간단히 두 개의 group을 분류할 수 있습니다. system performance를 향상시키기 위해, latency 민감한 application은 bandwidth 민감한 application보다 우선시 됩니다. application 전반에 fairness를 보장하기 위하여 DMPS는 동적으로 application을 multiple level로 우선순위 처리하고, memory occupancy가 큰 application은 가장 낮은 순위로 처리합니다. 

DMPS를 80 개의 workload로 다른 system configuration에서 평가하였고 이전의 5 개의 scheduler와 비교해보았습니다. 평가 결과, DMPS는 굉장히 높은 시스템 성능과 fairness를 FRFCFS, FRFCFS-Cap 그리고 per-group ranking scheduler BLISS보다 갖췄습니다. per-application ranking scheduler와 비교했을 때, DMPS는 HW 복잡도를 크게 감소시켰으며, 시스템 성능은 미세하게 낮췄지만, 높은 fairness를 달성하였습니다. 

그러므로 본 논문은 DMPS가 높은 system performance와 fairness를 낮은 HW 복잡도로 달성할 수 있으며, 현재나 미래의 multicore system에서도 적합하다는 것에 결론내릴 수 있습니다. 


---

### 추가설명

**(1) Memory Controller에서 공정성의 의미**

- 여러 프로세스나 application들이 공유하는 메모리 resource에 대한 access를 공정하게 조절하는 것을 의미함. 공정성을 보장한다는 것은, 어떤 프로세스나 application이 메모리 resource를 독점하거나 다른 프로세스의 메모리 access를 방해하지 않도록 하는 것을 말함. 

**(2) Memory Intensity**

- 메모리에 대한 access 빈도나 메모리 사용량을 나타내는 지표임. 즉, 프로그램 또는 작업이 주어진 시간 동안 얼마나 많은 메모리 access를 요청하는지를 나타내는 것을 의미할 수 있음. 고메모리 강도면 메모리 시스템에 더 큰 부하를 주는 경향이 있으며, 메모리 scheduling이나 resource allocation에서 특별한 전략이 필요할 수 있음. 

**(3) Application-Unaware**

- 특정 application의 특성이나 요구 사항을 고려하지 않고 동작하는 시스템이나 메커니즘을 말함. 예를 들어, Application-Unaware memory scheduler는 실행 중인 application의 특정 메모리 request 사항이나 동작 패턴을 고려하지 않고 메모리 access request를 처리함. 일반적으로 고려해야할 변수나 조건이 적기 때문에 단순하지만, 다양한 application의 특성에 따라 최적의 성능을 제공하기 어려울 수 있음. 

**(4) Application-Aware**

- 특정 application의 특성이나 요구 사항을 인식하고 그에 따라 동작하는 시스템이나 메커니즘을 말함. 실행 중인 application의 특정 요구 사항, 동작 패턴, 또는 성능 목표를 고려하여 최적화된 성능을 제공하려고 함. 이러한 접근 방식은 각 application의 개별적인 요구 사항에 맞게 자원을 할당하거나 우선순위를 조정하여 전체 시스템의 효율성을 향상시키는 데 도움이 됨. 

**(5) Memory Interference**

- 다중 processing 환경에서 여러 process 또는 thread가 동시에 memory resource(특히 main memory와 cache)에 access할 때 발생하는 현상을 가리킴. 이러한 상황에서 한 process의 memory access가 다른 process의 memory access에 영향을 미칠 수 있음.

- 특징
  - **Contention** : 여러 process 또는 thread가 동시에 memory resource에 access하려고 할 때 경쟁 상황이 발생. 이로 인해 access latency가 발생할 수 있음. 
  - **Cache Pollution** : 한 process의 memory access가 다른 process의 data를 cache에서 제거하는 경우 cache pollution이 발생. 이로 인해 다음에 그 data에 access할 때 memory latency가 증가하게 됨. 
  - **Row Buffer Conflict in DRAM** : DRAM에서, 특정 row에 access하려는 request가 다른 row에 access 중인 경우 row buffer conflict가 발생. 이로 인해 추가적인 latency가 발생. 

**(6) Application의 의미**

- 컴퓨터 프로그램이나 프로세스를 의미함. 멀티코어나 멀티스레드 환경에서 여러 application(또는 작업, task, process)이 동시에 실행될 수 있음. 이러한 application들은 resource(memory, CPU, I/O 등)에 대한 access를 경쟁적으로 수행하며, 이로 인해 서로 간의 interference나 performance 저하가 발생할 수 있음. application의 특성에 따라 scheduler는 효율적인 memory access 순서를 결정하게 됨.

**(7) memory occupancy의 의미**

- 컴퓨터 시스템에서 메모리 사용량을 나타내는 지표임. 구체적으로, 메모리 내에서 특정 프로세스, application, 또는 data가 차지하는 메모리 공간의 양이나 비율을 의미함. 

 