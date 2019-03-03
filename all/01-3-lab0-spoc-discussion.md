# lec2：lab0 SPOC思考题

## **提前准备**
（请在上课前完成，option）

- 完成lec2的视频学习
- git pull ucore_os_lab, os_tutorial_lab, os_course_exercises  in github repos。这样可以在本机上完成课堂练习。
- 了解代码段，数据段，执行文件，执行文件格式，堆，栈，控制流，函数调用,函数参数传递，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在不同操作系统（如linux, ucore,etc.)与不同硬件（如 x86, riscv, v9-cpu,etc.)中是如何相互配合来体现的。
- 安装好ucore实验环境，能够编译运行ucore labs中的源码。
- 会使用linux中的shell命令:objdump，nm，file, strace，gdb等，了解这些命令的用途。
- 会编译，运行，使用v9-cpu的dis,xc, xem命令（包括启动参数），阅读v9-cpu中的v9\-computer.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
- 了解基于v9-cpu的执行文件的格式和内容，以及它是如何加载到v9-cpu的内存中的。
- 在piazza上就学习中不理解问题进行提问。

---

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

*1. 进程操作：需要支持时钟中断和I/O中断的支持以及相关的特权指令*

*2. 虚存系统：需要TLB相关的支持、中断和特权指令*

*3. 文件系统：需要磁盘的访问、将磁盘中数据读取到内存中的相关的硬件支持和特权指令*

- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？

*x86的实模式中，程序使用的地址为物理地址，用户可以直接修改物理地址中的内容，而在保护模式中，程序使用的地址为虚拟地址，需要经过操作系统转换为物理地址，防止一些程序直接修改系统的物理地址空间内容。*

*物理地址：在地址总线中传递的地址，直接对应内存中的一块内容*

*线性地址：也叫虚拟地址，是逻辑地址和物理地址之间转换的中间层，逻辑地址加上基地址之后就是线性地址*

*逻辑地址：在有地址变换的计算机中，仿内指令给出的地址（操作数）叫逻辑地址。*

- 你理解的risc-v的特权模式有什么区别？不同模式在地址访问方面有何特征？

*risc-v的特权模式有三种：分别是机器模式（Machine Mode）、监督模式（Supervisor Mode）和用户模式（User Mode）。一般应用程序运行在用户模式之中。机器模式和监督模式有着一个特权指令用于完成中断和异常的处理以及其他的功能。机器模式是在一个硬件线程中所能具有的最高权限的模式，机器模式用于实现中断和异常的处理。操作系统一般运行在监督模式中，且监督模式中有对基于页面的内存系统的支持。*

*机器模式通过直接指定物理地址访问，用户模式和监督模式使用虚拟地址，且用户模式下地址的访问受限。*

- 理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义
```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
```

*后面数字的含义是前面的标识占用的位数*

- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
请问执行上述指令后， intr的值是多少？

*instr的值为0x00008F000002FFFF*

### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

```
    cli                              # 禁止中断
    cld                              # 串流数据通过地址递增的方式传输

    # 下面代码负责初始化DS，ES和SS寄存器
    xorw %ax, %ax                    # 是ax寄存器变为0
    movw %ax, %ds                    # 给DS寄存器赋值为0
    movw %ax, %es                    # 给ES寄存器赋值为0
    movw %ax, %ss                    # 给SS寄存器赋值为0
```

2. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

*1. 用于定义一些常量。如`#define SECTSIZE        512`设置一个磁盘中一个扇区的大小*

*2. 用于构造一些常用的函数，方便之后的调用。*

#### reference
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
