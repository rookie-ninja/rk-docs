---
title: "AppInfo"
linkTitle: "AppInfo"
weight: 6
description: >
  How to specify application info in boot.yaml?
---

## Overview
Application info consist of bellow fields.

```yaml
app:
  description: "this is description"  # Optional, default: ""
  keywords: ["rk", "golang"]          # Optional, default: []
  homeUrl: "http://example.com"       # Optional, default: ""
  iconUrl: "http://example.com"       # Optional, default: ""
  docsUrl: ["http://example.com"]     # Optional, default: []
  maintainers: ["rk-dev"]             # Optional, default: []
```

## Quick start
```yaml
app:
  description: "this is description"  # Optional, default: ""
  keywords: ["rk", "golang"]          # Optional, default: []
  homeUrl: "http://example.com"       # Optional, default: ""
  iconUrl: "http://example.com"       # Optional, default: ""
  docsUrl: ["http://example.com"]     # Optional, default: []
  maintainers: ["rk-dev"]             # Optional, default: []
gf:
  - name: greeter
    port: 8080
    enabled: true
    commonService:
      enabled: true                   # Enable for validation
    tv: 
      enabled: true                   # Enable for validation
```

### 1.Access from RK TV
> [http://localhost:8080/rk/v1/tv/info](http://localhost:8080/rk/v1/tv/info)

![](/bootstrapper/user-guide/gf-golang/advanced/app-info.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 2.Access from common service
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

### 3.Access from AppInfoEntry
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