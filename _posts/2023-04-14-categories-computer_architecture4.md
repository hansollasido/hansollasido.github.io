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

### 추가설명
