---
title: os实验2
categories:
- os
tag :
- os
---

##                      实验2: Rinux 时钟中断处理

**课程名称：操作系统**

**学生姓名：郭伟京**

**学号：3200102538**

**电子邮件地址：3200102538@zju.edu,cn**

**实验日期：10月31日**

### 1 实验目的

**学习 RISC-V 的异常处理相关寄存器与指令，完成对异常处理的初始化。**

**理解 CPU 上下文切换机制，并正确实现上下文切换功能。**

**编写异常处理函数，完成对特定异常的处理。**

**调用 OpenSBI 提供的接口，完成对时钟中断事件的设置。**

###  2 实验环境

**Docker in Lab0**

### 3实验步骤

#### 3.1准备工程

**从repo的src/lab2目录中同步以下代码: stddef.hprintk.hprintk.c，并按如下目录结构放置。并需要将之前所有print.h, puti和puts的引用修改为printk.h和printk。**

![image-20221031163704885](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20221031163704885.png)

**修改vmlinux.lds以及head.S：**

  **vmlinux.lds:**

```c
*(.text.init)   //加入了 .text.init
```

   **head.S:**

```c
//before
.section .text.entry
============
.section .text.init
// after
```

#### 3.2开启异常处理

**对CSR进行初始化：**

1. **设置stvec，将_traps ( _trap在 4.3 中实现 ) 所表示的地址写入stvec，这里我们采用Direct 模式, 而_traps则是中断处理入口函数的基地址。**

2. **开启时钟中断，将sie[STIE]置 1。**

3. **设置第一次时钟中断，参考后文 4.5 小节中介绍clock_set_next_event()的汇编逻辑实现。**

4. **开启 S 态下的中断响应，将sstatus[SIE]置 1。**

   **sie和sstatus结构如下：**

![image-20221101225036737](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20221101225036737.png)

![image-20221101224825223](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20221101224825223.png)

**head.S：**

```assembly
.extern start_kernel
.extern clock_set_next_event

    .section .text.init
    .globl _start
_start:

   la sp, boot_stack_top #负增长
   
    # set stvec = _traps
    la  t0,_traps  #加载到t0
    csrw stvec,t0  #写入stvec

    # set sie[STIE] = 1 

    csrr t0, sie  
    ori t0, t0, 0x20  #100000
    csrw sie, t0
    
    # set first time interrupt
    jal ra, clock_set_next_event
    
    # set sstatus[SIE] = 1
    csrr t0, sstatus
    ori t0, t0, 0x2   #10
    csrw sstatus, t0


    j start_kernel

    .section .bss.stack
    .globl boot_stack
boot_stack:
    .space 4096  //4k

    .globl boot_stack_top
boot_stack_top:
```

#### 3.3实现上下文切换

1. **在arch/riscv/kernel/目录下添加entry.S文件。**
2. **保存CPU的寄存器（上下文）到内存中（栈上）。**
3. **将scause和sepc中的值传入异常处理函数trap_handler ( trap_handler在 4.4 中介绍 ) ，我们将会在trap_handler中实现对异常的处理。**
4. **在完成对异常的处理之后，我们从内存中（栈上）恢复CPU的寄存器（上下文）。**
5. **从 trap 中返回。**

**entry.S：**

```assembly
.section .text.entry
    .align 2
    .globl _traps 
_traps:
    # 1. save 32 registers to stack
    addi sp, sp, -256
    sd x1, 0(sp)
    sd x2, 8(sp)
    sd x3, 16(sp)
    sd x4, 24(sp)
    sd x5, 32(sp)
    sd x6, 40(sp)
    sd x7, 48(sp)
    sd x8, 56(sp)
    sd x9, 64(sp)
    sd x10, 72(sp)
    sd x11, 80(sp)
    sd x12, 88(sp)
    sd x13, 96(sp)
    sd x14, 104(sp)
    sd x15, 112(sp)
    sd x16, 120(sp)
    sd x17, 128(sp)
    sd x18, 136(sp)
    sd x19, 144(sp)
    sd x20, 152(sp)
    sd x21, 160(sp)
    sd x22, 168(sp)
    sd x23, 176(sp)
    sd x24, 184(sp)
    sd x25, 192(sp)
    sd x26, 200(sp)
    sd x27, 208(sp)
    sd x28, 216(sp)
    sd x29, 224(sp)
    sd x30, 232(sp)
    sd x31, 240(sp)
    #and sepc 
    csrr t0, sepc
    sd t0, 248(sp)
# 2. call trap_handler
csrr a0, scause #取出scause，sepc
csrr a1, sepc
jal x1, trap_handler 

# 3. restore sepc and 32 registers (x2(sp) should be restore last) from stack
ld t0, 248(sp)
csrw sepc, t0
ld x1, 0(sp)
ld x3, 16(sp)
ld x4, 24(sp)
ld x5, 32(sp)
ld x6, 40(sp)
ld x7, 48(sp)
ld x8, 56(sp)
ld x9, 64(sp)
ld x10, 72(sp)
ld x11, 80(sp)
ld x12, 88(sp)
ld x13, 96(sp)
ld x14, 104(sp)
ld x15, 112(sp)
ld x16, 120(sp)
ld x17, 128(sp)
ld x18, 136(sp)
ld x19, 144(sp)
ld x20, 152(sp)
ld x21, 160(sp)
ld x22, 168(sp)
ld x23, 176(sp)
ld x24, 184(sp)
ld x25, 192(sp)
ld x26, 200(sp)
ld x27, 208(sp)
ld x28, 216(sp)
ld x29, 224(sp)
ld x30, 232(sp)
ld x31, 240(sp)
ld x2, 8(sp)
addi sp, sp, 256

# 4. return from trap
sret  #sret返回
```

