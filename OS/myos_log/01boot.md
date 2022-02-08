## Boot OS的工作以及中间遇到的坑 ##

### 主要工作 ###

- 配置内存布局
- boot
  - switch exception level
  - prepre the rust enviroment to init (set the init stack)
  - clear bss
  - create init page
  - enable mmu
  - prepre the rust enviroment to run kernel (set the kernel stack)

### 配置内存布局 ###

这部分工作包括链接脚本的编写，主要的几个点：

- 利用链接脚本对内存进行分段布局
- 利用AT（LOAD_ADDR）进行加载地址和运行地址的区分

难点在于boot和kernel的安排，kernel的加载地址在低区域，但是运行地址是在高区域(0xffff_xxxx_..)，而boot运行地址有两种安排方式：

- 低区域，这样boot的流程代码比较难写，没办法调用到kernel中的一些模块，踩到的坑是在低区域的boot代码中利用**相对寻址**去调用高区域的kernel代码，这是不可行的，因为编译会报错相对偏移量过大

- 这是我采用的方式，将boot和内核把运行地址放在高区域，这么做有一个前提就是**必须编译成PIC（位置无关）代码**，否则在没有启动MMU之前直接访问高区域的绝对地址（0xffff_xxxx..）是无效的。这个方式有一个好处是可以调用kernel中的模块简化boot的工作，比如页表初始化。**坏处是如果遇到编译出的代码不是采用相对寻址方式访问内存就会出现错误**。

  > 在实践中我发现rust似乎不需要多余的配置本身就是生成pic代码。

### set the init stack ###

这部分代码参考的是(https://github.com/rcore-os/rCore/blob/master/kernel/src/arch/aarch64/boot/mod.rs)[rcore],一开始没有看懂，因为里面的init stack是没有显式声明的。由于整个程序是加载到0x8000(机器执行的第一个代码位置)，因此init stack直接使用的是0x8000上面的空间

### create init page ###

这部分工作主要就是建立页表这层抽象，具体放在另一篇日志中讲述，主要都是参考rcore项目。

踩到的坑主要有一开始没有把PageTableEntry配置ACCESS标志位，导致开启MMU后访问内存直接就报错了，查阅文档发现Access位如果不置1那么访问的时候是会出异常的。

顺便挖两个坑，**内存的sharebility和cachebility**。

### enable mmu ###

挖个坑，**内存屏障**

