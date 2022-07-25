Enable JWT middleware

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-grpc/v2
```

## Options
| options                     | description                        | type     | default                |
|----------------------------------------|--------------------------------| ------ |------------------------|
| grpc.middleware.jwt.enabled             | Enable JWT middleware                     | boolean | false                  |
| grpc.middleware.jwt.ignore  | Ignore by path                                                                       | []string | []                     |
| grpc.middleware.jwt.skipVerify           | Skip verify JWT token                                                   | boolean  | false                  |
| grpc.middleware.jwt.signerEntry         | SignerEntry name                 | string | ""                     |
| grpc.middleware.jwt.symmetric.algorithm | Symmetric algorithm, options: HS256, HS384, HS512                         | string | ""                     |
| grpc.middleware.jwt.symmetric.token     | Symmetric token                         | string | ""                     |
| grpc.middleware.jwt.symmetric.tokenPath | Symmetric token path                     | string | ""                     |
| grpc.middleware.jwt.asymmetric.algorithm| Asymmetric algorithm, options: RS256, RS384, RS512, ES256, ES384, ES512                        | string | ""                     |
| grpc.middleware.jwt.tokenLookup         | JWT Token format，see example bellow | string | "header:Authorization" |
| grpc.middleware.jwt.authScheme          | Auth Scheme                 | string | Bearer                 |

**tokenLookup**

```
// Optional. Default value "header:Authorization".
// Possible values:
// - "header:<name>"
// - "query:<name>"
// Multiply sources example:
// - "header: Authorization,cookie: myowncookie"
```

## Quick start
### 1.Create and compile protocol buffer
[Compile protobuf](../buf)

### 2.Create boot.yaml
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
#        ignore: [ "" ]
#        skipVerify: false
#        asymmetric:
#          algorithm: ""
#          privateKey: ""
#          privateKeyPath: ""
#          publicKey: ""
#          publicKeyPath: ""
#        tokenLookup: "header:<name>"
#        authScheme: "Bearer"
```

### 3.Create main.go
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

### 4.Validate
> Valid JWT Token

```bash
$ curl localhost:8080/v1/hello -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.EpM5XBzTJZ4J8AfoJEcJrjth8pfH28LWdjLo90sYb9g"
{"message":"hello!"}
```

> Invalid JWT Token

```bash
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