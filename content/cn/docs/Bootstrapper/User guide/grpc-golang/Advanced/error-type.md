---
title: "错误类型"
linkTitle: "错误类型"
weight: 8
description: >
  RK 推荐的 RPC 标准错误类型。
---

## 概述
设计一个合理的 API 是一件不容易的事情，同时，API 还会产生各种不同的错误。

为了能让 API 使用者对于 API 的错误有一个清晰的视图，定义一个标准的 RPC 错误类型是**非常重要**的事情。

当使用启动器的时候，panic 拦截器会默认加载到服务中，拦截器会从 panic 中恢复，并且返回[内部错误]给用户。

> 拦截器返回的错误类型定义：[rkerror.ErrorResp](https://github.com/rookie-ninja/rk-common/blob/master/error/error.go)

```go
func (server *GreeterServer) Greeter(ctx context.Context, request *greeter.GreeterRequest) (*greeter.GreeterResponse, error) {
	panic("Panic manually!")

	return &greeter.GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", request.Name),
	}, nil
}
```

```json
{
    "error":{
        "code":500,
        "status":"Internal Server Error",
        "message":"Panic manually!",
        "details":[]
    }
}
```

## 快速开始
- 安装

```shell script
$ go get github.com/rookie-ninja/rk-boot
$ go get github.com/rookie-ninja/rk-grpc
```

### RPC 中返回错误
> 本例子中，介绍如何返回错误给 API 使用者的[最佳实践] 

```go
func (server *GreeterServer) Greeter(ctx context.Context, request *greeter.GreeterRequest) (*greeter.GreeterResponse, error) {
	return nil, rkerror.Unimplemented("Trigger manually!", errors.New("this is detail")).Err()
}
```
```shell script
$ curl "localhost:8080/v1/greeter?name=rk-dev"
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

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)
