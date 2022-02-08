## ARMV8相关笔记 ##

### Exception

##### Interrupt和GIC

- 中断类型

  - Software Generated Interrupt (SGI) 

    通过配置Software Generated Interrupt Register (GICD_SGIR)实现核间通信，Interrupt IDs 0-15

  - Private Peripheral Interrupt (PPI)

    外围中断，每个核私有，Interrupt IDs16-31

  - Shared Peripheral Interrupt (SPI)

    外围中断，公共，Interrupt IDs 32-1020

- GIC运行机制

  interrupt -> distributor (state:pending) <br>

  The Distributor determines the highest priority  pending interrupt that can be delivered to a core and forwards that to the CPU interface of the  core. <br>

  At the CPU interface, the interrupt is in turn signaled to the core, at which point the core  takes the FIQ or IRQ exception.

- Distributor Register(每种中断对应一个)
  - GICD_IPRIORITY 配置优先级如何决定优先级？
  - GICD_ICFGR 决定触发方式，边缘触发or
  - GICD_ITARGETSR 决定该中断可以路由到哪个核，only applicable to SPIs
  - GICD_ISENABLER and  GICD_ICENABLER enable和disable
  - GICD_IGROUPR 决定是否分配给secure world
- GIC的初始化工作
  - For distributor : configure the priority, target, security and enable individual  interrupts
  - enable distributor through its control register (GICD_CTLR)
  - For each CPU interface : software must program the priority mask and  preemption settings. 
  - enable CPU interface through its control register (GICD_CTLR)
  - 准备interrupt vector和pstate相应位
- 对中断的控制机制
  - The entire interrupt mechanism in the system can be disabled by disabling the Distributor.  
  - Interrupt delivery to an individual core can be disabled by disabling its CPU interface.  
  - Individual interrupts can also be disabled (or enabled) in the distributor.

- Interrupt handling
  1. Reads the Interrupt Acknowledge Register from the CPU to get the Interrupt ID
  2. the read causes the interrupt to be marked as active in the  Distributor
  3. When the device-specific handler finishes execution, the top-level handler writes the same  interrupt ID to the End of Interrupt (EoI) register in the CPU Interface block, indicating the end  of interrupt processing.
  4. It is possible for there to be more than one interrupt waiting to be serviced on the same core, but  the CPU Interface can signal only one interrupt at a time. The top-level interrupt handler could  repeat the above sequence until it reads the special interrupt ID value 1023, indicating that there  are no more interrupts pending at this core.
- 额外一些问题
  - 怎么配置FIQ,IRQ
  - 低中断处理的时候高中断进来会发生什么,IRQ中断处理时候，FIQ中断进来会发生什么？

