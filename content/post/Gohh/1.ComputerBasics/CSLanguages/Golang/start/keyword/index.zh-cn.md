---
title: "关键字与标识符"
date: 2024-05-20T10:37:53+08:00
draft: false
taps: ["Golang", "start"]
categories: ["Golang", "CS", "Languages"]
author: ["Yeelight"]
showtoc: true
weight:
math: false
readingTime: true
---

## 25个关键字

```go
break     default      func    interface  select
case      defer        go      map        struct
chan      else         goto    package    switch
const     fallthrough  if      range      type
continue  for          import  return     var
```

## 规定的标识符（Unicode 字符）

- 标识符由字母（Unicode字母，包括中文等）、数字和下划线（`_`）组成。
- 标识符的第一个字符必须是字母或下划线。
- 标识符区分大小写，例如`count`和`Count`是两个不同的标识符。
- 标识符不能是Go语言的关键字。

注意：
 中文的标识符是 `private`
