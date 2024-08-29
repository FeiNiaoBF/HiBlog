---
title: '数据结构和理论知识'
date: 2024-05-13T09:10:19+08:00
draft: false
taps: ['Golang', 'start']
categories: ['Golang', 'CS']
author: ['Yeelight']
showtoc: true
weight:
math: false
readingTime: true
---

## Go 语言中的数据结构

### 基础数据结构

1. **数组（Array）** - 由固定长度的相同类型元素组成的数据结构。
2. **切片（Slice）** - 由数组构成的动态长度序列，提供了更灵活的操作方式。
3. **映射（Map）** - 存储键值对的集合，用于快速检索数据。
4. **结构体（Struct）** - 可以包含不同类型字段的复合数据类型。

### 其他数据结构和类型

1. **通道（Channel）** - 用于在 Go 协程之间进行通信的类型。
2. **接口（Interface）** - 定义对象的行为，是一种抽象类型。
3. **指针（Pointer）** - 存储变量的内存地址，用于直接访问内存中的值。

## 命名规范

注意：

> **首字母大小写：** 以大写字母开头的标识符是 **public** 的（可导出的），可以被其他包访问。以小写字母开头的标识符是私有的，只能在当前包内访问。

在 Go 语言中，有一些命名规范适用于不同的命名情况。以下是一些常见的命名规范：

1. **包名**：包名应该使用单数形式，且应该是小写的，例如 `utils`。
2. **文件名**：文件名应该全部使用小写字母，可以包含下划线 `_`，例如 `my_file.go`。
3. **变量**：变量名使用驼峰命名法，例如 `myVariable`。私有变量的命名应该以小写字母开头，公共变量则以大写字母开头。
4. **常量**：常量的命名应该全部使用大写字母，可以包含下划线 `_`，例如 `MAX_SIZE`。
5. **函数**：函数名同样使用驼峰命名法，例如 `calculateTotal`。
6. **结构体**：结构体的命名同样使用驼峰命名法，例如 `type MyStruct struct`。
7. **接口**：接口的命名同样使用驼峰命名法，例如 `type MyInterface interface`。
8. **枚举**：枚举的命名同样使用驼峰命名法，例如 `type Color int`。

> 目前（Go 1.21），Go 有 26 个类型种类。

## 类型

### 类型定义（type definition declaration）

用如下形式来定义新的类型。`type`为一个关键字。

```go
// 定义单个类型。
type NewTypeName SourceType

// 定义多个类型（将多个类型描述合并在一个声明中）。
type (
 NewTypeName1 SourceType1
 NewTypeName2 SourceType2
)
```

- 一个新定义的类型和它的源类型为两个不同的类型。
- 在两个不同的类型定义中所定义的两个类型肯定是两个不同的类型。
- 一个新定义的类型和它的源类型的底层类型一致并且它们的值可以相互显式转换。
- 类型定义可以出现在函数体内（局部）。

### 具名类型(named type)

一个具名类型可能为

- 一个预声明类型；
- 一个定义（非自定义泛型）类型；
- 一个（泛型类型的）实例化类型；
- 一个类型参数类型（使用在自定义泛型中）。

