# lec1: 操作系统概述

---

## **提前准备**

（请在上课前完成）

* 完成lec1的视频学习和提交对应的在线练习
* git pull ucore\_os\_lab, ucore\_os\_docs, os\_tutorial\_lab, os\_course\_exercises in github repos。这样可以在本机上完成课堂练习。
* 知道OS课程的入口网址，会使用在线视频平台，在线练习/实验平台，在线提问平台\(piazza\)
  * [http://os.cs.tsinghua.edu.cn/oscourse/OS2019spring](http://os.cs.tsinghua.edu.cn/oscourse/OS2019spring)


* 会使用linux shell命令，如ls, rm, mkdir, cat, less, more, gcc等，也会使用linux系统的基本操作。
* 在piazza上就学习中不理解问题进行提问。



# 思考题

## 填空题

* 当前常见的操作系统主要用 C, C++, 汇编 编程语言编写。
* "Operating system"这个单词起源于 Operator 。
* 在计算机系统中，控制和管理 硬件资源 、有效地组织 应用程序 运行的系统软件称作 操作系统 。
* 允许多用户将若干个作业提交给计算机系统集中处理的操作系统称为 多道 操作系统。
* 你了解的当前世界上使用最多的操作系统是 Linux 。
* 应用程序通过 系统调用 接口获得操作系统的服务。
* 现代操作系统的特征包括 并发 ， 共享 ， 虚拟 ，异步 。
* 操作系统内核的架构包括 分层结构 ， 微内核结构 ， 外核结构 。


## 问答题

- 请总结你认为操作系统应该具有的特征有什么？并对其特征进行简要阐述。

操作系统应该具有的特征有并发、共享、虚拟、异步和易用。

并发要求操作系统能够在一段时间内管理多个程序的处理，并且兼顾每个程序的性能效率和公平进行权衡。

共享要求实现不同的应用程序能够不出错的共享硬件资源。

虚拟要求操作系统对硬件资源进行一定程度的抽象，让每个程序认为自己独占了整个计算机的资源。同时应用程序切换的速度应该足够快。

异步要求程序在相同的运行环境中要保证运行的结果是相同的。而且程序在运行过程中由于不断切换程序，需要保存一定的信息用于恢复运行。

易用是我新加入的一个特征，在PC普及的年代，很多人并没有足够的系统和计算机的知识，但操作系统应该能让所有人以最小的代价学会使用计算机，这就要求操作系统有易用的特征。

- 为什么现在的操作系统基本上用C语言来实现？为什么没有人用python，java来实现操作系统？

C语言相比python和java是一种更底层的语言技术，在效率和性能上要比python和java等语言表现更好，同时对硬件的调用也有着更好的支持。

同时在C语言中，程序员可以对所有涉及的内存、变量的资源有更好的掌握，如内存的申请与释放等，这些在python和java中都是对程序员不透明的，而在操作系统这样一个大型的复杂软件中，需要程序员代码的运行处于程序员的掌握之中。

---

## 可选练习题

---

- 请分析并理解[v9\-computer](https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/docs/v9_computer.md)以及模拟v9\-computer的em.c。理解：在v9\-computer中如何实现时钟中断的；v9 computer的CPU指令，关键变量描述有误或不全的情况；在v9\-computer中的跳转相关操作是如何实现的；在v9\-computer中如何设计相应指令，可有效实现函数调用与返回；OS程序被加载到内存的哪个位置,其堆栈是如何设置的；在v9\-computer中如何完成一次内存地址的读写的；在v9\-computer中如何实现分页机制。


- 请编写一个小程序，在v9-cpu下，能够输出字符


- 输入的字符并输出你输入的字符


- 请编写一个小程序，在v9-cpu下，能够产生各种异常/中断


- 请编写一个小程序，在v9-cpu下，能够统计并显示内存大小



- 请分析并理解[RISC-V CPU](http://www.riscvbook.com/chinese/)以及会使用模拟RISC\-V(简称RV)的qemu工具。理解：RV的特权指令，CSR寄存器和在RV中如何实现时钟中断和IO操作；OS程序如何被加载运行的；在RV中如何实现分页机制。
  - 请编写一个小程序，在RV下，能够输出字符
  - 输入的字符并输出你输入的字符
  - 请编写一个小程序，在RV下，能够产生各种异常/中断
  - 请编写一个小程序，在RV下，能够统计并显示内存大小

## 参考资料
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
 - [v9 cpu architecture](https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/docs/v9_computer.md)
 - [RISC-V cpu architecture](http://www.riscvbook.com/chinese/)
 - [OS相关经典论文](https://github.com/chyyuu/aos_course_info/blob/master/readinglist.md)
