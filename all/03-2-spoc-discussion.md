# lec6 SPOC思考题


NOTICE
- 有"w3l2"标记的题是助教要提交到学堂在线上的。
- 有"w3l2"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。

## 与视频相关思考题

### 6.1	非连续内存分配的需求背景
 1. 为什么要设计非连续内存分配机制？
  + 可以更好的利用内存空间以解决之前连续内存存储中出现的碎片问题

 2. 非连续内存分配中内存分块大小有哪些可能的选择？大小与大小是否可变?
  + 有段式、页式和段页式结合的方式
  + 段式和段页式的分块可以变化，页式的无法改变

 3. 为什么在大块时要设计大小可变，而在小块时要设计成固定大小？小块时的固定大小可以提供多种选择吗？
  + 大块相比小块灵活性较差，若固定大小，则可能会导致很大的内碎片，而小块的即使只有一个字节使用，也不会过多的浪费内存空间。

### 6.2	段式存储管理
 1. 什么是段、段基址和段内偏移？
  + 段是一段连续的内存空间，一个段内存储同一类型的信息，如CS段中存储代码，SS段中存储堆栈。
  + 段基址是段的起始地址。
  + 段内偏移是某一个具体的地址在段内的偏移，由段基址和段内偏移相加可以得到线性地址。

 2. 段式存储管理机制的地址转换流程是什么？为什么在段式存储管理中，各段的存储位置可以不连续？这种做法有什么好处和麻烦？
  + 地址转换流程：
    + 通过CS、DS、SS等寄存器获得对应的段基址的index
    + 在GDT或LDT中得到该index对应的段的信息，包括段基址和段的最大长度。
    + 判断段内偏移是否大于段的最大长度，若大于则报内存访问异常错误。
    + 若是合法的偏移，则将段基址和段内偏移相加得到线性地址
  + 段的存储位置可以不连续，因为都是依靠段内偏移和段基址进行访址，不会进行跨段的访问，所以只需要能够得到段基址和段内偏移就可以实现对地址的访问，而不需要要求段之间连续。
  + 好处：实现了非连续内存空间的存储，可以更好地利用内存空间。
  + 麻烦：增加了访址的开销，需要首先在内存中查找段基址，并进行合法性的判断。


### 6.3	页式存储管理
 1. 什么是页（page）、帧（frame）、页表（page table）、存储管理单元（MMU）、快表（TLB, Translation Lookaside Buffer）和高速缓存（cache）？
  + 页(page)：虚拟地址的基本单位
  + 帧(frame)：是物理地址划分的基本单位，与page的大小相同
  + 页表(page table)：是存储虚拟地址到物理地址对应原则的表，以虚拟地址为index，可以查找到对应的物理地址和一些控制信息。
  + 快表(TLB)：是页表的cache，依据局部性原理将页表中一些表项存储在CPU中加快访问速度。
  + 存储管理单元(MMU)：用于将虚拟地址转换为物理地址并在TLB缺失之后进行填充。同时还有这内存保护的作用。
 2. 页式存储管理机制的地址转换流程是什么？为什么在页式存储管理中，各页的存储位置可以不连续？这种做法有什么好处和麻烦？
  + 页式存储管理机制的地址转换流程：
    + 首先确定页和帧的大小，以4Kb为例，则虚拟地址后12位为页内偏移，以前面所有的位数作为index在页表和TLB中同步查找。
    + 若TLB中找到则获得对应的物理地址的前几位，与页内偏移组合到一起得到对应的物理地址。若TLB中没有找到，则在内存中获得后将该表项依据一定规则填充到TLB中。
  + 不连续的原因：物理地址和虚拟地址都以很小的单位进行划分，每个虚拟的地址都可以通过页表对应到物理地址，所以不需要物理地址存储位置连续。
  + 好处：可以进行不连续的内存存储，提高内存利用率，同时可以通过页式存储提供给应用程序大于实际物理内存的存储空间，最后还可以进行物理内存的权限控制。
  + 麻烦：加大了内存访问的开销，内存访问时间增加。同时页表可能会很大，占用很多内存空间。


