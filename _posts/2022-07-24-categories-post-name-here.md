---
title: "[💻] 컴퓨터 구조"
excerpt: "컴퓨터 구조 과목에서 들은 내용 정리"

categories:
  - 컴퓨터 구조
tags:
  - [과목]

permalink: /categories1/post-name-here/

toc: true
toc_sticky: true

date: 2022-10-28
last_modified_at: 2022-10-28
---

##  컴퓨터 구조 (1)

컴퓨터를 만들기 위해서는 instruction code가 필요합니다.

Compiler가 있는 이유는 명령어를 해석하기 위해서 존재하는 것이고

microoperation으로 정의된 컴퓨터 명령어는 레지스터에 data와 함께 저장되어 수행됩니다.

### Instruction

1) 컴퓨터를 위한 microopeartions의 binary 집합체

2) 메모리에 데이터와 함께 저장되는 Instruction code

3) Operaction Code (OP code)
  - ADD나 SUB처럼 opeartion을 정의하는 bit
  - 적어도 n bit라 하면 2^n 의 distinct한 명령어가 생성됩니다.
  - control unit은 메모리로부터 instruction을 받아내고 OP code를 해석하는데 이후 control signal로 내보내는 역할을 담당합니다.

4) Instruction은 레지스터에 저장되어 있는 데이터를 통해 미리 형성함 -> 즉, Instruction은 OP code 뿐만 아니라 레지스터, 메모리 주소까지 정의할 수 있습니다.

### Instruction Codes

<!--
![image.jpg](../../assets/images/102801.jpg/)
-->
<img src="../../assets/images/102801.jpg" width="450px" height="300px" title="OP code 예시" alt="OP code"><img><br/>


OP code는 4 bit, Address는 12 bit로 할당됩니다. 

위 OP code와 Address를 가지고 명령어를 구현할 수 있습니다.

Processor Register (Accumulator or AC) : 연산하는데 대부분을 담당하고, AC를 통하여 모든 데이터가 들어갈 수 있습니다.

- Immediate operand
- Direct address
- Indirect address
- Effective address : the actual address of operand

I flag - 1 bit, op code - 3 bit, address - 12 bit 로 정의하여 실행합니다.

**I = 0 일 때 direct, I = 1 일 때 indirect**

**indirect의 장단점**

장점 : 2^12 4k -> 2^16 64k로 메모리가 access 할 수 있는 주소를 키웁니다.

단점 : Flip-Flop, clk이 있어 속도가 느립니다.

### Instruction Code 예시

<img src="../../assets/images/102802.jpg" width="450px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

위 그림을 보면 I flag에 따라 Direct와 Indirect가 나뉘고 Indirect는 16bit의 address를 표현할 수 있는 것으로 보여집니다.

**무조건 적으로 Indirect가 좋다고 말은 못합니다!! 속도의 문제가 있어요**

위 그림에서 보았을 때, 그림처럼 memory에 할당이 필요한 것처럼 보여집니다. 메모리 할당은 운영체제가 할 것이니 걱정안하셔도 됩니다.

### Computer Registers

<img src="../../assets/images/102803.jpg" width="450px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

Computer를 구성하는 Register은 다양한데 그 몇가지를 소개해드리자면,

1) PC (Program Counter)
- Holds address of instruction
- 다음 명령어 수행될 곳의 주소를 저장합니다.
<br><br>

2) AR (Address Register)
- Holds address for memory
- 메모리 주소를 저장합니다.
<br><br>


3) IR (Instruction Register)
- Holds instruction code
- 명령어 code를 저장합니다.
<br><br>


4) TR (Temporary Register)
- Holds temporary data
- 일시적으로 저장하는 레지스터인데 아무용도로 사용 가능합니다.
<br><br>


5) DR (Data Register)
- Holds memory operand
- 메모리 operand를 저장합니다.
<br><br>


6) OUTR (Output Register), INPR (Input Register)
- Hold input character, output character
- 외부에서 들어올 때나 나갈 때 사용하는 레지스터입니다.
<br><br>


7) Accumulator (Processor Register)
- 중요한 레지스터!
- 메모리 주소를 physical하게 가리켜줍니다.
<br><br>


<img src="../../assets/images/102804.jpg" width="450px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

위 그림은 Bus에 연결된 register를 그린 것인데, register의 데이터를 사용할 때 마다 S0, S1, S2를 변경해가며 Bus를 열고 닫습니다.

(S0, S1, S2) = (1,1,1) 이면 Memory unit의 버스가 열리게 되는 것이죠.

이렇게 Bus를 사용하여 clk 마다 순차적으로 명령어를 실행하고 레지스터에 저장합니다.

### 예시

이제 예시를 자세히 들어볼게요.

<img src="../../assets/images/102805.jpg" width="450px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

위 그림처럼 메모리와 레지스터가 있다고 예를 들어봅시다. 그림에서는 PC가 없는데 PC가 35라고 가정해볼게요.

일단 IR에서는 메모리에 있는 명령어를 가져오기 때문에 0 ADD 457을 그대로 가져옵니다.

IR에서 I flag가 0인 것으로 보아 Direct address인 것을 볼 수 있네요. IR의 address가 457인 것으로 보아 457를 AR에 저장하고 457 주소에 있는 값을 DR에 저장하게 됩니다. 

<img src="../../assets/images/102806.jpg" width="450px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

즉 위와 같은 그림이 완성되는 것이죠. 이 때 IR과 AR, DR을 load하는 과정이 반드시 필요합니다. 사용할 때마다 Bus를 열고 닫아야하는데 Bus를 사용하기 위한 S0, S1, S2는 위 그림과 같습니다.

<img src="../../assets/images/102807.jpg" width="450px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

다른 예시도 들어볼게요. 이제 I flag는 1입니다. I flag가 1이면 indirect로 좀 더 복잡한 과정에 들어가게 됩니다. 

35번지에 있는 명령어를 가져오고 그것을 IR에 저장합니다. IR의 address는 457번지로 457에 들어가면 900의 값이 나오게 됩니다. 하지만 indirect address방법이므로 900번지의 주소로 다시 들어가 그 주소의 값을 최종적으로 DR에 저장하게 됩니다. 

즉 AR은 35->457->900이라는 과정을 거치게 되는 것이죠.

Bus도 여는 횟수가 많아집니다. 확실히 direct보다 손이 가는게 많네요. 속도가 느려지는 이유도 진행과정이 많아지기 때문인 것이 느껴집니다.

## Instruction cycle

- Fetch an instruction from memory
- Decode the instruction
- Read the effective address from memory (if indirect address)
- Execute the instruction
- (Store the result)

위와 같은 cycle로 명령어를 수행합니다.

1) Initially

- PC is loaded with the address of the first address of instruction in the program

- SC <- 0 (providing T0)

- After each clock, SC incremented

2) Fetch and decode phases

- T0 : AR <- PC, SC <- SC + 1
- T1 : IR <- M[AR], PC <- PC + 1, SC <- SC + 1
- T2 : D0-D7 <- Decode IR(12-14), AR <- IR(0-11), I <- IR(15), SC <- SC + 1
- (no branch assumed)

`즉, SC에 따라서 동작하는 게 달라지게 됩니다.`

<img src="../../assets/images/103101.jpg" width="450px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

T0 T1 T2 때 OR 게이트를 사용하여 원하는 Bus를 열고 닫습니다.

아래는 flowchart 인데 다음과 같은 logic으로 구동되는 것을 볼 수 있어요.

<img src="../../assets/images/103102.jpg" width="450px" height="300px" title="OP code 예시" alt="OP code"><img><br/>