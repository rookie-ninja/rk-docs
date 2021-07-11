---
title: "TLS/SSL"
linkTitle: "TLS/SSL"
weight: 4
description: >
  Enable TLS/SSL for the server.
---

## Overview
In order to enable TLS/SSL on the server, we need server certificate and server key at least.

Bootstrapper will read CertEntry section in boot.yaml file and load certificates into memory.

We support four types of CertEntry retrievers. We will extend retriever future.
- LocalFs
- RemoteFs
- Consul
- Etcd

## Architecture
![](/bootstrapper/user-guide/go/grpc/advanced/tls-arch.png)

## Locale
> CertEntry support concept of [locale](/docs/bootstrapper/user-guide/go/grpc/advanced/locale/).

## Generate Self-Signed Certificate
> There is a convenient way to generate certificates with [cfssl](https://github.com/cloudflare/cfssl)

### 1.Download cfssl & cfssljson
> Install cfssl with rk cli
```shell script
$ go get -u github.com/rookie-ninja/rk/cmd/rk
$ rk install cfssl
$ rk install cfssljson
```

### 2.Generate CA
```shell script
$ cfssl print-defaults config > ca-config.json
$ cfssl print-defaults csr > ca-csr.json
```
> Adjust ca-config.json and ca-csr.json as needed.
```shell script
$ cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
```

### 3.Generate server certificates
> server.csr, server.pem and server-key.pem would be created.
```shell script
$ cfssl gencert -config ca-config.json -ca ca.pem -ca-key ca-key.pem -profile www csr.json | cfssljson -bare server
```

## Quick start
### 1.localFs
| Name | Description | Default |
| ------ | ------ | ------ |
| cert.localFs.name | Name of localFs retriever | "" |
| cert.localFs.locale | Represent environment of current process follows schema of \<realm\>::\<region\>::\<az\>::\<domain\> | \*::\*::\*::\* | 
| cert.localFs.serverCertPath | Path of server cert in local file system. | "" |
| cert.localFs.serverKeyPath | Path of server key in local file system. | "" |
| cert.localFs.clientCertPath | Path of client cert in local file system. | "" |
| cert.localFs.clientCertPath | Path of client key in local file system. | "" |

> Enable GRPC TLS only

```yaml
---
cert:
  - name: "local-cert"                     # Required
    description: "Description of entry"    # Optional
    provider: "localFs"                    # Required, etcd, consul, localFs, remoteFs are supported options
    locale: "*::*::*::*"                   # Optional, default: *::*::*::*
    serverCertPath: "cert/server.pem"      # Optional, default: "", path of certificate on local FS
    serverKeyPath: "cert/server-key.pem"   # Optional, default: "", path of certificate on local FS
grpc:
  - name: greeter
    port: 1949
    cert:
      ref: "local-cert"                    # Enable grpc TLS
    commonService: 
      enabled: true
```

```shell script
$ curl "localhost:8080/rk/v1/healthy"
{"healthy":true}
```

> Enable GRPC TLS and grpc-gateway TLS

```yaml
---
cert:
  - name: "local-cert"                     # Required
    description: "Description of entry"    # Optional
    provider: "localFs"                    # Required, etcd, consul, localFs, remoteFs are supported options
    locale: "*::*::*::*"                   # Optional, default: *::*::*::*
    serverCertPath: "cert/server.pem"      # Optional, default: "", path of certificate on local FS
    serverKeyPath: "cert/server-key.pem"   # Optional, default: "", path of certificate on local FS
grpc:
  - name: greeter
    port: 1949
    cert:
      ref: "local-cert"                    # Enable grpc TLS
    commonService: 
      enabled: true
    gw:
      enabled: true                 # Enable grpc-gateway, https://github.com/grpc-ecosystem/grpc-gateway
      port: 8080                    # Port of grpc-gateway
      cert:
        ref: "local-cert"
```

```shell script
$ curl -X GET --insecure https://localhost:8080/rk/v1/healthy
{"healthy":true}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 2.remoteFs
| Name | Description | Default |
| ------ | ------ | ------ |
| cert.remoteFs.name | Name of remoteFileStore retriever | "" |
| cert.remoteFs.locale | Represent environment of current process follows schema of \<realm\>::\<region\>::\<az\>::\<domain\> | \*::\*::\*::\* | 
| cert.remoteFs.endpoint | Endpoint of remoteFileStore server, http://x.x.x.x or x.x.x.x both acceptable. | N/A |
| cert.remoteFs.basicAuth | Basic auth for remoteFileStore server, like <user:pass>. | "" |
| cert.remoteFs.serverCertPath | Path of server cert in remoteFs server. | "" |
| cert.remoteFs.serverKeyPath | Path of server key in remoteFs server. | "" |
| cert.remoteFs.clientCertPath | Path of client cert in remoteFs server. | "" |
| cert.remoteFs.clientCertPath | Path of client key in remoteFs server. | "" |

```yaml
---
cert:
  - name: "remote-cert"                    # Required
    description: "Description of entry"    # Optional
    provider: "remoteFs"                   # Required, etcd, consul, localFs, remoteFs are supported options
    endpoint: "localhost:8081"             # Required, both http://x.x.x.x or x.x.x.x are acceptable
    locale: "*::*::*::*"                   # Optional, default: *::*::*::*
    serverCertPath: "cert/server.pem"      # Optional, default: "", path of certificate on local FS
    serverKeyPath: "cert/server-key.pem"   # Optional, default: "", path of certificate on local FS
grpc:
  - name: greeter
    port: 1949
    cert:
      ref: "remote-cert"                   # Enable grpc TLS
    commonService: 
      enabled: true
    gw:
      enabled: true                        # Enable grpc-gateway, https://github.com/grpc-ecosystem/grpc-gateway
      port: 8080                           # Port of grpc-gateway
      cert:
        ref: "remote-cert"
```

```shell script
$ curl -X GET --insecure https://localhost:8080/rk/v1/healthy
{"healthy":true}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 3.consul
| Name | Description | Default |
| ------ | ------ | ------ |
| cert.consul.name | Name of consul retriever | "" |
| cert.consul.locale | Represent environment of current process follows schema of \<realm\>::\<region\>::\<az\>::\<domain\> | \*::\*::\*::\* | 
| cert.consul.endpoint | Endpoint of Consul server, http://x.x.x.x or x.x.x.x both acceptable. | N/A |
| cert.consul.datacenter | consul datacenter. | "" |
| cert.consul.token | Token for access consul. | "" |
| cert.consul.basicAuth | Basic auth for consul server, like <user:pass>. | "" |
| cert.consul.serverCertPath | Path of server cert in Consul server. | "" |
| cert.consul.serverKeyPath | Path of server key in Consul server. | "" |
| cert.consul.clientCertPath | Path of client cert in Consul server. | "" |
| cert.consul.clientCertPath | Path of client key in Consul server. | "" |

```yaml
---
cert:
  - name: "consul-cert"                    # Required
    provider: "consul"                     # Required, etcd, consul, localFS, remoteFs are supported options
    description: "Description of entry"    # Optional
    locale: "*::*::*::*"                   # Optional, default: *::*::*::*
    endpoint: "localhost:8500"             # Required, http://x.x.x.x or x.x.x.x both acceptable.
    datacenter: "dc1"                      # Optional, default: "", consul datacenter
    serverCertPath: "server.pem"           # Optional, default: "", key of value in consul
    serverKeyPath: "server-key.pem"        # Optional, default: "", key of value in consul
grpc:
  - name: greeter
    port: 1949
    cert:
      ref: "consul-cert"                   # Enable grpc TLS
    commonService: 
      enabled: true
    gw:
      enabled: true                        # Enable grpc-gateway, https://github.com/grpc-ecosystem/grpc-gateway
      port: 8080                           # Port of grpc-gateway
      cert:
        ref: "consul-cert"
```

```shell script
$ curl -X GET --insecure https://localhost:8080/rk/v1/healthy
{"healthy":true}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 4.etcd
| Name | Description | Default |
| ------ | ------ | ------ |
| cert.etcd.name | Name of etcd retriever | "" |
| cert.etcd.locale | Represent environment of current process follows schema of \<realm\>::\<region\>::\<az\>::\<domain\> | \*::\*::\*::\* | 
| cert.etcd.endpoint | Endpoint of etcd server, http://x.x.x.x or x.x.x.x both acceptable. | N/A |
| cert.etcd.basicAuth | Basic auth for etcd server, like <user:pass>. | "" |
| cert.etcd.serverCertPath | Path of server cert in etcd server. | "" |
| cert.etcd.serverKeyPath | Path of server key in etcd server. | "" |
| cert.etcd.clientCertPath | Path of client cert in etcd server. | "" |
| cert.etcd.clientCertPath | Path of client key in etcd server. | "" |

```yaml
---
cert:
  - name: "etcd-cert"                      # Required
    description: "Description of entry"    # Optional
    provider: "etcd"                       # Required, etcd, consul, localFs, remoteFs are supported options
    locale: "*::*::*::*"                   # Optional, default: *::*::*::*
    endpoint: "localhost:2379"             # Required, http://x.x.x.x or x.x.x.x both acceptable.
    serverCertPath: "server.pem"           # Optional, default: "", key of value in etcd
    serverKeyPath: "server-key.pem"        # Optional, default: "", key of value in etcd
grpc:
  - name: greeter
    port: 1949
    cert:
      ref: "etcd-cert"                   # Enable grpc TLS
    commonService: 
      enabled: true
    gw:
      enabled: true                        # Enable grpc-gateway, https://github.com/grpc-ecosystem/grpc-gateway
      port: 8080                           # Port of grpc-gateway
      cert:
        ref: "etcdetcd-cert"
```

```shell script
$ curl -X GET --insecure https://localhost:8080/rk/v1/healthy
{"healthy":true}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 5.Access CertEntry
User can access CertEntry by calling **rkentry.GlobalAppCtx.GetCertEntry()**.
