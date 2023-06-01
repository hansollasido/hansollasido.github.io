---
title: "[👊컴퓨터구조설계 및 응용 7]"
excerpt: "Hazards"

categories:
  - 컴구설
tags:
  - [수업]

permalink: /categories10/computer_architecture7/

use_math: true
toc: true
toc_sticky: true

date: 2023-06-01
last_modified_at: 2023-06-01
---

# 컴퓨터구조설계 및 응용

수업을 듣고 정리한 페이지 입니다.

---

#### Hazard

- 다음 cycle에 다음 instruction이 들어가야 하는데 pipeline 사용시 문제가 발생할 수 있음
- **Structure hazard**
  - 이전 instruction을 기다려서 data를 read/write 완료해야 함
- **Control hazard**
  - 이전 instruction에 따라 control action을 결정함

---

#### Structure Hazards

- resource의 사용으로 인한 충돌이 발생
- single memory, RISC-V pipeline
  - load/store은 data access를 필요로 함
  - Instruction fetch가 해당 사이클에 *stall*할 수 있음
    - 이로 인해 pipeline에 bubble이 발생할 수 있음

- **pipelined datapath는 instruction과 data memorie를 분리할 필요가 있음**
  - 아니면 instruction과 data cache

---

#### Data Hazards

- instruction이 이전 instruction으로 인한 data access에 의존될 때

예)

add **x19**, x0, x1

sub x2, **x19**, x3

<p align="center"><img src="../../assets/images/060101.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

- bubble을 생성하여 올바른 값을 가져오도록 만들어서 해결

---

#### Forwarding (aka Bypassing)

- Use result when it is computed
  - register에서 저장될 때까지 기다리지 않음
  - datapath에서 추가적인 연결을 필요로 함

<p align="center"><img src="../../assets/images/060102.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

**ALU instruction에서 생각해보자**


sub **x2**, x1, x3 \
and x12, **x2**, x5 \
or x13, x6, **x2** \
add x14, **x2**, **x2** \
sd x15, 100(**x2**)

- x2가 계속 쓰이는데 hazard가 발생할 수 있음
- 이를 forawrding로 해결 가능
  - 하지만 어떻게 forward를 detect할까

---

#### Dependencies & Forwarding

<p align="center"><img src="../../assets/images/060103.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

- sub x2, x1, x3의 연산 결과를 바로 forwarding하여 사용

---

#### Forward의 필요성 파악하기

- Data hazards when
  - 1a. EX/MEM.RegisterRd = ID/EX.RegisterRs1
  - 1b. EX/MEM.RegisterRd = ID/EX.RegisterRs2
  - 2a. MEM/WB.RegisterRd = ID/EX.RegisterRs1
  - 2b. MEM/WB.RegisterRd = ID/EX.RegisterRs2
  - 이 때 hazard 발생

---

#### Forwarding Paths

<p align="center"><img src="../../assets/images/060104.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

---

#### Forwarding 조건

<p align="center"><img src="../../assets/images/060105.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

---

#### Detecting the Need to Forward

- Only if forwarding instruction will write to a register
  - EX/MEM.RegWrite, MEM/WB.RegWrite

- Only if Rd for that instruction is not x0
  - EX/MEM.RegisterRd $\neq$ 0
  - MEM/WB.RegisterRd $\neq$ 0

---

#### Double Data Hazard

- Consider the sequence:

add **x1**, x1, x2 \
add **x1**, **x1**, x3 \
add x1, **x1**, x4

- 위와 같은 경우 연속으로 hazard가 발생함
- 가장 최근의 연산된 결과를 받아야 함
- Revise MEM hazard condition
  - Only fwd if EX hazard condition isn't true

---

#### Revised Forwarding Condition

- MEM hazard
  - if (MEM/WB.RegWrite) and (MEM/WB.RegisterRd $\neq$ 0) 
  - <font color="blue">and not (EX/MEM.RegWrite and (EX/MEM.RegisterRd $\neq$ 0)</font>
  - <font color="blue">and (EX/MEM.RegisterRd $\neq$ ID/EX.RegisterRs1))</font>
  - and (MEM/WB.RegisterRd = ID/EX.RegisterRs1) ForwardA = 01

  <br>

  - if (MEM/WB.RegWrite) and (MEM/WB.RegisterRd $\neq$ 0)
  - <font color="blue">and not (EX/MEM.RegWrite and (EX/MEM.RegisterRd $\neq$ 0)</font>
  - <font color="blue">and (EX/MEM.RegisterRd $\neq$ ID/EX.RegisterRs2))</font>
  - and (MEM/WB.RegisterRd = ID/EX.RegisterRs2) ForwardB = 01

