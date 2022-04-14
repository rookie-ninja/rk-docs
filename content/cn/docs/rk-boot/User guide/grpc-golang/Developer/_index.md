---
title: "开发者指南"
linkTitle: "开发者指南"
weight: 3
description: >
  如何创建自定义 Entry?
---

## 概述
每一个启动器默认的 Entry 都实现了 [Entry](https://github.com/rookie-ninja/rk-entry/blob/master/entry/entry.go) 的接口。

```go
// Entry interface which must be implemented for bootstrapper to bootstrap
type Entry interface {
	// Bootstrap entry
	Bootstrap(context.Context)

	// Interrupt entry
	// Wait for shutdown signal and wait for draining incomplete procedure
	Interrupt(context.Context)

	// Get name of entry
	GetName() string

	// Get type of entry
	GetType() string

	// Get description of entry
	GetDescription() string

	// print entry as string
	String() string
}
```

## 快速开始
> Entry 可以是任意形式的，比如在线服务，一个库，一个周期性任务。
> 
> 第三方的 Entry 可以通过 boot.yaml 文件，注入到启动器中。
> 
> **如何创建自定义 Entry?**
> 
> 1. 自定义 Entry 的 YAML 格式
> 2. 实现 Entry 接口
> 3. 实现 EntryRegFunc
> 4. 创建 init() 函数，init() 函数中，需要把已经实现的 EntryRegFunc 注册到 GlobalAppCtx 中
> 
> **Entry 如何与启动器互动？**
> 1. Entry 将会通过 EntryRegFunc 被创建，并且注册到 rkentry.GlobalAppCtx
> 2. 启动器将会调用 Entry 的 Bootstrap() 函数
> 3. 服务将会等待 shutdown signal
> 4. 启动器将会 调用 Interrupt() 函数

### 1.实现 Entry
```go
type MyEntry struct {
	EntryName        string                    `json:"entryName" yaml:"entryName"`
	EntryType        string                    `json:"entryType" yaml:"entryType"`
	EntryDescription string                    `json:"entryDescription" yaml:"entryDescription"`
}

func (entry *MyEntry) Bootstrap(context.Context) {}

func (entry *MyEntry) Interrupt(context.Context) {}

func (entry *MyEntry) GetName() string {
	return entry.EntryName
}

func (entry *MyEntry) GetType() string {
	return entry.EntryType
}

func (entry *MyEntry) GetDescription() string {
	return entry.EntryDescription
}

func (entry *MyEntry) String() string {
	bytes, _ := json.Marshal(entry)
	return string(bytes)
}
```

### 2.创建 bootstrapper config
> 这个结构与 boot.yaml 文件中的 **myEntry** 部分一致。

```go
// A struct which is for unmarshalled YAML
type BootConfig struct {
	MyEntry struct {
		Enabled     bool   `yaml:"enabled" json:"enabled"`
		Name        string `yaml:"name" json:"name"`
		Description string `yaml:"description" json:"description"`
	} `yaml:"myEntry" json:"myEntry"`
}
```

```yaml
myEntry:
  enabled: true
  name: "xxx"
  descriptin: "xxx"
```

### 3.实现 EntryRegFunc
> 虽然我们只需要实现 **RegisterMyEntriesFromConfig()** 函数，我们强烈推荐以如下的方式实现，因为如下的方式可以同时提供，通过代码方式启动的功能。

```go
// An implementation of:
// type EntryRegFunc func(string) map[string]rkentry.Entry
func RegisterMyEntriesFromConfig(configFilePath string) map[string]rkentry.Entry {
	res := make(map[string]rkentry.Entry)

	// 1: decode config map into boot config struct
	config := &BootConfig{}
	rkcommon.UnmarshalBootConfig(configFilePath, config)

	// 3: construct entry
	if config.MyEntry.Enabled {
		entry := RegisterMyEntry(
			WithName(config.MyEntry.Name),
			WithDescription(config.MyEntry.Description),
		res[entry.GetName()] = entry
	}

	return res
}

func RegisterMyEntry(opts ...MyEntryOption) *MyEntry {
	entry := &MyEntry{
		EntryName:        "myEntry",
		EntryType:        "myEntry",
		EntryDescription: "Please contact maintainers to add description of this entry.",
	}

	for i := range opts {
		opts[i](entry)
	}

	if len(entry.EntryName) < 1 {
		entry.EntryName = "my-default"
	}

	if len(entry.EntryDescription) < 1 {
		entry.EntryDescription = "Please contact maintainers to add description of this entry."
	}

	rkentry.GlobalAppCtx.AddEntry(entry)

	return entry
}

type MyEntryOption func(*MyEntry)

func WithName(name string) MyEntryOption {
	return func(entry *MyEntry) {
		entry.EntryName = name
	}
}

func WithDescription(description string) MyEntryOption {
	return func(entry *MyEntry) {
		entry.EntryDescription = description
	}
}
```

### 4.创建 init() function
> 把你的注册函数注册到 rkentry 中。

```go
func init() {
	rkentry.RegisterEntryRegFunc(RegisterMyEntriesFromConfig)
}
```

### 5.完整例子
> boot.yaml
```yaml
---
myEntry:
  enabled: true
  name: my-entry
  description: "This is my entry."
```
> main.go
```go
package main

import (
	"context"
	"encoding/json"
	"fmt"
	"github.com/rookie-ninja/rk-boot"
	"github.com/rookie-ninja/rk-common/common"
	"github.com/rookie-ninja/rk-entry/entry"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Bootstrap
	boot.Bootstrap(context.Background())

	fmt.Println(rkentry.GlobalAppCtx.GetEntry("my-entry"))

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}


// Register entry, must be in init() function since we need to register entry at beginning
func init() {
	rkentry.RegisterEntryRegFunc(RegisterMyEntriesFromConfig)
}

// A struct which is for unmarshalled YAML
type BootConfig struct {
	MyEntry struct {
		Enabled     bool   `yaml:"enabled" json:"enabled"`
		Name        string `yaml:"name" json:"name"`
		Description string `yaml:"description" json:"description"`
	} `yaml:"myEntry" json:"myEntry"`
}

// An implementation of:
// type EntryRegFunc func(string) map[string]rkentry.Entry
func RegisterMyEntriesFromConfig(configFilePath string) map[string]rkentry.Entry {
	res := make(map[string]rkentry.Entry)

	// 1: decode config map into boot config struct
	config := &BootConfig{}
	rkcommon.UnmarshalBootConfig(configFilePath, config)

	// 3: construct entry
	if config.MyEntry.Enabled {
		entry := RegisterMyEntry(
			WithName(config.MyEntry.Name),
			WithDescription(config.MyEntry.Description))
		res[entry.GetName()] = entry
	}

	return res
}

func RegisterMyEntry(opts ...MyEntryOption) *MyEntry {
	entry := &MyEntry{
		EntryName:        "default",
		EntryType:        "myEntry",
		EntryDescription: "Please contact maintainers to add description of this entry.",
	}

	for i := range opts {
		opts[i](entry)
	}

	if len(entry.EntryName) < 1 {
		entry.EntryName = "my-default"
	}

	if len(entry.EntryDescription) < 1 {
		entry.EntryDescription = "Please contact maintainers to add description of this entry."
	}

	rkentry.GlobalAppCtx.AddEntry(entry)

	return entry
}

type MyEntryOption func(*MyEntry)

func WithName(name string) MyEntryOption {
	return func(entry *MyEntry) {
		entry.EntryName = name
	}
}

func WithDescription(description string) MyEntryOption {
	return func(entry *MyEntry) {
		entry.EntryDescription = description
	}
}

type MyEntry struct {
	EntryName        string                    `json:"entryName" yaml:"entryName"`
	EntryType        string                    `json:"entryType" yaml:"entryType"`
	EntryDescription string                    `json:"entryDescription" yaml:"entryDescription"`
}

func (entry *MyEntry) Bootstrap(context.Context) {}

func (entry *MyEntry) Interrupt(context.Context) {}

func (entry *MyEntry) GetName() string {
	return entry.EntryName
}

func (entry *MyEntry) GetDescription() string {
	return entry.EntryDescription
}

func (entry *MyEntry) GetType() string {
	return entry.EntryType
}

func (entry *MyEntry) String() string {
	bytes, _ := json.Marshal(entry)
	return string(bytes)
}
```

### 5.验证
```shell script
$ go run main.go
{"entryName":"my-entry","entryType":"myEntry","entryDescription":"This is my entry."}
```
