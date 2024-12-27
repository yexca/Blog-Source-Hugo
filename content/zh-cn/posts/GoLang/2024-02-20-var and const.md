---
slug: 155
# layout: post
title: 'GoLang 变量与常量'
author: yexca
date: 2024-02-20T06:41:18+08:00
# permalink: /archives/155
categories:
    - 编程基础
    - 编程语言
tags:
    - Go
---

> **Golang Series**
>
> Hello GoLang: <https://blog.yexca.net/archives/154>  
> GoLang (var and const) 变量与常量: This Page  
> GoLang (func) 函数: <https://blog.yexca.net/archives/156>  
> GoLang (slice and map) 切片: <https://blog.yexca.net/archives/160>  
> GoLang (OOP) 面向对象: <https://blog.yexca.net/archives/162>  
> GoLang (reflect) 反射: <https://blog.yexca.net/archives/204>  
> GoLang (struct tag) 结构体标签: <https://blog.yexca.net/archives/205>  
> GoLang (goroutine) go 程: <https://blog.yexca.net/archives/206>  
> GoLang (channel) 通道: <https://blog.yexca.net/archives/207>  

## 变量

声明变量一般使用 var 关键字

### 单变量

* 定义类型

不指定初始值的声明，默认为 0

```go
package main

import "fmt"

func main() {
	var a int
	fmt.Println("a =", a)
}
```

指定初始值，a 为 100

```go
package main

import "fmt"

func main() {
	var a int = 100
	fmt.Println("a =", a)
}
```

* 省略类型

在声明时不知道类型的话，Go 会自动判断变量类型

```go
package main

import (
	"fmt"
	"reflect"
)

func main() {
	var a = 100
	fmt.Println("a =", a)
	fmt.Printf("Type of a = %s", reflect.TypeOf(a))
}
```

* :=

基于省略类型会自动判断，可以使用 := 直接声明变量

```go
package main

import (
	"fmt"
	"reflect"
)

func main() {
	p := 3.14
	fmt.Println("p =", p)
	fmt.Printf("Type of p is %s", reflect.TypeOf(p))
}

/*
 * 输出
 * p = 3.14
 * Type of p is float64
 */
```

### 多变量

* 相同类型

```go
package main

import "fmt"

func main() {
	var a, b int
	fmt.Println("a =", a)
	fmt.Println("b =", b)
}
```

* 相同类型赋值

```go
package main

import "fmt"

func main() {
	var a, b int = 100, 200
	fmt.Println("a =", a)
	fmt.Println("b =", b)
}
```

* 不同类型 1

```go
package main

import (
	"fmt"
	"reflect"
)

func main() {
	var a, b = 100, 3.14
	fmt.Println("a =", a)
	fmt.Printf("Type of a is %s\n", reflect.TypeOf(a))
	fmt.Println("b =", b)
	fmt.Printf("Type of b is %s", reflect.TypeOf(b))
}

/*
 * 输出
 * a = 100
 * Type of a is int
 * b = 3.14
 * Type of b is float64
 */
```

* 不同类型 2

```go
package main

import (
	"fmt"
	"reflect"
)

func main() {
	a, b := 100, "Hello"
	fmt.Println("a =", a)
	fmt.Printf("Type of a is %s\n", reflect.TypeOf(a))
	fmt.Println("b =", b)
	fmt.Printf("Type of b is %s", reflect.TypeOf(b))
}
```

> 字符串类型在 go 里是个结构，包含指向底层数组的指针和长度，这两部分每部分都是 8 个字节，所以字符串类型大小为 16 个字节
>
> 可以使用 unsafe.Sizeof() 函数查看类型占用

### 全局变量

全局变量的声明不能使用 :=

```go
package main

import (
	"fmt"
	"reflect"
)

var a, b int

func main() {
	fmt.Println("a =", a)
	fmt.Printf("Type of a is %s\n", reflect.TypeOf(a))
	fmt.Println("b =", b)
	fmt.Printf("Type of b is %s", reflect.TypeOf(b))
}
```

或者使用分解的写法，这种写法一般用于全局变量

```go
package main

import (
	"fmt"
	"reflect"
)

var (
	a int    = 1
	b string = "Go"
)

func main() {
	fmt.Println("a =", a)
	fmt.Printf("Type of a is %s\n", reflect.TypeOf(a))
	fmt.Println("b =", b)
	fmt.Printf("Type of b is %s", reflect.TypeOf(b))
}
```

## 常量

常量一般用 const 关键字定义

### 定义

```go
package main

import "fmt"

func main() {
	const c int = 9
	fmt.Println("c = ", c)
}
```

也可以省略类型

```go
package main

import "fmt"

func main() {
	const c = 9
	fmt.Println("c = ", c)
}
```

### 枚举

常量定义可以用于枚举

```go
package main

func main() {
	const (
		BEIJING = 0
		SHANGHAI = 1
		SHENZHEN = 2
	)
}
```

### iota 自增长

上述枚举以 0 开始递增，可以使用 iota 代替

```go
package main

import "fmt"

func main() {
	const (
		BEIJING = iota	// 0
		SHANGHAI		// 1
		SHENZHEN		// 2
	)

	fmt.Println(BEIJING, SHANGHAI, SHENZHEN)
}
```

> iota 可以用于表达式，但一般用于自增