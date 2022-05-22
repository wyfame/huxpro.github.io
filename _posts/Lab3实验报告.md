# Lab3实验报告

## 实验思考题

### Thinking3.1

> **为什么envid2env 中需要判断e->env_id != envid 的情况？如果没有这步判断会发生什么情况？**

- 我们最初给系统分配了1024的进程控制块，并分配不同的id以标识它们。在envid2env函数中使用ENVX宏取出的是id的低十位，用这十位作为索引去获取相应的控制块。但Id中[16:11]为asid号，当发生进程替换时会出现id后十位相同但asid不同的情况，因此需要加以比较
- 没有这步判断会得到一个错误的进程块，会导致之后的操作出现异常

### Thinking3.2

> **结合include/mmu.h 中的地址空间布局，思考env_setup_vm 函数：**
>
> **• UTOP 和ULIM 的含义分别是什么，UTOP 和ULIM 之间的区域与UTOP以下的区域相比有什么区别？**
>
> **• 请结合系统自映射机制解释代码中pgdir[PDX(UVPT)]=env_cr3的含义。**
>
> **• 谈谈自己对进程中物理地址和虚拟地址的理解。**

- ULIM是用户空间和内核空间的分界线；UTOP是用户空间只读区和可读写区的分界线；二者之间的区域只读
- 这句代码完成了UVPT区域的映射，将UVPT对应的页目录项映射到了自身的物理地址
- 一个进程的虚拟地址空间是连续的（像一个结构体）；页表和MMU可以将虚拟地址映射到不同的物理地址

### Thinking3.3

> **找到 user_data 这一参数的来源，思考它的作用。没有这个参数可不可以？为什么？（可以尝试说明实际的应用场景，举一个实际的库中的例子）**

- user_data是一个函数指针，用于给外层函数传递信息等。在这里的作用是提供了env数组
- 不可以没有这个参数，没有的话可能会导致执行错误
- 比如sort类函数，可以传入一个cmp规定排序方法

### Thinking3.4

>  **结合load_icode_mapper 的参数以及二进制镜像的大小，考虑该函数可能会面临哪几种复制的情况？你是否都考虑到了？**

![image-20220509210312318](C:\Users\WYF\AppData\Roaming\Typora\typora-user-images\image-20220509210312318.png)

- 开头页不对齐，存在上图所示的`offset`，此时对第一页有两种处理
  - 如果`BY2PG - offset < bin_size`，则加载`BY2PG - offset`大小的文件
  - 否则直接加载`bin_size`大小的文件

- 结尾页不对齐，如上图所示的`va+i`的位置
  - 如果`sgsize - i < BY2PG`，则清零`sgsize - i`大小的内容
  - 否则清零`BY2PG`大小的内容

### Thinking3.5

> **•** **你认为这里的 env_tf.pc 存储的是物理地址还是虚拟地址?**
>
> **• 你觉得entry_point其值对于每个进程是否一样？该如何理解这种统一或不同？**

- env_tf.pc为程序运行的入口地址，是虚拟地址
- 一样；使CPU每次从同一地址开始执行程序，保证了每个进程所见地址空间的统一性

### Thinking3.6

> **请查阅相关资料解释，上面提到的epc是什么？为什么要将env_tf.pc设置为epc呢？**

- cp0_epc为保存异常时，系统正在执行的指令的地址；将pc值设为cp0_epc，便于再次执行该进程时，从上次中断的地方继续执行

### Thinking3.7

> **•** **操作系统在何时将什么内容存到了 TIMESTACK 区域**
>
> **•** **TIMESTACK 和 env_asm.S 中所定义的 KERNEL_SP 的含义有何不同**

- 在进程切换时，操作系统将此时进程寄存器的值保存在TIMESTACK区域
- TIMESTACK是用于时钟中断的，而KERNEL_SP用于其他中断

### Thinking3.8

> **试找出上述 5 个异常处理函数的具体实现位置**

- handle_int在genex.S中
- handle_mod在genex.S中用BUILD_HANDLER实现
- handle_tlb在genex.S中用BUILD_HANDLER实现
- handle_sys在syscall.S中

### Thinking3.9

> **阅读 kclock_asm.S 和 genex.S 两个文件，并尝试说出 set_timer 和timer_irq 函数中每行汇编代码的作用**

```
LEAF(set_timer)

    li t0, 0xc8						//加载立即数0xc8到t0
    sb t0, 0xb5000100				//向0xb5000100 位置写入0xc8，表示一秒钟中断200次
    sw  sp, KERNEL_SP				//将sp寄存器的值存入KERNEL_SP的地址空间中
setup_c0_status STATUS_CU0|0x1001 0	//将CP0的SR寄存器值变为0x10001001，即1，12，28位置为1
    jr ra

    nop
END(set_timer)
```

```
timer_irq:

    sb zero, 0xb5000110	//向0xb5000100 位置写入0xc8，表示关闭实时钟
1:  j   sched_yield		//跳转到进程调度函数
    nop
    /*li t1, 0xff
    lw    t0, delay
    addu  t0, 1
    sw  t0, delay
    beq t0,t1,1f    
    nop*/
    j   ret_from_exception //跳转到ret_from_exception函数
    nop
```

### Thinking3.10

> **阅读相关代码，思考操作系统是怎么根据时钟周期切换进程的**

- 先设置CPU的时钟中断和进程的时间片数量；当时钟中断发生时，进入异常处理，跳转到进程调度函数，若时间片未用完，则继续执行当前进程；若时间片用完，则进行进程切换。

## 实验难点图示

- 中断异常（由于不是很了解，所以本次实验重点关注了一下）

```c
struct Trapframe { //lr:need to be modified(reference to linux pt_regs) TODO
	unsigned long regs[32]; 	// 32 个通用寄存器

	/* 特殊寄存器 */
	unsigned long cp0_status; 	// CP0 状态寄存器
	unsigned long hi;			// 乘（除）法高位（模）寄存器
	unsigned long lo;			// 乘（除）法低位（商）寄存器
	unsigned long cp0_badvaddr;	// 异常发生地址
	unsigned long cp0_cause;	// CP0 cause 寄存器
	unsigned long cp0_epc;		// 异常返回地址
	unsigned long pc;			// PC计数器，程序运行的地址
};
```

- SR寄存器：有中断引脚使能，其他 CPU 模式等位域；
  - 第28位CU0是1表示允许在用户模式下用cp0（set_timer中设置）
  - 15-8 位为中断屏蔽位，表示不同的中断活动，其中 15-10 位使能外部中断源，9-8 位是 Cause 寄存器软件可写的中断位。

- Cause寄存器：记录导致异常的原因
  - 15-8 位保存着哪一些中断发生了，其中 15-10 位来自硬件，9-8 位可以由软件写入
  - 6-2 位，记录发生了什么异常。



- 进程调度

![](C:\Users\WYF\Desktop\进程调度流程.png.png)





## 体会与感想

- 通过本次实验，切实体验了进程创建的过程，理解了asid的作用以及生成和自增过程，以及进程的整个生命周期
- 了解了时钟中断的机制以及相应的中断异常
- 此次实验填写的函数，无论是进程的创建，调度或是时钟中断的产生，都是一系列函数的嵌套调用来实现的；因此这次实验需要梳理各个模块的流程，这也锻炼了我阅读长代码以及总结的能力

