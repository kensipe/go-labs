## Lab 3: Working with Dependencies

## Introduction

In this lab you will become familar with Go dependencies using them to create a better command-line.

* Go dependencies
* Cobra CLI

Starting with lab 2, this lab we will add a dependency to the project using a tool known for better commandline development called [Cobra](https://github.com/spf13/cobra).

> **Note:** This lab assumes you have a solution for lab 2 as a starting point.

## References

* CLI tools: https://awesome-go.com/#command-line

## Steps

#### 1: Add the cobra dependency to the project

From the root of the project:
`go get -u github.com/spf13/cobra`

**note:** need help understanding `go get`?  checkout out `go help get`

Take a look at `go.mod`

```go
cat go.mod 
module github.com/codementor/wman

go 1.14

require (
	github.com/fsnotify/fsnotify v1.4.9 // indirect
	github.com/mitchellh/mapstructure v1.3.1 // indirect
	github.com/pelletier/go-toml v1.8.0 // indirect
	github.com/spf13/afero v1.2.2 // indirect
	github.com/spf13/cast v1.3.1 // indirect
	github.com/spf13/cobra v1.0.0 // indirect
	github.com/spf13/jwalterweatherman v1.1.0 // indirect
	github.com/spf13/pflag v1.0.5 // indirect
	github.com/spf13/viper v1.7.0 // indirect
	golang.org/x/sys v0.0.0-20200602225109-6fdc65e7d980 // indirect
	gopkg.in/ini.v1 v1.57.0 // indirect
)
```

#### 2: Create root CLI

From project root, `mkdir -p pkg/cmd`
Create a file under `cmd` named `root.go`
```go
package cmd

import (
	"github.com/spf13/cobra"
)

// NewWmanCmd creates a new root command for wman
func NewWmanCmd() *cobra.Command {
	cmd := &cobra.Command{
		Use:          "wman",
		Short:        "Weather Man CLI",
		Long:         `Weather Man (wman) CLI for capturing weather information.`,
		SilenceUsage: true,
		Example: `  # Run reverse function
  wman reverse nofluff
`,
	}
	return cmd
}
```

Here the func `NewWmanCmd` returns a pointer to corba.Command.  Inside the func, we configure the Command structure.

#### 3: Update Main.go

Lets change main.go to call and run this command.

```go
import (
	"os"

	"github.com/codementor/wman/pkg/cmd"
)

func main() {
	if err := cmd.NewWmanCmd().Execute(); err != nil {
		os.Exit(-1)
	}
}
```

Try running: `make run`
or with some flags:  `go run cmd/wman/main.go --help`

#### 4: Add a Print command

Create a file under `cmd` named `print.go`

```go
package cmd

import (
	"fmt"

	"github.com/spf13/cobra"
)

var (
	printExample = `  # Prints a hello message to the passed in name
  wman print nofluff`
)

// newPrintCmd returns a new initialized instance of the print sub command
func newPrintCmd() *cobra.Command {
	printCmd := &cobra.Command{
		Use:     "print",
		Short:   "Print hello to the passed in argument",
		Example: printExample,
		RunE:    PrintCmd,
	}

	return printCmd
}

// PrintCmd performs the print sub command
func PrintCmd(cmd *cobra.Command, args []string) error {
	// might be worth err checking args
	fmt.Printf("Hello %q\n", args[0])
	return nil
}

```

#### 5: Updating root.go

Add before the `return cmd` the following (which includes the return for reference):
```go
	cmd.AddCommand(newPrintCmd())
	return cmd
```

Lets run it: `go run cmd/wman/main.go print nofluff`
try: `go run cmd/wman/main.go`
try: `go run cmd/wman/main.go p`

how great is:
```shell
go run cmd/wman/main.go p
Error: unknown command "p" for "wman"

Did you mean this?
	print

Run 'wman --help' for usage.
exit status 255
```
Run `go run cmd/wman/main.go print nofluff`
```
go run cmd/wman/main.go print nofluff
Hello "nofluff"
```

#### 6: Adding flags

In the `print.go` file, do the following:

Add a structure for print options:

```go
type printOptions struct {
	Reverse bool
}
```
**note:** a structure is not required but is common.

First line after the `newPrintCmd` func, create an instance of the options.

```go
func newPrintCmd() *cobra.Command {
	opts := &printOptions{}

```

After the `printCmd` is created and before it is returned write the following:

```go
	printCmd.Flags().BoolVar(&opts.Reverse, "reverse", false, "If set to true, it reverses the argument passed in (default \"false\")")
```

Test the command: `go run cmd/wman/main.go print -h`

Change the method signature to `PrintCmd` func and implement the logic:

```go
	func PrintCmd(options *printOptions, args []string) error {
```

Now we need to change the `RunE` of construction of `cobra.Command` for `printCmd`
```go
	RunE: func(cmd *cobra.Command, args []string) error {
		return PrintCmd(opts, args)
	},
```

In order to implement the logic in `PrintCmd` it is necessary to import our string package: `"github.com/codementor/wman/pkg/string"`, if you try to import that package and run you get:

```shell
go run cmd/wman/main.go print tests
# github.com/codementor/wman/pkg/cmd
pkg/cmd/print.go:27:39: use of package string without selector
pkg/cmd/print.go:38:43: use of package string without selector
```

the issue if you search through code is there are other uses of the word `string` in fact string is a keyword.  To resolve this, try importing:

`import wstring "github.com/codementor/wman/pkg/string"`
which means that a call to `Reverse` func is `wstring.Reverse()`

Implement the rest of it such that `print nofluff --reverse` produces:

```shell
go run cmd/wman/main.go print nofluff --reverse
Hello "ffulfon"
```

Before we are done...

1. `make lint`
2. `make test`
3. `go mod tidy` # see the change in go.mod

#### Bonus: User Defined types and Methods

In `pkg/string/string.go` add the following:

```go
// Reversable is a string that is reversable
type Reversable string

// Reverse reverse a reversable
func (s Reversable) Reverse() Reversable {
	return Reversable(Reverse(string(s)))
}
```

Here we define a new type called `Reversable` which is of type string.  Followed by a method which is specific to "Reversable" types.  The function is complicated by a series of casting, the `s` Reversable is casted to string, which is passed to the `Reverse` function that returns a string, so it must be casted back to `Reversable` as a return type.

The logic in `print.go` changes to:

```go
	name := wstring.Reversable(args[0])
	if options.Reverse {
		name = name.Reverse()
	}

	fmt.Printf("Hello %q\n", name)
	return nil
```

## Lab Solution

https://github.com/codementor/wman/tree/lab3-solution

Clone:  `git clone -b lab3-solution https://github.com/codementor/wman.git`

and 

https://github.com/codementor/wman/tree/lab3-with-bonus-solution

Clone:  `git clone -b lab3-with-bonus-solution https://github.com/codementor/wman.git`
