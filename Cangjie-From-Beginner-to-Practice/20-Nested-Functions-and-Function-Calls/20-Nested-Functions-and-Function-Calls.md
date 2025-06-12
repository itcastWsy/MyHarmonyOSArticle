# 20-Nested Functions and Function Calls

# Nested Functions

Functions defined at the top level of a source file are called global functions. Functions defined within a function body are called nested functions.

Example: Function `foo` defines a nested function `nestAdd` inside it. You can call the nested function `nestAdd` within `foo`, and you can also return the nested function `nestAdd` as a return value and call it outside of `foo`:

```javascript
func foo() {
    func nestAdd(a: Int64, b: Int64) {
        a + b + 3
    }

    println(nestAdd(1, 2))  // 6

    return nestAdd
}

main() {
    let f = foo()
    let x = f(1, 2)
    println("result: ${x}")
}
```

The program will output:

```javascript
6;
result: 6;
```

# Function Calls

Functions defined at the top level of a source file are called global functions. Functions defined within a function body are called nested functions.

Example: Function `foo` defines a nested function `nestAdd` inside it. You can call the nested function `nestAdd` within `foo`, and you can also return the nested function `nestAdd` as a return value and call it outside of `foo`:

```javascript
func foo() {
    func nestAdd(a: Int64, b: Int64) {
        a + b + 3
    }

    println(nestAdd(1, 2))  // 6

    return nestAdd
}

main() {
    let f = foo()
    let x = f(1, 2)
    println("result: ${x}")
}
```

The program will output:

```javascript
6;
result: 6;
```
