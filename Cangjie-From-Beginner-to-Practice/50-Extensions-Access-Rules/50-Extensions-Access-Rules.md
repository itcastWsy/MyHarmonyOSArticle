# 50-Extensions - Access Rules

## Extension Modifiers

Extensions themselves cannot be modified with modifiers.

For example, in the following example, using the `public` modifier before the direct extension of A will result in a compilation error.

```typescript
public class A {}

public extend A {}  // Error, expected no modifier before extend
```

The modifiers that can be used for extension members are: `static`, `public`, `protected`, `internal`, `private`, `mut`.

- Members modified with `private` can only be used within this extension and are not visible externally.
- Members modified with `internal` can be used within the current package and sub-packages (including sub-packages of sub-packages), which is the default behavior.
- Members modified with `protected` can be accessed within this module (subject to export rules). When the extended type is a class, it can also be accessed within the definition body of subclasses of that class.
- Members modified with `static` can only be accessed through the type name, not through instance objects.
- Extensions for `struct` types can define `mut` functions.

```typescript
package p1

public open class A {}

extend A {
    public func f1() {}
    protected func f2() {}
    private func f3() {}
    static func f4() {}
}

main() {
    A.f4()
    var a = A()
    a.f1()
    a.f2()
}
```

Member definitions within extensions do not support modification with `open`, `override`, or `redef`.

```typescript
class Foo {
    public open func f() {}
    static func h() {}
}

extend Foo {
    public override func f() {} // Error
    public open func g() {} // Error
    redef static func h() {} // Error
}
```

## Extension Orphan Rules

Implementing an interface from another `package` for a type from another `package` may cause confusion in understanding.

To prevent a type from accidentally implementing inappropriate interfaces, Cangjie does not allow defining orphan extensions, which refers to interface extensions that are neither defined in the same package as the interface (including all interfaces in the interface inheritance chain) nor in the same package as the extended type.

As shown in the following code, you cannot implement `Bar` from `package b` for `Foo` from `package a` in `package c`.

You can only implement `Bar` for `Foo` in `package a` or `package b`.

```typescript
// package a
public class Foo {}

// package b
public interface Bar {}

// package c
import a.Foo
import b.Bar

extend Foo <: Bar {} // Error
```

## Extension Access and Shadowing

Instance members of extensions can use `this` just like at the type definition site, and the functionality of `this` remains consistent. You can also omit `this` to access members. Instance members of extensions cannot use `super`.

```typescript
class A {
    var v = 0
}

extend A {
    func f() {
        print(this.v) // Ok
        print(v) // Ok
    }
}
```

Extensions cannot access members modified with `private` in the extended type.

```typescript
class A {
    private var v1 = 0
    protected var v2 = 0
}

extend A {
    func f() {
        print(v1) // Error
        print(v2) // Ok
    }
}
```

Extensions cannot shadow any members of the extended type.

```typescript
class A {
    func f() {}
}

extend A {
    func f() {} // Error
}
```

Extensions also do not allow shadowing any members added by other extensions.

```typescript
class A {}

extend A {
    func f() {}
}

extend A {
    func f() {} // Error
}
```

Within the same package, the same type can be extended multiple times, and in extensions, you can directly call functions from other extensions of the extended type that are not modified with `private`.

```typescript
class Foo {}

extend Foo { // OK
    private func f() {}
    func g() {}
}

extend Foo { // OK
    func h() {
        g() // OK
        f() // Error
    }
}
```

When extending generic types, additional generic constraints can be used. The visibility rules between any two extensions of a generic type are as follows:

- If the constraints of two extensions are the same, then the two extensions are mutually visible, meaning functions or properties from each other can be used directly within the two extensions;
- If the constraints of two extensions are different and there is a containment relationship between the two extension constraints, the extension with looser constraints is visible to the extension with stricter constraints, but not vice versa;
- When the constraints of two extensions are different and there is no containment relationship between the two constraints, then the two extensions are mutually invisible.

Example: Assume two extensions of the same type `E<X>` are extension `1` and extension `2`, where the constraint of `X` in extension `1` is stricter than in extension `2`. Then functions and properties in extension `1` are not visible to extension `2`, but conversely, functions and properties in extension `2` are visible to extension `1`.

```typescript
open class A {}
class B <: A {}
class E<X> {}

interface I1 {
    func f1(): Unit
}
interface I2 {
    func f2(): Unit
}

extend<X> E<X> <: I1 where X <: B {  // extension 1
    public func f1(): Unit {
        f2() // OK
    }
}

extend<X> E<X> <: I2 where X <: A   { // extension 2
    public func f2(): Unit {
        f1() // Error
    }
}
```

## Extension Import and Export

Extensions can also be imported and exported, but extensions themselves cannot be modified with `public`. Extension export has a special set of rules.

For direct extensions, the extension functionality will only be exported when the extension is in the same package as the extended type, and both the extended type and the members added in the extension are modified with `public` or `protected`.

All other direct extensions cannot be exported and can only be used in the current package.

As shown in the following code, `Foo` is a type modified with `public`, and `f` is in the same package as `Foo`, so `f` will be exported together with `Foo`. However, `g` and `Foo` are not in the same package, so `g` will not be exported.

```typescript
// package a

public class Foo {}

extend Foo {
    public func f() {}
}

// package b
import a.*

extend Foo {
    public func g() {}
}

// package c
import a.*
import b.*

main() {
    let a = Foo()
    a.f() // OK
    a.g() // Error
}
```

For interface extensions, there are two situations:

1. If the interface extension and the extended type are in the same package, but the interface is imported, the extension functionality will only be exported when the extended type is modified with `public`.
2. If the interface extension is in the same package as the interface, the extension functionality will only be exported when the interface is modified with `public`.

As shown in the following code, both `Foo` and `I` are modified with `public`, so the extension of `Foo` can be exported.

```typescript
// package a

public class Foo {}

public interface I {
    func g(): Unit
}

extend Foo <: I {
    public func g(): Unit {}
}

// package b
import a.*

main() {
    let a: I = Foo()
    a.g()
}
```

Similar to extension export, extension import does not require explicit `import` statements. Extension import only requires importing the extended type and interface to import all accessible extensions.

As shown in the following code, in `package b`, you only need to import `Foo` to use the function `f` from the corresponding extension of `Foo`.

For interface extensions, you need to import both the extended type and the extension interface to use them. Therefore, in `package c`, you need to import both `Foo` and `I` to use the function `g` from the corresponding extension.

```typescript
// package a
public class Foo {}
extend Foo {
    public func f() {}
}

// package b
import a.Foo

public interface I {
    func g(): Unit
}
extend Foo <: I {
    public func g() {
        this.f() // OK
    }
}

// package c
import a.Foo
import b.I

func test() {
    let a = Foo()
    a.f() // OK
    a.g() // OK
}
```