## Lab 6: HTTP Get Requests

## Introduction

In this lab you will level up on Go structs and our use of Cobra CLI,  along with learning how to make HTTP Get requests.

* Structs 
* JSON Encoding / Decoding
* HTTP Requests

> **Note:** This lab assumes you have a solution for lab 5 as a starting point.

## References

* OpenWeather Map:  https://openweathermap.org/current
* JSON Encoding: https://golang.org/pkg/encoding/json/

## Steps

#### 1: Update Config File

Using OpenWeather Map, update the `config.yaml` with your key. (if you are pushing your work to github, don't push you api key).
We will make the config file, configurable in this lesson. Making it possible to check in your example file and keeping your "production" configuration separate.

example:
```yaml
key: 50d643dfd26b8b293a63d953f078abba
unit: fahrenheit
cities:
  - Chicago
  - London
  - Paris
  - Austi
```

#### 2: Create weather command and sub commands

The goal is to create a command for:

* `wman weather get <city>`
* `wman weather list`  (for future lab)

Addinng weather command with `weather get` command:

```go
package cmd

import (
	"github.com/spf13/cobra"
)

const weatherDesc = `
This command consists of multiple sub-commands to interact with weather for open weather map.

There is a option for retrieving weather for a single city or for a group of cities.
`

const (
	weatherGetExample = `  # Retrieving weather for a city
  wman weather get [city]
`
)

type weatherOptions struct {
	config string
}

// newWeatherCmd returns a new initialized instance of the weather sub command
func newWeatherCmd() *cobra.Command {
	opts := &weatherOptions{}
	cmd := &cobra.Command{
		Use:   "weather",
		Short: "Displays different options for weather",
		Long:  weatherDesc,
	}
	cmd.PersistentFlags().StringVarP(&opts.config, "config", "c", "config.yaml", "The config to use for weather.")
	cmd.AddCommand(newWeatherGetCmd())
	return cmd
}

// newWeatherGetCmd creates a command that shows the weather for a city.
func newWeatherGetCmd() *cobra.Command {

	cmd := &cobra.Command{
		Use:     "get",
		Short:   "Displays the weather for a city.",
		Example: weatherGetExample,
		RunE: func(cmd *cobra.Command, args []string) error {
			return nil
		},
	}

	return cmd
}

```
and connect into "root" command

Make sure to:
* lint
* and test `go run cmd/wman/main.go  weather`

#### 3: Weather Models

To `types.go` in the weather package, lets add the structures need to parse output from [open weather map](https://openweathermap.org/current).  Example call:  https://samples.openweathermap.org/data/2.5/weather?id=2172797&appid=439d4b804bc8187953eb36d2a8c26a02

```go
type main struct {
	Temp float64 `json:"temp"`

}

type description struct {
	ID int `json:"id"`
	Desc string `json:"description"`
}

type weather struct {
	Name string `json:"name"`
	Main main `json:"main"`
	Weathers []description `json:"weather"`
}
```

**note:**
* There is more data if you want it. This defines the minimum data required for this lab.
* The structures are design to be encapsulated to this package only.


Lets define the mode we want to use throughout the app in the same file:

```go
// App weather structure
type Model struct {
	City string
	Temp float64
	Desc string
}
```

#### 4: Making HTTP Requests

First, we will need the url which is based at `url := "https://api.openweathermap.org/data/2.5/weather?"

We will need `appid` and `units` from the config.
We will need `city` which is passed in.

In the `weather` package, create a `weather.go` file.

For this use case, it is likely that once we have configured a way to communicate to open weather API that we may want to make multiple queries using the same configuration.

Lets start out with a constant for the base URL, a struct we will use as the configuration to fetch weather and lets control the way to create an instance of this Fetcher.

```go
package weather

import (
	"encoding/json"
	"errors"
	"fmt"
	"net/http"
	"strings"
)

const (
	wurl = "https://api.openweathermap.org/data/2.5/weather"
)

type Fetcher struct {
	url string
}

func New(config *Config) (*Fetcher, error) {

	if strings.TrimSpace(config.Key) == "" {
		return nil, fmt.Errorf("apikey required")
	}
	// open weather defines units as metric or imperial
	var units string
	switch config.Unit {
	case Celsius:
		units = "metrics"
	case Fahrenheit:
		fallthrough
	default:
		units = "imperial"
	}

	return &Fetcher{
		url: fmt.Sprintf("%s?appid=%s&units=%s", wurl, config.Key, units),
	}, nil
}
```

The `New` method is our only way to create an instance of Fetcher as the `url` is only accessible to this package.  With the provided `Config`, the contructor method will check for an "apikey", and convert the units types to the weather API requirements.

Notice the construction of the url through `Sprintf`, this is common to see in the wild.  We will see another way to do this later in this lab.

We also are working with the weather API structs and our app structs, so lets build a converter method to handle this.

```go
func convertToModel(w *weather) *Model {
	desc := ""
	if len(w.Weathers) > 0 {
		desc = w.Weathers[0].Desc
	}
	return &Model{
		City: w.Name,
		Temp: w.Main.Temp,
		Desc: desc,
	}
}
```

Now lets make the actually HTTP method on `Fetcher`:

```go
func (f *Fetcher) Get(city string) (*Model, error) {

	// adding city to the query str
	req, _ := http.NewRequest("GET", f.url, nil)
	req.Header.Add("Accept", "application/json")
	q := req.URL.Query()
	q.Add("q", city)
	req.URL.RawQuery = q.Encode()
	// at this point we have a query with city added

	client := http.Client{}
	r, err := client.Do(req)
	if err != nil {
		return nil, err
	}
	defer r.Body.Close()

	// anything other than 200 is an error to us
	// we also know that 401 is an author issue that we can be app specific about.
	switch r.StatusCode {
	case 200:
	case 401:
		return nil, errors.New("Not Authorized.  Bad or missing key.")
	default:
		return nil, fmt.Errorf("Unknown HTTP status %d", r.StatusCode)
	}

   // here a reference to weather is passed to `Decode` which is a common thing to see in Go.
	weather := new(weather)
	json.NewDecoder(r.Body).Decode(weather)

	return convertToModel(weather), nil
}
```

#### 5: Connecting to the Commandline 

Switching attention back to `weather.go` in the `cmd` package.

In `newWeatherGetCmd` function, we are going to replace the following:

```go
RunE: func(cmd *cobra.Command, args []string) error {
			return nil
		},
```

with the follow error handling:

```go
		RunE: func(cmd *cobra.Command, args []string) error {
			file, err := cmd.Flags().GetString("config")
			if err != nil {
				return err
			}
			config, err := weather.GetConfig(file)
			if err != nil {
				return err
			}
			if len(args) < 1 {
				return fmt.Errorf("city must be provided")
			}
			return printCityWeather(config, args[0])
		},
```

Next we need to provide the `printCityWeather` function:

```go
func printCityWeather(config *weather.Config, city string) error {
	f, err := weather.New(config)
	if err != nil {
		return err
	}
	m, err := f.Get(city)
	if err != nil {
		return err
	}
	fmt.Println("weather: ", m)
	return nil
}
```

With that you should be able to test the app with `go run cmd/wman/main.go  weather get Paris`

```shell
go run cmd/wman/main.go  weather get Paris
weather:  &{Paris 62.22 scattered clouds}
```

test it with a known unknown:

```shell
go run cmd/wman/main.go  weather get Foobar
Error: Unknown HTTP status 404
exit status 255
```

Perhaps you can add more error handling for 404.

#### 6: More linting

```shell
make lint
golangci-lint run
pkg/weather/weather.go:65:32: Error return value of `(*encoding/json.Decoder).Decode` is not checked (errcheck)
	json.NewDecoder(r.Body).Decode(weather)
	                              ^
pkg/weather/weather.go:59:26: error strings should not be capitalized or end with punctuation or a newline (golint)
		return nil, errors.New("Not Authorized.  Bad or missing key.")
		                       ^
make: *** [lint] Error 1 
```

1. fixing error strings

```go
	case 401:
		return nil, errors.New("not Authorized.  bad or missing key")
```

2. fixing decode

```go
	err = json.NewDecoder(r.Body).Decode(weather)
	if err != nil {
		return nil, err
	}
```

#### 7: Working with Configs

you should also be able to switch config files

```shell
go run cmd/wman/main.go  weather get Paris --config config.yaml
weather:  &{Paris 61.93 scattered clouds}
```

vs. 

```shell
go run cmd/wman/main.go  weather get Paris --config con
Error: config file does not exist
exit status 255
```

