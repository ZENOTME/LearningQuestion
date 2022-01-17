SE315

- lab1 

  - 机器是如何多核启动的？
    - 读取特定标识各个核的寄存器，令标识为0的寄存器进行启动，其他跑入死循环中
  - 内核是如何加载和运行的？
    - 加载入LMA地址，运行到VA地址，bootloader负责建立了页表。	

- lecture 3 hardware,exception and syscall

  three question the lecture talking about:

  - **how do OS interact with the IO device?**

    - use Memory Map IO or port IO to control the IO device (including receiving and writing data to the device)
      - about Memory Map IO (https://www.youtube.com/watch?v=2Cbcb2yGjiM)

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

