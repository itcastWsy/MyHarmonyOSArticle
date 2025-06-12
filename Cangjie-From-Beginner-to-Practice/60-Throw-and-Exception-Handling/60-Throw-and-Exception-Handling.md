# 60-Throw and Exception Handling

The previous section introduced how to define custom exceptions. Now let's learn how to throw and handle exceptions.

- Since exceptions are `class` types, you only need to create exceptions according to the construction method of class objects. For example, the expression `FatherException()` creates an exception of type `FatherException`.
- The Cangjie language provides the `throw` keyword for throwing exceptions. When using `throw` to throw an exception, the expression after `throw` must be a subtype of `Exception` (the `Error` type, which is also an exception, cannot be manually thrown), such as `throw ArithmeticException("I am an Exception!")` (when executed) will throw an arithmetic operation exception.
- Exceptions thrown by the `throw` keyword need to be caught and handled. If an exception is not caught, the system calls the default exception handling function.

Exception handling is completed by `try` expressions, which can be divided into:

- Ordinary try expressions that do not involve automatic resource management;
- Try-with-resources expressions that perform automatic resource management.

## Ordinary try Expressions

Ordinary try expressions include three parts: try block, catch block, and finally block.

- Try block: starts with the keyword `try`, followed immediately by a block composed of expressions and declarations (enclosed in a pair of braces, defining a new local scope, can contain any expressions and declarations, hereinafter referred to as "block"). The block after try can throw exceptions, which are caught and handled by the immediately following catch block (if there is no catch block or the exception is not caught, the exception continues to be thrown after executing the finally block).
- Catch block: an ordinary try expression can contain zero or more catch blocks (when there are no catch blocks, there must be a finally block). Each catch block starts with the keyword `catch`, followed by a `catchPattern` and a block. The `catchPattern` matches the exception to be caught through pattern matching. Once the match is successful, it is handled by the block that follows, and other catch blocks after it are ignored. When the exception types that can be caught by a certain catch block can all be caught by a catch block defined before it, a "catch block unreachable" warning will be reported at this catch block.
- Finally block: starts with the keyword `finally`, followed immediately by a block. In principle, the finally block mainly implements some "cleanup" work, such as releasing resources, and should try to avoid throwing exceptions again in the finally block. And regardless of whether an exception occurs (i.e., whether an exception is thrown in the try block), the content in the finally block will be executed (if the exception is not handled, after executing the finally block, the exception continues to be thrown outward). A try expression can not contain a finally block when it contains catch blocks, otherwise it must contain a finally block.

The block immediately following `try` and each `catch` block have independent scopes.

Here is a simple example with only try and catch blocks:

```typescript
main() {
    try {
        throw NegativeArraySizeException("I am an Exception!")
    } catch (e: NegativeArraySizeException) {
        println(e)
        println("NegativeArraySizeException is caught!")
    }
    println("This will also be printed!")
}
```

The execution result is:

```text
NegativeArraySizeException: I am an Exception!
NegativeArraySizeException is caught!
This will also be printed!
```

Variables introduced in `catchPattern` have the same scope level as variables in the block after `catch`. Introducing the same name again in the catch block will trigger a redefinition error. For example:

```typescript
main() {
    try {
        throw NegativeArraySizeException("I am an Exception!")
    } catch (e: NegativeArraySizeException) {
        println(e)
        let e = 0 // Error, redefinition
        println(e)
        println("NegativeArraySizeException is caught!")
    }
    println("This will also be printed!")
}
```

Here is a simple example of a try expression with a finally block:

```typescript
main() {
    try {
        throw NegativeArraySizeException("NegativeArraySizeException")
    } catch (e: NegativeArraySizeException) {
        println("Exception info: ${e}.")
    } finally {
        println("The finally block is executed.")
    }
}
```

The execution result is:

```text
Exception info: NegativeArraySizeException: NegativeArraySizeException.
The finally block is executed.
```

Try expressions can appear anywhere expressions are allowed. The type determination method of try expressions is similar to that of multi-branch syntax structures like `if` and `match` expressions, which is the least common supertype of all branches except the finally branch. For example, in the following code, both the try expression and variable `x` have type D, which is the least common supertype of E and D; `C()` in the finally branch does not participate in the common supertype calculation (if it did, the least common supertype would become `C`).

Additionally, when the value of a `try` expression is not used, its type is `Unit`, and the types of each branch are not required to have a least common supertype.

```typescript
open class C { }
open class D <: C { }
class E <: D { }
main () {
    let x = try {
        E()
    } catch (e: Exception) {
        D()
    } finally {
        C()
    }
    0
}
```

## try-with-resources Expressions

try-with-resources expressions are mainly for automatically releasing non-memory resources. Unlike ordinary try expressions, both catch blocks and finally blocks in try-with-resources expressions are optional, and one or more `ResourceSpecification`s can be inserted between the try keyword and the subsequent block to request a series of resources (`ResourceSpecification` does not affect the type of the entire try expression). The resources mentioned here correspond to objects at the language level, so `ResourceSpecification` is actually instantiating a series of objects (multiple instantiations are separated by ","). An example of using try-with-resources expressions is shown below:

