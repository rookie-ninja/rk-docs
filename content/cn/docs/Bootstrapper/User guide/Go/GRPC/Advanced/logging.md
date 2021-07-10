---
title: "Logging"
linkTitle: "Logging"
weight: 3
description: >
  Customise logging.
---

## Architecture
![](/bootstrapper/user-guide/go/grpc/advanced/grpc-logger-arch.png)

## ZapLoggerEntry
[ZapLoggerEntry](https://github.com/rookie-ninja/rk-entry#zaploggerentry) is used for initializing zap logger.

```go
// ZapLoggerEntry contains bellow fields.
// 1: EntryName: Name of entry.
// 2: EntryType: Type of entry which is ZapLoggerEntryType.
// 3: EntryDescription: Description of ZapLoggerEntry.
// 4: Logger: zap.Logger which was initialized at the beginning.
// 5: LoggerConfig: zap.Logger config which was initialized at the beginning which is not accessible after initialization..
// 6: LumberjackConfig: lumberjack.Logger which was initialized at the beginning.
type ZapLoggerEntry struct {
	EntryName        string             `yaml:"entryName" json:"entryName"`
	EntryType        string             `yaml:"entryType" json:"entryType"`
	EntryDescription string             `yaml:"entryDescription" json:"entryDescription"`
	Logger           *zap.Logger        `yaml:"-" json:"-"`
	LoggerConfig     *zap.Config        `yaml:"zapConfig" json:"zapConfig"`
	LumberjackConfig *lumberjack.Logger `yaml:"lumberjackConfig" json:"lumberjackConfig"`
}
```

### YAML
> ZapLoggerEntry follows zap and lumberjack YAML hierarchy.
>
> Please refer to [zap](https://pkg.go.dev/go.uber.org/zap#section-documentation) and [lumberjack](https://github.com/natefinch/lumberjack) site for details.

```yaml
---
zapLogger:
  - name: zap-logger                      # Required
    description: "Description of entry"   # Optional
    zap:
      level: info                         # Optional, default: info, options: [debug, DEBUG, info, INFO, warn, WARN, dpanic, DPANIC, panic, PANIC, fatal, FATAL]
      development: true                   # Optional, default: true
      disableCaller: false                # Optional, default: false
      disableStacktrace: true             # Optional, default: true
      sampling:                           # Optional, default: empty map
        initial: 0
        thereafter: 0
      encoding: console                   # Optional, default: "console", options: [console, json]
      encoderConfig:
        messageKey: "msg"                 # Optional, default: "msg"
        levelKey: "level"                 # Optional, default: "level"
        timeKey: "ts"                     # Optional, default: "ts"
        nameKey: "logger"                 # Optional, default: "logger"
        callerKey: "caller"               # Optional, default: "caller"
        functionKey: ""                   # Optional, default: ""
        stacktraceKey: "stacktrace"       # Optional, default: "stacktrace"
        lineEnding: "\n"                  # Optional, default: "\n"
        levelEncoder: "capitalColor"      # Optional, default: "capitalColor", options: [capital, capitalColor, color, lowercase]
        timeEncoder: "iso8601"            # Optional, default: "iso8601", options: [rfc3339nano, RFC3339Nano, rfc3339, RFC3339, iso8601, ISO8601, millis, nanos]
        durationEncoder: "string"         # Optional, default: "string", options: [string, nanos, ms]
        callerEncoder: ""                 # Optional, default: ""
        nameEncoder: ""                   # Optional, default: ""
        consoleSeparator: ""              # Optional, default: ""
      outputPaths: [ "stdout" ]           # Optional, default: ["stdout"], stdout would be replaced if specified
      errorOutputPaths: [ "stderr" ]      # Optional, default: ["stderr"], stderr would be replaced if specified
      initialFields:                      # Optional, default: empty map
        key: "value"
    lumberjack:                           # Optional
      filename: "rkapp-event.log"         # Optional, default: It uses <processname>-lumberjack.log in os.TempDir() if empty.
      maxsize: 1024                       # Optional, default: 1024 (MB)
      maxage: 7                           # Optional, default: 7 (days)
      maxbackups: 3                       # Optional, default: 3 (days)
      localtime: true                     # Optional, default: true
      compress: true                      # Optional, default: true
```

### Access
```go
// Access entry
rkentry.GlobalAppCtx.GetZapLoggerEntry("zap-logger")

// Access zap logger
rkentry.GlobalAppCtx.GetZapLoggerEntry("zap-logger").GetLogger()

// Access zap logger config
rkentry.GlobalAppCtx.GetZapLoggerEntry("zap-logger").GetLoggerConfig()

// Access lumberjack config
rkentry.GlobalAppCtx.GetZapLoggerEntry("zap-logger").GetLumberjackConfig()
```

## EventLoggerEntry
RK bootstrapper treat RPC request as an **Event**, and record every RPC request into Event type in [rk-query](https://github.com/rookie-ninja/rk-query).

```go
// EventLoggerEntry contains bellow fields.
// 1: EntryName: Name of entry.
// 2: EntryType: Type of entry which is EventLoggerEntryType.
// 3: EntryDescription: Description of EventLoggerEntry.
// 4: EventFactory: rkquery.EventFactory was initialized at the beginning.
// 5: EventHelper: rkquery.EventHelper was initialized at the beginning.
// 6: LoggerConfig: zap.Config which was initialized at the beginning which is not accessible after initialization.
// 7: LumberjackConfig: lumberjack.Logger which was initialized at the beginning.
type EventLoggerEntry struct {
	EntryName        string                `yaml:"entryName" json:"entryName"`
	EntryType        string                `yaml:"entryType" json:"entryType"`
	EntryDescription string                `yaml:"entryDescription" json:"entryDescription"`
	EventFactory     *rkquery.EventFactory `yaml:"-" json:"-"`
	EventHelper      *rkquery.EventHelper  `yaml:"-" json:"-"`
	LoggerConfig     *zap.Config           `yaml:"zapConfig" json:"zapConfig"`
	LumberjackConfig *lumberjack.Logger    `yaml:"lumberjackConfig" json:"lumberjackConfig"`
}
```

### Fields
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
> endTime=2021-07-10T03:00:12.153392+08:00
> startTime=2021-07-10T03:00:12.153261+08:00
> elapsedNano=130727
> timezone=CST
> ids={"eventId":"c9a1f6b0-b9ec-4e46-9ed4-238c3c6759ab","requestId":"c9a1f6b0-b9ec-4e46-9ed4-238c3c6759ab","traceId":"5441ff5c3855f03b573e95d81139123b"}
> app={"appName":"rk-demo","appVersion":"master-f414049","entryName":"greeter","entryType":"GrpcEntry"}
> env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"*","region":"*"}
> payloads={"grpcMethod":"Greeter","grpcService":"api.v1.Greeter","grpcType":"unaryServer","gwMethod":"GET","gwPath":"/v1/greeter","gwScheme":"http","gwUserAgent":"curl/7.64.1"}
> error={}
> counters={}
> pairs={}
> timing={}
> remoteAddr=localhost:59631
> operation=/api.v1.Greeter/Greeter
> resCode=OK
> eventStatus=Ended
> EOE
> ```


### YAML
EventLoggerEntry needs application name while creating event log. 
As a result, it is recommended to add AppInfoEntry while initializing event logger entry. 
Otherwise, default application name would be assigned.

```yaml
---
eventLogger:
  - name: event-logger                 # Required
    description: "This is description" # Optional
    encoding: console                  # Optional, default: console, options: console and json
    outputPaths: ["stdout"]            # Optional
    lumberjack:                        # Optional
      filename: "rkapp-event.log"      # Optional, default: It uses <processname>-lumberjack.log in os.TempDir() if empty.
      maxsize: 1024                    # Optional, default: 1024 (MB)
      maxage: 7                        # Optional, default: 7 (days)
      maxbackups: 3                    # Optional, default: 3 (days)
      localtime: true                  # Optional, default: true
      compress: true                   # Optional, default: true
```

### Access
```go
// Access entry
rkentry.GlobalAppCtx.GetEventLoggerEntry("event-logger")

// Access event factory
rkentry.GlobalAppCtx.GetEventLoggerEntry("event-logger").GetEventFactory()

// Access event helper
rkentry.GlobalAppCtx.GetEventLoggerEntry("event-logger").GetEventHelper()

// Access lumberjack config
rkentry.GlobalAppCtx.GetEventLoggerEntry("event-logger").GetLumberjackConfig()
```

## Quick start
### 1.Custom ZapLoggerEntry
```yaml
---
zapLogger:
  - name: zap-logger                      # Required
    description: "Description of entry"   # Optional
    zap:
      encoding: json
grpc:
  - name: greeter
    port: 1949
```

```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
	"github.com/rookie-ninja/rk-entry/entry"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()
    
    // Get custom ZapLoggerEntry
	rkentry.GlobalAppCtx.GetZapLoggerEntry("zap-logger").GetLogger().Info("Custom zap logger")

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

```shell script
{"level":"INFO","ts":"2021-07-06T05:06:16.252+0800","msg":"Custom zap logger"}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 2.Reference ZapLoggerEntry
Bootstrapper will use this logger by default including interceptor.

```yaml
---
zapLogger:
  - name: zap-logger
grpc:
  - name: greeter
    port: 1949
    logger:
     zapLogger:
        ref: zap-logger
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 3.Custom EventLoggerEntry
```yaml
---
eventLogger:
  - name: event-logger
grpc:
  - name: greeter
    port: 1949
```

```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
	"github.com/rookie-ninja/rk-entry/entry"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()
    
    // Get custom EventLoggerEntry
	helper := rkentry.GlobalAppCtx.GetEventLoggerEntry("event-logger").GetEventHelper()
	event := helper.Start("demo")
	event.AddPair("key", "value")
	helper.Finish(event)

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

```shell script
------------------------------------------------------------------------
endTime=2021-07-10T03:02:58.540804+08:00
startTime=2021-07-10T03:02:58.540803+08:00
elapsedNano=568
timezone=CST
ids={"eventId":"55f9dd94-9bb8-42a0-b3ed-9164d5f00a64"}
app={"appName":"rk-demo","appVersion":"master-f414049","entryName":"","entryType":""}
env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"*","region":"*"}
payloads={}
error={}
counters={}
pairs={"key":"value"}
timing={}
remoteAddr=localhost
operation=demo
resCode=OK
eventStatus=Ended
EOE
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 4.Reference EventLoggerEntry
Bootstrapper will use this logger by default including interceptor.

```yaml
---
eventLogger:
  - name: event-logger
grpc:
  - name: greeter
    port: 1949
    logger:
      eventLogger:
        ref: event-logger
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 5.Set env at EventLoggerEntry
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
	"github.com/rookie-ninja/rk-entry/entry"
)

// Application entrance.
func main() {
    // Set env
	os.Setenv("REALM", "my-realm")
	os.Setenv("REGION", "my-region")
	os.Setenv("AZ", "my-az")
	os.Setenv("DOMAIN", "my-domain")

	// Create a new boot instance.
	boot := rkboot.NewBoot()
    
    // Get custom EventLoggerEntry
	helper := rkentry.GlobalAppCtx.GetEventLoggerEntry("event-logger").GetEventHelper()
	event := helper.Start("demo")
	helper.Finish(event)

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

```shell script
------------------------------------------------------------------------
...
env={"arch":"amd64","az":"my-az","domain":"my-domain","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"my-realm","region":"my-region"}
...
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 6.Access ZapLoggerEntry & EventLoggerEntry
User can access by calling **rkentry.GlobalAppCtx.GetEventLoggerEntry()** & **rkentry.GlobalAppCtx.GetZapLoggerEntry()**.