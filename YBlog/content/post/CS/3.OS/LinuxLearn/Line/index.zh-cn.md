---
title: "命令行的魅力"
date: 2024-03-10T16:38:17+08:00
draft: false
taps: ['command_line']
categories: ['linux', 'OS']
author: ["Yeelight"]
showtoc: true
weight:
math: false
readingTime: true
---


[命令行的艺术](https://github.com/jlevy/the-art-of-command-line/blob/master/README-zh.md)

[Missing-Semester](https://missing.csail.mit.edu/2020/)
[计算机教育中缺失的一课](https://missing-semester-cn.github.io/)

## 基础

- [ ] 学习 `Vim`  。
- [ ] 学会如何使用 `man` 命令去阅读文档。
  - [ ] `man bash`  [[Bash]]
- [x] 学会使用 `apropos` 去查找文档
- [ ] 知道有些命令并不对应可执行文件，而是在 Bash 内置好的，此时可以使用 `help` 和 `help -d` 命令获取帮助信息。
- [x] 你可以用 `type 命令` 来判断这个命令到底是可执行文件、shell 内置命令还是别名。
- [ ] 学会重定向
  - [ ] 了解标准输出 stdout 和标准错误 stderr。
  - [ ] 使用 `>` 和 `<` 来重定向输出和输入。
  - [ ] 学会使用 `|` 来重定向管道。明白 `>` 会覆盖了输出文件而 `>>` 是在文件末添加。

- [ ] 学会使用 `ssh` 进行远程命令行登录。

- [ ] 熟悉 Bash 中的任务管理工具。
  `&`，**ctrl-z**，**ctrl-c**，`jobs`，`fg`，`bg`，`kill`
- [ ] 学会使用特定符。
  - 通配符 `*`
- [ ] 学会基本的文件管理工具
- [ ] 熟悉正则表达式
    [[正则表达式]]

- [ ] 学会使用 `apt-get`，`yum`，`dnf` 或 `pacman` 等来查找和安装软件包。
