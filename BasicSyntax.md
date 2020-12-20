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