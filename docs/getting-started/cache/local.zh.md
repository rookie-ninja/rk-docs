通过 rk-boot，配合 rk-cache/redis 插件，快速实现本地缓存。

## 概述
我们将会使用 rk-boot 与 [rk-cache/redis](https://github.com/rookie-ninja/rk-cache) 插件实现本地缓存。

为了例子的完整性，使用 [rk-gin](https://github.com/rookie-ninja/rk-gin/) 启动一个后台进程，进行验证。

本例子中，我们会创建如下几个 API 来验证 Cache

- GET /v1/get, get value
- POST /v1/set, set value

## 安装

- rk-boot: rk-boot 基础包
- rk-gin: 用于启动 [gin-gonic/gin](https://github.com/gin-gonic/gin) 微服务
- rk-cache/redis: 初始化本地缓存

```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gin/v2
go get github.com/rookie-ninja/rk-cache/redis
```

## 快速开始
### 1. 创建 boot.yaml
```yaml
---
gin:
  - name: cache-service
    port: 8080
    enabled: true
cache:
  - name: redis-cache
    enabled: true
    local:
      enabled: true
```

### 2. 创建 main.go

```go
package main

import (
	"context"
	"github.com/gin-gonic/gin"
	"github.com/rookie-ninja/rk-boot/v2"
	"github.com/rookie-ninja/rk-cache/redis"
	"github.com/rookie-ninja/rk-gin/v2/boot"
	"net/http"
)

var cacheEntry *rkcache.CacheEntry

func main() {
	boot := rkboot.NewBoot()

	boot.Bootstrap(context.TODO())

	// assign cache
	cacheEntry = rkcache.GetCacheEntry("redis-cache")

	// assign router
	ginEntry := rkgin.GetGinEntry("cache-service")
	ginEntry.Router.GET("/v1/get", Get)
	ginEntry.Router.GET("/v1/set", Set)

	boot.WaitForShutdownSig(context.TODO())
}

func Get(ctx *gin.Context) {
	val := ""
	resp := cacheEntry.GetFromCache(&rkcache.CacheReq{
		Key:   "demo-key",
		Value: &val,
	})

	if resp.Error != nil || !resp.Success {
		ctx.JSON(http.StatusInternalServerError, resp.Error)
		return
	}

	ctx.JSON(http.StatusOK, map[string]string{
		"value": val,
	})
}

func Set(ctx *gin.Context) {
	val, ok := ctx.GetQuery("value")
	if !ok {
		ctx.JSON(http.StatusBadRequest, "No value found")
	}

	cacheEntry.AddToCache(&rkcache.CacheReq{
		Key:   "demo-key",
		Value: val,
	})

	ctx.JSON(http.StatusOK, map[string]string{
		"value": val,
	})
}
```

### 3.文件夹结构
```bash
$ tree
.
├── boot.yaml
├── go.mod
├── go.sum
└── main.go
```

### 4.运行 main.go
```bash
$ go run main.go

2022-06-17T20:41:22.660+0800    INFO    redis@v1.2.1/entry.go:189       Bootstrap CacheRedisEntry       {"entryName": "redis-cache", "localCache": true, "redisCache": false}
2022-06-17T20:41:22.660+0800    INFO    boot/gin_entry.go:666   Bootstrap GinEntry      {"eventId": "85880c56-5669-4862-aa79-02a64f7d3908", "entryName": "cache-service", "entryType": "GinEntry"}
------------------------------------------------------------------------
endTime=2022-06-17T20:41:22.660751+08:00
startTime=2022-06-17T20:41:22.660721+08:00
elapsedNano=30540
timezone=CST
ids={"eventId":"85880c56-5669-4862-aa79-02a64f7d3908"}
app={"appName":"rk","appVersion":"local","entryName":"cache-service","entryType":"GinEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin"}
payloads={"ginPort":8080}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
```

### 5.验证
#### 5.1 Set value

```bash
$ curl "localhost:8080/v1/set?value=my-value"
{"value":"my-value"}
```

#### 5.2 Get value

```bash
$ curl localhost:8080/v1/get
{"value":"my-value"}
```

## 完整 YAML 配置
```yaml
cache:
  - name: redis-cache
    enabled: true
    domain: "*"
    description: ""
    local:
      enabled: true
      size: 100           # Optional, number of elements, default: 10000
      ttlMin: 10          # Optional, default: 60
    loggerEntry: ""       # Optional, logger entry name
    certEntry: ""         # Optional, cert entry name
```