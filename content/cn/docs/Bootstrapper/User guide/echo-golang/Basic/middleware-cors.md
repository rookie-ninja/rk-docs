---
title: "CORS 拦截器"
linkTitle: "CORS 拦截器"
weight: 15
description: >
  启动 CORS 拦截器。
---

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot
```

## 通用选项
> 启动器包含了如下通用选项，这些选项是启动 Echo 服务的必要选项。

| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| echo.name | Echo 服务名称 | string | N/A |
| echo.port | Echo 服务端口 | integer | nil, 服务不会启动 |
| echo.enabled | Echo 服务启动开关 | bool | false |
| echo.description | Echo 服务的描述 | string | "" |

## CORS 选项
| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| echo.interceptors.cors.enabled | 启动 CORS 拦截器 | boolean | false |
| echo.interceptors.cors.allowOrigins | 可通过验证的 Origin 地址。 | []string | * |
| echo.interceptors.cors.allowMethods | 可通过的 http method, 会包含在 OPTIONS 请求的 Header 中。| []string | All http methods |
| echo.interceptors.cors.allowHeaders | 可通过的 http header, 会包含在 OPTIONS 请求的 Header 中。 | []string | Headers from request |
| echo.interceptors.cors.allowCredentials | 会包含在 OPTIONS 请求的 Header 中。 | bool | false |
| echo.interceptors.cors.exposeHeaders | 会包含在 OPTIONS 请求的 Header 中的 Header。 | []string | "" |
| echo.interceptors.cors.maxAge | 会包含在 OPTIONS 请求的 Header 中的 MaxAge。 | int | 0 |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
echo:
  - name: greeter                     # Required
    port: 8080                        # Required
    enabled: true                     # Required
    commonService:
      enabled: true                   # Optional, default: false
    interceptors:
      cors:
        enabled: true                 # Optional, default: false
        # Accept all origins from localhost
        allowOrigins:
          - "http://localhost:*"      # Optional, default: *
```

### 2.创建 main.go
```go
// Copyright (c) 2021 rookie-ninja
//
// Use of this source code is governed by an Apache-style
// license that can be found in the LICENSE file.
package rkdemo

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
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

### 3.创建 cors.html
```html
<!DOCTYPE html>
<html>
<body>

<h1>CORS Test</h1>

<p>Call http://localhost:8080/rk/v1/healthy</p>

<script type="text/javascript">
    window.onload = function() {
        var apiUrl = 'http://localhost:8080/rk/v1/healthy';
        fetch(apiUrl).then(response => response.json()).then(data => {
            document.getElementById("res").innerHTML = data["healthy"]
        }).catch(err => {
            document.getElementById("res").innerHTML = err
        });
    };
</script>

<h4>Response: </h4>
<p id="res"></p>

</body>
</html>
```

### 4.文件夹结构
```shell script
.
├── boot.yaml
├── cors.html
├── go.mod
├── go.sum
└── main.go

0 directories, 5 files
```

### 5.验证
打开 cors.html

![](/bootstrapper/user-guide/echo-golang/basic/cors-success.png)

### 6.被拦截的 CORS
将 echo.interceptors.cors.allowOrigins 设置成 http://localhost:8080。

```yaml
---
echo:
  - name: greeter                     # Required
    port: 8080                        # Required
    enabled: true                     # Required
    commonService:
      enabled: true                   # Optional, default: false
    interceptors:
      cors:
        enabled: true                 # Optional, default: false
        allowOrigins:
          - "http://localhost:8080"   # Optional, default: *
```

![](/bootstrapper/user-guide/echo-golang/basic/cors-fail.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)