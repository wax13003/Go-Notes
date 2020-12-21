## **Docs** 
https://golang.org The official Go site has a lot of great articles, tutorials, and "mini"-books on Go.

https://godoc.org has literally all of the GoDoc's for every published package out there. It's an amazing resource.

## **Syntax and Types**
### Declaring Variables 
long variable declarations:
- not initializing  
```var i int```
- declaring and initializing   
```var i int = 1``` 

short variable declaration:
- Go will infer the data type   
```i := 1```
- wrap your value in your desired type  
```i := int64(1)```

All builtin types have a zero value.

nil type default value for many common types:
- maps
- slices
- functions
- channels
- interfaces
- errors

Naming Rules  
- one word (as in no spaces)
- only - letters, numbers, and _
- cannot begin with a number

case-sensitive 
- starts with a uppercase letter and can be accessed by other packages
- starts with a lowercase letter, and is only accessible inside the package it is declared in.
```
    var Email string  
    var password string  
```

Naming Rules  

- Underscores are not conventional  
**userName** vs user_name 
- use very terse (or short) variable names    
**i** vs index   
- acronyms should be capitalized     
**userID** vs UserId 

Multiple Assignment  
    ```j, k, l := "string", 2.05, 15```

Statically Typed  
the type is declared when declaring a variable:
```
var pi float64 = 3.14
var week int = 7
```
the data type is bound to the variable, each statement in the program is checked at compile time

Numeric Types  
Go has two types of numeric types. The first type is an **architecture independent** type, the second type is a **implementation** specific type.
<pre>
    uint32      unsigned 32-bit integers (0 to 4294967295)
    uint64      unsigned 64-bit integers (0 to 18446744073709551615)
    int8        signed  8-bit integers (-128 to 127)
    int16       signed 16-bit integers (-32768 to 32767)
</pre>
in addition, specific types:
<pre>
uint     either 32 or 64 bits
int      same size as uint
uintptr  an unsigned integer large enough to store the uninterpreted bits of a pointer value

* int32 and int are not the same type even though they may have the same size on a particular architecture.
</pre>  
notes:   
it's common in Go to use the implementation types like int or uint.   
If you know you won't exceed a specific size range, then picking an architecture-independent type can both increase speed decrease memory usage.   

Overflow Vs. Wraparound  
One happens vs. the other is dependent on if the value can be calculated at compile time or at runtime.

it will compile:
```
var maxUint8 uint8 = 255 // Max Uint8 size
fmt.Println(maxUint8)  // 255
fmt.Println(maxUint8 + 1) // it will wraparound to 0
```

At compile time, if the compiler can determine a value will be too large to hold in the data type specified, it will throw an overflow error. 

it will not compile:
```
var maxUint8 uint8 = 255 + 1
fmt.Println(maxUint8)  // constant 256 overflows uint8
```

There is no saturation in Go.
```
var maxUint8 uint8 = 11
maxUint8 = maxUint8 * 25
fmt.Println("new value:", maxUint8)  // 19
```
Booleans  
bool data type, the zero value is false.

Strings
back quotes `, you are creating a **raw** string literal. 
double quotes ", you are creating an **interpreted** string literal.

*You will almost always use interpreted string literals because they allow for escape characters within them.

Go supports UTF-8 characters out of the box  
```a := "Hello, 世界"```

Constants  
Constants are like variables, except they can't be modified once they have been declared.

Constants can only be a character, string, boolean, or numeric value.  
If you try to modify a constant after it was declared, you will get a compile time error

Untyped
Constants can be **untypted**. This can be useful when working with numbers such as integer type data. If the constant is **untyped**, it is explicitly converted, where **typed** constants are not.
```
const (
	year     = 365        // untyped
	leapYear = int32(366) // typed
)

func main() {
	hours := 24
	minutes := int32(60)
	fmt.Println(hours * year)       // multiplying an int and untyped
	fmt.Println(minutes * year)     // multiplying an int32 and untyped
	fmt.Println(minutes * leapYear) // multiplying both int32 types
}
```

Typed
If you try to use a typed constant with anything other than it's type, Go will throw a compile time error.

```go
const (
	leapYear = int32(366) // typed
)

func main() {
	hours := 24
	fmt.Println(hours * leapYear) // multiplying int and int32 types}
}

/*
	output:
	invalid operation: hours * leapYear (mismatched types int and int32)
*/
```

notes: 
untyped int can use it with any integer data type. 
typed int32 can only operate with int32 data types
untyped const or var will be converted to the type it is combined with for any mathematical operation

Iota
Go's iota identifier is used in const declarations to simplify definitions of incrementing numbers.  
* Order Matters With Iota
  * changing the order of the defined constants in an iota block will change their values.
* Shifting
  * You can use the shift operator and iota to create bitmasks as well:
  * 1 << iota
* Powers
  * Iota's can also help define patterns with powers calculations:
  * _  = 1 << (iota * 10) // ignore the first value
* Using Iota
  * best practice to avoid them 
  * re-write it as just constants and ensure that the code is not brittle

Print
fmt package to Print out our values
* All Print statements can use verbs. 
```
    // Use %v to print the value
    // Use %s to print a string
    // Use %q to quote a string
    // Use %d to print a decimal (int, int32, etc)
    // Use %T to print out the data type of the variable
    // Use \ to escape a character, like a quote:  \"
    // Use \n to print a new line (line return)
    // Use \t to insert a tab
    // Use %+v to print the name and value
```
Structs  
A struct is a collection of fields, often called members (or attributes).
* Defining A Struct
    * Structs may have zero or more fields.
    ```go
    type User struct {
        Name  string
        Email string
    }
    ```
* Initializing Structs
    Without initial values:
    ```u := User{}```
    With initial values:
    ```go
    u := User{
	    Name:  "Homer Simpson",
	    Email: "homer@example.com",
    }  
    u := User{Email: "marge@example.com"} //set as many(as few)
    u.Name = "Marge Simpson"
    ``` 

    you can instantiate and initialize a struct in one line, but bad practice as it can lead to future undesired refactoring.
* Struct Tags
    * metadata attached to fields
    ```go
    type User struct {
        Name string `example:"name"`
    }   
    ```
    * 
    ```go 
    type User struct {
        ID       int    `json:"id"`
        Name     string `json:"name"`
        Phone    string `json:"phone,omitempty"`
        Password string `json:"-"`
    }

    func main() {
        u := User{
            ID:       1,
            Name:     "Rob Pile",
            Password: "goIsAwesome",
        }

        err := json.NewEncoder(os.Stdout).Encode(&u)
        if err != nil {
            log.Fatal(err)
        }
    }

    * special case of omitempty to state that we don not want the Phone field encoded if the value is empty, as well as the - to direct the encoder to never encode the Password field.
    ```

