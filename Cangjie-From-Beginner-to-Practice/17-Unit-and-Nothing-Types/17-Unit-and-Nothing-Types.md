# 17-Unit Type and Nothing Type

## Unit Type

For expressions that only care about side effects and not values, their type is `Unit`. For example, the `print` function, assignment expressions, compound assignment expressions, increment and decrement expressions, and loop expressions all have the type `Unit`.

The `Unit` type has only one value, which is also its literal: `()`. **Except for assignment, equality, and inequality checks**, the `Unit` type does not support other operations.

For example:

```javascript
    let a = println("Let's try") // Outputs "Let's try"
    println(a) // Outputs () 
```

For example:

```javascript
    var a = 10
    var b = a++
    println(b) // Outputs () 
```

For example:

```javascript
    var a = 10
    var b = a++
    println(b == ()) // Outputs true  
```

## Nothing Type

> Currently, the compiler does not allow explicit use of the Nothing type in places where types are used.

`Nothing` is a special type that does not contain any values, and the `Nothing` type is a subtype of all types.

The types of `break`, `continue`, `return`, and `throw` expressions are `Nothing`. When program execution reaches these expressions, the code after them will not be executed. Among them, `break` and `continue` can only be used in loop bodies, and `return` can only be used in function bodies.

```javascript
main() {
   let a = terminateProgram() // Returns Nothing type
}

func terminateProgram() {
    throw NegativeArraySizeException("Program actively encountered an error")
}
```