# 49-Extensions (1)

## Extension Overview

Extensions can add new functionality to types (except functions, tuples, interfaces) that are visible in the current `package`.

Extensions can be used when you cannot break the encapsulation of the extended type but want to add additional functionality.

Functionality that can be added includes:

- **Adding member functions**
- **Adding operator overload functions**
- **Adding member properties**
- **Implementing interfaces**

Although extensions can add additional functionality, they cannot change the encapsulation of the extended type, so extensions do not support the following features:

1. Extensions cannot add member variables.
2. Functions and properties in extensions must have implementations.
3. Functions and properties in extensions cannot be modified with `open`, `override`, or `redef`.
4. Extensions cannot access members modified with `private` in the extended type.

Based on whether extensions implement new interfaces, extensions can be divided into two types: **direct extensions** and **interface extensions**. Direct extensions are extensions that do not include additional interfaces; interface extensions are extensions that include interfaces, which can be used to add new functionality to existing types and implement interfaces, enhancing abstraction flexibility.

## Direct Extensions

Directly extending a type is called direct extension.

## Simple Extensions

Here's a demonstration of a relatively simple usage of extending types. Adding a printSize method to the String type.

Note that the String type itself has a size property, so `this.size` will not cause an error.

```typescript
extend String {
    public func printSize() {
        println("the size is ${this.size}")
    }
}

main() {
    let a = "123"
    a.printSize() // the size is 3
}
```

## Generic Extensions

When the extended type is generic, two ways are provided to extend generic types.

1. Extending specific generic instantiated types
2. Generic extensions that introduce generic type parameters after `extend`

The main difference between the two is in the syntax.

### Extending Specific Generic Instantiated Types

For extending specific generic instantiated types, the keyword `extend` allows a fully instantiated generic type. The functionality added to these types can only be used when the types match completely, and the type arguments of the generic type must comply with the constraint requirements at the generic type definition.

As shown in the following code, A is the parent class of B. B must contain all implementations of A, so the where constraint takes effect.

```typescript
open class A {
    func a() {
        return true
    }
}

class B <: A {
    func b() {
        return true
    }
}

class Foo<T> where T <: A {}

extend Foo<B> {} 

main() {
    var a = Foo<B>()
}
```

### Generic Extensions with Type Parameters After extend

Generic extensions that introduce generic type parameters after `extend`. Generic extensions can be used to extend uninstantiated or incompletely instantiated generic types. The generic type parameters declared after `extend` must be used directly or indirectly on the extended generic type. The functionality added to these types can only be used when the types and constraints match completely.

```typescript
class MyList<T> {
    public let data: Array<T> = Array<T>()
}

extend<T> MyList<T> {} // OK
extend<R> MyList<R> {} // OK
extend<T, R> MyList<(T, R)> {} // OK
```

For example, you can define a type called Pair that can conveniently store two elements (similar to Tuple).

We want the Pair type to be able to accommodate any type, so the two generic variables should not have any constraints, ensuring that Pair can accommodate all types.

But at the same time, we want Pair to also support equality comparison when the two elements can be compared for equality. This functionality can be implemented using extensions.

As shown in the code below, using extension syntax, we constrain T1 and T2 so that when they support equals, Pair can also implement the equals function.

```typescript
class Pair<T1, T2> {
    var first: T1
    var second: T2
    public init(a: T1, b: T2) {
        first = a
        second = b
    }
}

interface Eq<T> {
    func equals(other: T): Bool
}

extend<T1, T2> Pair<T1, T2> where T1 <: Eq<T1>, T2 <: Eq<T2> {
    public func equals(other: Pair<T1, T2>) {
        first.equals(other.first) && second.equals(other.second)
    }
}

class Foo <: Eq<Foo> {
    public func equals(other: Foo): Bool {
        true
    }
}

main() {
    let a = Pair(Foo(), Foo())
    let b = Pair(Foo(), Foo())
    println(a.equals(b)) // true
}
```

## Interface Extensions

Extending generics while implementing generics is called interface extension.

```typescript
interface PrintSizeable {
    func printSize(): Unit
}

extend<T> Array<T> <: PrintSizeable {
    public func printSize() {
        println("The size is ${this.size}")
    }
}
```

After using extensions to implement `PrintSizeable` for `Array`, it's equivalent to implementing the interface `PrintSizeable` when defining `Array`.

Therefore, `Array` can be used as an implementation type of `PrintSizeable`, as shown in the following code.

```typescript
main() {
    let a: PrintSizeable = Array<Int64>()
    a.printSize() // 0
}
```

You can implement multiple interfaces simultaneously in the same extension, with multiple interfaces separated by `&`. The order of interfaces doesn't matter.

As shown in the code below, you can implement `I1`, `I2`, and `I3` for `Foo` simultaneously in an extension.

```typescript
interface I1 {
    func f1(): Unit
}

interface I2 {
    func f2(): Unit
}

interface I3 {
    func f3(): Unit
}

class Foo {}

extend Foo <: I1 & I2 & I3 {
    public func f1(): Unit {}
    public func f2(): Unit {}
    public func f3(): Unit {}
}
```

You can also declare additional generic constraints in interface extensions to implement interfaces that can only be satisfied under specific constraints.

For example, you can make the above `Pair` type implement the `Eq` interface, so that `Pair` itself can become a type that conforms to the `Eq` constraint, as shown in the following code.

```typescript
class Pair<T1, T2> {
    var first: T1
    var second: T2
    public init(a: T1, b: T2) {
        first = a
        second = b
    }
}

interface Eq<T> {
    func equals(other: T): Bool
}

extend<T1, T2> Pair<T1, T2> <: Eq<Pair<T1, T2>> where T1 <: Eq<T1>, T2 <: Eq<T2> {
    public func equals(other: Pair<T1, T2>) {
        first.equals(other.first) && second.equals(other.second)
    }
}

class Foo <: Eq<Foo> {
    public func equals(other: Foo): Bool {
        true
    }
}

main() {
    let a = Pair(Foo(), Foo())
    let b = Pair(Foo(), Foo())
    println(a.equals(b)) // true
}
```