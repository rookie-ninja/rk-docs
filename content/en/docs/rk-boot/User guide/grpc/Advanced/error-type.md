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
- Install

```shell script
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-grpc/v2
```

### RPC Error
```go
func (server *GreeterServer) Greeter(ctx context.Context, request *greeter.GreeterRequest) (*greeter.GreeterResponse, error) {
	return nil, rkerror.Unimplemented("Trigger manually!", errors.New("this is detail")).Err()
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

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)
