---
title: "MakeFile使用笔记"
date: 2023-04-24T19:08:45+08:00
draft: false
taps: ["tools", "MakeFile"]
categories: ["MakeFile"]
author: ["Yeelight"]
showtoc: true
math: false
readingTime: true
---

# MakeFile的编译和连接

> 对于大量的c语言文件一个很好的自动化工具,其实可以用到任何语言

## MakeFile的一般使用

### 使用规则

既然要用MakeFile，那就要知道它是怎么使用的；主要还是`编译`&`链接`，将大量的文件，通过直接或间接的方式来一键编译，就不必像`gcc -g -o pro1.c pro2.c pro3.c... filename`如此这般麻烦的编译了。

写入Make的文件的规则:

> 1. 如果这个工程没有编译过，那么我们的所有c文件都要编译并被链接。
> 2. 如果这个工程之中的某几个c文件被修改，那么我们只编译被修改的c文件，并链接目标程序。
> 3. 如果这个工程的头文件被改变了，那么我们需要编译引用了这几个头文件的c文件，并链接目标程序。

对makefile的书写规则：

```makefile
target ... : prerequisites ...
    command
    ...
    ...
```


> 1. **target:** 这个是的目标文件，也可以是一个执行文件，还可以是一个标签（label）。
> 2. **prerequisites:** 这个是一个依赖文件，是对`target`文件的输入。
> 3. **command:** 这个是对文件的命令具体操作。`eg：cc -o file.h`

简而言之，target这一个或多个的目标文件依赖于prerequisites中的文件，其生成规则定义在command中。

### make的使用技巧

#### 变量的使用

在makefile里面也是可以使用变量的，但是这是不可变的(是不是有点矛盾),它更像是c语言里面的宏(#define)。

```makefile
object = $(boo)
```

这样的好处是我们可以简化我们的make文件，是它不是这么的杂乱无章。


## 书写规则
