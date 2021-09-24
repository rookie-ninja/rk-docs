---
title: "Locale"
linkTitle: "Locale"
weight: 2
description: >
  Distinguish entries based on different environment.
---

## Overview
Lets assuming we need to use different config files based on the region like Singapore or Frankfurt.

Since bootstrapper support multiple ConfigEntries in boot.yaml, we need a mechanism to distinguish ConfigEntries.


## Concept
> **How entries selected by bootstrapper?**
> 
> RK use REALM, REGION, AZ, DOMAIN environment variable to distinguish environments. 

![](/bootstrapper/user-guide/gin-golang/advanced/locale-arch.png)

## Quick start
### 1.Create config files
**config/singapore.yaml**
```yaml
---
region: sinpapore
```
**config/frankfurt.yaml**
```yaml
---
region: frankfurt
```
**config/default.yaml**
```yaml
---
region: default
```

### 2.Create boot.yaml
```yaml
config:
  # Load this config if nothing matched
  - name: my-config
    locale: "*::*::*::*"
    path: config/default.yaml
  # Load this config if REGION=singapore
  - name: my-config
    locale: "*::singapore::*::*"
    path: config/singapore.yaml
  # Load this config if REGION=frankfurt
  - name: my-config
    locale: "*::frankfurt::*::*"
    path: config/frankfurt.yaml
gin:
  - name: greeter
    port: 8080
    enabled: true
```

### 3.With REGION=singapore
```go
package main

import (
	"context"
	"fmt"
	"github.com/rookie-ninja/rk-boot"
	"os"
)

// Application entrance.
func main() {
    // Set REGION=singapore
	os.Setenv("REGION", "singapore")

	// Create a new boot instance.
	boot := rkboot.NewBoot()

	fmt.Println(boot.GetConfigEntry("my-config").GetViper().GetString("region"))

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```
> Output
```shell script
singapore
```

### 4.With REGION=frankfurt
```go
package main

import (
	"context"
	"fmt"
	"github.com/rookie-ninja/rk-boot"
	"os"
)

// Application entrance.
func main() {
    // Set REGION=singapore
	os.Setenv("REGION", "frankfurt")

	// Create a new boot instance.
	boot := rkboot.NewBoot()

	fmt.Println(boot.GetConfigEntry("my-config").GetViper().GetString("region"))

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```
> Output
```shell script
frankfurt
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 5.With REGION=not-matched
```go
package main

import (
	"context"
	"fmt"
	"github.com/rookie-ninja/rk-boot"
	"os"
)

// Application entrance.
func main() {
    // Set REGION=singapore
	os.Setenv("REGION", "not-matched")

	// Create a new boot instance.
	boot := rkboot.NewBoot()

	fmt.Println(boot.GetConfigEntry("my-config").GetViper().GetString("region"))

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```
> Output
```shell script
default
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 6.With REGION=""
```go
package main

import (
	"context"
	"fmt"
	"github.com/rookie-ninja/rk-boot"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	fmt.Println(boot.GetConfigEntry("my-config").GetViper().GetString("region"))

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```
> Output
```shell script
default
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)