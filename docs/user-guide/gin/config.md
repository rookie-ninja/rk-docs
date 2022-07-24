Read config from local file system.

## Overview
rk-boot uses [viper](https://github.com/spf13/viper) as config reader.

Bellow file types are supported by Viper:
- JSON
- TOML
- YAML
- HCL
- envfile
- Java propertie

## DOMAIN
Environment variable of DOMAIN can be used to distinguish environment. 

## Quick start
### 1.Install

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-gin/v2
```

### 2.Create config/default.yaml
```yaml
---
value: default
```

### 3.Create boot.yaml
```yaml
---
config:
  - name: my-config
    path: "config/default.yaml"
gin:
  - name: greeter
    port: 8080
    enabled: true
```

### 4.Create main.go
```go
package main

import (
  "context"
  "fmt"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-entry/v2/entry"
  _ "github.com/rookie-ninja/rk-gin/v2/boot"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Read config
  fmt.Println(rkentry.GlobalAppCtx.GetConfigEntry("my-config").GetString("value"))

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}
```

### 5.Validate
```bash
$ go run main.go
default
...
```

### _**Cheers**_
![](../../img/user-guide/cheers.png)

## Set config values in boot.yaml
### 1.Create boot.yaml
```yaml
config:
  - name: my-config
    content:
      value: embed-value
gin:
  - name: greeter
    port: 8080
    enabled: true
```

### 2.Validate
```bash
$ go run main.go
embed-value
...
```

### _**Cheers**_
![](../../img/user-guide/cheers.png)

## Override with environment variable
### 1.Create boot.yaml
```yaml
config:
  - name: my-config
    envPrefix: "demo"
    path: "config/default.yaml"
gin:
  - name: greeter
    port: 8080
    enabled: true
```

### 2.Override
```go
package main

import (
	"context"
	"fmt"
	"github.com/rookie-ninja/rk-boot/v2"
	"github.com/rookie-ninja/rk-entry/v2/entry"
	_ "github.com/rookie-ninja/rk-gin/v2/boot"
	"os"
)

func main() {
	// Use upper case env prefix
	os.Setenv("DEMO_VALUE", "override-value")

	// Create a new boot instance.
	boot := rkboot.NewBoot()

	fmt.Println(rkentry.GlobalAppCtx.GetConfigEntry("my-config").GetString("value"))

	// Bootstrap
	boot.Bootstrap(context.TODO())

	boot.WaitForShutdownSig(context.TODO())
}
```

### 3.Validate
```bash
$ go run main.go
override-value
...
```

### _**Cheers**_
![](../../img/user-guide/cheers.png)
