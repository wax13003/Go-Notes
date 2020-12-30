## Context

**The Context Interface**
```go
type Context interface {
  Deadline() (deadline time.Time, ok bool)
  Done() <-chan struct{}
  Err() error
  Value(key interface{}) interface{}
}
```
The **context** package has default implementations for almost all use cases.

### Cancellation
One of the most useful, and common, uses for **context.Context** is cancellation of multiple goroutines.

Using a **context.Context** and the **select** statement we can control a goroutine.

### Creating A New Context
All contexts start with a **context.Background()** context. This is just an "empty" context onto which we can create new "child" contexts.

Contexts behave like trees with a top node, **context.Background()**, and then further sub-nodes.

#### Canceling With Contexts
```go
ctx, cancel := context.WithCancel(context.Background())
```
When the **cancel** function is a called a message is sent to the **ctx.Done()** channel that we can listen to and stop our goroutines safely.

### Selecting On Done
use a **select** statement to listen to the **ctx.Done()** channel on the context
```go
func search(ctx context.Context, i int) {
	for {
		select {
		case <-ctx.Done():
			fmt.Print("x")
			return
		default:
			fmt.Print(".")
			x := rand.Intn(N)
			if x == 42 {
				fmt.Print("found")
				found <- fmt.Sprintf("routine %d: found 42!", i)
				return
			}
			time.Sleep(10 * time.Millisecond)
		}
	}
}
```
### Canceling With Timeouts
```go
context.WithTimeout
context.WithDeadline
```
**WithTimeout** will cancel the context after a certain amount of time.   
**WithDeadline** will cancel the context at a certain time.

### Context Values
Contexts also allow us to pass values through our applications easily, using **context.WithValue**. This can clean up an application and can help make sure that necessary information is passed along between concurrent processes.

With **context.WithValue** we can set a key/value pair into a new context.
```go
ctx = context.WithValue(ctx, "magic", 42)
// returns an interface or nil
i := ctx.Value("magic")
```
If the current context does not have the requested value it will keep looking up parent tree until it either finds the value, or hits the top most node.

### Passing Context Values  
**Non-string Context Keys**   
Context keys must be comparable, so if two packages use the same string key, will cause interference  

**Namespacing Context Values**
To avoid collisions and provide better APIs for working with context values we can first define a type to represent keys:  
```type key int```  
Then define a const to use as a key:
```const userKey key = 0```
the value of the key doesn't matter, but each const need to be unique 

**Context Getters / Setters**
exported getter and setter functions that also provide strongly-typed semantics  

Setter:
```go
func WithUser(ctx context.Context, u *User) context.Context {
	return context.WithValue(ctx, userKey, u)
}
```

Getter:
```go
func FromContext(ctx context.Context) (*User, error) {
	u, ok := ctx.Value(userKey).(*User)
	if !ok {
		return nil, errors.New("user not found")
	}
	return u, nil
}
```
Using Getters & Setters
```go
func main() {
	u := &User{
		ID:   0,
		Name: "Tim",
	}

	ctx := WithUser(context.Background(), u)
	printUser(ctx)

}

func printUser(ctx context.Context) {
	u, err := FromContext(ctx)
	if err != nil {
		fmt.Println("err getting user:", err)
		return
	}
	fmt.Println("user:", u)
}
```

