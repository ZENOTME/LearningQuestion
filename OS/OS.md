SE315 

- lecture 3 hardware,exception and syscall

  three question the lecture talking about:

  - **how do OS interact with the IO device?**

    - use Memory Map IO or port IO to control the IO device (including receiving and writing data to the device)
      - about [Memory Map IO](https://www.youtube.com/watch?v=2Cbcb2yGjiM)

    - how do OS realize the IO device already to communicate? 
      - Poll. Keep watching the state of the device.
      - Interrupt(asynchronous exception).

  - **GIC(general interrupt controller)**

  - **Interrupt Handler**

    - separate the handler into two component: Top half and Bottom Half

      - Top half:  smallest, public task
      - Bottom half: defer rest of task using **softirqs,tasklets,task queue**
      - **Question:** ***How did a RTOS handle this to keep real-time property***

    - the limit of Interrupt Handler

      - can't sleep and schedule

        - when interrupt occur, the processor gets into an exception state (Interrupt context). while this happens the scheduler is disabled until the processor exit this state. if you put a task into sleep, the task get into wait queue and tell the scheduler to dequeue another task. if it happens in interrupt context, there is no scheduler until we finish this context and the processor hangs because we never finished the interrupt.

      - can't access the userspace

        - It doesn’t make sense to do that, and it’s possible there is no userspace memory currently mapped.

          Interrupts (besides system calls which are a different point of view already) don’t ever occur in the context of a process and thus don’t have any user memory to handle. They can’t rely on any user memory being available at all. They might appear while an exiting task is running and the usermode memory is partially torn down.

  - **the process of handle exception** 

    - a certain case of handling the system call

### Memory Management ###

#### 内存管理的演化 ####

- Protection Key 机制的挑战
- 虚拟内存的建立
- 分段到分页
- 单级分页到多级分页
- 多级页表到大页机制

#### 硬件MMU机制和特性

- 页表项对相应内存的控制
  - 读写权限，执行权限，访问权限
  - 内存类型：Devive(cacheable),Normal(non-cacheable) 
- **Cache Lockdown**
- **TLB Flush机制**
  - AARCH64：ASID（Address Space ID）
  - TLB在多核下挑战
    - 一个进程运行在多个核时，利用ASID对多个核进行识别核刷新
  - 全局TLB
    - AARCH64通过页表项的nG位对TLB的全局性控制
    - 需要场景：共享库

#### 按需分配与换页

- 换页基本思想
- 按需分配中的权衡
  - 优势：节约内存资源 
  - 劣势：异常导致延迟
  - **（预取）Prefetching机制**
- 页替换策略
- **Thrashing Problem**（见PPT）
  - 解决方案：**工作集模型(Working Set Model)**
- **重新思考现在还需要按需分配的思想吗**
  - [Anti-Caching -- A New Approach to Database Management System Architecture](https://www.vldb.org/pvldb/vol6/p1942-debrabant.pdf)

#### 物理内存管理

- 物理内存结构
- 物理内存管理的碎片问题
  - 外部碎片
  - 内部碎片
- Linux对物理内存的管理机制
  - Buddy System ：解决外部碎片
  - SLAB分配器：解决内部碎片
- **安全问题**
  - Rowhammer Attack
  - Cache side channel

- **Cache相关的性能问题**
  - Cache missing 的高开销以及恶意攻击
  - 一系列的解决措施
    - **cache coloring**
    - **AARCH 64 MPAM**
    - **Intel CAT**

#### 操作系统内存管理的功能

- shared memory
- copy-on-write
- **memory deduplication (内存去重)**
  - 经典案例：Linux KSM
  - 潜在安全隐患：导致新的side channel
    - 访问被合并的页会导致访问延迟明显

- **内存压缩的思想**
  - 将要换出的内存先进行压缩，压缩后可换出可不换出
  - 好处：减少甚至避免磁盘I/O；增加设备寿命
- **Linux madvise API**

#### 思考题

- 不依赖MMU，是否可以替换虚拟内存的方法
  - **Software Fault Isolation**
