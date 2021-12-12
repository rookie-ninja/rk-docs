---
title: "环境区分"
linkTitle: "环境区分"
weight: 2
description: >
  启动器会可以根据 Locale 信息来根据不同的环境变量，分辨参数。
---

## 概述
让我们考虑如下的场景，我们需要在【新加坡】和【法兰克福】两个地域的服务器中，使用不同的配置文件，比如数据库配置文件等等。

因为启动器支持多个 ConfigEntry，我们需要一个机制来分辨不同的 ConfigEntry.

> 注意，为了代码的整洁性，不同的 ConfigEntry 会用同样的名字，这样一来，对于代码来说，才可以做到与环境无关。

## 概念
> **Entry 是如何被启动器筛选？**
> 
> 启动器会使用 REALM，REGION，AZ，DOMAIN 环境变量来区分不同的环境。这也是 RK 推荐的云原生环境分辨法。

![](/bootstrapper/user-guide/gin-golang/advanced/locale-arch.png)

## 快速开始
- 安装

```shell script
$ go get github.com/rookie-ninja/rk-boot
$ go get github.com/rookie-ninja/rk-gin
```

### 1.创建配置文件
**config/singapore.yaml**
```yaml
---
region: sinpapore
```
**config/frankfurt.yaml**
```yaml
---
region: frankfurt
```
**config/default.yaml**
```yaml
---
region: default
```

### 2.创建 boot.yaml
```yaml
config:
  # Load this config if nothing matched
  - name: my-config
    locale: "*::*::*::*"
    path: config/default.yaml
  # Load this config if REGION=singapore
  - name: my-config
    locale: "*::singapore::*::*"
    path: config/singapore.yaml
  # Load this config if REGION=frankfurt
  - name: my-config
    locale: "*::frankfurt::*::*"
    path: config/frankfurt.yaml
gin:
  - name: greeter
    port: 8080
    enabled: true
```

### 3.环境变量：REGION=singapore
```go
package main

import (
	"context"
	"fmt"
	"github.com/rookie-ninja/rk-boot"
	_ "github.com/rookie-ninja/rk-gin/boot"
	"os"
)

// Application entrance.
func main() {
    // Set REGION=singapore
	os.Setenv("REGION", "singapore")

	// Create a new boot instance.
	boot := rkboot.NewBoot()

	fmt.Println(boot.GetConfigEntry("my-config").GetViper().GetString("region"))

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```
> 输出
```shell script
singapore
```

### 4.环境变量：REGION=frankfurt
```go
package main

import (
	"context"
	"fmt"
	"github.com/rookie-ninja/rk-boot"
	_ "github.com/rookie-ninja/rk-gin/boot"
	"os"
)

// Application entrance.
func main() {
    // Set REGION=singapore
	os.Setenv("REGION", "frankfurt")

	// Create a new boot instance.
	boot := rkboot.NewBoot()

	fmt.Println(boot.GetConfigEntry("my-config").GetViper().GetString("region"))

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

> 输出
```shell script
frankfurt
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 5.环境变量：REGION=not-matched
```go
package main

import (
	"context"
	"fmt"
	"github.com/rookie-ninja/rk-boot"
	_ "github.com/rookie-ninja/rk-gin/boot"
	"os"
)

// Application entrance.
func main() {
    // Set REGION=singapore
	os.Setenv("REGION", "not-matched")

	// Create a new boot instance.
	boot := rkboot.NewBoot()

	fmt.Println(boot.GetConfigEntry("my-config").GetViper().GetString("region"))

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```
> 输出
```shell script
default
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 6.环境变量：REGION=""
```go
package main

import (
	"context"
	"fmt"
	"github.com/rookie-ninja/rk-boot"
	_ "github.com/rookie-ninja/rk-gin/boot"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	fmt.Println(boot.GetConfigEntry("my-config").GetViper().GetString("region"))

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

> 输出
```shell script
default
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)