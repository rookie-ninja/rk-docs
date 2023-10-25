Enable trace middleware

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-grpc/v2
```

## Options
| options                                       | description                  | type     | default                          |
|-----------------------------------------------|------------------------------|----------|----------------------------------|
| grpc.middleware.trace.enabled                 | Enable tracing middleware    | boolean  | false                            |
| grpc.middleware.trace.ignore                   | Ignore by path               | []string | []                               |
| grpc.middleware.trace.exporter.file.enabled    | Enable file exporter         | boolean  | false                            |
| grpc.middleware.trace.exporter.file.outputPath | Path of file exporter        | string   | stdout                           |
| grpc.middleware.trace.exporter.otlp.enabled    | Enable otlp exporter   | boolean  | false                            |
| grpc.middleware.trace.exporter.otlp.endpoint   | Hostname of OTLP            | string   | localhost                        |

## Quick start
### 1.Create and compile protocol buffer
[Compile protobuf](../buf)

### 2.Create boot.yaml
```yaml
---
grpc:
  - name: greeter
    port: 8080
    enabled: true
    enableRkGwOption: true
    middleware:
      trace:
        enabled: true
        exporter:
          file:
            enabled: true
            outputPath: "stdout"
#        ignore: [""]
#          otlp:
#            enabled: true
#            endpoint: ""
```

### 3.Create main.go
```go
package main

import (
  "context"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-demo/api/gen/v1"
  "github.com/rookie-ninja/rk-grpc/v2/boot"
  "google.golang.org/grpc"
)

func main() {
  boot := rkboot.NewBoot()

  // register grpc
  entry := rkgrpc.GetGrpcEntry("greeter")
  entry.AddRegFuncGrpc(registerGreeter)
  entry.AddRegFuncGw(greeter.RegisterGreeterHandlerFromEndpoint)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  // Wait for shutdown sig
  boot.WaitForShutdownSig(context.TODO())
}

func registerGreeter(server *grpc.Server) {
  greeter.RegisterGreeterServer(server, &GreeterServer{})
}

type GreeterServer struct{}

func (server *GreeterServer) Hello(ctx context.Context, _ *greeter.HelloRequest) (*greeter.HelloResponse, error) {
  return &greeter.HelloResponse{
    Message: "hello!",
  }, nil
}
```

### 4.Validate
> Send request

```bash
$ curl localhost:8080/v1/hello 
{"message":"hello!"}
```

```json
{
  "Name": "/v1/greeter",
  "SpanContext": {
    "TraceID": "507aaae07729a9f7cae7ec25ee7f0ede",
    "SpanID": "ea556bd79f2f2732",
    "TraceFlags": "01",
    "TraceState": "",
    "Remote": false
  },
  "Parent": {
    "TraceID": "00000000000000000000000000000000",
    "SpanID": "0000000000000000",
    "TraceFlags": "00",
    "TraceState": "",
    "Remote": true
  },
  "SpanKind": 2,
  "StartTime": "2022-04-15T23:28:34.054496+08:00",
  "EndTime": "2022-04-15T23:28:34.054594239+08:00",
  "Attributes": [
    ...
    {
      "Key": "http.status_code",
      "Value": {
        "Type": "INT64",
        "Value": 200
      }
    }
  ],
  ...
  "Resource": [
    {
      "Key": "service.entryName",
      "Value": {
        "Type": "STRING",
        "Value": "greeter"
      }
    },
    {
      "Key": "service.entryType",
      "Value": {
        "Type": "STRING",
        "Value": "gRPCEntry"
      }
    },
  ],
  "InstrumentationLibrary": {
    "Name": "greeter",
    "Version": "semver:1.4.0",
    "SchemaURL": ""
  }
}
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 5.Output to file
> There will be a couple of seconds delays

```yaml
---
grpc:
  - name: greeter
    port: 8080
    enabled: true
    enableRkGwOption: true
    middleware:
      trace:
        enabled: true
        exporter:
          file:
            enabled: true
            outputPath: "logs/tracing.log"
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 6.Output to jaeger

> Start [jaeger-all-in-one](https://www.jaegertracing.io/docs/1.50/getting-started/) locally
> ```bash
> $ docker run --rm --name jaeger \
>     -e COLLECTOR_ZIPKIN_HOST_PORT=:9411 \
>     -p 6831:6831/udp \
>     -p 6832:6832/udp \
>     -p 5778:5778 \
>     -p 16686:16686 \
>     -p 4317:4317 \
>     -p 4318:4318 \
>     -p 14250:14250 \
>     -p 14268:14268 \
>     -p 14269:14269 \
>     -p 9411:9411 \
>     jaegertracing/all-in-one:1.50
> ```

```yaml
---
grpc:
  - name: greeter
    port: 8080
    enabled: true
    enableRkGwOption: true
    middleware:
      trace:
        enabled: true
        exporter:
          otpl:
            enabled: true
            endpoint: "localhost:4317"
```

> Jaeger:
>
> [http://localhost:16686/](http://localhost:16686/)

![jaeger](../../../../img/user-guide/gin/basic/gin-trace.png)

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)