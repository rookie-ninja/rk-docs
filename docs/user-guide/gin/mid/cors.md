Enable CORS middleware

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gin/v2
```

## CORS options
| options                     | description                        | type     | default |
|--------------------------------------|--------------------------|----------|----------------------|
| gin.middleware.cors.enabled          | Enable CORS middleware   | boolean  | false                |
| gin.middleware.cors.ignore           | Ignore by path           | []string | []                   |
| gin.middleware.cors.allowOrigins     | Allowed origin list      | []string | *                    |
| gin.middleware.cors.allowMethods     | Allowed http method list | []string | All http methods     |
| gin.middleware.cors.allowHeaders     | Allowed http header list | []string | Headers from request |
| gin.middleware.cors.allowCredentials | Allowed credential list  | bool     | false                |
| gin.middleware.cors.exposeHeaders    | Exposed list of headers  | []string | ""                   |
| gin.middleware.cors.maxAge           | Max age                  | int      | 0                    |

## Quick start
### 1.Create boot.yaml
```yaml
---
gin:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      cors:
        enabled: true
        allowOrigins:
          - "http://localhost:*"
#        ignore: [""]
#        allowCredentials: false
#        allowHeaders: []
#        allowMethods: []
#        exposeHeaders: []
#        maxAge: 0
```

### 2.Create main.go
```go
package main

import (
  "context"
  "fmt"
  "github.com/gin-gonic/gin"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-gin/v2/boot"
  "net/http"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Register handler
  entry := rkgin.GetGinEntry("greeter")
  entry.Router.GET("/v1/greeter", Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

func Greeter(ctx *gin.Context) {
  ctx.JSON(http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", ctx.Query("name")),
  })
}

type GreeterResponse struct {
  Message string
}
```

### 3.Create cors.html
```html
<!DOCTYPE html>
<html>
<body>

<h1>CORS Test</h1>

<p>Call http://localhost:8080/v1/greeter</p>

<script type="text/javascript">
    window.onload = function() {
        var apiUrl = 'http://localhost:8080/v1/greeter';
        fetch(apiUrl).then(response => response.json()).then(data => {
            document.getElementById("res").innerHTML = data["Message"]
        }).catch(err => {
            document.getElementById("res").innerHTML = err
        });
    };
</script>

<h4>Response: </h4>
<p id="res"></p>

</body>
</html>
```

### 4.Directory hierarchy
```bash
.
├── boot.yaml
├── cors.html
├── go.mod
├── go.sum
└── main.go

0 directories, 5 files
```

### 5.Validation
打开 cors.html

![](../../../img/user-guide/gin/basic/cors-success.png)

### 6.Blocked CORS
Set gin.middleware.cors.allowOrigins to http://localhost:8080。

```yaml
---
gin:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      cors:
        enabled: true
        allowOrigins:
          - "http://localhost:8080"
```

![](../../../img/user-guide/gin/basic/cors-fail.png)

### _**Cheers**_
![](../../../img/user-guide/cheers.png)