---
title: "Sqlite"
linkTitle: "Sqlite"
weight: 5
description: >
  通过 rk-boot，配合 rk-db/sqlite 插件，快速连接 Sqlite。
---

## 概述
我们将会使用 rk-boot 与 [rk-db/sqlite](https://github.com/rookie-ninja/rk-db) 插件连接 Sqlite 集群。
[rk-db/sqlite](https://github.com/rookie-ninja/rk-db) 插件默认使用了 [gorm](https://github.com/go-gorm/gorm)

rk-boot 会根据 boot.yaml 里的配置，自动创建 gorm.DB 实例，并且创建与 Sqlite 之间的连接。

为了例子的完整性，使用 [rk-gin](https://github.com/rookie-ninja/rk-gin/) 启动一个后台进程，进行验证。

本例子中，我们会创建如下几个 API 来验证与 Sqlite 的连通性.

- GET /v1/user, List users
- GET /v1/user/:id, Get user
- PUT /v1/user, Create user
- POST /v1/user/:id, Update user
- DELETE /v1/user/:id, Delete user

## 安装

- rk-boot: rk-boot 基础包
- rk-gin: 用于启动 [gin-gonic/gin](https://github.com/gin-gonic/gin) 微服务
- rk-db/sqlite: 用于初始化连接 Sqlite 的 [gorm](https://github.com/go-gorm/gorm) 实例

```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gin/v2
go get github.com/rookie-ninja/rk-db/sqlite
```

## 快速开始
### 1. 创建 boot.yaml
```yaml
---
gin:
  - name: user-service
    port: 8080
    enabled: true
sqlite:
  - name: user-db                     # Required
    enabled: true                     # Required
    domain: "*"                       # Optional
    database:
      - name: user                    # Required
#        inMemory: true               # Optional, default: false
#        dbDir: ""                    # Optional, default: "", directory where db file created or imported, can be absolute or relative path
#        dryRun: true                 # Optional, default: false
#        params: []                   # Optional, default: ["cache=shared"]
#    loggerEntry: ""                  # Optional, default: default logger with STDOUT
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
	"github.com/rookie-ninja/rk-boot/v2"
	"github.com/rookie-ninja/rk-db/sqlite"
	"github.com/rookie-ninja/rk-gin/v2/boot"
	"gorm.io/gorm"
	"net/http"
	"strconv"
	"time"
)

var userDb *gorm.DB

func main() {
	boot := rkboot.NewBoot()

	boot.Bootstrap(context.TODO())

	// Auto migrate database and init global userDb variable
	sqliteEntry := rksqlite.GetSqliteEntry("user-db")
	userDb = sqliteEntry.GetDB("user")
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
	CreatedAt time.Time      `yaml:"-" json:"-"`
	UpdatedAt time.Time      `yaml:"-" json:"-"`
	DeletedAt gorm.DeletedAt `yaml:"-" json:"-" gorm:"index"`
}

type User struct {
	Base
	Id   int    `yaml:"id" json:"id" gorm:"primaryKey"`
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
	res := userDb.Where("id = ?", uid).Find(user)

	if res.Error != nil {
		ctx.JSON(http.StatusInternalServerError, res.Error)
		return
	}
	ctx.JSON(http.StatusOK, user)
}

func CreateUser(ctx *gin.Context) {
	user := &User{
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
		Name: ctx.Query("name"),
	}

	res := userDb.Where("id = ?", uid).Updates(user)

	if res.Error != nil {
		ctx.JSON(http.StatusInternalServerError, res.Error)
		return
	}

	if res.RowsAffected < 1 {
		ctx.JSON(http.StatusNotFound, "user not found")
		return
	}

	// get user
	userDb.Where("id = ?", uid).Find(user)

	ctx.JSON(http.StatusOK, user)
}

func DeleteUser(ctx *gin.Context) {
	uid, _ := strconv.Atoi(ctx.Param("id"))
	res := userDb.Delete(&User{
		Id: uid,
	})

	if res.Error != nil {
		ctx.JSON(http.StatusInternalServerError, res.Error)
		return
	}

	if res.RowsAffected < 1 {
		ctx.JSON(http.StatusNotFound, "user not found")
		return
	}

	ctx.String(http.StatusOK, "success")
}
```

### 3.文件夹结构
```shell script
$ tree
.
├── boot.yaml
├── go.mod
├── go.sum
└── main.go
```

### 4.运行 main.go
```shell
$ go run main.go

2022-01-06T20:53:24.023+0800    INFO    boot/gin_entry.go:913   Bootstrap ginEntry      {"eventId": "505acbfa-7f0b-425b-be53-c9b80036a3f7", "entryName": "user-service"}
------------------------------------------------------------------------
endTime=2022-01-06T20:53:24.023552+08:00
startTime=2022-01-06T20:53:24.023405+08:00
elapsedNano=147732
timezone=CST
ids={"eventId":"505acbfa-7f0b-425b-be53-c9b80036a3f7"}
app={"appName":"rk","appVersion":"","entryName":"user-service","entryType":"GinEntry"}
env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"*","region":"*"}
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
2022-01-06T20:53:24.023+0800    INFO    Bootstrap SQLite entry  {"entryName": "user-db"}
```

### 6.验证
#### 6.1 创建用户
```shell script
$ curl -X PUT "localhost:8080/v1/user?name=rk-dev"
{"id":2,"name":"rk-dev"}
```

#### 6.2 更新用户
```shell script
$ curl -X POST "localhost:8080/v1/user/2?name=rk-dev-updated"
{"id":2,"name":"rk-dev-updated"}
```

#### 6.3 列出所有用户
```shell script
$ curl -X GET localhost:8080/v1/user
[{"id":2,"name":"rk-dev-updated"}]
```

#### 6.4 获取用户
```shell script
$ curl -X GET localhost:8080/v1/user/2
{"id":2,"name":"rk-dev-updated"}
```

#### 6.5 删除用户

```shell script
$ curl -X DELETE localhost:8080/v1/user/2
success
```

## 完整 YAML 配置

| name                     | Required | description                        | type     | default value                          |
|--------------------------|----------|------------------------------------|----------|----------------------------------------|
| sqlite.name              | Required | The name of entry                  | string   | SQLite                                 |
| sqlite.enabled           | Required | Enable entry or not                | bool     | false                                  |
| sqlite.domain            | Required | See locale description bellow      | string   | "*"                                    |
| sqlite.description       | Optional | Description of echo entry.         | string   | ""                                     |
| sqlite.database.name     | Required | Name of database                   | string   | ""                                     |
| sqlite.database.inMemory | Optional | SQLite in memory                   | bool     | false                                  |
| sqlite.database.dbDir    | Optional | Specify *.db file directory        | string   | "", current working directory if empty |
| sqlite.database.dryRun   | Optional | Run gorm.DB with dry run mode      | bool     | false                                  |
| sqlite.database.params   | Optional | Connection params                  | []string | ["cache=shared"]                       |
| sqlite.logger.zapLogger  | Optional | Reference of zap logger entry name | string   | ""                                     |

```yaml
sqlite:
  - name: user-db                     # Required
    enabled: true                     # Required
    locale: "*::*::*::*"              # Required
    database:
      - name: user                    # Required
#        inMemory: true               # Optional, default: false
#        dbDir: ""                    # Optional, default: "", directory where db file created or imported, can be absolute or relative path
#        dryRun: true                 # Optional, default: false
#        params: []                   # Optional, default: ["cache=shared"]
#    loggerEntry: ""                  # Optional, default: default logger with STDOUT
```

