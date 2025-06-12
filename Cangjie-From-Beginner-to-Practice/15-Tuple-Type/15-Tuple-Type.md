# 15-Tuple Type

Tuples (`/ˈtjʊpəl; ˈtʌpəl/`) function similarly to arrays, both managing a group of data and accessing elements through indices, but there are also differences:

1. Tuples can store different types of data
2. Once the data types and number of elements in a tuple are specified, they cannot be changed



## Basic Usage

```javascript
    let tuple = (0, "abc", true) // Specifies the number and types of elements
    println(tuple[0]) // 0
    println(tuple[1]) // "abc"
    println(tuple[2]) // true
```

Cannot be modified

```javascript
tuple[0] = 100 // error: 'tuple element' can not be assigned
```

You can also specify the type and assign values

```javascript
let x: (Int64, Float64) = (3, 3.141592)
let y: (Int64, Float64, String) = (3, 3.141592, "PI")
```

## Tuple Destructuring

Tuples support quick access to their contents through destructuring.

**Without using destructuring**

```javascript
    let tuple = (0, "abc", true)
    let a = tuple[0] // 0 
    let b = tuple[1] // "abc"
    let c = tuple[2] // true 
```

**Using destructuring**

```javascript
    let tuple = (0, "abc", true)
    let (a, b, c) = tuple  // a = 0 ,  b = "abc" , c = true
```

If we only want certain values from the tuple and want to ignore others, we can use `_` as a placeholder.

```javascript
    let tuple = (0, "abc", true)
    let (a, _, c) = tuple
    println(a) // 0 
    println(c) // true 
    println(b) // error: undeclared identifier 'b' 
```