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

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-zero
```

### 2.Create boot.yaml
```yaml
zero:
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
  "encoding/json"
  "fmt"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-zero/boot"
  "github.com/zeromicro/go-zero/rest"
  "net/http"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Register handler
  alice := rkzero.GetZeroEntry("alice")
  alice.Server.AddRoute(rest.Route{
    Method:  http.MethodGet,
    Path:    "/v1/greeter",
    Handler: Greeter,
  })

  bob := rkzero.GetZeroEntry("bob")
  bob.Server.AddRoute(rest.Route{
    Method:  http.MethodGet,
    Path:    "/v1/greeter",
    Handler: Greeter,
  })

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

func Greeter(writer http.ResponseWriter, request *http.Request) {
  writer.WriteHeader(http.StatusOK)
  resp := &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", request.URL.Query().Get("name")),
  }
  bytes, _ := json.Marshal(resp)
  writer.Write(bytes)
}

type GreeterResponse struct {
  Message string
}
```

### 4.Validate
```bash
$ curl  "localhost:8080/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}

$ curl  "localhost:8081/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)