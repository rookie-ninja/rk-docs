Manage logging.

## Overview
By default, logs will be flushed to stdout. User can configure logs to multiple channels.

## Install

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-gin/v2
$ go get github.com/rookie-ninja/rk-db/postgres
```

## Log level
By default, the log level is **warn** which prints with error.

Supported log levels are:

- info
- warn
- error
- silent

### 1.Create boot.yaml
```yaml
---
gin:
  - name: user-service
    port: 8080
    enabled: true
postgres:
  - name: demo-db
    enabled: true
    addr: "localhost:5432"
    user: postgres              # Optional, default: postgres
    pass: pass                  # Optional, default: pass
    logger:
      level: info               # Set log level to info
    database:
      - name: demo
        autoCreate: true
```

### 2.Create main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot/v2"
	"github.com/rookie-ninja/rk-db/postgres"
	"gorm.io/gorm"
	"time"
)

func main() {
	boot := rkboot.NewBoot()

	boot.Bootstrap(context.TODO())

	// Auto migrate database and init global userDb variable
	pgEntry := rkpostgres.GetPostgresEntry("demo-db")
	userDb := pgEntry.GetDB("demo")

	if !userDb.DryRun {
		userDb.AutoMigrate(&User{})
	}

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
```

> Output
```bash
...
2022-09-19T00:42:46.790+0800    INFO    postgres@v1.3.6/migrator.go:250    [1.337ms] [rows:-] SELECT DATABASE()
2022-09-19T00:42:46.794+0800    INFO    postgres@v1.3.6/migrator.go:253    [3.696ms] [rows:1] SELECT SCHEMA_NAME from Information_schema.SCHEMATA where SCHEMA_NAME LIKE 'demo%' ORDER BY SCHEMA_NAME='demo' DESC,SCHEMA_NAME limit 1
...
```

## Output paths
By default, the log output path is stdout.

### 1.Create boot.yaml
```yaml
---
gin:
  - name: user-service
    port: 8080
    enabled: true
postgres:
  - name: demo-db
    enabled: true
    addr: "localhost:5432"
    user: postgres
    pass: pass
    logger:
      level: info
      outputPaths: [ "log/db.log" ]
```

### 2.Validate
```bash
.
├── boot.yaml
├── go.mod
├── go.sum
├── log
│   └── db.log
└── main.go
```

## Full options
```yaml
postgres:
  - name: demo-db                               # Required
    enabled: true                               # Required
    domain: "*"                                 # Optional
    addr: "localhost:5432"                      # Optional, default: localhost:5432
    user: postgres                              # Optional, default: postgres
    pass: pass                                  # Optional, default: pass
    protocol: tcp                               # Optional, default: tcp
    logger:
      entry: ""                                 # Optional, default: "", logger entry name
      level: warn                               # Optional, default: warn, options: [warn, info, error, silent]
      encoding: json                            # Optional, default: console, options: [json, console]
      outputPaths: [ "log/db.log" ]             # Optional, default: empty
      slowThresholdMs: 5000                     # Optional, default: 5000
      ignoreRecordNotFoundError: false          # Optional, default: false
    database:
      - name: demo                              # Required
        plugins:
          prom:
            enabled: false
        autoCreate: true                        # Optional, default: false
        dryRun: false                           # Optional, default: false
        preferSimpleProtocol: false             # Optional, default: false
        params: []                              # Optional, default: ["sslmode=disable","TimeZone=Asia/Shanghai"]
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)
