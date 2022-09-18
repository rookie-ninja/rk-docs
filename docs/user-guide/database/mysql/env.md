rk-db/mysql can distinguish environments with environment variable of DOMAIN

## Overview
There can be multiple environments in real life, and we hope to use different config files in different environments.

rk-db/mysql support multiple DB entries with same name where DOMAIN is used to distinguish environments.

## Concept
**How rk-db choose entries？**

rk-db/mysql will use environment variable of DOMAIN to distinguish environment

## Quick start
### 1.Install

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-gin/v2
$ go get github.com/rookie-ninja/rk-db/mysql
```

### 2.Create boot.yaml
```yaml
---
gin:
  - name: user-service
    port: 8080
    enabled: true
mysql:
  - name: demo-db
    enabled: true
    domain: dev                 # ENV: DOMAIN=dev
    addr: "localhost:3306"
    database:
      - name: demo
        autoCreate: true
  - name: demo-db
    enabled: true
    domain: prod                # ENV: DOMAIN=prod
    addr: "remote.host:3306"
    database:
      - name: demo
        autoCreate: true
```

### 3.ENV：DOMAIN=dev
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot/v2"
	_ "github.com/rookie-ninja/rk-db/mysql"
	"os"
)

func main() {
	os.Setenv("DOMAIN", "dev")

	boot := rkboot.NewBoot()

	boot.Bootstrap(context.TODO())

	boot.WaitForShutdownSig(context.TODO())
}
```

> Output
```bash
2022-09-19T00:27:01.273+0800    INFO    mysql/boot.go:378       Bootstrap MySqlEntry    {"eventId": "bb4c7b17-db51-4c58-b611-0543e6689f2d", "entryName": "demo-db", "entryType": "MySqlEntry"}
2022-09-19T00:27:01.273+0800    INFO    mysql/boot.go:497       Creating database [demo]
2022-09-19T00:27:01.286+0800    INFO    mysql/boot.go:519       Creating database [demo] successs
2022-09-19T00:27:01.286+0800    INFO    mysql/boot.go:522       Connecting to database [demo]
2022-09-19T00:27:01.295+0800    INFO    mysql/boot.go:540       Connecting to database [demo] success
```

### 4.No ENV
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot/v2"
	_ "github.com/rookie-ninja/rk-db/mysql"
)

func main() {
	boot := rkboot.NewBoot()

	boot.Bootstrap(context.TODO())

	boot.WaitForShutdownSig(context.TODO())
}
```
> Output
```bash
# no logs which means no connection
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)
