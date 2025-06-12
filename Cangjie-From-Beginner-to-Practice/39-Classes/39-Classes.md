# 39-Classes

The `class` type is a classic concept in object-oriented programming, and Cangjie also supports using `class` to implement object-oriented programming. The main differences between `class` and `struct` are: `class` is a reference type while `struct` is a value type, and they behave differently during assignment or parameter passing; classes can inherit from each other, but structs cannot.

This section introduces how to define `class` types, how to create objects, and class inheritance.

## Class Definition

The definition of a `class` type starts with the keyword `class`, followed by the name of the `class`, and then the class definition body enclosed in a pair of braces. The class definition body can define a series of member variables, member properties (see [Properties](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/class_and_interface/prop.html)), static initializers, constructors, member functions, and operator functions (see [Operator Overloading](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/function/operator_overloading.html) chapter).

```javascript
class Rectangle {
    let width: Int64
    let height: Int64

    public init(width: Int64, height: Int64) {
        this.width = width
        this.height = height
    }

    public func area() {
        width * height
    }
}
```

In the above example, a `class` type named `Rectangle` is defined, which has two `Int64` type member variables `width` and `height`, a constructor with two `Int64` type parameters, and a member function `area` (which returns the product of `width` and `height`).

> **Note:**
>
> `class` can only be defined at the top level of source files.

### Class Member Variables

Class member variables are divided into instance member variables and static member variables. Static member variables are modified with the `static` modifier, must have initial values, and can only be accessed through the type name. See the following example:

```javascript
class Rectangle {
    let width = 10
    static let height = 20
}

let l = Rectangle.height // l = 20
```

Instance member variables can be defined without setting initial values (but must be annotated with types) or with initial values, and can only be accessed through objects (i.e., instances of the class). See the following example:

```javascript
class Rectangle {
    let width = 10
    let height: Int64
    init(h: Int64){
        height = h
    }
}
let rec = Rectangle(20)
let l = rec.height // l = 20
```

### Class Static Initializers

Classes support defining static initializers, and static member variables can be initialized through assignment expressions in static initializers.

Static initializers start with the keyword combination `static init`, followed by a parameterless parameter list and function body, and cannot be modified by access modifiers. The function body must complete the initialization of all uninitialized static member variables, otherwise a compilation error will occur.

```javascript
class Rectangle {
    static let degree: Int64
    static init() {
        degree = 180
    }
}
```

At most one static initializer is allowed to be defined in a class, otherwise a redefinition error will be reported.

```javascript
class Rectangle {
    static let degree: Int64
    static init() {
        degree = 180
    }
    static init() { // Error, redefinition with the previous static init function
        degree = 180
    }
}
```

### Class Constructors

Like `struct`, classes also support defining ordinary constructors and primary constructors.

Ordinary constructors start with the keyword `init`, followed by a parameter list and function body. The function body must complete the initialization of all uninitialized instance member variables, otherwise a compilation error will occur.

```javascript
class Rectangle {
    let width: Int64
    let height: Int64

    public init(width: Int64, height: Int64) { // Error, 'height' is not initialized in the constructor
        this.width = width
    }
}
```

Multiple ordinary constructors can be defined in a class, but they must constitute overloading (see [Function Overloading](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/function/function_overloading.html)), otherwise a redefinition error will be reported.

```javascript
class Rectangle {
    let width: Int64
    let height: Int64

    public init(width: Int64) {
        this.width = width
        this.height = width
    }

    public init(width: Int64, height: Int64) { // Ok: overloading with the first init function
        this.width = width
        this.height = height
    }

    public init(height: Int64) { // Error, redefinition with the first init function
        this.width = height
        this.height = height
    }
}
```

In addition to defining several ordinary constructors named `init`, classes can also define (at most) one primary constructor. The primary constructor has the same name as the `class` type name, and its parameter list can have two forms of formal parameters: ordinary formal parameters and member variable formal parameters (which need to be prefixed with `let` or `var` before the parameter name). Member variable formal parameters have both the function of defining member variables and constructor parameters.

Using a primary constructor can usually simplify the definition of a `class`. For example, the above `Rectangle` containing an `init` constructor can be simplified to the following definition:

```javascript
class Rectangle {
    public Rectangle(let width: Int64, let height: Int64) {}
}
```

The parameter list of the primary constructor can also define ordinary formal parameters, for example:

