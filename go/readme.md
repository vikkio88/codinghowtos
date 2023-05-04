# Go
[[web](https://go.dev)] [[docs](https://go.dev/doc/)] 

## Types
```go
nil
rune // is char basically
bool
int
uint
int32
int64
float32
float64
string

interface{} 
any //  <- this is also a type alias to interface{}
```

**Enums**
```go
// Enums do not exist but you can kind of type alias const and give them methods
type Currency uint8
const (
    Dollar Currency = iota // this means that Dollar is 1 and everything else will be +1
    Euro // iota makes this be Currency = 2
    Pound //  this will be Currency = 3
)

// this method will only work for Currency "enum" and will be called when you try to cast to string a Currency
func (c Currency) String() string {
	switch c {
	case Dollar:
		return "$"
	case Euro:
		return "€"
	case Pound:
		return "£"
	}

	return ""
}

dollar := Dollar
dollarSign := fmt.Sprintf("%s", dollar)
// dollar == "$"
```

## Types Casting
```go
// primitive promotion or parsing is quite easy
i := int32(someOther)
f := float32(someOther)

// atoi and itoa are part of strconv package
res, err := strconv.ParseInt(str, 10, 0)
// res is an int64
if err != nil {
    fmt.Println("that is not a number, try again")
}
// here is to get a 32bit int
int32(res)
```

## Collections
`arrays` if dynamic and on the heap are called `slices`
```go
// create an empty slice of strings
var someStuff []string
// create and initialise a slice of strings
someStuff := []string{"a", "aa"}

// add stuff to array
someStuff = append(someStuff, "b")

// if you know the size either use an array
iKnow := [1]string{"aaa"}
// or you can create already with capacity
aSlice := make([]string, 3)
// be careful as the first 3 elements will be zeroed to a ""
// you need to access via index

// to search stuff in array you either iterate
for index, element := range someStuff {
    if element == "the one I am looking for" {
	    break
	}
}

// or you can use the extension module
// go get golang.org/x/exp/slices
idx := slices.IndexFunc(users, func(u User) bool { return u.Id == id })
if idx == -1 {
    return nil, fmt.Errorf("No User")
}

return users[idx], nil
```
`maps` are not ordered, so the order you init them is not always respected
```go
var mappingStuff map[string]string
mappingStuff["A"] = "this is the letter A"

// or 
mapping := map[string]string{
    "a": "the letter a",
}

// to access things
mapping["a"]
// or
val, exists := mapping["b"]
if exists {
    fmt.Println(val)
} else {
    // val will be ""
}

// to iterate over a map
for k,v := range mapping {
    // k == "a" and v == "the letter a"
}
```

## Functions
```go
func doStuff(someParam string, someOther int) string {
    return ""
}

func doStuffError() (string, error) {
    return "", errors.New("Some Error")
}

func howLongIsAPieceOfString(pieceOf string) (namedReturn int) {
    namedReturn = len(pieceOf)
    return // this returns the named var
    // careful that it does not work really well with complex types,
    // you get some nil pointer exception
}
```

## Strings
```go
aString := "yo I am a string"
```
### String Manipulation
```go
// the strings module defines loads of strings methods
import "strings"

strings.ToLower(strings.ReplaceAll(strings.TrimSpace(fullName), " ", ".")),
```

## OOP
```go
// objects are defined like this
type User struct {
	Id       string
	Username string
	FullName string
}

// by standard the constructor is a function like this
func NewUser(fullName string) User {
	return User{
		Id:       "someId",
		Username: strings.ToLower(strings.ReplaceAll(strings.TrimSpace(fullName), " ", ".")),
		FullName: fullName
	}
}
// and used like this
user := NewUser("Mario Rossi")

// methods are defined like this
func (u User) String() string {
	return h.F("%s %s", u.Id, u.Username)
}

// and can also be defined only to work with pointers
// so they will modify in place rather than a copy
func (u *User) ChangeUsername(newUsername string) {
	u.Username = newUsername
}
```

## Serialization
```go
// json is in the std lib
import "encoding/json"

// you need to specify attributes and it will only serialise Public fields (UpperCamelCase)
type User struct {
	Id       string    `json:"id"`
	Username string    `json:"username"`
	FullName string    `json:"fullName"`
}

// to serialize an object to Json
data, err := json.Marshal(dtos)

// to get it back from a json string
var users []User
json.Unmarshal(data, &users)
```

## Tests
A Test framework is bundled up in golang itself, by default is not great as you need to fail tests yourself by using `if` and `t.Fatalf`

```go
import  "testing"

func TestCheckingStuff(t *testing.T) {
    msg := ""
    if msg != "" {
        t.Fatalf(`Hello("") = %q, %v, want "", error`, msg, err)
    }
}
```

but there is a nice assert package called `testify` which allows you to write better assertions
```go
import (
	"testing"
	"github.com/stretchr/testify/assert"
)

func TestSomeStuff(t *testing.T) {
	msg := ""
	assert.Equal(t, "", msg)
}
```

to run all the tests in a folder and all the subfolders I normally use a Makefile script
```makefile
tests:
	go test ./...
```

if you want to see a verbose test output
```makefile
tests-v:
	go test -v ./...
```

## Package Management
Go packages are usually hosted in github or other websites, and they are usually Urls.
in here [[Repo](https://pkg.go.dev/)] you can find most of the commonly used ones.
```sh
# to init a module in your folder project
go mod init name_of_the_project
# if you are going to host it on gh you can also use
go mod init github.com/name/project

# this creates a go.mod and go.sum files
go get "some module"
# will add it to the go.mod

go mod tidy
# fixes dependencies and removes unused ones
```

## Build
I use make files to create run/build scripts
```makefile
build:
	go build -o dist/binary_name .

build-prod:
	go build -ldflags "-s -w" -o dist/binary_name .

run:
	go run .
```

This has to be run on the same folder of the main script.

## Little Quirks

### Time/Date format is weird

Time and date formatting is interesting and weird.

// Rather than using YYY DD MM and other string placeholders go uses some real data as example for time/date formatting
| Time        | Options                                        |
| ----------- | ---------------------------------------------- |
| Year        | 2006 ; 06                                      |
| Month       | Jan ; January ; 01 ; 1                         |
| Day         | 02 ; 2 ; \_2 (For preceding 0)                 |
| Weekday     | Mon ; Monday                                   |
| Hour        | 15 ( 24 hour time format ) ; 3 ; 03 (AM or PM) |
| Minute      | 04 ; 4                                         |
| Second      | 05 ; 5                                         |
| AM/PM Mark  | PM                                             |
| Day of Year | 002 ; \_\_2                                    |
```go
// formatting from and to date strings looks like this
const (
	baseFormat        = "1/2/2006 15:04:05"
	descriptionFormat = "Monday, January 2, 2006, at 15:04"
)

// Schedule returns a time.Time from a string containing a date.
func Schedule(date string) time.Time {
	result, _ := time.Parse(baseFormat, date)

	return result
}

func Description(date string) string {
	appDate := Schedule(date)

	return fmt.Sprintf("You have an appointment on %s.", appDate.Format(descriptionFormat))
	// in: "6/6/2005 10:30:00"  will return: "You have an appointment on Monday, June 6, 2005, at 10:30.
}
```

### Modules and files
code is organised in packages, each folder can have one or more package defined at the beginning of the file, but you cannot do subpackages.

```
// This is how I structure projects
.
├── Makefile
├── Readme.md
├── go.mod
├── go.sum
├── main.go 		<-- MAIN being part of package main
├── app 			<-- package app
│   ├── app.go
│   └── menu.go
├── console 		<-- package console
│   └── console.go
├── db 				<-- package db
│   └── file_db.go
├── dist			<-- folder that contains binary output
│   └── user_store
├── libs			<-- package libs
│   ├── crypto.go
│   └── crypto_test.go
└── models			<-- package models
    ├── account.go
    ├── account_test.go	<-- package models_test
    ├── money.go
    ├── money_test.go 	<-- package models_test
    ├── user.go
    └── user_test.go 	<-- package models_test
```