Connect to ClickHouse with rk-boot and rk-db/clickhouse plugin.

## Overview
We will use rk-boot & [rk-db/clickhouse](https://github.com/rookie-ninja/rk-db) to connect to ClickHouse cluster.
[rk-db/clickhouse](https://github.com/rookie-ninja/rk-db) uses [gorm](https://github.com/go-gorm/gorm) as driver by default.

In order to demonstrate full example，we will use [rk-gin](https://github.com/rookie-ninja/rk-gin/) to start a back end service with APIs as bellow.

- GET /v1/user, List users
- GET /v1/user/:id, Get user
- PUT /v1/user, Create user
- POST /v1/user/:id, Update user
- DELETE /v1/user/:id, Delete user

## Install

- rk-boot: Base package.
- rk-gin: To start [gin-gonic/gin](https://github.com/gin-gonic/gin) microservice.
- rk-db/clickhouse: Plugin to connect to ClickHouse with [gorm](https://github.com/go-gorm/gorm).

```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gin/v2
go get github.com/rookie-ninja/rk-db/clickhouse
```

## Quick start
### 1. Create boot.yaml
```yaml
---
gin:
  - name: user-service
    port: 8080
    enabled: true
clickhouse:
  - name: user-db                          # Required
    enabled: true                          # Required
    domain: "*"                            # Optional
    addr: "localhost:9000"                 # Optional, default: localhost:9000
    user: default                          # Optional, default: default
    pass: ""                               # Optional, default: ""
    database:
      - name: user                         # Required
        autoCreate: true                   # Optional, default: false
#        dryRun: false                     # Optional, default: false
#        params: []                        # Optional, default: []
#    loggerEntry: ""                       # Optional, default: default logger with STDOUT
```

### 2. Create main.go

```go
// Copyright (c) 2021 rookie-ninja
//
// Use of this source code is governed by an Apache-style
// license that can be found in the LICENSE file.
package main

import (
	"context"
	"github.com/gin-gonic/gin"
	"github.com/rookie-ninja/rk-boot/v2"
	"github.com/rookie-ninja/rk-db/clickhouse"
	"github.com/rookie-ninja/rk-gin/v2/boot"
	"github.com/rs/xid"
	"gorm.io/gorm"
	"net/http"
	"time"
)

var userDb *gorm.DB

func main() {
	boot := rkboot.NewBoot()

	boot.Bootstrap(context.TODO())

	// Auto migrate database and init global userDb variable
	clickHouseEntry := rkclickhouse.GetClickHouseEntry("user-db")
	userDb = clickHouseEntry.GetDB("user")
	if !userDb.DryRun {
		userDb.AutoMigrate(&User{})
	}

	// Register APIs
	ginEntry := rkgin.GetGinEntry("user-service")
	ginEntry.Router.GET("/v1/user", ListUsers)
	ginEntry.Router.GET("/v1/user/:id", GetUser)
	ginEntry.Router.PUT("/v1/user", CreateUser)
	ginEntry.Router.POST("/v1/user/:id", UpdateUser)
	ginEntry.Router.DELETE("/v1/user/:id", DeleteUser)

	boot.WaitForShutdownSig(context.TODO())
}

// *************************************
// *************** Model ***************
// *************************************

type Base struct {
	CreatedAt time.Time `yaml:"-" json:"-"`
	UpdatedAt time.Time `yaml:"-" json:"-"`
}

type User struct {
	Base
	Id   string `yaml:"id" json:"id"`
	Name string `yaml:"name" json:"name"`
}

func ListUsers(ctx *gin.Context) {
	userList := make([]*User, 0)
	res := userDb.Find(&userList)

	if res.Error != nil {
		ctx.JSON(http.StatusInternalServerError, res.Error)
		return
	}
	ctx.JSON(http.StatusOK, userList)
}

func GetUser(ctx *gin.Context) {
	uid := ctx.Param("id")
	user := &User{}
	res := userDb.Find(user, "id = ?", uid)

	if res.Error != nil {
		ctx.JSON(http.StatusInternalServerError, res.Error)
		return
	}
	ctx.JSON(http.StatusOK, user)
}

func CreateUser(ctx *gin.Context) {
	user := &User{
		Id:   xid.New().String(),
		Name: ctx.Query("name"),
	}

	res := userDb.Create(user)

	if res.Error != nil {
		ctx.JSON(http.StatusInternalServerError, res.Error)
		return
	}
	ctx.JSON(http.StatusOK, user)
}

func UpdateUser(ctx *gin.Context) {
	uid := ctx.Param("id")
	user := &User{
		Id:   uid,
		Name: ctx.Query("name"),
	}

	res := userDb.Where("id = ?", uid).Updates(user)

	if res.Error != nil {
		ctx.JSON(http.StatusInternalServerError, res.Error)
		return
	}

	ctx.JSON(http.StatusOK, user)
}

func DeleteUser(ctx *gin.Context) {
	uid := ctx.Param("id")

	res := userDb.Delete(&User{}, "id = ?", uid)

	if res.Error != nil {
		ctx.JSON(http.StatusInternalServerError, res.Error)
		return
	}

	ctx.String(http.StatusOK, "success")
}
```

### 3.Start ClickHouse locally

```shell
$ docker run -it --rm --name rk-clickhouse --ulimit nofile=262144:262144 -p 262144:262144 clickhouse/clickhouse-server
```

### 4.Directory hierarchy
```bash
$ tree
.
├── boot.yaml
├── go.mod
├── go.sum
└── main.go
```

### 5.Start main.go
```bash
$ go run main.go

2022-01-07T03:11:18.538+0800    INFO    boot/gin_entry.go:913   Bootstrap ginEntry      {"eventId": "181b17a7-591f-419a-95cc-2cda7efc61f2", "entryName": "user-service"}
------------------------------------------------------------------------
endTime=2022-01-07T03:11:18.53883+08:00
startTime=2022-01-07T03:11:18.538741+08:00
elapsedNano=88391
timezone=CST
ids={"eventId":"181b17a7-591f-419a-95cc-2cda7efc61f2"}
app={"appName":"rk","appVersion":"","entryName":"user-service","entryType":"GinEntry"}
env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.6","os":"darwin","realm":"*","region":"*"}
payloads={"ginPort":8080}
error={}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
2022-01-07T03:11:18.538+0800    INFO    Bootstrap ClickHouse entry      {"entryName": "user-db", "clickHouseUser": "default", "clickHouseAddr": "localhost:9000"}
2022-01-07T03:11:18.538+0800    INFO    creating database user if not exists
2022-01-07T03:11:18.556+0800    INFO    creating successs or database user exists
2022-01-07T03:11:18.556+0800    INFO    connecting to database user
2022-01-07T03:11:18.567+0800    INFO    connecting to database user success
```

### 6.Validate
#### 6.1 Create user
```bash
$ curl -X PUT "localhost:8080/v1/user?name=rk-dev"
{"id":"c7bjufjd0cvqfaenpqjg","name":"rk-dev"}
```

#### 6.2 Update user
```bash
$ curl -X POST "localhost:8080/v1/user/c7bjufjd0cvqfaenpqjg?name=rk-dev-updated"
{"id":"c7bjufjd0cvqfaenpqjg","name":"rk-dev-updated"}
```

#### 6.3 List users
```bash
$ curl -X GET localhost:8080/v1/user
[{"id":"c7bjufjd0cvqfaenpqjg","name":"rk-dev-updated"}]
```

#### 6.4 Get user
```bash
$ curl -X GET localhost:8080/v1/user/c7bjufjd0cvqfaenpqjg
{"id":"c7bjufjd0cvqfaenpqjg","name":"rk-dev-updated"}
```

#### 6.5 Delete user

```bash
$ curl -X DELETE localhost:8080/v1/user/c7bjufjd0cvqfaenpqjg
success
```

## Full YAML options

| name                           | Required | description                        | type     | default value  |
|--------------------------------|----------|------------------------------------|----------|----------------|
| clickhouse.name                | Required | The name of entry                  | string   | ClickHouse     |
| clickhouse.enabled             | Required | Enable entry or not                | bool     | false          |
| clickhouse.domain              | Optional | See locale description bellow      | string   | ""             |
| clickhouse.description         | Optional | Description of echo entry.         | string   | ""             |
| clickhouse.user                | Optional | ClickHouse username                | string   | root           |
| clickhouse.pass                | Optional | ClickHouse password                | string   | pass           |
| clickhouse.addr                | Optional | ClickHouse remote address          | string   | localhost:9000 |
| clickhouse.database.name       | Required | Name of database                   | string   | ""             |
| clickhouse.database.autoCreate | Optional | Create DB if missing               | bool     | false          |
| clickhouse.database.dryRun     | Optional | Run gorm.DB with dry run mode      | bool     | false          |
| clickhouse.database.params     | Optional | Connection params                  | []string | [""]           |
| clickhouse.loggerEntry         | Optional | Reference of zap logger entry name | string   | ""             |

```yaml
clickhouse:
  - name: user-db                          # Required
    enabled: true                          # Required
    domain: "*"                            # Optional
    addr: "localhost:9000"                 # Optional, default: localhost:9000
    user: default                          # Optional, default: default
    pass: ""                               # Optional, default: ""
    database:
      - name: user                         # Required
        autoCreate: true                   # Optional, default: false
#        dryRun: false                     # Optional, default: false
#        params: []                        # Optional, default: []
#    loggerEntry: ""                       # Optional, default: default logger with STDOUT
```

