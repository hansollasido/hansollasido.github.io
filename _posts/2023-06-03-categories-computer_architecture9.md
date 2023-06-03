---
title: "[👊컴퓨터구조설계 및 응용 9]"
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
  