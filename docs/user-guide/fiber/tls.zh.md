启动 TLS/SSL。

## 概述
为了能在服务中启动 TLS/SSL，我们需要服务证书和服务证书密钥。

启动器会从 boot.yaml 中读取 CertEntry 部分，并且载入到内存中。

## DOMAIN
CertEntry 支持通过 DOMAIN 区分环境

## 生成 Self-Signed Certificate
> 我们推荐使用 [cfssl](https://github.com/cloudflare/cfssl) 来生成自定义证书。

### 1.下载 cfssl & cfssljson
> 用 rk 命令行安装 cfssl 与 cfssljson
```bash
$ go get github.com/rookie-ninja/rk/cmd/rk
$ rk install cfssl
$ rk install cfssljson
```

### 2.生成 CA
```bash
$ cfssl print-defaults config > ca-config.json
$ cfssl print-defaults csr > ca-csr.json
```
> 根据需要修改 ca-config.json 和 ca-csr.json。
```bash
$ cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
```

### 3.生成服务端证书
> server.csr，server.pem 和 server-key.pem 将会被生成。
```bash
$ cfssl gencert -config ca-config.json -ca ca.pem -ca-key ca-key.pem -profile www csr.json | cfssljson -bare server
```

## 快速开始
### 1.安装

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-fiber
```

### 2.创建 boot.yaml
```yaml
---
cert:
  - name: my-cert
    certPemPath: "server.pem"
    keyPemPath: "server-key.pem"
fiber:
  - name: greeter
    port: 8080
    enabled: true
    certEntry: "my-cert"
```

```bash
$ curl --insecure  "https://localhost:8080/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}
```

```shell
.
├── boot.yaml
├── go.mod
├── go.sum
├── main.go
├── server-key.pem
└── server.pem
```

### _**Cheers**_
![](../../img/user-guide/cheers.png)

## YAML 选项
```yaml
cert:
  - name: my-cert                                         # Required
    description: "Description of entry"                   # Optional, default: ""
    domain: "*"                                           # Optional, default: "*"
    caPath: "certs/ca.pem"                                # Optional, default: ""
    certPemPath: "certs/server-cert.pem"                  # Optional, default: ""
    keyPemPath: "certs/server-key.pem"                    # Optional, default: ""
```