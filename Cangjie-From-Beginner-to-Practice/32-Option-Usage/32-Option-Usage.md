# 32-Option Usage

## Introduction

Because the Option type can represent both states of having a value and not having a value, and no value can also be understood as an error in some situations, the Option type can also be used for error handling.

## Example

```javascript
func getString(p: ?Int64): String {
    match (p) {
        case Some(x) => "${x}"
        case None => "none"
    }
}

main() {
    let a = Some(1)
    let b: ?Int64 = None
    let r1 = getString(a)
    let r2 = getString(b)
    println(r1) // 1
    println(r2) // "none"
}
```

## Common Option Scenarios

Because `Option` is a very commonly used type, Cangjie provides multiple ways to destructure it to facilitate the use of `Option` types. These include: pattern matching, error handling functions, coalescing operator (`??`), and question mark operator (`?`).

## Pattern Matching

Because the Option type is an enum type, you can use the enum pattern matching mentioned above to destructure `Option` values. For example, in the following example, function `getString` accepts a parameter of type `?Int64`. When the parameter is a `Some` value, it returns the string representation of the numeric value inside. When the parameter is a `None` value, it returns the string `"none"`.

```javascript
func getString(p: ?Int64): String{
    match (p) {
        case Some(x) => "${x}"
        case None => "none"
    }
}
main() {
    let a = Some(1)
    let b: ?Int64 = None
    let r1 = getString(a)
    let r2 = getString(b)
    println(r1)
    println(r2)
}
```

## getOrThrow Function

`getOrThrow` is a function of the Option-wrapped type, with the following characteristics. For example, `a.getOrThrow()`:

1. If a is of `Some` type, then `a.getOrThrow()` returns the data itself
2. If a is of `None` type, then `a.getOrThrow()` throws an exception

```javascript
main() {
    let a = Some(1)
    let b: ?Int64 = None
    let r1 = a.getOrThrow()
    println(r1)
    try {
        let r2 = b.getOrThrow()
    } catch (e: NoneValueException) {
        println("b is None")
    }
}
```

## Coalescing ??

`Coalescing` `/ˌkəʊəˈlesɪŋ/` means union and merging. For expression `a??b`:

1. If **a** is of `Some` type, return **a** itself
2. If **a** is of `None` type, return **b**

```javascript
main() {
    let a = Some(1)
    let b: ?Int64 = None
    let r1: Int64 = a ?? 0 //  r1 = 1
    let r2: Int64 = b ?? 0 //   r2 = 0
    println(r1) // 1
    println(r2) // 0
}
```

## Question Mark Operator ?

The question mark operator `?` can be understood as optional. Because using `Option` wraps normal data, it might be `None`, so when using `Option`-wrapped data and wanting to continue using its inner properties, you need to use `?`.

```javascript
struct R {
    public var a: Int64
    public init(a: Int64) {
        this.a = a
    }
}

main() {
    let r = R(100)
    let x = Some(r) // Equivalent to using Option.Some to wrap
    let y = Option<R>.None // Equivalent to using Option.None
    let r1 = x?.a // r1 = Option<Int64>.Some(100)
    let r2 = y?.a // r2 = Option<Int64>.None
    println(r1) // Some(100)
    println(r2) // None
}
```
