# Interface

An interface is used to define an abstract type that contains no data but can define the behavior of a type. If a type declares that it implements an interface and implements all the members of that interface, it is said to implement the interface.

Interface members can include:

- Member functions
- Operator overload functions
- Member properties

These members are all abstract, requiring implementing types to have corresponding member implementations.

## Interface Definition

A simple interface definition is as follows:

```javascript
interface I { // 'open' modifier is optional.
    func f(): Unit
}
```

An interface is declared using the keyword `interface`, followed by the interface identifier `I` and the interface members. Interface members can be modified by the `open` modifier, and the `open` modifier is optional.

When interface `I` declares a member function `f`, to implement `I` for a type, a corresponding `f` function must be implemented in that type.

Because `interface` has `open` semantics by default, the `open` modifier is optional when defining an `interface`.

As shown in the code below, a `class Foo` is defined, and `Foo <: I` declares that `Foo` implements interface `I`.

`Foo` must contain implementations of all members declared by `I`, i.e., it needs to define an `f` of the same type, otherwise it will fail to compile due to not implementing the interface.

```javascript
class Foo <: I {
    public func f(): Unit {
        println("Foo")
    }
}

main() {
    let a = Foo()
    let b: I = a
    b.f() // "Foo"
}
```

When a type implements an interface, that type becomes a subtype of the interface.

For the above example, `Foo` is a subtype of `I`, so any instance of type `Foo` can be used as an instance of type `I`.

In `main`, we assign a variable `a` of type `Foo` to a variable `b` of type `I`. Then when we call function `f` in `b`, it will print the version of `f` implemented by `Foo`. The program output is:

```text
Foo
```

`interface` can also use the `sealed` modifier to indicate that the `interface` can only be inherited, implemented, or extended within the package where the `interface` is defined. `sealed` already implies `public`/`open` semantics, so when defining a `sealed interface`, if `public`/`open` modifiers are provided, the compiler will warn. Sub-interfaces inheriting from a `sealed` interface or classes implementing a `sealed` interface can still be modified with `sealed` or not use the `sealed` modifier. If a sub-interface of a `sealed` interface is modified with `public` and not modified with `sealed`, then its sub-interface can be inherited, implemented, or extended outside the package. Types that inherit or implement sealed interfaces may not be modified with `public`.

```javascript
package A
public interface I1 {}
sealed interface I2 {}         // OK
public sealed interface I3 {}  // Warning, redundant modifier, 'sealed' implies 'public'
sealed open interface I4 {}    // Warning, redundant modifier, 'sealed' implies 'open'

class C1 <: I1 {}
public open class C2 <: I1 {}
sealed class C3 <: I2 {}
extend Int64 <: I2 {}
package B
import A.*

class S1 <: I1 {}  // OK
class S2 <: I2 {}  // Error, I2 is sealed interface, cannot be inherited here.
```

Through this constraint capability of interfaces, we can agree on common functionality for a series of types, achieving the purpose of abstracting functionality.

For example, in the code below, we can define a Flyable interface and have other classes with Flyable properties implement it.

```javascript
interface Flyable {
    func fly(): Unit
}

class Bird <: Flyable {
    public func fly(): Unit {
        println("Bird flying")
    }
}

class Bat <: Flyable {
    public func fly(): Unit {
        println("Bat flying")
    }
}

class Airplane <: Flyable {
    public func fly(): Unit {
        println("Airplane flying")
    }
}

func fly(item: Flyable): Unit {
    item.fly()
}

main() {
    let bird = Bird()
    let bat = Bat()
    let airplane = Airplane()
    fly(bird)
    fly(bat)
    fly(airplane)
}
```

Compiling and executing the above code, we will see the following output:

```text
Bird flying
Bat flying
Airplane flying
```

Interface members can be instance or static. The above example has already shown the role of instance member functions. Now let's look at the role of static member functions.

Static member functions are similar to instance member functions in that they both require implementing types to provide implementations.

For example, in the following example, we define a `NamedType` interface that contains a static member function `typename` to get the string name of each type.

This way, when other types implement the `NamedType` interface, they must implement the `typename` function, and then we can safely get the type name on subtypes of `NamedType`.

```javascript
interface NamedType {
    static func typename(): String
}

class A <: NamedType {
    public static func typename(): String {
        "A"
    }
}

class B <: NamedType {
    public static func typename(): String {
        "B"
    }
}

main() {
    println("the type is ${ A.typename() }")
    println("the type is ${ B.typename() }")
}
```

