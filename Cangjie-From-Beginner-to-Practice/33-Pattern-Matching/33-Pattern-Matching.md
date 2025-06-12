# 33-Pattern Matching

Pattern matching mainly uses the keyword `match` to match appropriate values and then execute corresponding content. At the syntax level, it can be understood as `switch-case` in many programming languages.

## Classification of Pattern Matching

Based on different data types to be matched, it can be classified into the following categories:

1. Constant patterns
2. Wildcard patterns
3. Binding patterns
4. Tuple patterns
5. Type patterns
6. enum patterns
7. Nested combinations of patterns

## Constant Patterns

Constant patterns can be integer literals, floating-point literals, character literals, boolean literals, string literals (string interpolation is not supported), and Unit literals.

### Number Matching

> `|` indicates that multiple patterns can be connected

```javascript
main() {
    let score = 90
    let level = match (score) {
        case 0 | 10 | 20 | 30 | 40 | 50 => "D"
        case 60 => "C"
        case 70 | 80 => "B"
        case 90 | 100 => "A" // Matched.
        case _ => "Not a valid score"
    }
    println(level)
}
```

### Character Matching

When the target of pattern matching is a value with static type `Rune`, both `Rune` literals and single-character string literals can be used to represent constant patterns of `Rune` type literals.

```javascript
func translate(n: Rune) {
    match (n) {
        case "A" => 1
        case "B" => 2
        case "C" => 3
        case _ => -1
    }
}

main() {
    println(translate(r"C"))
}
```

### Byte Matching

A string literal representing an [ASCII](https://baike.baidu.com/item/ASCII/309296?fr=ge_ala) character can be used to represent a constant pattern of `Byte` type literals.

```javascript
func translate(n: Byte) {
    match (n) {
        case "1" => 1
        case "2" => 2
        case "3" => 3
        case _ => -1   // Wildcard pattern 
    }
}

main() {
    println(translate(51)) // UInt32(r'3') == 51
}
```

## Wildcard Patterns

`_` represents a wildcard pattern. When all the above conditions do not match, it will match the wildcard `_`.

```javascript
func translate(n: Byte) {
    match (n) {
        case "1" => 1
        case "2" => 2
        case "3" => 3
        case _ => -1
    }
}

main() {
    println(translate(51)) // UInt32(r'3') == 51
}
```

## Binding Patterns

Binding patterns are similar to wildcard patterns, but the difference is that they can bind the current variable that meets the condition to a certain variable.

Binding patterns use `id` to represent, where `id` is a valid identifier.

`id` is not a specific name, it can be understood as a placeholder. In practice, various variable names can be used to represent `id`.

Multiple can also be used, but only the first one will be matched.

```javascript
main() {
    let x = -10
    let y = match (x) {
        case 0 => "zero"
        case n => "x is not zero and x = ${n}" // Matched.   n represents id    
    }
    println(y)
}
```

Note that binding patterns cannot be shared with multiple patterns connected by `|`, nor can they appear nested in other patterns.

```javascript
main() {
    let opt = Some(0)
    match (opt) {
        case x | x => {} // Error, variable cannot be introduced in patterns connected by '|'
        case Some(x) | Some(x) => {} // Error, variable cannot be introduced in patterns connected by '|'
        case x: Int64 | x: String => {} // Error, variable cannot be introduced in patterns connected by '|'
    }
}
```

## Tuple Patterns

Tuple patterns are used for matching tuple values.

```javascript
main() {
    let tv = ("Alice", 24)
    let s = match (tv) {
        case ("Bob", age) => "Bob is ${age} years old"
        case ("Alice", age) => "Alice is ${age} years old" // Matched, "Alice" is a constant pattern, and 'age' is a variable pattern.
        case (name, 100) => "${name} is 100 years old"
        case (_, _) => "someone"
    }
    println(s)
}
```

## Type Patterns

Type patterns are used to determine whether the runtime type of a value is a subtype of a certain type. Type patterns have two forms: `_: Type` (nesting a wildcard pattern `_`) and `id: Type` (nesting a binding pattern `id`). The difference is that the latter will perform variable binding, while the former will not.

```javascript
open class Base {
    var a: Int64
    public init() {
        a = 10
    }
}

class Derived <: Base {
    public init() {
        a = 20
    }
}

main() {
    var d = Derived()
    var r = match (d) {
        case b: Base => b.a // Matched.
        case _ => 0
    }
    println("r = ${r}")
}
```

## enum Patterns

enum patterns are used to match instances of `enum` types.

```javascript
enum TimeUnit {
    | Year(UInt64)
    | Month(UInt64)
}

main() {
    let x = Year(2)
    let s = match (x) {
        case Year(n) => "x has ${n * 12} months" // Matched.
        case TimeUnit.Month(n) => "x has ${n} months"
    }
    println(s)
}
```

Using `|` to connect multiple enum patterns:

```javascript
enum TimeUnit {
    | Year(UInt64)
    | Month(UInt64)
}

main() {
    let x = Year(2)
    let s = match (x) {
        case Year(0) | Year(1) | Month(_) => "Ok" // Ok
        case Year(2) | Month(m) => "invalid" // Error, Variable cannot be introduced in patterns connected by '|'
        case Year(n: UInt64) | Month(n: UInt64) => "invalid" // Error, Variable cannot be introduced in patterns connected by '|'
    }
    println(s)
}
```

When using `match` expressions to match `enum` values, the patterns after `case` are required to cover all constructors in the enum type to be matched. If complete coverage is not achieved, the compiler will report an error:

```javascript
enum RGBColor {
    | Red | Green | Blue
}

main() {
    let c = Green
    let cs = match (c) { // Error, Not all constructors of RGBColor are covered.
        case Red => "Red"
        case Green => "Green"
    }
    println(cs)
}
```

## Nested Combinations of Patterns

Tuple patterns and enum patterns can nest arbitrary patterns.

```javascript
enum TimeUnit {
    | Year(UInt64)
    | Month(UInt64)
}

enum Command {
    | SetTimeUnit(TimeUnit)
    | GetTimeUnit
    | Quit
}

main() {
    let command = SetTimeUnit(Year(2022))
    match (command) {
        case SetTimeUnit(Year(year)) => println("Set year ${year}")
        case SetTimeUnit(Month(month)) => println("Set month ${month}")
        case _ => ()
    }
}
```