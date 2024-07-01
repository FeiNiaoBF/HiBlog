---
title: "MIT6.824 Distributed System(1)---Lab01"
date: 2024-07-01T17:16:23+08:00
draft: false
taps: ['分布式','Lab','6.824']
categories: ['分布式', '课程']
author: ["Yeelight"]
showtoc: true
weight:
math: false
readingTime: true
---

## MapReduce 基本工作原理

> 来自 MapReduce 的论文

1. The MapReduce library in the user program first  splits the input files into M pieces of typically 16  megabytes to 64 megabytes (MB) per piece (controllable by the user via an optional parameter). It  then starts up many copies of the program on a cluster of machines.

    用户程序中的 MapReduce 库首先将输入文件分割成 M 个片段，每个片段通常为 16 到 64 兆字节（MB）（用户可以通过可选参数控制片段大小）。然后，它在一组机器集群上启动该程序的多个副本。


2. One of the copies of the program is special – the  master. The rest are workers that are assigned work  by the master. There are M map tasks and R reduce  tasks to assign. The master picks idle workers and  assigns each one a map task or a reduce task.

    这些程序副本中有一个特殊的是主节点，其余的是工作节点，由主节点分配任务。有 M 个 map 任务和 R 个 reduce 任务需要分配。主节点选择空闲的工作节点，并为其分配一个 map 任务或一个 reduce 任务

3. A worker who is assigned a map task reads the  contents of the corresponding input split. It parses  key/value pairs out of the input data and passes each  pair to the user-defined Map function. The intermediate key/value pairs produced by the Map function  are buffered in memory.

    被分配了 map 任务的工作节点读取相应输入片段的内容。它从输入数据中解析出键/值对，并将每一对传递给用户定义的 Map 函数。由 Map 函数生成的中间键/值对被缓存在内存中。

4. Periodically, the buffered pairs are written to local  disk, partitioned into R regions by the partitioning  function. The locations of these buffered pairs on  the local disk are passed back to the master, who  is responsible for forwarding these locations to the  reduce workers.

    定期地，缓存的键/值对被写入本地磁盘，并通过分区函数分成 R 个区域。这些缓存在本地磁盘上的键/值对的位置被传递回主节点，主节点负责将这些位置转发给 reduce 工作节点。

5. When a reduce worker is notified by the master  about these locations, it uses remote procedure calls  to read the buffered data from the local disks of the  map workers. When a reduce worker has read all intermediate data, it sorts it by the intermediate keys  so that all occurrences of the same key are grouped  together. The sorting is needed because typically  many different keys map to the same reduce task. If  the amount of intermediate data is too large to fit in  memory, an external sort is used.

    当 reduce 工作节点被主节点通知这些位置时，它通过远程过程调用从 map 工作节点的本地磁盘读取缓存数据。当 reduce 工作节点读取了所有中间数据后，它按中间键对数据进行排序，以便将相同键的所有出现组合在一起。排序是必要的，因为通常许多不同的键会映射到同一个 reduce 任务。如果中间数据量太大，无法放入内存，则使用外部排序

6. The reduce worker iterates over the sorted intermediate data and for each unique intermediate key encountered, it passes the key and the corresponding  set of intermediate values to the user’s Reduce function. The output of the Reduce function is appended  to a final output file for this reduce partition.

    reduce 工作节点遍历排序后的中间数据，对于遇到的每个唯一的中间键，它将键和相应的中间值集合传递给用户的 Reduce 函数。Reduce 函数的输出被追加到该 reduce 分区的最终输出文件中。

7. When all map tasks and reduce tasks have been  completed, the master wakes up the user program.  At this point, the MapReduce call in the user program returns back to the user code.

    当所有 map 任务和 reduce 任务完成后，主节点唤醒用户程序。此时，用户程序中的 MapReduce 调用返回到用户代码。

## Lab 01

### 简介

