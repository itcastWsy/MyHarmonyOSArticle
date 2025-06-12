# 24-Function Overloading

## Definition of Function Overloading

In the Cangjie programming language, if **one function name corresponds to multiple function definitions** within a scope, this phenomenon is called function overloading.

### Same Function Name, Different Function Parameters

This refers to two functions that constitute overloading when they have different numbers of parameters, or the same number of parameters but different parameter types. Examples are as follows:

```javascript
// Scenario 1
func f(a: Int64): Unit {
}

func f(a: Float64): Unit {
}

func f(a: Int64, b: Float64): Unit {
}
```

### Generic Functions with Different Generic Parameters

For two generic functions with the same name, if after renaming the generic formal parameters of one function, its non-generic part has different function parameters from the non-generic part of another function, then the two functions constitute overloading. Otherwise, these two generic functions constitute a redefinition error (type parameter constraints do not participate in the judgment). Examples are as follows:

```javascript
interface I1{}
interface I2{}

func f1<X, Y>(a: X, b: Y) {}
func f1<Y, X>(a: X, b: Y) {} // Ok: a b parts are different

func f2<T>(a: T) where T <: I1 {}
func f2<T>(a: T) where T <: I2 {} // Error, other generic parts are the same
```

### Two Constructors in a Class with Different Parameters

```javascript
// Scenario 2
class C {
    var a: Int64
    var b: Float64

    public init(a: Int64, b: Float64) {
        this.a = a
        this.b = b
    }

    public init(a: Int64) {
        b = 0.0
        this.a = a
    }
}
```

### Primary Constructor and init Constructor Parameters

```javascript
// Scenario 3
class C {
    C(var a!: Int64, var b!: Float64) {
        this.a = a
        this.b = b
    }

    public init(a: Int64) {
        b = 0.0
        this.a = a
    }
}
```

### Two Functions Defined in Different Scopes

```javascript
// Scenario 4
func f(a: Int64): Unit {
}

func g() {
    func f(a: Float64): Unit {
    }
}
```

### Two Functions Defined in Parent and Child Classes Respectively

```javascript
// Scenario 5
open class Base {
    public func f(a: Int64): Unit {
    }
}

class Sub <: Base {
    public func f(a: Float64): Unit {
    }
}
```

## Cases That Do Not Constitute Overloading

- Static member functions and instance member functions of class, interface, struct types cannot be overloaded
- Constructor, static member functions and instance member functions of enum types cannot be overloaded

In the following example, both variables are of function type and have different function parameter types, but since they are not function declarations, they cannot be overloaded. The following example will result in a compilation error (redefinition error):

```javascript
main() {
    var f: (Int64) -> Unit
    var f: (Float64) -> Unit
}
```

In the following example, although variable `f` is of function type, since variables and functions cannot have the same name, the following example will result in a compilation error (redefinition error):

```javascript
main() {
    var f: (Int64) -> Unit

    func f(a: Float64): Unit {   // Error, functions and variables cannot have the same name.
    }
}
```

In the following example, static member function `f` and instance member function `f` have different parameter types, but since static member functions and instance member functions within a class cannot be overloaded, the following example will result in a compilation error:

```javascript
class C {
    public static func f(a: Int64): Unit {
    }
    public func f(a: Float64): Unit {
    }
}
```

## Function Overload Resolution

When calling a function, all callable functions (functions that are visible in the current scope and can pass type checking) form a candidate set. When there are multiple functions in the candidate set, determining which function to choose from the candidate set requires function overload resolution, with the following rules:

- Prioritize functions in scopes with higher scope levels. In nested expressions or functions, the more inner the scope, the higher the scope level.

  In the following example, when calling `g(Sub())` within the `inner` function body, the candidate set includes the function `g` defined within the `inner` function and the function `g` defined outside the `inner` function. Function resolution chooses the function `g` defined within the `inner` function, which has a higher scope level.

  ```javascript
  open class Base {}
  class Sub <: Base {}
  
  func outer() {
      func g(a: Sub) {
          print("1")
      }
  
      func inner() {
          func g(a: Base) {
              print("2")
          }
  
          g(Sub())   // Output: 2
      }
  }
  ```

- If there are still multiple functions with the relatively highest scope level, then the most matching function needs to be selected (for functions f and g and given actual parameters, if f can be called when g can always be called, but not vice versa, then we say f is more matching than g). If there is no unique most matching function, an error is reported.

  In the following example, two functions `g` are defined in the same scope, and the more matching function `g(a: Sub): Unit` is selected.

  ```javascript
  open class Base {}
  class Sub <: Base {}
  
  func outer() {
      func g(a: Sub) {
          print("1")
      }
      func g(a: Base) {
          print("2")
      }
  
      g(Sub())   // Output: 1
  
  }
  ```

- Child classes and parent classes are considered to be in the same scope. In the following example, one function `g` is defined in the parent class, and another function `g` is defined in the child class. When calling `s.g(Sub())`, the two functions `g` are resolved as the same scope level, so the more matching function `g(a: Sub): Unit` defined in the parent class is selected.

  ```javascript
  open class Base {
      public func g(a: Sub) { print("1") }
  }
  
  class Sub <: Base {
      public func g(a: Base) {
          print("2")
      }
  }
  
  func outer() {
      let s: Sub = Sub()
      s.g(Sub())   // Output: 1
  }
  ```