### 6.4	页表概述
 1. 每个页表项有些什么内容？有哪些标志位？它们起什么作用？
  + 存在位：一个逻辑页号是否有一个物理页号与之对应
  + 修改位：对应的页中的内容是否被修改
  + 引用位：过去一段时间内是否有过对它的引用，是否访问过该页中的某一个存储单元。
 2. 页表大小受哪些因素影响？
  + 受虚拟地址的大小影响
  + 受页表项的大小的影响


### 6.5	快表和多级页表
 1. 快表（TLB）与高速缓存（cache）有什么不同？
  + 快表是对页表的快速缓存，高速缓存中暂存的是内存中的数据。
  + 快表的填充与更换需要通过中断实现，高速缓存不需要。
 2. 为什么快表中查找物理地址的速度非常快？它是如何实现的？为什么它的的容量很小？
  + 因为快表位于CPU内部，访问快表的开销很小，同时TLB是由关联存储器实现的，可以很快的通过index找到对应的表项。
  + 由于在CPU内部，所以无法做的过大，同时关联存储器本身也对大小有着一定的限制，无法做的很大。
 3. 什么是多级页表？多级页表中的地址转换流程是什么？多级页表有什么好处和麻烦？
  + 多级页表是设置多个页表，由虚拟的地址的不同位置的bit位分别在逐层的在多个页表中查找。
  + 转换流程：
    + 首先通过第一层的index在第一曾的页表中查找得到对应的下一个页表的基地址。
    + 再由下一层的index找到对应的表项，进而逐层的进行查找。直到最终得到对应的物理地址。
  + 好处：可以节省内存空间。
  + 麻烦：加大了内存的访问开销。


### 6.6	反置页表
 1. 页寄存器机制的地址转换流程是什么？
  + 页寄存器是把地址存在哈希表中的，转换流程是：
    + 计算逻辑页号的哈希值，访问对应寄存器，若页号与寄存器中存储的页号不匹配，沿冲突链表访问下去，直到匹配为止
 2. 反置页表机制的地址转换流程是什么？
  + 首先根据虚拟地址的index通过hash得到在反置页表中的位置。
  + 根据反置页表中的项的进程编号和虚拟地址index进行验证，若为目标项，则获得其中物理帧号，否则根据其中存储的下一个位置的指针访问下一个页表项，并进行验证直到找到对应的页表项。
 3. 反置页表项有些什么内容？
  + 进程编号、虚拟地址页号、物理地址帧号、冲突后的下一个页表项的index

### 6.7	段页式存储管理
 1. 段页式存储管理机制的地址转换流程是什么？这种做法有什么好处和麻烦？
  + 地址转换流程：
    + 首先通过段式存储的转换得到对应的线性地址（即虚拟地址）
    + 然后通过页式存储管理机制将虚拟地址翻译为物理地址。
  + 好处：可以实现对内存空间更加精确的控制，方便实现内存空间的共享等
  + 麻烦：地址转换过程更加复杂，开销加大。
 2. 如何实现基于段式存储管理的内存共享？
  + 在段表的基址中指向同一个页表，进而实现内存的共享。
 3. 如何实现基于页式存储管理的内存共享？
  + 不同的进程的页表中的某几个页表项指向相同的物理基址。

