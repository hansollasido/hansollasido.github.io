---
title: "[🏈컴퓨터구조설계 및 응용 6]"
excerpt: "Pipeline"

categories:
  - 컴퓨터구조설계
tags:
  - [수업]

permalink: /categories10/computer_architecture6/

use_math: true
toc: true
toc_sticky: true

date: 2023-04-18
last_modified_at: 2023-04-18
---

# 컴퓨터구조설계 및 응용

수업을 듣고 정리한 페이지 입니다.

---

#### Pipelining Analogy

- Pipelined laundry: overlapping execution
  - Parallelism improves performance
  - 병렬화로 throughput의 성능을 극대화함
  - stage 개수 배만큼의 speed up을 볼 수 있음
  - latency는 보장 못함

---

#### RISC-V pipeline

- Five stages, one step per stage
  - IF: Instruction fetch from memory
  - ID: Instruction decode & register read
  - EX: Execute operation or calculate address
  - MEM: Access memory operand
  - WB: Write result back to register

---

#### Pipeline Performance

- Assume time for stages is
  - 100ps for register read or write
  - 200ps for other stages

- Compare pipelined datapath with single-cycle datapath

<p align="center"><img src="../../assets/images/041811.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

- 가장 느린 Total time을 기준으로 clock period를 설정합니다.

---

#### Pipeline Performance

<p align="center"><img src="../../assets/images/041812.jpg" width="800px" height="600px" title="OP code 예시" alt="OP code" ><img></p>

---

#### Pipeline Speedup

- If all stages are balanced
  - all take the same time
  - $\mathrm{Time between instructions(pipelined) = \frac{Time between instructions(nonpipelined)}{Number of stages}}$
- If not balanced, speedup is less
  - 따라서 balance를 맞추는 것이 중요합니다.
- Speedup due to increased throughput
  - **Latency (time for each instruction) does not decrease**

  ---

#### Pipeline Operation

- Cycle-by-cycle flow of instructions through the pipelined datapath
  - "Single-clock-cycle" pipeline diagram
    - Shows pipeline usage in a single cycle
    - Highlight resources used
  - c.f. "multi-clock-cycle" diagram
    - Graph of operation over time
- We'll look at "single-clock-cycle" diagrams for load & store

---

#### Single-Cycle Pipeline Diagram

- State of pipeline in a given cycle

<p align="center"><img src="../../assets/images/041813.jpg" width="800px" height="600px" title="OP code 예시" alt="OP code" ><img></p>

각 stage를 나누고 그 사이사이에 register을 둡니다. 각 stage마다 역할이 다르며 access하는 memory, register도 다릅니다.

---

#### Multi-Cycle Pipeline Diagram

- Form showing resource usage

<p align="center"><img src="../../assets/images/041814.jpg" width="800px" height="600px" title="OP code 예시" alt="OP code" ><img></p>

---

#### WB for Load

<p align="center"><img src="../../assets/images/041815.jpg" width="800px" height="600px" title="OP code 예시" alt="OP code" ><img></p>

Pipeline시 발생하는 문제점입니다. WB시 register에 write하게 되는데 해당 stage를 수행하고 있는 instruction이 그에 해당하는 data를 가져오는 것이 아닌 WB된 data를 가져올 수 있습니다. 

따라서 아래와 같이 해당하는 data를 가져올 수 있도록 datapath를 수정해줍니다.

<p align="center"><img src="../../assets/images/041816.jpg" width="800px" height="600px" title="OP code 예시" alt="OP code" ><img></p>

---

#### Pipelined Control

- Control signals derived from instruction
  - As in single-cycle implementation
  - Control signal도 instruction에 맞게끔 계속 끌고 가야합니다.

<p align="center"><img src="../../assets/images/041817.jpg" width="800px" height="600px" title="OP code 예시" alt="OP code" ><img></p>

---

#### Pipelined Control

<p align="center"><img src="../../assets/images/041818.jpg" width="800px" height="600px" title="OP code 예시" alt="OP code" ><img></p>

---

### 퀴즈

어떤 프로세서에서 아래와 같은 Load/Store, Arithmetic, Branch의 세 가지 type의 instruction을 제공하려 하며, 각 type 별로 필요한 수행 단계와 단계별 소요시간은 아래와 같다.

<p align="center"><img src="../../assets/images/041819.png" width="800px" height="600px" title="OP code 예시" alt="OP code" ><img></p>

이러한 프로세서를 single-cycle, multi-cycle, 6-stage pipelining 방법으로 설계할 때, 성능을 다음과 같이 분석하시오.

1) Maximum operating clock frequency
  - Single-cycle <font color="red"> 1 GHZ </font>
    - <font color="red">(100 + 50 + 150 + 200 + 250 + 250 = 1000 ps = 1ns)</font>
  - Multi-cycle <font color="red"> 4 GHZ </font>
    - <font color="red">(한 stage에 250ps)</font>
  - Pipelining <font color="red">4 GHZ </font>
    - <font color="red">(한 stage에 250ps)</font>

2) 10만개의 instruction으로 구성된 프로그램(각 type이 30%, 40%, 30%로 구성)을 동작시킬 때의 MIPS(Millon instructions per second)
- Single-cycle <font color="red"> 1,000 MIPS </font>
  - <font color="red">(avg.CPI = 1 -> CPU time per instruction = 1ns -> 10^9 instructions/s -> 1,000 MIPS)</font>
- Multi-cycle <font color="red"> 800 MIPS </font>
  - <font color="red">(avg.CPI = 6x0.3 + 5x0.4 + 4x0.3 = 5 -> CPU time per instruction = 0.25x5 = 1.25ns -> 10^9/1.25 instructions/s -> 800 MIPS)</font>
- Pipelining <font color="red"> 4,000 MIPS </font>
  - <font color="red">(avg.CPI $\approx$1->CPU time per instruction = 0.25ns -> 4x10^9 instructions/s -> 4,000 MIPS)</font>