## **Arrays** 
### Initializing An Array
**Array Zero Value**  
The zero value of each element in an array is the zero value for the type of elements in the array.

**Indexing Arrays**  
You will receive an error (either compile time or a panic) when trying to access an index of the array beyond it's size.

**Array Type**  
An array can only be of the type is was declared

**Setting Array Values**  
It is important to note that if you create two like arrays, and then set the value of one array to the other, they still continue to have their own memory space.
```
a1 := [2]string{"one", "two"}
a2 := [2]string{}
fmt.Println("a1:", a1)
fmt.Println("a2:", a2)
a2 = a1
a1[0] = "bob"
fmt.Println("a1:", a1)
fmt.Println("a2:", a2)

Output:
a1: [one two]
a2: [one two]
a1: [bob two]
a2: [one two]

```

Two Dimensional Arrays
Go's arrays are one-dimensional. To create an equivalent of a 2D array, it is necessary to define an array-of-arrays.
```
type Matrix [3][3]int
m := Matrix{
    {0, 0, 0},
    {1, 1, 1},
    {2, 2, 2},
}
```

The for Loop
In Go there is only one looping construct; the **for** loop.

Use it for **for, while, do while, do until,** etc...

Iterating Over Arrays
```
func main() {
	names := [4]string{"John", "Paul", "George", "Ringo"}

	for i := 0; i < len(names); i++ {
		fmt.Println(names[i])
	}
}
```

Continuing A Loop
The **continue** keyword allows us to go back to the start of the loop and stop executing the rest of the code in the **for** block.
```
for {
  if i == 3 {
    // go to the start of the loop
    continue
  }
  // do work
}
```
Breaking A Loop
To stop execution of a loop we can use the break keyword.
```
for {
  if i == 3 {
    // stop looping
    break
  }
  // do work
}
```

Do While Loop  
A **do while** loop is used in a situation where you want the loop to run at least 1 iteration, regardless of the **condition**.

Java-style:
```
do {
	task();
} while (condition);
```

Go:
```
var i int
for {
	fmt.Println(i)
	i += 2
	if i >= 3 {
		break
	}
}
```

The **range** Keyword  
Looping over arrays, and other collection types, is so common that Go created the range tool to simplify this code.

```
for i, n := range names {
    fmt.Printf("%d - %s\n", i, n)
}
```

