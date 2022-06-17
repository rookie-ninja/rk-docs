---
title: "MongoDB"
linkTitle: "MongoDB"
weight: 1
description: >
  Connect to mongoDB with rk-boot and rk-db/mongodb plugin.
---

## Overview
We will use rk-boot & [rk-db/mongodb](https://github.com/rookie-ninja/rk-db/mongodb) to connect to MongoDB Cluster。
[rk-db/mongodb](https://github.com/rookie-ninja/rk-db/mongodb) uses [mongo-driver](https://pkg.go.dev/go.mongodb.org/mongo-driver?utm_source=godoc) as driver by default.

In order to demonstrate full example，we will use [rk-gin](https://github.com/rookie-ninja/rk-gin/) to start a back end service with APIs as bellow.

- GET /v1/user, List users
- GET /v1/user/:id, Get user
- POST /v1/user, Create user
- PUT /v1/user/:id, Update user
- DELETE /v1/user/:id, Delete user

## Install

- rk-boot: Base package.
- rk-gin: To start [gin-gonic/gin](https://github.com/gin-gonic/gin) microservice.
- rk-db/mongodb: Plugin to connect to MongoDB with [mongo-driver](https://pkg.go.dev/go.mongodb.org/mongo-driver?utm_source=godoc) Client

```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gin/v2
go get github.com/rookie-ninja/rk-db/mongodb
```

## Quick start
### 1. Create boot.yaml
```yaml
---
gin:
  - name: user-service
    port: 8080
    enabled: true
mongo:
  - name: "my-mongo"                            # Required
    enabled: true                               # Required
    simpleURI: "mongodb://localhost:27017"      # Required
    database:
      - name: "users"                           # Required
```

### 2. Create main.go

```go
package main

import (
	"context"
	"encoding/json"
	"fmt"
	"github.com/gin-gonic/gin"
	"github.com/rookie-ninja/rk-boot/v2"
	"github.com/rookie-ninja/rk-db/mongodb"
	"github.com/rookie-ninja/rk-gin/v2/boot"
	"github.com/rs/xid"
	"go.mongodb.org/mongo-driver/bson"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
	"net/http"
)

var (
	userCollection *mongo.Collection
)

func createCollection(db *mongo.Database, name string) {
	opts := options.CreateCollection()
	err := db.CreateCollection(context.TODO(), name, opts)
	if err != nil {
		fmt.Println("collection exists may be, continue")
	}
}

func main() {
	boot := rkboot.NewBoot()

	boot.Bootstrap(context.TODO())

	// Auto migrate database and init global userDb variable
	db := rkmongo.GetMongoDB("my-mongo", "users")
	createCollection(db, "meta")

	userCollection = db.Collection("meta")

	// Register APIs
	ginEntry := rkgin.GetGinEntry("user-service")
	ginEntry.Router.GET("/v1/user", ListUsers)
	ginEntry.Router.GET("/v1/user/:id", GetUser)
	ginEntry.Router.POST("/v1/user", CreateUser)
	ginEntry.Router.PUT("/v1/user/:id", UpdateUser)
	ginEntry.Router.DELETE("/v1/user/:id", DeleteUser)

	boot.WaitForShutdownSig(context.TODO())
}

// *************************************
// *************** Model ***************
// *************************************

type User struct {
	Id   string `bson:"id" yaml:"id" json:"id"`
	Name string `bson:"name" yaml:"name" json:"name"`
}

func ListUsers(ctx *gin.Context) {
	userList := make([]*User, 0)

	cursor, err := userCollection.Find(context.Background(), bson.D{})

	if err != nil {
		ctx.JSON(http.StatusInternalServerError, err)
		return
	}

	if err = cursor.All(context.TODO(), &userList); err != nil {
		ctx.JSON(http.StatusInternalServerError, err)
		return
	}

	ctx.JSON(http.StatusOK, userList)
}

func GetUser(ctx *gin.Context) {
	res := userCollection.FindOne(context.Background(), bson.M{"id": ctx.Param("id")})

	if res.Err() != nil {
		ctx.AbortWithError(http.StatusInternalServerError, res.Err())
		return
	}

	user := &User{}
	err := res.Decode(user)
	if err != nil {
		ctx.AbortWithError(http.StatusInternalServerError, err)
		return
	}

	ctx.JSON(http.StatusOK, user)
}

func CreateUser(ctx *gin.Context) {
	user := &User{
		Id:   xid.New().String(),
		Name: ctx.Query("name"),
	}

	_, err := userCollection.InsertOne(context.Background(), user)

	if err != nil {
		ctx.JSON(http.StatusInternalServerError, err)
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

	res, err := userCollection.UpdateOne(context.Background(), bson.M{"id": uid}, bson.D{
		{"$set", user},
	})

	if err != nil {
		ctx.AbortWithError(http.StatusInternalServerError, err)
		return
	}

	if res.MatchedCount < 1 {
		ctx.JSON(http.StatusNotFound, "user not found")
		return
	}

	ctx.JSON(http.StatusOK, user)
}

func DeleteUser(ctx *gin.Context) {
	res, err := userCollection.DeleteOne(context.Background(), bson.M{
		"id": ctx.Param("id"),
	})

	if err != nil {
		ctx.AbortWithError(http.StatusInternalServerError, err)
		return
	}

	if res.DeletedCount < 1 {
		ctx.JSON(http.StatusNotFound, "user not found")
		return
	}

	ctx.String(http.StatusOK, "success")
}
```

### 3.Start MongoDB locally

```shell
$ docker run -it --rm --name my-mongo -p 27017:27017 mongo
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
2022-06-17T15:13:33.494+0800    INFO    mongodb@v1.2.1/boot.go:330      Bootstrap mongoDbEntry  {"eventId": "7a5eacbc-5709-41b8-9b5a-ede98d2176a9", "entryName": "my-mongo", "entryType": "MongoEntry"}
2022-06-17T15:13:33.494+0800    INFO    mongodb@v1.2.1/boot.go:351      Creating mongoDB client at [localhost:27017]
2022-06-17T15:13:33.494+0800    INFO    mongodb@v1.2.1/boot.go:357      Creating mongoDB client at [localhost:27017] success
2022-06-17T15:13:33.538+0800    INFO    mongodb@v1.2.1/boot.go:371      Creating database instance [users] success
2022-06-17T15:13:33.538+0800    INFO    boot/gin_entry.go:666   Bootstrap GinEntry      {"eventId": "7a5eacbc-5709-41b8-9b5a-ede98d2176a9", "entryName": "user-service", "entryType": "GinEntry"}
------------------------------------------------------------------------
endTime=2022-06-17T15:13:33.538883+08:00
startTime=2022-06-17T15:13:33.538854+08:00
elapsedNano=28320
timezone=CST
ids={"eventId":"7a5eacbc-5709-41b8-9b5a-ede98d2176a9"}
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
```shell
$ curl -X PUT "localhost:8080/v1/user?name=rk-dev"
{"id":"cam2jnbd0cvr8b0hpmm0","name":"rk-dev"}
```

#### 6.2 Update user
```shell
$ curl -X POST "localhost:8080/v1/user/cam2jnbd0cvr8b0hpmm0?name=rk-dev-updated"
{"id":"cam2jnbd0cvr8b0hpmm0","name":"rk-dev-updated"}
```

#### 6.3 List users
```shell
$ curl -X GET localhost:8080/v1/user
[{"id":"cam2jnbd0cvr8b0hpmm0","name":"rk-dev-updated"}]%
```

#### 6.4 Get user
```shell
$ curl -X GET localhost:8080/v1/user/cam2jnbd0cvr8b0hpmm0
{"id":"cam2jnbd0cvr8b0hpmm0","name":"rk-dev-updated"}
```

#### 6.5 Delete user
```shell
$ curl -X DELETE localhost:8080/v1/user/cam2jnbd0cvr8b0hpmm0
success
```

## With TLS/SSL
In this example, we will connect to MongoDB with TLS/SSL enabled.

### 1.Create TLS/SSL certificates
```shell
$ openssl req -newkey rsa:2048 -new -x509 -days 365 -nodes -out mongodb-cert.crt -keyout mongodb-cert.key -subj "/CN=localhost"
$ cat mongodb-cert.key mongodb-cert.crt > mongodb.pem
```

### 2.Start MongoDB with Certificate
```shell
$ docker run -it --name mongo-tls --rm -v /your-path-contains-certs:/etc/ssl -p 27017:27017 mongo --tlsMode requireTLS --tlsCertificateKeyFile /etc/ssl/mongodb.pem
```

### 3.Modify boot.yaml
We add a cert entry in boot.yaml.

```yaml
---
gin:
  - name: user-service
    port: 8080
    enabled: true
cert:
  - name: mongo-cert   
    caPath: "/your-path-contains-certs/mongodb.pem"      # Cert file path
mongo:
  - name: "my-mongo"
    enabled: true
    certEntry: mongo-cert                                # CertEntry name, required
    insecureSkipVerify: true                             # It is self-signed cert, required
    simpleURI: "mongodb://localhost:27017"
    database:
      - name: "users"
```

### 4.Directory hierarchy
```shell
.
├── boot.yaml
├── go.mod
├── go.sum
├── main.go
├── mongodb-cert.crt
├── mongodb-cert.key
└── mongodb.pem
```

### 5.Validate
```shell
$ go run main.go
2022-06-17T17:00:54.008+0800    INFO    mongodb@v1.2.1/boot.go:330      Bootstrap mongoDbEntry  {"eventId": "31e0cc59-b5d8-4078-ac4f-793363f05fce", "entryName": "my-mongo", "entryType": "MongoEntry"}
2022-06-17T17:00:54.008+0800    INFO    mongodb@v1.2.1/boot.go:351      Creating mongoDB client at [localhost:27017]
2022-06-17T17:00:54.008+0800    INFO    mongodb@v1.2.1/boot.go:357      Creating mongoDB client at [localhost:27017] success
2022-06-17T17:00:54.036+0800    INFO    mongodb@v1.2.1/boot.go:371      Creating database instance [users] success
2022-06-17T17:00:54.036+0800    INFO    boot/gin_entry.go:666   Bootstrap GinEntry      {"eventId": "31e0cc59-b5d8-4078-ac4f-793363f05fce", "entryName": "user-service", "entryType": "GinEntry"}
------------------------------------------------------------------------
endTime=2022-06-17T17:00:54.036918+08:00
startTime=2022-06-17T17:00:54.036878+08:00
elapsedNano=40568
timezone=CST
ids={"eventId":"31e0cc59-b5d8-4078-ac4f-793363f05fce"}
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

## Full YAML options
```yaml
mongo:
  - name: "my-mongo"
    enabled: true
    simpleURI: "mongodb://localhost:27017"
    database:
      - name: "users"
#    description: "description"
#    locale: "*::*::*::*"
#    certEntry: ""
#    insecureSkipVerify: true
#    loggerEntry: ""
#    # Belongs to mongoDB client options
#    # Please refer to https://github.com/mongodb/mongo-go-driver/blob/master/mongo/options/clientoptions.go
#    appName: ""
#    auth:
#      mechanism: ""
#      mechanismProperties:
#        a: b
#      source: ""
#      username: ""
#      password: ""
#      passwordSet: false
#    connectTimeoutMs: 500
#    compressors: []
#    direct: false
#    disableOCSPEndpointCheck: false
#    heartbeatIntervalMs: 10
#    hosts: []
#    loadBalanced: false
#    localThresholdMs: 1
#    maxConnIdleTimeMs: 1
#    maxPoolSize: 1
#    minPoolSize: 1
#    maxConnecting: 1
#    replicaSet: ""
#    retryReads: false
#    retryWrites: false
#    serverAPIOptions:
#      serverAPIVersion: ""
#      strict: false
#      deprecationErrors: false
#    serverSelectionTimeoutMs: 1
#    socketTimeout: 1
#    srvMaxHots: 1
#    srvServiceName: ""
#    zlibLevel: 1
#    zstdLevel: 1
#    authenticateToAnything: false
```
