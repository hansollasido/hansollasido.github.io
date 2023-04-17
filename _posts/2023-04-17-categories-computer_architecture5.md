---
title: "[컴퓨터구조설계 및 응용 5]"
excerpt: "The Processor Datapath"

categories:
  - 컴퓨터구조설계
tags:
  - [수업]

permalink: /categories10/computer_architecture5/

use_math: true
toc: true
toc_sticky: true

date: 2023-04-17
last_modified_at: 2023-04-17
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

