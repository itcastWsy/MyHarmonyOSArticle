# 19-Function Types

In the Cangjie programming language, functions are first-class citizens, which can be used as function parameters or return values, and can also be assigned to variables. Therefore, functions themselves have types, called function types.

Function types are composed of the parameter types and return type of the function, with parameter types and return type connected using `->`. Parameter types are enclosed in parentheses `()`, and there can be 0 or more parameters. If there are more than one parameter, parameter types are separated by commas (`,`).

For example:

```javascript
func hello(): Unit {
    println("Hello!")
}
```

The above example defines a function named hello, whose type is `() -> Unit`, indicating that the function has no parameters and returns type `Unit`.

Here are some other examples:

- Example: Function named `display`, whose type is `(Int64) -> Unit`, indicating that the function has one parameter of type `Int64` and returns type `Unit`.

  ```javascript
  func display(a: Int64): Unit {
      println(a)
  }
  ```

- Example: Function named `add`, whose type is `(Int64, Int64) -> Int64`, indicating that the function has two parameters, both of type `Int64`, and returns type `Int64`.

  ```javascript
  func add(a: Int64, b: Int64): Int64 {
      a + b
  }
  ```

- Example: Function named `returnTuple`, whose type is `(Int64, Int64) -> (Int64, Int64)`, with two parameters both of type `Int64`, and return type being tuple type: `(Int64, Int64)`.

  ```javascript
  func returnTuple(a: Int64, b: Int64): (Int64, Int64) {
      (a, b)
  }
  ```

## Type Parameters for Function Types

You can mark explicit type parameter names for function types. In the example below, `name` and `price` are `type parameter names`.

```javascript
func showFruitPrice(name: String, price: Int64) {
    println("fruit: ${name} price: ${price} yuan")
}

main() {
    let fruitPriceHandler: (name: String, price: Int64) -> Unit
    fruitPriceHandler = showFruitPrice
    fruitPriceHandler("banana", 10)
}
```

Additionally, for a function type, you can only uniformly write type parameter names, or uniformly not write type parameter names; they cannot exist alternately.

```javascript
let handler: (name: String, Int64) -> Int64   // Error
```

## Function Types as Parameter Types

Example: Function named `printAdd`, whose type is `((Int64, Int64) -> Int64, Int64, Int64) -> Unit`, indicating that the function has three parameters with types being function type `(Int64, Int64) -> Int64` and two `Int64` respectively, and returns type `Unit`.

```javascript
func printAdd(add: (Int64, Int64) -> Int64, a: Int64, b: Int64): Unit {
    println(add(a, b))
}
```

## Function Types as Return Types

Function types can serve as the return type of another function.

In the example below, the function named `returnAdd` has type `() -> (Int64, Int64) -> Int64`, indicating that the function has no parameters and returns a function type `(Int64, Int64) -> Int64`. Note that `->` is right-associative.

```javascript
func add(a: Int64, b: Int64): Int64 {
    a + b
}

func returnAdd(): (Int64, Int64) -> Int64 {
    add
}

main() {
    var a = returnAdd()
    println(a(1,2))
}
```

## Function Types as Variable Types

Function names themselves are also expressions, and their type is the corresponding function type.

```javascript
func add(p1: Int64, p2: Int64): Int64 {
    p1 + p2
}

let f: (Int64, Int64) -> Int64 = add
```

In the above example, the function name is `add`, and its type is `(Int64, Int64) -> Int64`. Variable `f` has the same type as `add`, and `add` is used to initialize `f`.

If a function is overloaded in the current scope (see [Function Overloading](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/function/function_overloading.html)), then directly using the function name as an expression may cause ambiguity. If ambiguity occurs, the compiler will report an error, for example:

```javascript
func add(i: Int64, j: Int64) {
    i + j
}

func add(i: Float64, j: Float64) {
    i + j
}

main() {
    var f = add   // Error, ambiguous function 'add'
    var plus: (Int64, Int64) -> Int64 = add  // OK
}
```