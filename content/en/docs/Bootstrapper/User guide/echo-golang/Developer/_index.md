---
title: "Developer"
linkTitle: "Developer"
weight: 2
description: >
  How to implement a custom entry?
---

## Overview
Every entries in builtin bootstrapper implement [Entry](https://github.com/rookie-ninja/rk-entry/blob/master/entry/entry.go) interface in [rk-entry](https://github.com/rookie-ninja/rk-entry) package.

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

## Quick start
> An entry could be any kinds of services or pieces of codes which needs to be bootstrap/initialized while application starts.
> 
> A third party entry could be implemented and inject to rk-boot via boot.yaml file.
> 
> **How to create a new custom entry?**
> 
> 1. Construct your own entry YAML struct as needed
> 2. Create a struct which implements Entry interface
> 3. Implements EntryRegFunc
> 4. Register your reg function in init() in order to register your entry while application starts
> 
> **How entry interact with rk-boot.Bootstrapper?**
> 1. Entry will be created and registered into rkentry.GlobalAppCtx
> 2. Bootstrap will be called from Bootstrapper.Bootstrap() function
> 3. Application will wait for shutdown signal
> 4. Interrupt will be called from Bootstrapper.Interrupt() function

### 1.Implement Entry
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

### 2.Create bootstrapper config
> This structure will match **myEntry** section in boot.yaml file

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

### 3.Implements EntryRegFunc
> Although we only need **RegisterMyEntriesFromConfig()** function, we strongly recommend implementing as bellow in order to run our entry from code either.

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

### 4.Create init() function
> Register your reg function in init() in order to register your entry while application starts

```go
func init() {
	rkentry.RegisterEntryRegFunc(RegisterMyEntriesFromConfig)
}
```

### 5.Full example
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

### 5.Validate
```shell script
$ go run main.go
{"entryName":"my-entry","entryType":"myEntry","entryDescription":"This is my entry."}
```
