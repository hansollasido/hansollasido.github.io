---
title: "[🤛컴퓨터구조설계 및 응용 9]"
excerpt: "Virtual Memory"

categories:
  - 컴구설
tags:
  - [수업]

permalink: /categories10/computer_architecture9/

use_math: true
toc: true
toc_sticky: true

date: 2023-06-03
last_modified_at: 2023-06-03
---

# 컴퓨터구조설계 및 응용

수업을 듣고 정리한 페이지 입니다.

---

**Virtual Memory**

- Use main memory as a "cache" for secondary (disk) storage
  - Managed jointly by CPU hardware and the operating system (OS)

- Programs share main memory 
  - Each gets a private virtual address space holding its frequently used code and data
  - Protected from other programs

- CPU and OS translate virtual addresses to physical addresses
  - VM "block" is called a page
  - VM translation "miss" is called a page fault

---

**Fixed-size pages**

- Virtual addresses와 Physical address, Disk addresses 사이에 Address translation이 있어 mapping이 존재함

---

**Page Tables**

- Stores placement information
  - Array of page table entries, indexed by virtual page number
  - Page table register in CPU points to page table in physical memory

- If page is present in memory
  - PTE stores the physical page number
  - Plus other status bits (referenced, dirty, ...)

- If page is not present
  - PTE can refer to location in swap spacec on disk

---

**Translation Using a Page Table**

<p align="center"><img src="../../assets/images/060301.jpg" width="500px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

---

**Address Translation**

- How many bits required?
  - VPN (Virtual page #)
  - PFN (Physical page #)
  - Page Offset

---

**Example: Address Translation**

- Virtual memory space: 4GB
  - virtual address: 32 bits
- Physical memory space: 1MB
  - Physical address: 20 bits
- Page size: 4kB
  - Offset: 12 bits
  - VPN: 20bits, PFN: 8bits
  - Number of pages: $2^{20}$
  - Number of page table entries: $2^{20}$
- Page table size?
  - $2^{20}\times(8+8)$

---

**Virtual Memory Performance**

- Example
  - Memory access time: 100ns
  - Disk access time: 25ms
  - Effective access time
    - Let p = the probability of a page fault
    - Effective access time = 100(1-p) + 25,000,000p
    - If we want only 10% degradation
      - 110 > 100 + 25,000,000p
      - 10 > 25,000,000p
      - p < 0.0000004 (one fault every 2,500,000 references)
  - Lesson: OS had better do a good job of page replacement!
  - OS가 굉장히 중요함

---

**Replacement and Writes**

- To reduce page fault rate, prefer least recently used (LRU) replacement
  - Reference bit (aka use bit) in PTE set to 1 on access to page
  - Periodically cleared to 0 by OS
  - A page with reference bit = 0  has not been used recently
- Disk writes take millions of cycles
  - Write through is impractical
  - Use write-back
  - Dirty bit in PTE set when page is written

---

**Page Size**

- Small page sizes
  - <font color="blue">less internal fragmentation, better memory utilization.</font>
  - <font color="red">large page table, high page fault handling overheads.</font>
- Large page sizes
  - <font color="blue">small page table, small page fault handling overheads.</font>
  - <font color="red">more internal fragmentation, worse memory utilization.</font>

---

**Fast Translation Using a TLB**

- Address translation would appear to require extra memory references
  - One to access the PTE
  - Then the actual memory access
- But access to page tables has good locality
  - So use a fast cache of PTEs within the CPU
  - Called a Tranlsation Look-aside Buffer (TLB)
  - Misses could be handled by hardware or software

---

**Address Translation with TLB**

<p align="center"><img src="../../assets/images/060302.jpg" width="500px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

---

**TLB Misses**

- If page is in memory
  - Load the PTE from memory and retry

- If page is not in memory (page fault)
  - Locate page on disk
  - Choose page to replace
    - If dirty, write to disk first
  - Read page into memory and update page table
  - Then restart the faulting instruction

---

**TLB and Cache Interaction**

<p align="center"><img src="../../assets/images/060303.jpg" width="500px" height="400px" title="OP code 예시" alt="OP code" ><img></p>


- If cache tag uses physical address  
  - Need to translate before cache lookup
- Alternative: use virtual address tag
  - Complications due to alisaing
    - Different virtual addresses for shared physical address

---

**The Big Picture**

<p align="center"><img src="../../assets/images/060304.jpg" width="700px" height="500px" title="OP code 예시" alt="OP code" ><img></p>

|TLB|Page table|Cache|Possible? If so, under whtat circumstance?|
|---|---|---|---|
|Hit|Hit|Miss|Possible, although the page table is never really checked if TLB hits.|
|Miss|Hit|Hit|TLB misses, but entry found in page table; after retry, data is found in cache.|
|Miss|Hit|Miss|TLB misses, but entry found in page table; after retry, data misses in cache.|
|Miss|Miss|Miss|TLB misses and is followed by a page fault; after retry, data must miss in cache.|
|Hit|Miss|Miss|**Impossible**: cannot have a translation in TLB if page is not present in memory.|
|Hit|Miss|Hit|**Impossible**: cannot have a translation in TLB if page is not present in memory.|
|Miss|Miss|Hit|**Impossible**: data cannot be allowed in cache if the page is not in memory.|

main memory안에 cache가 있고, page table에 TLB가 있음. TLB에는 있는데 Page Table에는 없다? 이건 말이 안됨.

---

**The Memory Hierarchy**

- Common principles apply at all levels of the memory hierarchy
  - Based on notions of caching
- At each level in the hierarchy
  - Block placement
  - Finding a block
  - Replacement on a miss
  - Write policy

---

**Block Placement**

- Determined by associativity
  - Direct mapped (1-way associative)
    - One choice for placement
  - n-way set associative
    - n choices within a set
  - Fully associative
    - Any location
- Higher associativity reduces miss rate
  - Increases complexity, cost, and access time

--- 

**Finding a Block**

|Associativity|Location method|Tag comparisons|
|---|---|---|
|Direct mapped|Index|1|
|n-way set associative|Set index, then search entries within the set|n|
|Fully associative|Search all entries|#entries|
|Fully associative|Full lookup table|0|

- Hardware caches
  - Reduce comparisons to reduce cost
- Virtual memory
  - Full table lookup makes full associativity feasible
  - Benefit in reduced miss rate

---

**Replacement**

- Choice of entry to replace o a miss
  - Least recently used (LRU)
    - Complex and costly hardware for high associativity
  - Random
    - Close to LRU, easier to implement
- Virtual memory
  - LRU approximation with hardware support

---

**Write Policy**

- Write-through (그 때 그 때)
  - Update both upper and lower levels
  - Simplifies replacement, but may require write buffer
- Write-back
  - Update upper level only
  - Update lower level when block is replaced
  - Need to keep more state
- Virtual memory
  - Only write-back is feasible, given disk write latency

---

**Sources of Misses**

- Compulsory misses (aka cold start misses)
  - First access to a block
- Capacity misses
  - Due to finite cache size
  - A replaced block is later accessed again
- Conflict misses (aka collision misses)
  - In a non-fully associative cache
  - Due to competition for entries in a set
  - Would not occur in a fully associative cache of the same total size

---

## 퀴즈

1) 다음과 같이 제약조건에 따라 컴퓨터 시스템을 설계하려고 할 때, 최대 virtual memory space의 크기를 구하시오. 

- 1GB physical memory space 
- 16kB page size
- 하나의 page table을 저장하는데 최대 1개의 physical frame 이용 가능
- Page table에서 page number 외에 추가 정보를 저장하는 데 하나의 PTE 당 16bit 필요

<p align="center"><img src="../../assets/images/060305.png" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>

2) 32-bit virtual addressing과 256MB physical memory를 사용하는 시스템에서 64byte 크기의 TLB를 탑재하려고 할 때 수용 가능한 TLB entry의 개수를 구하시오. 

