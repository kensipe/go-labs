## Lab 7: Go Routines and Chans

## Introduction

In this lab you will reinforce learning around goroutines and channels making concurrent HTTP requests. 

* struct sorting
* go routines
* channels

> **Note:** This lab assumes you have a solution for lab 6 as a starting point.

## References

* OpenWeather Map:  https://openweathermap.org/current
* JSON Encoding: https://golang.org/pkg/encoding/json/

## Steps

#### 1: Adding more cities to `weather get city`

The goal of this step is to make the following possible `weather get Paris Chicago London`.
There are a number of ways to manage this. It is common to have a preference to limit work in `cmd` package to command only and move out logic to the package specific to that function.  For the `weather.go` in `cmd` lets pass the full []string of `args` to the `printCityWeather`

Change `return printCityWeather(config, args[0])` to `return printCityWeather(config, arg)`

Change `func printCityWeather(config *weather.Config, city string) error {` to `func printCityWeather(config *weather.Config, cities []string) error {`

Now we are left with options.  We could call `f.Get(city)` mutiple times.  If we did, that would place concurrency logic and potentially a lot of error handling here in the `cmd` packages.  We could send `[]string` to `Get`.  This would have the consequnce of complicating that function which is fairly myoptic now.  We might want to "overload" that `Get` but we can't in Go.

Lets add a `GetCities` method to the `Fetcher` which will return `[]*Model`.  In `weather.go` in the weather package add:

```go
func (f *Fetcher) GetCities(cities []string) ([]*Model, error) {
	ml := make([]*Model, 0)
	for _, city := range cities {
		m, err := f.Get(city)
		if err != nil {
			return nil, err
		}
		ml = append(ml, m)
	}
	return ml, nil
}

```

Lets change the `printCityWeather` function in the `weather.go` in the `cmd` package.

```go
    // changed from GetCity to GetCities which returns an array
	mList, err := f.GetCities(cities)
	if err != nil {
		return err
	}

	table := uitable.New()
	table.AddRow("City", "Temp", "Desc")
	// which requires use to iterator the list
	for _, m := range mList {
		table.AddRow(m.City, m.Temp, m.Desc)	
	}

```

Testing:

```shell
go run cmd/wman/main.go  weather get Paris
City 	Temp 	Desc            
Paris	59.67	scattered clouds
```

```shell
go run cmd/wman/main.go  weather get Paris Chicago
City   	Temp 	Desc            
Paris  	59.88	scattered clouds
Chicago	78.26	few clouds  
```

#### 2: Sorting

Lets looking at sorting output.  Add the following after `mList` is assigned (and before it is added to the table).

```go
sort.Sort(mList)
```
**note:** this is from `import "sort"`

It won't compile.  The editor indicates:

```go
Cannot use 'mList' (type []*Model) as type Interface Type does not implement 'Interface' as some methods are missing: 

Len() int 
Less(i int, j int) 
bool Swap(i int, j int) 
```

compile indicates something similar 
```shell
make cli
go build -o bin/wman ./cmd/wman/main.go
# github.com/codementor/wman/pkg/cmd
pkg/cmd/weather.go:78:11: cannot use mList (type []*weather.Model) as type sort.Interface in argument to sort.Sort:
	[]*weather.Model does not implement sort.Interface (missing Len method)
make: *** [cli] Error 2
``` 

What is this `sort.Interface` the compile speaks of?

```shell
go doc sort.Interface
package sort // import "sort"

type Interface interface {
	// Len is the number of elements in the collection.
	Len() int
	// Less reports whether the element with
	// index i should sort before the element with index j.
	Less(i, j int) bool
	// Swap swaps the elements with indexes i and j.
	Swap(i, j int)
}
```

In order to sort this collection, we need to have the model implement this interface.  A technique for helping with this is to create a "dummy" var for that interface against the data type you want to sort.

In `types.go` in the `weather` package add:

```go
type Models []Model

var _ sort.Interface = &Models{}
```

At this point there is a compiler error as Models does NOT implement the `sort.Interface`.  In GoLand, you are given the option of "implement missing methods", which adds the templated code we need.  Here are the completed functions:

```go
func (m Models) Len() int {
	return len(m)
}

func (m Models) Less(i, j int) bool {
	// what do you want to sort on?  city?  temperature?
	return m[i].City < m[j].City
}

func (m Models) Swap(i, j int) {
	m[i], m[j] = m[j], m[i]
}

```

This will require some refactoring:
change `func (f *Fetcher) GetCities(cities []string) ([]*Model, error) {`  to `func (f *Fetcher) GetCities(cities []string) (Models, error) {`

