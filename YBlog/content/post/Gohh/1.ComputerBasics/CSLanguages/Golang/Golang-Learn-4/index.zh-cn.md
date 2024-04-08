---
title: "Golang 语言笔记(4)"
date: 2024-03-10T16:04:11+08:00
draft: false
taps: ["Golang"]
categories: ["Golang", "CS", "Languages"]
author: ["Yeelight"]
showtoc: true
math: false
readingTime: true
---


## 包和函数

### 包

### 函数定义

在 Go 语言中，函数名的大小写是有特殊含义的。

1. 如果一个函数名以大写字母开头，那么它是可导出的（public），可以被其他包（package）访问和调用。
2. 如果一个函数名以小写字母开头，那么它是不可导出的（private），只能在定义该函数的包内部使用。

这种大小写的规则也适用于其他标识符，比如变量名、常量名和类型名。这种规则有助于控制包内部的可见性，使得包可以选择性地暴露或隐藏其内部实现的细节。
