# 34-Refutability of Pattern Matching

Patterns can be divided into two categories: `refutable` patterns and `irrefutable` patterns.

1. `refutable` `/rɪˈfjuːtəbl/` refutable, meaning the pattern **may not match successfully**
2. `irrefutable` irrefutable, meaning the pattern will definitely succeed

## Constant Patterns

Constant patterns are `refutable` patterns. For example, in the following example, `1` in the first case and `2` in the second case may not be equal to the value of `x`.

```javascript
func constPat(x: Int64) {
    match (x) {
        case 1 => "one"
        case 2 => "two"
        case _ => "_"
    }
}
```

## Wildcard Patterns

Wildcard patterns are `irrefutable` patterns. For example, in the following example, regardless of the value of `x`, `_` can always match it.

```javascript
func wildcardPat(x: Int64) {
    match (x) {
        case _ => "_"
    }
}
```

## Binding Patterns

Binding patterns are `irrefutable` patterns. For example, in the following example, regardless of the value of `x`, the binding pattern `a` can always match it.

```javascript
func varPat(x: Int64) {
    match (x) {
        case a => "x = ${a}"
    }
}
```

## Tuple Patterns

Tuple patterns are `irrefutable` patterns if and only if every pattern they contain is an `irrefutable` pattern. For example, in the following example, both `(1, 2)` and `(a, 2)` may not match the value of `x`, so they are `refutable` patterns, while `(a, b)` can match any value of `x`, so it is an `irrefutable` pattern.

```javascript
func tuplePat(x: (Int64, Int64)) {
    match (x) {
        case (1, 2) => "(1, 2)"
        case (a, 2) => "(${a}, 2)"
        case (a, b) => "(${a}, ${b})"
    }
}
```

## Type Patterns

Type patterns are `refutable` patterns. For example, in the following example (assuming `Base` is the parent class of `Derived`, and `Base` implements interface `I`), the runtime type of `x` may be neither `Base` nor `Derived`, so both `a: Derived` and `b: Base` are `refutable` patterns.

```javascript
interface I {}
open class Base <: I {}
class Derived <: Base {}

func typePat(x: I) {
    match (x) {
        case a: Derived => "Derived"
        case b: Base => "Base"
        case _ => "Other"
    }
}
```

## enum Patterns

enum patterns are `irrefutable` patterns if and only if the corresponding `enum` type has only one parameterized constructor, and the other patterns contained in the enum pattern are also `irrefutable` patterns. For example, for the `E1` and `E2` definitions in the following example, `A(1)` in function `enumPat1` is a `refutable` pattern, while `A(a)` is an `irrefutable` pattern; and both `B(b)` and `C(c)` in function `enumPat2` are `refutable` patterns.

```javascript
enum E1 {
    A(Int64)
}

enum E2 {
    B(Int64) | C(Int64)
}

func enumPat1(x: E1) {
    match (x) {
        case A(1) => "A(1)"
        case A(a) => "A(${a})"
    }
}

func enumPat2(x: E2) {
    match (x) {
        case B(b) => "B(${b})"
        case C(c) => "C(${c})"
    }
}
```