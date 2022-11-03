---
title: "[🐰] 컴퓨터 구조 (4)"
excerpt: "컴퓨터 구조 과목에서 들은 내용 정리 4"

categories:
  - 컴퓨터 구조
tags:
  - [과목]

permalink: /categories1/computer-4/

toc: true
toc_sticky: true

date: 2022-11-03
last_modified_at: 2022-11-03
---

##  컴퓨터 구조 (4)

### Instruction

- Computer system은 HW와 SW의 조화로 이뤄짐
- SW : computer HW의 상세한 지식 없이도 컴퓨터 SW를 다루기 가능함
  - 하지만 HW & SW 개발자는 성능 향상을 위해 둘다 디테일한 지식을 요구함
- Machine instruction은 어려움 
  - 사용자 중심인 symbolic program을 machine instructions으로 해석해야함
- HW dependent 일 수 있고 아닐 수도 있음
  - C: Independent, Translator(compiler): Dependent

### Machine Language

- 컴퓨터는 2진수형태로 프로그램을 실행함
- 다른 타입으로 프로그램이 쓰여졌으면 반드시 해석이 되어야 함
- Program types
  - Binary code
  - Octal, Hexadecimal code : same as Binary code
  - Symbolic code - assembler language program : Assemler translates symbolic code into binary code
  - High level program languages : Compiler translates High level language program to binary

- Symbolic program (Assembly language program)
  - Easy to handle
  - One step further : Table 6-5 (Pseudo code: ORG, DEC, END)


<img src="../../assets/images/110301.jpg" width="500px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

<img src="../../assets/images/110302.jpg" width="500px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

- Machine language program
  - Binary code program : Table 6-2
  - Hexa code program : Table 6-3

- High lebel programming language
  - example : C, FORTRAN, PASCAL, Web, .....

<img src="../../assets/images/110303.jpg" width="500px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

Location 0 은 LDA 4, 1은 ADD 5, 10은 STA 6을 의미합니다.

<img src="../../assets/images/110304.jpg" width="500px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

Table 6-2를 16진수로 만들면 Table 6-3과 같습니다.

### Assembly Language

**Translation to Binary**
- Assembler는 binary로 Translation해주는 역할을 수행합니다.
- Pseudo-instruction은 해석하지 않습니다.

예를 들어 C 코드로

```c
integer j, sum, A(100);
sum = 0;
for (j=0; j<= 100; j++){
  sum = sum + A(j);
}
```

위 코드를 compiler가 translates하여 binary code로, 혹은 assembler language로 바꿔 줍니다.

<img src="../../assets/images/110305.jpg" width="500px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

### Subroutines

컴퓨터가 subroutine으로 들어갈 때 다양한 방법으로 return address를 저장하곤 합니다. 

<img src="../../assets/images/110306.jpg" width="500px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

BSA와 BUN으로 subroutine을 진행한 다음 main program으로 돌아가는 것을 볼 수 있습니다.

컴퓨터에서는 define하는 부분이 뒤에 있다는 점에 유의해야 합니다.