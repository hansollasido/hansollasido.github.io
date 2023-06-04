---
title: "[👏퓨터구조설계 및 응용 8]"
excerpt: "Large and Fast: Exploiting Memory Hierarchy"

categories:
  - 컴구설
tags:
  - [수업]

permalink: /categories10/computer_architecture8/

use_math: true
toc: true
toc_sticky: true

date: 2023-06-02
last_modified_at: 2023-06-02
---

# 컴퓨터구조설계 및 응용

수업을 듣고 정리한 페이지 입니다.

---

**Principle of Locality**

- Programs access a small proportion of their address space at any time
- Temporal locality (시간적 지역성)
  - Items accessed recently are likely to be accessed again soon
  - Instructions in a loop, induction variables
- Spatial locality (공간적 지역성)
  - Items near those accessed recently are likely to be accessed soon (메모리에서 address에 따라서 순차적 접근)
  - Sequential instruction access, array data

---

**Taking advantage of Locality**

- Memory hierarcy
- Store everything on disk (disk에 모든 data가 있음)
- Copy recently accessed (and nearby) items from disk to smaller DRAM memory 
  - Main memory 
- Copy more recently accessed (and nearby) items from DRAM to smaller SRAM memory
  - Cache memory attached to CPU

---

**Memory Hierarchy Levels**

- Block (aka line): unit of copying
  - May be multiple words
- If accessed data is present in upper level
  - Hit: access satisfied by upper level
- If accessed data is absent
  - Miss: block copied from lower level
    - Time taken: miss penalty
    - Miss ratio: misses/accesses = 1 - hit ratio
  - Then accessed data supplied from upper level

큰 메모리 (DRAM)은 용량은 크지만 그만큼 속도가 느림

Cache는 작은 메모리 (SRAM)이며 용량은 작지만 그만큼 속도가 빠름

이런 hierarchy 구조로 메모리에서 data를 가져오고 cache에서 data를 processor로 보냄

---

**Memory Technology**

- Static RAM (SRAM)
  - 0.5ns-2.5ns, 2000 dollars - 5000 dollars per GB
- Dynamic RAM (DRAM)
  - 50ns-70ns, 20 dollars - 75 dollars per GB
- Magnetic disk
  - 5ms -20ms, 0.2 dollars - 2 dollars per GB
- Ideal memory 
  - Access time of SRAM
  - Capacity and cost/GB of disk

SRAM에 access하는 것이 좋지만 비용과 용량 차원에서 단점이 있음

---

**Cache Memory**

- The level of the memory hierarchy closest to the CPU
- Given accesses $X_1, ..., X_{n-1}, X_n$
- How do we know if the data is present?
- Where do we look?

---

**Direct Mapped Cache**

