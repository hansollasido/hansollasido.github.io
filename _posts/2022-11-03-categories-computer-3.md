---
title: "[💪] 컴퓨터 구조 (3)"
excerpt: "컴퓨터 구조 과목에서 들은 내용 정리 3"

categories:
  - 컴퓨터 구조
tags:
  - [과목]

permalink: /categories1/computer-3/

toc: true
toc_sticky: true

date: 2022-11-03
last_modified_at: 2022-11-03
---

##  컴퓨터 구조 (3)

### I/O and Interrupt 

**사진이 약간 삐뚤빼뚤 거리네요...ㅎㅎ**

For a computer to communicate with the external devices, I/O instruction must be included in a instruction set.

-> 즉, 외부 디바이스와 소통하기 위해서는 instruction set에 I/O 관련 명령어가 포함되어야 합니다.

이 때 필요한 register가 INPR과 OUTR 입니다. 

<img src="../../assets/images/110212.jpg" width="500px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

FGI & FGO가 control flag로 사용되는데

- FGI = 1 이면 새로운 정보가 들어올 수 있다는 뜻
- FGI = 0 이면 컴퓨터가 그 정보를 받아들일 수 있다는 뜻
- FGI가 1일 경우 새로운 정보가 들어오지 않음
- 만일 컴퓨터가 FGI를 확인했을 때 1임을 보았다면, INPR 값을 AC에 넣고 FGI를 clear 시킴
- FGO = 1 이면 컴퓨터가 AC값을 OUTR에 옮기고 FGO를 0으로 clear 시킴
- FGO = 0 이면 OUTR의 값이 프린터에 옮겨지기 때문에 컴퓨터가 OUTR에 새로운 정보를 입력 못함

이런식으로 FGI와 FGO를 통해 외부 디바이스와 communication이 가능합니다.

I/O instruction은 T3에서 D7 = 1, I = 1 일 때 실행 됩니다.

interrupt는 지금 하고 있는 일보다 더 중요한 일을 해야할 때, interrupt를 줘서 하고 있는 일을 그만두고 더 중요한 일을 하게 만듭니다.

ION : Interrupt enable on (1111 0000 1000 0000) ; B7
  - I D7 T3 B7 : IEN <- 1, SC <- 0

IOF : Interrupt enable off (1111 0000 0100 0000) ; B6
  - I D7 T3 B6 : IEN <- 0, SC <- 0

(다른 interrupt는 생략하겠습니다.)

interrupt는 굉장히 중요한 메커니즘으로 interrupt를 set하고 reset하는 역할을 ION과 IOF가 담당합니다.

<img src="../../assets/images/110213.jpg" width="500px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

- R = 0 일 때, instruction cycle을 진행
- IEN을 확인하고 
- R = 1 이면 interrupt cycle에 들어가서 진행
  - return address를 메모리 0번에 저자
  - PC <- 1
  - IEN <- 0, R <- 0

interrupt가 발생했을 때 위 flowchart처럼 진행하게 되어 좀 더 중요한 일을 할 수 있게 만듭니다. 

이 때 subroutine으로 들어가 진행했다가 다시 return address로 돌아가야합니다.

<img src="../../assets/images/110214.jpg" width="500px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

위 사진을 보자면 Main program을 진행하던 중 interrupt가 발생하게 되면 메모리 0번지에 return address를 저장하고 pc=1이 됩니다. 이 때 해당하는 intterrupt service routine에 들어가 진행하게 되며 interrupt cycle이 끝난 다음에는 다시 main program에 들어가게 됩니다. 

특징이 있다면...

- R can be set except T0, T1, T2
  - T0' T1' T2' (IEN)(FGI + FGO) : R <- 1
  - If one of FGI and FGP == 1 then an interrupt occurs

- Modification of fetch and decode phase
  - T0 -> R'T0, T1 - R'T1, T2 -> R'T2

- Interrupt cycle
  - RT0 : AR <- 0, TR< - PC
  - RT1 : M[AR] <- TR, PC <- 0
  - RT2 : PC <- PC + 1, IEN <- 0, R <- 0, SC <- 0

예시를 들어보겠습니다.

<img src="../../assets/images/110215.jpg" width="500px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

위 그림에서 memory 33을 진행하고 있다가 interrupt가 발생하게 되면 R=1이 되면서 SC <- 0 이 될 때, R = 1 인지 확인하고 interrupt cycle에 들어갑니다.

0번지에 return address인 34을 넣고 1번을 수행하여 subroutine을 진행합니다. 

이 때 TR은 temporary register로 어느 값이나 넣을 수 있습니다. 여기서는 return address를 저장하기 위한 용도로 사용되었습니다. 

---
## 결론

이 내용들은 컴퓨터구조 수업 강의노트 5장에 있는 내용들인데, control logic으로 컴퓨터를 제어하는 방법에 대해 알 수 있었습니다. 

7장에서는 control memory 방법을 사용해서 5장과 다르게 컴퓨터를 구현합니다. 이제야 5장이 끝나니 가뿐해지네요.