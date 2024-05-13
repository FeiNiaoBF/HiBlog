---
title: "操作系统的具体学习"
date: 2024-03-10T16:34:18+08:00
draft: false
taps: ['OS']
categories: ['linux', 'OS', 'xv6', 'learning']
author: ["Yeelight"]
showtoc: true
weight:
readingTime: true
---

## 通过 XV6 了解操作系统

我希望通过 6. S081 的学习来学习操作系统的强大，以下的是我在学习中的笔记：
[通过 XV6 来学习操作系统(OSTEP)]({{< relref "post/Gohh/4.OS/S6.081/xv6_learn/index.zh-cn.md" >}})

- [XV6的gdb调试]({{< relref "post/Gohh/4.OS/S6.081/xv6_gdb_debug/index.zh-cn.md" >}})
- [XV6的源码解析]({{< relref "post/Gohh/4.OS/S6.081/xv6_src/index.zh-cn.md" >}})
<!-- - [XV6的内存虚拟化]({{< relref "post/Gohh/4.OS/S6.081/xv6_vmem/index.zh-cn.md" >}}) -->
- [XV6中的trap]({{< relref "post/Gohh/4.OS/S6.081/xv6_trap/index.zh-cn.md" >}})
- [XV6的中断]({{< relref "post/Gohh/4.OS/S6.081/xv6_interrupt/index.zh-cn.md" >}})
- [XV6的锁和并行]({{< relref "post/Gohh/4.OS/S6.081/xv6_lock/index.zh-cn.md" >}})
- [XV6的进程和线程]({{< relref "post/Gohh/4.OS/S6.081/xv6_process/index.zh-cn.md" >}})
- [XV6的文件系统]({{< relref "post/Gohh/4.OS/S6.081/xv6_file_system/index.zh-cn.md" >}})

## 个人 Linux 的学习

[Linux个人学习记录]({{< relref "" >}})
[命令行的魅力]({{< relref "post/Gohh/4.OS/LinuxLearn/Line/index.zh-cn.md" >}})

### 分类

[文件描述符]({{< relref "post/Gohh/4.OS/LinuxLearn/FD/index.zh-cn.md" >}})
<!-- [管道]({{< relref "post/Gohh/4.OS/LinuxLearn/pipe/index.zh-cn.md" >}}) -->
[进程（Process）和 线程（Thread）]()

### 在 XV 6 中 Syscall 函数

|**系统调用**|**描述**|
|---|---|
|`int fork()`|创建一个进程，返回子进程的PID|
|`int exit(int status)`|终止当前进程，并将状态报告给wait()函数。无返回|
| `int wait(int *status)` |等待一个子进程退出; 将退出状态存入 `*status`; 返回子进程 PID。|
|`int kill(int pid)`|终止对应PID的进程，返回0，或返回-1表示错误|
|`int getpid()`|返回当前进程的PID|
|`int sleep(int n)`|暂停n个时钟节拍|
|`int exec(char *file, char *argv[])`|加载一个文件并使用参数执行它; 只有在出错时才返回|
|`char *sbrk(int n)`|按n 字节增长进程的内存。返回新内存的开始|
|`int open(char *file, int flags)`|打开一个文件；flags表示read/write；返回一个fd（文件描述符）|
|`int write(int fd, char *buf, int n)`|从buf 写n 个字节到文件描述符fd; 返回n|
|`int read(int fd, char *buf, int n)`|将n 个字节读入buf；返回读取的字节数；如果文件结束，返回0|
|`int close(int fd)`|释放打开的文件fd|
|`int dup(int fd)`|返回一个新的文件描述符，指向与fd 相同的文件|
|`int pipe(int p[])`|创建一个管道，把read/write文件描述符放在p[0]和p[1]中|
|`int chdir(char *dir)`|改变当前的工作目录|
|`int mkdir(char *dir)`|创建一个新目录|
|`int mknod(char *file, int, int)`|创建一个设备文件|
|`int fstat(int fd, struct stat *st)`|将打开文件fd的信息放入*st|
|`int stat(char *file, struct stat *st)`|将指定名称的文件信息放入*st|
|`int link(char *file1, char *file2)`|为文件file1创建另一个名称(file2)|
|`int unlink(char *file)`|删除一个文件|

​ 来自表1.2：xv6系统调用（除非另外声明，这些系统调用返回0表示无误，返回-1表示出错）

[fork() 函数]({{< relref "" >}})
[exec() 函数]({{< relref "" >}})
[exit() 函数]({{< relref "" >}})

## 课程教材

book-riscv-rev2

操作系统-三个简单的部分-ostep

鸟哥的Linux私房菜 基础学习篇 第四版 (鸟哥)

## MIT6.S081 的实验记录

[MIT6.S081 的实验问题]({{< relref "" >}})
