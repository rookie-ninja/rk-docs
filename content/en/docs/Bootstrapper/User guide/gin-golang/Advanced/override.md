---
title: "Override bootstrapper"
linkTitle: "Override bootstrapper"
weight: 9
description: >
  Is there any way to override boot.yaml or values in boot.yaml at start time?
---

## Overview
Bootstrapper support two kinds of ways to override bootstrapper configs.
- Override config file (by \-\-rkboot)
- Override values in config file (by \-\-rkset)

## Quick start
- Install

```shell script
$ go get github.com/rookie-ninja/rk-boot
$ go get github.com/rookie-ninja/rk-gin
```

### 1.Override bootstrapper config file
In order to override bootstrapper file path, **\-\-rkboot** needs to be passed.

> boot.yaml
```yaml
---
gin:
  - name: greeter
    port: 8080
    enabled: true
    commonService:
      enabled: true
```

> boot-override.yaml
```yaml
---
gin:
  - name: greeter
    port: 8081
    enabled: true
    commonService:
      enabled: true
```

> main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
	_ "github.com/rookie-ninja/rk-gin/boot"
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

> Start server with \-\-rkboot args
```shell script
$ go build main.go
$ ./main --rkboot boot-override.yaml
```

> Send request to 8080
```shell script
$ curl localhost:8080/rk/v1/healthy
curl: (7) Failed to connect to localhost port 8080: Connection refused
```

> Send request to 8081
```shell script
$ curl localhost:8081/rk/v1/healthy
{"healthy":true}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 2.Override values in boot.yaml
In order to override bootstrapper file path, **\-\-rkset** needs to be passed.

Use **comma** to separate multiple overrides.

| Type | Example |
| ---- | ---- |
| Map | app.description="This is description" |
| List | gin[0].name="alice",gin[0].port=8081 |

> boot.yaml
```yaml
---
gin:
  - name: greeter
    port: 8080
    enabled: true
    commonService:
      enabled: true
```

> main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
	_ "github.com/rookie-ninja/rk-gin/boot"
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

> Start server with \-\-rkset args
```shell script
$ go build main.go
$ ./main --rkset "gin[0].port=8081"
```

> Send request to 8080
```shell script
$ curl localhost:8080/rk/v1/healthy
curl: (7) Failed to connect to localhost port 8080: Connection refused
```

> Send request to 8081
```shell script
$ curl localhost:8081/rk/v1/healthy
{"healthy":true}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)