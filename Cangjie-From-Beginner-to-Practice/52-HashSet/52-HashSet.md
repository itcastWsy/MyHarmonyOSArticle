# 52-HashSet

Use the HashSet type to construct a Collection that only contains non-duplicate elements. The Collection has the functionality to traverse elements, add elements, and delete elements, but does not have the functionality to modify individual elements.

## HashSet Initialization

Before using HashSet, you need to import the package first:

```typescript
import std.collection.*
```

Then you can initialize it:

```typescript
let a = HashSet<String>() // Created an empty HashSet whose element type is String
let b = HashSet<String>(100) // Created a HashSet whose capacity is 100
let c = HashSet<Int64>([0, 1, 2]) // Created a HashSet whose element type is Int64, containing elements 0, 1, 2
let d = HashSet<Int64>(c) // Use another Collection to initialize a HashSet
let e = HashSet<Int64>(10, {x: Int64 => (x * x)}) // Created a HashSet whose element type is Int64 and size is 10. All elements are initialized by specified rule function
```

Note that if you store duplicate elements in a HashSet, the HashSet will automatically filter them out:

```typescript
    let c = HashSet<Int64>([0, 1, 2, 2, 2, 2, 2, 2])
    println(c) // [0,1,2]
```

## HashSet Access

You can traverse HashSet to access each element through for-in, and it also has a size property to get the length of the entire HashSet.

However, note that when using for-in to access each element of HashSet, **it does not guarantee order**, and you cannot directly access HashSet elements through subscripts.

1. Subscript access - incorrect example:

   ```typescript
   let c = HashSet<Int64>([0, 1, 2])
   
   println(c[2]) // Subscript access error
   ```

2. For-in traversal of each element:

   ```typescript
   let c = HashSet<Int64>([0, 1, 2])
   
   for (val in c) {
       println(val)
   }
   ```

3. Get HashSet length using size:

   ```typescript
   let c = HashSet<Int64>([0, 1, 2])
   
   println(c.size) 
   ```

## HashSet Addition

If you need to add a single element to a HashSet, use the put function. If you want to add multiple elements at once, you can use the putAll function.

```typescript
let mySet = HashSet<Int64>()
mySet.put(0) // mySet contains elements 0
mySet.put(0) // mySet contains elements 0
mySet.put(1) // mySet contains elements 0, 1
let li = [2, 3]
mySet.putAll(li) // mySet contains elements 0, 1, 2, 3
```

## HashSet Deletion

To delete elements from a HashSet, you can use the remove function, specifying the element to delete.

```typescript
let mySet = HashSet<Int64>([0, 1, 2, 3])
mySet.remove(1) // mySet contains elements 0, 2, 3
```

## HashSet Contains

You can use the contains method to determine whether a certain element exists in the HashSet.

```typescript
    let c = HashSet<Int64>([0, 1, 2])
    println(c.contains(1)) // true 
```