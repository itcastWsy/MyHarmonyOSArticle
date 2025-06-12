# 26-const Functions and Constant Evaluation

Constant evaluation allows certain specific forms of expressions to be evaluated at compile time, which can reduce the computation needed during program runtime. This chapter mainly introduces the usage methods and rules of constant evaluation.

## const Functions and Constant Evaluation

Constant evaluation allows certain specific forms of expressions to be evaluated at compile time, which can reduce the computation needed during program runtime. This chapter mainly introduces the usage methods and rules of constant evaluation.

## const Variables

`const` variables are a special type of variable, modified by the keyword `const`, defined to complete evaluation at compile time, and immutable at runtime. For example, the following example defines the gravitational constant `G`:

```javascript
const G = 6.674e-11
```

`const` variables can omit type annotations, but cannot omit initialization expressions. `const` variables can be global variables, local variables, or static member variables. However, `const` variables cannot be defined in extensions. `const` variables can access all instance members of the corresponding type, and can also call all non-`mut` instance member functions of the corresponding type.

```javascript
struct Planet {
    const Planet(let mass: Float64, let radius: Float64) {}

    const func gravity(m: Float64, r: Float64) {
        G * mass * m / r**2
    }
}

main() {
    const myMass = 71.0
    const earth = Planet(5.972e24, 6.378e6)
    println(earth.gravity(myMass, earth.radius))
}
```

After `const` variables are initialized, all members of that type instance are `const` (deep `const`, including members of members), so they cannot be used as lvalues.

```javascript
main() {
    const myMass = 71.0
    myMass = 70.0 // Error, cannot assign to immutable value
}
```

## const Context and const Expressions

`const` context refers to `const` variable initialization expressions, which are always evaluated at compile time. Therefore, it is necessary to restrict the expressions allowed in `const` context to avoid side effects such as modifying global state and I/O, ensuring they can be evaluated at compile time.

`const` expressions have the capability to be evaluated at compile time. Expressions that satisfy the following rules are `const` expressions:

1. Literals of numeric types, `Bool`, `Unit`, `Rune`, `String` types (excluding interpolated strings).
2. `Array` literals where all elements are `const` expressions (cannot be `Array` type, can use `VArray` type), `tuple` literals.
3. `const` variables, `const` function formal parameters, local variables in `const` functions.
4. `const` functions, including function names declared with `const`, `lambda`s that meet `const` function requirements, and function expressions returned by these functions.
5. `const` function calls (including `const` constructors), where the function's expression must be a `const` expression, and all actual parameters must be `const` expressions.
6. `enum` constructor calls where all parameters are `const` expressions, and parameterless `enum` constructors.
7. Arithmetic expressions, relational expressions, bitwise operation expressions of numeric types, `Bool`, `Unit`, `Rune`, `String` types, where all operands must be `const` expressions.
8. `if`, `match`, `try`, control transfer expressions (including `return`, `break`, `continue`, `throw`), `is`, `as`. All expressions within these expressions must be `const` expressions.
9. Member access of `const` expressions (excluding property access), index access of `tuple`s.
10. `this` and `super` expressions in `const init` and `const` functions.
11. `const` instance member function calls of `const` expressions, where all actual parameters must be `const` expressions.

## const Functions

`const` functions are a special class of functions that have the capability to be evaluated at compile time. When these functions are called in a `const` context, they will perform calculations at compile time. In other non-`const` contexts, `const` functions will execute at runtime like ordinary functions.

The following example is a `const` function that calculates the distance between two points on a plane. `distance` uses `let` to define two local variables `dx` and `dy`:

```javascript
struct Point {
    const Point(let x: Float64, let y: Float64) {}
}

const func distance(a: Point, b: Point) {
    let dx = a.x - b.x
    let dy = a.y - b.y
    (dx ** 2 + dy ** 2) ** 0.5
}

main() {
    const a = Point(3.0, 0.0)
    const b = Point(0.0, 4.0)
    const d = distance(a, b) // Compiled at runtime
    println(d)
}
```

Compile and run output:

```
5.000000
```

Note:

1. `const` function declarations must be modified with `const`.
2. Global `const` functions and `static const` functions can only access externally declared `const` variables, including `const` global variables and `const` static member variables; other external variables are not accessible. `const init` functions and `const` instance member functions can access externally declared `const` variables as well as instance member variables of the current type.
3. All expressions in `const` functions must be `const` expressions, except for `const init` functions.
4. `const` functions can use `let` and `const` to declare new local variables. But `var` is not supported.
5. There are no special regulations for parameter types and return types in `const` functions. If the actual parameters of the function call do not meet the requirements of `const` expressions, then this function call cannot be used as a `const` expression, but can still be used as an ordinary expression.
6. `const` functions are not necessarily executed at compile time, for example, they can be called at runtime in non-`const` functions.
7. `const` functions and non-`const` functions follow the same overloading rules.
8. Numeric types, `Bool`, `Unit`, `Rune`, `String` types and `enum` support defining `const` instance member functions.
9. For `struct` and `class`, `const` instance member functions can only be defined if `const init` is defined. `const` instance member functions in `class` cannot be `open`. `const` instance member functions in `struct` cannot be `mut`.

Additionally, `const` functions can also be defined in interfaces, but are subject to the following rule restrictions:

1. For `const` functions in interfaces, the implementing type must also use `const` functions to be considered as implementing the interface.
2. For non-`const` functions in interfaces, the implementing type using either `const` or non-`const` functions is considered as implementing the interface.
3. `const` functions in interfaces, like `static` functions of interfaces, can only be used by constrained generic type variables or variables when the interface serves as a generic constraint.

In the following example, two `const` functions are defined in interface `I`, class `A` implements interface `I`, and the formal parameter type upper bound of generic function `g` is `I`.

```javascript
interface I {
    const func f(): Int64
    const static func f2(): Int64
}

class A <: I {
    public const func f() { 0 }
    public const static func f2() { 1 }
    const init() {}
}

const func g<T>(i: T) where T <: I {
    return i.f() + T.f2()
}

main() {
    println(g(A()))
}
```

## const init

If a `struct` or `class` defines a `const` constructor, then instances of this `struct`/`class` can be used in `const` expressions.

1. If the current type is `class`, it cannot have instance member variables declared with `var`, otherwise defining `const init` is not allowed. If the current type has a parent class, the current `const init` must call the parent class's `const init` (can be called explicitly or implicitly call the parameterless `const init`). If the parent class does not have `const init`, an error is reported.
2. If instance member variables of the current type have initial values, the initial values must be `const` expressions, otherwise defining `const init` is not allowed.
3. Within `const init`, assignment expressions can be used to assign values to instance member variables, but no other assignment expressions are allowed.

The difference between `const init` and `const` functions is that `const init` allows assignment to instance member variables (using assignment expressions).