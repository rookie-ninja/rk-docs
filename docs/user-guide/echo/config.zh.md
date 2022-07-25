如是使用 rk-boot，读取本地配置文件？

## 概述
rk-boot 使用 [viper](https://github.com/spf13/viper) 来读取本地配置文件。

rk-boot 会通过读取 boot.yaml 文件中 [config] 部分的内容，来初始化 [viper](https://github.com/spf13/viper) 实例。

Viper 支持如下的配置文件格式：

- JSON
- TOML
- YAML
- HCL
- envfile
- Java propertie

## DOMAIN
ConfigEntry 支持通过 DOMAIN 区分环境

## 快速开始
### 1.安装

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-echo
```

### 2.创建 config/default.yaml
```yaml
---
value: default
```

### 3.创建 boot.yaml
```yaml
---
config:
  - name: my-config
    path: "config/default.yaml"
echo:
  - name: greeter
    port: 8080
    enabled: true
```

### 4.创建 main.go
```go
package main

import (
  "context"
  "fmt"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-entry/v2/entry"
  _ "github.com/rookie-ninja/rk-echo/boot"
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

### 5.验证
```bash
$ go run main.go
default
...
```

### _**Cheers**_
![](../../img/user-guide/cheers.png)

## 在 boot.yaml 中嵌入 Config 值
除了创建 Config 文件，我们还可以直接在 boot.yaml 定义 Config 值。

### 1.创建 boot.yaml
```yaml
config:
  - name: my-config
    content:
      value: embed-value
echo:
  - name: greeter
    port: 8080
    enabled: true
```

### 2.验证
```bash
$ go run main.go
embed-value
...
```

### _**Cheers**_
![](../../img/user-guide/cheers.png)

## 使用环境变量覆盖
我们可以使用环境变量，覆盖 Config 的值。

### 1.创建 boot.yaml
```yaml
config:
  - name: my-config
    envPrefix: "demo"
    path: "config/default.yaml"
echo:
  - name: greeter
    port: 8080
    enabled: true
```

### 2.通过环境变量覆盖
```go
package main

import (
	"context"
	"fmt"
	"github.com/rookie-ninja/rk-boot/v2"
	"github.com/rookie-ninja/rk-entry/v2/entry"
	_ "github.com/rookie-ninja/rk-echo/boot"
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

### 3.验证
```bash
$ go run main.go
override-value
...
```

### _**Cheers**_
![](../../img/user-guide/cheers.png)
