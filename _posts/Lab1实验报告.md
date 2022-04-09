# Lab1实验报告

## 实验思考题

### Thinking1.1

> 请查阅并给出前述objdump 中使用的参数的含义。使用其它体系结构的编译器（如课程平台的MIPS交叉编译器）重复上述各步编译过程，观察并在实验报告中提交相应结果。

- -D 从objfile中反汇编所有section
- -S 尽可能反汇编除源代码

对下述C代码的main部分进行反汇编

```c
int main(){
    int a = 1;
    char c = 'b';
    int b = a;
    return 0;
}
```

- 只编译而不链接

```shell
/OSLAB/compiler/usr/bin/mips_4KC-gcc -c hello_world.c
/OSLAB/compiler/usr/bin/mips_4KC-objdump -DS hello_world.o > test.txt
```

main函数结果如下

```
hello_world.o:     file format elf32-tradbigmips

Disassembly of section .text:

00000000 <main>:
   0:   27bdffe0    addiu   sp,sp,-32
   4:   afbe0018    sw  s8,24(sp)
   8:   03a0f021    move    s8,sp
   c:   24020001    li  v0,1
  10:   afc20010    sw  v0,16(s8)
  14:   24020062    li  v0,98
  18:   a3c2000c    sb  v0,12(s8)
  1c:   8fc20010    lw  v0,16(s8)
  20:   afc20008    sw  v0,8(s8)
  24:   00001021    move    v0,zero
  28:   03c0e821    move    sp,s8
  2c:   8fbe0018    lw  s8,24(sp)
  30:   27bd0020    addiu   sp,sp,32
  34:   03e00008    jr  ra
  38:   00000000    nop
  3c:   00000000    nop
```

- 允许gcc链接，编译出可执行文件

```shell
/OSLAB/compiler/usr/bin/mips_4KC-gcc -c hello_world.c
/OSLAB/compiler/usr/bin/mips_4KC-ld -o helloworld hello_world.o//调用链接器ld
/OSLAB/compiler/usr/bin/mips_4KC-objdump -DS helloworld > test.txt 
```

main结果如下

```
004000b0 <main>:
  4000b0:   27bdffe0    addiu   sp,sp,-32
  4000b4:   afbe0018    sw  s8,24(sp)
  4000b8:   03a0f021    move    s8,sp
  4000bc:   24020001    li  v0,1
  4000c0:   afc20010    sw  v0,16(s8)
  4000c4:   24020062    li  v0,98
  4000c8:   a3c2000c    sb  v0,12(s8)
  4000cc:   8fc20010    lw  v0,16(s8)
  4000d0:   afc20008    sw  v0,8(s8)
  4000d4:   00001021    move    v0,zero
  4000d8:   03c0e821    move    sp,s8
  4000dc:   8fbe0018    lw  s8,24(sp)
  4000e0:   27bd0020    addiu   sp,sp,32
  4000e4:   03e00008    jr  ra
  4000e8:   00000000    nop
  4000ec:   00000000    nop
```

地址已由虚拟内存定位到实际内存中。

------

### Thinking1.2

> **也许你会发现我们的readelf程序是不能解析之前生成的内核文件(内核文件是可执行文件)的，而我们之后将要介绍的工具readelf则可以解析，这是为什么呢？(提示：尝试使用readelf -h，观察不同)**

- 使用`readelf -h vmlinux`查看内核文件

```
Magic:   7f 45 4c 46 01 02 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF32
  Data:                              2's complement, big endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ...
```

- 使用`readelf -h testELF`查看此程序

```
 Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF32
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ...
```

通过魔数可见，e_ident[EI_DATA]的值不同，vmlinux为大端存储，而我们的vmlinux是小端存储，因此无法用于解析内核文件

------

### Thinking1.3

> **在理论课上我们了解到，MIPS 体系结构上电时，启动入口地址为0xBFC00000（其实启动入口地址是根据具体型号而定的，由硬件逻辑确定，也有可能不是这个地址，但一定是一个确定的地址），但实验操作系统的内核入口并没有放在上电启动地址，而是按照内存布局图放置。思考为什么这样放置内核还能保证内核入口被正确跳转到？**

从CPU上电到操作系统内核被加载的整个启动的步骤如图所示。

