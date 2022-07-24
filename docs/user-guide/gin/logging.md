Manage logs.
## Concept
rk-boot defines two types of logs.

| Type   | Description                                                                             |
|--------|-----------------------------------------------------------------------------------------|
| Logger | [zap](https://github.com/uber-go/zap) logger                                            |
| Event  | [Event](https://github.com/rookie-ninja/rk-query) , used to record Event |

## Quick start
### 1.Install

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-gin/v2
```

### 2.Define output path
```yaml
---
logger:
  - name: my-logger
    zap:
      outputPaths: ["logs/log.log"]
event:
  - name: my-event
    outputPaths: ["logs/event.log"]
gin:
  - name: greeter
    port: 8080
    enabled: true
    loggerEntry: my-logger
    eventEntry: my-event
```

### 3.Create main.go
```go
package main

import (
  "context"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-entry/v2/entry"
  _ "github.com/rookie-ninja/rk-gin/v2/boot"
  "github.com/rookie-ninja/rk-query"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Try logger
  logger := rkentry.GlobalAppCtx.GetLoggerEntry("my-logger")
  logger.Info("This is my-logger")

  // Try event
  eventEntry := rkentry.GlobalAppCtx.GetEventEntry("my-event")
  event := eventEntry.CreateEvent(rkquery.WithOperation("test"))
  event.AddPair("key", "value")
  event.Finish()

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}
```

### 4.Directory hierarchy
```bash
.
├── boot.yaml
├── go.mod
├── go.sum
├── logs
│   ├── event.log
│   └── log.log
└── main.go
```

### 5.Validate
```bash
$ cat logs/log.log 
2022-04-16T21:45:04.855+0800    INFO    gin/main.go:35  This is my-logger

$ cat logs/event.log 
------------------------------------------------------------------------
endTime=0001-01-01T00:00:00Z
startTime=2022-04-16T21:45:04.855495+08:00
elapsedNano=-9223372036854775808
timezone=CST
ids={"eventId":"eaf7042c-8be3-4be1-8d85-91514a994764"}
app={"appName":"rk","appVersion":"local","entryName":"","entryType":""}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin"}
payloads={}
counters={}
pairs={"key":"value"}
timing={}
remoteAddr=localhost
operation=test
eventStatus=NotStarted
EOE
```

### _**Cheers**_
![](../../img/user-guide/cheers.png)

## YAML options
### Logger
- [zap](https://github.com/uber-go/zap) competitive
- [lumberjack](https://github.com/natefinch/lumberjack) competitive
- [loki](https://grafana.com/oss/loki/): Push logs to [Grafana Loki](https://grafana.com/oss/loki/)

> Most commonly used options

| option                 | description                 |
|------------------------|-----------------------------|
| logger.zap.outputPaths | log output path             |
| logger.zap.encoding    | log format: console or json |

```yaml
logger:
  - name: my-logger                                       # Required
    description: "Description of entry"                   # Optional
    domain: "*"                                           # Optional, default: "*"
    zap:                                                  # Optional
      level: info                                         # Optional, default: info
      development: true                                   # Optional, default: true
      disableCaller: false                                # Optional, default: false
      disableStacktrace: true                             # Optional, default: true
      encoding: console                                   # Optional, default: console
      outputPaths: ["stdout"]                             # Optional, default: [stdout]
      errorOutputPaths: ["stderr"]                        # Optional, default: [stderr]
      encoderConfig:                                      # Optional
        timeKey: "ts"                                     # Optional, default: ts
        levelKey: "level"                                 # Optional, default: level
        nameKey: "logger"                                 # Optional, default: logger
        callerKey: "caller"                               # Optional, default: caller
        messageKey: "msg"                                 # Optional, default: msg
        stacktraceKey: "stacktrace"                       # Optional, default: stacktrace
        skipLineEnding: false                             # Optional, default: false
        lineEnding: "\n"                                  # Optional, default: \n
        consoleSeparator: "\t"                            # Optional, default: \t
      sampling:                                           # Optional, default: nil
        initial: 0                                        # Optional, default: 0
        thereafter: 0                                     # Optional, default: 0
      initialFields:                                      # Optional, default: empty map
        key: value
    lumberjack:                                           # Optional, default: nil
      filename:
      maxsize: 1024                                       # Optional, suggested: 1024 (MB)
      maxage: 7                                           # Optional, suggested: 7 (day)
      maxbackups: 3                                       # Optional, suggested: 3 (day)
      localtime: true                                     # Optional, suggested: true
      compress: true                                      # Optional, suggested: true
    loki:
      enabled: true                                       # Optional, default: false
      addr: localhost:3100                                # Optional, default: localhost:3100
      path: /loki/api/v1/push                             # Optional, default: /loki/api/v1/push
      username: ""                                        # Optional, default: ""
      password: ""                                        # Optional, default: ""
      maxBatchWaitMs: 3000                                # Optional, default: 3000
      maxBatchSize: 1000                                  # Optional, default: 1000
      insecureSkipVerify: false                           # Optional, default: false
      labels:                                             # Optional, default: empty map
        my_label_key: my_label_value
```

### Event
- [lumberjack](https://github.com/natefinch/lumberjack) competitive
- [loki](https://grafana.com/oss/loki/): Push logs to [Grafana Loki](https://grafana.com/oss/loki/)

> Most commonly used options

| option            | description                    |
|-------------------|--------------------------------|
| event.outputPaths | log output path                  |
| event.encoding    | log format: console or json, flatten |

```yaml
event:
  - name: my-event                                        # Required
    description: "Description of entry"                   # Optional
    domain: "*"                                           # Optional, default: "*"
    encoding: console                                     # Optional, default: console
    outputPaths: ["stdout"]                               # Optional, default: [stdout]
    lumberjack:                                           # Optional, default: nil
      filename:
      maxsize: 1024                                       # Optional, suggested: 1024 (MB)
      maxage: 7                                           # Optional, suggested: 7 (day)
      maxbackups: 3                                       # Optional, suggested: 3 (day)
      localtime: true                                     # Optional, suggested: true
      compress: true                                      # Optional, suggested: true
    loki:
      enabled: true                                       # Optional, default: false
      addr: localhost:3100                                # Optional, default: localhost:3100
      path: /loki/api/v1/push                             # Optional, default: /loki/api/v1/push
      username: ""                                        # Optional, default: ""
      password: ""                                        # Optional, default: ""
      maxBatchWaitMs: 3000                                # Optional, default: 3000
      maxBatchSize: 1000                                  # Optional, default: 1000
      insecureSkipVerify: false                           # Optional, default: false
      labels:                                             # Optional, default: empty map
        my_label_key: my_label_value
```