# 46-Generics-Enum-Subtype-Relationships

## Generic Enums

In the Cangjie programming language, one of the most widely used examples of generic enum declarations is the `Option` type. For a detailed description of `Option`, see the [Option Type](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/enum_and_pattern_match/option_type.html) chapter. The `Option` type is used to represent that a value of a certain type might be an empty value. Thus, `Option` can be used to represent failure in computation on a certain type. Since what type the failure occurs on is uncertain, it's obvious that `Option` is a generic type that needs to declare type parameters.

```javascript
package core // `Option` is defined in core.

public enum Option<T> {
      Some(T)
    | None

    public func getOrThrow(): T {
        match (this) {
            case Some(v) => v
            case None => throw NoneValueException()
        }
    }
    ...
}
```

As you can see, `Option<T>` is divided into two cases: one is `Some(T)`, used to represent a normal return result, and the other is `None`, used to represent an empty result. The `getOrThrow` function will return the internal value of `Some(T)`, with the return result being of type `T`. If the parameter is None, it directly throws an exception.

For example: if we want to define a safe division, because division computation can fail. If the divisor is 0, return `None`; otherwise, return a result wrapped with `Some`:

```javascript
func safeDiv(a: Int64, b: Int64): Option<Int64> {
    var res: Option<Int64> = match (b) {
                case 0 => None
                case _ => Some(a/b)
            }
    return res
}
```

This way, when the divisor is 0, the program will not throw an arithmetic exception due to division by zero during execution.

## Subtype Relationships of Generic Types

Instantiated generic types also have subtype relationships. For example, when we write the following code:

```javascript
interface I<X, Y> { }

class C<Z> <: I<Z, Z> { }
```

According to line 3, we know that `C<Bool> <: I<Bool, Bool>` and `C<D> <: I<D, D>`, etc. Line 3 here can be interpreted as "for all types `Z` (without type variables), `C<Z> <: I<Z, Z>` holds".

But for the following code:

```javascript
open class C { }
class D <: C { }

interface I<X> { }
```

`I<D> <: I<C>` does not hold (even though `D <: C` holds), because in the Cangjie language, user-defined type constructors are **invariant** at their type parameter positions.

The specific definition of variance is: if `A` and `B` are (instantiated) types, `T` is a type constructor with a type parameter `X` (e.g., `interface T<X>`), then:

- If `T(A) <: T(B)` if and only if `A = B`, then `T` is **invariant**.
- If `T(A) <: T(B)` if and only if `A <: B`, then `T` is **covariant** at `X`.
- If `T(A) <: T(B)` if and only if `B <: A`, then `T` is **contravariant** at `X`.

Because in the current stage of Cangjie, all user-defined generic types are invariant at all their type variable positions, so given `interface I<X>` and types `A`, `B`, only when `A = B` can we get `I<A> <: I<B>`; conversely, if we know `I<A> <: I<B>`, we can also deduce `A = B` (except for built-in types: built-in tuple types are covariant for each of their element types; built-in function types are contravariant at their parameter type positions and covariant at their return type positions).

Invariance limits some expressive power of the language, but also avoids some safety issues, such as the "covariant array runtime exception" problem (Java has this problem).