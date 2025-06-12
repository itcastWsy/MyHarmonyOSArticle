# 36-if-let Expressions

The `if-let` expression first evaluates the expression on the right side of `<-` in the condition. If this value can match the pattern on the left side of `<-`, it executes the `if` branch; otherwise, it executes the `else` branch (which can be omitted). For example:

```javascript
main() {
    let result = Option<Int64>.Some(2023)

    if (let Some(value) <- result) {
        println("Operation successful, return value is: ${value}")
    } else {
        println("Operation failed")
    }
}
```

Running the above program will output:

```text
Operation successful, return value is: 2023
```

For the above program, if the initial value of `result` is modified to `Option<Int64>.None`, the pattern matching of `if-let` will fail, and the `else` branch will be executed:

```javascript
main() {
    let result = Option<Int64>.None

    if (let Some(value) <- result) {
        println("Operation successful, return value is: ${value}")
    } else {
        println("Operation failed")
    }
}
```

Running the above program will output:

```text
Operation failed
```