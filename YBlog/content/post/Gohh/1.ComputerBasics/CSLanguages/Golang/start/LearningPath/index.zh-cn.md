---
title: "学习之路"
date: 2024-05-13T09:08:32+08:00
draft: false
taps: ["Golang", "start"]
categories: ["Golang", "CS", "Languages"]
author: ["Yeelight"]
showtoc: true
weight:
math: false
readingTime: true
---

## Hello, Golang

Go 语言是 Google 开发的一种**静态强类型**、**编译型**、**并发型**，并具有**垃圾回收功能**的编程语言。

>下载地址：[Downloads - The Go Programming Language](https://go.dev/dl/)

## 快速开始

- 下载 Go 到你的学习机上，检查 go 的现状

```shell
go version
```

- 在你喜欢的 Workspace 上 Create a new directory

```shell
mkdir gowork
```

- 添加一个名为 `hello.go` 的文件，在其中输入代码

> In vim, vscode ...

```go
package main

import "fmt"

func main() {
 fmt.Println("Hello, world")
}
```

> In shell

```shell
echo
"package main

import "fmt"

func main() {
 fmt.Println("Hello, world")
}" >> hello.go
```

- 运行它

```shell
go run hell.go
```

Welcome to Go!

## 其他

![](https://s2.loli.net/2024/05/06/pE3MQOLJI2lNGRY.png)
