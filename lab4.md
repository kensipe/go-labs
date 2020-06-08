## Lab 4: Working with Data Types and Linting

## Introduction

Starting with lab 3, we will extend a command to calculate dog years. In the process we will need to convert strings to int and back.  We will work with STDIN for input and provide error handling. Then we will take a deeper look at linting.

* STDIN
* error handling
* data conversions
* linting
* build tags
* variable tests

> **Note:** This lab assumes you have a solution for lab 3 as a starting point.

## Steps

#### 1: create a `cmd/dogyears.go`

```go
package cmd

import (
	"github.com/spf13/cobra"
)

var (
	dogYearExample = `  # Calculates dog years
  wman dogyears`
)


// newDogYearCmd returns a new initialized instance of the dogyear sub command
func newDogYearCmd() *cobra.Command {

	cmd := &cobra.Command{
		Use:     "dogyear",
		Short:   "Calculates dogyears",
		Example: dogYearExample,
		RunE: DogYearCmd,
	}

	return cmd
}

// PrintCmd performs the print sub command
func DogYearCmd(cmd *cobra.Command, args []string) error {

	return nil
}

```

#### 2: add dogyear cmd to root

`cmd.AddCommand(newDogYearCmd())`


#### 3: calculate dogyears with error handling

```go
	fmt.Println("Enter Age:")
	reader := bufio.NewReader(os.Stdin)
	a, err := reader.ReadString('\n')
	if errors.Is(err, bufio.ErrBufferFull) {
		return fmt.Errorf("buffer full %w", err)
	}
	if err != nil {
		return err
	}
	a = strings.TrimSpace(a)
	age, err := strconv.Atoi(a)
	if err != nil {
		return err
	}
	dogyears := age * 7

	fmt.Printf("your age of %d is %v in dog years\n", age, dogyears)
	return nil
```

**notes:**
1. main logic flow in Go is focused left most indented, error handling is intended
2. review `errors.Is` code and the `fmt.Errorf`
3. what is the `%w`


#### 4: run the command 

`go run cmd/wman/main.go dogyear`


#### 5: create `cmd/lint.go`

```go
package cmd

import (
	"fmt"
	"regexp"

	"github.com/spf13/cobra"
)

var (
	lintExample = `  # linting is what we do
  wman lint`
)

// newLintCmd returns a new initialized instance of the lint sub command
func newLintCmd() *cobra.Command {

	cmd := &cobra.Command{
		Use:     "lint",
		Short:   "Linting exercise",
		Example: lintExample,
		RunE: lintTest,
	}

	return cmd
}

type T struct {
	Field1 string
	Field2 string
}

type T2 struct {
	Field1 string
	Field2 string
}

func (t T) String() string {
	return fmt.Sprintf("%s.%s", t.Field1, t.Field2)
}

func check(test bool) bool {
	if test {
		return true
	} else {
		return false
	}
}

func lintTest(cmd *cobra.Command, args []string) error {

	var x T
	y := T2{
		Field1: x.Field1,
		Field2: x.Field2,
	}

	fmt.Println(y)

	//Reversable was a good exercise right?
	strs := []string{"kind: Namespace", "kind:Namespace", "kind: Foo", "kind", "kind:  Namespace", "kind: Namespace"}
	newStrs := []string{}

	if strs != nil && len(strs) != 0 {
		fmt.Println("strs is not empty")
	}

	for _, str := range strs {
		newStrs = append(newStrs, str)
	}

	nsRegex := regexp.MustCompile("kind:\\s*Namespace")
	set := make(map[string]bool)

	for _, str := range newStrs {
		if (nsRegex.MatchString(str)) {
			fmt.Println(str)
		}
	}

	for key, _ := range set {
		fmt.Println(key)
	}

	return nil
}
```

run: `make lint` and start fixing issues

**notes:**
1. GoLand has a "Reformat Code" under "Code" (cmd+shft+L) super helpful
2. This will require connecting cmd to root cmd

#### 6:  Working with Sets

Review the following code:
`set := make(map[string]bool)`

and 
```go
for key := range set {
		fmt.Println(key)
}
```

Currently this does not print anything.  We want the set to be a "set" of all regex matches.  A set guaratees no duplicates.  If you were to print all the `strs`, there would be 6 elements.  How many print as part of the set?

#### 7: Writing an integration test

Create a `cmd/lint_test.go` with the following:

```go
// +build integration

package cmd

import (
	"testing"
)

func Test_check(t *testing.T) {

	tests := []struct {
		name string
		args string
		want bool
	}{
		{"zero length string", "", true},
		{"1 char", "1", true},
	}
	for _, tt := range tests {
		tt := tt
		t.Run(tt.name, func(t *testing.T) {
			if got := check(tt.args); got != tt.want {
				t.Errorf("check() = %v, want %v", got, tt.want)
			}
		})
	}
}
```

run `make test`
run `make integration-test`

What is the difference?


## Lab Solution

https://github.com/codementor/wman/tree/lab4-solution

Clone:  `git clone -b lab4-solution https://github.com/codementor/wman.git`