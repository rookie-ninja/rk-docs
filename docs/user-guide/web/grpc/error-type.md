What is the best way to return an RPC error?

## Overview
It is always not easy to define a user-friendly API. However, errors could happen anywhere.

In order to return standard error type to user, it is **important** to define an error type.

By default, panic middleware will handle unexpected panic and respond with google style error format:

- Google style

```json
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
```

- Amazon style

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

- Custom style

User can also define your own style which will be introduced bellow.

## Quick start
- Install

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-grpc/v2
```

### Default style(Google)
```go
func (server *GreeterServer) Greeter(ctx context.Context, request *greeter.GreeterRequest) (*greeter.GreeterResponse, error) {
    return nil, rkgrpcerr.Unimplemented("Trigger manually!", errors.New("this is detail")).Err()
}
```

```bash
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
```

### Amazon style
- boot.yaml

```yaml
grpc:
  - name: greeter              
    enabled: true              
    port: 8080
#   gwPort: 8081                  # Optional, default: gateway port will be the same as grpc port if not provided
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

### Custom style
User can register custom error builder and register it into Entry.

Please implement bellow interface.

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
