---
title: "컴퓨터구조 공부2"
excerpt: "Instruction"

categories:
  - 컴퓨터구조
tags:
  - [컴퓨터구조]

permalink: /categories22/컴퓨터구조2/

use_math: true
toc: true
toc_sticky: true

date: 2023-10-03
last_modified_at: 2023-10-03
---

## Computer Architecture

우리가 항상 사용하는 컴퓨터는 정말 복잡한 시스템으로 구성되어 있다. 

---

### Instruction

ISA (Instruction Set Architecture)에서는 RISC와 CISC가 있다. 그 중에서도 RISC-V가 예시 ISA로 많이 사용되는데, RISC-V의 instruction에 대하여 알아보겠다.

---

RISC-V Encoding Summary

|Name|||Field||||Comments
|Field Size|7 bits|5 bits|5 bits|3 bits|5 bits|7 bits||
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|R-type|funct7|rs2|rs1|funct3|rd|opcode|Arithmetic instruction format|
|I-type|immediate[11:0]||rs1|funct3|rd|opcode|Loads & immediate arithmetic|
|S-type|immed[11:5]|rs2|rs1|funct3|immed[4:0]|opcode|Stores|
|SB-type|immed[12,10:5]|rs2|rs1|funct3|immed[4:1,11]|opcode|Conditional branch format|
|UJ-type|immediate[20,10:1,11,19:12]||||rd|opcode|Uncoditional jump format|
|U-type|immediate[31:12]||||rd|opcode|Upper immediate format|

---

컴퓨터마다 instruction set이 다르다. 요즘에는 RISC를 많이 쓰는 편이다. HW가 간단해지기 때문이다. 

4-byte (word) 단위로 32-bit instruction가 encoding되어 있다. 따라서 규율에 따라 instruction을 뜯어보는 것이 굉장히 중요하다.

**C code:**

      f = (g + h) - (i + j);

**Compiled RISC-V Code:**

      add x5, x20, x21

      add x6, x22, x23

      sub x19, x5, x6

이 된다. 

---

Memory

- Main memory used for composite data
- To apply arithmetic operations
  - Load values from memory into registers
  - Store result from register to memory
- Memory is byte addressed
  - Each address identifies an 8-bit byte

---

Memory operand 예제를 들자면

**C code:**

      A[12] = h + A[8];
      //h in x21, base address of A in x22

**Compiled RISC-V code:**

      ld x9, 64(x22)

      add x9, x21, x9

      sd x9, 96(x22)

---

메모리에서 읽고 register에 저장하고 다시 store한다. 

x22에 Base Address 값이 있으며 x22에 저장된 값보다 64Byte더 떨어진 곳에서 값을 가져온다. 그리고 x9에 저장한다. 

위는 doubleword 단위의 memory addressing 구조이다. 

Register은 Memory보다도 훨씬 access하기 빠르기 때문에 많이 사용한다. Memory data를 가졍는 것은 load랑 store instruction을 거치기 때문에, memory access하는 것은 최소화해야한다. Compiler가 알아서 최적화된 instruction을 만들기 때문에 memory access는 최소화한다. 

---

Immediate Operands

- addi x22, x22, 4

---

RISC-V R-format instructions

|funct7|rs2|rs1|funct3|rd|opcode|
|---|---|---|---|---|---|
|7 bits|5 bits|5 bits|3 bits|5 bits|7 bits|

- Instruction fields
  - opcode: operation code
  - rd: destination register number
  - funct3: 3-bit function code (additional opcode)
  - rs1: the first source register number
  - rs2: the second source register number
  - funct7: 7-bit function code (additional opcode)

R-format의 예제를 들자면

add x9, x20, x21의 instruction은

|funct7|rs2|rs1|funct3|rd|opcode|
|---|---|---|---|---|---|
|0000000|10101|10100|000|01001|0110011|

으로 015A04B3이다. 

---

RISC-V I-format Instructions

|immediate|rs1|funct3|rd|opcode|
|---|---|---|---|---|
|12 bits|5 bits|3 bits|5 bits|7 bits|

- Immediate arithmetic and load instructions
  - rs1: source or base address register number
  - immediate: constant operand, or offset added to base address
    - 2's-complement, sign extended

---

RISC-V S-format Instructions

|imm[11:5]|rs2|rs1|funct3|imm[4:0]|opcode|
|---|---|---|---|---|---|
|7 bits|5 bits|5 bits|3 bits|5 bits|7 bits|

- Different immediate format for store instructions
  - rs1: base address register number
  - rs2: source operand register number
  - immediate: offset added to base address
    - split so that rs1 and rs2 fields always in the same place

---

여러 logical operations도 존재한다. 

And, or,  Xor 등 여러 논리적 계산도 가능하다. 

Conditional Operation도 존재한다. 

- beq rs1, rs2, L1
  - if (rs1 == rs2) branch to instruction labeled L1

- bne rs1, rs2, L1
  - if (rs1 != rs2) branch to instruction labeled L1


예시를 들어보자

**C code:**

      if (i==j) f = g + h;

      else f = g - h;

**Compiled RISC-V code:**

      bne x2,, x23, Else

      add x19, x20, x21

      beq x0, x0, Exit //unconditional

      Else: sub x19, x20, x21

      Exit: ....

---

- blt rs1, rs2, L1
  - if (rs1 < rs2) branch to instruction labeled L1

- bge rs1, rs2, L1
  - if (rs1 >= rs2) branch to instruction labeled L1

---

Branch Addressing

SB format:

|imm[12]|imm[10:5]|rs2|rs1|funct3|imm[4:1]|imm[11]|opcode|
|---|---|---|---|---|---|---|---|
||7 bits|5 bits|5 bits|3 bits|5 bits|7 bits|

---

**RISC-V addressing summary**

1) Immediate addressing

|immediate|rs1|funct3|rd|op|
|---|---|---|---|---|
|12 bits|5 bits|3 bits|5 bits|7 bits|

2) Register addressing

|funct7|rs2|rs1|funct3|rd|op|
|---|---|---|---|---|---|
|7 bits|5 bits|5 bits|3 bits|3 bits|5 bits|7 bits|

3) Base addressing

|immediate|rs1|funct3|rd|op|
|---|---|---|---|---|
|12 bits|5 bits|3 bits|5 bits|7 bits|

4) PC-relative addressing

|imm|rs2|rs1|funct3|imm|op|
|---|---|---|---|---|---|
|7 bits|5 bits|5 bits|3 bits|5 bits|7 bits|

---

Backward compatibility 때문에 instruction set은 변하지 않으나, instruction 개수가 점차 늘어나고 있다. 

가장 simple한 것이 가장 빠른 것이며, instruction 개수가 많다고 성능이 좋은 것은 아니다. 현재 RISC-V의 대표적인 예로 x86을 들 수 있다. 