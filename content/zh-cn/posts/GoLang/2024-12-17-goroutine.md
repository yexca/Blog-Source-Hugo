---
slug: 206
# layout: post
title: "GoLang go 程"
author: yexca
date: 2024-12-17T21:16:31+08:00
lastmod: 2025-01-28T21:11:18+09:00
# permalink: /archives/206
categories:
    - 编程基础
    - 编程语言
tags:
    - Go
--- 

> **Golang Series**
>
> Hello GoLang: <https://blog.yexca.net/archives/154>  
> GoLang (var and const) 变量与常量: <https://blog.yexca.net/archives/155>  
> GoLang (func) 函数: <https://blog.yexca.net/archives/156>  
> GoLang (slice and map) 切片: <https://blog.yexca.net/archives/160>  
> GoLang (OOP) 面向对象: <https://blog.yexca.net/archives/162>  
> GoLang (reflect) 反射: <https://blog.yexca.net/archives/204>  
> GoLang (struct tag) 结构体标签: <https://blog.yexca.net/archives/205>  
> GoLang (goroutine) go 程: 本文  
> GoLang (channel) 通道: <https://blog.yexca.net/archives/207>  

---

进程 -> 线程 -> 协程

协程 (coroutine) 也叫轻量级线程，可以轻松创建上万个而不会导致系统资源衰竭，多个协程共享该线程分配到的计算机资源

Go 语言原生支持协程，叫 goroutine，Go 的并发通过 goroutine 和 channel 实现

## 创建 goroutine

通过 `go` 关键字开启一个 goroutine

```go
package main

import (
    "fmt"
    "time"
)

func newTask() {
    i := 0
    for {
        fmt.Println("newTask goroutine i =", i)
        i++
        time.Sleep(1 * time.Second)
    }
}

func main() {
    go newTask()

    i := 0
    for {
        fmt.Println("main goroutine i =", i)
        i++
        time.Sleep(1 * time.Second)
    }
}
```

## 使用匿名函数

当然也可以使用匿名函数

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    go func() {
        defer fmt.Println("A.defer")
        func() {
            defer fmt.Println("B.defer")

            fmt.Println("B")
        }() // 表示执行该匿名函数

        fmt.Println("A")
    }()

    time.Sleep(2 * time.Second)
}
```

匿名函数也可以有形参与返回值，不过 goroutine 返回值需要通过 channel 传输，下例仅演示有形参

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    go func(a, b int) {
        fmt.Println("a =", a, "b =", b)
    }(10, 20)

    time.Sleep(2 * time.Second)
}
```

## 退出

主 goroutine 退出后，其他的工作 goroutine 也会自动退出

不过也可使用 `runtime.Goexit()` 立即终止当前 goroutine 执行 (defer 会执行)

```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func main() {
    go func() {
        defer fmt.Println("A.defer")
        func() {
            defer fmt.Println("B.defer")
            runtime.Goexit() // 退出 goroutine
            fmt.Println("B")
        }() // 表示执行该匿名函数

        fmt.Println("A")
    }()

    time.Sleep(2 * time.Second)
}

/*
 * 输出
 * B.defer
 * A.defer
 */
```
