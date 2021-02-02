---
title: "Go pointers"
date: 2021-01-07T10:31:42+01:00
tags: ["Golang"]
summary: "Just a small overview of the go pointers"
draft: false
---

Go supports pointers, allowing you to pass references to values and records within your program.

The `&` syntax gives the memory address of the variable, while the `*` gives the value fo the variable.

```go
package main
import "fmt"

var i int = 4

func main() {
	fmt.Println(i)
	fmt.Println(&i)
}
```


```go
$ go run pointers.go
4
0x53e108
```