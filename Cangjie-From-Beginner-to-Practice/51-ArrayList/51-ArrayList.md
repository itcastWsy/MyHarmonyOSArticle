# Collection Type Introduction

Several commonly used basic Collection types in Cangjie include Array, ArrayList, HashSet, and HashMap.

You can choose the appropriate type for your business scenario:

- Array: Use it if you don't need to add or remove elements but need to modify elements.
- **ArrayList**: Use it if you need frequent addition, deletion, querying, and modification of elements.
- HashSet: Use it if you want each element to be unique.
- HashMap: Use it if you want to store a series of mapping relationships.

The following table shows the basic characteristics of these types:

| Type Name       | Mutable Elements | Add/Remove Elements | Element Uniqueness | Ordered Sequence |
| --------------- | ---------------- | ------------------- | ------------------ | ---------------- |
| `Array<T>`      | Y                | N                   | N                  | Y                |
| `ArrayList<T>`  | Y                | Y                   | N                  | Y                |
| `HashSet<T>`    | N                | Y                   | Y                  | N                |
| `HashMap<K, V>` | K: N, V: Y       | Y                   | K: Y, V: N         | N                |

## ArrayList Initialization

You need to import it first.

```typescript
import std.collection.*
```

ArrayList supports multiple initialization methods:

```typescript
package pro
import std.collection.*

main() {
    // Create an empty string list with default initial capacity (10)
    let a = ArrayList<String>()
    
    // Create an empty string list with initial capacity of 100
    let b = ArrayList<String>(100)
    
    // Initialize Int64 list from static array (elements 0,1,2)
    let c = ArrayList<Int64>([0, 1, 2])
    
    // Copy constructor: create a copy with the same content as list c
    let d = ArrayList<Int64>(c)
    
    // Initialize using generator function: create a string list with capacity 2
    // Convert Int64 values to strings through lambda expression
    let e = ArrayList<String>(2, {x: Int64 => x.toString()})
}
```

## ArrayList Access

Supports direct access using subscripts.

```typescript
let c = ArrayList<Int64>([0, 1, 2])
println(c[1])
```

Access the length of the list through size.

```typescript
println(c.size)
```

You can access all members in the list through for-in.

```typescript
let list = ArrayList<Int64>([0, 1, 2])
for (i in list) {
    println("The element is ${i}")
}
```

## ArrayList Modification

You can modify directly through subscripts:

```typescript
let list = ArrayList<Int64>([0, 1, 2])
list[0] = 3
```

## ArrayList Addition

Use **append** and **appendAll** to add elements at the end of the list, and use **insert** and **insertAll** to insert elements at specified positions.

```typescript
let list = ArrayList<Int64>()
list.append(0) // list contains element 0
list.append(1) // list contains elements 0, 1

let li = [2, 3]
list.appendAll(li) // list contains elements 0, 1, 2, 3

let list = ArrayList<Int64>([0, 1, 2]) // list contains elements 0, 1, 2
list.insert(1, 4) // list contains elements 0, 4, 1, 2
```

If you know approximately how many elements you need to add, you can prepare enough memory before adding to avoid intermediate reallocation, which can improve performance.

```typescript
    let list = ArrayList<Int64>(100) // Allocate space at once
    for (i in 0..100) {
        list.append(i) // Does not trigger reallocation of space
    }
    list.reserve(100) // Prepare more space
    for (i in 0..100) {
        list.append(i) // Does not trigger reallocation of space
    }
```

## ArrayList Deletion

Use the remove method to delete at a specified position:

```typescript
let list = ArrayList<String>(["a", "b", "c", "d"]) // list contains the elements "a", "b", "c", "d"
list.remove(1) // Delete the element at subscript 1, now the list contains elements "a", "c", "d"
```