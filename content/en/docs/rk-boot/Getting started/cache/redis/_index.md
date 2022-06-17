---
title: "Redis"
linkTitle: "Redis"
weight: 2
description: >
  Enable redis cache with rk-boot and rk-cache/redis plugin.
---

## Overview
We will use rk-boot & [rk-cache/redis](https://github.com/rookie-ninja/rk-cache) to enable Redis cache.

In order to demonstrate full example，we will use [rk-gin](https://github.com/rookie-ninja/rk-gin/) to start a back end service with APIs as bellow.

- GET /v1/get, get value
- POST /v1/set, set value

## Install

- rk-boot: Base package.
- rk-gin: To start [gin-gonic/gin](https://github.com/gin-gonic/gin) microservice.
- rk-cache/redis: Enable Redis cache.

```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gin/v2
go get github.com/rookie-ninja/rk-cache/redis
```

## Quick start
### 1. Create boot.yaml
```yaml
---
gin:
  - name: cache-service
    port: 8080
    enabled: true
cache:
  - name: redis-cache
    enabled: true
    redis:
      enabled: true
```

### 2. Create main.go

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

### 3.Start Redis locally

```shell
$ docker run -it --rm --name rk-redis -p 6379:6379 redis
```

### 4.Directory hierarchy
```shell script
$ tree
.
├── boot.yaml
├── go.mod
├── go.sum
└── main.go
```

### 5.Start main.go
```go
$ go run main.go

2022-06-17T21:00:05.360+0800    INFO    redis@v1.2.1/entry.go:189       Bootstrap CacheRedisEntry       {"entryName": "redis-cache", "localCache": false, "redisCache": true}
2022-06-17T21:00:05.360+0800    INFO    redis@v1.2.0/boot.go:268        Bootstrap RedisEntry    {"eventId": "e44561a3-6cfd-48f7-b09a-554d5b4afad4", "entryName": "redis-cache", "entryType": "RedisEntry", "clientType": "Single"}
2022-06-17T21:00:05.360+0800    INFO    redis@v1.2.0/boot.go:276        Ping redis at [localhost:6379]
2022-06-17T21:00:05.368+0800    INFO    redis@v1.2.0/boot.go:282        Ping redis at [localhost:6379] success
2022-06-17T21:00:05.368+0800    INFO    boot/gin_entry.go:666   Bootstrap GinEntry      {"eventId": "e44561a3-6cfd-48f7-b09a-554d5b4afad4", "entryName": "cache-service", "entryType": "GinEntry"}
------------------------------------------------------------------------
endTime=2022-06-17T21:00:05.368854+08:00
startTime=2022-06-17T21:00:05.368818+08:00
elapsedNano=35145
timezone=CST
ids={"eventId":"e44561a3-6cfd-48f7-b09a-554d5b4afad4"}
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

### 6.Validate
#### 6.1 Set value

```shell
$ curl "localhost:8080/v1/set?value=my-value"
{"value":"my-value"}
```

#### 6.2 Get value

```shell
$ curl localhost:8080/v1/get
{"value":"my-value"}
```

## Full YAML options
```yaml
cache:
  - name: redis-cache
    enabled: true
    domain: "*"
    description: ""
    redis:
      enabled: true                   # Required
#      addrs: ["localhost:6379"]       # Required, One addr is for single, multiple is for cluster
#      mansterName: ""                 # Optional, required when connecting to Sentinel(HA)
#      sentinelPass: ""                # Optional, default: ""
#      maxRedirects: 3                 # Optional, default: 3
#      readOnly: false                 # Optional, default: false
#      routeByLatency: false           # Optional, default: false
#      routeRandomly: false            # Optional, default: false
#      db: 0                           # Optional, default: 0
#      user: ""                        # Optional, default: ""
#      pass: ""                        # Optional, default: ""
#      maxRetries: 3                   # Optional, default: 3
#      minRetryBackoffMs: 8            # Optional, default: 8
#      maxRetryBackoffMs: 512          # Optional, default: 512
#      dialTimeoutMs: 5000             # Optional, default: 5000 (5 seconds)
#      readTimeoutMs: 3000             # Optional, default: 3000 (3 seconds)
#      writeTimeoutMs: 1               # Optional, default: 3000 (3 seconds)
#      poolFIFO: false                 # Optional, default: false
#      poolSize: 10                    # Optional, default: 10
#      minIdleConn: 0                  # Optional, default: 0
#      maxConnAgeMs: 0                 # Optional, default: no aged connection
#      poolTimeoutMs: 1300             # Optional, default: 1300 (1.3 seconds)
#      idleTimeoutMs: 1                # Optional, default: 5 minutes
#      idleCheckFrequencyMs: 1         # Optional, default: 1 minutes
#      loggerEntry: ""                 # Optional, default: default logger with STDOUT
#    local:
#      enabled: true
#      size: 100                       # Optional, number of elements, default: 10000
#      ttlMin: 10                      # Optional, default: 60
#    loggerEntry: ""                   # Optional, logger entry name
#    certEntry: ""                     # Optional, cert entry name
```