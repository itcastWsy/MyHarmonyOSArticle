# 45-Generics-Interface-Class-Struct

## Generic Interfaces

Generics can be used to define generic interfaces. Taking the `Iterable` defined in the standard library as an example, it needs to return an `Iterator` type, which is a traverser for a container. `Iterator` is a generic interface. Inside `Iterator`, there is a `next` member function that returns the next element from the container type. The return type of the `next` member function is a type that needs to be specified when used, so `Iterator` needs to declare generic parameters.

```javascript
public interface Iterable<E> {
    func iterator(): Iterator<E>
}

public interface Iterator<E> <: Iterable<E> {
    func next(): Option<E>
}

public interface Collection<T> <: Iterable<T> {
     prop size: Int64
     func isEmpty(): Bool
}
```

## Generic Classes

The [Generic Interfaces](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/generic/generic_interface.html) section introduced the definition and usage of generic interfaces. In this section, we introduce the definition and usage of generic classes. For example, the key-value pairs in `Map` are defined using generic classes.

You can see that the key-value pair `Node` type in the `Map` type can be defined using generic classes:

```javascript
public open class Node<K, V> where K <: Hashable & Equatable<K> {
    public var key: Option<K> = Option<K>.None
    public var value: Option<V> = Option<V>.None

    public init() {}

    public init(key: K, value: V) {
        this.key = Option<K>.Some(key)
        this.value = Option<V>.Some(value)
    }
}
```

Since the types of keys and values may be different and can be any type that meets the conditions, `Node` needs two type parameters `K` and `V`. `K <: Hashable, K <: Equatable<K>` is a constraint on the key type, meaning that `K` must implement the `Hashable` and `Equatable<K>` interfaces, which are the conditions that `K` needs to satisfy. For generic constraints, see the [Generic Constraints](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/generic/generic_constraint.html) chapter.

## Generic Structs

Generics for struct types are similar to classes. Below we can use struct to define a type similar to a binary tuple:

```javascript
struct Pair<T, U> {
    let x: T
    let y: U
    public init(a: T, b: U) {
        x = a
        y = b
    }
    public func first(): T {
        return x
    }
    public func second(): U {
        return y
    }
}

main() {
    var a: Pair<String, Int64> = Pair<String, Int64>("hello", 0)
    println(a.first())
    println(a.second())
}
```

The program output is:

```text
hello
0
```

In `Pair`, we provide two functions `first` and `second` to get the first and second elements of the tuple.