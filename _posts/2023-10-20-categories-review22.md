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


### 추가설명

(1) Memory Controller에서 공정성의 의미

- 여러 프로세스나 application들이 공유하는 메모리 resource에 대한 access를 공정하게 조절하는 것을 의미함. 공정성을 보장한다는 것은, 어떤 프로세스나 application이 메모리 resource를 독점하거나 다른 프로세스의 메모리 access를 방해하지 않도록 하는 것을 말함. 

(2) Memory Intensity

- 메모리에 대한 access 빈도나 메모리 사용량을 나타내는 지표임. 즉, 프로그램 또는 작업이 주어진 시간 동안 얼마나 많은 메모리 access를 요청하는지를 나타내는 것을 의미할 수 있음. 고메모리 강도면 메모리 시스템에 더 큰 부하를 주는 경향이 있으며, 메모리 scheduling이나 resource allocation에서 특별한 전략이 필요할 수 있음. 

