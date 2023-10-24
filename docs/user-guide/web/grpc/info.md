Specify AppInfo in boot.yaml

## Quick start
### 1.Install

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-grpc/v2
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
grpc:
  - name: greeter
    port: 8080
#   gwPort: 8081                  # Optional, default: gateway port will be the same as grpc port if not provided
    enabled: true
```

### 3.Create main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot/v2"
	_ "github.com/rookie-ninja/rk-grpc/v2/boot"
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
2022-04-16T23:38:06.074+0800    INFO    boot/grpc_entry.go:656   Bootstrap GinEntry      {"eventId": "0fc01d4a-1faa-4c43-9055-1917ea6b9134", "entryName": "greeter", "entryType": "GinEntry"}
------------------------------------------------------------------------
endTime=2022-04-16T23:38:06.074485+08:00
startTime=2022-04-16T23:38:06.074359+08:00
elapsedNano=126065
timezone=CST
ids={"eventId":"0fc01d4a-1faa-4c43-9055-1917ea6b9134"}
app={"appName":"my-app","appVersion":"v1.0.0","entryName":"greeter","entryType":"gRPCEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.101.5","os":"darwin"}
payloads={"ginPort":8080}
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
grpc:
  - name: greeter
    port: 8080
    enabled: true
    enableRkGwOption: true
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
