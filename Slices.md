## **Slices** 
### What Are Slices
- Similar to Arrays
- Fixed type
- Dynamically sized
- Flexible

```go
// create an array of names
namesArray := [4]string{"John", "Paul", "George", "Ringo"}
// create a slice of names
namesSlice := []string{"John", "Paul", "George", "Ringo"}
```
Iterating Over Slices  
Slices can be iterated over in the same ways that arrays can be.
```
for i, n := range names {
    fmt.Printf("%d - %s\n", i, n)
}
```

### Why Slices?
- Change a slice without an allocation
- Operate on subsections of slices easily
- Dynamically grow the size of a slice

Slices don't have a length in the declaration:
```
var array [5]string // array
var slice []string // slice
```

Slice Internals  
Think of a slice as having three members:
- Length
- Capacity
- Pointer to the underlying array
```go
type slice struct {
	Length   int
	Capacity int
	Array    [10]array
}
```

### Appending To Slices
The append keyword allows us to add elements to a slice.
```go
names := []string{}
names = append(names, "John")

// Append multiple items at once
names = append(names, "Sally", "George")

// Append an entire slice to another slice
moreNames := []string{"Bill", "Ginger", "Wilma"}
names = append(names, moreNames...)
```
### Len And Cap
```go
fmt.Println("len:", len(names)) // 5
fmt.Println("cap:", cap(names)) // 8
```
The **len** keyword tells us how many elements the slice actually has.

The **cap** keyword tells us the capacity of the slice, or how many elements it can have.

### What Happens When A Slice Grows

Len -> 4  
Cap -> 4  
Arr -> Array Pointer -> Underlying Array ABCD

If we append the values E, F, and G, it will force the slice to expand  

It will create a new underlying array, copy the original values into the new one, and add the new values as well.

Original Array          New Array
ABCD                    ABCDEFG  

new slice will look like:
Len -> 7  
Cap -> 8  
Arr -> Array Pointer -> Underlying Array ABCDEFG

If the original underlying array is not reference by any other part of the program, it will be marked for garbage collection.

### Making A Slice
Slices can also be created using the **make** keyword.
```go
a := []string{}  //this one is easier to read (and write)
b := make([]string, 0)
var c []string
```
### Make With Length And Capacity
```go
a := make([]int, 1, 3)
```
*It's important to remember that even though you allocated extra capacity, you can't access that capacity until you assign a value to it

### Make And Append
Be careful when using both make and append, as you may inadvertantly create zero values in your slice.

```go
	a := make([]string, 2)
	a = append(a, "foo", "bar") //["" "" "foo" "bar"]
```

### Two Dimensional Slices
Go's slices are one-dimensional. To create an equivalent of a 2D slice, it is necessary to define an slice-of-slices.

``` go
type Modules [][]string

course := Modules{
	[]string{"Chapter One: Syntax"},
	[]string{"Chapter Two: Arrays and Slices"},
	[]string{"Chapter Three: Maps", "Chapter Four: Packages"},
}
```

### Slice Subsets
 slice[starting_index : (starting_index + length)]
 ```go
 names := []string{"John", "Paul", "George", "Ringo"}
 
 fmt.Println(names) // [John Paul George Ringo]
 fmt.Println(names[1:3]) // [Paul George] - names[1:1+2]

// functionally equivalent
fmt.Println(names[2:len(names)]) // [George Ringo]
fmt.Println(names[2:])           // [George Ringo]

// functionally equivalent
fmt.Println(names[0:2]) // [John Paul]
fmt.Println(names[:2])  // [John Paul]
 ```

 ### Mutating Slice Subsets
 If you mutate the subset, you mutate the original slice.

 ### Copy
 You can use the copy keyword to make a copy without sharing reference to the original underlying array.

 ### [Slice Tricks](https://github.com/golang/go/wiki/SliceTricks)