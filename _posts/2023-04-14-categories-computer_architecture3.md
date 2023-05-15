---
title: "[🥎컴퓨터구조설계 및 응용 3]"
excerpt: "Arithmetic operations"

categories:
  - 컴구설
tags:
  - [수업]

permalink: /categories10/computer_architecture3/

use_math: true
toc: true
toc_sticky: true

date: 2023-04-14
last_modified_at: 2023-04-14
---

# 컴퓨터구조설계 및 응용

수업을 듣고 정리한 페이지 입니다.

---

#### Integer Addition

+와 -를 더하면 overflow가 발생하지 않음.

하지만 +두 개 더했는데 sign bit가 1이 되거나, -두 개를 더했는데 sign bit가 0이 되면은 overflow가 발생.

---

#### Integer Subtraction

2의 보수로 만들어준 다음 add하면 subtract와 같음

---

#### Multiplication

<p align="center"><img src="../../assets/images/041411.jpg" width="200px" height="200px" title="OP code 예시" alt="OP code" ><img></p>

<p align="center"><img src="../../assets/images/041412.jpg" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

<p align="center"><img src="../../assets/images/041413.jpg" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

Multiplication할 때, HW를 보면, Multiplicand는 1bit 씩 왼쪽으로 shift, Multiplier는 1bit씩 오른쪽으로 shift함. Multplier LSB가 1일 때는 Product Register에 있는 값과 Multiplicand의 값과 더하여 Product Register에 저장함.

---

##### Faster Multiplier

<p align="center"><img src="../../assets/images/041414.jpg" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

adder을 여러 개 사용하여 multiplier를 만들 수 있음. pipelined가 될 수 있으나 cost가 높다는 단점이 존재함.

---

#### Fixed-Point Multiplication

<p align="center"><img src="../../assets/images/041415.jpg" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

Integer Multiplication과 비슷함. point를 sum하여 sum한 만큼 옮겨줌.

---

#### Floating-Point Addition

4-digit decimal인 $9.999\times 10^1+1.610\times10^{-1}$로 예를 들어보자면
1. **Align decimal points**
- 가장 작은 exponent에 맞춰 shift
- $9.999\times10^1+0.016\times10^1$
2. **Add significands**
- $9.999\times10^1+0.016\times10^1=10.015\times10^1$
3. **Normalize result & check for over/underflow**
- $1.0015\times10^2$
4. **Round and renormalize if necessary**
- $1.002\times10^2$

---

#### FP Adder Hardware

Integer adder보다 복잡하기 때문에, cycle을 여러 개 두고 진행.

<p align="center"><img src="../../assets/images/041416.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

---

#### Floating-Point Multiplication

4-digit decimal인 $1.110\times10^{10}\times9.200\times10^{-5}$을 예로 들어보자면
1. **Add exponents**
- exponents를 서로 더함
- New exponents = 10 + -5 = 5
2. **Multiply significands**
- $1.110\times9.200=10.212\rightarrow10.212\times10^5$
3. **Normalize result & check for over/underflow**
- $1.0212\times10^6$
4. **Round and renormalize if necessary**
- $1.021\times10^6$
5. **Determine sign of result from signs of operands**
- $\pm1.021\times10^6$

---

#### Division

<p align="center"><img src="../../assets/images/041418.jpg" width="200px" height="200px" title="OP code 예시" alt="OP code" ><img></p>

<p align="center"><img src="../../assets/images/041417.jpg" width="600px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

---

### 퀴즈

1) 특정 bit 수의 2's complement integer로 표현된 어떤 양수와 음수를 더해도 overflow는 일어나지 않는다: Y

2) 64bit의 숫자에 256을 곱하는 연산을 할 때, adder를 이용해 256번 덧셈하는 것보다 multiplier를 이용하여 'x 256'연산을 하는 것이 bit 단위 add 연산 횟수가 더 적다. : Y

3) Unsigned integer 숫자에 right shift를 한 번 하면 정확히 /2한 값으로 표현된다. : N

4) Single-precision floating point로 오차 없이 표현된 두 개의 숫자는 곱셈을 해도 single precision floating point로 오차 없이 표현할 수 있다. : N

5) Single-precision floating point에서 아래 계산을 수행한 결과는? 
- $2^{20}+1$ : <font color="red">$2^{20} + 1$</font>
- $2^{30}+1$ : <font color="red">$2^{30}$</font>

6) 다음 single-precision floating-point 숫자의 addition을 수행하고 binary 표현으로 나타내시오.
- 0100 0000 0111 0000 0000 0000 0000 0000 + 1011 1111 0100 0000 0000 0000 0000 0000
  - <font color="red">0100 0000 0100 0000 0000 0000 0000 0000 (3.75-0.75 = 3)</font>
    - <font color="red">$1.111 \times 2^1 + (-1.1) \times 2^{-1}=1.111\times2^{1}+(-0.011)\times2^1=1.1\times2^1=3$</font>

7) 다음 single-precision floating-point 숫자의 multiplication을 수행하고 binary 표현으로 나타내시오.
- 0100 0000 1011 0000 0000 0000 0000 0000 * 0100 0001 0110 0000 0000 0000 0000 0000 
  - <font color="red">0100 0010 1001 1010 0000 0000 0000 0000 (5.5*14=77)</font>
    - <font color="red">$1.011\times2^2\times1.11\times2^3=10.01101\times2^5=1.001101\times2^6$</font>