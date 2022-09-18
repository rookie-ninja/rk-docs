启动 API Docs UI。

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-grpc/v2
```

## API Docs 选项
| 名字                   | 描述                                       | 类型       | 默认值   |
|----------------------|------------------------------------------|----------|-------|
| grpc.docs.enabled     | 启动 API Docs                              | boolean  | false |
| grpc.docs.path        | API Docs Web 界面路径                        | string   | docs  |
| grpc.docs.specPath    | 本地 Swagger 参数文件（swagger.json）路径          | string   | ""    |
| grpc.docs.headers     | 每次 Swagger 界面请求，都会带着这些头部。格式： [key:value] | []string | []    |
| grpc.docs.style.theme | Web UI Theme，目前只支持 light                 | string   | light |
| grpc.docs.debug       | 开启 Debug 模式，像 Swagger UI 一样向后台发送请求       | bool     | false |

## 快速开始
### 1.创建 boot.yaml
> rk-boot 默认会在 docs/, api/gen/v1/ 文件夹中寻找 swagger JSON 文件。
>
> 如果 swagger JSON 文件在其他的路径，需要在 boot.yaml 文件中指定 **gin.sw.jsonPath**。

```yaml
---
grpc:
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
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot/v2"
	"github.com/rookie-ninja/rk-demo/api/gen/v1"
	"github.com/rookie-ninja/rk-grpc/v2/boot"
	"google.golang.org/grpc"
)

func main() {
	boot := rkboot.NewBoot()

	// register grpc
	entry := rkgrpc.GetGrpcEntry("greeter")
	entry.AddRegFuncGrpc(registerGreeter)
	entry.AddRegFuncGw(greeter.RegisterGreeterHandlerFromEndpoint)

	// Bootstrap
	boot.Bootstrap(context.TODO())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.TODO())
}

func registerGreeter(server *grpc.Server) {
	greeter.RegisterGreeterServer(server, &GreeterServer{})
}

type GreeterServer struct{}

func (server *GreeterServer) Hello(_ context.Context, _ *greeter.HelloRequest) (*greeter.HelloResponse, error) {
	return &greeter.HelloResponse{
		Message: "hello!",
	}, nil
}
```

### 3.启动 main.go
```bash
$ go run main.go
2022-04-17T18:36:57.925+0800    INFO    boot/grpc_entry.go:960  Bootstrap grpcEntry     {"eventId": "1628d013-ad5d-4b8e-9a2f-447404db7157", "entryName": "greeter", "entryType": "gRPCEntry"}
2022-04-17T18:36:57.927+0800    INFO    boot/grpc_entry.go:681  SwaggerEntry: http://localhost:8080/sw/
------------------------------------------------------------------------
endTime=2022-04-17T18:36:57.927808+08:00
startTime=2022-04-17T18:36:57.925577+08:00
elapsedNano=2230586
timezone=CST
ids={"eventId":"1628d013-ad5d-4b8e-9a2f-447404db7157"}
app={"appName":"","appVersion":"","entryName":"greeter","entryType":"gRPCEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.101.5","os":"darwin"}
payloads={"grpcPort":8080,"gwPort":8080,"swEnabled":true,"swPath":"/sw/"}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
```

### 4.验证
> **API Docs:** [http://localhost:8080/sw](http://localhost:8080/docs)

![](../../../img/example/docs.png)

### _**Cheers**_
![](../../../img/user-guide/cheers.png)

## Debug 模式
通过开启 Debug 模式，用户可以像使用 Swagger UI 一样，向后台发送请求。

### 修改 boot.yaml
```yaml
---
grpc:
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

![](../../../img/user-guide/gin/basic/gin-docs.png)

### _**Cheers**_
![](../../../img/user-guide/cheers.png)
