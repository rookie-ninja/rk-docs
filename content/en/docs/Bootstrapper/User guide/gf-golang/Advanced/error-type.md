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

By default, panic middleware/interceptor will be attached which will catch panic and return internal server error as bellow:

> Error type used by bootstrapper was defined in [rkerror.ErrorResp](https://github.com/rookie-ninja/rk-common/blob/master/error/error.go)

```go
func Greeter(ctx *ghttp.Request) {
	panic("Panic manually!")
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

## Quick start
### Return errors
> Here is the way how to return errors in user RPC implementation.

```go
func Greeter(ctx *ghttp.Request) {
	err := rkerror.New(
		rkerror.WithHttpCode(http.StatusAlreadyReported),
		rkerror.WithMessage("Trigger manually!"),
		rkerror.WithDetails("This is detail.", false, -1, 0.1))

	ctx.Response.WriteHeader(http.StatusAlreadyReported)
	ctx.Response.WriteJson(err)
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
