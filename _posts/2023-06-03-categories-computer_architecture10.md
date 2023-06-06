---
title: "[🤜컴퓨터구조설계 및 응용 10]"
excerpt: "Parallel PRocessors from Client to Cloud"

categories:
  - 컴구설
tags:
  - [수업]

permalink: /categories10/computer_architecture10/

use_math: true
toc: true
toc_sticky: true

date: 2023-06-04
last_modified_at: 2023-06-04
---

# 컴퓨터구조설계 및 응용

수업을 듣고 정리한 페이지 입니다.

---

**Introduction**

- Goal: connecting multiple computers to get higher performance
  - Multiprocessors
  - Scalability, availability, power efficiency
- Task-level (process-level) parallelism
  - High throughput for independent jobs
- Parallel processing program
  - Single program run on multiple processors
- Multicore microprocessors
  - Chips with multiple processors (cores)

Multi processor는 memory space가 독립적으로 다름. Multi core는 memory space share하고 연산만 따로 

---

**Hardware and Software**

- Hardware
  - Serial: e.g., Pentium 4
  - Parallel: e.g., quad-core Xeon e5345
- Software
  - Sequential: e.g., matrix multiplication
  - Concurrent: e.g., operating system
- Sequential/concurrent software can run on serial/parallel hardware
  - Challenge: making effective use of parallel hardware

막상 software가 병렬적으로 하는 것 같지만 실제로는  thread를 나눠서 번갈아 돌아가는 작업이 병렬적으로 보이는 것임.


HW도 중요하지만 그 자원을 사용하는 SW도 중요함.

---

**Parallel Programming**

- Parallel software is the problem
- Need to get significant performance improvement
  - Otherwise, just use a faster uniprocessor, since it's easier.
- Difficulties
  - Partitioning
  - Coordination
  - Communications overhead

--- 

**Amdahl's Law**

- Sequential part can limit speedup
- Example: 100 processors, 90x speedup?
  - parallel한 부분 많아야 speedup 효율이 좋아짐

---

**Scaling Example**

- Workload: sum of 10 scalars, and 10 x 10 matrix sum
  - Speed up from 10 to 100 processors
- Single processor: Time = (10 + 100) x $t_{add}$
- 10 processors
  - Time = 10 x $t_{add}$ + 100/10 x $t_{add}$ = 20 x $t_{add}$
  - Speedup = 110/20 = 5.5 (55% of potential)
- 100 processors
  - Time = 10 x $t_{add}$ + 100/100 x $t_{add}$ = 11 x $t_{add}$
  - Speedup = 110/11 = 10 (10% of potential)
- Assumes load can be balanced across processors

---

**Scaling Example (cont)**

- What if matrix size is 100 x 100?
- Single processor: Time = (10 + 10000) x $t_{add}$
- 10 processors
  - Time = 10 x $t_{add}$ + 10000/10 x $t_{add}$ = 1010 x $t_{add}$
  - speedup = 10010/1010 = 9.9 (99% of potential)
- 100 processors
  - Time = 10 x $t_{add}$ + 10000/100 x $t_{add}$ = 110 x $t_{add}$
  - speedup = 10010/110 = 91 (91% of potential)
- Assuming load balanced

자원이 많으면 많을수록 성능이 올라가긴 하지만, linear하게 올라가지는 않음

---

**Instruction and Data Streams**

- An alternate classification

<p align="center"><img src="../../assets/images/060601.jpg" width="500px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

- SPMD: Single Program Multiple Data
  - A parallel program on a MIMD computer
  - Conditional code for different processors

---

**SIMD**

- Operate elementwise on vecotrs of data
  - E.g., MMX and SSE instructions in x86
    - Multiple data elements in 128-bit wide registers
- All processors execute the same instruction at the same time
  - Each with different data address, etc.
- Simplifies synchronization
- Reduced instruction control hardware
- Works best for highly data-parallel applications

highly data-parallel applications은 딥러닝 가속기로 많이 쓰임

---

**Multithreading**

설계, 구조적인 얘기임

- Performing multiple threads of execution in parallel
  - Replicate register, PC, etc.
  - Fast switching between threads
- Fine-grain multithreading (작은 단위)
  - Switch threads after each cycle
  - Interleave instruction execution
  - If one thread stalls, others are executed
- Coarse-grain multithreading (큰 단위)
  - Only switch on long stall (e.g., L2-cache miss)
  - Simplifies hardware, but doesn't hide short stalls (eg, data hazards)

---

**History of GPUs**

- Early video cards
  - Frame buffer memory with address generation for video output
- 3D graphics processing
  - Originally high-end computers (e.g., SGI)
  - Moore's Law => lower cost, higher density
  - 3D graphics cards for PCs and game concoles
- Graphics Processing Units
  - Processors oriented to 3D graphics tasks
  - Vertex/pixel processing, shading, texture mapping, rasterization

---

**GPU Architectures**

- Processing is highly data-parallel
  - GPUs are highly multithreaded
  - Use thread switching to hide memory latency
    - Less reliance on multi-level caches
  - Graphis memory is wide and high-bandwidth
- Trend toward general purpose GPUs
  - Heterogeneous CPU/GPU systems
  - CPU for sequential code, GPU for parallel code
- Programming languages/APIs
  - DirectX, OpenGL
  - C for Graphics (Cg), High Level Shader Language 9HLSL
  - Compute Unified Device Architecture (CUDA)

--- 

**Modeling Performance**

