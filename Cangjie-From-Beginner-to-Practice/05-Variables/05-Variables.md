# 05-Variables

## What are Variables

In the complex system of computer programming, variables can be precisely defined as specific memory space areas within a computer that are dedicated to storing data. When we begin program design, whether developing an elegant alarm clock application or building a convenient and efficient shopping cart system, variables play an indispensable role. In an alarm clock program, to accurately record and display time information, corresponding variables are needed to store time data. This data is continuously updated as time progresses, ensuring that the alarm clock can accurately perform its timed reminder function. Similarly, in a shopping cart program, variables are used to store various shopping data, including product names, quantities, prices, and other detailed information. The dynamic changes of this data reflect users' shopping behavior and the real-time status of the shopping cart, serving as the foundational support for implementing shopping cart functionality.

## Characteristics of Variables

In the Cangjie language, variables have the following characteristics:

1. Variable names need to be valid identifiers
2. Variables also have types, such as number, text, and other types
3. Variables need to be initialized/assigned when used. That is, programs cannot access idle memory blocks
4. Variables are divided into mutable and immutable

## Variable Names Need to be Valid Identifiers

**Correct Example**

```
// Create a variable, text type => string type
let userName = "Xiao Wan" // Variable name is userName, stored content is "Xiao Wan"
```

**Incorrect Example**

```
let 123userName = "Xiao Wan" // Cannot start with a number
```

## Variables Have Types, Such as Number, Text, and Other Types

**Correct Example**

```
// Create a variable, number type
let num = 100 // Variable name is num, stored content is number 100
```

## Variables Need to be Initialized/Assigned When Used

That is, programs cannot access idle memory blocks

**Correct Example**

```
// Create first, then use
let age = 300 // Create variable
print(age) // Use variable
```

**Incorrect Example**

```
print(address) // address was never declared
```

## Variables are Divided into Mutable and Immutable

1. Immutable variables need to use the keyword **let**
2. Mutable variables need to use the keyword **var**

**Correct Example**

```
var age = 20
age = 30 // Mutable
```

**Incorrect Example**

```
let age = 20
age = 30 // Immutable
```
