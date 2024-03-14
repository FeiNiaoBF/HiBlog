---
title: "XV6 中的 trap"
date: 2024-03-10T16:35:36+08:00
draft: false
taps: ['linux', 'OS', 'xv6', 'trap']
categories: ['linux', 'OS', 'xv6', 'learning']
author: ["Yeelight"]
showtoc: true
weight:
math: false
readingTime: true
---


>失败是成功之母

## 陷阱指令和系统调用

CPU 在运行中强利将控制权交给特殊的系统代码叫陷阱（trap），其中有三种用到了：

1. 系统调用：**syscall** 当程序执行 **ecall** 时。
2. 异常：指令做了非法。如除0。
3. 设备中断系统进行1设备 **I/O**。

我们希望陷阱是透明的，在程序中中断是难以预料的
一般的顺序：

1. trap 强制控制权交予内核。
2. 内核保护寄存器、内存等其它。
3. 内核处理中断程序。
4. 内核恢复之前保存的状态。
5. 返回并重新恢复代码。

![](https://s2.loli.net/2023/12/24/c45jWnMR1Xs2xH7.png)

> 注：陷阱是由 CPU 上运行的当前进程导致的（其实不够准确），中断是由外部设备导致的。

### RISC -V trup 机利

用户态转到内核态
![](https://s2.loli.net/2023/12/24/9S7tflqZ5JGpjMg.png)

其中 `vm.c` 的工作是：![9S7tflqZ5JGpjMg.png](https://s2.loli.net/2023/12/24/9S7tflqZ5JGpjMg.png)

## ECALL 指令

首先应该知道的是：

- ecall 指令并不会**切换 page table**。
- 将控制权交到**内核 mode**。
- 将 PC 走到 **`trampoline page` 的地址**（STVEC）。
- 将 PC 原来的值存到 **SEPC 寄存器**。
- `ecall` 会跳转到 **STVEC 寄存器** 指向的指令。

之后待处理的事情：

- 目前还在 user page table.需要到 Kernel
- 保（小心）保存32个用户寄存器
- 需要 kernel stack
- 跳转到内核中合理的 C 代码位置

### 总结
>
> ecall 就是陷入 trap 陷阱的入口

注意：至少在 XV6中，有以下几个寄存器需要注意：
> 仅在 trap 中有用的寄存器。（待勘误）

- **stvec** ：保存 trap 处理程序的地址。
- **sepc** ： 保存进入 trap 之前的 PC。 `sret` 在之后返还给 PC。
- **scause** ：描述进入 trap 类型（原因）的（数字）数据。
- **sscratch** ：保存 `trapframe page` 的地址值。
- **sstatus** ： trap 的位控制信息，其中有 `SIE`、`SPP` 等信息。

## USERVEC 指令

在 ecall 指令之后，代码的地址由 **user process** 到 **trampoline pages** 地址 `PC <--- stvec`。

由于 RISC-V 硬件在 trap 中不会切换页表，所以在 `User page table` 中存在 `trampoline page` 的虚拟地址（在 page table 的后面），而 `Kernal page table` 也是一样的 `mapping`。

为了保存 **32个用户寄存器**，我们必须需要足够的内存空间。

1. 不可以使用用户空间！
因为我们不确定用户进程是否使用了栈，是否有足够的页表来可以用。是否有足够的空间来保存。

2. 不可以使用内核空间！（有使用的机器）:
因为我们在 trap 开始时不清楚 `kernel page table` 的地址，而且我们需要使用 **SATP 寄存器** 指向内核页表，这需要空闲寄存器（如： a0）。但是在此时，用户进程再用 a0 等寄存器。

因此，我们需要一个特定的 page — **trapframe page**

在之前内核就为每个用户页表 mapping 了这个 page。

```a
// 每个用户空间都有的
Ox3FFFFFE000      // TRAPFRAME
0x3FFFFFF000      // TRAPOLINE
```

在 `trapframe page` 有许多有趣的数据：

```asm
0  ------ Kernel_satp         // kemnel page table
8  ------ kernel_sp           // top of process＇s kennel stack
16 ------ kernel_trap         // usertrap（）address
24 ------ epc                 // saved user program counter
32 ------ kernel_hartid       // saved kernel tp
```

RISC-V 利用 `csrrw`（可 sawp 指令）将 sscratch（存有 `trapframe page` 地址）和 **a0 寄存器** 交换，接下来就是根据 `a0+offset` 来 `sb rd, offset(a0)`  完成用户寄存器的保存。
接下是使用 `t0` 来保存 `a0`。

然后，uservec 做完大部分的任务了，接下来是取出前5个（不一定都要）数据调用 `usertrup()` 。

```asm
ld sp，8(a0)            ＃0x3fffffc000
# 将内核的SP出，取出，指向kernel stack最顶端

ld tp，32(a0)           ＃ OxO (单核)
# 将当前运行进程的CPU核编号取出

ld t0，16(ao)           # usererap()
# 将 usertrap() 函数的地址写入 t0 寄存器

ld t1，0(a0)            # kernel page table
＃ 将 kernel page table 地址写入 t1 寄存器

csrw satp，t1           ＃ 注意是单向交换 sutp <- t1
# 将 SATP 和 t1 交换, 接下来的地址是为于 Kernel page table 中映射的。

sfence.vma zero，zero
# 清空TLB

jr t0
# 跳到usertrap（），执行usertrop（）

```

> 注：trampoline page 的地址映射在用户和内核的映射是相同

小结：
使用汇编的 **uservec 函数** 可以 **更细粒度的控制** 和 **更高的性能**、寄存器是粒度的，需要精确控制；利用映射相同可以“**_无痕_**”完成转化换页表。

## USERTRAP 函数

OK。
我们正式进入 trap，这个 C 函数的主要功能是确定进程进入 trap 的原因，并以确定相应的处理方式

1. 更改 `stvec` 寄存器。
这一做法取决于 trap 是由用户空间还是内核空间陷入的，将 `stvec` 指向内核页表，确保 trap 在内核中处理（如果 kernel 使用 trap，那就一直在内核页表，不必做多余操作，但有其他操作）。

2. 找到当前的进程。
根据此时 CPU 核（tp 寄存器）来找当前在 tp 上运行的进程。

3. 保存用户程序计数器（之前存在 `sepc` 中）。
这么做的原因是，trap 在执行中可能切换到另一个进程，然后可能再调用 **ecall** 导致 `sepc` 寄存器被覆盖。

```c
p->traptrame->epc = r-sepc();
```

4. 知道 trap 的原因。
根据触发 trap 的原因，`scause寄存器` 会有不同的数字如果值为 `8`，则是 `“systsm call”`，然后进入过系统调用的有关操作。
>
> 1. 检查是否有其他进程 `Kill` 当前进程
> 2. 对于保存在 `epc`的用户 PC+4
> 3. 打开中断，为系统调用作中断
> 4. 调用 syscallc ()

syscall 函数从表单中寻找系统调用编号，保护在 trap 里 a7中，（a0, a1, a2... ...为参数, a0 一般是返回值）。最后返回，调用 `usertrapret` 。

## USERTRAPRET 函数

在返回内用户空间之前，内核需要做的工作：此时，我们的 `stvec` 指向内核空间 trap 的处理代码。现在关闭中断，并更新 `stvec`

```c
w_stvec(TRAMPOLINE + (uservec-trampoline));
```

最终执行 sret 指令在再重新打开中断。之后，填入 `trapframe` 内容（前4个）为之后用户下一次转到内核做准备。

设置sstatus寄存器，控制寄存器

| bit  | 0              | 1                   |
| ---- | -------------- | ------------------- |
| SPP  | sret 返回 user | sret 返回supervisor |
| SPIE | 不打开中断     | 打开中断            |    |

将 `epc->sepc`，设置成用户 PC 的值

生成userret函数的参数和地址

**fn** :    trampeline 基础址 + 偏移量
**a0** :   trapframe 基址
**a1** :   user page table 基址

## USERRET 函数

返回用户汇编代码

1. 切换用户 page table.
将参数 `a1` 传给 `satp`。（不出问题是因为trap映射一样）

2. 根据 `trapframe`（a0参数）恢复寄存器。将 `trapframe` 传给 `t0` 再传给 `sscratch` 。

3. 将 `trapframe` 中 `a0`（保存着系统调用的返回值）传给 `a0` 寄存器。
4. **sret**
切换回 **user mode**，PC 拷贝 SEPC（之前 PC+4），重新打开中断。

总结：
trap 的系统调用为了保持隔离性，做得十分复杂。有些操作为了追求精细化用了汇编语言。
