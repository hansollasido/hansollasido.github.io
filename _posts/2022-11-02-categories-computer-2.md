---
title: "[💻] 컴퓨터 구조 (2)"
excerpt: "컴퓨터 구조 과목에서 들은 내용 정리 2"

categories:
  - 컴퓨터 구조
tags:
  - [과목]

permalink: /categories1/computer-2/

toc: true
toc_sticky: true

date: 2022-11-02
last_modified_at: 2022-11-02
---

##  컴퓨터 구조 (2)

이전에 이어서 하겠습니다.

메모리에 다음과 같이 데이터가 있다고 가정합시다.

<img src="../../assets/images/110201.jpg" width="300px" height="200px" title="OP code 예시" alt="OP code"><img><br/>

순서는 다음과 같습니다

- D7 = 0 & I = 1 (indirect memory reference instruction)
  - T0 : AR <- PC
  - T1 : IR <- M[AR], PC <- PC + 1
  - T2 : Decode IR(12-14), AR <- IR(0-11), I <- IR(15)
  - T3 : AR <- M[AR]

- D7 = 0 & I = 0 (direct memory reference instruction)
  - T0 : AR <- PC
  - T1 : IR <- M[AR], PC <- PC + 1
  - T2 : Decode IR(12-14), AR <- IR(0-11), I <- IR(15)
  - T3 : Nothing
  - (T4 - : depend on OP code)

- D7 = 1 & (I = 0 !! I = 1) (register reference or I/O instruction)
  - T0 : AR <- PC
  - T1 : IR <- M[AR], PC <- PC + 1
  - T2 : Decode IR(12-14), AR <- IR(0-11), I <- IR(15)
  - T3 : Execution

47번에 있는 메모리의 값은 0 ADD 457 입니다.

D7 = 0 이라 할 때 I = 0이기 때문에 direct memory reference instruction 방법을 사용합니다.

그럼 T0 부터 시작하겠습니다..

T0 일 때 AR에 PC값이 들어갑니다.

PC값이 47이기 때문에 AR에 47값이 들어갑니다.

이후 SC는 1 증가하고, T1에 들어갑니다. T1는 IR에 AR번지의 memory값을 저장하고 PC를 1 증가시킵니다.

T2이 끝나면 SC는 1증가하여 T3이 되고 T3은 direct 이기 때문에 아무것도 안하게 됩니다. 

또한 D7도 0이라 뒀기 때문에 T4에서도 아무것도 진행하지 않습니다.

이런식으로 SC에 따라서 레지스터 값을 변경하고 메모리에 있는 명령어들을 수행하게 합니다. 

<img src="../../assets/images/110202.jpg" width="300px" height="200px" title="OP code 예시" alt="OP code"><img><br/>

3 - bit 짜리 op code를 decode하고 이에 맞는 SC에 따라 명령을 수행하게 되는데요.

특히 D7I'T3은 r, IR(i) = B_i로 따로 정의하고 명령어를 수행하게 됩니다. 

<img src="../../assets/images/110203.jpg" width="300px" height="200px" title="OP code 예시" alt="OP code"><img><br/>

각 명령어에 따라 rB_i를 수행하게 됩니다. 이 때 구현을 NAND 게이트로 구현하며 원하는 명령어를 수행할 수 있게 됩니다. 

Memory-Reference Instructions

| Symbol | Operation decoder | Symbolic desccription |
|---|---|---|
| AND | D0 | AC <- AC ^ M[AR] |
| ADD | D1 | AC <- AC + M[AR], E <- Cout |
| LDA | D2 | A <- M[AR] |
| STA | D3 | M[AR] <- AC |
| BUN | D4 | PC <- AR |
| BSA | D5 | M[AR] <- PC, PC <- AR + 1 |
| ISZ | D6 | M[AR] <- M[AR] + 1, If M[AR] + 1 = 0 then PC <- PC + 1 |

AND (0000) : D0
- D0 T4 : DR <- M[AR]
- D0 T5 : AC <- AC ^ DR, SC <- 0

ADD (0001) : D1
- D1 T4 : DR <- M[AR]
- D1 T5 : AC <- AC + DR, E <- Cout, SC <- 0

LDA (0010) : D2
- D2 T4 : DR <- M[AR]
- D2 T5 : AC <- DR, SC <- 0

STA : Store AC (0011) : D3
- D3 T4 : M[AR] <- AC, SC <- 0

