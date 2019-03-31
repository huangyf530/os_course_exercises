# lec10: lab3 SPOC思考题

## 视频相关思考题
---
### 10.1 实验目标：虚存管理
---

1. 缺页和页访问非法的返回地址有什么不同？
    + 缺页返回地址为该条语句的起始地址，即再次执行该语句。
    + 页访问非法地址后直接结束该进程。

2. 虚拟内存管理中是否用到了段机制？
    + 没有

3. ucore如何知道页访问异常的地址？
    + 通过在vma中查找，vma表示的是合法的虚拟地址的集合。


### 10.2 回顾历史和了解当下
---

1. 中断处理例程的段表在GDT还是LDT？
    + GDT or LDT

2. 物理内存管理的数据结构在哪？
    + 通过list_entry和page的数据结构实现，在pmm.h文件中

3. 页表项的结构？
    + 31-12: 页帧号
    + 11-9: 保留位
    + 8: G是否全局
    + 7: PAT
    + 6: 修改位
    + 5: 访问位
    + 4: 是否缓存(Cache-enabled)
    + 3: write-through
    + 2: U/K 用户权限位
    + 1: writable，是否可写
    + 0: P位，存在位

4. 页表项的修改代码？
 
5. 如何设置一个虚拟地址到物理地址的映射关系？
    + 首先在页目录项中找到对应的页表项
    + 修改该页表项对应的物理页帧号
 
6. 为了建立虚拟内存管理，需要在哪个数据结构中表示“合法”虚拟内存
    + vma_struct和mm_struct

 
### 10.3 处理流程、关键数据结构和功能
---

1. swap_init()做了些什么？
    + 初始化内存分配算法

2. vmm_init()做了些什么？
    + 初始化vma_struct结构

3. vma_struct数据结构的功能？
    + 标记合法的虚拟地址

4. mmap_list是什么列表？
    + 是vma_struct链表

5. 外存中的页面后备如何找到？
    + 在页表项中记录磁盘编号

6. vma_struct和mm_struct的关系是什么？
    + mm_struct链接多个vma_struct

7. 画数据结构图，描述进程的虚拟地址空间、页表项、物理页面和后备页面的关系；

### 10.4 页访问异常
---

1. 页面不在内存和页面访问非法的处理中有什么区别？对应的代码区别在哪？
    + 页面不在内存意味这出现缺页异常，对应的为缺页异常的中断处理例程
    + 页面访问非法地址会直接中断程序的运行。

2. find_vma()做了些什么？
    + 找到对应的虚拟地址的vma并判断是否合法
 
3. swapfs_read()做了些什么？
    + 读取对应的磁盘空间到对应的物理地址
 
4. 缺页时的页面创建代码在哪？
 
5. struct rb_tree数据结构的原理是什么？在虚拟管理中如何用它的？
 
6. 页目录项和页表项的dirty bit是何时，由谁置1的？
    + 在对应的物理页面中内容被修改时置1
 
7. 页目录项和页表项的access bit是何时，由谁置1的？
    + 在访问该物理页面后置1

### 10.5 页换入换出机制
---

1. 虚拟页与磁盘后备页面的对应有关系？
    + 有关系，虚拟页对应的页表项中会记录磁盘后备页面信息。
 
2. 如果在开始加载可执行文件时，如何改？
 
3. check_swap()做了些什么检查？
    + 检查缺页处理是否正确
    + 检查能否正确地实现物理内存到磁盘的读写
    + 检查页面置换算法正确性
 
4. swap_entry_t数据结构做什么用的？放在什么地方？
    + 用于定位磁盘中的围追。
    + 放在页表项中
 
5. 空闲物理页面的组织数据结构是什么？
 
6. 置换算法的接口数据结构？
    ```c
    struct swap_manager
    {
     const char *name;
     /* Global initialization for the swap manager */
     int (*init)            (void);
     /* Initialize the priv data inside mm_struct */
     int (*init_mm)         (struct mm_struct *mm);
     /* Called when tick interrupt occured */
     int (*tick_event)      (struct mm_struct *mm);
     /* Called when map a swappable page into the mm_struct */
     int (*map_swappable)   (struct mm_struct *mm, uintptr_t addr, struct Page *page, int swap_in);
     /* When a page is marked as shared, this routine is called to
      * delete the addr entry from the swap manager */
     int (*set_unswappable) (struct mm_struct *mm, uintptr_t addr);
     /* Try to swap out a page, return then victim */
     int (*swap_out_victim) (struct mm_struct *mm, struct Page **ptr_page, int in_tick);
     /* check the page relpacement algorithm */
     int (*check_swap)(void);     
    };
    ```

================


## 小组思考题
---
(1)请参考lab3_result的代码，思考如何在lab3_results中实现clock算法，给出你的概要设计方案。可4人一个小组。要求说明你的方案中clock算法与LRU算法上相比，潜在的性能差异性。进而说明在lab3的LRU算法实现的可能性评价（给出理由）。

(2) 理解内存访问的异常。在x86中内存访问会受到段机制和页机制的两层保护，请基于lab3_results的代码（包括lab1的challenge练习实现），请实践并分析出段机制和页机制各种内存非法访问的后果。，可4人一个小组，，找出尽可能多的各种内存访问异常，并在代码中给出实现和测试用例，在执行了测试用例后，ucore能够显示出是出现了哪种异常和尽量详细的错误信息。请在说明文档中指出：某种内存访问异常的原因，硬件的处理过程，以及OS如何处理，是否可以利用做其他有用的事情（比如提供比物理空间更大的虚拟空间）？哪些段异常是否可取消，并用页异常取代？

## 课堂实践练习

请分析ucore中与物理内存管理和虚拟存储管理相关的数据结构组织；分析访问这些数据结构的函数，说明其对存储管理相关数据结构的修改情况；最后通过一个数据结构图示，描述进程的虚拟地址空间、页表项、物理页面和后备页面的关系。

 * struct Page
 * struct mm_struct
 * struct vma_struct
 * struct swap_entry

## v9-cpu相关
---
(1)分析并编译运行v9-cpu git repo的testing branch中的,root/etc/os_lab2.c os_lab3.c os_lab3_1.c,理解虚存机制是如何在v9-cpu上实现的，思考如何实现clock页替换算法，并给出你的概要设计方案。

(2)分析并编译运行v9-cpu git repo的testing branch中的,root/etc/os_lab2.c os_lab3.c os_lab3_1.c，理解内存访问异常的各种情况，并给出你的分析结果。