- Assume performance metric of interest is achievable GFLOPs/sec
  - Measured using computational kernels from Berkeley Design Patterns
- Arithmetic intensity of a kernel
  - FLOPs per byte of memory accessed
- For a given computer, determine
  - Peak  GFLOPS (from data sheet) 
  - Peak memory bytes/sec (using Stream benchmark)

FLOPs = Floating point operations

FLOPs/sec = FLOPS

FLOPS는 대표적 GPU 성능 수치임

---

**Roofline Diagram**

<p align="center"><img src="../../assets/images/060602.jpg" width="600px" height="600px" title="OP code 예시" alt="OP code" ><img></p>

HW 최고 성능 한계로 인해 GFLOPs/second가 높지는 않음

- 어떤 부분에서는 Memory bandwidth로 인해, 어떤 부분에서는 Processor로 인해 한계가 있음

---

**Comparing Systems**

- Opteron X2, Opteron X4
  - 2-core vs. 4-core, 2x FP performance/core, 2.2GHz vs. 2.3GHz
  - Same memory system

<p align="center"><img src="../../assets/images/060603.jpg" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

- To get higher performance on X4 than X2
  - Need high arithmetic intensity
  - Or working set must fit in X4's 2MB L-3 cache

X4의 성능이 좋게 만들려면, 연산 강도를 높이거나 set가 X4의 2MB L-3 cache에 맞으면 됨

---

**Optimizing Performance**

<p align="center"><img src="../../assets/images/060604.jpg" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

- Optimize FP performance
  - Balance adds & multiplies
  - Improve superscalar ILP and use of SIMD instructions

<p align="center"><img src="../../assets/images/060605.jpg" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

- Optimize memory usage
  - Software prefetch
    - Avoid load stalls
  - Memory affinity (프로세스 또는 스레드가 우선순위 또는 바인딩을 지정)
    - Avoid non-local data accesses

<p align="center"><img src="../../assets/images/060606.jpg" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

- Choice of optimization depends on arithmetic intensity of code
- Arithmetic intensity is not always fixed
  - May scale with problem size
  - Caching reduces memory accesses
    - Increases arithmetic intensity

Caching이 arithmetic intensity를 올려줌. arithmetic intensity는 고정적이지 않기 때문에 어떤 특성인지에 따라 달라짐.

---

**Rooflines**

<p align="center"><img src="../../assets/images/060607.jpg" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>
  
---

**Concluding Remarks**

- Goal: higher performance by using multiple processors
- Difficulites
  - Developing parallel software
  - Devising appropriate architectrues
- SaaS importance is growing and clusters are a good match
- Performance per dollar and performance per Joule drive both mobile and WSC

SaaS는 Software as a service의 약자. Iaas는 Infrastructure as a service의 약자. 

속도 올라가면 전력소모도 올라가기 때문에 overhead가 큼. 성능당 비용 및 성능당 에너지 소모 또한 매우 중요함. 

---

## 퀴즈

**1) 특정 SW코드의 서로 다른 두 line은 병렬 HW 없이는 동시에 실행될 수 없다.** <font color="red">Y</font>

**2-1) 8배의 speedup을 얻으려면 몇 개의 processor가 필요한가?**

<font color="red">(10 + 100)t / (10 + 100/x)t = 8 -> x>27</font>

**그리고 이 때는 potential 대비 몇 % 성능 향상인가?**

<font color="red">8.x/27 = 약 30%</font>

**2-2) 이 경우에서 이론적으로 얻을 수 있는 최대 speedup은 몇으로 수렴하는가?**

<font color="red">processor 개수가 100개 초과하면 실질적인 속도 향상이 없으므로 (10+100)/(10+1) = 10배</font>

**3) roofline diagram에서,** 

**3-1) Kernel 1에 대해 2배의 성능향상을 얻기 위한 방법?**
  - <font color="red">Arithmetic intensity 2배 증가 또는</font>
  - <font color="red">Peak memory BW 2배 증가</font>

**3-2) Kernel 2에 대해 2배의 성능 향상을 얻기 위한 방법?**
  - <font color="red">Peak floating-point performance 2배 증가</font>

**3-3) Roofline과 x,y 축이 이루는 영역의 넓이가 넓을수록 일반적으로 성능이 좋은 시스템이라고 할 수 있는가?**
  - <font color="red">일반적으로 yes</font>

**4) 아래의 표는 NVIDIA A100과 H100 GPU에 대한 연산/메모리 성능 수치 중 일부이다.** 

<p align="center"><img src="../../assets/images/060608.png" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

**4-1) 각 GPU의 성능 (FP16, INT8 경우 모두, with sparsity 가정)을 roofline diagram으로 나타내시오.**

<p align="center"><img src="../../assets/images/060609.png" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>

**4-2) Arithmetic intensity가 600 OPS/Byte일 때 두 GPU의 성능 차이는?**

- <font color="red">A100-FP16:624, INT8:1200</font>
- <font color="red">H100-FP16:1979, INT8:2010</font>

**4-3) H100 GPU로 3000 TFLOPS를 달성하기 위한 방법은?**

- <font color="red">INT8 이용, Arithmetic intensity=(3000/3.35)Ops/B 이상으로 동작

**4-4) H100 GPU의 memory BW가 1TB/s로 줄어들었다고 할 때, H100 이 A100보다 성능이 더 좋기 위한 arithmetic intensity의 범위는?**

<p align="center"><img src="../../assets/images/060610.png" width="400px" height="400px" title="OP code 예시" alt="OP code" ><img></p>