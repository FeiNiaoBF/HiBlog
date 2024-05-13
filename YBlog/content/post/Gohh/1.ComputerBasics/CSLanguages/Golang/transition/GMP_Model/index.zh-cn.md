---
title: "GMP_Model"
date: 2024-05-13T09:14:13+08:00
draft: true
taps: ["Golang", "GMP"]
categories: ["Golang", "CS"]
author: ["Yeelight"]
showtoc: true
weight:
math: false
readingTime: true
---

## Goroutine 协程

## Processor 逻辑处理器

## Thread 线程

## 设计策略

## 生命周期

这段 Go 代码定义了一个函数 `stealWork`，用于尝试从其他的处理器（P）中窃取一个可运行的 goroutine 或定时器。这个函数是 Go 的调度器的一部分，用于在多个处理器之间平衡工作负载。以下是代码的详细解析：

### 函数签名

`func stealWork(now int64) (gp *g, inheritTime bool, rnow, pollUntil int64, newWork bool)`

- **参数**:
  - `now`: 当前的时间，用于处理定时器。如果为 0，则函数会自行获取当前时间。

- **返回值**:
  - `gp`: 返回被窃取的 goroutine 的指针。
  - `inheritTime`: 布尔值，表示返回的 goroutine 是否应继承调度时间。
  - `rnow`: 返回的当前时间或传入的时间。
  - `pollUntil`: 表示下一个定时器事件的时间。
  - `newWork`: 布尔值，表示是否有新的工作被准备好。

### 函数体解析

1. **获取当前 P 的指针**:

   ```go
   pp := getg().m.p.ptr()
   ```

   这行代码获取当前 Goroutine 的 M（操作系统线程）关联的 P（处理器）的指针。

2. **初始化变量**:

   ```go
   ranTimer := false
   ```

   初始化一个标志，用于记录是否运行了定时器。

3. **尝试窃取工作的循环**:

   ```go
   const stealTries = 4
   for i := 0; i < stealTries; i++ {
       ...
   }
   ```

   函数尝试四次（由 `stealTries` 常量定义）来窃取工作。

4. **遍历所有 P 以窃取工作**:

   ```go
   for enum := stealOrder.start(cheaprand()); !enum.done(); enum.next() {
       ...
   }
   ```

   使用一个随机的开始点，遍历所有的 P。`stealOrder` 是一个遍历序列，可能基于当前状态进行优化选择。

5. **检查垃圾回收状态**:

   ```go
   if sched.gcwaiting.Load() {
       return nil, false, now, pollUntil, true
   }
   ```

   如果正在等待垃圾回收，直接返回，表示可能有 GC 相关的工作要做。

6. **检查和窃取定时器**:

   ```go
   if stealTimersOrRunNextG && timerpMask.read(enum.position()) {
       ...
   }
   ```

   在最后一次尝试中，尝试从其他 P 中窃取定时器。如果运行了定时器，可能会使一些 goroutine 变为可运行状态，这时会尝试从本地运行队列中获取一个 goroutine。

7. **尝试窃取运行队列中的 goroutine**:

   ```go
   if gp := runqsteal(pp, p2, stealTimersOrRunNextG); gp != nil {
       return gp, false, now, pollUntil, ranTimer
   }
   ```

   如果未运行定时器或定时器未使 goroutine 就绪，尝试从其他 P 的运行队列中窃取 goroutine。

8. **返回结果**:
   如果所有尝试都失败了，返回 nil 和相关的时间信息。