```typescript
class Worker <: Resource {
    var hasTools: Bool = false
    let name: String

    public init(name: String) {
        this.name = name
    }
    public func getTools() {
        println("${name} picks up tools from the warehouse.")
        hasTools = true
    }

    public func work() {
        if (hasTools) {
            println("${name} does some work with tools.")
        } else {
            println("${name} doesn't have tools, does nothing.")
        }
    }

    public func isClosed(): Bool {
        if (hasTools) {
            println("${name} hasn't returned the tool.")
            false
        } else {
            println("${name} has no tools")
            true
        }
    }
    public func close(): Unit {
        println("${name} returns the tools to the warehouse.")
        hasTools = false
    }
}

main() {
    try (r = Worker("Tom")) {
        r.getTools()
        r.work()
    }
    try (r = Worker("Bob")) {
        r.work()
    }
    try (r = Worker("Jack")) {
        r.getTools()
        throw Exception("Jack left, because of an emergency.")
    }
}
```

The program output is:

```text
Tom picks up tools from the warehouse.
Tom does some work with tools.
Tom hasn't returned the tool.
Tom returns the tools to the warehouse.
Bob doesn't have tools, does nothing.
Bob has no tools
Jack picks up tools from the warehouse.
Jack hasn't returned the tool.
Jack returns the tools to the warehouse.
An exception has occurred:
Exception: Jack left, because of an emergency.
         at test.main()(xxx/xx.cj:xx)
```

Names introduced between the `try` keyword and `{}` have the same scope level as variables introduced in `{}`. Introducing the same name again in `{}` will trigger a redefinition error.

```typescript
class R <: Resource {
    public func isClosed(): Bool {
        true
    }
    public func close(): Unit {
        print("R is closed")
    }
}

main() {
    try (r = R()) {
        println("Get the resource")
        let r = 0 // Error, redefinition
        println(r)
    }
}
```

The type of `ResourceSpecification` in try-with-resources expressions must implement the Resource interface:

```typescript
interface Resource {
    func isClosed(): Bool // When leaving the try-with-resources scope, determine whether the close function needs to be called to release resources
    func close(): Unit  // Release resources when isClosed returns true.
}
```

It should be noted that try-with-resources expressions generally do not need to contain catch blocks and finally blocks, and it is not recommended for users to manually release resources (logical redundancy). However, if you need to explicitly catch and handle exceptions that may be thrown during the try block or resource request and release process, you can still include catch blocks and finally blocks in try-with-resources expressions:

```typescript
class R <: Resource {
    public func isClosed(): Bool {
        true
    }
    public func close(): Unit {
        print("R is closed")
    }
}

main() {
    try (r = R()) {
        println("Get the resource")
    } catch (e: Exception) {
        println("Exception happened when executing the try-with-resources expression")
    } finally {
        println("End of the try-with-resources expression")
    }
}
```

The program output is:

```text
Get the resource
End of the try-with-resources expression
```

The type of try-with-resources expressions is `Unit`.

## Advanced Introduction to CatchPattern

Most of the time, you only want to catch exceptions of a certain type and its subtypes. In this case, use the **type pattern** of CatchPattern to handle; but sometimes you need to handle all exceptions uniformly (such as exceptions should not appear here, if they do, report errors uniformly). In this case, you can use the **wildcard pattern** of CatchPattern to handle.

Type patterns have two syntactic formats:

- `Identifier: ExceptionClass`. This format can catch exceptions of type `ExceptionClass` and its subclasses, convert the caught exception instance to `ExceptionClass`, then bind it with the variable defined by `Identifier`, and then you can access the caught exception instance through the variable defined by Identifier in the catch block.
- `Identifier: ExceptionClass_1 | ExceptionClass_2 | ... | ExceptionClass_n`. This format can concatenate multiple exception classes through the connector `|`, where the connector `|` represents an "or" relationship: it can catch exceptions of type `ExceptionClass_1` and its subclasses, or catch exceptions of type `ExceptionClass_2` and its subclasses, and so on, or catch exceptions of type `ExceptionClass_n` and its subclasses (assuming n is greater than 1). When the type of the exception to be caught belongs to any type in the above "or" relationship or its subtype, this exception will be caught. But since the type of the caught exception cannot be statically determined, the type of the caught exception will be converted to the least common supertype of all types connected by `|`, and the exception instance will be bound with the variable defined by `Identifier`. Therefore, in this type of pattern, the catch block can only access member variables and member functions in the least common supertype of `ExceptionClass_i (1 <= i <= n)` through the variable defined by `Identifier`. Of course, you can also use wildcards instead of `Identifier` in type patterns, the difference is only that wildcards do not perform binding operations.

Example:

```typescript
main(): Int64 {
    try {
        throw IllegalArgumentException("This is an Exception!")
    } catch (e: OverflowException) {
        println(e.message)
        println("OverflowException is caught!")
    } catch (e: IllegalArgumentException | NegativeArraySizeException) {
        println(e.message)
        println("IllegalArgumentException or NegativeArraySizeException is caught!")
    } finally {
        println("finally is executed!")
    }
    return 0
}
```

Execution result:

```text
This is an Exception!
IllegalArgumentException or NegativeArraySizeException is caught!
finally is executed!
```

Example of "the type of the caught exception is the least common supertype of all types connected by `|`":

```typescript
open class Father <: Exception {
    var father: Int32 = 0
}

class ChildOne <: Father {
    var childOne: Int32 = 1
}

class ChildTwo <: Father {
    var childTwo: Int32 = 2
}

main() {
    try {
        throw ChildOne()
    } catch (e: ChildTwo | ChildOne) {
        println("${e is Father}")
    }
}
```

Execution result:

```text
true
```

The syntax of **wildcard pattern** is `_`, which can catch any type of exception thrown in the same-level try block, equivalent to `e: Exception` in type patterns, i.e., catching exceptions defined by Exception subclasses. Example:

```typescript
// Catch with wildcardPattern.
try {
    throw OverflowException()
} catch (_) {
    println("catch an exception!")
}
```