## Pointers
A pointer is a type that holds the address to the value of a variable.

How Go passes data? It does this by passing around a copy of the data, this is called **"pass by value"**.

It can not modify the original value, Go passes all data by value, including pointers.

### Value Of Local Copy
We are allowed to modify whatever the local copy of the value is.

### Pointer Syntax
Pointers allow us to point at the original memory space of a value.

To indicate we are expecting a pointer, we use the * modifier:  
```func changeBeatleName(b *Beatle) {}``` 
To get the address of a value we use the & modifier:  
```&Beatle{}```  
And we can store pointers too:  
```b := &Beatle{}```

### Passing By Pointer
```go
type Beatle struct {
	Name string
}

func main() {
	b := &Beatle{Name: "Ringo"}
	changeBeatleName(b)
	fmt.Println(b.Name) // George
}

func changeBeatleName(b *Beatle) {
	b.Name = "George"
	fmt.Println(b.Name) // George
}
```

### Dereferencing
We can dereference a pointer with the * as well.
```go
// Create a new pointer to a string
ps := new(string)

// Set the value of ps by dereferencing the pointer
*ps = "hello"

// Capture a new "value" as a string
s := *ps

/* output
ps type:                *string
ps value:               0xc000092030
dereferenced value:     "hello"
s type:                 string
s value:                hello
*/
```

### New
The built-in function **new** takes a type **T**, allocates storage for a variable of that type at run time, and returns a value of type ***T** pointing to it. The variable is initialized with it's zero value.

```go
s := new(string)
i := new(int)

type foo struct {
	id   int
	name string
}
f := new(foo)
// Functionally equivalent to this:
f1 := &foo{} // this is the idiomatic preferred style now

fmt.Printf("s: %v, *s: %q\n", s, *s)
fmt.Printf("i: %v, *i: %d\n", i, *i)
fmt.Printf("f: %v, *f: %+v\n", f, *f)
fmt.Printf("f1: %v, *f1: %+v\n", f1, *f1)

/* output
s: 0xc0000961e0, *s: ""
i: 0xc0000b6020, *i: 0
f: &{0 }, *f: {id:0 name:}
f1: &{0 }, *f1: {id:0 name:}
*/

```

### Panic
Accessing a pointer that has not been initialized will result in a runtime panic:
```go
var s *string
*s = "boom"
// panic: runtime error: invalid memory address or nil pointer dereference
```
### Security
One of the benefits of Go's default "pass by value" strategy is security. By defaulting to creating a copy of a value when it is passed we can rest assured knowing that our values can not be mutated by other functions and actors.

Should we want to allow others to modify our values we can pass them a pointer to the value and they will be able to modify the value accordingly.

### Performance
Pointers in Go can both help, and hurt, performance. In the case of a large chunk of memory (think an image file) passing a pointer around can reduce memory allocation and operational overhead.

However, pointers also indicate to the garbage collector and the compiler that the value in question might be used outside of its current location, which may place the value on the heap vs. the stack.

### When To Use Pointers?
Use pointers if:

- You want others to be able to modify your values or
- You have a large (memory) value that you don't want to keep copying (This is typically done as a result of profiling and you have proven this is an actual performance issue)