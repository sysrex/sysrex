---
title: "Go variables"
date: 2019-10-16T14:16:40+01:00
tags: ["GO"]
summary: "Just a small overview of the go variables"
draft: false
---

In Go, variables are explicitly declared and used by the compiler to e.g. check type-correctness of function calls.

{{< highlight go >}}

package main
   import "fmt" 

func main() {
  var a = "variable"
  fmt.Println(a)
}

{{< /highlight >}}


{{< highlight go >}}

$ go run variables.go
variable
{{< /highlight >}}