---
title: "Go Web的学习2"
date: 2024-06-11T22:45:30+08:00
draft: false
taps: ['web', 'router']
categories: ['Golang', 'Web']
author: ["Yeelight"]
showtoc: true
weight:
math: false
readingTime: true
---

## Router 框架
>
> 大门管理员
>
### 路由功能

为了将代码模块化，而随之带来的另外一个问题是关于前端页面的跳转问题，由于代码的隔离，代码之间有时候会无法互相访问。因此需要一个**大门管理员**， Router 就此实现。

它负责将传入的 HTTP 请求**映射（map）** 到相应的处理函数上。简单来说，路由决定了当用户访问一个特定的 URL 时，服务器应该执行哪些代码来处理这个请求。

类似下面这样：

```go
type router struct {
    // map Handler function
    handlers map[string]HandlerFunc
}
```

因此，路由要满足一下功能：

1. **URL匹配**：路由框架能够识别和匹配传入请求的URL，根据URL的不同部分（如路径、查询参数等）来决定调用哪个处理函数。

```go
func (r *router) handle(ctx *Context) {
    key := ctx.Method + "-" + ctx.Path
    if handler, ok := r.handlers[key]; ok {
       handler(ctx.Res, ctx.Req)
    } else {
       ctx.String(http.StatusNotFound, "404 NOT FOUND: %s\n", ctx.Path)
    }
}
```

2. **处理函数映射**：每个 URL 模式（pattern）都会被**映射**到一个或多个处理函数。当请求的 URL 匹配到某个模式时，相应的处理函数就会被调用。

```go
func (router *router) addRoute(method string, pattern string, handler HandlerFunc) {
    log.Printf("Route %4s - %s", method, pattern)
    key := method + "-" + pattern
    router.handlers[key] = handler
}
```

3. **参数提取**：对于**动态路由**，如 `/user/:id`，路由框架能够从 URL 中提取参数（如 `:id`），并将这些参数传递给处理函数。
4. **中间件支持**：路由框架通常支持中间件，这些中间件可以在请求到达**处理函数之前或之后**执行一些操作，如身份验证、日志记录等。

### 主要点

#### Query

在HTTP请求的上下文中，`query` 通常指的是URL中的查询参数。查询参数是URL中`?`后面的键值对，它们用于向服务器传递额外的信息。例如，在URL `http://example.com/search?q=golang` 中，`q=golang` 就是一个查询参数，其中`q`是参数名，`golang`是参数值。

#### Handler

`Handler` 或 `Request Handler` 是指处理HTTP请求的函数或方法。在大多数后端框架中，包括Go语言的许多Web框架（如Gin、Echo、net/http等），`handler` 是一个函数，它接收HTTP请求并返回HTTP**响应**。

### 封装

在 Web 服务中，封装可以用来简化 HTTP**请求**和**响应**的处理。对于原始的方式需要手动设置 *HTTP Header*、*状态码*和*消息体* 等等，而封装后的方法则提供了一个更简洁的接口，只需调用一个函数就可以完成所有这些设置。

#### Context 结构

在Web框架中，Context通常是一个结构体，它包含了处理HTTP请求和响应所需的所有信息。

- **统一接口**：通过Context，可以将所有与请求相关的信息集中管理，使得处理函数和中间件可以方便地访问这些信息。
- **动态路由支持**：Context 可以存储动态路由解析后的参数，使得这些参数可以在处理函数中直接使用。[[#动态路由]]
- **中间件支持**：中间件可以在Context中添加或修改信息，这些信息可以被后续的处理函数使用。

Context 随着每一个请求的出现而产生，请求的结束而销毁，和当前请求强相关的信息都应由 Context 承载。

```go
// Context 结构体用于封装HTTP请求和响应的相关信息
type Context struct {
    Res http.ResponseWriter
    Req *http.Request

    Path       string
    Method     string
    StatusCode int
}
```

在 Go 中的Context

#### API 设计

🚧🚧🚧🚧🚧🚧🚧🚧🚧

### 动态路由

动态路由是指路由中的某些部分是可变的，例如 `/hello/:name` 中的 `:name` 部分。当请求匹配到这个路由时，`:name` 会被替换为实际的值。在 Context 结构中，这些动态参数通常会被存储起来，以便在处理函数中使用。

为了保存这些 `pattern` 我们有许多的协议和数据结构，常用 `Trie-Tree` 来作 pattern 的底层结构。

了解 `pattern` 参数，其中 pattern 就是在路由器中预先定义的一组规则**模式** ，用来和用户输入的 URL 匹配。

一般有三种模糊匹配路由规则：

1. `:name` ---- **命名匹配规则**
2. `*any`   ---- **模糊匹配规**
3. `{field}` ---- **字段匹配规则**

```go
type node struct {
 //待匹配路由
 pattern string
 // important fields
 part     string
 children []*node
 // node color (isKey)
 // *any | :xxx
 isWildcard bool
}

type PatRoot struct {
    root *node
}
```

#### 要点

1. **Pattern（模式）**:
    - 模式是指在路由器中预先定义的一组规则，用于匹配传入的HTTP请求的URL路径。这些模式通常包含静态路径部分和动态路径部分（如参数或通配符）。例如，模式 `/user/:id` 表示一个用户详情页，其中 `:id` 是一个参数，可以匹配任何用户ID。
    - 在您的代码中，`pattern` 是 `router` 结构体中的一个字段，用于存储每个HTTP方法对应的路由树（`PatRoot`）。这些路由树是根据预定义的模式构建的，用于快速匹配传入的请求路径。

2. **URL（统一资源定位符）**:
    - URL 是客户端（如浏览器）发送给服务器的实际请求路径。它是一个具体的字符串，指定了请求的资源在服务器上的位置。例如，`/user/123` 是一个URL，它请求ID为123的用户详情页。
    - 在您的代码中，`URL` 是通过 `Context` 对象传递的，它是处理HTTP请求时的一个关键部分。`getRoute` 方法使用这个URL来在路由树中查找匹配的模式

第一种是对 httpRouter 进行简单的封装，然后提供定制的中间件和一些简单的小工具集成比如 gin，主打轻量，易学，高性能。第二种是借鉴其它语言的编程风格的一些 MVC 类框架，例如 beego，方便从其它语言迁移过来的程序员快速上手，快速开发。还有一些框架功能更为强大，除了数据库 schema 设计，大部分代码直接生成，例如 goa。不管哪种框架，适合开发者背景的就是最好的。

根据我们的经验，简单地来说，只要你的路由带有参数，并且这个项目的 API 数目超过了 10，就尽量不要使用 `net/http` 中默认的路由。

## 外部链接

[router 请求路由](https://chai2010.cn/advanced-go-programming-book/ch5-web/ch5-02-router.html#52-router-%E8%AF%B7%E6%B1%82%E8%B7%AF%E7%94%B1)