- Location determined by address
- Direct mapped: only one choice
  - (Block address) modulo (#Blocks in cache)

---

**Tags and Valid Bits**

- How do we know which particular block is stored in a cache location?
  - Store block address as well as the data
  - Actually, only need the high-order bits 
  - Called the tag
- What if there is no data in a location?
  - Valid bit: 1=present, 0=not present
  - Initially 0


---

**Cache Example**

- 8-blocks, 1 word/block, direct mapped
- Initial state

<p align="left"><img src="../../assets/images/060202.jpg" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>


|Word addr|Binary addr|Hit/miss|Cache block|
|---|---|---|---|
|22|10 110|Miss|110|


<p align="left"><img src="../../assets/images/060203.jpg" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

|Word addr|Binary addr|Hit/miss|Cache block|
|---|---|---|---|
|26|11 010|Miss|110|

<p align="left"><img src="../../assets/images/060204.jpg" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

|Word addr|Binary addr|Hit/miss|Cache block|
|---|---|---|---|
|22|10 110|Hit|110
|26|11 010|Hit|010|

---

|Word addr|Binary addr|Hit/miss|Cache block|
|---|---|---|---|
|16|10 000|Miss|000|
|3|00 011|Miss|011|
|16|10 000|Hit|000|

<p align="left"><img src="../../assets/images/060205.jpg" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

|Word addr|Binary addr|Hit/miss|Cache block|
|---|---|---|---|
|18|10 010|Miss|010|

<p align="left"><img src="../../assets/images/060206.jpg" width="500px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

---

**Address Subdivision**

<p align="center"><img src="../../assets/images/060207.jpg" width="500px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

- Index, Tag로 data를 가져옴

---

**Example: Larger Block Size**

- 64 blocks, 16 bytes/block
  - To what block number does address 1200 map?
- Block address = 1200/16 = 75
- Block number = 75 modulo 64 = 11


---

**Block Size Considerations**

- Larger blocks should reduce miss rate
  - Due to spatial locality
- But in a fixed-sized cache
  - Larger blocks => fewer of them
    - More competition => increased miss rate
  - Larger blocks => pollution
- Larger miss penalty 
  - Can override benefit of reduced miss rate

---

**Cache Misses**

- On cache hit, CPU proceeds normally 
- On cache miss
  - Stall the CPU pipeline
  - Fetch block from next level of hierarchy
  - Instruction cache miss
    - Restart instruction fetch
  - Data cache miss
    - Complete data access


---

<p align="center"><img src="../../assets/images/060208.jpg" width="500px" height="400px" title="OP code 예시" alt="OP code" ><img></p>


---

**Measuring Cache Performance**

- Components of CPU time
  - Program execution cycles
    - Includes cache hit time
  - Memory stall cycles
    - Mainly from cache misses

- With simplifying assumptions:

$\mbox{Memory stall cycles} $ <br><br>
$ = \frac{\mbox{Memory accesses}}{\mbox{Program}} \times \mbox{Miss rate} \times \mbox{Miss penalty}$ <br><br>
$= \frac{\mbox{Instructions}}{\mbox{Program}} \times \frac{\mbox{Misses}}{\mbox{Instruction}}\times \mbox{Miss penalty}$

---

**Cache Performance Example**

- Given 
  - I-cache miss rate = 2%
  - D-cache miss rate = 4%
  - Miss penalty  = 100 cycles
  - Base CPI (ideal cache) = 2
  - Load & stores are 36% of instructions
- Miss cycles per instruction
  - I-cache: 0.02 x 100 = 2 (평균적으로 2 cycle)
  - D-cache: 0.36 x 0.04 x 100 = 1.44
- Actual CPI = 2 + 2 + 1.44 = 5.44
  - Ideal CPU is 5.44/2 = 2.72 times faster

---

**Average Access Time**

- Hit time is also important for performance
- Average memory access time (AMAT)
  - AMAT = Hit time + Miss rate x Miss penalty
- Example
  - CPU with 1ns clock, hit time = 1 cycle, miss penalty = 20 cycles, I-cache miss rate = 5%
  - AMAT = 1 + 0.05 x 20 = 2ns
    - 2 cycles per instruction

---

**Performance Summary**

- When CPU performance increased
  - Miss penalty becomes more significant
- Decreasing base CPI
  - Greater proportion of time spent on memory stalls
- Increasing clock rate
  - Memory stalls account for more CPU cycles
- Can't neglect cache behavior when evaluating system performance

---

**Associative Caches**

- Fully associative
  - Allow a given block to go in any cache entry
  - Requires all entries to be searched at once 
  - Comparator per entry (expensive)
- n-way set associative
  - Each set contains n entries
  - Block number determines which set
    - (Block number) modulo (#Sets in cache)
  - Search all entries in a given set at once
  - n comparators (less expensive)

<p align="center"><img src="../../assets/images/060209.jpg" width="500px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

Directed-mapped, n-way set associative, fully associative 등이 있음

<p align="center"><img src="../../assets/images/060210.jpg" width="500px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

---

**Associativity Example**

- Compare 4-block caches
  - Direct mapped, 2-way set associative, fully associative
  - Block access sequence: 0, 8, 0, 6, 8

- Direct mapped

<p align="center"><img src="../../assets/images/060211.jpg" width="500px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

- 2-way set associative

<p align="center"><img src="../../assets/images/060212.jpg" width="500px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

- Fully associative

<p align="center"><img src="../../assets/images/060213.jpg" width="500px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

---

**How Much Associativity**

- Increased associativity decreases miss rate
  - But with diminishing returns
- Simulation of a system with 64KB D-cache, 16-word blocks, SPEC2000
  - 1-way: 10.3%
  - 2-way: 8.6%
  - 4-way: 8.3%
  - 8-way: 8.1%

--- 

**Set Associative Cache Organization**

<p align="center"><img src="../../assets/images/060214.jpg" width="500px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

---

**Replacement Policy**

- Direct mapped: no choice
- Set associative
  - Prefer non-valid entry, if there is one
  - Otherwise, choose among entries in the set
- Least-recently used (LRU)
  - Choose the one unused for the longest time
    - Simple for 2-way, manageable for 4-way, too hard beyond that
- Random 
  - Gives approximately the same performance as LRU for high associativity

---

#### Cache Design Trade-offs

|Design change|Effect on miss rate|Negative performance effect|
|---|---|---|
|Increase cache size|Decrease capacity misses|May increase access time|
|Increase associativity|Decrease conflict misses|May increase access time|
|Increase block size|Decrease compulsory misses|Increases miss penalty. For very large block size, may increase miss rate due to pollution.|

---

### 퀴즈

1) Cache memory에서 block size가 작을수록 한 block에 대한 miss penalty가 줄어든다. <font color="red">Y</font>


2) Cache memory의 크기가 일정할 때 block size를 늘리면 index bit width가 감소한다. <font color="red">Y</font>