#### 3.4实现异常处理函数

**scause结构：**

![image-20221101233238254](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20221101233238254.png)

**时间中断：execption code为5**

![image-20221101233623440](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20221101233623440.png)



**trap.C:**

```c
// trap.c
#include "printk.h"
#include "clock.h"
#include "trap.h"
void trap_handler(unsigned long scause, unsigned long sepc) 
{
    //sepc会记录触发异常的那条指令的地址。
    //scause 会记录异常发生的原因，还会记录该异常是Interrupt还是Exception。
    // 通过 `scause` 判断trap类型/
    //if it is interrupt
        unsigned long inter_sig = 1UL << 63;
        if(scause & inter_sig){
             unsigned long timer_int = 0x5;
            uint64 flag = scause & ~inter_sig;
            // 如果是interrupt 判断是否是timer interrupt/
            if(flag & timer_int ==5 ){
                //则打印相关信息, 并通过 `clock_set_next_event()` 设置下一次时钟中断
                printk("kernel is running!\n");
                printk("[S] Supervisor Mode Timer Interrupt\n");
                clock_set_next_event();
            }
        }  
}
```

#### 3.5实现时钟中断相关函数

1. **在arch/riscv/kernel/目录下添加clock.c文件。**

2. **在clock.c中实现get_cycles() : 使用rdtime汇编指令获得当前time寄存器中的值。**

3. **在clock.c中实现clock_set_next_event() : 调用sbi_ecall，设置下一个时钟中断事件。**

   **clock.c:**

```c
#include "clock.h"
#include "sbi.h"
unsigned long TIMECLOCK=10000000;
unsigned long get_cycles() {
    // 使用 rdtime 编写内联汇编，获取 time 寄存器中 (也就是mtime 寄存器 )的值并返回
    uint64 time;
    asm volatile("rdtime %0" : "=r"(time));
    return time;
    }
    void clock_set_next_event() 
    {
        // 下一次时钟中断的时间点
    unsigned long next=get_cycles() +TIMECLOCK;
        // 使用 sbi_ecall 来完成对下一次时钟中断的设置
    sbi_ecall(0, 0, next, 0, 0, 0, 0, 0); 
    }
```

#### **3.6编译及测试**

![image-20221101234712031](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20221101234712031.png)

### 4.思考题

1. **在我们使用make run时， OpenSBI 会产生如下输出:**

![image-20221101234820217](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20221101234820217.png)

**通过查看RISC-V Privileged Spec中的medeleg和mideleg解释上面MIDELEG值的含义。**

**medeleg和mideleg分别是机器异常委托寄存器和机器中断委托寄存器，上图中MIDELEG=0x0000000000000222，其第1位，第5位，第9位为1，这三位分别对应中断：supervisor software interrupt，supervisor timer interrupt和supervisor external interrupt，这些位为1的意义是：将这些中断委托给S模式，当他们在M模式下执行时不会被采用，其他位0的意义是其所代表的终端防止无论当前模式如何都会将控制转移到M 模式下运行。**

### 5.讨论与心得：

**此次实验总体上与上一周体系实验有所联系，所以对于中断的定义提前有所了解，但一开始还是比较迷茫的，但按照手册了解相应寄存器结构（scause,sie,sstause等）和系统调用之后，发现每部分完成起来还是比较可以的，但自己对于汇编语言还是比较弱**

