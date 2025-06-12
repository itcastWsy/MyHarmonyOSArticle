# 53-HashMap

HashMap is mainly used to store key-value pair data. It is a hash table that provides fast access to the elements it contains. Each element in the table is identified using its key, and you can use the key to access the corresponding value.

## HashMap Initialization

Before using HashMap, you need to import the package first:

```typescript
import std.collection.*
```

Cangjie uses `HashMap<K, V>` to represent the HashMap type, where K represents the key type of the HashMap. K must be a type that implements the Hashable and `Equatable<K>` interfaces, such as numeric types or String. V represents the value type of the HashMap, and V can be any type.

```typescript
let a = HashMap<String, Int64>() // Created an empty HashMap whose key type is String and value type is Int64
let b = HashMap<String, Int64>([("a", 0), ("b", 1), ("c", 2)]) // whose key type is String and value type is Int64, containing elements ("a", 0), ("b", 1), ("c", 2)
let c = HashMap<String, Int64>(b) // Use another Collection to initialize a HashMap
let d = HashMap<String, Int64>(10) // Created a HashMap whose key type is String and value type is Int64 and capacity is 10
let e = HashMap<Int64, Int64>(10, {x: Int64 => (x, x * x)}) // Created a HashMap whose key and value type is Int64 and size is 10. All elements are initialized by specified rule function
```

## HashMap Access

HashMap can access individual elements through subscript syntax, traverse all elements through for-in, and use size to get the length of the HashMap.

1. Access individual elements using subscript syntax:

   ```typescript
   let b = HashMap<String, Int64>([("a", 0), ("b", 1), ("c", 2)]) 
   println(b['a']) // 0 
   ```

2. Access all elements using for-in syntax:

   ```typescript
   let map = HashMap<String, Int64>([("a", 0), ("b", 1), ("c", 2)])
   for ((k, v) in map) {
       println("The key is ${k}, the value is ${v}")
   }
   ```

3. Get length using size:

   ```typescript
   let b = HashMap<String, Int64>([("a", 0), ("b", 1), ("c", 2)]) 
   println(b.size) 
   ```

## HashMap Addition

If you need to add a single key-value pair to a HashMap, use the put function. If you want to add multiple key-value pairs at once, you can use the putAll function.

```typescript
let map = HashMap<String, Int64>()
map.put("a", 0) // map contains the element ("a", 0)
map.put("b", 1) // map contains the elements ("a", 0), ("b", 1)
let map2 = HashMap<String, Int64>([("c", 2), ("d", 3)])
map.putAll(map2) // map contains the elements ("a", 0), ("b", 1), ("c", 2), ("d", 3)
```

## HashMap Modification

Use subscript syntax for modification:

```typescript
let map = HashMap<String, Int64>([("a", 0), ("b", 1), ("c", 2)])
map["a"] = 3
```

## HashMap Deletion

Use remove syntax for deletion, specifying the key to delete:

```typescript
let map = HashMap<String, Int64>([("a", 0), ("b", 1), ("c", 2), ("d", 3)])
map.remove("d") // map contains the elements ("a", 0), ("b", 1), ("c", 2)
```

## HashMap Contains

When you want to determine whether a certain key is contained in the HashMap, you can use the contains function. It returns true if the key exists, otherwise it returns false.

```typescript
let map = HashMap<String, Int64>([("a", 0), ("b", 1), ("c", 2)])
let a = map.contains("a") // a == true
let b = map.contains("d") // b == false
```