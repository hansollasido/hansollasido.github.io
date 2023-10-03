---
title: "컴퓨터구조 공부1"
excerpt: "Computer Performance"

categories:
  - 컴퓨터구조
tags:
  - [컴퓨터구조]

permalink: /categories22/컴퓨터구조1/

use_math: true
toc: true
toc_sticky: true

date: 2023-10-02
last_modified_at: 2023-10-02
---

## Computer Architecture

우리가 항상 사용하는 컴퓨터는 정말 복잡한 시스템으로 구성되어 있다. 

---

### Computer Performance

- Response time 
  - task를 얼마나 빨리 끝내느냐
- Throughput
  - 단일시간 동안 얼마나 많은 일을 하느냐

컴퓨터 성능은 작업 완료 시간이 얼마나 되느냐에 따라 크게 달라진다. 빠르면 빠를 수록 좋은 컴퓨터라는 것은 모두가 다 아는 사실이다. 

특히나 Response time과 Throughput은 중요한 계산 알고리즘이다. 

또한 대표적으로 CPU Time이 있다.

$\mbox{CPU Time} = \mbox{CPU Clock Cycles} \times \mbox{Clock Cycle Time} = \frac{\mbox{CPU Clock Cycles}}{\mbox{Clock Rate}}$

CPU Time이 낮으면 낮을수록 빠른 것인데, 컴퓨터 성능을 높이려면 
- Reducing the number of clock cycles
- Increasing clock rate
위와 같이 해야한다. 

무작정 clock cycle을 줄이고, clock rate를 높인다고 좋은 것은 아니다. Hardware 설계자는 cycle count에 대한 clock rate의 trade-off를 반드시 고려해야한다. 

$\mbox{Clock Cycles} = \mbox{instruction Count} \times \mbox{Cycles per Instruction}$

$\mbox{CPU Time} = \mbox{Instruction Count} \times \mbox{CPI} \times \mbox{Clock Cycle Time} = \frac{\mbox{Instruction Count} \times \mbox{CPI}}{\mbox{Clock Rate}}$

Instruction Count는 프로그램마다 다른데, 특히 ISA(Instruction Set Architecture), 그리고 compiler에 따라 결정된다. 

Cycles per Instruction (CPI)는 CPU Hardware에 따라 결정된다. 

즉 정리하면,

$\mbox{CPU Time} = \frac{\mbox{Instructions}}{\mbox{Program}} \times \frac{\mbox{Clock cycles}}{\mbox{Instruction}} \times \frac{\mbox{Seconds}}{\mbox{Clock cycle}}$

처럼 되며 CPU Time은 프로그램 하나당 몇 초가 걸리느냐에 정의를 둘 수 있다. 

속도도 성능에 중요한 요소지만, 그에 따른 Power 소모도 큰 요소이다. 

$\mbox{Power} = \mbox{Capacitive load} \times \mbox{Voltage}^2 \times \mbox{Frequency}$

Voltage를 낮추면 Power도 그만큼 낮출 수 있다. 하지만 위 식에서 Frequency가 있는데, Frequency가 낮으면 낮을수록 빠르다는 의미이지만, 그만큼 Power 소모가 크다는 것을 보여준다. 

Instruction의 개수가 프로그램당 셀 수 없을 정도로 많다. 이에 따라 MIPS라는 Performance metric을 사용한다. 

$\mbox{MIPS}=\frac{\mbox{Instruction count}}{\mbox{Execution time}\times 10^6}=\frac{\mbox{Instruction count}}{\frac{\mbox{Instruction count}\times\mbox{CPI}}{\mbox{Clock rate}}\times 10^6}=\frac{\mbox{Clock rate}}{\mbox{CPI}\times 10^6}$

---

### Arithmetic operations

컴퓨터는 bit 단위로 연산을 한다. operator와 operand, carry가 있다. 

Multiplication도 진행할 수 있는데, left shift와 alu를 두어 곱하기를 하는 식으로 진행한다. 

<p align="center"><img src="../../assets/images/100201.jpg" width="500px" height="500px" title="Multiply" alt="Multiply" ><img></p>

여러 ALU를 두어 더 빠른 multiplier를 만드는 것도 가능하나, cost가 많이 든다. 

Integer Adder보다도 Floating Point Adder가 더 복잡하다. integer operation보다도 더 길고 더 많은 cycle이 필요하기 때문이다. 

<p align="center"><img src="../../assets/images/100202.jpg" width="500px" height="500px" title="FP Adder" alt="FP Adder" ><img></p>

