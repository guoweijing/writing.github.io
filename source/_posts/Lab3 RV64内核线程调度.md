---
title: os实验3
categories:
- os
tag :
- os
---
## Lab3 RV64内核线程调度

**课程名称：操作系统**

**学生姓名：郭伟京**

**学号：3200102538**

**电子邮件地址：3200102538@zju.edu.cn**

**实验日期：10月31日**

### 1.实验目的

**了解线程概念, 并学习线程相关结构体, 并实现线程的初始化功能。**

**了解如何使用时钟中断来实现线程的调度。**

**了解线程切换原理, 并实现线程的切换。**

**掌握简单的线程调度算法, 并完成两种简单调度算法的实现**

### 2.实验环境

**Environment in previous labs**

### 3.实验步骤

#### 3.1 准备工程，修改源文件

**目录结构：**

![image-20221110142023386](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20221110142023386.png)

**defs.h:**

```c
#ifndef _DEFS_H
#define _DEFS_H

#define PHY_START 0x0000000080000000
#define PHY_SIZE  128 * 1024 * 1024 // 128MB,  QEMU 默认内存大小
#define PHY_END   (PHY_START + PHY_SIZE)

#define PGSIZE 0x1000 // 4KB
#define PGROUNDUP(addr) ((addr + PGSIZE - 1) & (~(PGSIZE - 1)))
#define PGROUNDDOWN(addr) (addr & (~(PGSIZE - 1)))


#include "types.h"

#define csr_read(csr)                       \
({                                          \
    register uint64 __v;                    \
    asm volatile ("csrr %[v], " #csr    \
    : [v] "=r" (__v)::);     \
    __v;                                    \
})

#define csr_write(csr, val)                         \
({                                                  \
    uint64 __v = (uint64)(val);                     \
    asm volatile ("csrw " #csr ", %0"               \
                    : : "r" (__v)                   \
                    : "memory");                    \
})

#endif
```

#### 3.2线程调度实现

#### 3.2.1线程初始化

为每个线程分配一个 4KB 的物理页, 我们将task_struct存放在该页的低地址部分,  将线程的栈指针sp指向该页的高地址，结构如下：

![image-20221110143043472](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20221110143043472.png)

**1.OS本身就是一个线程idle 线程，我们要为idle设置task_struct。并将current,task[0]都指向idle。**

**2.将task[1] ~ task[NR_TASKS - 1], 全部初始化,  这里和idle设置的区别在于要为这些线程设置thread_struct中的ra和sp.**

```c
void task_init() {
    // 1. 调用 kalloc() 为 idle 分配一个物理页
    // 2. 设置 state 为 TASK_RUNNING;
    // 3. 由于 idle 不参与调度 可以将其 counter / priority 设置为 0
    // 4. 设置 idle 的 pid 为 0
    // 5. 将 current 和 task[0] 指向 idle

    /* YOUR CODE HERE */
    uint64 addr_idle = kalloc();
    idle = (struct task_struct*)addr_idle;
    idle->state = TASK_RUNNING;
    idle->counter =0;
    idle->priority = 0;
    idle->pid = 0;

    current = task[0] = idle;

    // 1. 参考 idle 的设置, 为 task[1] ~ task[NR_TASKS - 1] 进行初始化
    // 2. 其中每个线程的 state 为 TASK_RUNNING, counter 为 0, priority 使用 rand() 来设置, pid 为该线程在线程数组中的下标。
    // 3. 为 task[1] ~ task[NR_TASKS - 1] 设置 `thread_struct` 中的 `ra` 和 `sp`,
    // 4. 其中 `ra` 设置为 __dummy （见 4.3.2）的地址,  `sp` 设置为 该线程申请的物理页的高地址

    /* YOUR CODE HERE */
    for(int i = 1; i < NR_TASKS; i++){
        uint64 task_addr = kalloc();
        task[i] = (struct task_struct*)task_addr;
        task[i]->state = TASK_RUNNING;
        task[i]->counter = 0;
        task[i]->priority = rand();
        task[i]->pid = i;
        task[i]->thread.ra = (uint64)__dummy;
        task[i]->thread.sp = task_addr + 4096;//4k
    }

    printk("...proc_init done!\n");
}
```

