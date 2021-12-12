---
title: "TLS/SSL 认证"
linkTitle: "TLS/SSL 认证"
weight: 4
description: >
  启动 TLS/SSL。
---

## 概述
为了能在服务中启动 TLS/SSL，我们需要服务证书和服务证书密钥。

启动器会从 boot.yaml 中读取 CertEntry 部分，并且载入到内存中。

启动器目前支持如下目的地（证书所在目的地）：
- 本地文件系统（localFs）
- 远程文件服务（remoteFs）
- Consul
- Etcd

## 架构
![](/bootstrapper/user-guide/gin-golang/advanced/tls-arch.png)

## Locale
> CertEntry 支持 [locale](/docs/bootstrapper/user-guide/go/gin/advanced/locale/).

## 生成 Self-Signed Certificate
> 我们推荐使用 [cfssl](https://github.com/cloudflare/cfssl) 来生成自定义证书。

### 1.下载 cfssl & cfssljson
> 用 rk 命令行安装 cfssl 与 cfssljson
```shell script
$ go get -u github.com/rookie-ninja/rk/cmd/rk
$ rk install cfssl
$ rk install cfssljson
```

### 2.生成 CA
```shell script
$ cfssl print-defaults config > ca-config.json
$ cfssl print-defaults csr > ca-csr.json
```
> 根据需要修改 ca-config.json 和 ca-csr.json。
```shell script
$ cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
```

### 3.生成服务端证书
> server.csr，server.pem 和 server-key.pem 将会被生成。
```shell script
$ cfssl gencert -config ca-config.json -ca ca.pem -ca-key ca-key.pem -profile www csr.json | cfssljson -bare server
```

## 快速开始
- 安装

```shell script
$ go get github.com/rookie-ninja/rk-boot
$ go get github.com/rookie-ninja/rk-gin
```

### 1.从本地文件系统获取
| 配置项 | 详情 | 默认值 |
| ------ | ------ | ------ |
| cert.localFs.name | 本地文件系统获取器名称 | "" |
| cert.localFs.locale | 遵从 locale: \<realm\>::\<region\>::\<az\>::\<domain\> | "" | 
| cert.localFs.serverCertPath | 服务器证书路径 | "" |
| cert.localFs.serverKeyPath | 服务器证书密钥路径 | "" |
| cert.localFs.clientCertPath | 客户端证书路径 | "" |
| cert.localFs.clientCertPath | 客户端证书密钥路径 | "" |

```yaml
---
cert:
  - name: "local-cert"                     # Required
    description: "Description of entry"    # Optional
    provider: "localFs"                    # Required, etcd, consul, localFs, remoteFs are supported options
    locale: "*::*::*::*"                   # Required, default: ""
    serverCertPath: "cert/server.pem"      # Optional, default: "", path of certificate on local FS
    serverKeyPath: "cert/server-key.pem"   # Optional, default: "", path of certificate on local FS
gin:
  - name: greeter
    port: 8080
    enabled: true
    cert:
      ref: "local-cert"
    commonService: 
      enabled: true
```

```shell script
$ curl -X GET --insecure https://localhost:8080/rk/v1/healthy
{"healthy":true}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 2.从远程文件服务获取
| 配置项 | 详情 | 默认值 |
| ------ | ------ | ------ |
| cert.remoteFs.name | 远程文件服务获取器名称 | "" |
| cert.remoteFs.locale | 遵从 locale：\<realm\>::\<region\>::\<az\>::\<domain\> | "" | 
| cert.remoteFs.endpoint | 远程地址： http://x.x.x.x 或者 x.x.x.x | N/A |
| cert.remoteFs.basicAuth | Basic auth： <user:pass>. | "" |
| cert.remoteFs.serverCertPath | 服务器证书路径	 | "" |
| cert.remoteFs.serverKeyPath | 服务器证书密钥路径		 | "" |
| cert.remoteFs.clientCertPath | 客户端证书路径	 | "" |
| cert.remoteFs.clientCertPath | 客户端证书密钥路径	 | "" |

```yaml
---
cert:
  - name: "remote-cert"                    # Required
    description: "Description of entry"    # Optional
    provider: "remoteFs"                   # Required, etcd, consul, localFs, remoteFs are supported options
    endpoint: "localhost:8081"             # Required, both http://x.x.x.x or x.x.x.x are acceptable
    locale: "*::*::*::*"                   # Required, default: ""
    serverCertPath: "cert/server.pem"      # Optional, default: "", path of certificate on local FS
    serverKeyPath: "cert/server-key.pem"   # Optional, default: "", path of certificate on local FS
gin:
  - name: greeter
    port: 8080
    enabled: true
    cert:
      ref: "remote-cert"
    commonService: 
      enabled: true
```

```shell script
$ curl -X GET --insecure https://localhost:8080/rk/v1/healthy
{"healthy":true}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 3.从 Consul 获取
| 配置项 | 详情 | 默认值 |
| ------ | ------ | ------ |
| cert.consul.name | Consul 获取器名称 | "" |
| cert.consul.locale | 遵从 locale: \<realm\>::\<region\>::\<az\>::\<domain\> | "" | 
| cert.consul.endpoint | Consul 地址： http://x.x.x.x or x.x.x.x | N/A |
| cert.consul.datacenter | Consul 数据中心 | "" |
| cert.consul.token | Consul 访问密钥 | "" |
| cert.consul.basicAuth | Consul Basic auth，格式：<user:pass>. | "" |
| cert.consul.serverCertPath | 服务器证书路径	 | "" |
| cert.consul.serverKeyPath | 服务器证书密钥路径 | "" |
| cert.consul.clientCertPath | 服务器证书密钥路径	 | "" |
| cert.consul.clientCertPath | 服务器证书密钥路径 | "" |

```yaml
---
cert:
  - name: "consul-cert"                    # Required
    provider: "consul"                     # Required, etcd, consul, localFS, remoteFs are supported options
    description: "Description of entry"    # Optional
    locale: "*::*::*::*"                   # Required, default: ""
    endpoint: "localhost:8500"             # Required, http://x.x.x.x or x.x.x.x both acceptable.
    datacenter: "dc1"                      # Optional, default: "", consul datacenter
    serverCertPath: "server.pem"           # Optional, default: "", key of value in consul
    serverKeyPath: "server-key.pem"        # Optional, default: "", key of value in consul
gin:
  - name: greeter
    port: 8080
    enabled: true
    cert:
      ref: "consul-cert"
    commonService: 
      enabled: true
```

```shell script
$ curl -X GET --insecure https://localhost:8080/rk/v1/healthy
{"healthy":true}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 4.从 ETCD 获取
| 配置项 | 详情 | 默认值 |
| ------ | ------ | ------ |
| cert.etcd.name | ETCD 获取器名称 | "" |
| cert.etcd.locale | 遵从 locale:  \<realm\>::\<region\>::\<az\>::\<domain\> | "" | 
| cert.etcd.endpoint | ETCD 地址：http://x.x.x.x or x.x.x.x | N/A |
| cert.etcd.basicAuth | ETCD basic auth，格式：<user:pass>. | "" |
| cert.etcd.serverCertPath | 服务器证书路径	 | "" |
| cert.etcd.serverKeyPath | 服务器证书路径	 | "" |
| cert.etcd.clientCertPath | 客户端证书路径	 | "" |
| cert.etcd.clientCertPath | 客户端证书密钥路径	 | "" |

```yaml
---
cert:
  - name: "etcd-cert"                      # Required
    description: "Description of entry"    # Optional
    provider: "etcd"                       # Required, etcd, consul, localFs, remoteFs are supported options
    locale: "*::*::*::*"                   # Required, default: ""
    endpoint: "localhost:2379"             # Required, http://x.x.x.x or x.x.x.x both acceptable.
    serverCertPath: "server.pem"           # Optional, default: "", key of value in etcd
    serverKeyPath: "server-key.pem"        # Optional, default: "", key of value in etcd
gin:
  - name: greeter
    port: 8080
    enabled: true
    cert:
      ref: "etcd-cert"
    commonService: 
      enabled: true
```

```shell script
$ curl -X GET --insecure https://localhost:8080/rk/v1/healthy
{"healthy":true}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 5.访问 CertEntry 实例
用户可以通过 **rkentry.GlobalAppCtx.GetCertEntry()** 获取 CertEntry 实例。
