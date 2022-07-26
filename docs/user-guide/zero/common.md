Enable Common API。

## Overview
Common service contains builtin API commonly used by user.

| Path         | Description           |
|--------------|-----------------------|
| /rk/v1/alive | K8S liveness。         |
| /rk/v1/ready | K8S readiness。        |
| /rk/v1/info  | Get Application Info。 |
| /rk/v1/gc    | Trigger GC。           |

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-zero
```

## CommonService options
| name                          | description            | type    | default value |
|-------------------------------|------------------------|---------|---------------|
| zero.commonService.enabled    | Enable Common API      | boolean | false         |
| zero.commonService.pathPrefix | Common API path prefix | string  | /rk/v1        |

## Quick start
### 1.Create boot.yaml
```yaml
---
zero:
  - name: greeter
    port: 8080
    enabled: true
    commonService:
      enabled: true
#      pathPrefix: ""
```

### 2.Create main.go
```go
package main

import (
	"context"
    "github.com/rookie-ninja/rk-boot/v2"
    _ "github.com/rookie-ninja/rk-zero/boot"
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
```bash
$ curl localhost:8080/rk/v1/ready
{
  "ready": true
}
```

### _**Cheers**_
![](../../img/user-guide/cheers.png)

### 4.Enable swagger UI
```yaml
---
zero:
  - name: greeter
    port: 8080
    enabled: true
    commonService:
      enabled: true
    sw:
      enabled: true
```

> Validate
>
> [http://localhost:8080/sw](http://localhost:8080/sw)

![sw-common](../../img/user-guide/gin/basic/gin-sw-common.png)

### _**Cheers**_
![](../../img/user-guide/cheers.png)