```javascript
class Rectangle {
    public Rectangle(name: String, let width: Int64, let height: Int64) {}
}
```

When creating an instance of a class, the constructor called will execute expressions in the class in the following order:

1. First initialize variables with default values defined outside the primary constructor;
2. If the constructor body does not explicitly call a parent class constructor or another constructor of this class, call the parent class's parameterless constructor `super()`. If the parent class does not have a parameterless constructor, an error will be reported;
3. Execute the code in the constructor body.

```javascript
func foo(x: Int64): Int64 {
    println("I'm foo, got ${x}")
    x
}

open class A {
    init() {
        println("I'm A")
    }
}

class B <: A {
    var x = foo(0)
    init() {
        x = foo(1)
        println("init B finished")
    }
}

main() {
    B()
    0
}
```

In the above example, when calling the constructor of `B`, first initialize the variable `x` with a default value, at which time `foo(0)` is called; then call the parent class's parameterless constructor, at which time the constructor of `A` is called; next execute the code in the constructor body, at which time `foo(1)` is called and a string is printed. Therefore, the output of the above example is:

```text
I'm foo, got 0
I'm A
I'm foo, got 1
init B finished
```

If there are no custom constructors (including primary constructors) in the `class` definition, and all instance member variables have initial values, a parameterless constructor will be automatically generated (calling this parameterless constructor will create an object where all instance member variables have values equal to their initial values); otherwise, this parameterless constructor will not be automatically generated. For example, for the following `class` definition, the compiler will automatically generate a parameterless constructor:

```javascript
class Rectangle {
    let width = 10
    let height = 20

    /* Auto-generated parameterless constructor:
    public init() {

    }
    */
}

// Invoke the auto-generated parameterless constructor
let r = Rectangle() // r.width = 10，r.height = 20
```

### Class Finalizers

Classes support defining finalizers, which are functions called when instances of the class are garbage collected. The function name of a finalizer is fixed as `~init`. Finalizers are generally used to release system resources:

```javascript
class C {
    var p: CString

    init(s: String) {
        p = unsafe { LibC.mallocCString(s) }
        println(s)
    }
    ~init() {
        unsafe { LibC.free(p) }
    }
}
```

There are some restrictions on using finalizers that developers need to pay attention to:

1. Finalizers have no parameters, no return type, no generic type parameters, no modifiers, and cannot be explicitly called.
2. Classes with finalizers cannot be modified with `open`; only non-`open` classes can have finalizers.
3. A class can define at most one finalizer.
4. Finalizers cannot be defined in extensions.
5. The timing of when finalizers are triggered is uncertain.
6. Finalizers may execute on any thread.
7. The execution order of multiple finalizers is uncertain.
8. Throwing uncaught exceptions from finalizers is undefined behavior.
9. Creating threads or using thread synchronization features in finalizers is undefined behavior.
10. If an object can still be accessed after the finalizer execution ends, this is undefined behavior.
11. If an object throws an exception during initialization, the finalizer of such incompletely initialized objects will not execute.

### Class Member Functions

Class member functions are also divided into instance member functions and static member functions (modified with the `static` modifier). Instance member functions can only be accessed through objects, and static member functions can only be accessed through the `class` type name. Static member functions cannot access instance member variables or call instance member functions, but instance member functions can access static member variables and static member functions.

In the following example, `area` is an instance member function, and `typeName` is a static member function.

```javascript
class Rectangle {
    let width: Int64 = 10
    let height: Int64 = 20

    public func area() {
        this.width * this.height
    }

    public static func typeName(): String {
        "Rectangle"
    }
}
```

Based on whether they have function bodies, instance member functions can be divided into abstract member functions and non-abstract member functions. Abstract member functions have no function body and can only be defined in abstract classes or interfaces (see [Interface](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/class_and_interface/interface.html) chapter). For example, in the following example, the abstract function `foo` is defined in the abstract class `AbRectangle` (modified with the keyword `abstract`).

```javascript
abstract class AbRectangle {
    public func foo(): Unit
}
```

Note that abstract instance member functions have `open` semantics by default, the `open` modifier is optional, and must be modified with `public` or `protected`.

Non-abstract functions must have function bodies, and instance member variables can be accessed through `this` in the function body, for example:

```javascript
class Rectangle {
    let width: Int64 = 10
    let height: Int64 = 20

    public func area() {
        this.width * this.height
    }
}
```

