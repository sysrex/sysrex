---
title: "Go variables"
date: 2020-06-01T14:16:40+01:00
tags: ["Golang"]
summary: "Just a small overview of the go variables"
draft: false
---

In Go, variables are explicitly declared and used by the compiler to e.g. check type-correctness of function calls.

```go
package main
   import "fmt" 

func main() {
  var a = "variable"
  fmt.Println(a)
}
```


```zsh
go run variables.go
variable
```