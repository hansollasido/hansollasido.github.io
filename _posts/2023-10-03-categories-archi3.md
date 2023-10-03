---
title: "컴퓨터구조 공부3"
excerpt: "DataPath"

categories:
  - 컴퓨터구조
tags:
  - [컴퓨터구조]

permalink: /categories22/컴퓨터구조3/

use_math: true
toc: true
toc_sticky: true

date: 2023-10-03
last_modified_at: 2023-10-03
---

## Computer Architecture

우리가 항상 사용하는 컴퓨터는 정말 복잡한 시스템으로 구성되어 있다. 

---

### Data Path

ISA에서는 **RISC**와 **CISC**가 있다. RISC는 instruction 종류가 CISC보다 적다. Simplified Version이자, 현실적인 Pipelined이 가능한 버전이다. 

**Register**은 data를 회로 안에서 저장하는 기능을 한다. 우리가 아는 D-FF도 register의 한 종류이다.

**PC**는 Program Counter의 약자로 instruction memory가 수행할 곳을 가리킨다. 해당 instruction이 수행되면, PC <- PC + 4가 된다. 

<p align="center"><img src="../../assets/images/100301.jpg" width="500px" height="500px" title="CPU" alt="CPU" ><img></p>

C코드를 짤 때, if문과 case문을 많이 쓰는 것을 보았다. if문과 case문 처럼 특정 조건 만족시 해당하는 instruction을 불러와야 하는데, 이 때 PC는 PC + 4로 변경하는 것이 아닌 해당 조건에 맞는 PC의 값으로 변경되어야 한다. 

이에 Control Signal을 따로 준다. 

<p align="center"><img src="../../assets/images/100302.jpg" width="500px" height="500px" title="CPU" alt="CPU" ><img></p>

RISC-V는 여러가지 type들이 있다. s-type, i-type 등 어떤 instruction을 사용하느냐에 따라 구조가 달라진다. 

---

**R-Format**

- Read two register operands
- Perform arithmetic/logical operation
- Write register result

---

**Load/Store Instruction**

- Read register operands
- Calculate address using 12-bit offset
  - Use ALU, but sign-extend offset
- Load: Read memory and update register
- Store: Write register value to memory

메모리를 읽고 register에 저장하는 load instruction.

---

**Branch Instruction**

- Read register operands
- Compare operands
  - Use ALU, subtract and check Zero output
- Calculate target address
  - Sign-extend displacement
  - Shift left 1 place (halfword displacement)
  - Add to PC value

Target address = PC + immediate * 2 

<p align="center"><img src="../../assets/images/100303.jpg" width="500px" height="500px" title="Full Datapath" alt="Full Datapath" ><img></p>

중간에 ALU가 있어 어떤 instruction이냐에 따라 Data memory에 있는 Address가 달라진다. 


|ALU Control|Function|
|:---:|:---:|
|0000|AND|
|0001|OR|
|0010|add|
|0110|subtract|

opcode에서 2-bit ALUOp를 추출하여 ALU control을 만들 수 있다.

|opcode|ALUOp|Operation|Opcode field|ALU function|ALU control|
|:---:|:---:|:---:|:---:|:---:|:---:|
|ld|00|load register|XXXXXX|add|0010|
|sd|00|store register|XXXXXX|add|0010|
|beq|01|branch on equal|XXXXXX|subtract|0110|
|R-type|10|add|100000|add|0010|
|||subtract|100010|subtract|0110|
|||AND|100100|AND|0000|
|||OR|100101|OR|0001|

<p align="center"><img src="../../assets/images/100304.jpg" width="500px" height="500px" title="Full Datapath" alt="Full Datapath" ><img></p>

Instruction이 어떠냐에 따라 control이 달라지면서 해당되는 ALU가 작용한다. 

---

**Performance Issues**

- Longest delay determines clock period
  - Critical path: load instruction
  - instruction memory -> register file -> ALU -> data memory -> register file
- Not feasible to vary period for different instructions

---

한 instruction이 완료될 때까지 다음 instruction을 수행못한다는 치명적인 문제가 발생한다. 이에 **pipeline**을 두어 성능을 올린다. 