### Access Modifiers for Class Members

For class members (including member variables, member properties, constructors, member functions), there are 4 access modifiers that can be used: `private`, `internal`, `protected`, and `public`. The default meaning is `internal`.

- `private` means visible within the `class` definition.
- `internal` means visible only within the current package and sub-packages (including sub-packages of sub-packages, see [Package](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/package/toplevel_access.html) chapter).
- `protected` means visible within the current module (see [Package](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/package/toplevel_access.html) chapter) and subclasses of the current class.
- `public` means visible both inside and outside the module.

```javascript
package a
public open class Rectangle {
    public var width: Int64
    protected var height: Int64
    private var area: Int64
    public init(width: Int64, height: Int64) {
        this.width = width
        this.height = height
        this.area = this.width * this.height
    }
    init(width: Int64, height: Int64, multiple: Int64) {
        this.width = width
        this.height = height
        this.area = width * height * multiple
    }
}

func samePkgFunc() {
    var r = Rectangle(10, 20) // Ok: constructor 'Rectangle' can be accessed here
    r.width = 8               // Ok: public 'width' can be accessed here
    r.height = 24             // Ok: protected 'height' can be accessed here
    r.area = 30               // Error, private 'area' cannot be accessed here
}
package b
import a.*
public class Cuboid <: Rectangle {
    private var length: Int64
    public init(width: Int64, height: Int64, length: Int64) {
        super(width, height)
        this.length = length
    }
    public func volume() {
        this.width * this.height * this.length // Ok: protected 'height' can be accessed here
    }
}

main() {
    var r = Rectangle(10, 20, 2) // Error, Rectangle has no `public` constructor with three parameters
    var c = Cuboid(20, 20, 20)
    c.width = 8               // Ok: public 'width' can be accessed here
    c.height = 24             // Error, protected 'height' cannot be accessed here
    c.area = 30               // Error, private 'area' cannot be accessed here
}
```

## This Type

Inside a class, we support the `This` type placeholder, which refers to the type of the current class. It can only be used as the return type of instance member functions. When a subclass object calls a function defined in the parent class that returns the `This` type, the type of that function call will be recognized as the subclass type, not the parent class type where it is defined.

If an instance member function does not declare a return type and only has return expressions of `This` type, the return type of the current function will be inferred as `This`. Example:

```javascript
open class C1 {
    func f(): This {  // its type is `() -> C1`
        return this
    }

    func f2() { // its type is `() -> C1`
        return this
    }

    public open func f3(): C1 {
        return this
    }
}
class C2 <: C1 {
    // member function f is inherited from C1, and its type is `() -> C2` now
    public override func f3(): This { // Ok
        return this
    }
}

var obj1: C2 = C2()
var obj2: C1 = C2()

var x = obj1.f()    // During compilation, the type of x is C2
var y = obj2.f()    // During compilation, the type of y is C1
```

## Creating Objects

After defining a `class` type, you can create objects by calling its constructors (calling constructors through the `class` type name). For example, in the following example, a `Rectangle` type object is created through `Rectangle(10, 20)` and assigned to variable `r`.

```javascript
let r = Rectangle(10, 20)
```

After creating an object, you can access (publicly modified) instance member variables and instance member functions through the object. For example, in the following example, the values of `width` and `height` in `r` can be accessed through `r.width` and `r.height` respectively, and the member function `area` can be called through `r.area()`.

```javascript
let r = Rectangle(10, 20) // r.width = 10, r.height = 20
let width = r.width       // width = 10
let height = r.height     // height = 20
let a = r.area()          // a = 200
```

