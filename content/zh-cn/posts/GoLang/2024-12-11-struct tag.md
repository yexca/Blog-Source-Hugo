---
slug: 205
# layout: post
title: "GoLang 结构体标签"
author: yexca
date: 2024-12-11T18:31:18+08:00
lastmod: 2025-01-28T21:04:18+09:00
# permalink: /archives/205
categories:
    - 技术学习
tags:
    - Go
    - 编程语言
    - 编程基础
--- 

> **Golang Series**
>
> Hello GoLang: <https://blog.yexca.net/archives/154>  
> GoLang (var and const) 变量与常量: <https://blog.yexca.net/archives/155>  
> GoLang (func) 函数: <https://blog.yexca.net/archives/156>  
> GoLang (slice and map) 切片: <https://blog.yexca.net/archives/160>  
> GoLang (OOP) 面向对象: <https://blog.yexca.net/archives/162>  
> GoLang (reflect) 反射: <https://blog.yexca.net/archives/204>  
> GoLang (struct tag) 结构体标签: 本文  
> GoLang (goroutine) go 程: <https://blog.yexca.net/archives/206>  
> GoLang (channel) 通道: <https://blog.yexca.net/archives/207>  

---

通过结构体标签可以描述该类在某包的作用

## 获取标签值

通过 ` 来定义 tag (markdown 代码块的键)

```go
package main

import (
    "fmt"
    "reflect"
)

type User struct {
    // 多个tag用空格隔离
    name string `doc:"name" info:"nameOfUser"`
    age  int    `info:"ageOfUser"`
}

func main() {
    user := User{"zhangSan", 18}

    findTag(&user)
}

func findTag(input interface{}) {
    t := reflect.TypeOf(input).Elem()

    for i := 0; i < t.NumField(); i++ {
        tagInfo := t.Field(i).Tag.Get("info")
        tagDoc := t.Field(i).Tag.Get("doc")
        fmt.Println("info:", tagInfo, "doc:", tagDoc)
    }
}
```

## JSON 转换

```go
package main

import (
    "encoding/json"
    "fmt"
)

type User struct {
    // 注意要是公有属性才可转换json
    Name string `json:"name"`
    Age  int    `json:"age"`
}

func main() {
    user := User{"zhangSan", 18}

    // struct --> json
    jsonStr, err := json.Marshal(user)
    if err != nil {
        fmt.Println("error", err)
    } else {
        fmt.Printf("jsonStr : %s\n", jsonStr)
    }

    // json --> struct
    var user2 User
    err = json.Unmarshal(jsonStr, &user2)
    if err != nil {
        fmt.Println("error", err)
    } else {
        fmt.Println(user2)
    }
}
```
