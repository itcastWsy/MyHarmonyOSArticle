# 29-mut Functions

## Introduction

`mut` is an abbreviation for **mutable**, meaning **changeable**.

The struct type is a value type, and its instance member functions cannot modify the instance itself. For example, in the following example, the member function `g` cannot modify the value of member variable `i`.

```javascript
struct Foo {
    var i = 0

    public func g() {
        i += 1 // Error: instance member variable 'i' cannot be modified in immutable function
    }

    public func f() {
        this.i += 1 // Error: instance member variable 'i' cannot be modified in immutable function
    }
}
```

A mut function is a special instance member function that can modify the `struct` instance itself. Inside a `mut` function, the semantics of `this` are special, and this `this` has the ability to modify fields in place.

> **Note:**
>
> `mut` functions are only allowed to be defined within interfaces, structs, and struct extensions (class is a reference type, and instance member functions can modify instance member variables without adding `mut`, so defining `mut` functions in classes is prohibited).

## mut Function Definition

1. Compared to regular instance member functions, mut functions have an additional `mut` keyword for modification.

```javascript
struct Foo {
    var i = 0

    public mut func g() {
        i += 1  // Ok
    }
}
```

2. `mut` can only modify instance member functions, not static member functions.

```javascript
struct A {
    public mut func f(): Unit {} // Ok
    public mut operator func +(rhs: A): A { // Ok
        A()
    }
    public mut static func g(): Unit {} // Error, static member functions cannot be modified with 'mut'
}
```

3. `this` in mut functions cannot be captured and cannot be used as an expression. You cannot capture **instance member variables** of a struct in `mut` functions.

   ```javascript
   struct Foo {
       var i = 0
   
       public mut func f(): Foo {
           let f1 = { => this } // Error, 'this' in mut functions cannot be captured
           let f2 = { => this.i = 2 } // Error, instance member variables in mut functions cannot be captured
           let f3 = { => this.i } // Error, instance member variables in mut functions cannot be captured
           let f4 = { => i } // Error, instance member variables in mut functions cannot be captured
           this // Error, 'this' in mut functions cannot be used as expressions
       }
   }
   ```

## mut Functions in Interfaces

1. Instance member functions in interfaces can also be modified with `mut`

```javascript
interface I {
    mut func f1(): Unit
    func f2(): Unit
}

struct A <: I {
    public mut func f1(): Unit {} // Ok: as in the interface, the 'mut' modifier is used
    public func f2(): Unit {} // Ok: as in the interface, the 'mut' modifier is not used
}

struct B <: I {
    public func f1(): Unit {} // Error, 'f1' is modified with 'mut' in interface, but not in struct
    public mut func f2(): Unit {} // Error, 'f2' is not modified with 'mut' in interface, but did in struct
}

class C <: I {
    public func f1(): Unit {} // Ok
    public func f2(): Unit {} // Ok
}
```

2. When a `struct` instance is assigned to an `interface` type, it uses copy semantics, so the `mut` function of the `interface` cannot modify the value of the `struct` instance.

```javascript
interface I {
    mut func f(): Unit
}
struct Foo <: I {
    public var v = 0
    public mut func f(): Unit {
        v += 1
    }
}
main() {
    var a = Foo()
    var b: I = a  
    b.f()  // Calling 'f' via 'b' cannot modify the value of 'a'
    println(a.v) // 0
}
```

## Usage Restrictions for mut Functions

1. Because `struct` is a value type, if a variable is of `struct` type and declared with `let`, then you cannot access the `mut` functions of that type through this variable.

   ```javascript
   interface I {
       mut func f(): Unit
   }
   struct Foo <: I {
       public var i = 0
       public mut func f(): Unit {
           i += 1
       }
   }
   main() {
       let a = Foo()
       a.f() // Error, 'a' is of type struct and is declared with 'let', the 'mut' function cannot be accessed via 'a'
       var b = Foo()
       b.f() // Ok
       let c: I = Foo()
       c.f() // Ok
   }
   ```

2. To avoid escaping, if a variable's type is a `struct` type, then this variable cannot use functions modified with `mut` of that type as first-class citizens, and can only call these `mut` functions.

   ```javascript
   interface I {
       mut func f(): Unit
   }
   
   struct Foo <: I {
       var i = 0
   
       public mut func f(): Unit {
           i += 1
       }
   }
   
   main() {
       var a = Foo()
       var fn = a.f // Error, mut function 'f' of 'a' cannot be used as a first class citizen.
       var b: I = Foo()
       fn = b.f // Ok
   }
   ```

3. To avoid escaping, non-`mut` instance member functions (including `lambda` expressions) cannot directly access `mut` functions of their type, but the reverse is allowed.

   ```javascript
   struct Foo {
       var i = 0
   
       public mut func f(): Unit {
           i += 1
           g() // Ok
       }
   
       public func g(): Unit {
           f() // Error, mut functions cannot be invoked in non-mut functions
       }
   }
   
   interface I {
       mut func f(): Unit {
           g() // Ok
       }
   
       func g(): Unit {
           f() // Error, mut functions cannot be invoked in non-mut functions
       }
   }
   ```