If you want to modify the value of member variables through objects (this approach is not encouraged; it's better to modify through member functions), you need to define the member variables in the `class` type as mutable member variables (i.e., defined using `var`). Example:

```javascript
class Rectangle {
    public var width: Int64
    public var height: Int64

    ...
}

main() {
    let r = Rectangle(10, 20) // r.width = 10, r.height = 20
    r.width = 8               // r.width = 8
    r.height = 24             // r.height = 24
    let a = r.area()          // a = 192
}
```

Unlike `struct`, objects are not copied during assignment or parameter passing. Multiple variables point to the same object. Modifying the value of members in an object through one variable will also modify the corresponding member variables in other variables. Taking assignment as an example, in the following example, after assigning `r1` to `r2`, modifying the values of `width` and `height` of `r1` will also modify the values of `width` and `height` of `r2`.

```javascript
main() {
    var r1 = Rectangle(10, 20) // r1.width = 10, r1.height = 20
    var r2 = r1                // r2.width = 10, r2.height = 20
    r1.width = 8               // r1.width = 8
    r1.height = 24             // r1.height = 24
    let a1 = r1.area()         // a1 = 192
    let a2 = r2.area()         // a2 = 192
}
```

## Class Inheritance

Like most programming languages that support `class`, classes in Cangjie also support inheritance. If class B inherits from class A, we call A the parent class and B the subclass. Subclasses will inherit all members from the parent class except `private` members and constructors.

Abstract classes can always be inherited, so the `open` modifier is optional when defining abstract classes. You can also use the `sealed` modifier to modify abstract classes, indicating that the abstract class can only be inherited within this package. But non-abstract classes can be inherited conditionally: they must be modified with the `open` modifier when defined. When an `open` modified instance member is inherited by a class, the `open` modifier is also inherited. When there are `open` modified members in a non-`open` modified class, the compiler will give a warning.

You can specify the parent class that a subclass inherits from using `<:` in the subclass definition, but the parent class must be inheritable. For example, in the following example, class A is modified with `open` and can be inherited by class B, but because class B is not inheritable, C will report an error when inheriting B.

```javascript
open class A {
    let a: Int64 = 10
}

class B <: A { // Ok: 'B' Inheritance 'A'
    let b: Int64 = 20
}

class C <: B { // Error, 'B' is not inheritable
    let c: Int64 = 30
}
```

Classes only support single inheritance, so the following code where a class inherits from two classes is illegal (`&` is the syntax for a class implementing multiple interfaces, see [Interface](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/class_and_interface/interface.html) chapter).

```javascript
open class A {
    let a: Int64 = 10
}

open class B {
    let b: Int64 = 20
}

class C <: A & B { // Error, 'C' can only inherit one class
    let c: Int64 = 30
}
```

Because classes have single inheritance, any class can have at most one direct parent class. For classes that specify a parent class when defined, their direct parent class is the class specified when defined. For classes that do not specify a parent class when defined, their direct parent class is the `Object` type. `Object` is the parent class of all classes (note that `Object` has no direct parent class and contains no members).

Because subclasses inherit from parent classes, subclass objects can naturally be used as parent class objects, but not vice versa. For example, in the following example, B is a subclass of A, so B type objects can be assigned to A type variables, but A type objects cannot be assigned to B type variables.

```javascript
open class A {
    let a: Int64 = 10
}

class B <: A {
    let b: Int64 = 20
}

let a: A = B() // Ok: subclass objects can be assigned to superclass variables
open class A {
    let a: Int64 = 10
}

class B <: A {
    let b: Int64 = 20
}

let b: B = A() // Error, superclass objects can not be assigned to subclass variables
```

Class-defined types are not allowed to inherit from the type itself.

```javascript
class A <: A {}  // Error, 'A' inherits itself.
```

The `sealed` modifier can only modify abstract classes, indicating that the modified class definition can only be inherited by other classes within the package where this definition is located. `sealed` already implies the semantics of `public`/`open`, so when defining a sealed abstract class, if `public`/`open` modifiers are provided, the compiler will warn. Subclasses of `sealed` classes may not be `sealed` classes and can still be modified with `open`/`sealed` or use no inheritance modifiers. If a subclass of a `sealed` class is modified with `open`, its subclasses can be inherited outside the package. Subclasses of `sealed` classes may not be modified with `public`.

```javascript
package A
public sealed abstract class C1 {}   // Warning, redundant modifier, 'sealed' implies 'public'
sealed open abstract class C2 {}     // Warning, redundant modifier, 'sealed' implies 'open'
sealed abstract class C3 {}          // OK, 'public' is optional when 'sealed' is used

class S1 <: C1 {}  // OK
public open class S2 <: C1 {}   // OK
public sealed abstract class S3 <: C1 {}  // OK
open class S4 <: C1 {}   // OK

package B
import A.*

class SS1 <: S2 {}  // OK
class SS2 <: S3 {}  // Error, S3 is sealed class, cannot be inherited here.
sealed class SS3 {} // Error, 'sealed' cannot be used on non-abstract class.
```

### Parent Class Constructor Calls

Subclass `init` constructors can call parent class constructors using the form `super(args)`, or call other constructors of this class using the form `this(args)`, but only one of the two can be called. If called, it must be at the first expression in the constructor body, and there cannot be any expressions or declarations before it.

```javascript
open class A {
    A(let a: Int64) {}
}

class B <: A {
    let b: Int64
    init(b: Int64) {
        super(30)
        this.b = b
    }

    init() {
        this(20)
    }
}
```

In subclass primary constructors, you can call parent class constructors using the form `super(args)`, but you cannot call other constructors of this class using the form `this(args)`.

If a subclass constructor does not explicitly call a parent class constructor or explicitly call other constructors, the compiler will insert a call to the direct parent class's parameterless constructor at the beginning of that constructor body. If the parent class does not have a parameterless constructor at this time, a compilation error will occur.

```javascript
open class A {
    let a: Int64
    init() {
        a = 100
    }
}

open class B <: A {
    let b: Int64
    init(b: Int64) {
        // OK, `super()` added by compiler
        this.b = b
    }
}

open class C <: B {
    let c: Int64
    init(c: Int64) {  // Error, there is no non-parameter constructor in super class
        this.c = c
    }
}
```

### Override and Redefinition

Subclasses can override non-abstract instance member functions with the same name in the parent class, that is, define new implementations for certain instance member functions in the parent class within the subclass. When overriding, the member function in the parent class must be modified with `open`, and the function with the same name in the subclass must be modified with `override`, where `override` is optional. For example, in the following example, function `f` in subclass B overrides function `f` in parent class A.

```javascript
open class A {
    public open func f(): Unit {
        println("I am superclass")
    }
}

class B <: A {
    public override func f(): Unit {
        println("I am subclass")
    }
}

main() {
    let a: A = A()
    let b: A = B()
    a.f()
    b.f()
}
```

For overridden functions, the version to call will be determined based on the runtime type of the variable (determined by the object actually assigned to that variable) when called (so-called dynamic dispatch). For example, in the above example, the runtime type of `a` is A, so `a.f()` calls function `f` in parent class A; the runtime type of `b` is B (compile-time type is A), so `b.f()` calls function `f` in subclass B. So the program will output:

```text
I am superclass
I am subclass
```

For static functions, subclasses can redefine non-abstract static functions with the same name in the parent class, that is, define new implementations for certain static functions in the parent class within the subclass. When redefining, the static function with the same name in the subclass must be modified with `redef`, where `redef` is optional. For example, in the following example, function `foo` in subclass D redefines function `foo` in parent class C.

```javascript
open class C {
    public static func foo(): Unit {
        println("I am class C")
    }
}

class D <: C {
    public redef static func foo(): Unit {
        println("I am class D")
    }
}

main() {
    C.foo()
    D.foo()
}
```

For redefined functions, the version to call will be determined based on the type of the `class` when called. For example, in the above example, `C.foo()` calls function `foo` in parent class C, and `D.foo()` calls function `foo` in subclass D.

```text
I am class C
I am class D
```

If abstract functions or functions modified with `open` have named formal parameters, then implementation functions or functions modified with `override` also need to maintain the same named formal parameters.

```javascript
open class A {
    public open func f(a!: Int32): Int32 {
        a + 1
    }
}

class B <: A {
    public override func f(a!: Int32): Int32 { // Ok
        a + 2
    }
}

class C <: A {
    public override func f(b!: Int32): Int32 { // Error
        b + 3
    }
}

main() {
    B().f(a: 0)
    C().f(b: 0)
}
```

It should also be noted that when the implemented or redefined function is a generic function, the type variable constraints of the subtype function need to be looser than or the same as the corresponding function in the parent type.

```javascript
open class A {}
open class B <: A {}
open class C <: B {}

open class Base {
    static func f<T>(a: T): Unit where T <: B {}
    static func g<T>(): Unit where T <: B {}
}

class D <: Base {
    redef static func f<T>(a: T): Unit where T <: C {} // Error, stricter constraint
    redef static func g<T>(): Unit where T <: C {} // Error, stricter constraint
}

class E <: Base {
    redef static func f<T>(a: T): Unit where T <: A {} // OK: looser constraint
    redef static func g<T>(): Unit where T <: A {} // OK: looser constraint
}

class F <: Base {
    redef static func f<T>(a: T): Unit where T <: B {} // OK: same constraint
    redef static func g<T>(): Unit where T <: B {} // OK: same constraint
}
```