其它类型称为*无名类型*。一个无名类型肯定是一个*组合类型*。
[Named and Unnamed Types](https://stackoverflow.com/questions/32983546/named-and-unnamed-types)

### 类型别名声明

```go
type (
 Name = string
 Age  = int
)

type table = map[string]int
type Table = map[Name]Age
```

其中 `map[string]int` 和 `map[Name]Age` 表示同一类型

### 底层类型（underlying type）

规则：

- 一个内置类型的底层类型为它自己。
- `unsafe`标准库包中定义的`Pointer`类型的底层类型是它自己。`unsafe.Pointer`也被视为一个内置类型。
- 一个无名类型（必为一个组合类型）的底层类型为它自己。
- 在一个类型声明中，新声明的类型和源类型共享底层类型。

```go
// 这四个类型的底层类型均为内置类型int。
type (
 MyInt int
 Age   MyInt
)

// 下面这三个新声明的类型的底层类型各不相同。
type (
 IntSlice   []int   // 底层类型为[]int
 MyIntSlice []MyInt // 底层类型为[]MyInt
 AgeSlice   []Age   // 底层类型为[]Age
)

// 类型[]Age、Ages和AgeSlice的底层类型均为[]Age。
type Ages AgeSlice
```

#### 在 Go 中

- 底层类型为内置类型`bool`的类型称为**布尔类型**；
- 底层类型为任一内置整数类型的类型称为**整数类型**；
- 底层类型为内置类型`float32`或者`float64`的类型称为**浮点数类型**；
- 底层类型为内置类型`complex64`或`complex128`的类型称为**复数类型**；
- 整数类型、浮点数类型和复数类型统称为**数字值类型**；
- 底层类型为内置类型`string`的类型称为**字符串类型**。

## 值（value）

一个类型的一个**实例**称为此类型的一个值。一个类型可以有很多不同的值，其中一个为它的零值。同一类型的不同值共享很多相同的属性。

每个类型有一个零值。一个类型的零值可以看作是此类型的默认值。预声明的标识符 `nil` 可以看作是切片、映射、函数、通道、指针（包括非类型安全指针）和接口类型的零值的字面量表示。

值分为类型确定的和类型不确定的。

## 其他概念

### 指针类型的基类型（base type）

若一个指针类型的底层类型表示为 `*T`，则此指针类型的基类型为 `T` 所表示的类型。

### 结构体类型的字段（field）

一个结构体类型由若干成员变量组成。每个这样的成员变量称为此结构体的一个字段。

### 函数类型的签名（signature）

一个函数和其类型的签名由此函数的**输入参数**和**返回结果**的类型列表组成。函数名称和函数体不属于函数签名的构成部分。
如：

```go
 Write(p []byte) (n int, err error)
```

### 类型的方法（method）和方法集（method set）

在 Go 中，我们可以给满足某些条件的类型（构造器）声明方法。方法也常被称为成员函数。一个类型的所有方法组成了此类型的**方法集**。

### 接口类型的动态类型和动态值

接口类型的值称为**接口值**。一个接口值可以包裹装载一个非接口值。*包裹在一个接口值中的非接口值称为此接口值的动态值*。此动态值的类型称为此接口值的动态类型。 一个什么也没包裹的接口值为一个零值接口值。零值接口值的动态值和动态类型均为不存在。
一个接口类型可以指定若干个（可以是零个）方法，这些方法形成了此接口类型的方法集。
如果一个类型（可以是接口或者非接口类型）的方法集是一个接口类型的方法集的超集，则我们说此类型实现了此接口类型。

### 一个值的具体类型（concrete type）和具体值（concrete value）

对于一个（类型确定的）非接口值，它的具体类型就是它的类型，它的具体值就是它自己。
一个 *零值接口值* 没有具体类型和具体值。对于一个非零值接口值，它的具体类型和具体值就是它的动态类型和动态值。

### 容器类型

**数组** `[]int`、**切片** `slice` 和**映射** `map` 是 Go 中的三种正式意义上的内置容器类型。
有时候，**字符串**和**通道类型**也可以被非正式地看作是容器类型。
（正式和非正式的）容器类型的每个值都有一个**长度属性**。

### 映射类型的键值（key）类型

如果一个映射类型的底层类型表示为 `map[Tkey]T`，则此映射类型的键值类型为 `Tkey`。 `Tkey` 必须为一个**可比较类型** 。

### 容器类型的元素（element）类型

存储在一个容器值中的所有元素的类型必须为同一个类型。此同一类型称为此容器值的（容器）类型的元素类型。

### 通道类型的方向

一个通道值可以被看作是 **先入先出（first-in-first-out，FIFO）** 队列。一个通道值可能是可读可写的、只读的（receive-only）或者只写的（send-only）。

- 一个可读可写的通道值也称为一个双向通道。 一个双向通道类型的底层类型可以被表示为`chan T`。
- 我们只能向一个只写的通道值发送数据，而不能从其中接收数据。 只写通道类型的底层类型可以被表示为`chan<- T`。
- 我们只能从一个只读的通道值接收数据，而不能向其发送数据。 只读通道类型的底层类型可以被表示为`<-chan T`。

### 可比较类型和不可比较类型

目前（~~Go 1.21~~），下面这些类型的值不支持（使用 `==` 和 `!=` 运算标识符）比较。这些类型称为 *不可比较类型*。

- 切片类型
- 映射类型
- 函数类型
- 任何包含有不可比较类型的字段的结构体类型和任何元素类型为不可比较类型的数组类型。
  其它类型称为可比较类型。
  映射类型的键值类型必须为可比较类型。

## 外部链接

[Go 101](https://gfw.go101.org/article/type-system-overview.html)