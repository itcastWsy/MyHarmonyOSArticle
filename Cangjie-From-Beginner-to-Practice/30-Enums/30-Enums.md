# 30-Enums

The `enum` type provides a way to define a type by enumerating all possible values of this type, for example:

1. Monday to Sunday can be an enum
2. Different color compositions can be an enum
3. A list of watched movies can be an enum

## Enum Definition

When defining an `enum`, you need to list all its possible values one by one. We call these values the constructors (or `constructor`) of the `enum`.

```javascript
enum RGBColor {
    | Red | Green | Blue
}
```

**enum** is the keyword for defining enums, **RGBColor** is the name of this enum type, **Red**, **Green**, **Blue** are the optional values in the enum. They are separated by **|**, where the **|** before **Red** is optional.

```javascript
enum RGBColor {
    Red | Green | Blue
}
```

## Enum Constructors

Constructors in `enum` can also carry several (at least one) parameters, called parameterized constructors. For example, constructors in `enum` can also carry several (at least one) parameters, called parameterized constructors.

```javascript
enum RGBColor {
    | Red(UInt8) | Green(UInt8) | Blue(UInt8)
}
```

## `enum` Supports Recursive Definition

```javascript
enum Expr {
    | Num(Int64)
    | Add(Expr, Expr)
    | Sub(Expr, Expr)
}

main() {
    let a = Expr.Add(Expr.Num(23), Expr.Num(Int64(42)))
}
```

## `enum` Defining Member Functions and Operator Functions

But constructors, member functions, and member properties cannot have the same name.

```javascript
enum RGBColor {
    | Red | Green | Blue

    public static func printType() {
        print("RGBColor")
    }
}
```

## Using Enums

After defining an `enum` type, you can create instances of this type (i.e., `enum` values). `enum` values can only take one of the constructors defined in the `enum` type. `enum` has no constructor functions. You can construct an `enum` value through `TypeName.Constructor`, or directly using the constructor (for parameterized constructors, you need to pass arguments).

```javascript
enum RGBColor {
    | Red | Green | Blue(UInt8)
}

main() {
    let r = RGBColor.Red
    let g = Green
    let b = Blue(100)
}
```

When omitting the type name, the `enum` constructor's name may conflict with type names, variable names, or function names. In this case, you must add the `enum` type name to use the `enum` constructor, otherwise only the same-named type, variable, or function definition will be selected.

In the following example, only constructor `Blue(UInt8)` can be used without the type name. `Red` and `Green(UInt8)` will both have name conflicts and cannot be used directly - the type name `RGBColor` must be added.

```javascript
let Red = 1

func Green(g: UInt8) {
    return g
}

enum RGBColor {
    | Red | Green(UInt8) | Blue(UInt8)
}

let r1 = Red                 // Will choose 'let Red'
let r2 = RGBColor.Red        // Ok: constructed by enum type name

let g1 = Green(100)          // Will choose 'func Green'
let g2 = RGBColor.Green(100) // Ok: constructed by enum type name

let b = Blue(100)            // Ok: can be uniquely identified as an enum constructor
```

In the following example, only constructor `Blue` will have name conflicts and cannot be used directly - the type name `RGBColor` must be added.

```javascript
class Blue {}

enum RGBColor {
    | Red | Green(UInt8) | Blue(UInt8)
}

let r = Red                 // Ok: constructed by enum type name

let g = Green(100)          // Ok: constructed by enum type name

let b = Blue(100)           // Will choose constructor of 'class Blue' and report an error
```
