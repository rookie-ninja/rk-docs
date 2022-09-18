---
title: "Error type"
linkTitle: "Error type"
weight: 8
description: >
  What is the best way to return an RPC error?
---

## Overview
It is always not easy to define a user-friendly API. However, errors could happen anywhere.

In order to return standard error type to user, it is **important** to define an error type.

By default, panic middleware will handle unexpected panic and respond with google style error format:

- Google style

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

- Amazon style

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

- Custom style

User can also define your own style which will be introduced bellow.

## Quick start
### 1.Install

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-zero
```

### 2.Create boot.yaml
```yaml
zero:
  - name: greeter
    port: 8080
    enabled: true
```

### 3.Create main.go
```go
package main

import (
  "context"
  "embed"
  "encoding/json"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-entry/v2/entry"
  "github.com/rookie-ninja/rk-entry/v2/error"
  "github.com/rookie-ninja/rk-zero/boot"
  "github.com/zeromicro/go-zero/rest"
  "net/http"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Register handler
  zeroEntry := rkzero.GetZeroEntry("greeter")
  zeroEntry.Server.AddRoute(rest.Route{
    Method:  http.MethodGet,
    Path:    "/v1/greeter",
    Handler: Greeter,
  })

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

func Greeter(writer http.ResponseWriter, request *http.Request) {
  err := rkmid.GetErrorBuilder().New(http.StatusAlreadyReported, "Trigger manually!",
    "This is detail.",
    false, -1,
    0.1)
  writer.WriteHeader(http.StatusAlreadyReported)
  bytes, _ := json.Marshal(err)
  writer.Write(bytes)
}
```

```bash
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

## Amazon style error code
- boot.yaml

```yaml
mux:
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

## Custom style
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

### 1.main.go

```go
package main

import (
	"context"
	"encoding/json"
	"fmt"
	"github.com/rookie-ninja/rk-boot/v2"
	"github.com/rookie-ninja/rk-entry/v2/error"
	"github.com/rookie-ninja/rk-entry/v2/middleware"
	"github.com/rookie-ninja/rk-zero/boot"
	"github.com/zeromicro/go-zero/rest"
	"net/http"
)

func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Register handler
	zeroEntry := rkzero.GetZeroEntry("greeter")
	zeroEntry.Server.AddRoute(rest.Route{
		Method:  http.MethodGet,
		Path:    "/v1/greeter",
		Handler: Greeter,
	})

	// Bootstrap
	boot.Bootstrap(context.TODO())

	// Set default error builder after bootstrap
	rkmid.SetErrorBuilder(&MyErrorBuilder{})

	boot.WaitForShutdownSig(context.TODO())
}

func Greeter(writer http.ResponseWriter, request *http.Request) {
	writer.WriteHeader(http.StatusAlreadyReported)
	resp := rkmid.GetErrorBuilder().New(http.StatusAlreadyReported, "Trigger manually!",
		"This is detail.",
		false, -1,
		0.1)
	bytes, _ := json.Marshal(resp)
	writer.Write(bytes)
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

### 2.Validate

```bash
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
![](../../../img/user-guide/cheers.png)
