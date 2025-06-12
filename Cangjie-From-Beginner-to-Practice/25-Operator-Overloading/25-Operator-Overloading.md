# 25-Operator Overloading

## Introduction

Operator overloading means allowing developers to assign different capabilities to common operators according to their needs. For example:

1. Assign **+** the capability of **\***
2. Assign **-** the capability of **squaring**

## Ways to Define Operator Functions

There are two ways to define operator functions:

1. For types that can directly contain function definitions (including `struct`, `enum`, `class`, and `interface`), you can directly define operator functions within them to implement operator overloading.
2. Use the `extend` method to add operator functions to them, thereby implementing operator overloading on these types.

The conventions for operator function parameters are as follows:

1. Unary operators
2. Binary operators
3. Index operator (`[]`)
4. Operator functions have only one parameter, with no requirements on the return value type.

### Unary Operators and Binary Operators

`-` implements taking the negative value of two member variables `x` and `y` in a `Point` instance, then returns a new `Point` object. `+` implements summing the two member variables `x` and `y` of two `Point` instances respectively, then returns a new `Point` object.

```javascript
open class Point {
    var x: Int64 = 0
    var y: Int64 = 0
    public init (a: Int64, b: Int64) {
        x = a
        y = b
    }

   // Overload unary operator -
    public operator func -(): Point {
        Point(-x, -y)
    }
   // Overload binary operator +
    public operator func +(right: Point): Point {
        Point(this.x + right.x, this.y + right.y)
    }
}
```

Directly use the unary `-` operator and binary `+` operator:

```javascript
main() {
    let p1 = Point(8, 24)
    let p2 = -p1      // p2 = Point(-8, -24)
    let p3 = p1 + p2  // p3 = Point(0, 0)
}
```

### Index Operator

The index operator refers to `[]`, which is divided into two forms: one for getting values, and one for setting values. For example:

```javascript
let a = arr[1]; // Getting value
arr[2] = 300; // Setting value
```

So the overloading syntax also changes accordingly.

Overloading for getting values:

```javascript
class A {
    operator func [](arg1: Int64, arg2: String): Int64 {
        return 0
    }
}

func f() {
    let a = A()
    let b: Int64 = a[1, "2"] //  Overloading for getting values
    // b == 0
}
```

Overloading for setting values must add a fixed-name parameter `value` at the end, and the overloaded function's return value must be `unit` type:

```javascript
class A {
    var num: Int64 = 100
    operator func [](arg1: Int64, arg2: Int64, value!: Int64): Unit {
        this.num = arg1 + arg2 + value
    }
}

main() {
    let a = A()
    // arg1 = 1 ,  arg2 = 2 , value = 30
    a[1, 2] = 30

    println(a.num) // 33
}
```

### Function Call Operator

Function call operator (`()`) overloading function, input parameters and return value types can be any type:

```javascript
open class A {
    public init() {}

    public operator func ()(): Unit {}
}

func test1() {
    let a = A()
    a()
}
```

Note that you cannot use this and super for overloading in non-constructor functions.

```javascript
open class A {
    public init() {}
    public init(x: Int64) {
        this() // Ok, this() calls the constructor of A.
    }

    public operator func ()(): Unit {}

	// Non-constructor function using this() overloading
    public func foo() {
        this()  // Error, this() calls the constructor of A.
        super() // Error
    }
}

class B <: A {
    public init() {
        super() // Ok, super() calls the constructor of the super class.
    }

    public func goo() {
        super() // Error
    }
}
```

For enum types, when both constructor form and `()` operator overloading function form are satisfied, constructor form takes priority. Example:

```javascript
enum E {
    Y | X | X(Int64)

    public operator func ()(p: Int64) {}
    public operator func ()(p: Float64) {}
}

main() {
    let e = X(1) // Ok, X(1) is to call the constructor X(Int64).
    X(1.0) // Ok, X(1.0) is to call the operator () overloading function.
    let e1 = X
    e1(1) // Ok, e1(1) is to call the operator () overloading function.
    Y(1) // Ok, Y(1) is to call the operator () overloading function.
}
```

# [Operator Overloading](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/function/operator_overloading.html#操作符重载)

If you want to support operators that are not supported by default on a certain type, you can use operator overloading to implement this.

If you need to overload a certain operator on a certain type, you can implement this by defining a function for the type with the operator as the function name. This way, when an instance of this type uses this operator, the operator function will be automatically called.

Operator function definition is similar to ordinary function definition, with the following differences:

- When defining an operator function, you need to add the `operator` modifier before the `func` keyword;
- The number of parameters in an operator function needs to match the requirements of the corresponding operator (see Appendix [Operators](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/Appendix/operator.html));
- Operator functions can only be defined in class, interface, struct, enum, and extend;
- Operator functions have the semantics of instance member functions, so the `static` modifier is prohibited;
- Operator functions cannot be generic functions.

Additionally, note that overloaded operators do not change their inherent precedence and associativity (see Appendix [Operators](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/Appendix/operator.html)).

## [Operator Overloading Function Definition and Usage](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/function/operator_overloading.html#操作符重载函数定义和使用)

There are two ways to define operator functions:

1. For types that can directly contain function definitions (including `struct`, `enum`, `class`, and `interface`), you can directly define operator functions within them to implement operator overloading.
2. Use the `extend` method to add operator functions to them, thereby implementing operator overloading on these types. For types that cannot directly contain function definitions (meaning types other than `struct`, `class`, `enum`, and `interface`) or types whose implementation cannot be changed, such as third-party defined `struct`, `class`, `enum`, and `interface`, only this method can be used (see [Extensions](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/extension/extend_overview.html));

The conventions for operator function parameters are as follows:

1. For unary operators, operator functions have no parameters, with no requirements on the return value type.

2. For binary operators, operator functions have only one parameter, with no requirements on the return value type.
