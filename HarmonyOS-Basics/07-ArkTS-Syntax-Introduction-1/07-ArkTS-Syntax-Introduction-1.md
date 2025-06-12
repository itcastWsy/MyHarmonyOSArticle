# 07-ArkTS Syntax Introduction (1)

# ArkTS Introduction

ArkTS is the preferred main application development language for HarmonyOS. ArkTS extends upon the [TypeScript](https://www.typescriptlang.org/) (TS for short) ecosystem foundation for application development, maintaining the basic style of TS while strengthening development-time static checking and analysis through specification definitions to improve program execution stability and performance.

Starting from API version 10, ArkTS further strengthens static checking and analysis through specifications. For differences compared to standard TS, please refer to [Adaptation Rules from TypeScript to ArkTS](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/typescript-to-arkts-migration-guide-V5):

- Mandatory use of static types: Static typing is one of the most important features of ArkTS. If static types are used, the types of variables in the program are determined. At the same time, since all types are known before the program actually runs, the compiler can verify the correctness of the code, thereby reducing runtime type checking and helping improve performance.
- Prohibition of changing object layout at runtime: To achieve maximum performance, ArkTS requires that object layout cannot be changed during program execution.
- Restriction of operator semantics: To achieve better performance and encourage developers to write clearer code, ArkTS restricts the semantics of some operators. For example, the unary plus operator can only act on numbers and cannot be used on variables of other types.
- No support for Structural typing: Supporting Structural typing requires extensive consideration and careful implementation in the language, compiler, and runtime. Currently, ArkTS does not support this feature. Based on actual scenario requirements and feedback, we will reconsider this in the future.

Currently, in the UI development framework, ArkTS mainly extends the following capabilities:

- [Basic Syntax](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-basic-syntax-overview-V5): ArkTS defines declarative UI description, custom components, and dynamic UI element extension capabilities, which together with system components in the ArkUI development framework and their related event methods, property methods, etc., constitute the main body of UI development.
- [State Management](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-state-management-overview-V5): ArkTS provides multi-dimensional state management mechanisms. In the UI development framework, data related to UI can be used within components, and can also be passed between different component levels, such as between parent and child components, grandparent and grandchild components, and can also be passed within the global scope of applications or across devices. In addition, from the perspective of data transmission forms, it can be divided into read-only one-way transmission and changeable two-way transmission. Developers can flexibly use these capabilities to achieve data and UI linkage.
- [Rendering Control](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-rendering-control-overview-V5): ArkTS provides rendering control capabilities. Conditional rendering can render UI content corresponding to different states based on different application states. Loop rendering can iteratively obtain data from data sources and create corresponding components in each iteration process. Data lazy loading iteratively obtains data from data sources on demand and creates corresponding components in each iteration process.

ArkTS is compatible with TS/JavaScript (JS for short) ecosystem, and developers can use TS/JS for development or reuse existing code. For detailed information on HarmonyOS system support for TS/JS, please refer to [Constraints for TS/JS Compatibility](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-migration-background-V5#方舟运行时兼容tsjs).

In the future, ArkTS will continue to evolve based on application development/runtime requirements, gradually providing more features such as parallel and concurrent capability enhancement, system type enhancement, distributed development paradigms, and more.

For more detailed understanding of ArkTS language, please see [ArkTS Specific Guide](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-overview-V5).

# Advantages

ArkTS is a programming language designed for building high-performance applications. ArkTS is optimized based on inheriting TypeScript syntax to provide higher performance and development efficiency.

As mobile devices become more prevalent in people's daily lives, many programming languages were not designed with mobile devices in mind initially, resulting in slow, inefficient, and power-hungry applications. The need for programming language optimization for mobile environments is also growing. ArkTS is specifically designed to solve these problems, focusing on improving runtime efficiency.

The currently popular programming language TypeScript is extended from JavaScript by adding type definitions, while ArkTS is a further extension of TypeScript. TypeScript is loved by developers because it provides a more structured way of coding JavaScript. ArkTS aims to maintain most of TypeScript's syntax, enabling seamless transition for existing TypeScript developers and allowing mobile developers to quickly get started with ArkTS.

A major characteristic of ArkTS is its focus on **low runtime overhead**. ArkTS imposes **stricter restrictions on TypeScript's dynamic typing features** to reduce runtime overhead and improve execution efficiency. **By eliminating dynamic typing features**, ArkTS code can be more effectively compiled and optimized before runtime, thus achieving faster application startup and lower power consumption.

# Declarations

ArkTS introduces variables, constants, functions, and types through declarations.

## **Variable Declaration**

Declarations beginning with the keyword let introduce variables that can have different values during program execution.

```typescript
let hi: string = "hello";
hi = "hello, world";
```

## **Constant Declaration**

Declarations beginning with the keyword const introduce read-only constants that can only be assigned once.

```typescript
const hello: string = "hello";
```

Reassigning a constant will cause a compile-time error.

## **Automatic Type Inference**

Since ArkTS is a statically typed language, all data types must be determined at compile time.

However, if a variable or constant declaration includes an initial value, developers do not need to explicitly specify its type. The ArkTS specification lists all scenarios where automatic type inference is allowed.

In the following example, both declaration statements are valid, and both variables are of string type:

```typescript
let hi1: string = "hello";
let hi2 = "hello, world";
```

# Types

## **Number Type**

ArkTS provides number and Number types, and any integer or floating-point number can be assigned to variables of this type.

Number literals include integer literals and decimal floating-point literals.

Integer literals include the following categories:

- Decimal integers consisting of digit sequences. For example: 0, 117, -345
- Hexadecimal integers beginning with 0x (or 0X), which can contain digits (0-9) and letters a-f or A-F. For example: 0x1123, 0x00111, -0xF1A7
- Octal integers beginning with 0o (or 0O), which can only contain digits (0-7). For example: 0o777
- Binary integers beginning with 0b (or 0B), which can only contain digits 0 and 1. For example: 0b11, 0b0011, -0b11

Floating-point literals include the following:

- Decimal integers, which can be signed numbers (i.e., prefixed with "+" or "-");
- Decimal point (".")
- Fractional part (represented by a string of decimal digits)
- Exponent part beginning with "e" or "E", followed by a signed (i.e., prefixed with "+" or "-") or unsigned integer.

Example:

```typescript
let n1 = 3.14;
let n2 = 3.141592;
let n3 = 0.5;
let n4 = 1e2;

function factorial(n: number): number {
  if (n <= 1) {
    return 1;
  }
  return n * factorial(n - 1);
}

factorial(n1); //  7.660344000000002
factorial(n2); //  7.680640444893748
factorial(n3); //  1
factorial(n4); //  9.33262154439441e+157
```

## **Boolean Type**

The boolean type consists of two logical values: true and false.

Boolean type variables are commonly used in conditional statements:

```typescript
let isDone: boolean = false;
// ...
if (isDone) {
  console.log("Done!");
}
```

## **String Type**

string represents a sequence of characters; escape characters can be used to represent characters.

String literals consist of zero or more characters enclosed in single quotes (') or double quotes ("). String literals also have a special form, which is template literals enclosed in backticks (`).

```typescript
let s1 = "Hello, world!\n";
let s2 = "this is a string";
let a = "Success";
let s3 = `The result is ${a}`;
```

## **Void Type**

The void type is used to specify that a function has no return value.

This type has only one value, which is also void. Since void is a reference type, it can be used for generic type parameters.

```typescript
class Class<T> {
  //...
}
let instance: Class<void>;
```

## **Object Type**

The Object type is the base type for all reference types. Any value, including values of basic types (which are automatically boxed), can be directly assigned to variables of Object type. The object type is used to represent types other than basic types.

## **Array Type**

Array is an object composed of data that can be assigned to the element type specified in the array declaration.

Arrays can be assigned by array composite literals (i.e., a list of zero or more expressions enclosed in square brackets, where each expression is an element in the array). The length of an array is determined by the number of elements in the array. The index of the first element in an array is 0.

The following example will create an array containing three elements:

```typescript
let names: string[] = ["Alice", "Bob", "Carol"];
```

## **Enum Type**

The enum type, also known as enumeration type, is a value type of a predefined set of named values, where the named values are called enumeration constants.

When using enumeration constants, they must be prefixed with the enumeration type name.

```typescript
enum ColorSet {
  Red,
  Green,
  Blue,
}
let c: ColorSet = ColorSet.Red;
```

Constant expressions can be used to explicitly set the values of enumeration constants.

```typescript
enum ColorSet {
  White = 0xff,
  Grey = 0x7f,
  Black = 0x00,
}
let c: ColorSet = ColorSet.Black;
```

**Union Type**

Union type is a reference type composed of multiple types. Union types contain all possible types that a variable might have.

```typescript
class Cat {
  name: string = "cat";
  // ...
}
class Dog {
  name: string = "dog";
  // ...
}
class Frog {
  name: string = "frog";
  // ...
}
type Animal = Cat | Dog | Frog | number;
// Cat, Dog, Frog are some types (classes or interfaces)

let animal: Animal = new Cat();
animal = new Frog();
animal = 42;
// Variables of union type can be assigned any valid value of the constituent types
```

Different mechanisms can be used to get values of specific types in union types.

Example:

```typescript
class Cat {
  sleep() {}
  meow() {}
}
class Dog {
  sleep() {}
  bark() {}
}
class Frog {
  sleep() {}
  leap() {}
}

type Animal = Cat | Dog | Frog;

function foo(animal: Animal) {
  if (animal instanceof Frog) {
    animal.leap(); // animal is of Frog type here
  }
  animal.sleep(); // Animal has sleep method
}
```

## **Aliases Type**

Aliases type provides names for anonymous types (arrays, functions, object literals, or union types) or provides alternative names for existing types.

```typescript
type Matrix = number[][];
type Handler = (s: string, no: number) => string;
type Predicate<T> = (x: T) => boolean;
type NullableObject = Object | null;
```

# Operators

## **Assignment Operators**

Assignment operator =, used as x=y.

Compound assignment operators combine assignment with operators, where x op = y equals x = x op y.

Compound assignment operators are listed as follows: +=, -=, \*=, /=, %=, <<=, >>=, >>>=, &=, |=, ^=.

## **Comparison Operators**

| Operator | Description                                                                                             |
| :------- | :------------------------------------------------------------------------------------------------------ |
| ===      | Returns true if two operands are strictly equal (operands of different types are considered unequal).   |
| !==      | Returns true if two operands are strictly unequal (operands of different types are considered unequal). |
| ==       | Returns true if two operands are equal.                                                                 |
| !=       | Returns true if two operands are unequal.                                                               |
| >        | Returns true if the left operand is greater than the right operand.                                     |
| >=       | Returns true if the left operand is greater than or equal to the right operand.                         |
| <        | Returns true if the left operand is less than the right operand.                                        |
| <=       | Returns true if the left operand is less than or equal to the right operand.                            |

## **Arithmetic Operators**

Unary operators are -, +, --, ++.

Binary operators are listed as follows:

| Operator | Description    |
| :------- | :------------- |
| +        | Addition       |
| -        | Subtraction    |
| \*       | Multiplication |
| /        | Division       |
| %        | Remainder      |

## **Bitwise Operators**

| Operator | Description                                                                                                         |
| :------- | :------------------------------------------------------------------------------------------------------------------ |
| a & b    | Bitwise AND: Sets this bit to 1 if both corresponding bits of the operands are 1, otherwise sets to 0.              |
| a \| b   | Bitwise OR: Sets this bit to 1 if at least one of the corresponding bits of the operands is 1, otherwise sets to 0. |
| a ^ b    | Bitwise XOR: Sets this bit to 1 if the corresponding bits of the operands are different, otherwise sets to 0.       |
| ~ a      | Bitwise NOT: Inverts the bits of the operand.                                                                       |
| a << b   | Left shift: Shifts the binary representation of a left by b bits.                                                   |
| a >> b   | Arithmetic right shift: Shifts the binary representation of a right by b bits with sign extension.                  |
| a >>> b  | Logical right shift: Shifts the binary representation of a right by b bits, filling with 0s on the left.            |

## **Logical Operators**

| Operator | Description |
| :------- | :---------- |
| a && b   | Logical AND |
| a \|\| b | Logical OR  |
| ! a      | Logical NOT |

# Statements

## **If Statement**

The if statement is used in scenarios where different statements need to be executed based on logical conditions. When the logical condition is true, the corresponding set of statements is executed; otherwise, another set of statements is executed (if available).

The else part may also contain if statements.

The if statement is shown as follows:

```typescript
if (condition1) {
  // statement1
} else if (condition2) {
  // statement2
} else {
  // else statement
}
```

Conditional expressions can be of any type. However, for types other than boolean, implicit type conversion is performed:

```typescript
let s1 = "Hello";
if (s1) {
  console.log(s1); // Prints "Hello"
}

let s2 = "World";
if (s2.length != 0) {
  console.log(s2); // Prints "World"
}
```

## **Switch Statement**

Use switch statements to execute code blocks that match the value of the switch expression.

The switch statement is shown as follows:

```typescript
switch (expression) {
  case label1: // If label1 matches, execute
    // ...
    // statement1
    // ...
    break; // Optional
  case label2:
  case label3: // If label2 or label3 matches, execute
    // ...
    // statement23
    // ...
    break; // Optional
  default:
  // Default statement
}
```

If the value of the switch expression equals the value of a certain label, the corresponding statement is executed.

If no label value matches the expression value and the switch has a default clause, the program will execute the code block corresponding to the default clause.

The break statement (optional) allows jumping out of the switch statement and continuing to execute statements after the switch statement.

If there is no break statement, the code block corresponding to the next label in the switch is executed.

## **Conditional Expressions**

Conditional expressions determine which of the other two expressions to return based on the boolean value of the first expression.

Example:

```typescript
condition ? expression1 : expression2;
```

If condition is a truthy value (value that converts to true), expression1 is used as the result of this expression; otherwise, expression2 is used.

Example:

```typescript
let isValid = Math.random() > 0.5 ? true : false;
let message = isValid ? "Valid" : "Failed";
```

## **For Statement**

The for statement will be repeatedly executed until the loop exit statement value is false.

The for statement is shown as follows:

```typescript
for ([init]; [condition]; [update]) {
  statements;
}
```

The execution flow of the for statement is as follows:

1. Execute the init expression (if any). This expression usually initializes one or more loop counters.

2. Evaluate condition. If it is a truthy value (value that converts to true), execute the statements in the loop body. If it is a falsy value (value that converts to false), the for loop terminates.

3. Execute the statements in the loop body.

4. If there is an update expression, execute this expression.

5. Return to step 2.

Example:

```typescript
let sum = 0;
for (let i = 0; i < 10; i += 2) {
  sum += i;
}
```

## **For-of Statement**

Use for-of statements to iterate over arrays or strings. Example:

```typescript
for (forVar of expression) {
  statements;
}
```

Example:

```typescript
for (let ch of "a string object") {
  /* process ch */
}
```

## **While Statement**

The while statement will execute statements as long as condition is a truthy value (value that converts to true). Example:

```typescript
while (condition) {
  statements;
}
```

Example:

```typescript
let n = 0;
let x = 0;
while (n < 3) {
  n++;
  x += n;
}
```

## **Do-while Statement**

If the value of condition is a truthy value (value that converts to true), then the statements will be repeatedly executed. Example:

```typescript
do {
  statements;
} while (condition);
```

Example:

```typescript
let i = 0;
do {
  i += 1;
} while (i < 10);
```

## **Break Statement**

Use break statements to terminate loop statements or switch.

Example:

```typescript
let x = 0;
while (true) {
  x++;
  if (x > 5) {
    break;
  }
}
```

If the break statement is followed by an identifier, control flow is transferred outside the statement block contained by that identifier.

Example:

```typescript
let x = 1;
label: while (true) {
  switch (x) {
    case 1:
      // statements
      break label; // Break the while statement
  }
}
```

## **Continue Statement**

The continue statement stops the execution of the current loop iteration and passes control to the next iteration.

Example:

```typescript
let sum = 0;
for (let x = 0; x < 100; x++) {
  if (x % 2 == 0) {
    continue;
  }
  sum += x;
}
```

## **Throw and Try Statements**

The throw statement is used to throw exceptions or errors:

```typescript
throw new Error("this error");
```

The try statement is used to catch and handle exceptions or errors:

```typescript
try {
  // Statement block where exceptions may occur
} catch (e) {
  // Exception handling
}
```

In the following example, throw and try statements are used to handle division by zero errors:

```typescript
class ZeroDivisor extends Error {}

function divide(a: number, b: number): number {
  if (b == 0) throw new ZeroDivisor();
  return a / b;
}

function process(a: number, b: number) {
  try {
    let res = divide(a, b);
    console.log("result: " + res);
  } catch (x) {
    console.log("some error");
  }
}
```

Finally statements are supported:

````typescript
function processData(s: string) {
  let error: Error | null = null;

  try {
    console.log('Data processed: ' + s);
    // ...
    // Statements where exceptions may occur
    // ...
  } catch (e) {
    error = e as Error;
    // ...
    // Exception handling
    // ...
  } finally {
    if (error != null) {
      console.log(`Error caught: input='${s}', message='${error.message}'`);
    }
  }
}
```typescript
class Cat {
  name: string = "cat";
  // ...
}
class Dog {
  name: string = "dog";
  // ...
}
class Frog {
  name: string = "frog";
  // ...
}
type Animal = Cat | Dog | Frog | number;
// Cat, Dog, Frog are some types (classes or interfaces)

let animal: Animal = new Cat();
animal = new Frog();
animal = 42;
// Variables of union type can be assigned any valid value of the constituent types
````

Different mechanisms can be used to get values of specific types in union types.

Example:

```typescript
class Cat {
  sleep() {}
  meow() {}
}
class Dog {
  sleep() {}
  bark() {}
}
class Frog {
  sleep() {}
  leap() {}
}

type Animal = Cat | Dog | Frog;

function foo(animal: Animal) {
  if (animal instanceof Frog) {
    animal.leap(); // animal is of Frog type here
  }
  animal.sleep(); // Animal has sleep method
}
```

## **Aliases Type**

Aliases type provides names for anonymous types (arrays, functions, object literals, or union types) or provides alternative names for existing types.

```typescript
type Matrix = number[][];
type Handler = (s: string, no: number) => string;
type Predicate<T> = (x: T) => boolean;
type NullableObject = Object | null;
```
