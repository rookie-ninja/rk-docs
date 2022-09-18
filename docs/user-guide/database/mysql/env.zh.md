rk-db/mysql 会可以根据 DOMAIN 环境变量区分不同的环境。

## 概述
让我们考虑如下的场景，我们需要在【test】和【prod】两个环境的服务器中，使用不同的配置文件，比如数据库配置文件等等。

因为 rk-db/mysql 支持多个 DB Entry，我们需要一个机制来分辨不同的 DB Entry.

> 注意，为了代码的整洁性，不同的 DB Entry 会用同样的名字，这样一来，对于代码来说，才可以做到与环境无关。

## 概念
**Entry 是如何被 rk-db/mysql 筛选？**

rk-db/mysql 会使用 DOMAIN 环境变量来区分不同的环境。这也是 RK 推荐的云原生环境分辨法。

## 快速开始
### 1.安装

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-gin/v2
$ go get github.com/rookie-ninja/rk-db/mysql
```

### 2.创建 boot.yaml
```yaml
---
gin:
  - name: user-service
    port: 8080
    enabled: true
mysql:
  - name: demo-db
    enabled: true
    domain: dev                 # ENV: DOMAIN=dev
    addr: "localhost:3306"
    database:
      - name: demo
        autoCreate: true
  - name: demo-db
    enabled: true
    domain: prod                # ENV: DOMAIN=prod
    addr: "remote.host:3306"
    database:
      - name: demo
        autoCreate: true
```

### 3.ENV：DOMAIN=dev
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot/v2"
	_ "github.com/rookie-ninja/rk-db/mysql"
	"os"
)

func main() {
	os.Setenv("DOMAIN", "dev")

	boot := rkboot.NewBoot()

	boot.Bootstrap(context.TODO())

	boot.WaitForShutdownSig(context.TODO())
}
```

> 输出
```bash
2022-09-19T00:27:01.273+0800    INFO    mysql/boot.go:378       Bootstrap MySqlEntry    {"eventId": "bb4c7b17-db51-4c58-b611-0543e6689f2d", "entryName": "demo-db", "entryType": "MySqlEntry"}
2022-09-19T00:27:01.273+0800    INFO    mysql/boot.go:497       Creating database [demo]
2022-09-19T00:27:01.286+0800    INFO    mysql/boot.go:519       Creating database [demo] successs
2022-09-19T00:27:01.286+0800    INFO    mysql/boot.go:522       Connecting to database [demo]
2022-09-19T00:27:01.295+0800    INFO    mysql/boot.go:540       Connecting to database [demo] success
```

### 4.无环境变量
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot/v2"
	_ "github.com/rookie-ninja/rk-db/mysql"
)

func main() {
	boot := rkboot.NewBoot()

	boot.Bootstrap(context.TODO())

	boot.WaitForShutdownSig(context.TODO())
}
```

> 输出
```bash
# no logs which means no connection
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)
