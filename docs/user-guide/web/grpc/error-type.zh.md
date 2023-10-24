RK 推荐的 RPC 标准错误类型。

## 概述
设计一个合理的 API 是一件不容易的事情，同时，API 还会产生各种不同的错误。

为了能让 API 使用者对于 API 的错误有一个清晰的视图，定义一个标准的 RPC 错误类型是**非常重要**的事情。

当使用 rk-boot 的时候，panic 中间件会默认加载到服务中，中间件会从 panic 中恢复，并且返回[内部错误]给用户。
默认使用 Google 样式的错误。

```json
{
  "error":{
    "code":500,
    "status":"Internal Server Error",
    "message":"Panic occurs",
    "details":[
      "panic manually"
    ]
  }
}
```

- Amazon 样式

```json
{
    "response":{
        "errors":[
            {
                "error":{
                    "code":500,
                    "status":"Internal Server Error",
                    "message":"Panic occurs",
                    "details":[
                        "panic manually"
                    ]
                }
            }
        ]
    }
}
```

- 自定义样式

请参照后面的例子。

## 快速开始
- 安装

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-grpc/v2
```

### RPC 中返回错误
> 本例子中，介绍如何返回错误给 API 使用者的[最佳实践]

```go
func (server *GreeterServer) Greeter(ctx context.Context, request *greeter.GreeterRequest) (*greeter.GreeterResponse, error) {
    return nil, rkgrpcerr.Unimplemented("Trigger manually!", errors.New("this is detail")).Err()
}
```

```shell script
$ curl localhost:8080/v1/hello
{
    "error":{
        "code":501,
        "status":"Not Implemented",
        "message":"Trigger manually!",
        "details":[
            {
                "code":12,
                "status":"Unimplemented",
                "message":"[from-grpc] Trigger manually!"
            },
            {
                "code":2,
                "status":"Unknown",
                "message":"this is detail"
            }
        ]
    }
}
```

### Amazon 样式错误
- boot.yaml

```yaml
grpc:
  - name: greeter              
    enabled: true              
    port: 8080
#   gwPort: 8081                  # 可选项，如果不指定，会使用与 port 一样的端口
    enableRkGwOption: true
    middleware:
      errorModel: amazon
```

```json
{
  "response":{
    "errors":[
      {
        "error":{
          "code":501,
          "status":"Not Implemented",
          "message":"Trigger manually!",
          "details":[
            {
              "code":12,
              "status":"Unimplemented",
              "message":"Trigger manually!"
            },
            {
              "code":2,
              "status":"Unknown",
              "message":"this is detail"
            }
          ]
        }
      }
    ]
  }
}
```

### 自定义样式
请实现如下两个接口，并注册到 Entry 中。

```go
type ErrorInterface interface {
	Error() string

	Code() int

	Message() string

	Details() []interface{}
}

type ErrorBuilder interface {
	New(code int, msg string, details ...interface{}) ErrorInterface

	NewCustom() ErrorInterface
}
```

- main.go

```go
package main

import (
	"context"
	"errors"
	"fmt"
	"github.com/rookie-ninja/rk-boot/v2"
	"github.com/rookie-ninja/rk-demo/api/gen/v1"
	"github.com/rookie-ninja/rk-entry/v2/error"
	"github.com/rookie-ninja/rk-entry/v2/middleware"
	"github.com/rookie-ninja/rk-grpc/v2/boot"
	"github.com/rookie-ninja/rk-grpc/v2/boot/error"
	"google.golang.org/grpc"
	"net/http"
)

func main() {
	boot := rkboot.NewBoot()

	// register grpc
	entry := rkgrpc.GetGrpcEntry("greeter")
	entry.AddRegFuncGrpc(registerGreeter)
	entry.AddRegFuncGw(greeter.RegisterGreeterHandlerFromEndpoint)

	// Bootstrap
	boot.Bootstrap(context.TODO())

	// Set default error builder after bootstrap
	rkmid.SetErrorBuilder(&MyErrorBuilder{})

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.TODO())
}

func registerGreeter(server *grpc.Server) {
	greeter.RegisterGreeterServer(server, &GreeterServer{})
}

//GreeterServer GreeterServer struct
type GreeterServer struct{}

// Hello response with hello message
func (server *GreeterServer) Hello(_ context.Context, _ *greeter.HelloRequest) (*greeter.HelloResponse, error) {
	return nil, rkgrpcerr.Unimplemented("Trigger manually!", errors.New("this is detail")).Err()
}

type MyError struct {
	ErrCode    int
	ErrMsg     string
	ErrDetails []interface{}
}

func (m MyError) Error() string {
	return fmt.Sprintf("%d-%s", m.ErrCode, m.ErrMsg)
}

func (m MyError) Code() int {
	return m.ErrCode
}

func (m MyError) Message() string {
	return m.ErrMsg
}

func (m MyError) Details() []interface{} {
	return m.ErrDetails
}

type MyErrorBuilder struct{}

func (m *MyErrorBuilder) New(code int, msg string, details ...interface{}) rkerror.ErrorInterface {
	return &MyError{
		ErrCode:    code,
		ErrMsg:     msg,
		ErrDetails: details,
	}
}

func (m *MyErrorBuilder) NewCustom() rkerror.ErrorInterface {
	return &MyError{
		ErrCode:    http.StatusInternalServerError,
		ErrMsg:     "Internal Error",
		ErrDetails: []interface{}{},
	}
}
```

```json
{
    "ErrCode":501,
    "ErrMsg":"Trigger manually!",
    "ErrDetails":[
        {
            "code":12,
            "status":"Unimplemented",
            "message":"Trigger manually!"
        },
        {
            "code":2,
            "status":"Unknown",
            "message":"this is detail"
        }
    ]
}
```


### _**Cheers**_
![](../../../img/user-guide/cheers.png)
