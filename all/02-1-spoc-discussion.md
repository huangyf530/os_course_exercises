# lec 3 SPOC Discussion

## **提前准备**
（请在上课前完成）


 - 完成lec3的视频学习和提交对应的在线练习
 - git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises  　in github repos。这样可以在本机上完成课堂练习。
 - 仔细观察自己使用的计算机的启动过程和linux/ucore操作系统运行后的情况。搜索“80386　开机　启动”
 - 了解控制流，异常控制流，函数调用,中断，异常(故障)，系统调用（陷阱）,切换，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在linux, ucore, v9-cpu中的os*.c中是如何具体体现的。
 - 思考为什么操作系统需要处理中断，异常，系统调用。这些是必须要有的吗？有哪些好处？有哪些不好的地方？
 - 了解在PC机上有啥中断和异常。搜索“80386　中断　异常”
 - 安装好ucore实验环境，能够编译运行lab8的answer
 - 了解Linux和ucore有哪些系统调用。搜索“linux 系统调用", 搜索lab8中的syscall关键字相关内容。在linux下执行命令: ```man syscalls```
 - 会使用linux中的命令:objdump，nm，file, strace，man, 了解这些命令的用途。
 - 了解如何OS是如何实现中断，异常，或系统调用的。会使用v9-cpu的dis,xc, xem命令（包括启动参数），分析v9-cpu中的os0.c, os2.c，了解与异常，中断，系统调用相关的os设计实现。阅读v9-cpu中的cpu.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
 - 在piazza上就lec3学习中不理解问题进行提问。

## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS
- x86中BIOS从磁盘读入的第一个扇区是是什么内容？为什么没有直接读入操作系统内核映像？
	- 第一个扇区的内容是bootloader，是操作系统的加载程序。
    - 不直接读入操作系统内核映像的原因有两个，首先是BIOS最多只能读取一个扇区的内容，一个扇区大小为512字节，而一个操作系统内核映像远远大于512字节，所以无法直接通过BIOS将操作系统内核读入。其次是不同的磁盘的文件结构不完全相同，需要通过bootloader来确定磁盘的文件格式。