3._start中调用task_int

```assembly
    jal ra, task_init # initialize task threads
```

#### 3.3.2 _dumy的调用

**当我们创建一个新的线程, 此时线程的栈为空, 当这个线程被调度时,是没有上下文需要被恢复的, 所以我们需要为线程第一次调度提供一个特殊的返回函数dummy**

**在entry.S添加_dummy：在dummy中将 sepc 设置为dummy()的地址, 并使用sret从中断中返回。_**

```assembly
    .extern dummy
    .globl __dummy
__dummy:
    la t0, dummy
    csrw sepc, t0
    sret
```

#### 3.3.3实现线程切换

**判断下一个执行的线程next与当前的线程current是否为同一个线程, 如果是同一个线程,则无需做任何处理, 否则调用switch_to进行线程切换。**

```c

extern void __switch_to(struct task_struct* prev, struct task_struct* next);

void switch_to(struct task_struct* next) {
    if (next == current) return; // 同一个线程无需处理
    else {
        struct task_struct* current_saved = current;
        current = next;  //切换
        __switch_to(current_saved, next);
    }
}

```

**在entry.S中实现线程上下文切换 _switch_to:**

- **switch_to接受两个task_struct指针作为参数**
- **保存当前线程的ra, sp, s0~s11到当前线程的thread_struct中**
- **将下一个线程的thread_struct中的相关数据载入到ra, sp, s0~s11中。**

```assembly
# arch/riscv/kernel/entry.S
.globl _switch_to
_switch_to:
# save state to prev process
 add t0, a0, 40  # offset of thread_struct in current task_struct
    sd ra, 0(t0)
    sd sp, 8(t0)
    sd s0, 16(t0)
    sd s1, 24(t0)
    sd s2, 32(t0)
    sd s3, 40(t0)
    sd s4, 48(t0)
    sd s5, 56(t0)
    sd s6, 64(t0)
    sd s7, 72(t0)
    sd s8, 80(t0)
    sd s9, 88(t0)
    sd s10, 96(t0)
    sd s11, 104(t0)

# restore state from next process
    add t0, a1, 40  # offset of thread_struct in next task_struct
    ld ra, 0(t0)
    ld sp, 8(t0)
    ld s0, 16(t0)
    ld s1, 24(t0)
    ld s2, 32(t0)
    ld s3, 40(t0)
    ld s4, 48(t0)
    ld s5, 56(t0)
    ld s6, 64(t0)
    ld s7, 72(t0)
    ld s8, 80(t0)
    ld s9, 88(t0)
    ld s10, 96(t0)
    ld s11, 104(t0)
# YOUR CODE HERE
ret
```

#### 3.3.4实现调度入口函数

```c
void do_timer(void) {
    // 1. 如果当前线程是 idle 线程 直接进行调度
    // 2. 如果当前线程不是 idle 对当前线程的运行剩余时间减1 若剩余时间仍然大于0 则直接返回 否则进行调度
    if (current == task[0]) schedule();//直接调度
    else {
        current->counter = current->counter-1;
        if (current->counter <= 0) schedule();
    }
}
```

#### 3.3.5实现线程调度

**短作业优先调度算法**：

**当需要进行调度时按照一下规则进行调度：**

- **遍历线程指针数组task(不包括idle , 即task[0] ), 在所有运行状态(TASK_RUNNING) 下的线程运行剩余时间最少的线程作为下一个执行的线程。**
- **如果所有运行状态下的线程运行剩余时间都为0, 则对task[1] ~ task[NR_TASKS-1]的运行剩余时间重新赋值 (使用rand()) , 之后再重新进行调度。**

