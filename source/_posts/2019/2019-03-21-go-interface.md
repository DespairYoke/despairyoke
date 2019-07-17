---
layout: post
title:  接口的使用
category: go语言
tags: [go]
date: 2019-03-21
description: 学习go语言接口记录
---

### 接口的特点

- 可以包含0个或多个方法的签名
- 只定义方法的签名，不包含实现
- 实现接口不需要显式的声明，只需实现相应方法即可

### 接口的定义

定义方式如下：

```go
type Namer interface {
    method1(param_list) return_list
    method1(param_list) return_list
    ...
}
```

这里的 Namer 就是接口的名字，只要符合标识符的规则即可。不过，通常约定的接口的名字最好以 er,　r, able 结尾(视情况而定)，这样一眼就知道它是接口。

### 实现接口

在 go 中实现接口很简单，不需要显式的声明实现了某个接口，想要实现某个接口，只需要实现接口中的所有方法即可。

```go
package main

import (
	"fmt"
	"math"
)

type Shaper interface {
	Area() float32
	Circumference() float32
}

type Rect struct {
	Width float32
	Height float32
}

type Circle struct {
	Radius float32
}

func (r Rect) Area() float32  {
	return r.Width * r.Height
}

func (r Rect) Circumference() float32 {
	return 2 * (r.Width + r.Height)
}

func (c Circle) Area() float32 {
	return math.Pi * c.Radius * c.Radius
}

func (c Circle) Circumference() float32 {
	return math.Pi * 2 * c.Radius
}

func showInfo(s Shaper) {
	fmt.Printf("Area: %f, Circumference: %f", s.Area(), s.Circumference())
}

func main () {
	r := Rect{10, 20}
	showInfo(r)

	c := Circle{5}
	showInfo(c)
}

```

这里`Rect`和`Circle`都实现了这个接口。

### 获取实现接口的实际类型

在上面的 `showInfo` 中我们传入了接口类型的对象，如果将实现了接口的类型传递进去，那么会将实际类型的其他特性掩盖住，因此通常我们会想要获取其真正的类型, 可以使用下面的方法:

####  switch的使用

```go
func showInfo(s Shaper) {
	switch s.(type) {
	case Rect:
		fmt.Println("this is Rect")
	case Circle:
		fmt.Println("this is Circle")
	}
}
```

#### 断言的使用

使用类型断言，来判断某一时刻接口是否是某个具体类型

```go
v, ok := s.(Rect)   // s 是一个接口类型
```

如果 s 此时实际上是 Rect 类型，那么会将 s 转换为 Rect 类型，并且 ok 为 true。否则 ok 为 false。