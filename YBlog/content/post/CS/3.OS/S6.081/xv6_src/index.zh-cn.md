---
title: "XV6的源码解析"
date: 2024-03-10T16:35:17+08:00
draft: false
taps: ['linux', 'OS', 'xv6', 'Source code']
categories: ['linux', 'OS', 'xv6', 'learning', 'Source code']
author: ["Yeelight"]
showtoc: true
weight:
math: false
readingTime: true
---


## 启动 xv6

当我们输入 `make qeum` 之后它会初始化自己并运行一个存储在只读内存中的引导加载程序（boot），CPU 从内存地址 `0x80000000` （程序的起始位置）开始的:

```c
_entry:
    # set up a stack for C.
        # stack0 is declared in start.c,
        # with a 4096-byte stack per CPU.
        # sp = stack0 + (hartid * 4096)
        la sp, stack0
        li a0, 1024*4
    csrr a1, mhartid
        addi a1, a1, 1
        mul a0, a0, a1
        add sp, sp, a0
    # jump to start() in start.c
        call start
```

`_entry` 的指令设置了一个栈区 (stack 0)，这样 xv6就可以运行 C 代码。Xv6在 *start. c*   文件中为初始栈  `stack0` 声明了空间。由于 RISC-V 上的栈是向下扩展的，所以 `_entry` 的代码将栈顶地址 `stack0+4096` 加载到栈顶指针寄存器 `sp` 中。现在内核有了栈区，`_entry` 便调用 C 代码 `start` 。在其中 `csrr a1, mhartid` 在地址0x8000000a 上，读取了控制系统寄存器（Control System Register）mhartid，并将结果加载到了 a1寄存器。我认为是在确定几号 CPU 在设置 stack。

在 entry. S 中没有内存分页，没有中断，没有隔离等等一切都是那么的  `原始`   ，在 ` start () ` 中其实就是在 M-mod (machine mode) 进行基础配置，它在寄存器 `mstatus` 中将先前的运行模式改为管理模式，并通过将 `main()` 函数的地址写入寄存器 `mepc` 将返回地址设为 `main`，它通过向页表寄存器 `satp` 写入0来在管理模式下禁用虚拟地址转换，并将所有的中断和异常委托给管理模式。

```c
void
start()
{
  // set M Previous Privilege mode to Supervisor, for mret.
  unsigned long x = r_mstatus();
  x &= ~MSTATUS_MPP_MASK;
  x |= MSTATUS_MPP_S;
  w_mstatus(x);
  // set M Exception Program Counter to main, for mret.
  // requires gcc -mcmodel=medany
  w_mepc((uint64)main);
  // disable paging for now.
  w_satp(0);
  ......
  }
```

最后，当做完这一切后，`mepc` 返回 main，并中断计时器，PC 改为 main 。
在 `main()` 中执行一系列的初始化

```c
void
main()
{
  if(cpuid() == 0){
    consoleinit();  // 初始化控制台
    printfinit();
    printf("\n");
    printf("xv6 kernel is booting\n");
    printf("\n");
    kinit();         // 物理页面分配器
    kvminit();       // 内核页表
    kvminithart();   // 开启分页机制
    procinit();      // 进程表初始化
    trapinit();      // trap向量表初始化
    trapinithart();  // 安装内核trap向量
    plicinit();      // 设置中断控制器
    plicinithart();  // 向 PLIC 询问设备中断
    binit();         // buffer cache
    iinit();         // inode table
    fileinit();      // file table
    virtio_disk_init(); // emulated hard disk
    userinit();      // first user process
    ...
    }
    ...
}   
```

> 初始化函数的调用顺序重要吗？
> 	重要，有些函数需要其他函数作为前置
## 启动第一个进程

着重说明 `userinit()` 他是利用了 `initcode. S` 来通过调用 `exec(init, argv)` 以达到一个用户进程的实现

```
# exec(init, argv)
.globl start
start:
        la a0, init        # 第一个参数
        la a1, argv        # 第二个参数
        li a7, SYS_exec
        ecall              # 交给操作系统
```

接着进入到 `syscall()`

```c
void
syscall(void)
{
  int num;
  struct proc *p = myproc();
  num = p->trapframe->a7;
  if(num > 0 && num < NELEM(syscalls) && syscalls[num]) {
    p->trapframe->a0 = syscalls[num]();
    if (p->make & (1 << (uint32)num)) {
        printf("%d: syscall %s -> %d\n", p->pid, sys_name[num], p->trapframe->a0);
    }
  } else {
    printf("%d %s: unknown sys call %d\n",
            p->pid, p->name, num);
    p->trapframe->a0 = -1;
  }
}
```

`syscalls[num]()` 是函数指针·数组，它去找 exec 的系统函数 `sys_exec` 从 exec 进入 init 函数，`init` 为用户内存设置好一些东西，如 console、标准输入、标准输出和利用子进程打开 *sh* 。ok，结束了。