- 比较UEFI和BIOS的区别。
    - UEFI从通电到关机可以分为7个阶段：SEC（安全验证）、PEI（EFI前期初始化）、DXE（驱动执行环境）、BDS（启动设备选择）、TSL（操作系统加载前期）、RT（Run Time）、AL（系统灾难恢复期）。而BIOS较为简单，BIOS加载bootloader、bootloader加载OS。
    - UEFI想比BIOS更加安全，在SEC阶段并不对memory等硬件进行初始化，会先对PEI的进行验证，在通过验证之后才可以执行PEI阶段，初始化memory、I/O控制器等硬件
    - *关于UEFT启动的详细过程可以参考[https://blog.csdn.net/gjq_1988/article/details/50593564](https://blog.csdn.net/gjq_1988/article/details/50593564)*
- 理解rcore中的Berkeley BootLoader (BBL)的功能。
    - bbl执行的功能如下：
    - 选择一个物理进程（hart）作为主进程，其他物理进程进入sleep状态
    - 对上一个步骤中传过来的device tree进行筛选信息，将OS不需要的信息过滤掉
    - 其他harts被唤醒来初始化他们的PMP，trap handler并进入supervisor mode
    - CSR寄存器被初始化，以便Linux使用
    - 物理内存保护机制（PMP）启动，以使Linux可以使用所有内存
    - 执行mret进入到supervisor mode

## 3.2 系统启动流程

- x86中分区引导扇区的结束标志是什么？
    - 0x55AA
- x86中在UEFI中的可信启动有什么作用？
    - 防止一些即插即用的硬件中的操作系统对原系统或硬件系统造成危害，如可以通过U盘启动后修改原磁盘上的内容，进而造成数据的丢失。
- RV中BBL的启动过程大致包括哪些内容？
    - 选择一个物理进程（hart）作为主进程，其他物理进程进入sleep状态
    - 对上一个步骤中传过来的device tree进行筛选信息，将OS不需要的信息过滤掉
    - 其他harts被唤醒来初始化他们的PMP，trap handler并进入supervisor mode
    - CSR寄存器被初始化，以便Linux使用
    - 物理内存保护机制（PMP）启动，以使Linux可以使用所有内存
    - 执行mret进入到supervisor mode

## 3.3 中断、异常和系统调用比较
- 什么是中断、异常和系统调用？
    - 中断是由外部设备引发的。
    - 异常式应用程序运行时出现的错误需要进行单独处理。
    - 系统调用是由用户程序主动发起，方便应用程序进行一些操作，如I/O操作。
-  中断、异常和系统调用的处理流程有什么异同？
    - 首先都是去查看中断向量表。
    - 若为中断则跳转到中断处理程序（设备驱动）
    - 若为异常，则跳转到异常处理例程
    - 若为系统调用，由于系统调用有很多，所以首先查看系统调用表，再跳转到对应系统调用的处理程序位置。
    - 处理完毕后都恢复原程序状态，恢复源程序的运行。
- 以ucore/rcore lab8的answer为例，ucore的系统调用有哪些？大致的功能分类有哪些？
    - ucore的系统调用有22个，分别是sys_exit, sys_fork, sys_wait, sys_exec, sys_yield, sys_kill, sys_getpid, sys_putc, sys_pgdir, sys_gettime, sys_lab6_set_priority, sys_sleep, sys_open, sys_close, sys_read, sys_write, sys_seek, sys_fstat, sys_fsync, sys_getcwd, sys_getdirentry, sys_dup。
    - 大致功能分类为两种：第一种为与进程和文件的执行相关。如sys_exit, sys_wait等等。第二种为与文件或外设的读写有关，如sys_open, sys_close, sys_read, sys_write等。

## 3.4 linux系统调用分析
- 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(仅实践，不用回答)
- 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(仅实践，不用回答)


## 3.5 ucore/rcore系统调用分析 （扩展练习，可选）
-  基于实验八的代码分析ucore的系统调用实现，说明指定系统调用的参数和返回值的传递方式和存放位置信息，以及内核中的系统调用功能实现函数。
- 以ucore/rcore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
- 以ucore/rcore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。

 
## 3.6 请分析函数调用和系统调用的区别
- 系统调用与函数调用的区别是什么？
    - 在汇编层面：系统调用通过int，syscall指令进行，返回通过eret指令。而函数调用通过call指令调用，通过ret指令返回。
    - 系统调用会进行进程的切换，调用的函数有自己的堆栈。函数调用不会进行进程的切换，与调用者共享堆栈。
- 通过分析x86中函数调用规范以及`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？
    - `int`通过制定中断码在中断向量表中查找对应的处理程序所在的位置，进入中断处理程序。`int`指令会首先将标志寄存器入栈，使IF和TF为0，之后CS、IP入栈，查找中断向量表。`eret`将IP、CS和标志寄存器恢复，并返回到源程序继续处理。
    - `call`通过制定地址调用子函数，并将`call`语句的下一条语句压入栈中，在子程序执行完毕之后，通过`ret`指令返回到`call`语句的下一条语句。
    - 从上面可以看出函数调用与调用者使用的是同一套堆栈，而系统调用进行了进程的切换，使用的是系统调用函数自己的堆栈系统。
- 通过分析RV中函数调用规范以及`ecall`、`eret`、`jal`和`jalr`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？
    - 函数调用规范
        + 将参数保存在函数能够访问到的位置
        + 通过`jal`或`jalr`指令跳转到函数开始的位置
        + 获取函数需要的局部资源，按需保存寄存器
        + 执行函数中的指令
        + 将返回值存储到调用者能够访问到的位置，恢复寄存器，释放局部存储资源
        + 通过`ret`指令返回
    - `ecall`指令用于向运行时环境发出请求，如进行系统调用，此时切换到supervisor mode
    - `eret`指令返回中断或异常发生的位置，并切换会user mode
    - `jal`跳转到特定的立即数地址并将返回地址保存在$ra寄存器中
    - `jalr`跳转到特定的寄存器中所指的地址，并将返回地址保存在$ra寄存器中


## 课堂实践 （在课堂上根据老师安排完成，课后不用做）
### 练习一
通过静态代码分析，举例描述ucore/rcore键盘输入中断的响应过程。

### 练习二
通过静态代码分析，举例描述ucore/rcore系统调用过程，及调用参数和返回值的传递方法。
