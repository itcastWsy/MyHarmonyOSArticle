# Subtype Relationships

Like other object-oriented languages, the Cangjie language provides subtype relationships and subtype polymorphism. Examples include (but are not limited to the following use cases):

- If a function's formal parameter is of type `T`, then the actual type of the argument passed when calling the function can be either `T` or a subtype of `T` (strictly speaking, subtypes of `T` already include `T` itself, same below).
- If the variable on the left side of an assignment expression `=` is of type `T`, then the actual type of the expression on the right side of `=` can be either `T` or a subtype of `T`.
- If the user-annotated return type in a function definition is `T`, then the type of the function body (as well as the types of all `return` expressions within the function body) can be either `T` or a subtype of `T`.

So how do we determine whether two types have a subtype relationship? We will explain this below.

## Subtype Relationships from Class Inheritance

After inheriting a class, the subclass becomes a subtype of the parent class. In the following code, `Sub` is a subtype of `Super`.

```javascript
open class Super { }
class Sub <: Super { }
```

## Subtype Relationships from Interface Implementation

After implementing an interface (including extension implementation), the type that implements the interface becomes a subtype of the interface. In the following code, `I3` is a subtype of `I1` and `I2`, `C` is a subtype of `I1`, and `Int64` is a subtype of `I2`:

```javascript
interface I1 { }
interface I2 { }

interface I3 <: I1 & I2 { }

class C <: I1 { }

extend Int64 <: I2 { }
```

Note that some cross-extension type assignment scenarios with downward type conversion (`is` or `as`) are not currently supported and may result in judgment failures, as shown in the following example:

```javascript
// file1.cj
package p1

public class A{}

public func get(): Any {
    return A()
}

// =====================
// file2.cj
import p1.*

interface I0 {}

extend A <: I0 {}

main() {
    let v: Any = get()
    println(v is I0) // Cannot correctly determine type, printed content is uncertain
}
```

## Subtype Relationships of Tuple Types

Tuple types in the Cangjie language also have subtype relationships. Intuitively, if each element type of a tuple `t1` is a subtype of the corresponding position element type of another tuple `t2`, then the type of tuple `t1` is also a subtype of the type of tuple `t2`. For example, in the following code, since `C2 <: C1` and `C4 <: C3`, we also have `(C2, C4) <: (C1, C3)` and `(C4, C2) <: (C3, C1)`.

```javascript
open class C1 { }
class C2 <: C1 { }

open class C3 { }
class C4 <: C3 { }

let t1: (C1, C3) = (C2(), C4()) // OK
let t2: (C3, C1) = (C4(), C2()) // OK
```

## Subtype Relationships of Function Types

In the Cangjie language, functions are first-class citizens, and function types also have subtype relationships: given two function types `(U1) -> S2` and `(U2) -> S1`, `(U1) -> S2 <: (U2) -> S1` if and only if `U2 <: U1` and `S2 <: S1` (note the order). For example, the following code defines two functions `f : (U1) -> S2` and `g : (U2) -> S1`, and the type of `f` is a subtype of the type of `g`. Since the type of `f` is a subtype of `g`, anywhere `g` is used in the code can be replaced with `f`.

```javascript
open class U1 { }
class U2 <: U1 { }

open class S1 { }
class S2 <: S1 { }


func f(a: U1): S2 { S2() }
func g(a: U2): S1 { S1() }

func call1() {
    g(U2()) // Ok.
    f(U2()) // Ok.
}

func h(lam: (U2) -> S1): S1 {
    lam(U2())
}

func call2() {
    h(g) // Ok.
    h(f) // Ok.
}
```

For the above rule, the `S2 <: S1` part is easy to understand: the result data produced by function calls will be used by subsequent programs. Function `g` can produce result data of type `S1`, function `f` can produce results of type `S2`, and the result data produced by `g` should be replaceable by the result data produced by `f`, so we require `S2 <: S1`.

For the `U2 <: U1` part, it can be understood this way: before a function call produces a result, it should be callable itself. The actual argument type in function calls is fixed, and when the formal parameter type requirement is more relaxed, it can still be called, but when the formal parameter type requirement is stricter, it may not be callable—for example, given the definitions in the above code, `g(U2())` can be replaced with `f(U2())` precisely because the actual argument type `U2` requirement is stricter than the formal parameter type `U1`.

## Always Valid Subtype Relationships

In the Cangjie language, some preset subtype relationships are always valid:

- A type `T` is always a subtype of itself, i.e., `T <: T`.
- The `Nothing` type is always a subtype of any other type `T`, i.e., `Nothing <: T`.
- Any type `T` is a subtype of the `Any` type, i.e., `T <: Any`.
- Any type defined by `class` is a subtype of `Object`, i.e., if there is `class C {}`, then `C <: Object`.

## Subtype Relationships from Transitivity

Subtype relationships have transitivity. In the following code, although only `I2 <: I1`, `C <: I2`, and `Bool <: I2` are described, according to the transitivity of subtypes, there are also implicit subtype relationships `C <: I1` and `Bool <: I1`.

```javascript
interface I1 { }
interface I2 <: I1 { }

class C <: I2 { }

extend Bool <: I2 { }
```

## Subtype Relationships of Generic Types

Generic types also have subtype relationships. See the [Subtype Relationships of Generic Types](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/generic/generic_subtype.html) chapter for details.