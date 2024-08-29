---
title: "XV6 的 gdb调试"
date: 2024-03-10T16:35:00+08:00
draft: false
taps: ['gdb', 'xv6', 'debug']
categories: ['OS', 'xv6', 'debug']
author: ["Yeelight"]
showtoc: true
math: false
readingTime: true
---

## 基础

1. 利用 `tmux` 来实现两个终端的同时运行，具体的学习可以[看这里](https://101.lug.ustc.edu.cn/Ch04/#tmux) 。
2. 需要准备的工具。

 ```zsh
 sudo apt-get update && sudo apt-get upgrade
 sudo apt-get install git build-essential gdb-multiarch qemu-system-misc        \        gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu
 ```

3. 在自己的 xv 6 目录下工作。
4. 练习 GDB 的基本学习 --> [这里](https://linuxtools-rst.readthedocs.io/zh-cn/latest/tool/gdb.html) 。

## 步骤

1. 进入 xv6 的工作目录  (对于我的)

```zsh
cd ~/workspace/xv6-labs-2021
```

如下：
![ xv6 的工作目录](https://s2.loli.net/2023/11/29/CWyA7BolcYTSjwN.png)

2. 用 `tmux` 进入环境，分两个屏

![](https://s2.loli.net/2023/12/02/UenKoWS8h2CIGAj.png)
 第一个输入 `make qemu-gdb`  第二个输入 `gdb-multiarch`
 ![](https://s2.loli.net/2023/12/02/pPy8eNamC9VgRB7.png)
![](https://s2.loli.net/2023/12/02/5SIylXBwPn2qYdc.png)

在第二中输入执行 `set architecture riscv:rv64` 以调试 RISCV 架构，然后执行 `target remote localhost:26000`（这里端口号要看 `make qemu-gdb` 的输出）

![](https://s2.loli.net/2023/12/02/ZIw6R5UrmA4JWMo.png)

(未做完)
