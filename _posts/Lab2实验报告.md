# Lab2实验报告

## 实验思考题

### Thinking2.1

> - **在我们编写的程序中，指针变量中存储的地址是虚拟地址还是物理地址？**
> - **MIPS 汇编程序中lw, sw使用的是虚拟地址还是物理地址？**

- 由于CPU只会发出虚拟地址，而我们直接操作的是内核代码，所以使用的指针变量中存的也是虚拟地址
- lw, sw指令是访存指令，所以为物理地址

------

### Thinking2.2

> - **请从可重用性的角度，阐述用宏来实现链表的好处。**
> - **请你查看实验环境中的 /usr/include/sys/queue.h，了解其中单向链表与循环链表的实现，比较它们与本实验中使用的双向链表，分析三者在插入与删除操作上的性能差异。**

- 使用宏相当于将一个个功能封装起来，在需要时即可调用，宏的名字也清晰易懂。同时，由于操作系统中不只有page链表，还有调度链表等，因此此处定义的宏可用在多处链表中。此外，宏定义增强了代码的可扩展性，也便于修改。
- 三种链表差异
  - 单向链表：单向链表中每一项只能获取到后面一项的内容，所以若是在某一项后面插入，则可直接操作；而若是在某一项前面插入，或是删除某一项时，因为都需要找到前一项的内容，所以要从头开始遍历，时间复杂度较高。
  - 双向链表：双向链表中每一项可以获取到前和后两项的内容，因此插入和删除操作都可以直接实现；但若是在尾部插入，则需要遍历整个链表
  - 循环链表：循环链表也是单向的，因此时间复杂度基本与单向链表相同；但循环链表尾部直接指向头部，所以在尾部插入无需遍历

------

### Thinking2.3

> **请阅读 `include/queue.h` 以及 `include/pmap.h`, 将 `Page_list` 的结构梳理清楚，选择正确的展开结构**

```c
struct Page_list{
    struct {
        struct {
            struct Page *le_next;
            struct Page **le_prev;
        } pp_link;
        u_short pp_ref;
    }* lh_first;
}
```

- Page的总体结构如下

```c
struct Page {
    Page_LIST_entry_t pp_link;  /* free list link */
    u_short pp_ref;
};
```

- queue.h定义的内容如下

```c
#define LIST_ENTRY(type)                                                    \
        struct {                                                                \
                struct type *le_next;   /* next element */                      \
                struct type **le_prev;  /* address of previous next element */  \
        }
#define LIST_HEAD(name, type)                                               \
        struct name {                                                           \
                struct type *lh_first;  /* first element */                     \
        }
```

将上述宏定义代入至Page中，即可得到展开结构

------

### Thinking2.4

> **请你寻找上述两个 boot_\* 函数在何处被调用**

- boot_map_segment()函数在mips_vm_init()中被调用，将一级页表基地址 pgdir 对应的两级页表结构做区间地址映射
- boot_pgdir_walk()函数在boot_map_segment()中被调用，通过虚拟地址找到二级页表项

------

### Thinging2.5

> - 请阅读上面有关 R3000-TLB 的叙述，从虚拟内存的实现角度，阐述 ASID 的必要性
> - 请阅读《IDT R30xx Family Software Reference Manual》的 Chapter 6，结合 ASID 段的位数，说明 R3000 中可容纳不同的地址空间的最大数量

- ASID必要性
  - ASID可用来唯一标识进程，并为进程提供地址空间保护。当TLB工作时，需确保确保当前运行进程的ASID与虚拟页相关的ASID相匹配。
  - ASID允许TLB同时包含多个进程的条目。这样使每次选择一个页表时，TLB无需被冲刷或删除。

- 通过查阅资料可知，ASID在寄存器中占[11:6]共6位，因此最多能容纳地址空间的数量为64；

------

### Thinking2.6

> - **tlb_invalidate 和 tlb_out 的调用关系是怎样的？**
> - ***\*请用一句话概括 tlb_invalidate 的作用\****
> - ***\*逐行解释 tlb_out 中的汇编代码\****

- tlb_invalidate函数中调用了tlb_out

- 在页表更新（插入或移除）后更新TLB

- ```shell
  LEAF(tlb_out)
  //1: j 1b
  nop
      mfc0    k1,CP0_ENTRYHI   //从ENTRYHI寄存器中取出数据放到k1
      mtc0    a0,CP0_ENTRYHI   //将a0中的值放入ENTRYHI寄存器中
      nop
      // insert tlbp or tlbwi
      tlbp					 //根据EntryHi中的Key，查找TLB中与之对应的表项并将表项的索引存入Index寄存器
      nop
      nop
      nop
      nop
      mfc0    k0,CP0_INDEX	 //从INDEX寄存器中取出数据放到k0中
      bltz    k0,NOFOUND       //若k0小于0（即在TLB中未找到匹配项），则跳转到NOFOUND
      nop
      mtc0    zero,CP0_ENTRYHI //将ENTRYHI寄存器置0
      mtc0    zero,CP0_ENTRYLO0//将ENTRYLO寄存器置0
      nop
      tlbwi					 //以Index寄存器中的值为索引,将此时EntryHi与EntryLo的值写到索引指定的TLB表项中
      // insert tlbp or tlbwi
  NOFOUND:                     //若没找到，则执行下列步骤
  
      mtc0    k1,CP0_ENTRYHI   //将k1的值放入ENTRYHI寄存器中
  
      j   ra                   //返回调用处
      nop
  END(tlb_out)
  ```

