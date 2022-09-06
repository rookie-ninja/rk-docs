通过 rk-boot，配合 rk-db/sqlserver 插件，快速连接 SqlServer。

## 概述
我们将会使用 rk-boot 与 [rk-db/sqlserver](https://github.com/rookie-ninja/rk-db) 插件连接 SqlServer 集群。
[rk-db/sqlserver](https://github.com/rookie-ninja/rk-db) 插件默认使用了 [gorm](https://github.com/go-gorm/gorm)

rk-boot 会根据 boot.yaml 里的配置，自动创建 gorm.DB 实例，并且创建与 SqlServer 之间的连接。

为了例子的完整性，使用 [rk-gin](https://github.com/rookie-ninja/rk-gin/) 启动一个后台进程，进行验证。

本例子中，我们会创建如下几个 API 来验证与 SqlServer 的连通性.

- GET /v1/user, List users
- GET /v1/user/:id, Get user
- PUT /v1/user, Create user
- POST /v1/user/:id, Update user
- DELETE /v1/user/:id, Delete user

## 安装

- rk-boot: rk-boot 基础包
- rk-gin: 用于启动 [gin-gonic/gin](https://github.com/gin-gonic/gin) 微服务
- rk-db/sqlserver: 用于初始化连接 SqlServer 的 [gorm](https://github.com/go-gorm/gorm) 实例

```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gin/v2
go get github.com/rookie-ninja/rk-db/sqlserver
```

## 快速开始
### 1. 创建 boot.yaml
```yaml
---
gin:
  - name: user-service
    port: 8080
    enabled: true
sqlServer:
  - name: user-db                       # Required
    enabled: true                       # Required
    domain: "*"                         # Optional
    addr: "localhost:1433"              # Optional, default: localhost:1433
    user: sa                            # Optional, default: sa
    pass: pass                          # Optional, default: pass
    database:
      - name: user                      # Required
        autoCreate: true                # Optional, default: false
#        dryRun: true                   # Optional, default: false
#        params: []                     # Optional, default: []
#    logger:
#      entry: ""
#      level: info
#      encoding: json
#      outputPaths: [ "stdout", "log/db.log" ]
#      slowThresholdMs: 5000
#      ignoreRecordNotFoundError: false
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
	"github.com/rookie-ninja/rk-db/sqlserver"
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
	sqlServerEntry := rksqlserver.GetSqlServerEntry("user-db")
	userDb = sqlServerEntry.GetDB("user")
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

### 3.本地启动 SqlServer

```bash
$ docker run -it --rm --name rk-sqlserver -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=yourStrong(!)Password" -p 1433:1433 mcr.microsoft.com/mssql/server:2022-latest
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

2022-01-07T00:54:02.448+0800    INFO    boot/gin_entry.go:913   Bootstrap ginEntry      {"eventId": "c67c3f2b-9d51-4908-86c9-5b7df25ce719", "entryName": "user-service"}
------------------------------------------------------------------------
endTime=2022-01-07T00:54:02.448265+08:00
startTime=2022-01-07T00:54:02.448131+08:00
elapsedNano=134519
timezone=CST
ids={"eventId":"c67c3f2b-9d51-4908-86c9-5b7df25ce719"}
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
2022-01-07T00:54:02.448+0800    INFO    Bootstrap sqlServer entry       {"entryName": "user-db", "sqlServerUser": "sa", "sqlServerAddr": "localhost:1433"}
```

### 6.验证
#### 6.1 创建用户
```bash
$ curl -X PUT "localhost:8080/v1/user?name=rk-dev"
{"id":2,"name":"rk-dev"}
```

#### 6.2 更新用户
```bash
$ curl -X POST "localhost:8080/v1/user/2?name=rk-dev-updated"
{"id":2,"name":"rk-dev-updated"}
```

#### 6.3 列出所有用户
```bash
$ curl -X GET localhost:8080/v1/user
[{"id":2,"name":"rk-dev-updated"}]
```

#### 6.4 获取用户
```bash
$ curl -X GET localhost:8080/v1/user/2
{"id":2,"name":"rk-dev-updated"}
```

#### 6.5 删除用户

```bash
$ curl -X DELETE localhost:8080/v1/user/2
success
```

## 完整 YAML 配置

| name                                       | Required | description                                | type     | default value  |
|--------------------------------------------|----------|--------------------------------------------|----------|----------------|
| sqlServer.name                             | Required | The name of entry                          | string   | SqlServer      |
| sqlServer.enabled                          | Required | Enable entry or not                        | bool     | false          |
| sqlServer.domain                           | Required | See locale description bellow              | string   | "*"            |
| sqlServer.description                      | Optional | Description of echo entry.                 | string   | ""             |
| sqlServer.user                             | Optional | SQL Server username                        | string   | sa             |
| sqlServer.pass                             | Optional | SQL Server password                        | string   | pass           |
| sqlServer.addr                             | Optional | SQL Server remote address                  | string   | localhost:1433 |
| sqlServer.database.name                    | Required | Name of database                           | string   | ""             |
| sqlServer.database.autoCreate              | Optional | Create DB if missing                       | bool     | false          |
| sqlServer.database.dryRun                  | Optional | Run gorm.DB with dry run mode              | bool     | false          |
| sqlServer.database.params                  | Optional | Connection params                          | []string | []             |
| sqlServer.logger.entry                     | Optional | Reference of zap logger entry name         | string   | ""             |
| sqlServer.logger.level                     | Optional | Logging level, [info, warn, error, silent] | string   | warn           |
| sqlServer.logger.encoding                  | Optional | log encoding, [console, json]              | string   | console        |
| sqlServer.logger.outputPaths               | Optional | log output paths                           | []string | ["stdout"]     |
| sqlServer.logger.slowThresholdMs           | Optional | Slow SQL threshold                         | int      | 5000           |
| sqlServer.logger.ignoreRecordNotFoundError | Optional | As name described                          | bool     | false          |

```yaml
sqlServer:
  - name: user-db                       # Required
    enabled: true                       # Required
    domain: "*"                         # Optional
    addr: "localhost:1433"              # Optional, default: localhost:1433
    user: sa                            # Optional, default: sa
    pass: pass                          # Optional, default: pass
    database:
      - name: user                      # Required
        autoCreate: true                # Optional, default: false
#        dryRun: true                   # Optional, default: false
#        params: []                     # Optional, default: []
#    logger:
#      entry: ""
#      level: info
#      encoding: json
#      outputPaths: [ "stdout", "log/db.log" ]
#      slowThresholdMs: 5000
#      ignoreRecordNotFoundError: false
```

