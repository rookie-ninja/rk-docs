如何在 rk-boot 中，启动多个服务？

## 概述
通过 rk-boot，用户可以在一个进程里面，启动多个 Entry。

用户还可以同时启动 Gin 服务和 gRPC 等服务。

## 快速开始
### 1.安装

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-mux
```

### 2.创建 boot.yaml
```yaml
mux:
  - name: alice
    port: 8080
    enabled: true
  - name: bob
    port: 8081
    enabled: true
```

### 3.创建 main.go
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

### 4.验证
```bash
$ curl  "localhost:8080/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}

$ curl  "localhost:8081/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)