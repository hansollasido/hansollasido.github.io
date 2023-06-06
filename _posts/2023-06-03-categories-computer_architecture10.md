---
title: "[🤜컴퓨터구조설계 및 응용 10]"
excerpt: "Parallel PRocessors from Client to Cloud"

categories:
  - 컴구설
tags:
  - [수업]

permalink: /categories10/computer_architecture10/

use_math: true
toc: true
toc_sticky: true

date: 2023-06-04
last_modified_at: 2023-06-04
---

# 컴퓨터구조설계 및 응용

수업을 듣고 정리한 페이지 입니다.

---

**Introduction**

- Goal: connecting multiple computers to get higher performance
  - Multiprocessors
  - Scalability, availability, power efficiency
- Task-level (process-level) parallelism
  - High throughput for independent jobs
- Parallel processing program
  - Single program run on multiple processors
- Multicore microprocessors
  - Chips with multiple processors (cores)

Multi processor는 memory space가 독립적으로 다름. Multi core는 memory space share하고 연산만 따로 

---

**Hardware and Software**

- Hardware
  - Serial: e.g., Pentium 4
  - Parallel: e.g., quad-core Xeon e5345
- Software
  - Sequential: e.g., matrix multiplication
  - Concurrent: e.g., operating system
- Sequential/concurrent software can run on serial/parallel hardware
  - Challenge: making effective use of parallel hardware

막상 software가 병렬적으로 하는 것 같지만 실제로는  thread를 나눠서 번갈아 돌아가는 작업이 병렬적으로 보이는 것임.