The program output is:

```text
the type is A
the type is B
```

Static member functions (or properties) in interfaces can have no default implementation or can have default implementations.

When they have no default implementation, they cannot be accessed through the interface type name. For example, in the following code, directly accessing the `typename` function of `NamedType` will cause a compilation error because `NamedType` does not have an implementation of the `typename` function.

```javascript
main() {
    NamedType.typename() // Error
}
```

Static member functions (or properties) in interfaces can also have default implementations. When another type inherits an interface with default static function (or property) implementations, that type may not need to implement this static member function (or property), and the function (or property) can be directly accessed through the interface name and the type name. As in the following use case, the member function `typename` of `NamedType` has a default implementation, and in `A` it does not need to be re-implemented, and it can also be directly accessed through the interface name and the type name.

```javascript
interface NamedType {
    static func typename(): String {
        "interface NamedType"
    }
}

class A <: NamedType {}

main() {
    println(NamedType.typename())
    println(A.typename())
    0
}
```

The program output is:

```text
interface NamedType
interface NamedType
```

Usually we use these static members in generic functions through generic constraints.

For example, the following `printTypeName` function, when we constrain the generic type parameter `T` to be a subtype of `NamedType`, we need to ensure that all static member functions (or properties) in the instantiated type of `T` must have implementations, to ensure that we can use `T.typename` to access the implementation of the generic type parameter, achieving our purpose of abstracting static members. See the [Generics](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/generic/generic_overview.html) chapter for details.

```javascript
interface NamedType {
    static func typename(): String
}

interface I <: NamedType {
    static func typename(): String {
        f()
    }
    static func f(): String
}

class A <: NamedType {
    public static func typename(): String {
        "A"
    }
}

class B <: NamedType {
    public static func typename(): String {
        "B"
    }
}

func printTypeName<T>() where T <: NamedType {
    println("the type is ${ T.typename() }")
}

main() {
    printTypeName<A>() // Ok
    printTypeName<B>() // Ok
    printTypeName<I>() // Error, 'I' must implement all static function. Otherwise, an unimplemented 'f' is called, causing problems.
}
```

Note that interface members are modified with `public` by default and cannot declare additional access control modifiers, and also require implementing types to use `public` implementations.

```javascript
interface I {
    func f(): Unit
}

open class C <: I {
    protected func f() {} // Compiler Error, f needs to be public semantics
}
```

## Interface Inheritance

When we want a type to implement multiple interfaces, we can use `&` to separate multiple interfaces in the declaration, and there is no order requirement for the implemented interfaces.

For example, in the following example, we can have MyInt implement both Addable and Subtractable interfaces.

```javascript
interface Addable {
    func add(other: Int64): Int64
}

interface Subtractable {
    func sub(other: Int64): Int64
}

class MyInt <: Addable & Subtractable {
    var value = 0
    public func add(other: Int64): Int64 {
        value + other
    }
    public func sub(other: Int64): Int64 {
        value - other
    }
}
```

An interface can inherit one or more interfaces, but cannot inherit classes. At the same time, when an interface inherits, it can add new interface members.

For example, in the following example, the Calculable interface inherits both Addable and Subtractable interfaces and adds multiplication and division operator overloads.

```javascript
interface Addable {
    func add(other: Int64): Int64
}

interface Subtractable {
    func sub(other: Int64): Int64
}

interface Calculable <: Addable & Subtractable {
    func mul(other: Int64): Int64
    func div(other: Int64): Int64
}
```

This way, when implementing types implement the Calculable interface, they must implement all four operator overloads for addition, subtraction, multiplication, and division, and cannot miss any member.

```javascript
class MyInt <: Calculable {
    var value = 0
    public func add(other: Int64): Int64 {
        value + other
    }
    public func sub(other: Int64): Int64 {
        value - other
    }
    public func mul(other: Int64): Int64 {
        value * other
    }
    public func div(other: Int64): Int64 {
        value / other
    }
}
```

When MyInt implements Calculable, it also implements all interfaces that Calculable inherits, so MyInt also implements Addable and Subtractable, i.e., it is also their subtype.

```javascript
main() {
    let myInt = MyInt()
    let add: Addable = myInt
    let sub: Subtractable = myInt
    let calc: Calculable = myInt
}
```

