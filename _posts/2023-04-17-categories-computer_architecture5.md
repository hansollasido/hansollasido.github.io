---
title: "[🏐컴퓨터구조설계 및 응용 5]"
excerpt: "The Processor Datapath"

categories:
  - 컴구설
tags:
  - [수업]

permalink: /categories10/computer_architecture5/

use_math: true
toc: true
toc_sticky: true

date: 2023-04-17
last_modified_at: 2023-04-18
---

# 컴퓨터구조설계 및 응용

수업을 듣고 정리한 페이지 입니다.

---

#### Introduction

- CPU performance factors
  - Instruction count
    - Determined by ISA and compiler
    
  - CPI and Cycle time
    - Determined by CPU hardware

- RISC-V의 두 가지 실행법
  - Simplified version
  - More realistic pipelined version

- Simple subset, shows most aspects
  - Memory reference: ld, sd
  - Arithmetic/logical: add, sub, and, or
  - Control transfer: beq

---

#### Sequential Elements

- Register with write control
  - clock이 edged될 때 write control이 1이면 update합니다.
  - 저장된 값이 나중에 필요하면 사용됩니다.

---

#### Clocking Methodology

Sequential logic 중간에 combinational logic을 둡니다. 이 때, 소요되는 시간은 clock cycle 한 주기 안에 들어와야 합니다. Critical Path가 clock period를 결정합니다.

---

#### Instruction Execution

- PC -> instruction memory, fetch instruction
  - PC는 program counter의 약자로 실행할 instruction을 가리킵니다.
- Register numbers -> register file, read registers
- Depending on instruction class
  - Use ALU to calculate
    - Arithmetic result
    - Memory address for load/store
    - Branch comparison
  - Access data memory for load/store
  - PC <- target address or PC +4
    - PC는 매번 4씩 늘어납니다. 
    - Instruction크기가 4bite 이기 때문에 4씩 늘어납니다.

---

#### CPU Overview

<p align="center"><img src="../../assets/images/041801.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

wire가 두 개 이상 겹칠 때에는 multiplexer를 사용하여 control 해야합니다.

---

#### Control

<p align="center"><img src="../../assets/images/041802.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

control signal을 보내줘서 multiplexer를 제어하는 것을 볼 수 있습니다. 

---

#### Instruction Fetch

Instruction이 아닌 Instruction 주소값을 가리키기 때문에 64bit register PC를 사용합니다. 

---

#### R-Format Instructions

- Read two register operands 
- Perform arithmetic/logical operation
- Write register result

---

#### Load/Store Instructions

- Read register operands
- Calculate address using 12-bit offset
  - Use ALU, but sign-extend offset
- Load: Read memory and update register
- Store: Write register value to memory

이 때 sign extension이 있습니다. 

- Datapath

<p align="center"><img src="../../assets/images/041804.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>


---

#### Branch Instructions

- Read register operands
- Compare operands
  - Use ALU, subtract and check Zero output
- Calculate target address
  - Sign-extend displacement
  - Shift left 1 place (halfword displacement)
    - Target Address = PC + immediate x 2
  - Add to PC value

- DataPath

<p align="center"><img src="../../assets/images/041803.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>


---

#### Full Datapath


<p align="center"><img src="../../assets/images/041805.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

---

#### ALU Control

- ALU used for 
  - Load/Store: F = add
  - Branch: F = subtract
  - R-type: F depends on opcode

|ALU control|Function|
|---|---|
|0000|AND|
|0001|OR|
|0010|add|
|0110|subtract|

- Assume 2-bit ALUOp derived from opcode
  - Combinational logic derives ALU control

|opcode|ALUOp|Operation|Opcode field|ALU function|ALU control|
|---|---|---|---|---|---|
|ld|00|load register|XXXXX|add|0010|
|sd|00|store register|XXXXX|add|0010|
|beq|01|branch on equal|XXXXX|subtract|0110|
|R-type|10|add|100000|add|0010|
|R-type|10|subtract|100010|subtract|0110|
|R-type|10|AND|100100|AND|0000|
|R-type|10|OR|100101|OR|0001|

---

#### Datapath With Control

<p align="center"><img src="../../assets/images/041806.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

#### R-Type Instruction

<p align="center"><img src="../../assets/images/041807.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

#### Load Instruction

<p align="center"><img src="../../assets/images/041808.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

#### BEQ Instruction

<p align="center"><img src="../../assets/images/041809.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

---

#### Performance Issues

- Longest delay etermines clock period
  - Critical path: load instruction
  - Instruction memory -> register file -> ALU -> data memory -> register file
- Not feasible to vary period for different instructions
- Violates design principle
  - Making the common case fast
- We will improve performance by pipelining
  - pipelining은 정말 중요함.

---

### 퀴즈

1) 이번주에 배운 'single-cycle datapath'에서 짧은 delay의 instruction에 대해서 비효율성이 발생하는 이유는?
- <font color="red">가장 긴 instr에 clock interval을 맞추어야 하므로, 짧은 instr는 interval 내에 수행을 완료하고 idle한 시간 발생</font>

2) 한 instruction이 끝난 후 다음 instruction을 수행하기 위해 PC 값에 더하는 값이 4인 이유는?
- <font color="red">한 instr의 크기가 32bit (4 byte)이므로 

3) 아래 두 경우의 register에 대해 출력되는 Q 값을 표시하시오. 
<p align="center"><img src="../../assets/images/041810.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>
   