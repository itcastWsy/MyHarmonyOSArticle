# 44-Generics-Functions

## Overview of Generics

In the Cangjie programming language, generics refer to parameterized types, which are types that are unknown at declaration time and need to be specified when used. Type declarations and function declarations can be generic. The most common examples are container types like `Array<T>`, `Set<T>`, etc. Taking array types as an example, when using the array type `Array`, we need to store different types in it. We cannot define arrays for all types. By declaring type parameters in type declarations and specifying the types when applying arrays, we can reduce code duplication.

In Cangjie, declarations of class, interface, struct, and enum can all declare type parameters, which means they can all be generic.

## Concept of Generic Functions

If a function declares one or more type parameters, it is called a generic function. Syntactically, type parameters immediately follow the function name and are enclosed in `<>`. If there are multiple type parameters, they are separated by commas.

According to usage scenarios, generic functions can be classified into the following categories:

1. Global generic functions
2. Local generic functions
3. Generic member functions
4. Static generic functions

## Global Generic Functions

If a function declares one or more type parameters, it is called a generic function. Syntactically, type parameters immediately follow the function name and are enclosed in `<>`. If there are multiple type parameters, they are separated by commas.

```javascript
func id<T>(a: T): T {
    return a
}

main() {
    let res = id<Int64>(100) // 100 
}
```

Here `(a: T)` is the formal parameter of the function declaration, which uses the type parameter `T` declared by the `id` function, and is also used in the return type of the `id` function.

Here's a more complex example:

```javascript
func composition<T1, T2, T3>(f: (T1) -> T2, g: (T2) -> T3): (T1) -> T3 {
    return {x: T1 => g(f(x))}
}

func times2(a: Int64): Int64 {
    return a * 2
}

func plus10(a: Int64): Int64 {
    return a + 10
}

func times2plus10(a: Int64) {
    return composition<Int64, Int64, Int64>(times2, plus10)(a)
}

main() {
  println(times2plus10(9))
  return 0
}
```

## Local Generic Functions

Local functions can also be generic functions. For example, the generic function `id` can be nested and defined within other functions:

```javascript
func foo(a: Int64) {
    func id<T>(a: T): T { a }

    func double(a: Int64): Int64 { a + a }

    return (id<Int64> ~> double)(a) == (double ~> id<Int64>)(a)
}

main() {
    println(foo(1))
    return 0
}
```

Here, due to the identity property of `id`, the functions `id<Int64> ~> double` and `double ~> id<Int64>` are equivalent, and the result is `true`.

```javascript
true
```

## Generic Member Functions

Member functions of class, struct, and enum can be generic. For example:

```javascript
class A {
    func foo<T>(a: T): Unit where T <: ToString {
        println("${a}")
    }
}

struct B {
    func bar<T>(a: T): Unit where T <: ToString {
        println("${a}")
    }
}

enum C {
    | X | Y

    func coo<T>(a: T): Unit where T <: ToString {
        println("${a}")
    }
}

main() {
    var a = A()
    var b = B()
    var c = C.X
    a.foo<Int64>(10)
    b.bar<String>("abc")
    c.coo<Bool>(false)
    return 0
}
```

The program output is:

```text
10
abc
false
```

It should be noted that generic member functions declared in a class cannot be modified with `open`. If modified with `open`, it will result in an error, for example:

```javascript
class A {
    public open func foo<T>(a: T): Unit where T <: ToString { // Error, open generic function is not allowed
        println("${a}")
    }
}
```

When extending types using extend declarations, functions in the extension can also be generic. For example, we can add a generic member function to the `Int64` type:

```javascript
extend Int64 {
    func printIntAndArg<T>(a: T) where T <: ToString {
        println(this)
        println("${a}")
    }
}

main() {
    var a: Int64 = 12
    a.printIntAndArg<String>("twelve")
}
```

The program output will be:

```text
12
twelve
```

## Static Generic Functions

Static generic functions can be defined in interface, class, struct, enum, and extend. For example, the following example shows a `ToPair` class that returns a tuple from an `ArrayList`:

```javascript
import std.collection.*

class ToPair {
    public static func fromArray<T>(l: ArrayList<T>): (T, T) {
        return (l[0], l[1])
    }
}

main() {
    var res: ArrayList<Int64> = ArrayList([1,2,3,4])
    var a: (Int64, Int64) = ToPair.fromArray<Int64>(res)
    return 0
}
```