**BUN : Branch Unconditionally (0100) : D4**
- D4 T4 : PC <- AR, SC <- 0

BSA : Branch and Save Return Address (0101) : D5
- D5 T4 : M[AR] <- PC, AR <- AR + 1
- D5 T5 : PC <- AR, SC <- 0
  - subroutine call 하는데에 유용함

ISA : Increment and Skip if Zero (0110) : D6
- D6 T4 : DR <- M[AR]
- D6 T5 : DR <- DR + 1
- D6 T6 : M[AR] <- DR, SC <- 0, if (DR=0) then PC <- PC + 1

<img src="../../assets/images/110204.jpg" width="300px" height="200px" title="OP code 예시" alt="OP code"><img><br/>

PC는 21을 가리키기 때문에 21번지 memory 명령어부터 수행하게 됩니다. 

D7 = 0, I = 0 임을 확인한 뒤 시작하겠습니다.

- T0 : AR <- PC
- T1 : IR <- M[AR], PC <- PC + 1
- T2 : Decode I(12-14), AR <- IR(0-11), I <- IR(15)
- T3 : nothing
- D0 T4 : DR <- M[AR]
- D0 T5 : AC <- AC ^ DR, SC <- 0

위 과정을 거칩니다.

이에 레지스터 결과는 다음과 같이 나오게 됩니다.

<img src="../../assets/images/110205.jpg" width="500px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

SC 가 0이 되었기 때문에 다시 T0로 돌아가게 됩니다.

T0부터 다시 실행합니다.

- T0 일 때 AR <- PC
- T1 일 때 IR <- M[AR], PC <- PC + 1
- T2 일 때 Decode IR(12-14), AR <- IR(0-11), I <- IR(15)
- T3 일 때 AR <- M[AR]
- D1 T4 : DR <- M[AR]
- D1 T5 : AC <- AC + DR, E <- Cout, SC <- 0

이런식으로 메모리 전 과정을 거칩니다. 

각각의 SC 따라 해당하는 명령어를 입력하고 그에 맞도록 레지스터의 값들을 변경하여 컴퓨터를 조작합니다.

(LDA와 STA는 생략하겠습니다.)

BUN과 BSA, ISZ는 다음과 같습니다. 

<img src="../../assets/images/110206.jpg" width="300px" height="200px" title="OP code 예시" alt="OP code"><img><br/>

다음과 같은 메모리가 있고

<img src="../../assets/images/110207.jpg" width="300px" height="200px" title="OP code 예시" alt="OP code"><img><br/>

다음과 같이 초기 레지스터값이 있다면 연산은 다음과 같이 진행합니다.

- T0 : AR <- PC
- T1 : IR <- M[AR], PC <- PC + 1
- T2 : Decode I(12-14), AR <- IR(0-11), I <- IR(15)
- T3 : nothing

**(여기까지는 같습니다.)**

- D4 T4 : PC <- AR, SC <- 0

**D4 T4 일 때 AR값을 PC에 넣어서 subroutine으로 갈 수 있게 끔 만들어줍니다**

<img src="../../assets/images/110208.jpg" width="300px" height="200px" title="OP code 예시" alt="OP code"><img><br/>

Branch Uncoditionally라는 말과 같게 무조건적으로 subroutine call을 진행하게 되는데 BUN을 진행하면 위와 같이 PC가 설정되고 

원래 같으면 37을 수행해야하지만, 70을 수행하게 됩니다. 

<img src="../../assets/images/110209.jpg" width="300px" height="200px" title="OP code 예시" alt="OP code"><img><br/>

BUN을 진행하여 70번지의 명령어를 실행한 뒤 레지스터 결과는 위와 같습니다.

BSA는 Return address를 저장하는 것에 BUN과 큰 차이점이 있습니다. 

Subroutine call을 진행한 다음 return address로 돌아가게 됩니다. 

마지막 300번지에 1 BUN 200이 있네요. 이 과정을 거치면 return address로 돌아가서 진행할 수 있습니다.

<img src="../../assets/images/110210.jpg" width="300px" height="200px" title="OP code 예시" alt="OP code"><img><br/>

return address로 다시 돌아가 명령어를 수행할 수 있다는 것이죠.

return address가 있다는 점에서 BUN과 큰 차이점이 있습니다.

ISZ는 값을 1 씩 증가시키는데 0이 되면 skip하는 명령어 입니다. 

<img src="../../assets/images/110211.jpg" width="500px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

다음은 Interrupt에 대해 설명해드리겠습니다.