![MapReduce]( https://imgs.search.brave.com/kRnqIwaErmRijpPv2RKP_0fFRkVYZkSCNJ0WCW5355A/rs:fit:500:0:0:0/g:ce/aHR0cHM6Ly9pLnNz/dGF0aWMubmV0L0VI/OHRZLmpwZw )

这就是一个典型的 MapReduce Job。从整体来看，为了保证完整性，有一些术语要介绍一下：

- Job。整个 MapReduce 计算称为 Job。
- Task。每一次 MapReduce 调用称为 Task。

### 任务描述

实现分布式 MapReduce 中 `Coordinator`（Master）和可多次使用的 `Worker`, 在这次实验都在一个机器上运行。worker通过 RPC 和 coordinator 交互。worker请求任务,进行运算（`mapf` 和 `reducef`）, 写出结果并重命名到 `mr-X-Y` 文件。

### 实施

#### 抽象 Mater

在实验中提到，首先需要修改工作结构中的工作，然后通过 RPC 向 Coordinator 请求任务，即请求一个新的 map task。但在实际操作中，我发现不能直接开始编写这部分代码，必须先定义完 Coordinator 的抽象数据结构。只有明确了协调器管理的数据结构，才能更好地进行后续的开发工作。因此，首先需要抽象出协调器的数据结构，然后再编写 Worker 部分的代码。

```go
type Coordinator struct {
mu              sync.Mutex // lock
files           []string   // all input file
nReduce         int        // number of reduce tasks
nMap            int        // number of map tasks
mapTasks        []Task     // indxe(id): 0~(nmap-1)
reduceTasks     []Task
nmapFinished    int
nreduceFinished int
}
```


之后来写 `Worker` 函数，结合在实验的例子中，我们可以参考 `RPC` 的例子。这个例子中有两个参数：第一个参数是传递给 coordinator 的参数，第二个参数是一个空参数，由 coordinator 返回。通过 `RPC`，我们可以在不同的系统之间进行协调，实现分布式系统。

```go
// example function to show how to make an RPC call to the coordinator.
//
// the RPC argument and reply types are defined in rpc.go.
func CallExample() {
// declare an argument structure.
args := ExampleArgs{}

// fill in the argument(s).
args.X = 10

// declare a reply structure.
reply := ExampleReply{}

// send the RPC request, wait for the reply.
// the "Coordinator.Example" tells the
// receiving server that we'd like to call
// the Example() method of struct Coordinator.
ok := call("Coordinator.Example", &args, &reply)
if ok {
   // reply.Y should be 100.
   fmt.Printf("reply.Y %v\n", reply.Y)
} else {
   fmt.Printf("call failed!\n")
   }
}
```

#### 抽象 Wokers（Task）

按照例子，我可以模仿“照猫画虎”的方式来编写关于工作的具体实现。具体来说，我需要明确一个任务（task）的概念。看完 *MapRudce* 后，我发现 `Worker` 代码必须根据不同的 task 类型进行选择操作。其中发现 **workers** 有三种不同的形态（实际上应该是四种，其中一种是特殊的管理者---**Mater**）：map、reduce和idle（空闲）。因此，我们需要为每种工作类型抽象出一个数据结构。为了方便管理和协调，我们将这些工作抽象成一个 task，每个 `task` 代表一个独立的工作，它包含了该工作的状态信息。

```go
// Task is a struct that represents a worker task

type Task struct {
    Id          int        // worker ID
    Type        workerType // worker state
    Status      statusType // 0 "Unassigned", 1 "Assigned", 2 "Finished"
    Timestamp   time.Time  // Start time
    MapFile     string     // File for map task
    ReduceFiles []string   // List of files for reduce task
}
// MapReduceArgs is the message worker sends to coordinator
// that have the worker's task and message type
type MapReduceArgs struct {
    Task        Task
    MessageType messageType
}

// MapReduceReply is the message coordinator sends to worker
// when coordinator receives the worker's request.
// that have the worker's task and number of reduce tasks of can be assigned
type MapReduceReply struct {
    Task    Task
    NReduce int
    NMap    int
}
// workerType is an enum for the type of worker
type workerType int
// Idle: 0, Map: 1, Reduce: 2
const (
    Idle workerType = iota
    Map
    Reduce
)

// messageType: 0 "RequestTask", 1 "FinishTask"
type messageType int
const (
    RequestTask messageType = iota
    FinishTask
)

type statusType int
const (
    Unassigned statusType = iota
    Assigned
    Finished
)
```

#### Coordinator 分配

当然写完 `worker` 后，那么就要去写 `coordinator`，他是对每个请求的 `Woker` 进行分配的管理者，主要工作就是分配，那么我去根据我的类型。然后进行一次轮巡，当我为空的时候，我就把这个新的一个工作给予给工作，新的工作，它的类型一开始永远都是map，到后面它会变成readers。当我当控制，我整个控制流，当我整个分配结束后，那说明我这个系统也完成了。

![test map task](https://s2.loli.net/2024/07/01/WjyKLIMPOtJVRUv.png)

有一点小坑的就是在当我在写分配的时候，需要注意对每个任务（`task`）的状态进行区分。任务状态的改变或不变会导致不同的系统行为，因此需要非常清晰地理解和处理这些状态变化。


### 实验结束

![done lab](https://s2.loli.net/2024/07/01/IEe7s4hyFQRuqxw.png)

## 外部链接

[6.5840 Lab 1: MapReduce](https://pdos.csail.mit.edu/6.824/labs/lab-mr.html)
