## Fouctions
Functions in Go are a primitive type.  

### Arguments
Functions can take N number of arguments, including other functions.

### Return Arguments
Functions can return 0 to N numbers of values, though it is considered best practice to not return more than two or three.

When returning an error it is considered best practice to return that value as the last value.  
```func three() (int, string, error)```

### Declaring Return Arguments
```go
func sayHello() string {
	return "Hello"
}
```
### Multiple Return Arguments
Multiple return arguments need to be placed inside of ().
```go
func slicer(s []string) (int, int) {
	return len(s), cap(s)
}
```

### Named Returns
```go
func exists() (exist bool) {
    exist = true
    return
}
```
better not using it, hard to remember which parameter is meant for which value with this given signature

### Shadowing
Go has the ability to shadow a variable.

### Variadic Arguments
Functions can take variadic arguments, meaning 0 to N arguments of the same type. This is done by prepending the type of the argument with ....  
```func sayHello(names ...string) {}```  

Variadic arguments must be the last argument of the function.   
```func sayHello(group string, names ...string) {}```

Slices can be "exploded" using trailing ... to send them as variadic arguments to a function.
```go
beatles := []string{"John", "Paul", "George", "Ringo"}
sayHello(beatles...)
```

### When To Use Variadic
The most common reason to use a variadic is when your function is commonly called with zero, one, or many arguments.

You may have one or many users to look up. Without using a variadic signature, your code would look like this:
```go
func LookupUsers(ids []int) []Users {
    for i, id := range ids {
        ...
    }
}

var id1, id2, id3 int
id1 = 1
id2 = 2
id3 = 3
var ids := []int{1, 2, 3)
LookupUsers([]int{id1})
LookupUsers([]int{id1, id2, id3})
LookupUsers(ids)
```
By changing the signature to be variadic as follows:
```go
func LookupUsers(ids ...int) []Users {
    for i, id := range ids {
        ...
    }
}

var id1, id2, id3 int
id1 = 1
id2 = 2
id3 = 3
var ids := []int{1, 2, 3)

LookupUsers(id1)
LookupUsers(id1, id2, id3)
LookupUsers(ids...)
```
### First Class Functions
- Function signatures are a type
- Functions can be arguments and return values to/from other functions

### Closures
Functions close over their environment when they are defined. This means that a function has access to arguments declared before its declaration.

### Functions As Arguments
Functions in Go can be passed as values to other functions.
```go
func main() {
	f := func() string {
		return "Hello"
	}
	sayHello(f)
}

func sayHello(fn func() string) {
	fmt.Println(fn())
}
```

The signature of the function being passed in must match the function signature of the argument in the callee function.

### Anonymous Functions
Anonymous functions are functions that aren't assigned to a variable, or that don't have a "name" associated with them. It's common pattern in Go

### Functions As Types
Function signatures act as a defacto "type", but explicit types can also be created for a function signature.

This has a few benefits:

- Argument declaration is cleaner
- The new type can have other functions hung off it
- The new type can now easily be documented
  
```go
type greeter func() string

func main() {
	sayHello(func() string {
		return "Hello"
	})
}

func sayHello(fn greeter) {
	fmt.Println(fn())
}
```

### Functions On Functions
By creating a new type for a function, we can treat that type like all types in Go, and can hang other functions off of it.

```go
type greeter func() string

func (greeter) Name() string {
	return "World"
}

func sayHello(fn greeter) {
	fmt.Println(fn(), fn.Name())
}
```

### Functions Accepting Arguments From Functions

```go 
func main() {
	takeTwo(returnTwo()) //return types and number of arguments match
}

func returnTwo() (string, string) {
	return "hello", "world"
}

func takeTwo(a, b string) {
	fmt.Println(a, b)
}
```

### Methods
Methods are syntactic sugar for declaring functions on types.

### Method Receivers
Methods also allow us to namespace a function to a receiver.
```go
func (b Beatle) String() string {
	return fmt.Sprintf("%s plays %s", b.Name, b.Instrument)
}

func String(b Beatle) string {
	return fmt.Sprintf("%s plays %s", b.Name, b.Instrument)
}

func main() {
	b := Beatle{Name: "Ringo", Instrument: "Drums"}
	fmt.Println(b.String()) // Ringo plays Drums
	fmt.Println(String(b))  // Ringo plays Drums
}
```
Both functions are equivalent, but the former namespaces the String function to the Beatle type, allowing us to have multiple String functions with different receivers.

### **Value Vs. Pointer Receivers**
You can define a receiver for a method as either a value or a pointer receiver. They will behave differently.

A value receiver can not mutate the data, while a pointer receiver can.

```go
package main

import (
	"fmt"
	"strings"
)

type User struct {
	First string
	Last  string
}

// Pointer Receiver
func (u *User) Titleize() {
	u.First = strings.Title(u.First)
	u.Last = strings.Title(u.Last)
}

// Value Receiver
func (u User) Reset() {
	u.First = ""
	u.Last = ""
}

func main() {
	u := User{First: "cory", Last: "lanou"}
	u.Titleize()
	fmt.Println(u)

	// Reset can't mutate the data as it's defined with a value receiver
	u.Reset()
	fmt.Println(u)
}
```

### Methods Expressions
```u := User{First: "John", Last: "Smith"}```  
call as method  
```u.Notify()```  
call as method expression     
```User.Notify()``` 

### Deferring Function Calls

- The defer keyword in Go allows us to defer the execution of a function call until the return of the parent function.
- defer is useful for ensuring functions get called regardless of which path your code takes.

Defer Call Order:  
LIFO (last in, first out).

Deferred Calls And Panic:  
Deferred calls will still get called even if another deferred call panics

Defers And Exit/Fatal:  
Deferred calls will not fire if the code explicitly calls os.Exit or log.Fatal

Defer And Anonymous Functions:  
It is also common to combine cleanup tasks into a single defer using an anonumous function.

### Init
Any time you declare an init() function, Go will load and run it prior to anything else in that package.

Multiple Init Statements:  
init() function can be declared multiple times.  
In most cases, init() functions will execute in the order that you encounter them.

### Init Order
run based on the order it loads files, if having multiple init() declares