---
title: "Middleware logging"
linkTitle: "Middleware logging"
weight: 6
description: >
  Enable RPC logging interceptor/middleware for server.
---

## Installation
```shell script
go get github.com/rookie-ninja/rk-boot
go get github.com/rookie-ninja/rk-gf
```

## General options
> These are general options to start a GoFrame server with rk-boot

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.name | The name of GoFrame server | string | N/A |
| gf.port | The port of GoFrame server | integer | nil, server won't start |
| gf.enabled | Enable GoFrame entry | bool | false |
| gf.description | Description of GoFrame entry. | string | "" |

## Logging options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.interceptors.loggingZap.enabled | Enable log interceptor | boolean | false |
| gf.interceptors.loggingZap.zapLoggerEncoding | json or console | string | console |
| gf.interceptors.loggingZap.zapLoggerOutputPaths | Output paths | []string | stdout |
| gf.interceptors.loggingZap.eventLoggerEncoding | json or console | string | console |
| gf.interceptors.loggingZap.eventLoggerOutputPaths | Output paths | []string | false |

## Concept
In order to monitor every RPC request, we introduce [EventLogger](https://github.com/rookie-ninja/rk-query) and [ZapLogger](https://github.com/uber-go/zap)

### ZapLogger
RK bootstrapper adopt [zap](https://github.com/uber-go/zap) as the logger. [lumberjack](https://github.com/natefinch/lumberjack) as the logger rotation.

> Example
> 
> ```shell script
> 2021-12-06T15:16:15.366+0800    INFO    boot/gf_entry.go:848    Bootstrapping GfEntry.  {"eventId": "abec7c4f-7678-45a2-8824-df315864eea0", "entryName": "greeter", "entryType": "GfEntry", "port": 8080}
> ```

### EventLogger
RK bootstrapper treat RPC request as an **Event**, and record every RPC request into Event type in [rk-query](https://github.com/rookie-ninja/rk-query).

| Field | Description |
| ---- | ---- |
| endTime | As name described |
| startTime | As name described |
| elapsedNano | Elapsed time for RPC in nanoseconds |
| timezone | As name described |
| ids | Contains three different ids(eventId, requestId and traceId). If meta interceptor was enabled or event.SetRequestId() was called by user, then requestId would be attached. eventId would be the same as requestId if meta interceptor was enabled. If trace interceptor was enabled, then traceId would be attached. |
| app | Contains [appName, appVersion](https://github.com/rookie-ninja/rk-entry#appinfoentry), entryName, entryType. |
| env | Contains arch, az, domain, hostname, localIP, os, realm, region. realm, region, az, domain were retrieved from environment variable named as REALM, REGION, AZ and DOMAIN. "*" means empty environment variable.|
| payloads | Contains RPC related metadata |
| error | Contains errors if occur |
| counters | Set by calling event.SetCounter() by user. |
| pairs | Set by calling event.AddPair() by user. |
| timing | Set by calling event.StartTimer() and event.EndTimer() by user. |
| remoteAddr |  As name described |
| operation | RPC method name |
| resCode | Response code of RPC |
| eventStatus | Ended or InProgress |

> Example
> 
> ```shell script
> ------------------------------------------------------------------------
> endTime=2021-06-25T01:30:45.144023+08:00
> startTime=2021-06-25T01:30:45.143767+08:00
> elapsedNano=255948
> timezone=CST
> ids={"eventId":"3332e575-43d8-4bfe-84dd-45b5fc5fb104","requestId":"3332e575-43d8-4bfe-84dd-45b5fc5fb104","traceId":"65b9aa7a9705268bba492fdf4a0e5652"}
> app={"appName":"rk-gf","appVersion":"master-xxx","entryName":"greeter","entryType":"GfEntry"}
> env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"*","region":"*"}
> payloads={"apiMethod":"GET","apiPath":"/rk/v1/healthy","apiProtocol":"HTTP/1.1","apiQuery":"","userAgent":"curl/7.64.1"}
> error={}
> counters={}
> pairs={}
> timing={}
> remoteAddr=localhost:60718
> operation=/rk/v1/healthy
> resCode=200
> eventStatus=Ended
> EOE
> ```

## Quick start
### 1.Create boot.yaml
```yaml
---
gf:
  - name: greeter
    port: 8080
    enabled: true
    commonService:
      enabled: true          # Enable common service for testing
    interceptors:
      loggingZap:
        enabled: true        # Enable logging interceptor/middleware, by default, stdout would be destination of log
```

### 2.Create main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
	_ "github.com/rookie-ninja/rk-gf/boot"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

### 3.Validate
> Send request

```shell script
$ curl -X GET localhost:8080/rk/v1/healthy
{"healthy":true}
```

> Log

```shell script
------------------------------------------------------------------------
endTime=2021-07-05T23:42:35.588164+08:00
startTime=2021-07-05T23:42:35.588095+08:00
elapsedNano=69414
timezone=CST
ids={"eventId":"9b874eea-b16b-4c46-b0f5-d2b7cff6844e"}
app={"appName":"rk-demo","appVersion":"master-f414049","entryName":"greeter","entryType":"GfEntry"}
env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"*","region":"*"}
payloads={"apiMethod":"GET","apiPath":"/rk/v1/healthy","apiProtocol":"HTTP/1.1","apiQuery":"","userAgent":"curl/7.64.1"}
error={}
counters={}
pairs={}
timing={}
remoteAddr=localhost:56274
operation=/rk/v1/healthy
resCode=200
eventStatus=Ended
EOE
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 4.JSON format
```yaml
---
gf:
  - name: greeter
    ...
    interceptors:
      loggingZap:
        ...
        zapLoggerEncoding: "json"          # Override to json format, option: json or console
        eventLoggerEncoding: "json"        # Override to json format, option: json or console
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 5.Output paths
> By default, logs will be rotated by 1GB and compressed.

```yaml
---
gf:
  - name: greeter
    ...
    interceptors:
      loggingZap:
        ...
        zapLoggerOutputPaths: ["logs/app.log"]        # Override output paths, option: json or console
        eventLoggerOutputPaths: ["logs/event.log"]    # Override output paths, option: json or console
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 5.Get RPC scope logger
A new zap logger instance with requestId(if exist enabled by meta interceptor) will be created for every RPC request.
```go
func Greeter(ctx *ghttp.Request) {
	rkgfctx.GetLogger(ctx).Info("Received request")

	ctx.Response.WriteHeader(http.StatusOK)
	err := ctx.Response.WriteJson(&GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", ctx.GetQuery("name")),
	})

	if err != nil {
		panic(err)
	}
}
```

```shell script
2021-07-06T02:04:55.306+0800    INFO    basic/main.go:43        Received request        {"requestId": "b8522178-9fac-47d3-b866-ce43e119f7a3"}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 6.Modify Event
A new event logger instance will be created for every RPC request.

User can add pairs, counters, errors in event which will be logged as soon as RPC finish.
```go
func Greeter(ctx *ghttp.Request) {
	event := rkgfctx.GetEvent(ctx)
	event.AddPair("key", "value")

	ctx.Response.WriteHeader(http.StatusOK)
	err := ctx.Response.WriteJson(&GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", ctx.GetQuery("name")),
	})

	if err != nil {
		panic(err)
	}
}
```

```shell script
------------------------------------------------------------------------
endTime=2021-12-06T15:26:53.364949+08:00
startTime=2021-12-06T15:26:53.364839+08:00
elapsedNano=110410
timezone=CST
ids={"eventId":"1271b868-b875-40f7-9b74-e0013675acc1"}
app={"appName":"rk-demo","appVersion":"master-878d8ab","entryName":"greeter","entryType":"GfEntry"}
env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"*","region":"*"}
payloads={"apiMethod":"GET","apiPath":"/v1/greeter","apiProtocol":"HTTP/1.1","apiQuery":"","userAgent":"curl/7.64.1"}
error={}
counters={}
pairs={"key":"value"}
timing={}
remoteAddr=localhost:63073
operation=/v1/greeter
resCode=200
eventStatus=Ended
EOE
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)
