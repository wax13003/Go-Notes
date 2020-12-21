## Maps
A map is an unordered set of values indexed by a unique key.

### Initializing Maps
Maps can have their values assigned at creation time, just like arrays.

You can initialize maps two different ways:
```go
emails := map[string]string{
	"cory@gopherguides.com": "Cory LaNou",
	"mark@gopherguides.com": "Mark Bates",
	"tim@gopherguides.com":  "Tim Raymond",
}

var emails map[string]string
emails = make(map[string]string)
emails["cory@gopherguides.com"] = "Cory LaNou"
emails["mark@gopherguides.com"] = "Mark Bates"
emails["tim@gopherguides.com"] = "Tim Raymond"
```

### Length And Capacity
The **len** function can be used to find the length (the number of keys) in the map.  
Maps don't have a capacity, since they can grow as needed, so the **cap** function would raise an error.

### Map Values  
Map values can be set and retrieved using the the [] syntax.
```go
	beatles := map[string]string{}
	beatles["Paul"] = "bass"
	paul := beatles["Paul"]
```

### Iterating Maps
Maps can be iterated over in the same ways as arrays and slices.
``` go
for key, value := range beatles {
	fmt.Printf("%s plays %s\n", key, value)
}
```

NOTE: When iterating over a map with a range loop, the iteration order is not specified and is not guaranteed to be the same from one iteration to the next.

### Map Keys
Map keys must be comparable.   
You can't use functions, maps, or slices as the keys in your maps.

### Deleting Keys From A Map
The **delete** function can be used to remove a key, and its value, from a map.

```go 
delete(beatles, "John")
```
### Checking Map Values
```go
key := "Paul"
if value, ok := beatles[key]; ok { 

} else {

}
```

### Maps And Complex Values
Storing complex values (such as structs) in a map is a very common operation. However, updating those structs is not completely straightforward  

When updating complex map values, you have to retrieve the value, make your changes, and set it back into the map:

```
type Beatle struct {
	Name       string
	Instrument string
}

beatles := make(map[string]Beatle)

// Create Beatle dynamically
beatles["John"] = Beatle{Name: "John", Instrument: "guitar"}

// Create Beatle from an instance
b := Beatle{Name: "Paul", Instrument: "bass"}
beatles[b.Name] = b

b = Beatle{Name: "George"}
beatles[b.Name] = b

b = Beatle{Name: "Ringo", Instrument: "Drums"}
beatles[b.Name] = b

// Fix George:

// Lookup George
b, ok := beatles["George"]
if !ok {
	fmt.Println("couldn't find George...")
	return
}

// Update George
b.Instrument = "guitar"

// Update the map with the new value
beatles[b.Name] = b

fmt.Println(beatles)
```

*It's important to remember that when you use the **range** operator, that the loop variables are a **copy** of the value, and not a **reference** to the value

### Copy On Update  
When you insert into a map, it takes a copy of what you are inserting, and you no longer have a reference to the value stored in the map
```go
b := Beatle{Name: "Paul", Instrument: "bass"}
beatles[b.Name] = b

b.Instrument = "foo"
fmt.Println("variable:", b) // variable: {Paul foo}

lookup := beatles["Paul"]
fmt.Println("lookup:", lookup) // lookup: {Paul bass}
```

### Getting Map Keys
In java, we could use map.keySet(), values() and entrySet()  
Go does not provide a way to get just a list of keys, or values, from a map. To do this you must build that list yourself.
```go
	m := map[string]string{
		"foo": "bar",
		"bin": "baz",
		"boo": "woo",
	}

	keys := make([]string, 0, len(m))

	for k := range m {
		keys = append(keys, k)
	}
```

### Sorting Maps
maps are not sorted
- Sorting By Key
  - use the sort package to sort the keys:
   ```sort.Strings(keys)```
- Sorting By Value
  - The easiest way is to invert the map so you have the values as keys, and then do normal pattern of sorting by key.

### Maps And Concurrency
Maps are NOT concurrent safe! 
*As of Go 1.9, there is now a sync.Map available which in some cases may significantly reduce lock contention compared to a Go map paired with a separate Mutex or RWMutex.

Dave Cheney's [map](https://dave.cheney.net/tag/maps) articles.