3) Cache memory의 크기가 일정할 때 associativity를 늘리면 index bit width가 감소한다. <font color="red">Y</font>


4) 1-way set associative cache에서는 block replacement 알고리즘 선택이 의미가 없다. <font color="red">Y</font>


5) 일반적으로 Cache memory의 associativity가 증가할수록 miss rate는 대체로 감소한다. <font color="red">Y</font>


6) 8-bit addressing을 사용하는 컴퓨터 시스템에서 64B의 data 크기를 갖는 cache memory를 1-way, 2-way 및 full-associative의 세 가지 set associativity로 설계하여 성능을 비교하려고 한다(Cache memory의 데이터 access 단위는 double-word로 가정).


6-1) 각각의 경우에 대해 cache 메모리의 구조와, cache를 접근하기 위한 8-bit address의 구성을 표시하시오. 


<p align="left"><img src="../../assets/images/060215.jpg" width="100px" height="100px" title="OP code 예시" alt="OP code" ><img></p>

- <font color="red">1-way</font>
- <font color="red">Address(8bit)</font>

|Tag|Index|Offset|
|---|---|---|
|2bit|3bit|3bit|

---

<p align="left"><img src="../../assets/images/060216.jpg" width="300px" height="300px" title="OP code 예시" alt="OP code" ><img></p>

- <font color="red">2-way</font>
- <font color="red">Address(8bit)</font>

|Tag|Index|Offset|
|---|---|---|
|3bit|2bit|3bit|

---

<p align="left"><img src="../../assets/images/060217.jpg" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>

- <font color="red">fully-associative</font>
- <font color="red">Address(8bit)</font>

|Tag|Offset|
|---|---|
|5bit|3bit|

---

6-2) 각각의 경우에 대해 아래 주소로의 메모리 접근 (Read)이 들어올 때 Hit, Miss를 표시하시오 (Cache 메모리는 초기에 비어있으며, replacement policy는 LRU임)

||4A|51|0D|4B|D1|5C|
|---|---|---|---|---|---|---|
|1-way|Miss|Miss|Miss|Miss|Miss|Miss|
|2-way|Miss|Miss|Miss|<font color="red">Hit</font>|Miss|Miss|
|Fully-associative|Miss|Miss|Miss|<font color="red">Hit</font>|Miss|Miss|