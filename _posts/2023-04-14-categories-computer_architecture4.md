---
title: "[컴퓨터구조설계 및 응용 4]"
excerpt: "Instructions: Language of the Computer"

categories:
  - 컴퓨터구조설계
tags:
  - [수업]

permalink: /categories9/computer_architecture4/

use_math: true
toc: true
toc_sticky: true

date: 2023-04-14
last_modified_at: 2023-04-14
---

# 컴퓨터구조설계 및 응용

수업을 듣고 정리한 페이지 입니다.

---

컴퓨터는 instruction에 따라 명령을 수행하는데, instruction format은 다음과 같이 있습니다. 목적에 따라 instruction format을 다르게 생성합니다.

R-format/S-format/I-format

<p align="center"><img src="../../assets/images/041419.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

computer의 instruction에 대한 datapath는 위와 같습니다. 그럼 [(1) RISC-V](#추가설명)에서의 format을 알아보겠습니다.

---

#### RISC-V format

|Name|||Field||||Comments|
|---|---|---|
|<mark style='background-color: #fff5b1'>(Field Size)|<mark style='background-color: #fff5b1'>7 bits|<mark style='background-color: #fff5b1'>5 bits|<mark style='background-color: #fff5b1'>5 bits|<mark style='background-color: #fff5b1'>3 bits|<mark style='background-color: #fff5b1'>5 bits|<mark style='background-color: #fff5b1'>7 bits</mark>|
|R-type|funct7|rs2|rs1|funct3|rd|opcode|Arithmetic instruction format|
|I-type|immediate[11:0]||rs1|funct3|rd|opcode|Loads&immediate arithmetic|
|S-type|immed[11:5]|rs2|rs1|funct3|immed[4:0]|opcode|Stores|
|SB-type|immed[12,10:5]|rs2|rs1|funct3|immed[4:1,11]|opcode|Conditional branch format|
|UJ-type||immediate[20,10:1,11,19:12]|||rd|opcode|Unconditional jump format|
|U-type||immediate[31:12]|||rd|opcode|Upper immediate format|

---

#### Insturction Set

RISC : Reduced Instruction Set Architecture
- HW 간단해집니다.
- 요즘 많이 쓰입니다.

CISC : Complex Instruction Set Architecture

-> many modern computers have simple instruction sets.

RISC-V Foundation에서 관리하고 있습니다.
- [riscv.org](https://riscv.org/)

binary number로 encoding되어 있습니다. machine code라고도 불립니다.

32-bit instruction words로 Encoding되어 있습니다.

Regularity(간단한 것이 중요해서 규율이 중요합니다.)

---

#### Arithmetic

순차적으로 연산을 진행합니다.

**예)**

C code:
```c
f = (g+h)-(i+j);
```

Compiled된 RISC-V code:
```
add t0, g, h // temp t0 = g + h
add t1, i, j // temp t1 = i + j
sub f, t0, t1 // f = t0 - t1
```

---

#### RISC-V Registers

<p align="center"><img src="../../assets/images/041701.jpg" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

---

##### Register Operand Example

C code:
```c
f = (g+h) - (i+j);
```
- f, ..., j in x19, x20, ..., x23

Compiled된 RISC-V code:
```
add x5, x20, x21
add x6, x22, x23
sub x19, x5, x6
```

---

#### Memory Operands

- Main memory used for composite data
  - Arrays, structures, dynamic data

- 연산 진행하려면
  - Memory에서 register로 Load
  - Register에서 memory로 store

- RISC-V : Little Endian
  - Little Endian 0111 => LSB가 마지막 bit에
  - Big Endian 1110 => LSB가 첫번째 bit에

---

##### Memory Operand Example

C code:
```c
A[12] = h + A[8];
```
- h in x21, base address of A in x22

Compiled RISV-C code:
```
ld x9, 64(x22)
add x9, x21, x9
sd x9, 96(x22)
```

<p align="center"><img src="../../assets/images/041702.jpg" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>


Register안에 값으로 Base address를 나타냅니다. 

---

#### Registers vs. Memory 

- Register가 Memory를 access하는 것보다 빠릅니다.
- Memory data는 load와 store를 동반하기 때문에 instruction이 더 많이 필요합니다.
- Compiler는 register를 가능한 많이 사용해야합니다. 
  - 즉 Register 최적화가 중요합니다.

---

#### Immediate Operands

- Constant data specified in an instruction
  - addi x22, x22, 4
  - memory access가 없기 때문에 속도 향상됩니다.

- 많이 나오는 case에서 빨라집니다. load instruction이 필요 없기 때문입니다. 

---

#### RISC-V R-format Instructions

|funct7|rs2|rs1|funct3|rd|opcode|
|7bits|5bits|5bits|3bits|5bits|7bits|

- Instruction fields
  - opcode: operation code
  - rd: destination register number
  - funct3: 3-bit function code (additional opcode)
  - rs1: the first source register number
  - rs2: the second source register number
  - funct7: 7-bit function code (additional opcode)

---

##### R-format Example

add x9, x20, x21

|0|21|20|0|9|51|
|0000000|10101|10100|000|01001|0110011|

- 0000 0001 0101 1010 0000 0100 1011 0011 = 015A04B3

---

#### RISC-V I-format Instructions

|immediate|rs1|funct3|rd|opcode|
|12bits|5bits|3bits|5bits|7bits|

- register 한 개를 immediate로 대체함
  - rs1: source or base address register number
  - immediate: constant operand, or offset added to base address
    - 2s-complement, sign extended

--- 

#### RISC-V S-format Instructions

|imm[11:5]|rs2|rs1|funct3|imm[4:0]|opcode|
|7bits|5bits|5bits|3bits|5bits|7bits|

- Different immediate format for store instructions
  - rs1: base address register number
  - rs2: source operand register number
  - immediate: offset added to base address
    - Split so that rs1 and rs2 fields always in the same place


---

#### Shift Operations

|funct6|immed|rs1|funct3|rd|opcode|
|6bits|6bits|5bits|3bits|5bits|7bits|

- immed: how many positions to shift
- Shift left logical
  - Shift left and fill with 0 bits
  - slli by i bits multiplies by $2^i$
- Shift right logical
  - Shift right and fill with 0 bits
  - srli by i bits divides by $2^i$ (unsigned only)

---

#### And Operations

- Useful to mask bits in a word
  - Select some bits, clear others to 0

```
and x9, x10, x11
```

<p align="center"><img src="../../assets/images/041703.jpg" width="600px" height="600px" title="OP code 예시" alt="OP code" ><img></p>

---

#### Or Operations

- Useful to include bits in a word
  - Set some bits to 1, leave others unchanged

```
or x9, x10, x11
```

<p align="center"><img src="../../assets/images/041704.jpg" width="600px" height="600px" title="OP code 예시" alt="OP code" ><img></p>

---

#### Xor Operations

- Differencing operation
  - Set some bits to 1, leave others unchanged

```
xor x9, x10, x12 // Not operation
```

<p align="center"><img src="../../assets/images/041705.jpg" width="600px" height="600px" title="OP code 예시" alt="OP code" ><img></p>

---

#### Conditional Operations

- Branch to a labeled instruction if a condition is true
  - Otherwise, continue sequentially 

- beq rs1, rs2, l1
  - if (rs1 == rs2) branch to instruction labeled l1

- bne rs1, rs2, l1
  - if (rs1 != rs2) branch to instruction labeled l1

---

#### Compiling If Statements

C code:
```c
if (i==j) f=g+h;
else f = g-h;
```
- f, g, ... in x19, x20, ...

Compiled된 RISC-V code:
```
      bne x22, x23, else
      add x19, x20, x21
      beq x0, x0, Exit // unconditional 
Else: sub x10, x20, x21 
Exit: ...
```

---

#### Compiling Loop Statements

C code:
```c
while (save[i]==k) i += 1;
```
- i in x22, k in x24, address of save in x25

Compiled된 RISC-V code:
```
Loop: slli x10, x22, 3
      add  x10, x10, x25
      ld   x9,  0(x10)
      bne  x9,  x24, Exit
      addi x22, x22, 1
      beq  x0,  x0, Loop
Exit: ... 
```

8 byte memory이기 때문에 x25 + 8i로 해야합니다. 따라서 i를 3-bit left shift시켜주고 이를 x25에 더해서 x10에 저장합니다. 

---

#### More Conditional Operations

- blt rs1, rs2, l1
  - if (rs1<rs2) branch to instruction labeled l1

- bge rs1, rs2, l1
  - if (rs1>=rs2) branch to instruction labeled l1

- Example
  - if (a>b) a+=1;
  - a in x22, b in x23
    - bge x23, x22, Exit //branch if b>=a
    - addi x22, x22, 1

---

#### Branch Addressing

- Branch instructions specify 
  - opcode, two registers, target address

- Most branch targets are near branch
  - Forward or backward

- SB format

|imm[12]|imm[10:5]|rs2|rs1|funct3|imm[4:1]|imm[11]|opcode|

- PC-relative addressing
  - Target address = PC + immediate  2 
  - x 2인 이유는 호환성을 위해서 합니다.
  - RISC-V half word 단위에도 대응할 수 있게 끔 x2를 합니다.

---

#### Base, Offset을 두는 이유

- Base register 64bit인데 이것을 address로 쓰면 $2^64$만큼의 address를 표현할 수 있게 되기 때문에 base와 offset을 사용합니다.

---

#### Fallacies (오해)

- Powerful instruction => higher performance
  - Fewer instructions required
  - But complex instructions are hard to implement
    - May slow down all instructions, including simple onew
  - Compilers are good at making fast code from simple instructions

- Use assembly code for high performance
  - But modern compilers are better at dealing with modern processors
  - More lines of code => more errors and less productivity 

- Backward compatibility => instruction set doesn't change
  - But they do accrete more instructions
  - instruction 개수가 년마다 점차 늘어나고 있음

---

### 퀴즈

1) 프로세서에서 더 powerful한 (complex한) instruction을 제공할수록, 프로그램에 필요한 instruction 수가 감소하므로 연산 성능이 더 향상된다. <font color="red">X, instruction이 감소된다고 연산 성능이 향상되는 것은 아님, instruction cycle이 중요함</font>

2) RISC-V의 branch instruction (SB format)에서 immediate 주소값의 0번째 bit (imm[0])을 사용하지 않는 이유는 branch 할 수 있는 range를 늘리기 위해서이다. <font color="red">O, bit 하나를 빼서 x2를 함</font>

3) 아래의 instruction들에 대해 각각의 비어있는 항목에 들어갈 binary 값을 채우시오. 

<p align="center"><img src="../../assets/images/041706.jpg" width="600px" height="600px" title="OP code 예시" alt="OP code" ><img></p>

4) 아래 코드가 수행된 후 오른쪽 register와 memory의 각 공간에서 최종적으로 변화된 값을 표시하시오.

(1) sub x5, x20, x21
    add x24, x22, x5

(2) ld x6, 16(x23)
    add x6, x19, x6
    sd x6, 32(x23)

(3) addi x18, x18, 4

<p align="center"><img src="../../assets/images/041707.jpg" width="600px" height="600px" title="OP code 예시" alt="OP code" ><img></p>

---

### 추가설명 
