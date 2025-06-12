# 21-Lambda Expressions

## Introduction

Lambda expressions can be seen as a shorthand for functions. For functions, there are function declarations and function calls. So when learning Lambda expressions, we can also approach it from this perspective. Lambda expressions are particularly flexible, which requires special attention when learning.

## Basic Syntax

The syntax of Lambda expressions is in the following form: `{ p1: T1, ..., pn: Tn => expressions | declarations }`

1. The outer layer of Lambda expressions is represented by `{}`
2. The left side of `=>` is the parameter part
3. The right side of `=>` is the return value or logic code part

## Simple Declaration

```
    // Declare f1 with parameter a, a's type is Int64, return value is a+100
    let f1 = {a: Int64 => a + 100}
```

## Simple Invocation

```
	println(f1(200)) // Call f1 as a function
```

## Omitting Parameter Types

Lambda expressions can omit parameter types when the type is already declared:

```
    // Without omission
    let f1: (a: Int64) -> Int64 = {a: Int64 => a}
    // With omission
    let f2: (a: Int64) -> Int64 = {a => a}
```

## No Parameters

When Lambda expressions have no parameters, the syntax is as follows:

```
    // With parameters
    let f3 = {a: Int64 => a + 100}
    // Without parameters
    let f4 = {=> 100}
```

## No Return Value

When Lambda expressions have no return value, the syntax is as follows:

```
    // No return value and no parameters
    let f5 = { => }
```

## Immediate Invocation

Lambda expressions support immediate invocation, for example:

```
    let f6 = {a: Int64, b: Int64 => a + b}(1, 2)
    println(f6) // 3

    let f7 = {=> 123}()
    println(f7) // 123
```

## Passing and Calling as Variables

```
    func f() {
        var g = { x: Int64 => println("x = ${x}") }
        g(2)
    }
```
