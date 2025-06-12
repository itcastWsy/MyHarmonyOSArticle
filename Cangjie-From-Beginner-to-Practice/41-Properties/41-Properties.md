# Properties

Properties provide a getter and an optional setter to indirectly get and set values.

When using properties, they are no different from ordinary variables. We only need to operate on the data without being aware of the internal implementation, making it more convenient to implement access control, data monitoring, tracking debugging, data binding, and other mechanisms.

Properties can be used as expressions or assigned to when in use. This section uses classes and interfaces as examples for explanation, but properties are not limited to classes and interfaces.

Here is a simple example where b is a typical property that encapsulates external access to a:

```javascript
class Foo {
    private var a = 0

    public mut prop b: Int64 {
        get() {
            println("get")
            a
        }
        set(value) {
            println("set")
            a = value
        }
    }
}

main() {
    var x = Foo()
    let y = x.b + 1 // get
    x.b = y // set
}
```

Here Foo provides a property named b. For the getter/setter functionality, Cangjie provides get and set syntax to define them. When a variable x of type Foo accesses b, it will call b's get operation to return a value of type Int64, so it can be used to add with 1; and when x assigns to b, it will call b's set operation, passing the value of y to set's value parameter, and finally assigning the value of value to a.

Through property b, the external code is completely unaware of Foo's member variable a, but can still perform the same access and modification operations through b, achieving effective encapsulation. So the program output is as follows:

```text
get
set
```

## Property Definition

Properties can be defined in interface, class, struct, enum, and extend.

A typical property syntax structure is as follows:

```javascript
class Foo {
    public prop a: Int64 {
        get() { 0 }
    }
    public mut prop b: Int64 {
        get() { 0 }
        set(v) {}
    }
}
```

Here both a and b declared with prop are properties, and both a and b are of type `Int64`. a is a property without the `mut` modifier, and such properties have only a getter (corresponding to value retrieval) implementation. b is a property modified with `mut`, and such properties must define both getter (corresponding to value retrieval) and setter (corresponding to assignment) implementations.

The getter and setter of a property correspond to two different functions respectively.

1. The getter function type is `() -> T`, where T is the type of the property. The getter function is executed when the property is used as an expression.
2. The setter function type is `(T) -> Unit`, where T is the type of the property. The parameter name needs to be explicitly specified, and the setter function is executed when the property is assigned.

The implementations of getter and setter can contain declarations and expressions just like function bodies, following the same rules as function bodies. See the [Function Body](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/function/define_functions.html#函数体) chapter for details.

The parameter in the setter corresponds to the value passed in during assignment.

```javascript
class Foo {
    private var j = 0
    public mut prop i: Int64 {
        get() {
            j
        }
        set(v) {
            j = v
        }
    }
}
```

Note that accessing the property itself in the property's getter and setter is a recursive call, which may cause infinite loops just like function calls.

### Modifiers

We can declare the required modifiers before prop.

```javascript
class Foo {
    public prop a: Int64 {
        get() {
            0
        }
    }
    private prop b: Int64 {
        get() {
            0
        }
    }
}
```

Like member functions, member properties also support `open`, `override`, and `redef` modifiers, so we can also override/redefine parent type property implementations in subtypes.

When a subtype overrides a parent type's property, if the parent type property has the `mut` modifier, then the subtype property also needs to have the `mut` modifier, and must also maintain the same type.

As shown in the following code, A defines two properties x and y, and B can `override`/`redef` x and y respectively:

```javascript
open class A {
    private var valueX = 0
    private static var valueY = 0

    public open prop x: Int64 {
        get() { valueX }
    }

    public static mut prop y: Int64 {
        get() { valueY }
        set(v) {
            valueY = v
        }
    }
}
class B <: A {
    private var valueX2 = 0
    private static var valueY2 = 0

    public override prop x: Int64 {
        get() { valueX2 }
    }

    public redef static mut prop y: Int64 {
        get() { valueY2 }
        set(v) {
            valueY2 = v
        }
    }
}
```

### Abstract Properties

Similar to abstract functions, we can also declare abstract properties in interfaces and abstract classes. These abstract properties have no implementation.

```javascript
interface I {
    prop a: Int64
}

abstract class C {
    public prop a: Int64
}
```

When implementing types implement an interface or non-abstract subclasses inherit from an abstract class, they must implement these abstract properties.

Following the same rules as overriding, when implementing types or subclasses implement these properties, if the parent type property has the `mut` modifier, then the subtype property also needs to have the `mut` modifier, and must also maintain the same type.

```javascript
interface I {
    prop a: Int64
    mut prop b: Int64
}
class C <: I {
    private var value = 0

    public prop a: Int64 {
        get() { value }
    }

    public mut prop b: Int64 {
        get() { value }
        set(v) {
            value = v
        }
    }
}
```

Through abstract properties, we can make interfaces and abstract classes agree on some data operations in a more user-friendly way, which is more intuitive compared to using functions.

As shown in the following code, if we want to agree on getting and setting a size value, using the property approach (I1) compared to using the function approach (I2) requires less code and is more in line with the intent of data operations.

```javascript
interface I1 {
    mut prop size: Int64
}

interface I2 {
    func getSize(): Int64
    func setSize(value: Int64): Unit
}

class C <: I1 & I2 {
    private var mySize = 0

    public mut prop size: Int64 {
        get() {
            mySize
        }
        set(value) {
            mySize = value
        }
    }

    public func getSize() {
        mySize
    }

    public func setSize(value: Int64) {
        mySize = value
    }
}

main() {
    let a: I1 = C()
    a.size = 5
    println(a.size)

    let b: I2 = C()
    b.setSize(5)
    println(b.getSize())
}
5
5
```

## Property Usage

Properties are divided into instance member properties and static member properties. The usage of member properties is the same as the usage of member variables. See the [Member Variables](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/class_and_interface/class.html#class-成员变量) chapter for details.

```javascript
class A {
    public prop x: Int64 {
        get() {
            123
        }
    }
    public static prop y: Int64 {
        get() {
            321
        }
    }
}

main() {
    var a = A()
    println(a.x) // 123
    println(A.y) // 321
}
```

The result is:

```text
123
321
```

Properties without the `mut` modifier are similar to variables declared with let and cannot be assigned to.

```javascript
class A {
    private let value = 0
    public prop i: Int64 {
        get() {
            value
        }
    }
}

main() {
    var x = A()
    println(x.i) // OK
    x.i = 1 // Error
}
```

Properties with the `mut` modifier are similar to variables declared with var and can be both retrieved and assigned to.

```javascript
class A {
    private var value: Int64 = 0
    public mut prop i: Int64 {
        get() {
            value
        }
        set(v) {
            value = v
        }
    }
}

main() {
    var x = A()
    println(x.i) // OK
    x.i = 1 // OK
}
0
```