# Generic Constraints

The purpose of generic constraints is to clarify the operations and capabilities that generic type parameters possess when declaring functions, classes, enums, and structs. Only by declaring these constraints can corresponding member functions be called. In many scenarios, generic type parameters need to be constrained. Taking the `id` function as an example:

```javascript
func id<T>(a: T) {
    return a
}
```

The only thing we can do is return the function parameter `a`, but we cannot perform operations like `a + 1` or `println("${a}")`, because it could be any type, such as `(Bool) -> Bool`, which cannot be added to an integer, and similarly, being a function type, it cannot be output to the command line through the `println` function. However, if this generic type parameter has constraints, then more operations can be performed.

Constraints are roughly divided into interface constraints and subtype constraints. The syntax is to use the `where` keyword before the declaration body of functions and types. For declared generic type parameters `T1, T2`, you can use `where T1 <: Interface, T2 <: Type` to declare generic constraints. Multiple constraints for the same type variable can be connected using `&`. For example: `where T1 <: Interface1 & Interface2`.

For example, the `println` function in Cangjie can accept parameters of string type. If we need to convert a variable of generic type to a string and then print it on the command line, we can constrain this generic type variable. This constraint is the `ToString` interface defined in `core`, which is obviously an interface constraint:

```javascript
package core // `ToString` is defined in core.

public interface ToString {
    func toString(): String
}
```

This way we can use this constraint to define a function named `genericPrint`:

```javascript
func genericPrint<T>(a: T) where T <: ToString {
    println(a)
}

main() {
    genericPrint<Int64>(10)
    return 0
}
```

The result is:

```text
10
```

If the type argument of the genericPrint function does not implement the ToString interface, the compiler will report an error. For example, when we pass a function as a parameter:

```javascript
func genericPrint<T>(a: T) where T <: ToString {
    println(a)
}

main() {
    genericPrint<(Int64) -> Int64>({ i => 0 })
    return 0
}
```

If we compile the above file, the compiler will throw an error about generic type parameters not satisfying constraints. Because the generic type argument of the `genericPrint` function does not satisfy the constraint `(Int64) -> Int64 <: ToString`.

In addition to using interfaces to represent constraints as described above, subtypes can also be used to constrain a generic type variable. For example: when we want to declare a zoo type `Zoo<T>`, but we need the declared type parameter `T` to be constrained, this constraint is that `T` needs to be a subtype of the animal type `Animal`. The `Animal` type declares a `run` member function. Here we declare two subtypes `Dog` and `Fox` that both implement the `run` member function, so that in the `Zoo<T>` type, we can call the `run` member function on animal instances stored in the `animals` array list:

```javascript
import std.collection.*

abstract class Animal {
    public func run(): String
}

class Dog <: Animal {
    public func run(): String {
        return "dog run"
    }
}

class Fox <: Animal {
    public func run(): String {
        return "fox run"
    }
}

class Zoo<T> where T <: Animal {
    var animals: ArrayList<Animal> = ArrayList<Animal>()
    public func addAnimal(a: T) {
        animals.append(a)
    }

    public func allAnimalRuns() {
        for(a in animals) {
            println(a.run())
        }
    }
}

main() {
    var zoo: Zoo<Animal> = Zoo<Animal>()
    zoo.addAnimal(Dog())
    zoo.addAnimal(Fox())
    zoo.allAnimalRuns()
    return 0
}
```

The program output is:

```text
dog run
fox run
```