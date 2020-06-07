## Lab 5: Working with Structures and Files

## Introduction

In this lab you will become familar with Go struct and methods, along with working with dependencies. 

* Structs 
* JSON Encoding / Decoding
* Files

> **Note:** This lab assumes you have a solution for lab 3 as a starting point.

## References

* OpenWeather Map:  https://openweathermap.org/current
* JSON Encoding: https://golang.org/pkg/encoding/json/

## Steps

#### 1: OpenWeather Map Key

If you haven't yet... signup for access to [OpenWeather Map](https://openweathermap.org/current) which is free.  We will need it for the next lab but will be referencing your api key in this lab.

#### 2: create a configuration struct

create a `weather` package under `pkg`, then create a `types.go` file:

```go
package weather

type UnitType string

const (
	Fahrenheit UnitType = "fahrenheit"
	celsius             = "celsius"
)

type WeatherConfig struct {
	Key    string   `json:"key"`
	Unit   UnitType `json:"unit"`
	Cities []string `json:"cities"`
}
```

**notes:** 
1. `json` directive used to marshal data from json
2. `json:",omitempty"` can be used to allow for unspecified

#### 3: create example config file

At root of project create a `config.yaml` file:

```yaml
key: key
unit: fahrenheit
cities:
  - Chicago
  - London
  - Paris
  - Austin
```

#### 4: create func to read config

In the `weather` package create a `config.go` file:

```go
package weather

import (
	"io/ioutil"

	"gopkg.in/yaml.v2"
)

// GetConfig retrieves a WeatherConfig from a file
func GetConfig(file string) (*WeatherConfig, error) {

	wc := &WeatherConfig{}
	yamlFile, err := ioutil.ReadFile(file)
	if err != nil {
		return nil, err
	}
	err = yaml.Unmarshal(yamlFile, wc)
	if err != nil {
		return nil, err
	}

	return wc, nil
}
```

**note:**
1. `gopkg.in/yaml.v2 v2.2.2` will need to be added to go.mod

#### 5: create config cmd

In `cmd` package create a `config.go` file:

```go
package cmd

import (
	"fmt"

	"github.com/spf13/cobra"

	"github.com/codementor/wman/pkg/weather"
)

var (
	configExample = `  # Display current configuration
  wman config`
)

// newConfigCmd returns a new initialized instance of the config sub command
func newConfigCmd() *cobra.Command {

	cmd := &cobra.Command{
		Use:     "config",
		Short:   "Displays the current configuration",
		Example: configExample,
		RunE:    ConfigCmd,
	}

	return cmd
}

// ConfigCmd performs the print config command
func ConfigCmd(cmd *cobra.Command, args []string) error {

	wc, err := weather.GetConfig("config.yaml")
	if err != nil {
		return err
	}
	fmt.Print(wc)
	return nil
}
```

Now connect it to the root cmd and run it:
`go run cmd/wman/main.go config`
You should see something like:
```shell
go run cmd/wman/main.go config
&{key fahrenheit [Chicago London Paris Austin]}
```

#### 6: improving the output

Take a crack at improving the output displayed.

#### 7: linting

`make lint`

```shell
make lint
golangci-lint run
pkg/weather/types.go:10:6: type name will be used as weather.WeatherConfig by other packages, and that stutters; consider calling this Config (golint)
type WeatherConfig struct {
     ^
pkg/weather/types.go:6:2: SA9004: only the first constant in this group has an explicit type (staticcheck)
	Fahrenheit UnitType = "fahrenheit"
	^
make: *** [lint] Error 1
```

Fixing unit type:

```go
const (
	Fahrenheit UnitType = "fahrenheit"
	Celsius    UnitType = "celsius"
)
```

Fixing WeatherConfig

```go
type Config struct {
	Key    string   `json:"key"`
	Unit   UnitType `json:"unit"`
	Cities []string `json:"cities"`
}
```

**note:** this can seem unsual to a Java dev... The meaningful name of a struct includes its package.   This is read as `weather.Config`.  The change to this struct requires changes in all callers.  We write code and name structs in Go from the calling client perspective and as much as possible without "stuttering".  `weather.WeatherConfig` is an example of a strutter we try to avoid.