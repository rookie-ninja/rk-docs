---
title: "AppInfo"
linkTitle: "AppInfo"
weight: 6
description: >
  Specify AppInfo in boot.yaml
---

## Quick start
### 1.Install

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-zero
```

### 2.Create boot.yaml
```yaml
app:
  name: my-app                                            # Optional, default: "rk"
  version: "v1.0.0"                                       # Optional, default: "local"
  description: "this is description"                      # Optional, default: ""
  keywords: ["rk", "golang"]                              # Optional, default: []
  homeUrl: "http://example.com"                           # Optional, default: ""
  docsUrl: ["http://example.com"]                         # Optional, default: []
  maintainers: ["rk-dev"]                                 # Optional, default: []
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
	"github.com/rookie-ninja/rk-boot/v2"
	_ "github.com/rookie-ninja/rk-zero/boot"
)

func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Bootstrap
	boot.Bootstrap(context.TODO())

	boot.WaitForShutdownSig(context.TODO())
}
```

### 4.Validate
```bash
$ go run main.go
2022-05-09T12:50:40.167+0800    INFO    boot/zero_entry.go:761  Bootstrap zeroEntry     {"eventId": "4d1830fc-a1e6-4f95-9cdf-87834df2c6ac", "entryName": "greeter", "entryType": "ZeroEntry"}
------------------------------------------------------------------------
endTime=2022-05-09T12:50:40.167215+08:00
startTime=2022-05-09T12:50:40.167023+08:00
elapsedNano=191758
timezone=CST
ids={"eventId":"4d1830fc-a1e6-4f95-9cdf-87834df2c6ac"}
app={"appName":"my-app","appVersion":"v1.0.0","entryName":"greeter","entryType":"ZeroEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.104","os":"darwin"}
payloads={"zeroPort":8080}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)

## Get from CommonService
### 1.Create boot.yaml
```yaml
app:
  name: my-app                                            # Optional, default: "rk"
  version: "v1.0.0"                                       # Optional, default: "local"
  description: "this is description"                      # Optional, default: ""
  keywords: ["rk", "golang"]                              # Optional, default: []
  homeUrl: "http://example.com"                           # Optional, default: ""
  docsUrl: ["http://example.com"]                         # Optional, default: []
  maintainers: ["rk-dev"]                                 # Optional, default: []
zero:
  - name: greeter
    port: 8080
    enabled: true
    commonService:
      enabled: true
```

### 2.Send /rk/v1/info request
```bash
$ curl localhost:8080/rk/v1/info
{
  "appName": "my-app",
  "version": "v1.0.0",
  "description": "this is description",
  "keywords": [
    "rk",
    "golang"
  ],
  "homeUrl": "http://example.com",
  "docsUrl": [
    "http://example.com"
  ],
  "maintainers": [
    "rk-dev"
  ],
  ...
}
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)
