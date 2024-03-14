---
title: "XV6 的中断"
date: 2024-03-10T16:36:08+08:00
draft: false
taps: ['interrupt', 'xv6', 'OS']
categories: ['interrupt', 'xv6', 'OS']
author: ["Yeelight"]
showtoc: true
weight:
math: false
readingTime: true
---

> ls 去哪里了

## 中断硬件

中断对应的场景很简单，就是硬件（外设）想要得到操作系统的关注。当硬件设备需要处理器的注意时，它会发送一个中断信号，这会导致处理器中断当前正在执行的任务，保存当前状态，并开始执行一个称为中断处理程序（interrupt handler）的特殊程序。中断处理程序会处理中断事件，然后恢复之前的任务状态，使得处理器可以继续执行被中断的任务。

像系统调用，但有几点不一样：

- **异步 (asynchronous)**：在硬件中断中 interrupt handler 并不在进程的 context 下，所以 cpu 是异步的。
- **并发 (concurrency)**：对于中断来说，CPU 和生成中断的设备是并行的在运行。
- **程序设备 (program device)** ：要为用到的硬件依据相关的文档编程。

程序设备地址 mapping kernel
![Platform Level Interrupt Control.webp](https://s2.loli.net/2024/03/10/Q6eEPwmoN7DHZGh.webp)

设备都连接到处理器上，处理器上是通过 Platform Level Interrupt Control，简称 PLIC 来处理设备中断。
![Platform Level Interrupt Control.webp](https://s2.loli.net/2024/03/10/Q6eEPwmoN7DHZGh.webp)
PLIC 会路由（像收费站）这些中断，提醒 CPU 有中断，有空的 CPU 核会 Claim 中断，之后接收、处理中断，提醒 PLIC 。

## 设备驱动

设备驱动是一种软件，用于允许操作系统与计算机硬件进行通信和交互。简单的讲：管理设备的代码称为驱动且所有的驱动都在内核中。

大部分驱动都分为两个部分：

1. **驱动程序（bottom）**：驱动程序负责设备的控制和管理，包括设备的初始化、数据传输、状态监控等。驱动程序通常是由内核编写，并且运行在内核模式下，具有较高的权限。
2. **设备程序（top）**：设备设置程序通常是用户空间程序，用于配置和控制设备。这些程序通常不运行在内核模式下，具有较低的权限。设备设置程序可以通过系统调用与驱动程序进行交互，实现对设备的控制和管理。

这种分层结构进行了隔离，两者的交互通过之间的*队列* 来实现。

如何对设备进行编程。通常来说，编程是通过 memory mapped I/O 完成的。在一开始的时候，设备地址就规定出现在物理地址的特定区间内。在通过 `load` / `store` 指令就可以读写设备的控制寄存器。

## XV6 中断例子

当XV6启动时，Shell会输出提示符“`$` ”，如果我们在键盘上输入`ls`，最终可以看到`“$ ls”`。我们接下来通过研究Console是如何显示出`“$ ls”`，来看一下设备中断是如何工作的。

实际上“`$` ”和“`ls`”还不太一样，“`$` ”是Shell程序的输出，而“`ls`”是用户通过键盘输入之后再显示出来的。

RISC-V 有许多与中断相关的寄存器：

- SIE（Supervisor Interrupt Enable）寄存器。这个寄存器中有一个bit（E）专门针对例如UART的外部设备的中断；有一个bit（S）专门针对软件中断，软件中断可能由一个CPU核触发给另一个CPU核；还有一个bit（T）专门针对定时器中断。我们这节课只关注外部设备的中断。

- SSTATUS（Supervisor Status）寄存器。这个寄存器中有一个bit来打开或者关闭中断。每一个CPU核都有独立的SIE和SSTATUS寄存器，除了通过SIE寄存器来单独控制特定的中断，还可以通过SSTATUS寄存器中的一个bit来控制所有的中断。

- SIP（Supervisor Interrupt Pending）寄存器。当发生中断时，处理器可以通过查看这个寄存器知道当前是什么类型的中断。

- SCAUSE寄存器，这个寄存器我们之前看过很多次。它会表明当前状态的原因是中断。

- STVEC 寄存器，它会保存当 trap，page fault 或者中断发生时，CPU 运行的用户程序的程序计数器，这样才能在稍后恢复程序的运行。

![](https://s2.loli.net/2023/12/23/uAEnQoZ5sdzXICk.png)

这里首先初始化了锁，我们现在还不关心这个锁。然后调用了 uartinit，uartinit 函数位于 uart.c 文件。这个函数实际上就是配置好 UART 芯片使其可以被使用。

![](https://s2.loli.net/2023/12/23/4CsmZKpExjrnR2q.webp)
这里的流程是先关闭中断，之后设置波特率，设置字符长度为8bit，重置FIFO，最后再重新打开中断。

对于其中的 `WriteReg(IER, 0x00)` 就是把 `0x00` 写入到 IER register, 下同

UART 中的寄存器

![](https://s2.loli.net/2023/12/23/IdOHv6JsNlzQyRm.png)

![](https://s2.loli.net/2023/12/23/lDH3f7mJxdktWwq.png)

运行完这个函数之后，原则上UART就可以生成中断了。但是因为我们还没有对PLIC编程，所以中断不能被CPU感知。最终，在main函数中，需要调用plicinit函数。

```c
void
plicinit(void)
{
  // set desired IRQ priorities non-zero (otherwise disabled).
  *(uint32*)(PLIC + UART0_IRQ*4) = 1;     // 0x0c000000L + 10*4
  *(uint32*)(PLIC + VIRTIO0_IRQ*4) = 1;   // 0x0c000000L + 1*4
}
```

第一句是：设置 PLIC 会接收来自 **UART** 的中断
第二句是：设置 PLIC 会接收来自 **IO磁盘** 的中断

main 函数中，plicinit 之后就是 plicinithart 函数。plicinit 是由0号 CPU 运行，之后，每个 CPU 的核都需要调用 plicinithart 函数表明对于哪些外设中断感兴趣。所以在 plicinithart 函数中，每个 CPU 的核都表明自己对来自于 UART 和 VIRTIO 的中断感兴趣。因为我们忽略中断的优先级，所以我们将优先级设置为0。

```c
void
plicinithart(void)
{
  int hart = cpuid();
  // set uart's enable bit for this hart's S-mode.
  *(uint32*)PLIC_SENABLE(hart)= (1 << UART0_IRQ) | (1 << VIRTIO0_IRQ);

  // set this hart's S-mode priority threshold to 0.
  *(uint32*)PLIC_SPRIORITY(hart) = 0;
}
```

到目前为止，我们有了生成中断的外部设备，我们有了 PLIC 可以传递中断到单个的 CPU。但是 CPU 自己还没有设置好接收中断，因为我们还没有设置好 SSTATUS 寄存器。
在 main 函数的最后，程序调用了 scheduler 函数

```c
void
scheduler(void)
{
  struct proc *p;
  struct cpu *c = mycpu();
  c->proc = 0;
  for(;;){
    // Avoid deadlock by ensuring that devices can interrupt.
    intr_on();                  //
    for(p = proc; p < &proc[NPROC]; p++) {
      acquire(&p->lock);
      if(p->state == RUNNABLE) {
        // Switch to chosen process.  It is the process's job
        // to release its lock and then reacquire it
        // before jumping back to us.
        p->state = RUNNING;
        c->proc = p;
        swtch(&c->context, &p->context);

        // Process is done running for now.
        // It should have changed its p->state before coming back.
        c->proc = 0;
      }
      release(&p->lock);
    }
  }
}
```

## 来自 Shell 程序的输出

### (top 部分)

在上面 scheduler 函数其实蛮有趣的，它主要的功能就是为每个 CPU 进程调度程序。是无限循环的，为切换到所选进程，将进程 context switch  如： `swtch(&c->context, &p->context);`

在操作系统中，当内核完成所有的初始化操作后，会进入到第一个用户进程。这个用户进程通常是由 `init` 进程启动的，`init` 进程是所有进程的祖先进程，它是系统启动时自动创建的第一个用户进程。init 进程的 PID 为 1，它负责启动和管理其他用户进程，是系统中最后一个终止的进程。

XV6 中的 **init** main 函数（user/init.c : main）

首先这个进程的main函数创建了一个代表Console的设备。这里通过mknod操作创建了console设备。因为这是第一个打开的文件，所以这里的文件描述符0。之后通过dup创建stdout和stderr。这里实际上通过复制文件描述符0，得到了另外两个文件描述符1，2。最终文件描述符0，1，2都用来代表Console。

```c
  dup(0);  // stdout        fd = 1
  dup(0);  // stderr        fd = 2
```

之后 `fork()` 子进程运行 `exec("sh", argv)` ，来到 `sh.c : main` 。Shell程序首先打开文件描述符0，1，2。

```c
main：
  // Ensure that three file descriptors are open.
  while((fd = open("console", O_RDWR)) >= 0){
    if(fd >= 3){
      close(fd);
      break;
    }
  }
```

之后Shell向文件描述符2打印提示符“$ ”。

```c
int
getcmd(char *buf, int nbuf)
{
  fprintf(2, "$ ");
  memset(buf, 0, nbuf);
  gets(buf, nbuf);
  if(buf[0] == 0) // EOF
    return -1;
  return 0;
}
```

值得一提的是：在 Unix 系统中，设备是由文件表示。
因此，shell 只是向 fd = 2 的 **`文件`** 写了 `$` ，也就是说，fprintf () 函数用到了中断，而且是 write() 系统调用。所以由 Shell 输出的每一个字符都会触发一个 write 系统调用。最终来到 filewrite 函数(file.c) 。

在 filewrite 函数中首先会判断文件描述符的类型。mknod 函数生成的文件描述符属于设备（FD_DEVICE = 3），而对于设备类型的文件描述符，我们会为这个特定的设备执行设备相应的 write 函数。因为我们现在的设备是 Console，所以我们知道这里会调用 console.c 中的 consolewrite 函数。

这里先通过either_copyin将字符拷入，之后调用uartputc函数。uartputc函数将字符写入给UART设备，所以你可以认为consolewrite是一个UART驱动的top部分。uart.c文件中的uartputc函数会实际的打印字符。

```c
void
uartputc(int c)
{
  acquire(&uart_tx_lock);

  if(panicked){
    for(;;)
      ;
  }

  while(1){
    if(uart_tx_w == uart_tx_r + UART_TX_BUF_SIZE){
      // buffer is full.
      // wait for uartstart() to open up space in the buffer.
      sleep(&uart_tx_r, &uart_tx_lock);
    } else {
      uart_tx_buf[uart_tx_w % UART_TX_BUF_SIZE] = c;
      uart_tx_w += 1;
      uartstart();
      release(&uart_tx_lock);
      return;
    }
  }
}
```

uartputc 函数会稍微有趣一些。在 UART 的内部会有一个 buffer 用来发送数据，buffer 的大小是32个字符。同时还有一个为 consumer 提供的读指针和为 producer 提供的写指针，来构建一个环形的 buffer（注，或者可以认为是环形队列）。

在函数中第一件事情是判断环形 buffer 是否已经满了。如果读写指针相同，那么 buffer 是空的，如果写指针加1等于读指针，那么 buffer 满了。当 buffer 是满的时候，向其写入数据是没有意义的，所以这里会 sleep 一段时间，将 CPU 让出给其他进程。当然，对于我们来说，buffer 必然不是满的，因为提示符“$”是我们送出的第一个字符。所以代码会走到 else，字符会被送到 buffer 中，更新写指针，之后再调用 uartstart 函数。

```c
void
uartstart()
{
  while(1){
    if(uart_tx_w == uart_tx_r){
      // transmit buffer is empty.
      return;
    }

    if((ReadReg(LSR) & LSR_TX_IDLE) == 0){
      // the UART transmit holding register is full,
      // so we cannot give it another byte.
      // it will interrupt when it's ready for a new byte.
      return;
    }
    int c = uart_tx_buf[uart_tx_r % UART_TX_BUF_SIZE];
    uart_tx_r += 1;
    // maybe uartputc() is waiting for space in the buffer.
    wakeup(&uart_tx_r);   //
    WriteReg(THR, c);     //
  }
}
```

uartstart就是通知设备执行操作。首先是检查当前设备是否空闲，如果空闲的话，我们会从buffer中读出数据，然后将数据写入到THR（Transmission Holding Register）发送寄存器。这里相当于告诉设备，我这里有一个字节需要你来发送。一旦数据送到了设备，系统调用会返回，用户应用程序Shell就可以继续执行。这里从内核返回到用户空间的机制与trap机制是一样的。

### bottom 部分

在我们向 Console 输出字符时，如果发生了中断，RISC-V 会做什么操作。
我们之前已经在 SSTATUS 寄存器中打开了中断，所以处理器会被中断。假设键盘生成了一个中断并且发向了 PLIC，PLIC 会将中断路由给一个特定的 CPU 核，并且如果这个 CPU 核设置了 SIE 寄存器的 E bit（注，针对外部中断的 bit 位），那么会发生以下事情：

1. 会清除 SIE 寄存器相应的 bit，这样可以阻止 CPU 核被其他中断打扰，该 CPU 核可以专心处理当前中断。处理完成之后，可以再次恢复 SIE 寄存器相应的 bit。
2. 会设置 SEPC 寄存器为当前的程序计数器。我们假设 Shell 正在用户空间运行，突然来了一个中断，那么当前 Shell 的程序计数器会被保存。
3. 要保存当前的 mode。在我们的例子里面，因为当前运行的是 Shell 程序，所以会记录 user mode。
4. 再将 mode 设置为 Supervisor mode。
5. 最后将程序计数器的值设置成 STVEC 的值。

在 trap.c 中有可以判断中断的部分：

```c
usertrap：
else if((which_dev = devintr()) != 0){
    // ok
    }
```

在 trap.c 的 devintr 函数中，首先会通过 SCAUSE 寄存器判断当前中断是否是来自于外设的中断。如果是的话，再调用 plic_claim 函数来获取中断。

```c
// ask the PLIC what interrupt we should serve.
int
plic_claim(void)
{
  int hart = cpuid();
  int irq = *(uint32*)PLIC_SCLAIM(hart);
  return irq;
}
```

plic_claim 函数位于 plic.c 文件中。在这个函数中，当前 CPU 核会告知 PLIC，自己要处理中断，PLIC_SCLAIM 会将中断号返回，对于 UART 来说，返回的中断号是10。

```c
devintr:
if(irq == UART0_IRQ){
      uartintr();
    }
```

当 `irq` 为 10 的时候，就调用 ` uartintr() ` 函数，向 UART 发送数据，但现在没有输入数据，UART 的寄存器为空。

```c
void
uartintr(void)
{
  // read and process incoming characters.
  while(1){
    int c = uartgetc();
    if(c == -1)
      break;
    consoleintr(c);
  }

  // send buffered characters.
  acquire(&uart_tx_lock);
  uartstart();
  release(&uart_tx_lock);
}
```

这个函数会将 Shell 存储在 buffer 中的任意字符送出。实际上在提示符 `"$"` 之后，Shell 还会输出一个空格字符，write 系统调用可以在 UART 发送提示符 `"$”` 的同时，并发的将空格字符写入到 buffer 中。所以 UART 的发送中断触发时，可以发现在 buffer 中还有一个空格字符，之后会将这个空格字符送出。

## 中断并行

驱动的 top 和 bottom 部分是并行运行的。例如，Shell 会在传输完提示符“$”之后再调用 write 系统调用传输空格字符，代码会走到 UART 驱动的 top 部分（注，uartputc 函数），将空格写入到 buffer 中。但是同时在另一个 CPU 核，可能会收到来自于 UART 的中断，进而执行 UART 驱动的 bottom 部分，查看相同的 buffer。（buffer 是对于 top 和 bottom 的公有队列，对于 CPU 也一样）所以一个驱动的 top 和 bottom 部分可以并行的在不同的 CPU 上运行。

我们在之前的代码中出现了许多 `lock` ，这是来管理并行。我们想要的是 `buffer` 只可以在同一时间被唯一个 CPU 使用。

在 UART 中producer/consumser并发：

producer （shell）可以一直写入数据，直到**写指针 + 1等于读指针**，因为这时，buffer 已经满了。当 buffer 满了的时候，producer 必须停止运行。我们之前在 uartputc 函数中看过，如果 buffer 满了，代码会 `sleep`，暂时搁置 Shell 并运行其他的进程。

Interrupt handler，也就是 uartintr 函数，在这个场景下是 consumer ，每当有一个中断，并且读指针落后于写指针，uartintr 函数就会从读指针中读取一个字符再通过 UART 设备发送，并且将读指针加1。当读指针追上写指针，也就是两个指针相等的时候，buffer 为空，这时就不用做任何操作。

## 来自键盘的输出

Shell 会通过调用 `read 函数` 从键盘中读取字符，而接着调用 `fileread 函数`，因为是外设中断，之后调用 `consoleread 函数` 和写类似。
但是在这个场景下 Shell 变成了 `consumser`，因为 Shell 是从 buffer 中读取数据。而键盘是 `producer`，它将数据写入到 buffer 中。

输入一个字符的：

1. 检测读指针和写指针是否一样，判断 buffer 是否为空，进程是否会 sleep。
2. 在 Shell 在打印完 `“$ ”` 之后，如果键盘没有输入，Shell 进程会 sleep，直到键盘有一个字符输入。
3. 所以在某个时间点，假设用户通过键盘输入了 `“l”`，这会导致“l”被发送到主板上的 UART 芯片，产生中断之后再被 PLIC 路由到某个 CPU 核，之后会触发 `devintr 函数`，`devintr` 可以发现这是一个 UART 中断，然后通过 `uartgetc 函数` 获取到相应的字符，之后再将字符传递给 `consoleintr 函数`。这里面有对 ASCII 码的应用（`#define C(x)  ((x)-'@')`  Control-x）。

默认情况下，字符会通过 consputc，输出到 console 上给用户查看。之后，字符被存放在 buffer 中。在遇到换行符的时候，唤醒之前 sleep 的进程，也就是 Shell，再从 buffer 中将数据读出。

所以这里也是通过buffer将consumer和producer之间解耦，这样它们才能按照自己的速度，独立的并行运行。如果某一个运行的过快了，那么buffer要么是满的要么是空的，consumer和producer其中一个会sleep并等待另一个追上来。
