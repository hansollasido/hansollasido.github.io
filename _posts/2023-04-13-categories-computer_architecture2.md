---
title: "[컴퓨터구조설계 및 응용 2]"
excerpt: "Representations"

categories:
  - 컴퓨터구조설계
tags:
  - [수업]

permalink: /categories9/computer_architecture2/

use_math: true
toc: true
toc_sticky: true

date: 2023-04-13
last_modified_at: 2023-04-13
---

# 컴퓨터구조설계 및 응용

수업을 듣고 정리한 페이지 입니다.

---

#### 표현 방식

컴퓨터가 수를 표현하기 위해서는 다양한 bit representation을 배워야 하는데요. 이 페이지에서는 여러 representation에 대해 알아보겠습니다.

---

##### Unsigned Binary Integers

<center>$x = x_{n-1}2^{n-1}+x_{n-2}^{n-2}+\dots+x_12^1+x_02^0$</center>

Range: $0$  to  $2^n-1$

---

##### 2s-Complement Signed Integers

<center>$x = -x_{n-1}2^{n-1}+x_{n-2}2^{n-2}+\dots+x_12^1+x_02^0$</center>

Range: $-2^{n-1}$ to $2^{n-1}-1$

---

##### Fixed-Point Representation

<p align="center"><img src="../../assets/images/041408.png" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

Integer part와 Fractional part가 정의되어 있음. 

---

##### Floating Point Standard

거의 모든 분야에서 쓰는 소수 표기법

Single precision (32-bit)

Double precision (64-bit)

<p align="center"><img src="../../assets/images/041409.jpg" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

<center>$\mathrm{x = (-1)^S\times (1+Fraction)\times 2^{(Exponent-Bias)}}$</center>

Bias는 Single일 때 127, Double일 때 1203임

---

###### Single-Precision Range

Exponents는 00000000, 11111111 reserved되어 있음

Smallest Value
- Exponent: 00000001
- Fraction: 00000...00 $=1$
- $\pm1.0\times 2^{-126}$

Largest Value
- Exponent: 11111110
- Fraction: 11111...11 $\approx2$
- $\pm2.0\times 2^{+127}$

---

###### Double-Precision Range

Exponents는 0000...00, 1111...11 reserved되어 있음

Smallest Value
- Exponent: 00000000001
- Fraction: 00000...00 $=1$
- $\pm1.0\times 2^{-1022}$

Largest Value
- Exponent: 11111111110
- Fraction: 11111...11 $\approx2$
- $\pm2.0\times 2^{+1023}$

---

##### Denormal Numbers

Exponent = 000...0일 때는 hidden bit가 0임

<center>$\mathrm{x=(-1)^S\times(0+Fraction)\times2^{-Bias}}$</center>

fraction이 0일 때

<center>$\mathrm{x=(-1)^S\times(0+0)\times2^{-Bias}}=\pm0.0$</center>

---

##### Infinities and NaNs

Exponent = 111...1일 때, Fraction = 000...0 이라면
- $\pm$Infinity

Exponent = 111...1일 때, Fraction $\neq$ 000...0 이라면
- Not a Number (NaN)

---

#### 퀴즈

1) 동일한 bit수를 이용하여 unsigned integer와 2's complement signed integer 방식으로 표현할 수 있는 숫자의 개수는 서로 다르다. : N

2) 동일한 64 bit를 이용한다고 할 때, unsigned integer로 표현할 수 있는 가장 큰 숫자가 floating point로 표현할 수 있는 가장 큰 숫자보다 크다. : N

3) Single precision floating point에 비하여 double precision floating point 방식으로 절대값이 더 작은 소수를 표현할 수 있다. : Y

4) Floating point 방식으로 가능한 표현 범위 내에서는 모든 숫자를 오차 없이 표현할 수 있다. : N

5) 다음의 2진수로 표현된 숫자를 각각 decimal number로 나타내시오. 
- 1001 (Unsigned Integer) : 9
- 1010 0011 (2's complement signed integer) : -93
- 0100 0000 1110 0000 0000 0000 0000 0000 (single-precision floating point) : 7 (1.75 x 4)

6) 다음의 숫자를 32-bit floating-point format (Bias=127)의 비트열로 나타내시오.
- -15(decimal) : 1.875 x 8 -> 1100 0001 0111 0000 0000 0000 0000 0000
- 0010.0010 (8-bit fixed-point) : 1.125 x 2 -> 0100 0000 0001 0000 0000 0000 0000 0000

---

#### floating point의 중요한 점

<p align="center"><img src="../../assets/images/041410.jpg" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

작아질수록 촘촘하게 표현가능하지만 크면 클수록 오차가 심해짐. 때문에 컴퓨터가 0.1 + 1.1 == 1.2를 계산한다면 false를 답을 내놓음. floating point로 계산시 오차가 발생하기 때문. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/-GsrYvZoAdA" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
