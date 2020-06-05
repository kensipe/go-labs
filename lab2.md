## Lab 2: Working with Strings

## Introduction

Starting with lab 1 and using information we learned, this lab you will create a function in a package under `pkg`, create a test for it and use it from the main client app.

* packages and imports
* creating code within a package
* writing and running a go test
* data structure constraints between bytes and runes

> **Note:** This lab assumes you have a solution for lab 1 as a starting point.

## Steps

#### 1: make a package for string

`mkdir -p pkg/string`

#### 2: create a string.go file

`string/string.go`

> make sure to have this file in the acme.com/string package (which means `package string` at the top of the file)

Create a string reversing function with the following signature
```go
func Reverse(s string) string {
  // temp you may want to return a string 
  return "reversed"
}
```

**NOTES:** 
	b := []byte(s) is a way of creating a byte array from a string in golang.

#### 3: change the main.go to use this new function

Add the following code to the main.go main function.

```go
fmt.Printf("hello %s, your name backward is %q", name, string.Reverse(name))
```

Run it: `go run cmd/wman/main.go foo`

Common errors here:
1. is your code in GOPATH?
2. is the package imported?  `import 	"github.com/codementor/wman/pkg/string"`


#### 4: Create a string reverse test

testing in golang is accomplished by creating a file with a `_test.go` extension to the name of the go file primarily under test.  In our example, if we are testing `string.go`, you would create a `string_test.go` file.  Here is an example:

```go
import (
	"testing"
)

func TestReverse(t *testing.T) {
	var tests = []struct {
		s, want string
	}{
		{"Hello", "olleH"},
		{"¶", "¶"},
		{"", ""},
	}

	for _, c := range tests {
		got := Reverse(c.s)
		if got != c.want {
			t.Errorf("Reverse(%q) == %q, want %q", c.s, got, c.want)
		}
	}
}
```

#### 5: Test the code

From `$GOPATH` type: `go test acme.com\string` or from the `$GOPATH/src/acme.com/string` directory type `go test`

You can also run `make test` from project root

output so far:
```shell
=== RUN   TestReverse
    TestReverse: string_test.go:19: Reverse("Hello") == "foo", want "olleH"
    TestReverse: string_test.go:19: Reverse("¶") == "foo", want "¶"
    TestReverse: string_test.go:19: Reverse("") == "foo", want ""
--- FAIL: TestReverse (0.00s)
FAIL
```

#### 6: Writing Reverse
```go
	b := []byte(s)
	for i := 0; i < len(b)/2; i++ {
		j := len(b) - i - 1
		b[i], b[j] = b[j], b[i]
	}
	return string(b)
```

**And Test**
```shell
make test
go test ./pkg/... 
--- FAIL: TestReverse (0.00s)
    string_test.go:19: Reverse("¶") == "\xb6\xc2", want "¶"
FAIL
FAIL	github.com/codementor/wman/pkg/string	0.184s
FAIL
make: *** [test] Error 1
```

#### 7: Fixing the code

It may have been misleading :), but it is a good example of what can happen in the real world.  The code we used `b := []byte(s)` works for ascii, but a byte isn't large enough to handle unicode chars.  To fix our program, use a slice of runes with the following code: `b := []rune(s)`

Run the tests!

Run main:

```
go run cmd/wman/main.go NoFluff
hello NoFluff, your name backward is "ffulFoN"
```
