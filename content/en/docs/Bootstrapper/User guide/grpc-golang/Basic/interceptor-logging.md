---
title: "Interceptor logging"
linkTitle: "Interceptor logging"
weight: 7
description: >
  Enable RPC logging interceptor/middleware for server.
---

## Installation
```shell script
go get github.com/rookie-ninja/rk-boot
```

## General options
> These are general options to start a grpc server with rk-boot

| name | description | type | default value | Required |
| ------ | ------ | ------ | ------ | ------ |
| grpc.name | The name of grpc server | string | "", server won't start | Required |
| grpc.port | The port of grpc server | integer | 0, server won't start | Required |
| grpc.enabled | Enable grpc entry | bool | false | Required |
| grpc.description | Description of grpc entry. | string | "" | Optional |

## Logging options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.loggingZap.enabled | Enable log interceptor | boolean | false |
| grpc.interceptors.loggingZap.zapLoggerEncoding | json or console | string | console |
| grpc.interceptors.loggingZap.zapLoggerOutputPaths | Output paths | []string | stdout |
| grpc.interceptors.loggingZap.eventLoggerEncoding | json or console | string | console |
| grpc.interceptors.loggingZap.eventLoggerOutputPaths | Output paths | []string | stdout |

## Concept
In order to monitor every RPC request, we introduce [EventLogger](https://github.com/rookie-ninja/rk-query) and [ZapLogger](https://github.com/uber-go/zap)

### ZapLogger
RK bootstrapper adopt [zap](https://github.com/uber-go/zap) as the logger. [lumberjack](https://github.com/natefinch/lumberjack) as the logger rotation.

> Example
> 
> ```shell script
> 2021-07-09T23:52:13.667+0800    INFO    boot/grpc_entry.go:694  Bootstrapping grpcEntry.        {"eventId": "9bc192fb-567c-45d4-8775-7a097b0dab04", "entryName": "greeter", "entryType": "GrpcEntry", "grpcPort": 8080, "commonServiceEnabled": true, "tlsEnabled": false, "gwEnabled": true, "reflectionEnabled": false, "swEnabled": false, "tvEnabled": false, "promEnabled": false, "gwClientTlsEnabled": false, "gwServerTlsEnabled": false}
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
> endTime=2021-07-09T23:44:09.81483+08:00
> startTime=2021-07-09T23:44:09.814784+08:00
> elapsedNano=46065
> timezone=CST
> ids={"eventId":"67d64dab-f3ea-4b77-93d0-6782caf4cfee"}
> app={"appName":"rk-demo","appVersion":"master-f414049","entryName":"greeter","entryType":"GrpcEntry"}
> env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"*","region":"*"}
> payloads={"grpcMethod":"Healthy","grpcService":"rk.api.v1.RkCommonService","grpcType":"unaryServer","gwMethod":"","gwPath":"","gwScheme":"","gwUserAgent":""}
> error={}
> counters={}
> pairs={"healthy":"true"}
> timing={}
> remoteAddr=localhost:58205
> operation=/rk.api.v1.RkCommonService/Healthy
> resCode=OK
> eventStatus=Ended
> EOE
> ```

## Quick start
### 1.Create boot.yaml
```yaml
---
grpc:
  - name: greeter                   # Name of grpc entry
    port: 8080                      # Port of grpc entry
    enabled: true                   # Enable grpc entry
    commonService:
      enabled: true                 # Enable common service for testing
    interceptors:
      loggingZap:
        enabled: true
```

### 2.Create main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
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
endTime=2021-07-09T23:44:09.81483+08:00
startTime=2021-07-09T23:44:09.814784+08:00
elapsedNano=46065
timezone=CST
ids={"eventId":"67d64dab-f3ea-4b77-93d0-6782caf4cfee"}
app={"appName":"rk-demo","appVersion":"master-f414049","entryName":"greeter","entryType":"GrpcEntry"}
env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"*","region":"*"}
payloads={"grpcMethod":"Healthy","grpcService":"rk.api.v1.RkCommonService","grpcType":"unaryServer","gwMethod":"","gwPath":"","gwScheme":"","gwUserAgent":""}
error={}
counters={}
pairs={"healthy":"true"}
timing={}
remoteAddr=localhost:58205
operation=/rk.api.v1.RkCommonService/Healthy
resCode=OK
eventStatus=Ended
EOE
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 4.JSON format
```yaml
---
grpc:
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
grpc:
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
func (server *GreeterServer) Greeter(ctx context.Context, request *greeter.GreeterRequest) (*greeter.GreeterResponse, error) {
	rkgrpcctx.GetLogger(ctx).Info("Received request")

	return &greeter.GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", request.Name),
	}, nil
}
```
```shell script
2021-07-09T23:50:39.318+0800    INFO    basic/main.go:36        Received request        {"requestId": "c33698f2-3071-48d4-9d92-b1aa311e6c06"}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 6.Modify Event
A new event logger instance will be created for every RPC request.

User can add pairs, counters, errors in event which will be logged as soon as RPC finish.
```go
func (server *GreeterServer) Greeter(ctx context.Context, request *greeter.GreeterRequest) (*greeter.GreeterResponse, error) {
	event := rkgrpcctx.GetEvent(ctx)
	event.AddPair("key", "value")

	return &greeter.GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", request.Name),
	}, nil
}
```
```shell script
------------------------------------------------------------------------
endTime=2021-07-09T23:52:39.103351+08:00
startTime=2021-07-09T23:52:39.10332+08:00
elapsedNano=31154
timezone=CST
ids={"eventId":"92001951-80c1-4dda-8f14-f920834f5c61","requestId":"92001951-80c1-4dda-8f14-f920834f5c61"}
app={"appName":"rk-demo","appVersion":"master-f414049","entryName":"greeter","entryType":"GrpcEntry"}
env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"*","region":"*"}
payloads={"grpcMethod":"Greeter","grpcService":"api.v1.Greeter","grpcType":"unaryServer","gwMethod":"","gwPath":"","gwScheme":"","gwUserAgent":""}
error={}
counters={}
pairs={"key":"value"}
timing={}
remoteAddr=localhost:58269
operation=/api.v1.Greeter/Greeter
resCode=OK
eventStatus=Ended
EOE
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)