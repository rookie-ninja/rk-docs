---
title: "配置文件"
linkTitle: "配置文件"
weight: 5
description: >
  如是使用启动器，读取本地配置文件？
---

## 概述
启动器使用 [viper](https://github.com/spf13/viper) 来读取本地配置文件。

启动器会通过读取 boot.yaml 文件中 [config] 部分的内容，来初始化 [viper](https://github.com/spf13/viper) 实例。

Viper 支持如下的配置文件格式：
- JSON
- TOML
- YAML
- HCL
- envfile
- Java propertie

## 架构
![](/bootstrapper/user-guide/echo-golang/advanced/config-arch.png)

## Locale
> ConfigEntry 支持 [locale](/docs/bootstrapper/user-guide/go/echo/advanced/locale/).

## 快速开始
- 安装

```shell script
$ go get github.com/rookie-ninja/rk-boot
$ go get github.com/rookie-ninja/rk-echo
```

### 1.创建配置文件
> 创建配置文件，并且把路径添加到 boot.yaml 中。

**config/default.yaml**
```yaml
region: default
```
**boot.yaml**
```yaml
config:
  - name: my-config
    locale: "*::*::*::*"
    path: config/default.yaml
echo:
  - name: greeter
    port: 8080
    enabled: true
```

### 2.读取配置文件
**main.go**
```go
package main

import (
	"context"
	"fmt"
	"github.com/rookie-ninja/rk-boot"
	_ "github.com/rookie-ninja/rk-echo/boot"
	"github.com/rookie-ninja/rk-entry/entry"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Get ConfigEntry and get viper instance
	fmt.Println(rkentry.GlobalAppCtx.GetConfigEntry("my-config").GetViper().GetString("region"))

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

### 3.访问 ConfigEntry 实例
用户可以通过调用 **rkentry.GlobalAppCtx.GetConfigEntry()** 来访问 ConfigEntry。

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)