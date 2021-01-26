---
title: "Install go on mac"
date: 2021-01-04T14:16:40+01:00
tags: ["Golang"]
summary: "Installing and setup golang on mac Big Sur"
draft: false
---

# Installation
I use brew to install the latest version of mac:

```zsh
brew install golang
```

Then we check the installation:

```zsh
$ /usr/local/go/bin/go version
 go version go1.15.6 darwin/amd64
```

# Setting up GOPATH and GOBIN

```zsh
$ grep -q '^export PATH=/usr/local/go/bin:$PATH' ~/.zshrc || echo 'export PATH=/usr/local/go/bin:$PATH' >> ~/.zshrc
$ grep -q '^export GOPATH=~/go' ~/.zshrc || echo 'export GOPATH=~/go' >> ~/.zshrc
$ grep -q '^export GOBIN=$GOPATH/bin' ~/.zshrc || echo 'export GOBIN=$GOPATH/bin' >> ~/.zshrc
 
$ tail -n3 ~/.zshrc
export PATH=/usr/local/go/bin:$PATH
export GOPATH=~/go
export GOBIN=$GOPATH/bin
```

Now let's check out installation

```zsh
$ go version
go version go1.15.6 darwin/amd64

$ go env GOPATH
/Users/user/go

$ echo $GOPATH
/Users/user/go

$ echo $GOBIN
/Users/user/go/bin

$ go env
GO111MODULE=""
GOARCH="amd64"
GOBIN="/Users/user/go/bin"
GOCACHE="/Users/user/Library/Caches/go-build"
GOENV="/Users/user/Library/Application Support/go/env"
...
GOPROXY="https://proxy.golang.org,direct"
GOROOT="/usr/local/go"
GOSUMDB="sum.golang.org"
GOTMPDIR=""
GOTOOLDIR="/usr/local/go/pkg/tool/darwin_amd64"
...
```