## 个人思考题
（1） (w3l2) 请简要分析64bit CPU体系结构下的分页机制是如何实现的
  + 参考[https://blog.csdn.net/gatieme/article/details/52403013](https://blog.csdn.net/gatieme/article/details/52403013)
  + Linux内核中采用四级分页机制，即：
    + 页全局目录（Page Global Directory）
    + 页上级目录（Page Upper Directory）
    + 页中间目录（Page Middle Directory
    + 页表（Page Table）
  + 页全局目录包含若干页上级目录的地址；
  + 页上级目录又依次包含若干页中间目录的地址；
  + 而页中间目录又包含若干页表的地址；
  + 每一个页表项指向一个页框。
  + 因此线性地址因此被分成五个部分，而每一部分的大小与具体的计算机体系结构有关。

## 小组思考题
（1）(spoc) 某系统使用请求分页存储管理，若页在内存中，满足一个内存请求需要150ns (10^-9s)。若缺页率是10%，为使有效访问时间达到0.5us(10^-6s),求不在内存的页面的平均访问时间。请给出计算步骤。

设不在内存的页面的平均访问时间为$x$，可以列方程：
$$
  0.9 \times 0.15 + 0.1 \times x = 0.5
$$
解方程得到：$x = 3.65us$



（2）(spoc) 有一台假想的计算机，页大小（page size）为32 Bytes，支持32KB的虚拟地址空间（virtual address space）,有4KB的物理内存空间（physical memory），采用二级页表，一个页目录项（page directory entry ，PDE）大小为1 Byte,一个页表项（page-table entries
PTEs）大小为1 Byte，1个页目录表大小为32 Bytes，1个页表大小为32 Bytes。页目录基址寄存器（page directory base register，PDBR）保存了页目录表的物理地址（按页对齐）。

PTE格式（8 bit） :
```
  VALID | PFN6 ... PFN0
```
PDE格式（8 bit） :
```
  VALID | PT6 ... PT0
```
其
```
VALID==1表示，表示映射存在；VALID==0表示，表示映射不存在。
PFN6..0:页帧号
PT6..0:页表的物理基址>>5
```
在[物理内存模拟数据文件](./03-2-spoc-testdata.md)中，给出了4KB物理内存空间的值，请回答下列虚地址是否有合法对应的物理内存，请给出对应的pde index, pde contents, pte index, pte contents。
```
1) Virtual Address 6c74
   Virtual Address 6b22
2) Virtual Address 03df
   Virtual Address 69dc
3) Virtual Address 317a
   Virtual Address 4546
4) Virtual Address 2c03
   Virtual Address 7fd7
5) Virtual Address 390e
   Virtual Address 748b
```

比如答案可以如下表示： (注意：下面的结果是错的，你需要关注的是如何表示)
```
Virtual Address 7570:
  --> pde index:0x1d  pde contents:(valid 1, pfn 0x33)
    --> pte index:0xb  pte contents:(valid 0, pfn 0x7f)
      --> Fault (page table entry not valid)

Virtual Address 21e1:
  --> pde index:0x8  pde contents:(valid 0, pfn 0x7f)
      --> Fault (page directory entry not valid)

Virtual Address 7268:
  --> pde index:0x1c  pde contents:(valid 1, pfn 0x5e)
    --> pte index:0x13  pte contents:(valid 1, pfn 0x65)
      --> Translates to Physical Address 0xca8 --> Value: 16
```

[链接](https://piazza.com/class/i5j09fnsl7k5x0?cid=664)有上面链接的参考答案。请比较你的结果与参考答案是否一致。如果不一致，请说明原因。

**My answer**

页表基地址为0x220，即为下面的物理地址。
```
page 11: da f7 f2 a8 96 c5 9d 94 c8 b9 7f c4 98 e5 7f 7f d3 a1 82 8f a6 fb bf f0 7f 84 d2 a0 88 80 c9 92 
```

```
Virtual Address 6c74:
  --> pde index: 0x1b pde contents:(valid 1, pfn 0x20)
    --> pte index: 0x3 pte contents:(valid 1, pfn 0x20)
      --> Translates to Physical Address 0x414 --> Value: 127
Virtual Address 6b22:
  --> pde index: 0x1a pde contents:(valid 1, pfn 0x52)
    --> pte index: 0x19 pte contents:(valid 1, pfn 0x52)
      --> Translates to Physical Address 0xa42 --> Value: 127
Virtual Address 3df:
  --> pde index: 0x0 pde contents:(valid 1, pfn 0x5a)
    --> pte index: 0x1e pte contents:(valid 1, pfn 0x5a)
      --> Translates to Physical Address 0xb5f --> Value: 127
Virtual Address 69dc:
  --> pde index: 0x1a pde contents:(valid 1, pfn 0x52)
    --> pte index: 0xe pte contents:(valid 1, pfn 0x52)
      --> Translates to Physical Address 0xa5c --> Value: 223
Virtual Address 317a:
  --> pde index: 0xc pde contents:(valid 1, pfn 0x18)
    --> pte index: 0xb pte contents:(valid 1, pfn 0x18)
      --> Translates to Physical Address 0x31a --> Value: 127
Virtual Address 4546:
  --> pde index: 0x11 pde contents:(valid 1, pfn 0x21)
    --> pte index: 0xa pte contents:(valid 1, pfn 0x21)
      --> Translates to Physical Address 0x426 --> Value: 153
Virtual Address 2c03:
  --> pde index: 0xb pde contents:(valid 1, pfn 0x44)
    --> pte index: 0x0 pte contents:(valid 1, pfn 0x44)
      --> Translates to Physical Address 0x883 --> Value: 127
Virtual Address 7fd7:
  --> pde index: 0x1f pde contents:(valid 1, pfn 0x12)
    --> pte index: 0x1e pte contents:(valid 1, pfn 0x12)
      --> Translates to Physical Address 0x257 --> Value: 127
Virtual Address 390e:
  --> pde index: 0xe pde contents:(valid 0, pfn 0x7f)
      --> Fault (page directory entry not valid)
Virtual Address 748b:
  --> pde index: 0x1d pde contents:(valid 1, pfn 0x0)
    --> pte index: 0x4 pte contents:(valid 1, pfn 0x0)
      --> Translates to Physical Address 0xb --> Value: 127
```

（3）请基于你对原理课二级页表的理解，并参考Lab2建页表的过程，设计一个应用程序（可基于python、ruby、C、C++、LISP、JavaScript等）可模拟实现(2)题中描述的抽象OS，可正确完成二级页表转换。

[链接](https://piazza.com/class/i5j09fnsl7k5x0?cid=664)有上面链接的参考答案。请比较你的结果与参考答案是否一致。如果不一致，提交你的实现，并说明区别。

（4）假设你有一台支持[反置页表](http://en.wikipedia.org/wiki/Page_table#Inverted_page_table)的机器，请问你如何设计操作系统支持这种类型计算机？请给出设计方案。

 (5)[X86的页面结构](http://os.cs.tsinghua.edu.cn/oscourse/OS2019spring/lecture06)
---

## 扩展思考题

阅读64bit IBM Powerpc CPU架构是如何实现[反置页表](http://en.wikipedia.org/wiki/Page_table#Inverted_page_table)，给出分析报告。


## interactive　understand VM

[Virtual Memory with 256 Bytes of RAM](http://blog.robertelder.org/virtual-memory-with-256-bytes-of-ram/)：这是一个只有256字节内存的一个极小计算机系统。按作者的[特征描述](https://github.com/RobertElderSoftware/recc#what-can-this-project-do)，它具备如下的功能。
 - CPU的实现代码不多于500行；
 - 支持14条指令、进程切换、虚拟存储和中断；
 - 用C实现了一个小的操作系统微内核可以在这个CPU上正常运行；
 - 实现了一个ANSI C89编译器，可生成在该CPU上运行代码；
 - 该编译器支持链接功能；
 - 用C89, Python, Java, Javascript这4种语言实现了该CPU的模拟器；
 - 支持交叉编译；
 - 所有这些只依赖标准C库。
 
针对op-cpu的特征描述，请同学们通过代码阅读和执行对自己有兴趣的部分进行分析，给出你的分析结果和评价。
