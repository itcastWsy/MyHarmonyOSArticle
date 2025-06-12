# 14-Array Type (2)

## Introduction

In addition to the reference type array Array, Cangjie also introduces the value type array VArray.

Array is a reference type.

VArray is a value type.

Since value types themselves incur additional performance overhead during transmission and assignment due to copying, it is recommended not to use `VArray` with large lengths in performance-sensitive scenarios.

## Value Types and Reference Types

Regardless of the type of variable, it is always associated with a value. When actually using these variables, for some variables, we directly use the value itself. Such variables are called value type variables. For other variables, we treat the value as an **address**, and then use the data pointed to by this **address**. This type of variable is called a reference type variable. Value type variables each have their own copy of data, and they are independent of each other.

For example:

Each of us is assigned an identical dormitory. Although everyone's dormitory decoration is the same, they are independent of each other, each occupying their own space without affecting each other. This kind of room is actually a value type.

```
    var a = 100 // memory space 1
    var b = 100 // memory space 2
    a = 300 //  modified a but b is not affected
```

Another example:

Four of us are assigned to the same dormitory, and each of us has a key. The dormitory opened with the key is the same dormitory. If student A moves in a new TV, then when other students open the dormitory with their keys, they will also see this TV. This will result in changes to the data of the reference type, and other elements that still maintain this address will also experience changes when accessing this data.

```
    let a = [1, 2, 3, 4]
    let b = a
    b[0] = 100 // a[0] also becomes 100
```







## Basic Usage of VArray

When using VArray, the type declaration cannot be omitted, otherwise it will become a regular Array.

```
    let a: VArray<Int64, $3> = [1, 2, 3] // Int64 indicates the type, $3 indicates the length, $ cannot be omitted
```

## Specifying Length and Content for VArray

```
let b = VArray<Int64, $5>({i => i}) // [0, 1, 2, 3, 4]
```

Other usages are similar to Array