## Interfaces
- Interfaces allow us to specify behavior. They are about doing, not being.
- Unlike other languages interfaces in Go are implicitly not explicitly defined.
- Interfaces are found all around the standard library. 

### Why Interfaces?
Interfaces allow us to abstract code to make it more reusable, extensible and more testable.
```go
func entertain(b Beatle) {
	b.Play()
}
```
must be a **Beatle** to **Play**

```go
type Entertainer interface {
	Play()
}

func entertain(e Entertainer) {
	e.Play()
}
```
Anything implements **Entertainer** Interfaces can **Play**, using an interface allows to open that function up to a larger group of potential types that can implement the interface. 

### Duck Typing Or Structural Typing In Go

In most OO (object oriented) langauges you have to ***explicity*** declare that you are implementing an interface.

In Go this is ***implicit***.   
Provided a type implements the all behaviors specified in the interface, it can be said to implement that interface.    

The compiler will check to make sure a type is acceptable and report error if it does not

- The concrete type does not need to know about your interface
- You are able to write interfaces for concrete types that already exist
- You can write interfaces for other people's types, or types that appear in other packages

### Defining An Interface

```type MyInterface interface {}```

- Interfaces are a collection of methods.
- Interfaces can have zero, one, or many methods.
- Interfaces are not fields.
```go
type MyInterface interface {
    Method1()
    Method2() error
    Method3() (string, error)

    Email string // Invalid
    .
    .
    .
}
```
### Using Interfaces
Because interfaces are types, we can receive them as arguments to functions.

### Multiple Interfaces
Because interfaces are implemented implicitly, it means that types can implement many interfaces without even realizing it.

### Assignability
Remember that when determining assignability, Go is only concerned with the method set of an interface--not the name of the interface:

```go
type Greeter interface {
	Greet(string)
}

type Welcomer interface {
	Greet(string)
}
```
these two interfaces are equivalent

### The Empty Interface
If you declare an interface with zero methods, then every type will satisfy that interface.  
empty interface to represent "anything".  
```go
// generic empty interface:
interface{}

// a named empty interface:
type foo interface{}
```
### The Problem With Empty Interfaces
- No type information
- Runtime panics are very possible
- Difficult code (to test, understand, document, etc...)

### Type Assertions
type assertion returns an optional second boolean value that can be used to confirm whether the assertion was successful or not
```go
// object.(type)

func printInline(s interface{}) {
	if stringer, ok := s.(fmt.Stringer); ok {
		fmt.Println(stringer.String())
		return
	}
	fmt.Println("not a stringer")
}
```
If you do not check the second, optional boolean value from the type check you risk your application panicking.

### Switching On Type
Using a **switch** statement and the **type** keyword
```go
func print(i interface{}) {
	switch t := i.(type) {
	case int:
		t = t + 3
		fmt.Printf("INT: %d\n", t)
	case float64:
		fmt.Printf("FLOAT: %f\n", t)
	case fmt.Stringer:
		fmt.Printf("%T\n", t)
		fmt.Println(t.String())
	default:
		fmt.Println(t)
	}
}
```

### Interface Zero Value
Because interfaces aren't backed by any actual implementation the zero value of an interface is **nil**.  

Backing Interfaces:  
More commonly, you will be backing an interface with your concrete type.

Without backing, the code will compile, but result in a run-time panic
```go
type Stream struct {
	io.Writer
}

s := Stream{} // panic 

//backing
s := Stream{
	Writer: os.Stdout,
}   
fmt.Fprintf(s, "Hello Gophers!")
```

### Pointer Receivers
- Methods defined on **pointer** receivers can only be accessed when using a pointer.
- However, methods defined on a **value** type can be access by both pointers and values.

easily facing this error
```go
type User struct {
	First string
	Last  string
}

func (u *User) String() string {  //String method has pointer receiver
	return fmt.Sprintf("%s %s", u.First, u.Last)
}

func pretty(v fmt.Stringer) {
	fmt.Println(v.String())
}

func main() {
	u := User{First: "Rob", Last: "Pike"}
	pretty(u) //cannot use u (type User) as type fmt.Stringer in argument to pretty: User does not implement fmt.Stringer (String method has pointer receiver)
}

//change to Value receivers
func (u User) String() string {
	return fmt.Sprintf("%s %s", u.First, u.Last)
}
func main() {
	u := &User{First: "Rob", Last: "Pike"}
	pretty(u)
}
```
