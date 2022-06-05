---
title: "错误类型"
linkTitle: "错误类型"
weight: 8
description: >
  rk-boot 推荐的 RPC 标准错误类型。
---

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
### 1.安装

```shell script
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-gf
```

### 2.创建 boot.yaml
```yaml
gf:
  - name: greeter
    port: 8080
    enabled: true
```

### 3.创建 main.go
```go
package main

import (
  "context"
  "fmt"
  "github.com/gogf/gf/v2/net/ghttp"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-entry/v2/error"
  "github.com/rookie-ninja/rk-gf/boot"
  "net/http"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Add shutdown hook function
  boot.AddShutdownHookFunc("shutdown-hook", func() {
    fmt.Println("shutting down")
  })

  // Register handler
  entry := rkgf.GetGfEntry("greeter")
  entry.Server.BindHandler("/v1/greeter", Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

func Greeter(ctx *ghttp.Request) {
  ctx.Response.WriteHeader(http.StatusAlreadyReported)
  ctx.Response.WriteJson(rkmid.GetErrorBuilder().New(http.StatusAlreadyReported, "Trigger manually!",
    "This is detail.",
    false, -1,
    0.1))
}
```


```shell script
$ curl "localhost:8080/v1/greeter?name=rk-dev"
{
    "error":{
        "code":208,
        "status":"Already Reported",
        "message":"Trigger manually!",
        "details":[
            "This is detail.",
            false,
            -1,
            0.1
        ]
    }
}
```

## Amazon 样式错误
- boot.yaml

```yaml
echo:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      errorModel: amazon
```

```json
{
    "response":{
        "errors":[
            {
                "error":{
                    "code":208,
                    "status":"Already Reported",
                    "message":"Trigger manually!",
                    "details":[
                        "This is detail.",
                        false,
                        -1,
                        0.1
                    ]
                }
            }
        ]
    }
}
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)

## 自定义样式
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

### 1.main.go

```go
package main

import (
  "context"
  "fmt"
  "github.com/gogf/gf/v2/net/ghttp"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-entry/v2/error"
  "github.com/rookie-ninja/rk-entry/v2/middleware"
  "github.com/rookie-ninja/rk-gf/boot"
  "net/http"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Register handler
  entry := rkgf.GetGfEntry("greeter")
  entry.Server.BindHandler("/v1/greeter", Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  // Set default error builder after bootstrap
  rkmid.SetErrorBuilder(&MyErrorBuilder{})

  boot.WaitForShutdownSig(context.TODO())
}

func Greeter(ctx *ghttp.Request) {
  ctx.Response.WriteHeader(http.StatusAlreadyReported)
  ctx.Response.WriteJson(rkmid.GetErrorBuilder().New(http.StatusAlreadyReported, "Trigger manually!",
    "This is detail.",
    false, -1,
    0.1))
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

### 2.验证

```shell
$ curl "localhost:8080/v1/greeter?name=rk-dev"
{
    "ErrCode":208,
    "ErrMsg":"Trigger manually!",
    "ErrDetails":[
        "This is detail.",
        false,
        -1,
        0.1
    ]
}
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)
