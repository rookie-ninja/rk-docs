---
title: "API Docs"
linkTitle: "API Docs"
weight: 4
description: >
  启动 API Docs UI。
---

## 先决条件
我们将使用 [swag](https://github.com/swaggo/swag) 命令行工具来生成 API Docs UI 所需要的 swagger.json 文件。

> 通过 [swag](https://github.com/swaggo/swag) 官网
```shell script
$ go get -u github.com/swaggo/swag/cmd/swag
```

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-zero
```

## API Docs 选项
| 名字                   | 描述                                       | 类型       | 默认值   |
|----------------------|------------------------------------------|----------|-------|
| zero.docs.enabled     | 启动 API Docs                              | boolean  | false |
| zero.docs.path        | API Docs Web 界面路径                        | string   | docs  |
| zero.docs.specPath    | 本地 Swagger 参数文件（swagger.json）路径          | string   | ""    |
| zero.docs.headers     | 每次 Swagger 界面请求，都会带着这些头部。格式： [key:value] | []string | []    |
| zero.docs.style.theme | Web UI Theme，目前只支持 light                 | string   | light |
| zero.docs.debug       | 开启 Debug 模式，像 Swagger UI 一样向后台发送请求       | bool     | false |

## 快速开始
### 1.创建 boot.yaml
> rk-boot 默认会在 docs/, api/gen/v1/ 文件夹中寻找 swagger JSON 文件。
>
> 如果 swagger JSON 文件在其他的路径，需要在 boot.yaml 文件中指定 **echo.sw.jsonPath**。

```yaml
---
zero:
  - name: greeter
    port: 8080
    enabled: true
    docs:
      enabled: true                                        # Optional, default: false
#      path: "docs"                                        # Optional, default: "docs"
#      specPath: ""                                        # Optional
#      headers: ["sw:rk"]                                  # Optional, default: []
#      style:                                              # Optional
#        theme: "light"                                    # Optional, default: "light"
#      debug: false                                        # Optional, default: false
```

### 2.创建 main.go
> 为了能让 swag 命令行生成 Swagger JSON 文件，我们需要在代码中写注释。
>
> 详情可参考 [swag](https://github.com/swaggo/swag) 官方文档。

```go
package main

import (
  "context"
  "encoding/json"
  "fmt"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-zero/boot"
  "github.com/zeromicro/go-zero/rest"
  "net/http"
)

// @title Swagger Example API
// @version 1.0
// @description This is a sample rk-demo server.
// @termsOfService http://swagger.io/terms/

// @securityDefinitions.basic BasicAuth

// @contact.name API Support
// @contact.url http://www.swagger.io/support
// @contact.email support@swagger.io

// @license.name Apache 2.0
// @license.url http://www.apache.org/licenses/LICENSE-2.0.html

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Register handler
  zeroEntry := rkzero.GetZeroEntry("greeter")
  zeroEntry.Server.AddRoute(rest.Route{
    Method:  http.MethodGet,
    Path:    "/v1/greeter",
    Handler: Greeter,
  })

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

// Greeter handler
// @Summary Greeter
// @Id 1
// @Tags Hello
// @version 1.0
// @Param name query string true "name"
// @produce application/json
// @Success 200 {object} GreeterResponse
// @Router /v1/greeter [get]
func Greeter(writer http.ResponseWriter, request *http.Request) {
  writer.WriteHeader(http.StatusOK)
  resp := &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", request.URL.Query().Get("name")),
  }
  bytes, _ := json.Marshal(resp)
  writer.Write(bytes)
}

type GreeterResponse struct {
  Message string
}
```

### 3.生成 swagger 参数文件
```shell script
$ swag init

# swagger.json, swagger.yaml, docs.go files will be generated under ./docs folder.
$ tree
.
├── boot.yaml
├── docs
│   ├── docs.go
│   ├── swagger.json
│   └── swagger.yaml
├── go.mod
├── go.sum
└── main.go

1 directory, 7 files
```

### 4.验证
> **API Docs:** [http://localhost:8080/sw](http://localhost:8080/docs)

![](/rk-boot/example/docs.png)

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)

## Debug 模式
通过开启 Debug 模式，用户可以像使用 Swagger UI 一样，向后台发送请求。

### 修改 boot.yaml
```yaml
---
zero:
  - name: greeter
    port: 8080
    enabled: true
    docs:
      enabled: true                                        # Optional, default: false
      debug: true                                          # Optional, default: false
#      path: "docs"                                        # Optional, default: "docs"
#      specPath: ""                                        # Optional
#      headers: ["sw:rk"]                                  # Optional, default: []
#      style:                                              # Optional
#        theme: "light"                                    # Optional, default: "light"
```

![](/rk-boot/user-guide/gin/basic/gin-docs.png)

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)