For `interface` inheritance, if a sub-interface inherits functions or properties with default implementations from a parent interface, then in the sub-interface it is not allowed to only write declarations of these functions or properties (i.e., without default implementations), but must provide new default implementations, and the `override` modifier (or `redef` modifier) before the function definition is optional; if a sub-interface inherits functions or properties without default implementations from a parent interface, then in the sub-interface it is allowed to only write declarations of these functions or properties (of course, default implementations are also allowed), and the override modifier (or `redef` modifier) before the function declaration or definition is optional.

```javascript
interface I1 {
   func f(a: Int64) {
        a
   }
   static func g(a: Int64) {
        a
   }
   func f1(a: Int64): Unit
   static func g1(a: Int64): Unit
}

interface I2 <: I1 {
    /*'override' is optional*/ func f(a: Int64) {
       a + 1
    }
    override func f(a: Int32) {} // Error, override function 'f' does not have an overridden function from its supertypes
    static /*'redef' is optional*/ func g(a: Int64) {
       a + 1
    }
    /*'override' is optional*/ func f1(a: Int64): Unit {}
    static /*'redef' is optional*/ func g1(a: Int64): Unit {}
}
```

## Interface Implementation

All types in Cangjie can implement interfaces, including numeric types, Rune, String, struct, class, enum, Tuple, functions, and other types.

A type can implement an interface in three ways:

1. Declare to implement the interface when defining the type, as we have seen in the above content.
2. Implement the interface through extension, which is detailed in the [Extension](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/extension/interface_extension.html) chapter.
3. Built-in implementation by the language, see the "Cangjie Programming Language Library API" documentation for details.

When implementing types declare to implement an interface, they need to implement all members required by the interface, which requires satisfying the following rules.

1. For member functions and operator overload functions, the implementing type is required to provide function implementations with the same function name, parameter list, and return type as the corresponding functions in the interface.
2. For member properties, it is required that whether they are modified with `mut` remains consistent, and the property types are the same.

So in most cases, as in the above examples, we need to have the implementing type contain implementations of members that are the same as those required by the interface.

But there is one exception: if the return type of a member function or operator overload function in the interface is a class type, then the return type of the implementing function is allowed to be its subtype.

For example, in the following example, `f` in `I` has a return type that is a `class` type `Base`, so the `f` implemented in `C` can have a return type that is a subtype of `Base`, `Sub`.

```javascript
open class Base {}
class Sub <: Base {}

interface I {
    func f(): Base
}

class C <: I {
    public func f(): Sub {
        Sub()
    }
}
```

In addition, interface members can also provide default implementations for class types. Interface members with default implementations, when the implementing type is a class, the class can inherit the interface's implementation without providing its own implementation.

> **Note:**
>
> Default implementations are only valid for implementing types that are classes, and are invalid for other types.

For example, in the following code, `say` in `SayHi` has a default implementation, so `A` can inherit the implementation of `say` when implementing `SayHi`, while `B` can also choose to provide its own implementation of `say`.

```javascript
interface SayHi {
    func say() {
        "hi"
    }
}

class A <: SayHi {}

class B <: SayHi {
    public func say() {
        "hi, B"
    }
}
```

In particular, if a type implements multiple interfaces and multiple interfaces contain default implementations of the same member, this will cause a multiple inheritance conflict, and the language cannot choose the most appropriate implementation, so the default implementations in the interfaces will also become invalid, requiring the implementing type to provide its own implementation.

For example, in the following example, both `SayHi` and `SayHello` contain implementations of `say`, so `Foo` must provide its own implementation when implementing these two interfaces, otherwise a compilation error will occur.

```javascript
interface SayHi {
    func say() {
        "hi"
    }
}

interface SayHello {
    func say() {
        "hello"
    }
}

class Foo <: SayHi & SayHello {
    public func say() {
        "Foo"
    }
}
```

When struct, enum, and class implement interfaces, the `override` modifier (or `redef` modifier) before function or property definitions is optional, regardless of whether the functions or properties in the interface have default implementations.

```javascript
interface I {
    func foo(): Int64 {
        return 0
    }
}
enum E <: I{
    elem
    public override func foo(): Int64 {
        return 1
    }
}
struct S <: I {
    public override func foo(): Int64 {
        return 1
    }
}
```

## Any Type

The Any type is a built-in interface, defined as follows.

```javascript
interface Any {}
```

All interfaces in Cangjie inherit from it by default, and all non-interface types implement it by default, so all types can be used as subtypes of the Any type.

As in the following code, we can assign a series of variables of different types to variables of Any type.

```javascript
main() {
    var any: Any = 1
    any = 2.0
    any = "hello, world!"
}
```