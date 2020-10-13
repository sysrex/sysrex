---
title: Parse environment variables to structs in Go
tag: ["Golang"]
description: Parse environment variables to structs in Go
date: 2015-06-22T00:00:00-00:00
draft: false
---


In Go, it's dead simple to get the value from an environment variable:

```go
fmt.Println(os.Getenv("HOME"))
```

But, sometimes you have default values... so you would have to do something
like this:

```go
home := os.Getenv("HOME")
if home == "" {
  home = "THE DEFAULT HOME"
}
fmt.Println(home)
```

If you need those values in a lot of places, you would end up creating a
function for each one or something like that.

I found this to be extremely boring.

So, I created a small lib that let me do this:

```go
package main

import (
    "fmt"

    "github.com/caarlos0/env"
)

type Config struct {
    Home         string `env:"HOME"`
    Port         int    `env:"PORT" envDefault:"3000"`
    IsProduction bool   `env:"PRODUCTION"`
}

func main() {
    cfg := Config{}
    env.Parse(&cfg)
    fmt.Println(cfg)
}
```
