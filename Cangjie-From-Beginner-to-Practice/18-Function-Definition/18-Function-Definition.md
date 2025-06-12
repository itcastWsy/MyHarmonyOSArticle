# 18-Basic Function Usage

Functions are collections used for code reuse. When we find that certain business logic needs to be used repeatedly, we can implement it through functions. At the same time, functions also allow complex program structures to be split according to functionality, making the program more robust and reasonable.

## Function Definition

There are several different ways to define functions. Let's first look at a standard one.

```javascript
func add(a: Int64, b: Int64): Int64 {
    return a + b
}
```

1. `func` is the keyword for declaring a function
2. `a: Int64` - a is a parameter, which can be understood as a variable unique to the function, a placeholder, and Int64 is its type
3. b: Int64) : **Int64** - here Int64 indicates the return type
4. **return** indicates the value to be returned by the function, which is `a+b`

## Function Call

The fixed syntax for function calls is: **function_name()**

```javascript
let res = add(100, 200) // res = 300
```



**Complete Example**

```JavaScript

main() {
    let res = add(100, 200)
    println(res) // 300 
}

func add(a: Int64, b: Int64): Int64 {
    return a + b
}
```

## Function Parameters

In actual development, function parameters can:

1. Be omitted
2. Have specified names
3. Have default values



### Function Parameters Can Be Omitted

```javascript
main() {
    let res = add()
    println(res) // 300 
}

func add(): Int64 {
    return 300
}
```

### Function Parameters Can Have Specified Names

> Use ! to specify the parameter name
>
> Note: Named parameters must be placed after unnamed parameters!

```javascript
main() {
    // let res = add(100, 200) // Cannot directly pass 200, because the parameter name is fixed as b
    let res = add(100, b: 200) // The parameter name is fixed as b
    println(res) // 300 
}

func add(a: Int64, b!: Int64): Int64 {
    return a + b
}
```

### Function Parameters Can Have Default Values

> Default values can only be set for named parameters, not for unnamed parameters.

```javascript
main() {
    let res = add(100) // b has been assigned a default value. When b is not passed, b = 400; if passed, b = the passed content
    println(res) // 500 
}

func add(a: Int64, b!: Int64 = 400): Int64 {
    return a + b
}

```

## Function Return Values

If you want the function to return content when it finishes running, you can use return, but you can also choose not to use it.

**With Return Value**

```javascript
main() {
    let res = add()
    println(res) // 300 
}

func add() {
    return 300 // With return value
}
```

**With Return Value, Omitting return**

```javascript
main() {
    let res = add()
    println(res) // 300 
}

func add() {
    300 // With return value
}
```



**Without Explicit Return Value**

```javascript
main() {
    let res = add()
    println(res) // () 
}

func add() {
     // No return value, returns unit type, represented as ()
}
```

## Summary

In actual development, we can customize our functions according to requirements