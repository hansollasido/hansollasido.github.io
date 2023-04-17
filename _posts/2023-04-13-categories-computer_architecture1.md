---
title: "[컴퓨터구조설계 및 응용 1]"
excerpt: "Introduction & Performance"

categories:
  - 컴퓨터구조설계
tags:
  - [수업]

permalink: /categories10/computer_architecture1/

use_math: true
toc: true
toc_sticky: true

date: 2023-04-13
last_modified_at: 2023-04-13
---

# 컴퓨터구조설계 및 응용

수업을 듣고 정리한 페이지 입니다.

---

#### Below your program

**Application software**
- high-level language

**System software**
- compiler: translates HLL(High level language) code to machine code
- Operating system: service code
  - Handling input/output
  - Managing memory and storage
  - Scheduling tasks & sharing resources

**Hardware**
  - Processor, memory, I/O controllers

운영체제(OS)는 HW 자원 관리하는 시스템으로, input/output, memory와 저장소, 자원 등을 관리하는 시스템입니다. 예시로 Window와 Linux가 있습니다. 

---

#### Levels of Program Code

High-level language $\rightarrow$ **Compiler** $\rightarrow$ Assembly language $\rightarrow$ **Assembler** $\rightarrow$ Binary machine language


HLL을 Compiler가 컴파일하여 LLL(Low Level Language)로 바꿔줍니다. Low Level Language는 instruction을 text로 표현한 것으로 사람이 읽을 수 있는 수준의 언어입니다. 이 LLL을 Assembler가 Machine Language로 바꿔주면은 비로소 0과 1로 구성된 Language가 만들어지고 이를 컴퓨터가 읽습니다. 컴퓨터는 0과 1만 읽을 수 있고 사람은 0과 1로만 구성된 언어를 읽지 못하다보니 이와 같은 해석 과정이 필요합니다. 

---

### 성능 지표 정의

성능이 뛰어나다는 것을 어떻게 정의할 수 있을까요? 단순히 빠른 것으로 정의될까요?

#### Response Time and Throughput

**Response Time**은 task를 얼마만에 끝내는지에 대한 성능 지표입니다. 그와 달리 **Throughput**은 단일시간 동안 얼마나 많은 task를 하는지에 대한 성능 지표입니다. 보통 Response time과 throughtput은 서로 반비례 관계이지만 항상 그렇지만은 않습니다. 

Response time은 짧을수록 좋고 Throughput은 클수록 좋습니다. 이에 X와 Y의 성능을 비교한다면

<center>$\mathrm{Performance \ e_x / \ Performance \ e_y = Execution \ time_y / \ Execution \ time_x = n}$</center>

으로 볼 수 있습니다. 예를 들어 동일한 task를 A는 10s, B는 15s가 걸린다면 A는 B보다 1.5배 성능이 좋은 것입니다. 

---

#### CPU Time

clock 주기로 clock cycle과 frequency(cycles per second)를 구합니다. 이를 통해 **CPU Time**을 계산하는데 다음과 같습니다. 

<center>$\mathrm{CPU \ Time = CPU \ Clock \ Cycles \times Clock \ Cycle \ Time = \frac{CPU Clock Cycles}{Clock Rate}}$</center>

이와 같은 식을 얻을 수 있습니다. CPU Time은 짧으면 짧을수록 좋습니다. 짧게 줄이는 방법은 clock cycle을 줄이고 clock rate 높이는 것인데, clock rate와 cycle count는 trade-off 관계에 있으므로 쉽게 CPU Time을 줄일 수는 없습니다.

---

#### Instruction Count and CPI

Instruction Count는 program에 사용되는 instruction의 개수를 의미합니다. instruction의 개수가 많으면 많을수록 프로그램은 느리게 돌아가죠. program, ISA(Instruction Set Architecture), Compiler에 따라 달라지게 됩니다. CPI는 Cycles Per Instruction의 약자로 HW에 의해 결정되며 instruction에 따라서도 CPI가 달라집니다. 

<center>$\mathrm{Clock \ Cycles = Instruction \ n \ Count \times \ Cycles \ per \ Instruction}$</center>

<center>$\mathrm{CPU \ Time = Instruction \ n \ Count \times \ CPI \times \ Clock \ Cycle \ Time = \frac{Insturcion \ Count \times CPI}{Clock \ Rate}}$</center>

이 모든 수식을 통해 CPU Time을 재정의할 수 있습니다. 

<center>$\mathrm{CPU \ Time = \frac{Instructions}{Program} \times \frac{Clock cycles}{Instruction} \times \frac{Seconds}{Clock cycle}}$</center>

---

#### MIPS: Millions of Instructions Per Second

MIPS는 Millons of Instructions Per Second의 약자로 또 하나의 성능지표가 될 수 있습니다. 

<center>$\mathrm{MIPS=\frac{Instruction \ count}{Execution \ time \times 10^6}=\frac{Instruction \ count}{\frac{Instruction \ count \times CPI}{Clock \ rate} \times 10^6}=\frac{Clock \ rate}{CPI \times 10^6}}$</center>

CPU에 따라 CPI가 달라져서 프로그램의 성능이 달라질 수 있습니다. 

---

### 퀴즈


<p align="center"><img src="../../assets/images/041302.jpg" width="800px" height="800px" title="OP code 예시" alt="OP code" ><img></p>

1) 각 CPU를 타겟으로 한 어셈블리 코드의 IC
- CPU1 : 6 + 1 + 1 = 8
- CPU2 : 1 + 3 = 4

2) 각 CPU를 타겟으로 한 어셈블리 코드의 Average CPI
- CPU1 : (2x6 + 5x1 + 3x1)/8 = 2.5
- CPU2 : (6x1 + 10x3)/4 = 9

3) 이 프로그램이 각각의 CPU에서 실행될 때의 CPU Time
- CPU1 : 8x2.5x1 = 20ns (IC x Average CPI x clock period)
- CPU2 : 4x9x4 = 144ns 

4) 이 프로그램이 각각의 CPU에서 실행될 때의 MIPS
- CPU1 : 8/(20x10^-9x10^6) = 400
- CPU2 : 4/(144x10^-9x10^6) = 27.8

5) 두 CPU의 capacitive load와 voltage가 서로 동일할 때 전력 소모의 비율
- 4:1 (power = CVF^2, F는 frequency)(frequency가 4배 차이 나기 때문에 4:1)

6) 이 프로그램이 각 CPU에서 실행될 때 에너지 소모의 비율
- (4x20):(1x144) = 80:144

7) 어떤 CPU가 더 성능이 좋다고 할 수 있을까? 
- CPU1 (400/4 : 27.8/1 = 100:27.8  이므로)(MIPS(throughput) / POWER로 계산하면 좋음)
