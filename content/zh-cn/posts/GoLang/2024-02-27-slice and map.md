---
slug: 160
# layout: post
title: 'GoLang 切片'
author: yexca
date: 2024-02-27T20:00:00+08:00
lastmod: 2025-01-28T15:30:18+09:00
# permalink: /archives/160
categories:
    - 技术学习
tags:
    - Go
    - 编程语言
    - 编程基础
--- 

> **Golang 系列**
>
> Hello GoLang: <https://blog.yexca.net/archives/154>  
> GoLang (var and const) 变量与常量: <https://blog.yexca.net/archives/155>  
> GoLang (func) 函数: <https://blog.yexca.net/archives/156>  
> GoLang (slice and map) 切片: 本文  
> GoLang (OOP) 面向对象: <https://blog.yexca.net/archives/162>  
> GoLang (reflect) 反射: <https://blog.yexca.net/archives/204>  
> GoLang (struct tag) 结构体标签: <https://blog.yexca.net/archives/205>  
> GoLang (goroutine) go 程: <https://blog.yexca.net/archives/206>  
> GoLang (channel) 通道: <https://blog.yexca.net/archives/207>  

---

Go 的切片是对数组的抽象

## 数组

数组的长度不可改变

```go
package main

import "fmt"

func main() {
    // 定义方式一
    var arr1 [10]int
    // 遍历
    for i := 0; i < len(arr1); i++ {
        fmt.Println("arr1[", i, "]:", arr1[i])
    }

    // 定义方式二，赋值
    arr2 := [10]int{0, 1, 2, 3}
    // range遍历
    for index, value := range arr2 {
        fmt.Println("index =", index, "value =", value)
    }

    // 定义不同长度
    var arr3 [4]int

    fmt.Printf("type of arr1 is %T\n", arr1) // [10]int
    fmt.Printf("type of arr1 is %T\n", arr2) // [10]int
    fmt.Printf("type of arr1 is %T", arr3)   // [4]int
}
```

编译运行后可以发现 arr3 与 arr1，arr2 类型不同，那么在定义函数形参时也需要指定相应类型

```go
func test(arr [4]int) {
    for i := 0; i < len(arr); i++ {
        fmt.Println("fmt_arr[", i, "]:", arr[i])
    }
}
```

上述函数只能传递 arr3，值传递，修改值不影响原数据

## 定义切片

与数组相比切片长度不固定，可追加元素 (动态数组)，在追加时可能使切片的容量增大

定义切片可以通过声明一个未指定大小的数组

```go
var name []type
// 例如
var s []int
```

或者使用 make() 函数来创建切片

```go
var slice []type = make([]type, len) // len 为切片初始长度
// 也可以简写为
slice := make([]type, len)
```

可以使用可选参数 capacity 指定容量，省略与 length 相同

```go
var slice []type = make([]type, length, capacity)
```

![image](https://cdn.jsdelivr.net/gh/yexca/picx-images-hosting@master/2024/02-go/image.13li74f5h3.webp)

## 切片初始化

直接初始化

```go
s := []int {1, 2, 3}
```

将数组值为切片初始化，从 startIndex 到 endIndex-1，这俩值都可省略

```go
s := arr[startIndex:endIndex]
```

省略 startIndex 或 endIndex 表示从第一个元素开始索引或索引到最后一个元素

## len() 和 cap()

切片是可以索引的，通过 len() 函数获取长度

而 cap() 为计算容量的方法，可以测量切片最长可以达到多少

```go
package main

import "fmt"

// 切片传递为引用传递，函数内修改影响原数据
func printSlice(slice []int) {
    fmt.Printf("len=%d, cap=%d, slice=%v", len(slice), cap(slice), slice)
}

func main() {
    s := make([]int, 3, 5)
    printSlice(s)
}

/*
 * 输出
 * len=3, cap=5, slice=[0 0 0]
 */
```

## 空切片

一个切片在未初始化之前默认为 nil (空切片)，长度为 0

```go
package main

import "fmt"

func printSlice(slice []int) {
    fmt.Printf("len=%d, cap=%d, slice=%v\n", len(slice), cap(slice), slice)
}

func main() {
    var s []int
    printSlice(s)
    // 判断是否为空
    if s == nil {
        fmt.Println("slice is empty")
    }
}
```

## 切片截取

通过设置上下限截取切片

```go
package main

import "fmt"

func printSlice(slice []int) {
    fmt.Printf("len=%d, cap=%d, slice=%v\n", len(slice), cap(slice), slice)
}

func main() {
    s := []int{0, 1, 2, 3, 4, 5, 6, 7}

    // 打印原始切片
    fmt.Println(s)
    
    // 从2(包含)到5(不包含)
    printSlice(s[2:5])
    // 从第一个到5(不包含)
    printSlice(s[:5])
    // 从第二个到最后一个
    printSlice(s[2:])

    // 这样赋值修改 subS 将影响到 s
    subS := s[1:6]
    printSlice(subS)
}
```

## append() 和 copy()

增加切片的容量与拷贝切片

```go
package main

import "fmt"

func printSlice(slice []int) {
    fmt.Printf("len=%d, cap=%d, slice=%v\n", len(slice), cap(slice), slice)
}

func main() {
    var s []int
    printSlice(s)

    // 增加一个元素
    s = append(s, 0)
    printSlice(s)

    // 增加多个元素
    s = append(s, 1, 2, 3, 4)
    printSlice(s)

    // 创建一个容量为 s 两倍的 s2
    s2 := make([]int, len(s), cap(s)*2)
    // 拷贝 s 到 s2，此时修改 s2 不影响 s
    copy(s2, s)
    printSlice(s2)
}
```

> 切片的扩容：如果追加值超过容量，则将容量增加两倍

## map

map 有两种声明方式

```go
package main

import "fmt"

func main() {
    var map1 = make(map[string]string)
    // 插入数据
    map1["one"] = "1"
    map1["two"] = "2"
    fmt.Println(map1) // map[one:1 two:2]
}
```

第二种

```go
package main

import "fmt"

func main() {
    map1 := map[string]string{
        "one": "1",
        "two": "2",
    }
    fmt.Println(map1)
}
```

### map 嵌套

```go
package main

import "fmt"

func main() {
    map1 := make(map[string]map[string]string)
    map1["first"] = make(map[string]string, 2)
    map1["first"]["one"] = "1"
    map1["first"]["two"] = "2"
    fmt.Println(map1)
}

/*
 * 输出
 * map[first:map[one:1 two:2]]
 */
```

### 修改遍历与删除

```go
package main

import "fmt"

func main() {
    map1 := make(map[string]map[string]string)
    map1["first"] = make(map[string]string, 2)
    map1["first"]["one"] = "1"
    map1["first"]["two"] = "2"
    // 修改
    map1["first"]["one"] = "one"
    fmt.Println(map1)
    // 遍历
    for key, value := range map1{
        fmt.println("key =", key, "value =", value)
    }
    // 删除
    delete(map1, "first")
    fmt.Println(map1)
}
```

### 判断是否有某值

```go
package main

import "fmt"

func main() {
    map1 := make(map[string]string)
    map1["one"] = "1"
    val, key := map1["one"]
    if key {
        fmt.Println(val)
    } else {
        fmt.Println("empty")
    }
}
```
