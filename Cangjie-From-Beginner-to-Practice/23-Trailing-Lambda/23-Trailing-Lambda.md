# 23-Trailing Lambda

Trailing lambda can make function calls look like built-in language syntax, enhancing the language's extensibility.

When the last formal parameter of a function is a function type, and the corresponding actual parameter in the function call is a lambda, we can use trailing lambda syntax to place the lambda at the end of the function call, outside the parentheses.

For example, in the following example we define a `myIf` function whose first parameter is of type `Bool` and second parameter is of function type. When the value of the first parameter is `true`, it returns the value after calling the second parameter, otherwise it returns `0`. When calling `myIf`, you can call it like a normal function, or you can call it using trailing lambda syntax.

```javascript
func myIf(a: Bool, fn: () -> Int64) {
    if(a) {
        fn()
    } else {
        0
    }
}

func test() {
    myIf(true, { => 100 }) // General function call

    myIf(true) {        // Trailing closure call
        100
    }
}
```

When a function call has only one lambda argument, we can also omit the `()` and write only the lambda.

Example:

```javascript
func f(fn: (Int64) -> Int64) { fn(1) }

func test() {
    f { i => i * i }
}
```