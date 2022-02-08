提前声明：

该项目的所有工作设计灵感来自于rcore，chcore等社区，非原创

### 2/28

### 进度：

- boot完成
- 内存系统完成
- 驱动初始化完成
- 异常配置完成
- 地址空间完成

### 整体架构

![](E:\Users\ze\Desktop\LearningLog\OS\myos_log\FlexOS-arch-v0.1.png)

系统分为driver(驱动层)，kernel(内核层)，arch(架构层)，目标是为了将系统解耦成三层可以进行独立开发，互不影响

#### 关于驱动层和内核层的连接：

驱动层一部分在内核包中，一部分在另外的驱动包中。

virio_driver作为虚拟设备驱动代码是一个独立的驱动包，内核包对virio_driver导入后进行调用

**但是virtio_driver需要用到的内存分配**，因此virtio_driver通过符号链接的方式暴露了内存分配的接口,该接口需要在内核包利用frame_allocator(内存分配模块)进行实现,(否则链接会出错)，具体例子如下：

```
//virtio_driver
extern "C"{
	fn mm_alloc();
}
//kernel/virtio_impl
fn mm_alloc(){
	frame_allocator.alloc();
}
```

pl011是在内核包中进行实现串口驱动，通过该驱动实现将字符打印到屏幕上，内核中logging利用该驱动实现其他更加高级的打印方式(print!()等)

#### 关于内核层和架构层的连接：

内核层和架构层现在的连接只有addr_space(地址空间模块)，连接方式是通过rust的trait特性。

addr_space中定义了PageMode trait,该trait定义页表的一系列属性，包括初始化页表，建立页表的一系列接口,然后利用该trait类型建立地址空间结构和相应对外接口，具体例子如下：

```
//kernel/addr_space
pub trait PageMode: Copy {
    const FRAME_SIZE_BITS: usize;
    const PPN_BITS: usize;
    const MAX_LEVEL:u8;
    fn get_align_for_level(level: u8) -> usize;
    fn visit_levels_until(level: u8) -> &'static [u8];
    ...
}
pub struct PagedAddrSpace<M: PageMode> {
    root_frame: PhysFrame,
    frames: Vec<PhysFrame>,
    regions: Vec<PageRegion<M>>,
    page_mode: marker::PhantomData<M>,
}
impl PageAddrSpace{
	pub fn try_new_in();
	pub fn root_page_number(&self);
	pub fn allocate_map(&mut self, vpn: VirtPageNum, pfs: Vec<PhysFrame>, n: usize, flags: M::Flags)	
}
```

之后arch需要具体实现一个架构相关的paging系统并且定义实现Page Mode Trait的相应类型

### 目前系统的缺陷反思

#### Capability机制的实现？

系统准备利用rust的ownership机制实现capability机制，即任何一种资源(物理内存，驱动，地址空间，线程)的ownership都属于相应进程，利用rust的特性这一点可以很容易的保证，但是资源应该如何传递出去并且传递出去后应该如何使用和保存是一个问题。

#### ADDR_SPACE的分层

现在的地址空间分层理论上很简洁，但是实际上分层没办法做到非常干净，比如kernel内部还是必须要了解相应架构page table entry的flag才能对相应的架构做到很好控制，分层抽象必定会带来灵活性的损失，该模块是否有更好的实现方式还值得思考。

### 接下来的目标进度

- 完成easy_fs的移植，实现读取用户程序
- 完成进程，线程，相应的切换模块
- 完成简单的系统调用
- 利用已有条件完成创建，运行用户进程