- Page size = 32kB
- TLB에는 Tag와 page number 외에 valid bit과 dirty bit을 추가로 저장

- <font color="red">Virtual mem = $2^{32}$bytes -> #pages = $2^{32-15}$=$2^{17}$</font>
- <font color="red">Physical mem = $2^{28}$bytes -> #frames = $2^{28-15}$=$2^{13}$</font>


- <font color="red">One TLB entry = (17+13+2) bits = 4 bytes</font>


- <font color="red">-> # TLB entries  = 64/4 = 16개</font>

3) Paging을 사용하는 어떤 컴퓨터 시스템에서 page table과 physical memory의 구성과 내용이 아래와 같을 때 오른쪽 코드가 실행된다고 가정하자.

<p align="center"><img src="../../assets/images/060306.png" width="700px" height="500px" title="OP code 예시" alt="OP code" ><img></p>

3-1) virtual address 0x38에 해당하는 variable (array element)는?

- <font color="red">a[3]</font>

3-2) a[0]의 virtual address는?

- <font color="red">0x2C</font>

3-3) 3개의 entry를 가지는 TLB를 사용한다고 할 때, 위 코드가 실행되는 동안 TLB 내용의 변화를 표시하시오 (초기에 TLB는 비어있고 replacement policy는 LRU로 가정)

<p align="center"><img src="../../assets/images/060307.png" width="500px" height="500px" title="OP code 예시" alt="OP code" ><img></p>

3-4) 위 코드가 실행되는 동안의 TLB hit rate는?

- <font color="red">16/20</font>

