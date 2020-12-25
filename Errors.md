## Errors

No Try/Catch in Go  
Errors are return values that need to be handled like any other value returned from a function call

### Creating Errors
```go
errors.New("a message")
fmt.Errorf("a %s message", "formatted")
```
### Returning Errors
Errors are returned from functions just like any other type
```go
func greetOrBoom() (string, error) {
	return "hello", errors.New("boom!")
}
```
### Sentinel Errors
Sentinel errors use a specific value to signify that no further processing is possible.

***Custom Sentinel Errors***  
Error values are just variables  
You can also create your own sentinel errors.
```go
var errCustomized = errors.New("customized error message")
```

### Errors Are Interfaces
Errors in Go are defined as an interface in the standard library.
```go
type error interface {
	Error() string
}
```
***Defining Your Error***
```go
type strError string
func (s strError) Error() string {
	return string(s)
}
```
***Constant Sentinel Errors***
If you create sentinel errors, it may be desirable to declare them as a **constant**. This also prevents another package from changing the value of your variable at runtime.

better not use sentinel errors
- Sentinel errors become part of your public API
- Sentinel errors create a dependency between two packages

### Wrapping Errors
It is very common when writing code to wrap your error with more context, using fmt.Errorf function with %w verb
```go
fmt.Errorf("couldn't open config file: %w", err)
```

### When To Wrap?
- Wrap errors when you want to expose it to callers
- Don't wrap when doing so may expose implementation details

### Unwrapping Errors
- errors.As
- errors.Is
- errors.Unwrap

### Panics
Recover From A Panic  
With a combination of the **defer** keyword and the the **recover** function
```go
    defer func() {
        if err := recover(); err != nil {
            fmt.Println("oh no, a panic occurred:", err)
        }
    }()
    a := []string{}
    a[42] = "Bring a towel"
```
### Errors Package  
github.com/pkg/errors
