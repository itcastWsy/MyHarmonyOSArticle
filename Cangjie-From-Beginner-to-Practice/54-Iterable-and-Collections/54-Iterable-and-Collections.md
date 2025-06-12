# 54-Iterable and Collections

## Iterable and Collections

We have already learned about Range, Array, and ArrayList, all of which can be traversed using for-in operations. So, can a user-defined type implement similar traversal operations? The answer is yes.

Range, Array, and ArrayList actually all support for-in syntax through Iterable.

Iterable is a built-in interface in the following form (only core code is shown):

```javascript
interface Iterable<T> {
    func iterator(): Iterator<T>
    ...
}
```

The iterator function requires returning an Iterator type, which is another built-in interface in the following form (only core code is shown):

```javascript
interface Iterator<T> <: Iterable<T> {
    mut func next(): Option<T>
    ...
}
```

You can use for-in syntax to traverse any instance of a type that implements the Iterable interface.

Suppose there is such a for-in code:

```javascript
let list = [1, 2, 3]
for (i in list) {
    println(i)
}
```

Then it is equivalent to the following while code:

```javascript
let list = [1, 2, 3]
var it = list.iterator()
while (true) {
    match (it.next()) {
        case Some(i) => println(i)
        case None => break
    }
}
```

Another common way to traverse Iterable types is to use while-let. For example, another equivalent way to write the above while code is:

```javascript
let list = [1, 2, 3]
var it = list.iterator()
while (let Some(i) <- it.next()) {
    println(i)
}
```

Array, ArrayList, HashSet, and HashMap types all implement Iterable, so they can all be used in for-in or while-let.