![image-20220402163825793](C:\Users\WYF\AppData\Roaming\Typora\typora-user-images\image-20220402163825793.png)

- 上述bootloader执行的操作都可由gxemul仿真器进行
- 同时gxemul仿真器支持直接加载ELF格式的内核，所以启动流程被简化为加载内核到内存，之后跳转到内核的入口。
- 因此只需正确设置程序入口：本实验中在linker scirpt中使用ENTRY()指令指定了程序入口，可保证正确跳转。

------

### Thinking1.4

> **与内核相比，普通进程的sg_size 和bin_size 的区别在于它的开始加载位置并非页对齐，同时bin_size的结束位置（va+i，其中i为计算出的该段在ELF文件中的大小）也并非页对齐，最终整个段加载完毕的sg_size 末尾的位置也并非页对齐。请思考，为了保证页面不冲突（不重复为同一地址申请多个页，以及页上数据尽可能减少冲突），这样一个程序段应该怎样加载内存空间中。**

已知本操作系统的页大小为4KB

- 使用`readelf --segments vmlinux`查看内核的段信息


![image-20220402183630481](C:\Users\WYF\AppData\Roaming\Typora\typora-user-images\image-20220402183630481.png)

- 使用`readelf --segments helloworld`查看普通程序的段信息


![image-20220402183828062](C:\Users\WYF\AppData\Roaming\Typora\typora-user-images\image-20220402183828062.png)

- 从Entry point 程序入口地址可以看出内核的加载是页对齐的
- 为减少页冲突，或许可以将每个段的加载地址从上一个段结束的位置对页大小向上取整（~~我瞎说的~~）

------

### Thinking1.5

> **内核入口在什么地方？main 函数在什么地方？我们是怎么让内核进入到想要的 main 函数的呢？又是怎么进行跨文件调用函数的呢？** 

- 内核入口为_start()函数
- main函数在0x80010000位置
- 在start.S中令栈指针指向0x80010000,然后用jal指令跳转至main函数入口
- main 函数虽然为 c 语言所书写，但是在被编译成汇编之后，其入口点会被翻译为一个标签，汇编指令jal可进行跳转，实际地址会在链接时得到。

------

### Thinking1.6

> **查阅《See MIPS Run Linux》一书相关章节，解释boot/start.S 中下面几行对CP0 协处理器寄存器进行读写的意义。具体而言，它们分别读/写了哪些寄存器的哪些特定位，从而达到什么目的？**

```c
/* Disable interrupts */
    mtc0    zero, CP0_STATUS

    /* disable kernel mode cache */
    mfc0    t0, CP0_CONFIG
    and t0, ~0x7
    ori t0, 0x2
    mtc0    t0, CP0_CONFIG
```

一些相关宏定义如下：

```c
#define CP0_STATUS $12
#define CP0_CONFIG $16
```

- CP0_STATUS
  - 状态寄存器（Status Register）由可写的控制位域组成，包括确定CPU特权等级，哪些中断引脚使能和其他的CPU模式等位域
  - `mtc0 zero, CP0_STATUS`即将SR寄存器清零，目的为禁止中断，保证程序的正常启动

- CP0_CONFIG

  - CPU参数设置，通常由系统决定；一些可写，另一些只读。

  - 上述操作先将低三位清零，然后对第一位置1；即将config寄存器的低三位置为010

    ![image-20220402195849831](C:\Users\WYF\AppData\Roaming\Typora\typora-user-images\image-20220402195849831.png)

  - 由上图可知低三位K0域，用来决定固定的kseg0区是否经过高速缓存。查阅文档《**See MIPS Run Linux**》可知，置为010为不经过高速缓存

------

## 实验难点

![实验难点](C:\Users\WYF\Desktop\实验难点.png)



## 体会与感想

本次实验较Lab0难度有了很大提升，也需要很多理论知识的储备（如操作系统启动过程），花费了很长时间在理解指导书的内容和阅读代码上（像这样做完形填空需要对总体流程有大致的了解；比如各种数据结构的宏定义，函数的定义等）

- 初步了解了操作系统的启动过程以及bootloader的工作流程
- 深刻认识了一个程序预处理、编译、汇编、链接的过程，对整体流程有了大致把握
- 对ELF文件格式有了更深刻的了解
- 掌握了printf的实现原理
- 某种程度上加深了我对C语言指针的理解

