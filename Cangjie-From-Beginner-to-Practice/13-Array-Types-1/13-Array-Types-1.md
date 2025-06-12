# 13-Array Type (1)

## Introduction

Array type is represented using Array

We can use the Array type to construct ordered sequences of data with a single element type. For example, we can define string arrays, number arrays, boolean arrays.

Using arrays makes it more convenient for us to manage a group of similar data.

**Array is a Collection type with fixed length. Array does not provide member functions for adding and removing elements.**

## Array Creation Method One: Literal Method

This method has the most concise syntax and is commonly used.

```
    //  Method one: Literal method
    let arr1 = [1, 2, 3, 4] // Number array
    let arr2 = ['a', 'b', 'c'] // String array
    // let arr3=['a',100]// Cannot store different types of data
```



## Array Creation Method Two: Specifying Type

When creating an array, we can also actively specify the type.

```
    // Method two: Specifying type
    let arr3: Array<Int64> = [1, 2, 3]
```



## Array Creation Method Three: Specifying Length and Content

This method is more suitable for creating arrays with specified length and content.

```
    // Method three: Specifying length
    let arr4 = Array<Int64>() // Empty array
    let arr5 = Array<Int64>(5, {i => 100}) // Creates 5 elements with value 100, i represents each position in the array starting from 0, not used here, just to prevent syntax errors
    let arr6 = Array<Int64>(5, {i => i + 1}) // 5 elements, which are 1, 2, 3, 4, 5 respectively
```

## Accessing Arrays

We can access the length of an array and each element in the array.

### Accessing Length

size is a property that every array has, representing the number of elements in the array.

```
    let arr7 = [1, 2, 3, 4]
    print(arr7.size) // 4 
```

### Accessing Individual Elements

We access elements through their position in the array. It's important to note that the starting index is 0.

```
    // Accessing elements
    let arr7 = [1, 2, 3, 4]
    print(arr7[0]) // 1 
    print(arr7[3]) // 4 
    print(arr7[20]) // Out of range, will report an error: "array index 20 is past the end of array (which contains 4 elements)"
```

### Accessing Elements Within a Range

We can use specific syntax to quickly access elements within a range of the original array.

```
    // Getting a range of elements within the array
    let arr8 = ['a', 'b', 'c', 'd', 'e']
    // let arr9 = arr8[0..3] // a b c  does not include the element at index 3
    let arr10 = arr8[..3] // a b c 
    let arr11 = arr8[3..] // d e  
```

## Modifying Arrays

Here, modification refers to changing the elements of the array.

```
    // Modifying elements
    let arr8 = [1, 2, 3, 4]
    arr8[1] = 100
    print(arr8[1]) // 100   
```