## Lab 1: Experiments with Go and Project Setup

## Introduction
In this lab you will become familar with Go; go run, go build, go test and the Go environment.  In the end, you will understand the project structure for a Go project. 

* Go run, Go doc, Go build
* Go project Makefile
* Go linting introduction
* Go Project structure

## References

* First code: https://golang.org/doc/code.html
* Project layout:  https://github.com/golang-standards/project-layout
* Package naming:  https://rakyll.org/style-packages/
* Go by example: https://gobyexample.com/
* Awesome Go:  https://awesome-go.com/

## Getting Familar with Go

#### 1. Go Version
`go version`

#### 2. Go Env

echo $GOPATH
echo $GOROOT
echo $GOBIN

#### 3. Go Build / Run

create a simple go file in a random location

`foo.go`
```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println("hello")
}
```

Run the program:  `go run foo.go`
Build the program: `go build foo.go`
Run the created application: `./foo`

#### 4. GoDoc
What is fmt?  `go doc fmt`
What is the difference between `fmt.Println` and `fmt.Printf`?  `go doc fmt Println` or `go doc fmt.Printf`

#### 5. Change the main app to take an argument and print hello using Printf.

hint:  `os.Args` is when to get arguments
```go
    argsWithProg := os.Args
    argsWithoutProg := os.Args[1:]
```

you should be able to test with `go run foo.go Ken` with an output of `Hello Ken!`

#### 6. Editor

Download an editor.
* [Goland](https://www.jetbrains.com/go/)
* [VSCode](https://code.visualstudio.com/)

If you have vscode, you will need to add the [go extensions](https://code.visualstudio.com/docs/languages/go)

#### 7. Lets create a Go project called wman (weather man)

The "Go Way" is within GOPATH where the project is part of the `src` tree.  However Go modules made it possible to leave outside the GOPATH.  The "Go Way" is also to include the full version code path as well.  Assuming the company codementor on github, than github.com/codementor/<project_name> is the way to go.

You have a couple of options:
* create under GOPATH: `mkdir -p $GOPATH/src/github.com/codementor/wman` or
* `mkdir -p github.com/codementor/wman` whenever

The path you choose has consequences... 
next cd to the directory created ex. `cd $GOPATH/src/github.com/codementor/wman` 

Create cmd and pkg folders:  `mkdir -p cmd/wman` and `mkdir pkg`

Move or recreate your foo.go in the "cmd/wman" folder with the name `main.go`

Initialize your Go project with Go Modules:  `go mod init`.  If you have your project in the $GOTPATH that is all you need.  If you placed your project in a location other than GOPATH, then you will need: `go mod init github.com/codementor/wman`.

From the project root, run the project:  `go run cmd/wman/main.go foo`
From the project root, build the project: `go build -o bin/wman ./cmd/wman/main.go`
From the project root, test the project: `go test pkg/...`

#### 8. Adding a Linter

Download a linter:  `go install github.com/golangci/golangci-lint/cmd/golangci-lint@v1.46.2`

Lets add linting (it will make you a better go dev!)
Create a file: `.golangci.yml` adding:
```yaml
linters:
  auto-fix: false
  enable:
    - errcheck
    - goimports
    - golint
    - gosec
    - misspell
    - scopelint
    - unconvert
    - unparam
    - nakedret
    - gocyclo
    - dupl
    - goconst
    - lll
linters-settings:
  errcheck:
    check-type-assertions: true
  lll:
    line-length: 250
  dupl:
    threshold: 400
  goimports:
    # Choose your own project owner for <codementor>
    local-prefixes: github.com/codementor
```

Run the linter: `golangci-lint run`

#### 9. Create a Makefile
Lets make a `Makefile`:
```makefile
# Run unit tests
.PHONY: test
test:
	go test ./pkg/... 

.PHONY: lint
lint:
	golangci-lint run


.PHONY: cli
# Build CLI
cli:
	go build -o bin/wman ./cmd/wman/main.go

# Install CLI
.PHONY: cli-install
cli-install:
	go install  ./cmd/wman/main.go

.PHONY: run
run:
	go run  ./cmd/wman/main.go
	
```

Now build the project with `make cli`
Run `make lint`

#### 10. Understanding Go Mod
Take a look at your `go.mod`

Run `go mod tidy` and take another look `cat go.mod`

**Congratulations!** You now have the core of a Go project.
Left out are folders such as:
* internal
* hack
* testdata
* config
* dist
As well as configurations for CI and Release

## Lab Solution

https://github.com/codementor/wman/tree/lab1-solution

Clone:  `git clone -b lab1-solution https://github.com/codementor/wman.git`