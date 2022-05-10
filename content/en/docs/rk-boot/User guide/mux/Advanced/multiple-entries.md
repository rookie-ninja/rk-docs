---
title: "Multiple entries"
linkTitle: "Multiple entries"
weight: 7
description: >
  Start multiple entries in one process.
---

## Overview
User can start multiple Entry in one single process.

Furthermore, use can even start Gin and gRPC in one single process.

## Quick start
### 1.Install

```shell script
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-mux
```

### 2.Create boot.yaml
```yaml
mux:
  - name: alice
    port: 8080
    enabled: true
  - name: bob
    port: 8081
    enabled: true
```

### 3.Create main.go
```go
package main

import (
  "context"
  _ "embed"
  "fmt"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-mux/boot"
  "github.com/rookie-ninja/rk-mux/middleware"
  "net/http"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Register handler
  alice := rkmux.GetMuxEntry("alice")
  alice.Router.NewRoute().Path("/v1/greeter").HandlerFunc(Greeter)

  bob := rkmux.GetMuxEntry("bob")
  bob.Router.NewRoute().Path("/v1/greeter").HandlerFunc(Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

func Greeter(writer http.ResponseWriter, req *http.Request) {
  rkmuxmid.WriteJson(writer, http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", req.URL.Query().Get("name")),
  })
}

type GreeterResponse struct {
  Message string
}
```

### 4.Validate
```shell
$ curl  "localhost:8080/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}

$ curl  "localhost:8081/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)