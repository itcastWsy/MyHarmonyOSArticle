# 35-Match Expressions

Cangjie supports two types of `match` expressions: the first type contains a value to be matched, and the second type does not contain a value to be matched.

## Match Expressions with Values

```javascript
main() {
    let x = 0
    match (x) {
        case 1 => let r1 = "x = 1"
                  print(r1)
        case 0 => let r2 = "x = 0" // Matched.
                  print(r2)
        case _ => let r3 = "x != 1 and x != 0"
                  print(r3)
    }
}
```

`match` expressions require all matches to be exhaustive, meaning all possible values of the expression to be matched should be considered. When a `match` expression is not exhaustive, or when the compiler cannot determine whether it's exhaustive, a compilation error will occur.

### Pattern Guard

After the pattern in a `case` branch, you can use `pattern guard` to further judge the matched result. `pattern guard` is represented using `where cond`, requiring the expression `cond` to be of type `Bool`.

```javascript
enum RGBColor {
    | Red(Int16) | Green(Int16) | Blue(Int16)
}
main() {
    let c = RGBColor.Green(-100)
    let cs = match (c) {
        case Red(r) where r < 0 => "Red = 0"
        case Red(r) => "Red = ${r}"
        case Green(g) where g < 0 => "Green = 0" // Matched.
        case Green(g) => "Green = ${g}"
        case Blue(b) where b < 0 => "Blue = 0"
        case Blue(b) => "Blue = ${b}"
    }
    print(cs)
}
```

## Match Expressions without Values

Compared to `match` expressions with values to be matched, there is no expression to be matched after the keyword `match`, and after `case` there is no longer a `pattern`, but an expression of type `Bool`.

```javascript
main() {
    let x = -1
    match {
        case x > 0 => print("x > 0")
        case x < 0 => print("x < 0") // Matched.
        case _ => print("x = 0")
    }
}
```

## Types of Match Expressions

For `match` expressions (whether they have values to match or not):

- When the context has explicit type requirements, each `case` branch requires the code block after `=>` to be a subtype of the type required by the context;
- When the context has no explicit type requirements, the type of the `match` expression is the least common supertype of the types of the code blocks after `=>` in each `case` branch;
- When the value of the `match` expression is not used, its type is `Unit`, and there's no requirement for the branches to have a least common supertype.

Examples are provided below:

```javascript
let x = 2
let s: String = match (x) {
    case 0 => "x = 0"
    case 1 => "x = 1"
    case _ => "x != 0 and x != 1" // Matched.
}
```

In the above example, when defining variable `s`, its type is explicitly annotated as `String`, which is a case of explicit context type information. Therefore, each `case` requires the code block after `=>` to be a subtype of `String`. Obviously, the string literal types after `=>` in the above example all meet the requirements.

Let's look at an example without context type information:

```javascript
let x = 2
let s = match (x) {
    case 0 => "x = 0"
    case 1 => "x = 1"
    case _ => "x != 0 and x != 1" // Matched.
}
```

In the above example, when defining variable `s`, its type is not explicitly annotated. Since the type of the code block after `=>` in each `case` is `String`, the type of the `match` expression is `String`, and thus the type of `s` can be determined to be `String` as well.