```c
void schedule(void){
    uint64 min_count = 0xFFFFFFFFFFFFFFFF;
    struct task_struct* next = NULL;
    int flag= 1;
    for(int i = 1; i < NR_TASKS; i++){
        if (task[i]->state == TASK_RUNNING && task[i]->counter > 0) {
            //find shorter time task
            if (task[i]->counter < min_count) {
                min_count = task[i]->counter;
                next = task[i];
            }
            flag = 0;
        }
    }
//时间都剩下0
    if (flag) {
        printk("\n");
        for(int i = 1; i < NR_TASKS; i++){
            task[i]->counter = rand() % 20 + 1; //分配时间
            printk("SET [PID = %d COUNTER = %d]\n", task[i]->pid, task[i]->counter);
        }
        schedule(); //重新调度
    }
    //调度到next
    else {
        if (next) {
           printk("\nswitch to [PID = %d COUNTER = %d]\n", next->pid, next->counter);
            switch_to(next);
        }
    }
}
```

**优先级调度算法：**

```c
void schedule(void){
    int next, flag;
    uint64 max_priority;
      while(1){
        max_priority = 0x0;
        flag = 1;
        for(int i = 1; i < NR_TASKS; i++){
            if(task[i]->counter == 0) //已完成，跳过
                continue;
            else
                flag = 0; //不是全0
            
            if(task[i]->priority >= max_priority) //优先级更高
                next = i, max_priority = task[i]->priority;
        }
        if(!flag)
            break;
          //全0，重新分配时间
        for(int i = 1; i < NR_TASKS; i++){
            task[i]->counter = (task[i]->counter >> 1) + task[i]->priority;
            printk("SET [PID = %d PRIORITY = %d COUNTER = %d]\n", i, task[i]->priority, task[i]->counter);
        }
    }
    printk("\nswitch to [PID = %d PRIORITY = %d COUNTER = %d]\n", task[next]->pid, task[next]->priority, task[next]->counter);
	switch_to(task[next]);
}
```

### 3.4编译测试（更改Makefile对两种调度进行测试）

#### 3.4.1 短作业优先算法：

![image-20221111120435771](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20221111120435771.png)

![image-20221111120458906](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20221111120458906.png)

#### 3.4.2 优先级调度算法

![image-20221111120618024](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20221111120618024.png)

![image-20221111120741708](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20221111120741708.png)

### 4.思考题

1. **在 RV64 中一共用 32 个通用寄存器,  为什么context_switch中只保存了14个？**

   **首先每个进程需要保存 ra 和 sp 的值，之外需要 12 个寄存器来保存 s0~s11,这样就是总**
   **共的 14 个寄存 器，而剩下的部分寄存器并不需要保存**

   

2. **当线程第一次调用时,  其ra所代表的返回点是dummy。那么在之后的线程调用中context_switch中, ra保存/恢复的函数返回点是什么呢？请同学用 gdb 尝试追踪一次完整的线程切换流程,  并关注每一次ra的变换 (需要截图)。**

**在之后的线程调用 context_switch 中，ra 保存/恢复的函数返回点traps 中 trap_handler 返回后的下一条指令。因此当发生进程切换后，下一个进程最终将返回 dummy 中上一个进程发生时钟中断的地址处继续执行。**

**一次切换：**

![image-20221111131316806](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20221111131316806.png)

![image-20221111131658065](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20221111131658065.png)

![image-20221111131759024](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20221111131759024.png)

![image-20221111131911924](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20221111131911924.png)

### 5.讨论与心得

此次实验总的来说还可以，ppt上的介绍与流程都十分清楚能让你每一部分都写的比较顺利，最后也是完成的比较顺利，但对于一些实现原理和逻辑感觉自己一开始因此没有弄的很清楚，以后自己每部分写完后也应该多去观察实现的原理。
