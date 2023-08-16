---
title: "8/04"
excerpt: "cache, cpu 확인"

categories:
  - 연구 일지
tags:
  - [linux, code, research]

permalink: /categories15/research/

use_math: true
toc: true
toc_sticky: true

date: 2023-08-04
last_modified_at: 2023-08-04
---

## Linux에서 cpu에 대한 정보 확인하는 방법

```bash
lscpu
```

큰 거 없이 위를 실행하면 됨

## Linux에서 cache 대한 정보 확인하는 방법

```bash
getconf LEVEL1_DCACHE_ASSOC 
```
를 하거나

```bash
cat /sys/devices/system/cpu/cpu0/cache/index*/type
cat /sys/devices/system/cpu/cpu0/cache/index*/size
cat /sys/devices/system/cpu/cpu0/cache/index*/ways_of_associativity
cat /sys/devices/system/cpu/cpu0/cache/index*/coherency_line_size
```
를 하면 cache에 대한 정보를 얻을 수 있음

## GEMM 연산에 대하여

[GEMM](https://computing-jhson.tistory.com/40)

위 블로그를 참고하였음. IM2COL로 image를 변환한 다음에 convolution을 하는 GEMM-convolution은 기존의 Direct convolution보다 병렬 처리를 통한 성능 향상이 가능해짐. 

<p align="center"><img src="../../assets/images/080409.png" width="400px" height="400px" title="GEMM" alt="GEMM" ><img></p>

## 사용하는 컴퓨터 구조

사용하는 서버의 컴퓨터 구조를 확인해보았다.

```
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
Byte Order:          Little Endian
CPU(s):              32
On-line CPU(s) list: 0-31
Thread(s) per core:  2
Core(s) per socket:  16
Socket(s):           1
NUMA node(s):        1
Vendor ID:           GenuineIntel
CPU family:          6
Model:               85
Model name:          Intel(R) Xeon(R) Gold 6246R CPU @ 3.40GHz
Stepping:            7
CPU MHz:             3400.000
CPU max MHz:         4100.0000
CPU min MHz:         1200.0000
BogoMIPS:            6800.00
Virtualization:      VT-x
L1d cache:           32K
L1i cache:           32K
L2 cache:            1024K
L3 cache:            36608K
NUMA node0 CPU(s):   0-31
Flags:               fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf pni pclmulqdq dtes64 monitor ds_cpl vmx smx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid dca sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb cat_l3 cdp_l3 invpcid_single ssbd mba ibrs ibpb stibp ibrs_enhanced tpr_shadow vnmi flexpriority ept vpid ept_ad fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid cqm mpx rdt_a avx512f avx512dq rdseed adx smap clflushopt clwb intel_pt avx512cd avx512bw avx512vl xsaveopt xsavec xgetbv1 xsaves cqm_llc cqm_occup_llc cqm_mbm_total cqm_mbm_local dtherm ida arat pln pts hwp hwp_act_window hwp_epp hwp_pkg_req pku ospke avx512_vnni md_clear flush_l1d arch_capabilities
```

cpu은 32개이고 L1, L2, L3를 갖추고 있다. 

cache에 대한 정보를 분석해본 결과

**L1DCache**
- coherency line size : 64
- number of sets : 64
- shared cpu map : 00010001
- id : 0
- physical_line_partition : 1
- size : 32K
- ways of associativity : 8
- level : 1
- shared cpu list : 0,16
- Data

등을 확인할 수 있음 (L2, L3는 생략)

## 컴퓨터 구조

컴퓨터구조및설계 수업을 들었지만 잊어버린 것도 있어서 physical address관련하여 caching을 어떻게 하는지에 대한 개념을 다시 보았다.

아래 사이트를 참고하였다.

[cache 관련 정보](https://www.sciencedirect.com/topics/computer-science/cache-line-size)

<p align="center"><img src="../../assets/images/080410.png" width="400px" height="400px" title="GEMM" alt="GEMM" ><img></p>

대충 옛 기억을 통해 현재 서버 컴퓨터의 cache 구조를 보면

<p align="center"><img src="../../assets/images/080411.jpg" width="400px" height="400px" title="GEMM" alt="GEMM" ><img></p>

위와 같다. Tag는 훨씬 길겠지만 짧게 표현하였다. 이제 이것을 GEM5에 구현하려고 한다. 먼저 CPU 여러 개를 달고 L1, L2, L3를 구현하자. 

CORE 개수는 4개로 L1, L2, L3 Hierarchy 적용하고, associativity를 어떻게 적용할지 논의해보자.

GEMM 연산 할 때, coherency가 좋은 구조가 분명 있을 것 같다. 