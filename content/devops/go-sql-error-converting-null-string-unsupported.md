---
title: "Go SQL error “converting NULL to string is unsupported”"
date: 2023-11-10T14:16:40+01:00
tags: ["Golang"]
summary: "Go SQL error “converting NULL to string is unsupported”"
draft: false
---

Go is a strongly typed programming language, and many SQL databases also support unknown values. These unknowns can lead to complications in Go. Such as in this example, when a NULL value is in use for an empty string, it throws the unhelpful error converting NULL to string is unsupported.

```go
import (
    "fmt"
)

// LookupName returns the username from database ID.
func LookupName(id int) (string, error) {
    db := Connect()
    defer db.Close()
    var name string
    err := db.QueryRow("SELECT username FROM accounts WHERE id=?", id).Scan(&name)
    if err != nil {
        return "", fmt.Errorf("lookup name by id %q: %w", id, err)
    }
    return name, nil
}
```


```zsh
sql: Scan error on column index 0, name "username":
converting NULL to string is unsupported
```

Instead of saving SQL query values as basic Go types, database table columns that support NULL or other unknowns should save their values to the database.sql Null types. These include [NullString](https://pkg.go.dev/database/sql#NullString), NullBool, NullInt64, NullFloat64, and NullTime.



```go

import (
    "database/sql"
    "fmt"
)

// LookupName returns the username from database ID.
func LookupName(id int) (string, error) {
    db := Connect()
    defer db.Close()
    var name sql.NullString
    err := db.QueryRow("SELECT username FROM accounts WHERE id=?", id).Scan(&name)
    if err != nil {
        return "", fmt.Errorf("lookup name by id %q: %w", id, err)
    }
    return name.String, nil
}

```