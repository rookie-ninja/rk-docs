Enable CORS middleware

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-grpc/v2
```

## CORS options
| options                     | description                        | type     | default |
|--------------------------------------|--------------------------|----------|----------------------|
| grpc.middleware.cors.enabled          | Enable CORS middleware   | boolean  | false                |
| grpc.middleware.cors.ignore           | Ignore by path           | []string | []                   |
| grpc.middleware.cors.allowOrigins     | Allowed origin list      | []string | *                    |
| grpc.middleware.cors.allowMethods     | Allowed http method list | []string | All http methods     |
| grpc.middleware.cors.allowHeaders     | Allowed http header list | []string | Headers from request |
| grpc.middleware.cors.allowCredentials | Allowed credential list  | bool     | false                |
| grpc.middleware.cors.exposeHeaders    | Exposed list of headers  | []string | ""                   |
| grpc.middleware.cors.maxAge           | Max age                  | int      | 0                    |

## Quick start
### 1.Create and compile protocol buffer
[Compile protobuf](../buf)

### 2.Create boot.yaml
```yaml
---
grpc:
  - name: greeter
    port: 8080
    enabled: true
    enableRkGwOption: true
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

### 3.Create main.go
```go
package main

import (
  "context"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-demo/api/gen/v1"
  "github.com/rookie-ninja/rk-grpc/v2/boot"
  "google.golang.org/grpc"
)

func main() {
  boot := rkboot.NewBoot()

  // register grpc
  entry := rkgrpc.GetGrpcEntry("greeter")
  entry.AddRegFuncGrpc(registerGreeter)
  entry.AddRegFuncGw(greeter.RegisterGreeterHandlerFromEndpoint)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  // Wait for shutdown sig
  boot.WaitForShutdownSig(context.TODO())
}

func registerGreeter(server *grpc.Server) {
  greeter.RegisterGreeterServer(server, &GreeterServer{})
}

type GreeterServer struct{}

func (server *GreeterServer) Hello(ctx context.Context, _ *greeter.HelloRequest) (*greeter.HelloResponse, error) {
  return &greeter.HelloResponse{
    Message: "hello!",
  }, nil
}
```

### 4.Create cors.html
```html
<!DOCTYPE html>
<html>
<body>

<h1>CORS Test</h1>

<p>Call http://localhost:8080/v1/greeter</p>

<script type="text/javascript">
    window.onload = function() {
        var apiUrl = 'http://localhost:8080/v1/hello';
        fetch(apiUrl).then(response => response.json()).then(data => {}).catch(err => {
            document.getElementById("res").innerHTML = err
        });
    };
</script>

<h4>Response: </h4>
<p id="res"></p>

</body>
</html>
```

### 5.Validate
Open cors.html

![](../../../img/user-guide/grpc/basic/grpc-cors-success.png)

### 6.Blocked CORS
Set grpc.middleware.cors.allowOrigins to http://localhost:8080

```yaml
---
grpc:
  - name: greeter
    port: 8080
    enabled: true
    enableRkGwOption: true
    middleware:
      cors:
        enabled: true
        allowOrigins:
          - "http://localhost:8080"
```

![](../../../img/user-guide/grpc/basic/grpc-cors-fail.png)

### _**Cheers**_
![](../../../img/user-guide/cheers.png)