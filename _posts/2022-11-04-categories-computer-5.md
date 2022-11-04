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

- Hardwired vs Microprogrammed
  - binary information (called "Control Word")
  - Microinstruction
  - Microprogram

- Control words are stored in "Control Memory"
  - Control memory is usually ROM (fixed during fabrication)
  - Microprogrammed Control Unit has Control Memory
  - A Microprogrammed computer has a main memory and a control memory

- Dynamic microprogramming
  - A microprogram is loaded from auxiliary memory

즉, 이미 만들어져 있는 것을 Conrol memory에 넣고 꺼내서 쓴다는 점에서 전 장이랑 다릅니다.


<img src="../../assets/images/110306.jpg" width="500px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