which changes a couple more things in that function:
```go
   // lets make the Models instead of []Model
	ml := make(Models, 0)
	for _, city := range cities {
		m, err := f.Get(city)
		if err != nil {
			return nil, err
		}
		// this is a collection of model not *model requires a deref of model pointer
		ml = append(ml, *m)
	}
	return ml, nil

```

Testing:

```shell
 go run cmd/wman/main.go  weather get Paris Chicago
City   	Temp 	Desc            
Chicago	78.26	few clouds      
Paris  	59.74	scattered clouds
```

Now the output is sorted by city name!



#### 3: Execution Time

In `GetCities` function in `weather.go` add details on length of time of execution:

```go
// start of function
	start := time.Now()

// just prior to return
	elapsed := time.Since(start)
	fmt.Printf("GetCities took %s\n", elapsed)

```

Get a feel for how the length of time is dependent on number of arguments such as:

```shell
go run cmd/wman/main.go  weather get Paris Chicago London Singapore Austin Oslo Stockholm
GetCities took 555.499471ms
```

For me, 1 call is ~ 310ms and 7 calls is ~ 525ms AND it the more added the longer the time.

#### 4: Go Go Go routine

Within `GetCities` function we worked with above.

Lets put the `f.Get` call in a "go func" as in:

```go
		go func() {
			m, err := f.Get(city)
			if err != nil {
				return nil, err
			}
			ml = append(ml, *m)
		}()
```

What we find is that the `return nil, err` is not appropriate any more.  Error handling from a separate thread needs to be designed.  For now lets ignore it (not recommended for production but is a longer subject beyond this lab.)

```go
		go func(city string) {
			m, _ := f.Get(city)
			ml = append(ml, *m)
		}(city)
```

Lets run it!

```shell
go run cmd/wman/main.go  weather get Paris Chicago London Singapore Austin Oslo Stockholm
GetCities took 12.311µs
City	Temp	Desc
```

On the plus side, it is way fast!!  12µs!   On the downside, there is no output.  why?


Logical steps to manage this:
1. setup a chan to communicate between threads
2. add the model result to the chan
3. loop pulling from the chan until we get them all back

```go
	// channel for results in mchan or model channel
	mchan := make(chan *Model)
	for _, city := range cities {

		go func(city string) {
			m, _ := f.Get(city)
			// put the model in the channel
			mchan <- m
		}(city)
	}

	for {
		// put models from model channel
		m := <-mchan
		ml = append(ml, *m)

		// if we get all the cities back we can break
		if len(ml) == len(cities) {
			break
		}
	}
```

Testing...

```shell
go run cmd/wman/main.go  weather get Paris Chicago London Singapore Austin Oslo Stockholm
GetCities took 338.38363ms
```

Now most calls are ~300ms


#### 5: Hey Wait!

Many developers new to Go immediately want to use chans.  The number of implementations using chans used for Timeout is high. The Go core libraries have evolved to include a number of concepts backed in. For instance to work with timeouts, consider `context` with `WithDeadline`. https://golang.org/pkg/context/

Lets modify the `GetCities` again and use the `sync.WaitGroup`
For simple cases where a thread is spun off and there is no state to return, the `WaitGroup` would remove channels altogether, and yet would enable the ability to wait until all work is complete before moving on or exiting.  For cases like this one, there is still a need to return a value for which there are a few approaches; 1. we could pass a reference of the collection to the go func and have it function add it's model. or 2. we could continue to use a buffered channel and iterate the channel after the work is complete.

```go
func (f *Fetcher) GetCities(cities []string) (Models, error) {
	start := time.Now()
  
    // create the WaitGroup
	var wg sync.WaitGroup
	// set the wait size to size of expected threads
	wg.Add(len(cities))
	ml := make(Models, 0)

	// channel for results in mchan or model channel
	// set the buffer size to the size of returns
	mchan := make(chan Model, len(cities))
	for _, city := range cities {

		// pass in waitgroup and write only chan
		go func(city string, wait *sync.WaitGroup, ch chan<- Model) {
			// defer the done on waitgroup... when this thread is done we alway decrement the wait.
			defer wait.Done()

			m, _ := f.Get(city)
			// put the model in the channel
			ch <- *m

		}(city, &wg, mchan)
	}

	// wait here until the count is 0
	wg.Wait()
	// if we are going to iterate a chan, we must close it
	close(mchan)
	for model := range mchan {
		ml = append(ml, model)
	}

	elapsed := time.Since(start)
	fmt.Printf("GetCities took %s\n", elapsed)
	return ml, nil
}
```