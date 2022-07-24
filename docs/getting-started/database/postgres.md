Connect to PostgresSQL with rk-boot and rk-db/postgres plugin.

## Overview
We will use rk-boot & [rk-db/postgres](https://github.com/rookie-ninja/rk-db) to connect to PostgresSQL cluster.
[rk-db/postgres](https://github.com/rookie-ninja/rk-db) uses [gorm](https://github.com/go-gorm/gorm) as driver by default.

In order to demonstrate full example，we will use [rk-gin](https://github.com/rookie-ninja/rk-gin/) to start a back end service with APIs as bellow.

- GET /v1/user, List users
- GET /v1/user/:id, Get user
- PUT /v1/user, Create user
- POST /v1/user/:id, Update user
- DELETE /v1/user/:id, Delete user

## Install

- rk-boot: Base package.
- rk-gin: To start [gin-gonic/gin](https://github.com/gin-gonic/gin) microservice.
- rk-db/postgres: Plugin to connect to PostgresSQL with [gorm](https://github.com/go-gorm/gorm).

```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gin/v2
go get github.com/rookie-ninja/rk-db/postgres
```

## Quick start
### 1. Create boot.yaml
```yaml
---
gin:
  - name: user-service
    port: 8080
    enabled: true
postgres:
  - name: user-db                     # Required
    enabled: true                     # Required
    domain: "*"                       # Optional
    addr: "localhost:5432"            # Optional, default: localhost:5432
    user: postgres                    # Optional, default: postgres
    pass: pass                        # Optional, default: pass
    database:
      - name: user                    # Required
        autoCreate: true              # Optional, default: false
#        dryRun: true                 # Optional, default: false
#        preferSimpleProtocol: false  # Optional, default: false
#        params: []                   # Optional, default: ["sslmode=disable","TimeZone=Asia/Shanghai"]
#    loggerEntry: ""                  # Optional, default: default logger with STDOUT
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
  "github.com/rookie-ninja/rk-db/postgres"
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
  pgEntry := rkpostgres.GetPostgresEntry("user-db")
  userDb = pgEntry.GetDB("user")
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

### 3.Start postgresSQL locally

```bash
$ docker run -it --rm --name rk-postgres -e POSTGRES_PASSWORD=pass -p 5432:5432  postgres
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

2022-06-17T19:50:35.183+0800    INFO    postgres/boot.go:278    Bootstrap postgresEntry {"eventId": "20f92df1-7269-4354-967f-bf8e5285113b", "entryName": "user-db", "entryType": "PostgreSqlEntry"}
2022-06-17T19:50:35.183+0800    INFO    postgres/boot.go:378    Creating database [user] if not exists
2022-06-17T19:50:35.207+0800    INFO    postgres/boot.go:405    Database:user not found, create with owner:postgres, encoding:UTF8
2022-06-17T19:50:35.324+0800    INFO    postgres/boot.go:412    Creating database [user] successs
2022-06-17T19:50:35.324+0800    INFO    postgres/boot.go:415    Connecting to database [user]
2022-06-17T19:50:35.338+0800    INFO    postgres/boot.go:429    Connecting to database [user] success
2022-06-17T19:50:35.338+0800    INFO    boot/gin_entry.go:666   Bootstrap GinEntry      {"eventId": "20f92df1-7269-4354-967f-bf8e5285113b", "entryName": "user-service", "entryType": "GinEntry"}
------------------------------------------------------------------------
endTime=2022-06-17T19:50:35.338142+08:00
startTime=2022-06-17T19:50:35.338106+08:00
elapsedNano=36683
timezone=CST
ids={"eventId":"20f92df1-7269-4354-967f-bf8e5285113b"}
app={"appName":"rk","appVersion":"local","entryName":"user-service","entryType":"GinEntry"}
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
#### 6.1 Create user
```bash
$ curl -X PUT "localhost:8080/v1/user?name=rk-dev"
{"id":2,"name":"rk-dev"}
```

#### 6.2 Update user
```bash
$ curl -X POST "localhost:8080/v1/user/2?name=rk-dev-updated"
{"id":2,"name":"rk-dev-updated"}
```

#### 6.3 List users
```bash
$ curl -X GET localhost:8080/v1/user
[{"id":2,"name":"rk-dev-updated"}]
```

#### 6.4 Get user
```bash
$ curl -X GET localhost:8080/v1/user/2
{"id":2,"name":"rk-dev-updated"}
```

#### 6.5 Delete user

```bash
$ curl -X DELETE localhost:8080/v1/user/2
success
```

## Full YAML Option

| name                                   | Required | description                        | type     | default value                                |
|----------------------------------------|----------|------------------------------------|----------|----------------------------------------------|
| postgres.name                          | Required | The name of entry                  | string   | PostgreSQL                                   |
| postgres.enabled                       | Required | Enable entry or not                | bool     | false                                        |
| postgres.domain                        | Optional | See locale description bellow      | string   | "*"                                          |
| postgres.description                   | Optional | Description of echo entry.         | string   | ""                                           |
| postgres.user                          | Optional | PostgreSQL username                | string   | postgres                                     |
| postgres.pass                          | Optional | PostgreSQL password                | string   | pass                                         |
| postgres.addr                          | Optional | PostgreSQL remote address          | string   | localhost:5432                               |
| postgres.database.name                 | Required | Name of database                   | string   | ""                                           |
| postgres.database.autoCreate           | Optional | Create DB if missing               | bool     | false                                        |
| postgres.database.dryRun               | Optional | Run gorm.DB with dry run mode      | bool     | false                                        |
| postgres.database.preferSimpleProtocol | Optional | Disable prepared statement cache   | bool     | false                                        |
| postgres.database.params               | Optional | Connection params                  | []string | ["sslmode=disable","TimeZone=Asia/Shanghai"] |
| postgres.loggerEntry                   | Optional | Reference of zap logger entry name | string   | ""                                           |

```yaml
postgres:
  - name: user-db                     # Required
    enabled: true                     # Required
    domain: "*"                       # Optional
    addr: "localhost:5432"            # Optional, default: localhost:5432
    user: postgres                    # Optional, default: postgres
    pass: pass                        # Optional, default: pass
    database:
      - name: user                    # Required
        autoCreate: true              # Optional, default: false
#        dryRun: true                 # Optional, default: false
#        preferSimpleProtocol: false  # Optional, default: false
#        params: []                   # Optional, default: ["sslmode=disable","TimeZone=Asia/Shanghai"]
#    loggerEntry: ""                  # Optional, default: default logger with STDOUT
```

