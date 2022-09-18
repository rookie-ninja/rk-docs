用户自定义日志管理。

## 概念
日志默认会打印到 stdout.

## 安装

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-gin/v2
$ go get github.com/rookie-ninja/rk-db/mysql
```

## 日志 level
日志级别默认为 **warn**。

支持的日志级别:

- info
- warn
- error
- silent

### 1.创建 boot.yaml
```yaml
---
gin:
  - name: user-service
    port: 8080
    enabled: true
mysql:
  - name: demo-db
    enabled: true
    addr: "localhost:3306"
    logger:
      level: info             # Set log level to info
    database:
      - name: demo
        autoCreate: true
```

### 2.创建 main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot/v2"
	"github.com/rookie-ninja/rk-db/mysql"
	_ "github.com/rookie-ninja/rk-db/mysql"
	"gorm.io/gorm"
	"time"
)

func main() {
	boot := rkboot.NewBoot()

	boot.Bootstrap(context.TODO())

	// Auto migrate database and init global userDb variable
	mysqlEntry := rkmysql.GetMySqlEntry("demo-db")
	userDb := mysqlEntry.GetDB("demo")

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

> 输出
```bash
...
2022-09-19T00:42:46.790+0800    INFO    mysql@v1.3.6/migrator.go:250    [1.337ms] [rows:-] SELECT DATABASE()
2022-09-19T00:42:46.794+0800    INFO    mysql@v1.3.6/migrator.go:253    [3.696ms] [rows:1] SELECT SCHEMA_NAME from Information_schema.SCHEMATA where SCHEMA_NAME LIKE 'demo%' ORDER BY SCHEMA_NAME='demo' DESC,SCHEMA_NAME limit 1
...
```

## 日志路径
### 1.创建 boot.yaml
```yaml
---
gin:
  - name: user-service
    port: 8080
    enabled: true
mysql:
  - name: demo-db
    enabled: true
    addr: "localhost:3306"
    logger:
      level: info
      outputPaths: [ "log/db.log" ]
```

### 2.验证
```bash
.
├── boot.yaml
├── go.mod
├── go.sum
├── log
│   └── db.log
└── main.go
```

## 所有选项
```yaml
mysql:
  - name: demo-db                               # Required
    enabled: true                               # Required
    domain: "*"                                 # Optional
    addr: "localhost:3306"                      # Optional, default: localhost:3306
    user: root                                  # Optional, default: root
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
        params: []                              # Optional, default: ["charset=utf8mb4","parseTime=True","loc=Local"]
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)
