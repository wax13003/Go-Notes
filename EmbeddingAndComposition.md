## Embedding And Composition
Go does not have inheritance.

### Composing Complex Types
not only limited to "primitive" type, can use complex types as well
```go
type Admin struct {
	User        User
	Permissions map[string]bool
}

a := Admin{
    User: User{
        Name: "Homer",
        Age:  40,
    },
    Permissions: map[string]bool{},
}
```

### Embedding
In Go we can embed a type by just declaring its type, with no name.

Method Promotion:  
Inner type's method get promoted and can be accessed as if they were part of the outer type
```go
func (u User) Greet() {
	fmt.Printf("Hello, %s\n", u.Name)
}

func main() {
	a := Admin{
		User: User{
			Name: "Homer",
		},
	}

	a.Greet() // Hello, Homer
}
```

Method Collision: 
When there is a method collision (the inner type and the outer type both implement the same method) the outer type always wins.
```go
func (a Admin) Greet() {
	fmt.Printf("Admin: %s\n", a.Name)
}
a.Greet() // Admin: Homer
```
Ambiguous Methods:    
If two inner types provide the same method, neither is promoted.

Method Overriding:  
When embedding another type we can **override** their methods with new methods, while still being able to call the original method.
```go
func (u User) String() string {
	return u.Name
}

type Admin struct {
	User
	Permissions map[string]bool
}

func (a Admin) String() string {
	return fmt.Sprintf("Admin: %s", a.User.String())
}

fmt.Println(a.String()) // Admin: Homer
```
Overriding Via **Type Conversion** 
Type conversion happens when we assign the value of one data type to another:
```go
var i uint64 = 10
var j int = 20
k := (int(i) + j)
```
You can also create a type just for method overriding, and via type conversion change the behavior in your program:
```go
type prettyString string

func (p prettyString) String() string {
	return fmt.Sprintf("I'm a pretty string: %q", string(p))
}

func main() {
	s := "Hi"
	fmt.Println(s)
	fmt.Println(prettyString(s))
}
```

Embedding Pointers    
if that pointer is nil and you try to call a promoted method, you could open yourself up to panics

```go
type Admin struct {
	*User
}
func (u *User) String() string {
	return u.name
}

a := &Admin{}
fmt.Println(a.String())
```

- Embedding != Interface
- Outer type can satisfy any interfaces that the inner type satisfies
- Embedding Interfaces to make some powerful types

Embedding Interfaces Within Structs 

Interface types can also be embedded into structs as well, and similarly promote their methods onto the struct type. This can be useful for slightly modifying the behavior of an interface or adding methods to a smaller interface to satisfy a wider one.

Embedding Vs. Fields

Embedding
- Promotes all fields and methods to newly composed type
- New type can automatically satisfy any interface the embedded type satisfies
- Requires method overriding to protect any methods you wish to guard (if any)

Field
- No promotion of any fields or methods
- Type can be protected by not exporting it
- Can create your own methods and call underlying types fields or methods as needed