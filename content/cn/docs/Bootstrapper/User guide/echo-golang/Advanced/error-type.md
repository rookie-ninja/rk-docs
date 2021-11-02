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
func Greeter(ctx echo.Context) error {
	return panic("Panic manually!")
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
### RPC 中返回错误
> 本例子中，介绍如何返回错误给 API 使用者的[最佳实践] 

```go
func Greeter(ctx echo.Context) error {
	err := rkerror.New(
		rkerror.WithHttpCode(http.StatusAlreadyReported),
		rkerror.WithMessage("Trigger manually!"),
		rkerror.WithDetails("This is detail.", false, -1, 0.1))

	return ctx.JSON(http.StatusAlreadyReported, err)
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

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)
