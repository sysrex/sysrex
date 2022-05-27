---
title: "Go maps"
date: 2021-02-07T10:31:42+01:00
tags: ["Golang"]
summary: "Just a small overview of the go maps"
draft: false
---

Go maps is the equivalent of hash tables from other languages essentially. We create maps by using the `make` keyword.

# Creating a map in Go
```go
var idMap map[key]value
idMap = make (map[key]value)
```

or by using literals

```go
idMap := map[key]value
```

# Adding and removing a value to the map in Go
```go
idMap["key2"] = value2
```

```go
delete(idMap, "key")
```

# Accessing a map in Go
```go
fmt.Println(idMap["key"]) <- this will print the value of the key
```

# Working with maps
```go
len(idMap)

for key, value := range idMap {
    fmt.Println(key,value)
}
```