------

### Thinking2.7

> **在现代的 64 位系统中，提供了 64 位的字长，但实际上不是 64 位页式存储系统。假设在 64 位系统中采用三级页表机制，页面大小 4KB。由于 64 位系统中字长为 8B，且页目录也占用一页，因此页目录中有 512 个页目录项，因此每级页表都需要 9 位。因此在 64 位系统下，总共需要 3 × 9 + 12 = 39 位就可以实现三级页表机制，并不需要 64 位。现考虑上述 39 位的三级页式存储系统，虚拟地址空间为 512 GB，若记三级页表的基地址为 PTbase ，请你计算：**
>
> - **三级页表页目录的基地址**
> - **映射到页目录自身的页目录项(自映射)**

- **PTbase + (PTbase >> 9)**：页目录的第一个页目录项记录的是第一个页表的页表项，PTbase >> 12表示第一个页表页的物理页号，一个页表项8B，所以记录第一个页表页的页表项相对于页表的偏移为PTbase >> 12 << 3 = (PTbase >> 9)，可得记录第一个页表页的页表项地址为PTbase + (PTbase >> 9)，即页目录地址
- **PTbase + (PTbase >> 9) + (PTbase >> 18)** ：已经得到页目录基址，只需计算出该页目录项相对于页目录基址的偏移。页目录项对应的是页表页，那么只需计算当前页表页的页表页号即可。已经计算出第一个页表页的页表项地址（即页目录基址）相对页表基址的偏移为(PTbase >> 9)，一个页表页4KB，所以页目录的页表页号为（（PTbase >> 9） >> 12），一个页表项8B，故偏移为((PTbase >> 9) >> 12) << 3，即自映射页目录项为PTbase + (PTbase >> 9) + (PTbase >> 18) 

------

### Thinking2.8

> **简单了解并叙述 X86 体系结构中的内存管理机制，比较 X86 和 MIPS 在内存管理上的区别**

- x86架构
  - 使用段页式内存管理机制
  - 寻址过程：CPU发出逻辑地址，取得段选择子，在GDT（全局表述符表）或LDT（局部表述符表）上取出段描述符，将取出的段描述符中的Base与逻辑地址中的offset相加即可得到线性地址，从线性地址中取出页表索引，从页目录中得到页表的实际起始地址，从线性地址中取出页面偏移地址，结合页表起始地址，取出实际页号，将实际页号与线性地址中的offset相加即可得到物理地址

- mips架构
  - 使用页式内存管理机制
  - 寻址过程：CPU发出逻辑地址[][][31:22]为一级页表偏移量，[21:12]为二级页表偏移量，[11:0]为页内偏移量；先通过一级页表基地址和一级页表项的偏移量，找到对应的一级页表项，得到对应的二级页表基地址的物理页号，再根据二级页表项的偏移量找到所需的二级页表项，进而得到对应物理页的物理页号，再与页内偏移量相加即可得到物理地址

------

## 实验难点图示

![](C:\Users\WYF\wyfame-github\img\in-post\page.png)

- 在进行地址索引时，先通过虚拟地址的高十位和页表基址找到对应的一级页表项，若权限位为1，则拿到二级页表的物理页号，通过二级页表偏移量（虚拟地址[21:12]）找到对应的二级页表项，若权限位为1，则拿到对应物理页的物理页号，与页内偏移量（虚拟地址[11:0]）相加得到物理地址。（若过程中页表项权限位为0，则发生中断）

相关函数：

```c
static Pte *boot_pgdir_walk(Pde *pgdir, u_long va, int create){
    /* Step 3: Get the page table entry for `va`, and return it. */
    Pde *pgdir_entry = pgdir + PDX(va);      				  //页表基址与虚拟地址高十位相加得到一级页表项所在位置
    if ((*pgdir_entry & PTE_V) == 0) {  	 				  //若对应的二级页表不存在
        if (create) {
            *pgdir_entry = PADDR(alloc(BY2PG, BY2PG, 1));	  //分配一页物理内存用于存放
            *pgdir_entry = (*pgdir_entry) | PTE_V | PTE_R;	  //设置有效位、可写位为1
        } else return 0; // exception
    }
    return ((Pte *)(KADDR(PTE_ADDR(*pgdir_entry)))) + PTX(va); //返回二级页表项的地址
}
```

## 体会与感想

- 本次实验学习了物理内存管理和虚拟内存管理相关的知识；难度较之前有一定程度的提升，需要自己逐步梳理相关的结构（如Page）、各种宏定义的用法、相应指针的值，同时还需要有着足够的理论知识作为基础。
- 此实验还帮助解决了我在理论课上页目录自映射遗留的问题
- 对c语言指针也有了更深刻的理解
