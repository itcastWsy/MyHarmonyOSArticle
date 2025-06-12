# 22-Closures

## Introduction

Closures are a relatively abstract concept in Cangjie programming. Let's first understand the purpose of closures, then learn their syntax.

The emergence of closures allows us to implement the functionality of hiding private variables or caching data state. Additionally, their presence can simplify certain parts of our code.

Therefore, closures exist to provide a better programming experience in certain scenarios, rather than being absolutely necessary.

## Definition of Closures

When a function or lambda captures variables from the static scope where it is defined, the function or lambda together with the captured variables is called a closure. This allows the closure to run normally even when it leaves the scope where it was defined.

## Example Code

```javascript
class C {
    public var num: Int64 = 0
}

func returnIncrementer(){
    let c: C = C()

    func incrementer() {
        c.num++
        println( c.num)
    }

    incrementer
}

main() {
    let f = returnIncrementer()
    f() // 1 
    f() // 2
    f() // 3 
}
```

"When a function or lambda captures variables from the static scope where it is defined, the function or lambda together with the captured variables is called a closure. This allows the closure to run normally even when it leaves the scope where it was defined."

1. **incrementer** is the function that captures variables
2. **C** is the captured variable, which is also the cached and hidden variable
3. **f** is just the manifestation of calling the function in the closure
4. Finally, when we call **function f** multiple times, the captured variable inside will keep increasing.

This is the most concise understanding of closures.

The following content is simply about the restrictions and different usages of closure syntax.

## Variable Capture

When a function or lambda captures variables from the static scope where it is defined, the function or lambda together with the captured variables is called a closure. This allows the closure to run normally even when it leaves the scope where it was defined.

Access to the following types of variables in the definition of a function or lambda is called variable capture:

- Accessing local variables defined outside the current function in the default values of function parameters;
- Accessing local variables defined outside the current function or lambda within the function or lambda;
- Functions or lambdas defined within a `class`/`struct` that are not member functions accessing instance member variables or `this`.

The following cases of variable access are not variable capture:

- Access to local variables defined within the current function or lambda;
- Access to formal parameters of the current function or lambda;
- Access to global variables and static member variables;
- Access to instance member variables within instance member functions or properties. Since instance member functions or properties pass `this` as a parameter, all instance member variables are accessed through `this` within instance member functions or properties.

Variable capture occurs when the closure is defined, so variable capture has the following rules:

- The captured variable must be visible when the closure is defined, otherwise a compilation error occurs;
- The captured variable must have completed initialization when the closure is defined, otherwise a compilation error occurs.

## Example Code

Example 1: The closure `add` captures the local variable `num` declared with `let`, then returns it outside the scope where `num` is defined through the return value. When calling `add`, `num` can still be accessed normally.

```javascript
func returnAddNum(): (Int64) -> Int64 {
    let num: Int64 = 10

    func add(a: Int64) {
        return a + num
    }
    add
}

main() {
    let f = returnAddNum()
    println(f(10))
}
```

The program output is:

```text
20
```

Example 2: Captured variables must be visible when the closure is defined.

```javascript
func f() {
    let x = 99
    func f1() {
        println(x)
    }
    let f2 = { =>
        println(y)      // Error, cannot capture 'y' which is not defined yet
    }
    let y = 88
    f1()          // Print 99.
    f2()
}
```

Example 3: Captured variables must complete initialization before the closure is defined.

```javascript
func f() {
    let x: Int64
    func f1() {
        println(x)    // Error, x is not initialized yet.
    }
    x = 99
    f1()
}
```

If the captured variable is a reference type, you can modify the values of its mutable instance member variables.

```javascript
class C {
    public var num: Int64 = 0
}

func returnIncrementer(): () -> Unit {
    let c: C = C()

    func incrementer() {
        c.num++
    }

    incrementer
}

main() {
    let f = returnIncrementer()
    f() // c.num increases by 1
}
```

To prevent closures that capture `var` declared variables from escaping, such closures can only be called and cannot be used as first-class citizens, including not being assignable to variables, not being used as actual parameters or return values, and not directly using the closure's name as an expression.

```javascript
func f() {
    var x = 1
    let y = 2

    func g() {
        println(x)  // OK, captured a mutable variable.
    }
    let b = g  // Error, g cannot be assigned to a variable

    g  // Error, g cannot be used as an expression
    g()  // OK, g can be invoked

    g  // Error, g cannot be used as a return value.
}
```

Note that capture has transitivity. If a function `f` calls a function `g` that captures a `var` variable, and the `var` variable captured by `g` is not defined within function `f`, then function `f` also captures the `var` variable. In this case, `f` also cannot be used as a first-class citizen.

In the following example, `g` captures the `var` declared variable `x`, `f` calls `g`, and the `x` captured by `g` is not defined within `f`, so `f` also cannot be used as a first-class citizen:

```javascript
func h(){
    var x = 1

    func g() {  x }   // captured a mutable variable

    func f() {
        g()      // invoked g
    }
    return f // Error
}
```

In the following example, `g` captures the `var` declared variable `x`, and `f` calls `g`. However, the `x` captured by `g` is defined within `f`, and `f` does not capture any other `var` declared variables. Therefore, `f` can still be used as a first-class citizen:

```javascript
func h(){
    func f() {
        var x = 1
        func g() { x }   // captured a mutable variable

        g()
    }
    return f // Ok
}
```

Access to static member variables and global variables does not belong to variable capture, so functions or lambdas that access `var` modified global variables and static member variables can still be used as first-class citizens.

```javascript
class C {
    static public var a: Int32 = 0
    static public func foo() {
        a++       // OK
        return a
    }
}

var globalV1 = 0

func countGlobalV1() {
    globalV1++
    C.a = 99
    let g = C.foo  // OK
}

func g(){
    let f = countGlobalV1 // OK
    f()
}
```