---
title: "应用信息"
linkTitle: "应用信息"
weight: 6
description: >
  如何自定义应用信息？
---

## 概述
应用信息包含如下的信息：

```yaml
app:
  description: "this is description"  # Optional, default: ""
  keywords: ["rk", "golang"]          # Optional, default: []
  homeUrl: "http://example.com"       # Optional, default: ""
  iconUrl: "http://example.com"       # Optional, default: ""
  docsUrl: ["http://example.com"]     # Optional, default: []
  maintainers: ["rk-dev"]             # Optional, default: []
```

## 快速开始
```yaml
app:
  description: "this is description"  # Optional, default: ""
  keywords: ["rk", "golang"]          # Optional, default: []
  homeUrl: "http://example.com"       # Optional, default: ""
  iconUrl: "http://example.com"       # Optional, default: ""
  docsUrl: ["http://example.com"]     # Optional, default: []
  maintainers: ["rk-dev"]             # Optional, default: []
grpc:
  - name: greeter
    port: 8080
    commonService:
      enabled: true                   # Enable for validation
    gw:
      enabled: true
      port: 8080
      tv: 
        enabled: true                 # Enable for validation
```

### 1.从 RK TV 中访问
> [http://localhost:8080/rk/v1/tv/info](http://localhost:8080/rk/v1/tv/info)

![](/bootstrapper/user-guide/grpc-golang/advanced/app-info.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 2.从通用 API 中访问
```shell script
$ curl localhost:8080/rk/v1/info
{
    "appName":"rk-demo",
    "version":"master-f414049",
    "description":"this is description",
    "keywords":[
        "rk",
        "golang"
    ],
    "homeUrl":"http://example.com",
    "iconUrl":"http://example.com",
    "docsUrl":[
        "http://example.com"
    ],
    "maintainers":[
        "rk-dev"
    ],
    "uid":"501",
    "gid":"20",
    "username":"XXX",
    "startTime":"2021-07-08T00:38:51+08:00",
    "upTimeSec":13,
    "upTimeStr":"13 seconds",
    "region":"",
    "az":"",
    "realm":"",
    "domain":""
}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 3.从 AppInfoEntry 实例中访问
> rkentry.GlobalAppCtx.GetAppInfoEntry()

```go
type AppInfoEntry struct {
	EntryName        string   `json:"entryName" yaml:"entryName"`
	EntryType        string   `json:"entryType" yaml:"entryType"`
	EntryDescription string   `json:"description" yaml:"description"`
	AppName          string   `json:"appName" yaml:"appName"`
	Version          string   `json:"version" yaml:"version"`
	Lang             string   `json:"lang" yaml:"lang"`
	Keywords         []string `json:"keywords" yaml:"keywords"`
	HomeUrl          string   `json:"homeUrl" yaml:"homeUrl"`
	IconUrl          string   `json:"iconUrl" yaml:"iconUrl"`
	DocsUrl          []string `json:"docsUrl" yaml:"docsUrl"`
	Maintainers      []string `json:"maintainers" yaml:"maintainers"`
	License          string   `json:"-" yaml:"-"`
	Readme           string   `json:"-" yaml:"-"`
	GoMod            string   `json:"-" yaml:"-"`
	UtHtml           string   `json:"-" yaml:"-"`
}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)