---

#### Datapath with Forwarding

<p align="center"><img src="../../assets/images/060106.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

---

#### Load-Use Data Hazard

- forwarding으로 stall을 항상 피할 수는 없다
  - 연산이 끝나지 않았다면 stall 필요
  - 시간을 거꾸로 갈 수는 없기 때문에 stall 필요

  <p align="center"><img src="../../assets/images/060107.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

---

#### How to Stall the Pipeline

- Force control values in ID/EX register to 0
  - EX, MEM and WB do nop (no-operation)

- Prevent update of PC and IF/ID register
  - Using instruction is decoded again
  - Following instruction is fetched again
  - 1-cycle stall allows MEM to read data for ld
    - Can subsequently forward to EX stage


  <p align="center"><img src="../../assets/images/060108.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

---

#### Datapath with Hazard Detection

<p align="center"><img src="../../assets/images/060109.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

---

#### Stalls and Performance

- Stall은 성능을 떨어뜨릴 수 있음
  - 정확한 결과를 위하여 반드시 필요함

- Compiler는 hazard와 stall을 피하기 위해 code를 정리할 수 있음
  - pipeline 구조의 기본 지식을 알아야 함

---

#### Code Scheduling to Avoid Stalls

- Recoder code to avoid use of load result in the next instruction
- C code for a = b + e; c = b + f;

<p align="center"><img src="../../assets/images/060110.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

---

#### Control Hazards

- Branch determines flow of control
  - Fetching next instruction depends on branch outcome
  - Pipeline can't always fetch correct instruction 
    - Still working on ID stage of branch

- In RISC-V pipeline
  - Need to compare registers and compute target early in the pipeline
  - Add hardware to do it in ID stage

--- 

#### Branch Hazards

- If branch outcome determined in MEM

<p align="center"><img src="../../assets/images/060111.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

---

#### Branch Prediction

- Longer pipelines can't readily determine branch outcome early 
  - Stall penalty becomes unacceptable
- Predict outcome of branch
  - Only stall if prediction is wrong
- In RISC-V pipeline
  - Can predict branches not taken
  - Fetch instruction after branch, with no delay

---

#### Reducing Branch Delay

- Move hardware to determine outcome to ID stage
  - Target address adder
  - Register comparator

---

#### Stall on Branch

- next instruction fetch 이전에 branch outcome이 결정될 때까지 기다림

<p align="center"><img src="../../assets/images/060112.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

<p align="center"><img src="../../assets/images/060113.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

---

#### Dynamic Branch Prediction

- In deeper and superscalar pipelines, branch penalty is more significant
- Use dynamic prediction
  - Branch prediction buffer (aka branch history table)
  - Indexed by recent branch instruction addresses
  - Stores outcome (taken/not taken)
  - To execute a branch 
    - Check table, expect the same outcome
    - Start fetching from fall-through or target
    - If wrong, flush pipeline and flip prediction

##### 1-Bit Predictor: Shortcoming

- Inner loop branches mispredicted twice

##### 2-Bit Predictor

- Only change prediction on two successive mispredictions

<p align="center"><img src="../../assets/images/060114.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

---

#### Instruction-Level Parallelism (ILP)

- Pipelining: executing multiple instructions in parallel
- To increase ILP
  - Deeper pipeline
    - Less work per stage => shorter clock cycle
  - Multiple issue
    - Replicate pipeline stages => multiple pipelines
    - Start multiple instructions per clock cycle
    - CPI < 1, so use Instructions Per Cycle (IPC)

---

#### Concluding

- 파이프라인은 개념은 쉽지만 디테일하게 보면 어려움
- ISA가 datapath와 control 설계에 영향을 줌
- 병렬화를 사용하여 throughput을 향상시키는데 latency에서는 손해를 봄
- Hazards: structural, data, control 
  - Hazard를 해결해야 pipeline을 효과적으로 만들 수 있음
