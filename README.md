## Go for Java Developer Labs

As part of the [NFJS](https://nofluffjuststuff.com/) [virual workshop series](https://nofluffjuststuff.com/virtual-workshops), this repository is the labs for [Go for Java developer workshop](https://nofluffjuststuff.com/virtual-workshops/180/golang_for_java_developers)

This is a series of 7 labs (each one building on the last one), built to solidify the course concepts AND to provide real world insights

### Requirements

* Go 1.13+ (Go 1.14 used to build course)
* git
* apikey from https://openweathermap.org/api 

> Please get the apikey early in the course. provisioning a key can take time.

Solutions at: https://github.com/codementor/wman/ with solution branches for each lab.


### Lab 1:  [Go Project Setup and Layout](lab1.md)

In this lab you will become familar with Go; go run, go build, go test and the Go environment.  In the end, you will understand the project structure for a Go project. 

* Go run, Go doc, Go build
* Go project Makefile
* Go linting introduction
* Go Project structure

### Lab 2:  [Functions, data structures and tests](lab2.md)

Starting with lab 1 and using information we learned, this lab you will create a function in a package under `pkg`, create a test for it and use it from the main client app.

* packages and imports
* creating code within a package
* writing and running a go test
* data structure constraints between bytes and runes

### Lab 3: [Go Module Dependencies and Cobra CLI](lab3.md)

In this lab you will become familar with Go dependencies using them to create a better command-line.

* Go dependencies
* Cobra CLI

Starting with lab 2, this lab we will add a dependency to the project using a tool known for better commandline development called [Cobra](https://github.com/spf13/cobra).

> **Note:** This lab assumes you have a solution for lab 2 as a starting point

### Lab 4: [Data Conversion, Linting, Build Tags](lab4.md)

Starting with lab 3, we will extend a command to calculate dog years. In the process we will need to convert strings to int and back.  We will work with STDIN for input and provide error handling. Then we will take a deeper look at linting.

* STDIN
* error handling
* data conversions
* linting
* build tags
* variable tests

> **Note:** This lab assumes you have a solution for lab 3 as a starting point.

### Lab 5: [JSON Decoding to Data Structures and working with Files](lab5.md)

In this lab you will become familar with Go struct and methods, along with working with dependencies. 

* Structs 
* JSON Encoding / Decoding
* Files

> **Note:** This lab assumes you have a solution for lab 3 as a starting point.

### Lab 6: [HTTP Requests to Go Data Structs](lab6.md)

In this lab you will level up on Go structs and our use of Cobra CLI,  along with learning how to make HTTP Get requests.

* Structs 
* JSON Encoding / Decoding
* HTTP Requests
* UITable

> **Note:** This lab assumes you have a solution for lab 5 as a starting point.

### Lab 7: Go Routines and Channels

In this lab you will reinforce learning around goroutines and channels making concurrent HTTP requests. 

* struct sorting
* go routines
* channels
* sync.WaitGroup

> **Note:** This lab assumes you have a solution for lab 6 as a starting point.

