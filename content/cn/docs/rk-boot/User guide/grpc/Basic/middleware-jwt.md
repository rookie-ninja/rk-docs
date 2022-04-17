---
title: "JWT 中间件"
linkTitle: "JWT 中间件"
weight: 14
description: >
  启动 JWT 中间件。
---

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-grpc/v2
```

## 选项
| 名字                                     | 描述                             | 类型 | 默认值 |
|----------------------------------------|--------------------------------| ------ | -- |
| grpc.middleware.jwt.enabled             | 启动 JWT 中间件                     | boolean | false |
| grpc.middleware.jwt.ignore              | 局部选项，忽略 API 路径                 | []string | []             |
| grpc.middleware.jwt.signerEntry         | SignerEntry 名称                 | string | "" |
| grpc.middleware.jwt.symmetric.algorithm | 对称加密算法, 选项：HS256, HS384, HS512                         | string | "" |
| grpc.middleware.jwt.symmetric.token     | 对称加密密钥                         | string | "" |
| grpc.middleware.jwt.symmetric.tokenPath | 对称加密密钥本地路径                     | string | "" |
| grpc.middleware.jwt.asymmetric.algorithm| 非对称加密算法, 选项：RS256, RS384, RS512, ES256, ES384, ES512                        | string | "" |
| grpc.middleware.jwt.tokenLookup         | 寻找 JWT Token 的格式，参考下面的例子 | string | "header:Authorization" |
| grpc.middleware.jwt.authScheme          | 提供 Auth Scheme                 | string | Bearer |

**tokenLookup** 格式

```
// Optional. Default value "header:Authorization".
// Possible values:
// - "header:<name>"
// - "query:<name>"
// Multiply sources example:
// - "header: Authorization,cookie: myowncookie"
```

## 快速开始
### 1.创建并编译 protocol buffer
[使用 buf 编译 protocol buf](/cn/docs/rk-boot/user-guide/grpc/basic/buf/)

### 2.创建 boot.yaml
```yaml
---
grpc:
  - name: greeter
    port: 8080
    enabled: true
    enableRkGwOption: true
    middleware:
      jwt:
        enabled: true
        symmetric:
          algorithm: HS256
          token: "my-secret"
#          tokenPath: ""
#        signerEntry: ""
#        asymmetric:
#          algorithm: ""
#          privateKey: ""
#          privateKeyPath: ""
#          publicKey: ""
#          publicKeyPath: ""
#        tokenLookup: "header:<name>"
#        authScheme: "Bearer"
```

### 3.创建 main.go
```go
package main

import (
  "context"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-demo/api/gen/v1"
  "github.com/rookie-ninja/rk-grpc/v2/boot"
  "github.com/rookie-ninja/rk-grpc/v2/middleware/context"
  "google.golang.org/grpc"
)

func main() {
  boot := rkboot.NewBoot()

  // register grpc
  entry := rkgrpc.GetGrpcEntry("greeter")
  entry.AddRegFuncGrpc(registerGreeter)
  entry.AddRegFuncGw(greeter.RegisterGreeterHandlerFromEndpoint)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  // Wait for shutdown sig
  boot.WaitForShutdownSig(context.TODO())
}

func registerGreeter(server *grpc.Server) {
  greeter.RegisterGreeterServer(server, &GreeterServer{})
}

type GreeterServer struct{}

func (server *GreeterServer) Hello(ctx context.Context, _ *greeter.HelloRequest) (*greeter.HelloResponse, error) {
  // Get JWT token
  rkgrpcctx.GetJwtToken(ctx)

  return &greeter.HelloResponse{
    Message: "hello!",
  }, nil
}
```

### 4.验证
> 正确的 JWT Token

```shell script
$ curl localhost:8080/v1/hello -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.EpM5XBzTJZ4J8AfoJEcJrjth8pfH28LWdjLo90sYb9g"
{"message":"hello!"}
```

> 非法 JWT Token

```shell script
$ curl localhost:8080/v1/hello -H "Authorization: Bearer invalid-jwt-token"
{
    "error":{
        "code":401,
        "status":"Unauthorized",
        "message":"Invalid or expired jwt",
        "details":[
            {
                "code":16,
                "status":"Unauthenticated",
                "message":"[from-grpc] Invalid or expired jwt"
            }
        ]
    }
}
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)