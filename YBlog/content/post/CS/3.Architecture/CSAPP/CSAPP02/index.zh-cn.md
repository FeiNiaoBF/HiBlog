---
title: "二.程序的机器级表示(CSAPP)"
date: 2023-04-09T20:13:06+08:00
draft: false
tags: [CSAPP]
categories: ["CSAPP","CS"]
author: ["Yeelight"]
showtoc: true
weight: 3
readingTime: true
---

# 入门

## 微处理器历史

### Intel的x86

在这里我想来简单的说道说道微处理器的历史发展，特此说明一下我不是专业的🙄，因此我没有详细的`深入研究`，如果有任何的错误请告诉我，谢谢。

自从1971年的Intel4004既第一款微处理器，也是全球第一款微处理器开始，我们人类社会标志着进入微芯片时代，在这个时代有三个主要的趋势：

- 处理器的位长的倍增
- 指令集的快速发展
- 时钟频率的快速增加

Intel也逐步发布了`Intel 8008`一个8位的，`Intel 8086` 一个16位的，至此，Intel的x86`帝国`开始了。在1985年，Intel的32位处理器`IA32`问世，而随着摩尔定律等的种种限制，单核的处理器已经遇到瓶颈了，各大公司继而转向了高频率、低功耗的多核处理器，处理器进入多核/多线程时代（2005）。

在一些无论是竞争关系，还是研究关系，导致目前的市场上出现了两种**指令集计算机**

- CISC(*Complex instruction set computer*)
- RISC(*Reduced instruction set computer*)

在后面我会再提到的。

我不知道未来的处理器向哪个方向发展，也不知道Intel是否一直在前沿（AMD：呵呵），但我相信，人类的智慧使得世界自第二次工业革命以来史无前例的大发展，在未来一定有着不一样的发展。

## 基本概念

>嗯嗯，回来回来，不去想未来，做好当下。

在了解具体的体系系统前，我们来了解基本的一些东西：

