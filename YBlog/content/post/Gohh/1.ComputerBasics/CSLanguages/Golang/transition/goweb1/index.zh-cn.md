---
title: "Go Web的学习1"
date: 2024-06-11T22:45:30+08:00
draft: false
taps: ['web']
categories: ['Golang', 'Web']
author: ["Yeelight"]
showtoc: true
weight:
math: false
readingTime: true
---

> 如何通过 Go 来编写 Web
>
## Web

### HTTP 是什么

<!-- TODO LINK -->
-> [http 是什么]({{< ref "post/Gohh/5.Networking/http/index.zh-cn.md">}})

Web 是什么，有些人认为是一个网页，有些人认为是 World Wide Web（万维网）系统，有些人认为是网路时代的代指。都没错，但为了不在下面的文字中模糊这个词，孤自认为 Web 是一个管理资源的**服务端**，而 Go 所做的就是搭建这个服务端。

一个 Web 服务器，它监听来自客服端的请求，并通过 **HTTP 协议**将自己与客户端通信。这个客户端通常指的是 *Web Browser*。

### Web 服务器

Web 服务器的工作流程为：

1. 监听来自客户端通过 **TCP/IP 协议**建立到服务器的 TCP 连接请求
2. **处理并解析**请求的内容，确定客户端请求的是哪个资源
3. 服务端根据请求处理的结果生成 **HTTP 响应**
4. 服务端将响应和（静态/动态）资源通过互联网**送回**客户端
5. **持久连接/非持久连接**

### RESTful 架构

表现层状态转换（**Representational State Transfer**）是根基于**超文本传输协议（HTTP）** 之上而确定的一组约束和属性，是一种设计提供万维网络服务的**软件构建风格**。符合或兼容于这种架构风格（简称为 REST 或 RESTful）的网络服务，允许客户端发出以**统一资源标识符**访问和操作网络资源的请求，而与预先定义好的无状态操作集一致化。
![URL&URI&URN](https://s2.loli.net/2024/05/26/IO7laRYz1WBTG94.png)

- 每一个 **URI** 代表一种资源
- 客户端和服务器之间，传递这种资源的某种表现层
- 客户端通过四个 HTTP 方法，对服务器端资源进行操作，实现"表现层状态转化"

## net/http package

`net/http` 是 Go 官方的标准包

🚧🚧🚧

我对 net/http 的原理解析-> [[http package]]

```go
func SayHello(w http.ResponseWriter, r *http.Request) {
    _, err := fmt.Fprintln(w, "Hello Go Web")
    if err != nil {
       return
    }
}

func main() {
    http.HandleFunc("/hello", SayHello)
    http.ListenAndServe(":9999", nil)
}
```

Go 中的 http 很体贴的将一些过程抽象了，其中就只有 `ListenAndServe` 来监听客户端的请求，`HandleFunc`  来注册一个处理器和对应的 pattem 建立 route ，`SayHello` 是具体的 Handler 函数。

Web 工作方式的几个概念

- Request：用户请求的信息，用来解析用户的请求信息，包括 post、get、cookie、url 等信息
- Response：服务器需要反馈给客户端的信息
- Conn：用户的每次请求链接
- Handler：处理请求和生成返回信息的处理逻辑

## 外部链接

[理解RESTful架构](https://www.ruanyifeng.com/blog/2011/09/restful.html)
