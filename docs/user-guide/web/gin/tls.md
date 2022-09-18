Enable TLS/SSL。

## Overview
rk-boot will load element of CertEntry into memory and start server with TLS/SSL.

## DOMAIN
CertEntry use environment variable of DOMAIN to distinguish different environments.

## Generate Self-Signed Certificate
> We suggest use [cfssl](https://github.com/cloudflare/cfssl) to generate certificates

### 1.Download cfssl & cfssljson
```bash
$ go get github.com/rookie-ninja/rk/cmd/rk
$ rk install cfssl
$ rk install cfssljson
```

### 2.Generate CA
```bash
$ cfssl print-defaults config > ca-config.json
$ cfssl print-defaults csr > ca-csr.json
```
> Modify ca-config.json and ca-csr.json as needed
```bash
$ cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
```

### 3.Generate server side certs
```bash
$ cfssl gencert -config ca-config.json -ca ca.pem -ca-key ca-key.pem -profile www csr.json | cfssljson -bare server
```

## Quick start
### 1.Install

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-gin/v2
```

### 2.Create boot.yaml
```yaml
---
cert:
  - name: my-cert
    certPemPath: "server.pem"
    keyPemPath: "server-key.pem"
gin:
  - name: greeter
    port: 8080
    enabled: true
    certEntry: "my-cert"
```

```bash
$ curl --insecure  "https://localhost:8080/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}
```

```bash
.
├── boot.yaml
├── go.mod
├── go.sum
├── main.go
├── server-key.pem
└── server.pem
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)

## YAML options
```yaml
cert:
  - name: my-cert                                         # Required
    description: "Description of entry"                   # Optional, default: ""
    domain: "*"                                           # Optional, default: "*"
    caPath: "certs/ca.pem"                                # Optional, default: ""
    certPemPath: "certs/server-cert.pem"                  # Optional, default: ""
    keyPemPath: "certs/server-key.pem"                    # Optional, default: ""
```