rk-boot 会可以根据 DOMAIN 环境变量区分不同的环境。

## 概述
让我们考虑如下的场景，我们需要在【test】和【prod】两个环境的服务器中，使用不同的配置文件，比如数据库配置文件等等。

因为 rk-boot 支持多个 ConfigEntry，我们需要一个机制来分辨不同的 ConfigEntry.

> 注意，为了代码的整洁性，不同的 ConfigEntry 会用同样的名字，这样一来，对于代码来说，才可以做到与环境无关。

## 概念
**Entry 是如何被 rk-boot 筛选？**

rk-boot 会使用 DOMAIN 环境变量来区分不同的环境。这也是 RK 推荐的云原生环境分辨法。

## 快速开始
### 1.安装

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-gin/v2
```

### 2.创建配置文件
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

### 3.创建 boot.yaml
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
gin:
  - name: greeter
    port: 8080
    enabled: true
```

### 4.环境变量：DOMAIN=test
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

> 输出
```bash
test
```

### 5.无环境变量
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
> 输出
```bash
default
```

### _**Cheers**_
![](../../img/user-guide/cheers.png)
