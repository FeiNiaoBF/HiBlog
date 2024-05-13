---
title: "Strings"
date: 2024-05-13T09:10:39+08:00
draft: false
taps: ["Golang", "string"]
categories: ["Golang", "CS","start"]
author: ["Yeelight"]
showtoc: true
weight:
math: false
readingTime: true
---

## UTF-8编码和 Unicode 文本区别

在 Golang 中字符串的底层结构字节使用 UTF-8 编码来表示 Unicode 文本

其中 _Unicode_ 是一个**字符集**，定义了字符和它们的唯一**代码点**，而 _UTF-8_ 是一种**编码方式**，用于将 Unicode 字符表示为字节序列。

## 字符串

Go 语言中的字符串值是一个可空的**字节序列**，字节序列中的字节个数称为该字符串的长度。String 类型其实是一个"**描述符**”，它本身并不真正存储字符串数据，而是由一个指向底层存储的指针和字符串的长度字段组成的
