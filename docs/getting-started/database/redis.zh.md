通过 rk-boot，配合 rk-db/redis 插件，快速连接 Redis。

## 概述
我们将会使用 rk-boot 与 [rk-db/redis](https://github.com/rookie-ninja/rk-db) 插件连接 Redis 集群。
[rk-db/redis](https://github.com/rookie-ninja/rk-db) 插件默认使用了 [go-redis](https://github.com/go-redis/redis/v8)

为了例子的完整性，使用 [rk-gin](https://github.com/rookie-ninja/rk-gin/) 启动一个后台进程，进行验证。

本例子中，我们会创建如下几个 API 来验证与 Redis 的连通性.

- GET /v1/get, get value
- POST /v1/set, set value

## 安装

- rk-boot: rk-boot 基础包
- rk-gin: 用于启动 [gin-gonic/gin](https://github.com/gin-gonic/gin) 微服务
- rk-db/redis: 用于初始化连接 Redis 的 [go-redis](https://github.com/go-redis/redis) Client 实例

```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gin/v2
go get github.com/rookie-ninja/rk-db/redis
```

## 快速开始
### 1. 创建 boot.yaml
```yaml
---
gin:
  - name: server
    enabled: true
    port: 8080
redis:
  - name: redis                      # Required
    enabled: true                    # Required
    addrs: ["localhost:6379"]        # Required, One addr is for single, multiple is for cluster
```

### 2. 创建 main.go
```go
// Copyright (c) 2021 rookie-ninja
//
// Use of this source code is governed by an Apache-style
// license that can be found in the LICENSE file.
package main

import (
	"context"
	"github.com/gin-gonic/gin"
	"github.com/go-redis/redis/v8"
	"github.com/rookie-ninja/rk-boot/v2"
	"github.com/rookie-ninja/rk-db/redis"
	"github.com/rookie-ninja/rk-gin/v2/boot"
	"net/http"
	"time"
)

var redisClient *redis.Client

func main() {
	boot := rkboot.NewBoot()

	boot.Bootstrap(context.TODO())

	// Auto migrate database and init global userDb variable
	redisEntry := rkredis.GetRedisEntry("redis")
	redisClient, _ = redisEntry.GetClient()

	// Register APIs
	ginEntry := rkgin.GetGinEntry("server")
	ginEntry.Router.GET("/v1/get", Get)
	ginEntry.Router.POST("/v1/set", Set)

	boot.WaitForShutdownSig(context.TODO())
}

type KV struct {
	Key   string `json:"key"`
	Value string `json:"value"`
}

func Set(ctx *gin.Context) {
	payload := &KV{}

	if err := ctx.BindJSON(payload); err != nil {
		ctx.JSON(http.StatusInternalServerError, err)
		return
	}

	cmd := redisClient.Set(ctx.Request.Context(), payload.Key, payload.Value, time.Minute)

	if cmd.Err() != nil {
		ctx.JSON(http.StatusInternalServerError, cmd.Err())
		return
	}

	ctx.Status(http.StatusOK)
}

func Get(ctx *gin.Context) {
	key := ctx.Query("key")

	cmd := redisClient.Get(ctx.Request.Context(), key)

	if cmd.Err() != nil {
		if cmd.Err() == redis.Nil {
			ctx.JSON(http.StatusNotFound, "Key not found!")
		} else {
			ctx.JSON(http.StatusInternalServerError, cmd.Err())
		}
		return
	}

	payload := &KV{
		Key:   key,
		Value: cmd.Val(),
	}

	ctx.JSON(http.StatusOK, payload)
}
```

### 3.本地启动 Redis

```bash
$ docker run -it --rm --name rk-redis -p 6379:6379 redis
```

### 4.文件夹结构
```bash
$ tree
.
├── boot.yaml
├── go.mod
├── go.sum
└── main.go
```

### 5.运行 main.go
```bash
$ go run main.go

2022-06-17T19:34:27.176+0800    INFO    redis/boot.go:268       Bootstrap RedisEntry    {"eventId": "8a3bc26c-00b3-4792-9563-d2664cd1ea8e", "entryName": "redis", "entryType": "RedisEntry", "clientType": "Single"}
2022-06-17T19:34:27.176+0800    INFO    redis/boot.go:276       Ping redis at [localhost:6379]
2022-06-17T19:34:27.181+0800    INFO    redis/boot.go:282       Ping redis at [localhost:6379] success
2022-06-17T19:34:27.181+0800    INFO    boot/gin_entry.go:666   Bootstrap GinEntry      {"eventId": "8a3bc26c-00b3-4792-9563-d2664cd1ea8e", "entryName": "server", "entryType": "GinEntry"}
------------------------------------------------------------------------
endTime=2022-06-17T19:34:27.181813+08:00
startTime=2022-06-17T19:34:27.181779+08:00
elapsedNano=33309
timezone=CST
ids={"eventId":"8a3bc26c-00b3-4792-9563-d2664cd1ea8e"}
app={"appName":"rk","appVersion":"local","entryName":"server","entryType":"GinEntry"}
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

### 6.验证
#### 6.1 创建 KV
```bash
$ curl -X POST "localhost:8080/v1/set" -d '{"key":"my-key","value":"my-value"}'
```

#### 6.2 读取 KV
```bash
$ curl -X GET "localhost:8080/v1/get?key=my-key"
{"key":"my-key","value":"my-value"}
```

## 完整 YAML 配置
YAML 配置里的绝大部分都借鉴了 [Option](https://github.com/go-redis/redis/blob/master/options.go)

```yaml
redis:
  - name: redis                      # Required
    enabled: true                    # Required
    addrs: ["localhost:6379"]        # Required, One addr is for single, multiple is for cluster
    domain: "*"                      # Optional
#    description: ""                 # Optional
#
#    # For HA
#    mansterName: ""                 # Optional, required when connecting to Sentinel(HA)
#    sentinelPass: ""                # Optional, default: ""
#
#    # For cluster
#    maxRedirects: 3                 # Optional, default: 3
#    readOnly: false                 # Optional, default: false
#    routeByLatency: false           # Optional, default: false
#    routeRandomly: false            # Optional, default: false
#
#    # Common options
#    db: 0                           # Optional, default: 0
#    user: ""                        # Optional, default: ""
#    pass: ""                        # Optional, default: ""
#    maxRetries: 3                   # Optional, default: 3
#    minRetryBackoffMs: 8            # Optional, default: 8
#    maxRetryBackoffMs: 512          # Optional, default: 512
#    dialTimeoutMs: 5000             # Optional, default: 5000 (5 seconds)
#    readTimeoutMs: 3000             # Optional, default: 3000 (3 seconds)
#    writeTimeoutMs: 1               # Optional, default: 3000 (3 seconds)
#    poolFIFO: false                 # Optional, default: false
#    poolSize: 10                    # Optional, default: 10
#    minIdleConn: 0                  # Optional, default: 0
#    maxConnAgeMs: 0                 # Optional, default: no aged connection
#    poolTimeoutMs: 1300             # Optional, default: 1300 (1.3 seconds)
#    idleTimeoutMs: 1                # Optional, default: 5 minutes
#    idleCheckFrequencyMs: 1         # Optional, default: 1 minutes
#
#    # For logger
#    loggerEntry: ""                 # Optional, default: default logger with STDOUT
```