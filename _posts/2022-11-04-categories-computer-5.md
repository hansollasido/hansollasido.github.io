---
title: "[🦋] 컴퓨터 구조 (5)"
excerpt: "컴퓨터 구조 과목에서 들은 내용 정리 5"

categories:
  - 컴퓨터 구조
tags:
  - [과목]

permalink: /categories1/computer-5/

toc: true
toc_sticky: true

date: 2022-11-04
last_modified_at: 2022-11-04
---

##  컴퓨터 구조 (5)

### Micro - programmed control

이 장에서는 control logic을 없애고 미리 계산한 값을 메모리에 넣어서 계산합니다.

- The function of the control unit
  - controls sequences of microoperations
  - different instructions -> different microoperations
<br><br/>

- Hardwired vs Microprogrammed
  - binary information (called "Control Word")
  - Microinstruction
  - Microprogram
<br><br/>

- Control words are stored in "Control Memory"
  - Control memory is usually ROM (fixed during fabrication)
  - Microprogrammed Control Unit has Control Memory
  - A Microprogrammed computer has a main memory and a control memory
<br><br/>

- Dynamic microprogramming
  - A microprogram is loaded from auxiliary memory
<br><br/>

즉, 이미 만들어져 있는 것을 Conrol memory에 넣고 꺼내서 쓴다는 점에서 전 장이랑 다릅니다.

- Hardware-wired control : 5장
- Micro-programmed control : 7장 

-> Control signal 만드는 차이

- Control Address Register (CAR) 이 control memory에서 micro operation의 주소를 담당합니다.
<br><br/>
- Control Data Register (CDR) 이 메모리로 부터 읽어온 micro-operation을 저장합니다.
<br><br/>
- 한 명령어가 실행되면 그 다음 명령어가 결정됩니다.
<br><br/>

제어할 수 있는 control signal이 있어야 하며, Fetch and Decode를 위한 Signal이 필요하게 됩니다.

### Address sequencing - mapping of instruction

**Mapping process**
- instruction code로 부터 micro-program 주소를 실행시킵니다.
- ROM, PLD를 통해 좀 더 복잡한 Mapping이 가능합니다

OP code가 4bit이면 16개의 명령어를 사용할 수 있습니다.

CM 128 words는 address하기 위하여 7bits가 필요합니다.

이에 4 bit op code를 address로 mapping하는 과정이 반드시 필요합니다.

1000000 부터 1111111 까지 사용할 수 있습니다.

### Address sequencing - subroutine

- A common routine could be a subroutine to save control memory size.

- Need to save return address.

- Many ways to implement saving address.

- FIFO is one of best ways.

5장과는 다르게 7장에서는 IR 이 없다는 것이 특징입니다.


### Microprogram example - micro-instruction format

F1, F2, F3 : Microoperations fields

CD : Condition for branching

BR : Branch field

AD : Address field

5장과 기본 동작은 같습니다. (PC를 AR로 보내는 것 등등)

F1, F2, F3는 제어 시그널입니다.

CD와 BR, AD는 control memory의 주소를 정할 때 사용합니다.

이 메모리의 특징은

1) mapping function

2) IR이 없음

3) 무조건 DR로 operand가 감

이라는 점입니다. 

5장에서 배운 instruction format과 7장에서 배운 micro-instruction format에 대하여 둘 다 다룰 줄 알아야 합니다.

F1, F2, F3의 제어 시그널은 다음과 같습니다.

<img src="../../assets/images/110501.jpg" width="500px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

이러한 이름들을 보고 program을 예측합니다.

3개의 Microoperation Fields가 있는데
- CD : status bit condition (2 bits)
- BR : type of branches (2 bits)
- AD : address bits (7 bits -> 128 control words)

입니다.

F1F2F3 가 예를 들어 000100101 이면
- DR <- M[AR] and PC <- PC + 1 
입니다.
<br><br/>

만일 Symbol 이름이 예를 들어 아래와 같다면

Symbol example(DRTAC)
- "DR to AC"

입니다. symbol 이름을 가지고 예측이 가능합니다.

CD와 BR은 아래와 같습니다.

<img src="../../assets/images/110502.jpg" width="500px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

CD는 조건을 설정해주고 BR은 CD에 따라 명령어를 수행하게 됩니다. 

조건에 따라 실행되는 것이 다르게 됩니다. 

**symbolic microinstructions**

- Address field may be one of the following three options
  - label
  - NEXT
  - BR with RET or MAP has empty address field (later filled with zero by assembler)
<br><br/>

- ORG
  - defines the first address of a micro-operation routine
  - not included in the execution code (called "pseudo-instruction")
<br><br/>

ORG로 어느 주소에서 시작할 것인지 정의를 해줍니다. pseudo-instruction이기 때문에 진짜 명령어는 아닙니다.

### Microprogram example - fetch routine
- AR <- PC
- DR <- M[AR], PC <- PC + 1 (// IR was used for hardwired control)
- AR <- DR(0-10), CAR(2-5) <- DR(11-14), CAR(0,1,6) <- 0
- !! Starts from 64 !!

위 명령어를 translate 해보겠습니다.

        ORG 64
FETCH : PCTAR       U JMP NEXT
        READ, INCPC U JMP NEXT
        DRTAR       U MAP

위와 같이 FETCH로 Translate가 가능합니다. 

NEXT는 현 주소에서 +1을 해줍니다. 

7장에서는 CAR이 있는데 Control memory의 AR 역할을 맡고 있습니다. 


<!-- <img src="../../assets/images/110306.jpg" width="500px" height="300px" title="OP code 예시" alt="OP code"><img><br/> -->

