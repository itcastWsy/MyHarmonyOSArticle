# 27-Struct Type Definition

The definition of a `struct` type begins with the keyword `struct`, followed by the name of the `struct`, and then the `struct` definition body enclosed in a pair of curly braces. The `struct` definition body can contain a series of member variables, member properties (see [Properties](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/class_and_interface/prop.html)), static initializers, constructors, and member functions.

```javascript
struct Rectangle {
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

The above example defines a `struct` type named `Rectangle`, which has two member variables `width` and `height` of type `Int64`, a constructor with two `Int64` type parameters (defined using the keyword `init`, with the function body typically initializing member variables), and a member function `area` (returning the product of `width` and `height`).

## struct Member Variables

`struct` member variables are divided into instance member variables and static member variables (modified with the `static` modifier and must have initial values). The difference in access is that instance member variables can only be accessed through `struct` instances (when we say `a` is an instance of type `T`, we mean `a` is a value of type `T`), while static member variables can only be accessed through the `struct` type name.

Instance member variables can be defined without setting initial values (but must be annotated with types, such as `width` and `height` in the above example), or they can be set with initial values, for example:

```javascript
struct Rectangle {
    let width = 10
    let height = 20
}
```

## struct Static Initializers

`struct` supports defining static initializers and initializing static member variables through assignment expressions in static initializers.

Static initializers begin with the keyword combination `static init`, followed by a parameter list with no parameters and a function body, and cannot be modified by access modifiers. The function body must complete the initialization of all uninitialized static member variables, otherwise a compilation error occurs.

```javascript
struct Rectangle {
    static let degree: Int64
    static init() {
        degree = 180
    }
}
```

At most one static initializer is allowed to be defined in a `struct`, otherwise a redefinition error is reported.

```javascript
struct Rectangle {
    static let degree: Int64
    static init() {
        degree = 180
    }
    static init() { // Error, redefinition with the previous static init function
        degree = 180
    }
}
```

## struct Constructors

`struct` supports two types of constructors: regular constructors and primary constructors.

Regular constructors begin with the keyword `init`, followed by a parameter list and function body. The function body must complete the initialization of all uninitialized instance member variables (if parameter names and member variable names cannot be distinguished, you can use `this` before the member variable to distinguish them, where `this` represents the current instance of the `struct`), otherwise a compilation error occurs.

```javascript
struct Rectangle {
    let width: Int64
    let height: Int64

    public init(width: Int64, height: Int64) { // Error, 'height' is not initialized in the constructor
        this.width = width
    }
}
```

Multiple regular constructors can be defined in a `struct`, but they must constitute overloading (see [Function Overloading](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/function/function_overloading.html)), otherwise a redefinition error is reported.

```javascript
struct Rectangle {
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

In addition to defining several regular constructors named `init`, a `struct` can also define (at most) one primary constructor. The name of the primary constructor is the same as the `struct` type name, and its parameter list can have two forms of formal parameters: regular formal parameters and member variable formal parameters (which need to be prefixed with `let` or `var` before the parameter name). Member variable formal parameters serve the dual function of defining member variables and constructor parameters.

Using a primary constructor can usually simplify the definition of a `struct`. For example, the above `Rectangle` containing an `init` constructor can be simplified to the following definition:

```javascript
struct Rectangle {
    public Rectangle(let width: Int64, let height: Int64) {}
}
```

The parameter list of the primary constructor can also define regular formal parameters, for example:

```javascript
struct Rectangle {
    public Rectangle(name: String, let width: Int64, let height: Int64) {}
}
```

If there are no custom constructors (including primary constructors) in the `struct` definition, and all instance member variables have initial values, a parameterless constructor will be automatically generated (calling this parameterless constructor will create an object where all instance member variables have values equal to their initial values); otherwise, this parameterless constructor will not be automatically generated.

For example, for the following `struct` definition, the automatically generated parameterless constructor is given in the comments:

```javascript
struct Rectangle {
    let width: Int64 = 10
    let height: Int64 = 10
    /* Auto-generated memberwise constructor:
    public init() {
    }
    */
}
```

## struct Member Functions

`struct` member functions are divided into instance member functions and static member functions (modified with the `static` modifier). The difference between them is: instance member functions can only be accessed through `struct` instances, static member functions can only be accessed through the `struct` type name; static member functions cannot access instance member variables or call instance member functions, but instance member functions can access static member variables and static member functions.

In the following example, `area` is an instance member function, and `typeName` is a static member function.

```javascript
struct Rectangle {
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

Instance member functions can access instance member variables through `this`, for example:

```javascript
struct Rectangle {
    let width: Int64 = 1
    let height: Int64 = 1

    public func area() {
        this.width * this.height
    }
}
```

## Access Modifiers for struct Members

`struct` members (including member variables, member properties, constructors, member functions, operator functions (see [Operator Overloading](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/function/operator_overloading.html) chapter)) are modified with 4 types of access modifiers: `private`, `internal`, `protected`, and `public`. The default modifier is `internal`.

- `private` means visible within the `struct` definition.
- `internal` means visible only within the current package and sub-packages (including sub-packages of sub-packages, see [Package](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/package/toplevel_access.html) chapter).
- `protected` means visible within the current module (see [Package](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/package/toplevel_access.html) chapter).
- `public` means visible both inside and outside the module.

In the following example, `width` is a `public` modified member that can be accessed outside the class, `height` is a member with the default access modifier, visible only in the current package and sub-packages, and cannot be accessed by other packages.

```javascript
package a
public struct Rectangle {
    public var width: Int64
    var height: Int64
    private var area: Int64
    ...
}

func samePkgFunc() {
    var r = Rectangle(10, 20)
    r.width = 8               // Ok: public 'width' can be accessed here
    r.height = 24             // Ok: 'height' has no modifier and can be accessed here
    r.area = 30               // Error, private 'area' can't be accessed here
}
package b
import a.*
main() {
    var r = Rectangle(10, 20)
    r.width = 8               // Ok: public 'width' can be accessed here
    r.height = 24             // Error, no modifier 'height' can't be accessed here
    r.area = 30               // Error, private 'area' can't be accessed here
}
```

## Prohibition of Recursive struct

Both recursive and mutually recursive defined `struct` are illegal. For example:

```javascript
struct R1 { // Error, 'R1' recursively references itself
    let other: R1
}
struct R2 { // Error, 'R2' and 'R3' are mutually recursive
    let other: R3
}
struct R3 { // Error, 'R2' and 'R3' are mutually recursive
    let other: R2
}
```