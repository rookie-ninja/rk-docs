rk-boot can distinguish environments with environment variable of DOMAIN

## Overview
There can be multiple environments in real life, and we hope to use different config files in different environments.

rk-boot support multiple ConfigEntry with same name where DOMAIN is used to distinguish environments.

## Concept
**How rk-boot choose entries？**

rk-boot will use environment variable of DOMAIN to distinguish environment

## Quick start
### 1.Install

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-grpc/v2
```

### 2.Create config files
**config/test.yaml**
```yaml
---
value: test
```

**config/prod.yaml**
```yaml
---
value: prod
```

**config/default.yaml**
```yaml
---
value: default
```

### 3.Create boot.yaml
```yaml
config:
  - name: my-config
    domain: "test"
    path: "config/test.yaml"
  - name: my-config
    domain: "prod"
    path: "config/prod.yaml"
  - name: my-config
    domain: "*"
    path: "config/default.yaml"
grpc:
  - name: greeter
    port: 8080
    enabled: true
```

### 4.ENV：DOMAIN=test
```go
package main

import (
  "context"
  "fmt"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-entry/v2/entry"
  "os"
)

func main() {
  os.Setenv("DOMAIN", "test")

  // Create a new boot instance.
  boot := rkboot.NewBoot()

  fmt.Println(rkentry.GlobalAppCtx.GetConfigEntry("my-config").GetString("value"))

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}
```

> Output
```bash
test
```

### 5.No ENV
```go
package main

import (
  "context"
  "fmt"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-entry/v2/entry"
  "os"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  fmt.Println(rkentry.GlobalAppCtx.GetConfigEntry("my-config").GetString("value"))

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}
```
> Output
```bash
default
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)