![CS61cCS](https://s2.loli.net/2023/04/24/Jp2IxPhfSWjHNZz.png)

上面是来自CS61c的图片，我们现在要了解就是整个软件到硬件的过程，也可以说是抽象到具体的过程。

**Instructure Set Architecture**：指令集架构 (包括指令规格，不同规则寄存器等)，简称ISA，它是软硬件之间的`桥梁`。

**Processor**、**Memory**、**I/O system**：这是由OS进行控制的。

我们这章主要的是学习ISA，CSAPP主要是x86-64的CISC指令集，CS61c主要是RISC-V的RISC指令集，没错，它们是不同的指令集，在我的笔记里面我也会不时的写上RISC-V的一些表示来证明我学习过了（笑）。

### 什么是编译

```txt

高级语言 --> 汇编语言 --> 机器语言

```

在我们的零章的时候我说过一个*.c文件如何变成的一个可执行的程序的一个主要过程，它有一个步骤是**编译**，这是一个我们需要细细品味的步骤。

编译过程是一个由某个高级语言（比如c文件）经过编译器的一系列的处理成为可读性低的汇编语言。换而言之，就是把我们十分清楚明白的抽象语言转换成机器语言（值得注意的是，此时机器也不知道汇编语言），在经历汇编译器**翻译**成二进制代码，真正的机器语言，机器可以读懂了，但我们看不懂（除了某些黑客）。

![CALL](https://s2.loli.net/2023/04/24/ARQExTsuyrhdgYO.png)


上面是cs61c中的*从C到机器语言*的完整过程，十分的详细了。程序的运行就是想像是一个**翻译**过程，用上一些我们明文规定的语法规则，使用编译器（GCC等）来当我们程序员和机器之间的**翻译官**。

现在来看一看从 `C语言` 到`机器代码`(一个整型加法计算的汇编代码)

```c
  // filename: clcyle_one
 
#include <stdio.h>
int main(int argc, char const *argv[])
{
    int sum = 0;
    for (int i = 0; i < 10; ++i)
    {
        sum += i;
    }
    return 0;
}
```

经过gcc的编译，在自己的Linux机器上使用以下的代码

```shell
$ gcc -Og -S clcyle_one.c
```

```asm
    .file   "clcyle_one.c"
    .text
    .globl  main
    .type   main, @function
main:
.LFB7:
    .cfi_startproc
    movl    $0, %eax
    jmp .L2
.L3:
    addl    $1, %eax
.L2:
    cmpl    $9, %eax
    jle .L3
    movl    $0, %eax
    ret
    #（不用在意类似 .file 的指令，它们是伪指令， 要从main看起）
```

两者相互比较一下我们可以发现，C 语言代码被处理成统一格式的汇编代码，在汇编代码中，第一个字符串叫做操作符，后面的是源和目的`寄存器`（这是一个大玩意😣）。操作和操作数明确，在下面我们会到不同的操作符分类分析（我想把RISC-V的加入作比较），记住一个条件，`读取`/`运算操作`是一个**线性逐句逐次**的操作，PC指令是有现态和次态之分。

## 处理器的工作

在我们对操作指令分类讨论之前我们来认识处理器是怎么工作的，

![x86-64](https://s2.loli.net/2023/04/24/3isvAmjMY8E9IgP.jpg)



![cs61c cpu](https://s2.loli.net/2023/04/24/Fzb3uHQBLTlqOgD.png)



上面的图片(分别来自CSapp和CS61c)十分清楚的展示了处理器对于存放在主存里面的指令有着什么样的操作，主要的就两点**存、读取值**和**计算**。在x86-64里面还有一个叫作**条件码**的东东，我会在下面说到因为我也第一次看见这个。



这是一个**CPU到Memory**的一个过程，具体的是一个处理器从内存某个地址取值（有数据和指令）拿到**CPU**里的**寄存器**通过**ALU**计算，再根据**PC**选择下一步。



- **程序计数器**(PC, Program counter) - 存着下一条指令的地址，在 x86-64 中称为 RIP。

- **寄存器**(Register) - 用来存储数据以便操作。

- **条件代码**(Codition codes) - 通常保存最近的算术或逻辑操作后的信息，用来做条件跳转的条件。

## 什么是ISA

**Instruction Set Architecture** (指令集框架) 是包含了针对某个特定处理器执行的基本操作码（*opcode*），里面是基本命令，在我们学习的`x86-64`、`RISC-V`都有不同的ISA。

为了学习方便，我们一般要把ISA根据使用的方法不同进行不同的分类。

- 资料处理与访问操作
- 算术逻辑操作
- 控制过程操作


在下面我会一一分析。

## CISC&RISC区别

CISC(*Complex instruction set computer*)

> 兼容性性强，指令繁多，长度可变，由微程序实现。
> 代表：x86-64

RISC(*Reduced instruction set computer*)

> 指令少，使用频率接近，主要是依靠硬件实现（通用寄存器、硬布线逻辑控制）。
> 代表：RISC-V


# 开始快乐的汇编

## 整型寄存器

![Reg](https://s2.loli.net/2023/04/24/pYIjkJMEBKc413b.png)

在x86-64上有16个64位的**通用寄存器**；对于每个寄存器的低32、16和8位可以独立地通过其他不同指令名称访问，原则上，几乎任何寄存器都可以用于保存几乎任何逻辑和算术操作的操作数，但有些具有特殊或受限制的用途。

一个寄存器有着64位、32位、16位、8位这可以对以前的低位程序向下兼容。

按照惯例，%rsp被保留作为堆栈指针，并且因为一些指令（例如push、pop、call）隐含使用它。%rsp指向最低占用的堆栈位置（而不是下一个要使用的位置）。

寄存器%rbp有时被用作帧指针，即当前堆栈帧的基址。指令计数器寄存器（％rip） 指向要执行的下一条命令; 程序员无法直接访问它, 但是大量地被用作基于位置无关代码寻址的基础。还有一些其他指令隐含地使用某些寄存器；例如，整数乘法和除法指令需要%rax和%rdx。

## 数据类型

既然我们的寄存器有着不同位的表示，那就是说处理器在操作不同的数据时使用着不同位数的寄存器来提高速率。

我们针对不同的数据类型使用不同的**suffix**和**size**：

| Data type | suffix | size | word|
| :---: | :---: | :---: | :---: |
| char | b | 1B | Byet |
| short | w | 2B | 1word |
| int | l | 4B | 2word |
| long | q | 8B | 4word |
| char* | q | 8B | 4word |
| float | s | 4B | 2word |
| double | l | 8B | 4word |

上面的**suffix**列显示了在 GNU 汇编程序用来指定适当大小的变体的字母指示。

而一个新单位**word**是相等于2个字节的大小。

## 操作数指示符

### 操作数基本

以下的都是在操作数里面主要数值表达的意思：

- **Imm**   refers to a constant value, e.g. 0x8048d8e or 48
- **r**     refers to a register.  e.g. %rax or %edi
- **R[r]**  refers to the value stored in register address r.
- **M[i]**  refers to the value stored at memory address i .

不同的格式表示不同的类型。

### 寻址

> 很重要！！！

对于寻址来说，比较通用的格式是：` Imm(Rb, Ri, S) -> M[R[Rb] + S*R[Ri]+ Imm] `，其中：

- **Imm** - 常数偏移量
- **Rb** - 基寄存器
- **Ri** - 索引寄存器，不能是 %rsp
- **S** - 系数

![OperandSpecifiers](https://s2.loli.net/2023/04/24/LPK7kRoNwmFVfi2.png)

## 指令

接下来我们就来看看不同分类的指令格式，我按书上的顺序来说的，它也是按我们在平时使用的频率顺序来教的。

大多数的指令都是使用上文提过的 **suffix** 来显示操作数的大小的。

### 数据移动指令

对于 **mov** 指令来说，需要**源操作数**和**目标操作数**。指令的具体格式可以这样写 `mov? Src, Dest`，第一个是源操作数，第二个是目标操作数

```asm
mov[b|w|l|q] Src, Dest                             # 将src移动到dest
movs[bw|bl|bq|wl|wq|lq] Src, Dest                  # 带符号扩展的移动
movz[bw|bl|bq|wl|wq] Src, Dest                     # 带零扩展的移动
movabsq imm, r                                     # 移动绝对四字（imm为64位）

cltq Src, Dest                                     # 把%eax 符号扩展到%rax
```

在使用 **mov** 指令的时候需要值得注意的是我们的源值和目的值的选址是有标准的, 源操作数可以是立即数、寄存器值或内存值的任意一种，但目标操作数只能是寄存器值或内存值

|Src| Dest |
| :--: | :--: |
| imm | Rag |
| imm | Mem |
|Reg| Reg |
|Reg|Mem|
|Mem|Reg|

> 只有这五种的选择 ， 如果要把Mem -> Mem 的值移动，需要两步 Mem -> Reg -> Mem


### 程序栈指令

这一部分就只有两个主要的指令，但是无比的重要。可以把数据压入程序栈中，以及在栈中弹出，程序栈在过程调用中起至关重要的作用。

```asm
pushq  Src                    # 将4word的数据压入栈，并把%rsp - 8 -> %rsp
popq  Dest                    # 将4word的数据弹入栈，并把%rsp + 8 -> %rsp
```

对于程序栈指令十分重要的一点是我们对内存的变化要注意。在程序员的眼里内存是一个有限的数组，我们在把寄存器里面的数据 `push` 进内存的时候栈指针（%rsp）要向着地址减小的方向移动，这就是 `%rsp - 8` 的原因。

图片

### 算术与逻辑指令

对于算术指令我们想起CPU中最重要的部件 `ALU` 算术逻辑单元，基本上所有的这些指令通过
 `opcode` 来在多路选择上 `指挥` ALU正确的使用算术。

 > 注：下面的所有指令都可以根据数据类型加 suffix (b/w/l/q)

#### Unary Operation(一元操作)

```asm
inc Deat         Deat+1->Deat           # 按1递增
dec Deat         Deat-1->Deat           # 按1递减
neg Deat         -Deat->Deat    (取反)  # 算术取反
not Deat         ~Deat-1->Deat （取补） # 按位取反
```

一元操作只有一个操作数，即做源也是目的。可以是`Reg or Mem` 。

#### Binary Operation(二元操作)

```asm
leaq S，D      &S -> D        # 将源地址的有效地址加载到目标中
add S，D      D + S -> D      # 将源加到目标中
sub S，D      D - S -> D      # 将源从目标中减去
imul S，D     D * S -> D      # 目标乘以源
xor S，D      D ^ S -> D      # 按位异或目标和来源
or S，D       D | S -> D      # 按位或目标和来源
and S, D      D & S -> D      # 按位与目标和来源
```

对于第二个到最后一个不需要再说了，都是字面意思。主要来说一说 `leaq` 这个指令。

Load effective address(加载有效地址) , leaq 有两个作用：

> - 将其源操作数的有效地址（而不是该地址处的数据）加载到其目标寄存器中
>   在 C语言里面就是 `&S` , 这样的好处是可以给下面的内存产生指针。
> - 也可用于执行与寻址无关的算术运算。（eg： `leaq (%rdi, %rsi, 4), %rax` 相同与 `x + 4*y ` )

#### Shift Operations(移位操作)

```asm
sal[b|w|l|q] imm,d   d = d << imm   # 左移imm位
sar[b|w|l|q] imm,d   d = d >> imm   # 算术右移imm位
shr[b|w|l|q] imm,d   d = d >> imm   # 逻辑右移imm位
```

#### Special Arithmetic Operations(特殊算术操作)

```asm
imulq S                  # 有符号全乘法   四字到八字
mulq S                   # 无符号全乘法   四字到八字
idivq S                  # 有符号全除法   八字到四字
divq S                   # 无符号全除法   八字到四字
cltd                     # sign extend %eax into %edx::%eax
cqto                     # sign extend %rax into %rdx::%rax
```

在特殊算术里面，这样的设计是为了补码的乘除有扩展。由两个64位的到全128位的乘积和整数除法的截断。

除法需要特殊的安排：`idiv（有符号）` 和 `div（无符号）` 操作在2n字节被除数和n字节除数上，产生一个n字节商和n字节余数。被除数总是存在于一对固定寄存器中（32位情况下为%edx和%eax；64位情况下为%rdx和%rax）；除数作为指令中的源操作数来指定。商放在％eax（resp. ％rax）中; 余数放在％edx（resp. ％rdx）中。对于有符号的除法，使用cltd（resp.ctqo）指令来准备％edx(resp.%rdx)，并将其与％eax(resp.%rax)的符号扩展配合使用。例如，如果a、b、c是保存四个字长的内存位置，则可以使用以下序列设置c = a / b：

```asm
    movq a(%rip), %rax
    ctqo
    idivq b(%rip)
    movq %rax, c(%rip)
```

上文来自文档）


### 控制指令

到目前为止，我们看到的都是顺序一条接着一条的操作的，但是在我们的c语言里面还有条件语句（if）、循环语句（while）、分支语句（switch）等，很明显都不是顺序的，要进行某种**跳转**，而这种跳转是由机器代码来实现的，根据测试数据值来判断机器此时是否改变控制流。

#### 条件码

好了，到我们心心念的**条件码**了，条件码在CPU中是有单独的**条件码寄存器**，但是它只有单个位，它们描述的是距离最近的算术和逻辑操作的某些属性，CPU根据条件码寄存器来断定是否执行分支跳转。以下是四种条件码：

- **ZF** result was Zero
- **CF** result caused Carry out of most significant bit (unsigned)
- **SF** result was negative (Sign bit was set)
- **OF** result caused (**signed**) Overflow （negative overflow, positive overflow）

> leaq 指令不改变任何的条件码。而不是所有的指令都要改变，如**xor**对于CF、OF会设置为0。

指令集中也有专门来设置条件码的指令，它们不会改变任何的其他寄存器，只会改变条件码：

```asm

cmp[b|w|l|q]  s2,s1                         # 比较两个值，S1 - S2 用减法的方法来比较

test[b|w|l|q] s2,s1                         # 测试两个值，S1 & S2 可以来检查是负or正，也可以比较具体位的值

```

其实cmp和test有时是十分好用的测试指令，比如在对（x == 0）的时候，可以用 `cmpl %eax, %eax` 或者 `testl %eax, %eax` 来与自己比较来设置**ZF**条件码，也用来判断 `%eax` 是正数或负数。

#### 访问条件码

条件码通常是不会直接读取的，在x86中采用三种使用方式：

- 可以根据条件码的某种位逻辑组合，将一个字节设置为0 或1。
- 可以条件跳转到程序的某个其他的部分。
- 可以有条件地传送数据。


> 这里会发现用了位的逻辑计算来确认大于或小于等情况。（需要好好看看第二章）

1. 第一点的实现 **SET指令**

```asm
sete / setz   D        Set if equal/zero                               ZF
setne / setnz D        Set if not equal/nonzero                      ~ ZF
sets          D        Set if negative                                 SF
setns         D        Set if nonnegative                            ~ SF
setg / setnle D        Set if greater (signed)                 ~ (SF ^ 0F)& ~ ZF
setge / setnl D        Set if greater or equal (signed)           ~ (SF ^ 0F)
setl / setnge D        Set if less (signed)                           SF^0F
setle / setng D        Set if less or equal                       (SF ^ OF)|ZF
seta / setnbe D        Set if above (unsigned)                     ~ CF& ~ ZF
setae / setnb D        Set if above or equal (unsigned)              ~ CF
setb / setnae D        Set if below (unsigned)                         CF
setbe / setna D        Set if below or equal (unsigned)               CF|ZF
```

SET指令，每条指令根据条件码的各种组合将一个字节设置为 **0或1**。

2. 第二点的实现 **Jump指令**

```asm
jmp Label             Jump to label                                   true
jmp *Operand          Jump to specified location                      true
je / jz Label         Jump if equal/zero                               ZF
jne / jnz Label       Jump if not equal/nonzero                       ~ ZF
js Label              Jump if negative                                 SF
jns Label             Jump if nonnegative                             ~ SF
jg / jnle Label       Jump if greater (signed)                ~ (SF ^ 0F)& ~ ZF
jge / jnl Label       Jump if greater or equal (signed)           ~ (SF ^ 0F)
jl / jnge Label       Jump if less (signed)                           SF^0F
jle / jng Label       Jump if less or equal                       (SF ^ OF)|ZF
ja / jnbe Label       Jump if above (unsigned)                    ~ CF& ~ ZF
jae / jnb Label       Jump if above or equal (unsigned)               ~ CF
jb / jnae Label       Jump if below (unsigned)                         CF
jbe / jna Label       Jump if below or equal (unsigned)              CF|ZF
```

跳转(jump) 指令会导致执行切换到程序中一个全新的位置。在汇编代码中，这些跳转的目的地通常用一个标号(**Label**) 指明。在下一个标题再继续深入jump指令。

3. 第三点的实现 **cmove指令**

```asm
cmove / cmovz   S, D   Move if equal/zero                               ZF
cmovne / cmovnz S, D   Move if not equal/nonzero                       ~ ZF
cmovs           S, D   Move if negative                                 SF
cmovns          S, D   Move if nonnegative                             ~ SF
cmovg / cmovnle S, D   Move if greater                (signed)    ~ (SF ^ 0F)& ~ ZF
cmovge / cmovnl S, D   Move if greater or equal       (signed)      ~ (SF ^ 0F)
cmovl / cmovnge S, D   Move if less (signed)                           SF^0F
cmovle / cmovng S, D   Move if less or equal                       (SF ^ OF)|ZF
cmova / cmovnbe S, D   Move if above (unsigned)                     ~ CF& ~ ZF
cmovae / cmovnb S, D   Move if above or equal        (unsigned)        ~ CF
cmovb / cmovnae S, D   Move if below                 (unsigned)         CF
cmovbe / cmovna S, D   Move if below or equal        (unsigned)        CF|ZF
```

条件传送指令, 但传送条件满足的时候,指令把`S`复制到`D`中。

#### C语言中的条件分支

现在来进行对C语言中一些常见的分支跳转操作来看看翻译后的机器代码。

##### if-else

C 语言中的江-else 语旬的通用形式模板如下：

```c
if (test-expr)
    then-statement
else
    els-statement
```

汇编器工作是为 `then-statement` 和 `else-statement` 产生各自的代码块。它会插入条件和无条件分支，以保证能执行正确的代码块。

看一个例子

```c

long absdiff(long x, long y)
{
    long result;
    if (x < y)
        result = y-x;
    else
        result = x-y;
    return result;
}

```

```c

long absdiff_es(long x, long y)
{
    long result;
    if (x > y)
        result = x-y;
    else
        result = y-x;
    return result;
}

```

分别产生的汇编代码

```asm
# x in %rdi, y in %rai
absdiff :
    movq %rsi, %rax
    subq %rdi, %rax          rval = y-x
    movq %rdi, %rdx
    subq %rsi, %rdx          eval = x-y
    cmpq %rsi, %rdi          比较 x:y
    cmovge %rdx, %rax       If >=, rval = eval
    ret                      Return tval

```

```asm

absdiff_es:
    cmpq    %rsi, %rdi
    jle     .L4
    movq    %rdi, %rax
    subq    %rsi, %rax
    ret
.L4:    # x <= y
    movq    %rsi, %rax
    subq    %rdi, %rax
    ret
```

> [为什么基于条件数据传送(1)的代码会比基于条件控制转移(2)的代码性能要好？]

##### while

while 语句的通用形式如下：

```c
while (test-expr)
    body-statement
```

例子

```c
long fact_while(long n)
{
    long result = 1;
    while (n > 1) {
        result *= n;
        n = n-1;
    }
    return result;
}
```

##### do-while

do-while 语句的通用形式如下：

```c
do
    body-statement
    while (test-expr);
```

例子

```c

// Do While 的 C 语言代码
long pcount_do(unsigned long x)

{
    long result = 0;
    do {
        result += x & 0x1;
        x >>= 1;
    } while (x);
    return result;
}
```

产生的汇编代码

```asm

    movl    $0, %eax    # result = 0
.L2:                    # loop:
    movq    %rdi, %rdx
    andl    $1, %edx    # t = x & 0x1
    addq    %rdx, %rax  # result += t
    shrq    %rdi        # x >>= 1
    jne     .L2         # if (x) goto loop
    rep                 # ret
```

##### for

for 循环的通用形式如下：

```c
for (init-expr; test-expr; update-expr)
    body-statement
```

#### switch

switch 循环的通用形式如下：

```c
switch(n){
    case test-expr:
        body-statement
        break;
    case test-expr:
        body-statement
        break;
}

```

对于C语言的这些语法我只是在这里举出例子来，最好看看书上的讲解。

### 分支跳转(RISC-V)

对于这个分类其实我分给了RISC-V， 主要是在x86-64中的控制指令和RISC-V的分支跳转其实是一回事，主要区别是否使用的条件码（其实在我看来RISC-V也用了条件码，但是是隐式的使用）。

RISC-V的 **Branch Instruction**

```asm

（施工中🚧）

```

## 过程调用

过程是软件中一种很重要的抽象。它提供了一种封装代码的方式，用一组指定的参数和一个可选的返回值实现了某种功能。然后，可以在程序中不同的地方调用这个函数。设计良好的软件用过程作为抽象机制，隐藏某个行为的具体实现，同时又提供清晰简洁的接口定义，说明要计算的是哪些值，过程会对程序状态产生什么样的影响。不同编程语言中，过程的形式多样：函数(function) 、方法(method) 、子例(subroutine) 、处理函数(handler) 等等，但是它们有一些共有的特性。

过程调用主要有三个机制：

1. **控制传递**。包括如何开始执行过程代码，以及如何返回到开始的地方。本质上是代码执行地址的改变和切换。

2. **数据传递**。调用函数时要传给函数一些参数，在返回函数时也可能会将一些函数计算结果以返回值形式返回给原函数。本质上是数据传入新过程，又传回原过程。

3. **内存管理**。在过程进行时，如何分配内存空间；在过程返回后，如何销毁内存中存储的局部变量。

![callsk](https://s2.loli.net/2023/05/07/iWSVILyRpxh63Dl.jpg)

## 数据存储分配

## 外部链接

[摩尔定律---Wiki](https://zh.wikipedia.org/wiki/%E6%91%A9%E5%B0%94%E5%AE%9A%E5%BE%8B)

[微处理器---Wiki](https://zh.wikipedia.org/wiki/%E5%BE%AE%E5%A4%84%E7%90%86%E5%99%A8)

[*The 50 Year History of the Microprocessor*](https://zhuanlan.zhihu.com/p/511271401)

[芯片相关-- Cpu历史--AMD系列](https://zhuanlan.zhihu.com/p/464721330)

[芯片相关-- Cpu历史--intel系列](https://zhuanlan.zhihu.com/p/464413953)

[【读薄 CSAPP】贰 机器指令与程序优化](https://wdxtub.com/csapp/thin-csapp-2/2016/04/16/)

[RISC-V手册](http://riscvbook.com/chinese/RISC-V-Reader-Chinese-v2p1.pdf)

[RISC-V基本指令集概述](https://www.sunnychen.top/archives/riscvbasic)

[过程调用](https://segmentfault.com/a/1190000022161796)
