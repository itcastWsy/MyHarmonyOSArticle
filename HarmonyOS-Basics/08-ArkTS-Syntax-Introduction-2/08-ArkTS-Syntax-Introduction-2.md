# 08-ArkTS Syntax Introduction (2)

# Functions

## Function Declaration

Function declaration introduces a function, including its name, parameter list, return type, and function body.

The following example is a simple function with two string type parameters and a return type of string:

```typescript
function add(x: string, y: string): string {
  let z: string = `${x} ${y}`;
  return z;
}
```

In function declarations, each parameter must be marked with a type. If a parameter is an optional parameter, then it is allowed to omit that parameter when calling the function. The last parameter of a function can be a rest parameter.

## Optional Parameters

Optional parameters can be formatted as name?: Type.

```typescript
function hello(name?: string) {
  if (name == undefined) {
    console.log("Hello!");
  } else {
    console.log(`Hello, ${name}!`);
  }
}
```

Another form of optional parameters is setting parameter default values. If this parameter is omitted in the function call, the default value of this parameter will be used as the actual parameter.

```typescript
function multiply(n: number, coeff: number = 2): number {
  return n * coeff;
}
multiply(2); // Returns 2*2
multiply(2, 3); // Returns 2*3
```

## Rest Parameters

The last parameter of a function can be a rest parameter. When using rest parameters, functions or methods are allowed to accept any number of actual parameters.

```typescript
function sum(...numbers: number[]): number {
  let res = 0;
  for (let n of numbers) res += n;
  return res;
}
sum(); // Returns 0
sum(1, 2, 3); // Returns 6
```

## Return Type

If the function return type can be inferred from the function body, the return type annotation can be omitted in the function declaration.

```typescript
// Explicitly specify return type
function foo(): string {
  return "foo";
}

// Inferred return type is string
function goo() {
  return "goo";
}
```

Functions that do not need to return a value can have their return type explicitly specified as void or the annotation can be omitted. Such functions do not need return statements.

Both function declaration methods in the following example are valid:

```typescript
function hi1() {
  console.log("hi");
}
function hi2(): void {
  console.log("hi");
}
```

## Function Scope

Variables and other instances defined in functions can only be accessed within the function and cannot be accessed from outside.

If a variable defined in a function has the same name as an existing instance in the outer scope, the local variable definition within the function will override the outer definition.

## Function Calls

Call a function to execute its function body, and actual parameter values will be assigned to the function's formal parameters.

If the function is defined as follows:

```typescript
function join(x: string, y: string): string {
  let z: string = `${x} ${y}`;
  return z;
}
```

Then the call to this function needs to include two string type parameters:

```typescript
let x = join("hello", "world");
console.log(x);
```

## Function Types

Function types are commonly used to define callbacks:

```typescript
type trigFunc = (x: number) => number; // This is a function type
function do_action(f: trigFunc) {
  f(3.141592653589); // Call function
}

do_action(Math.sin); // Pass function as parameter
```

## Arrow Functions (Also Known as Lambda Functions)

Functions can be defined as arrow functions, for example:

```typescript
let sum = (x: number, y: number): number => {
  return x + y;
};
```

The return type of arrow functions can be omitted; when omitted, the return type is inferred through the function body.

Expressions can be specified as arrow functions to make expressions shorter, so the following two expressions are equivalent:

```typescript
let sum1 = (x: number, y: number) => {
  return x + y;
};
let sum2 = (x: number, y: number) => x + y;
```

## Closures

A closure is a combination of a function and the environment in which the function is declared. This environment contains any local variables that were in scope at the time the closure was created.

In the following example, function f returns a closure that captures the count variable. Each time z is called, the value of count is preserved and incremented.

```typescript
function f(): () => number {
  let count = 0;
  let g = (): number => {
    count++;
    return count;
  };
  return g;
}
let z = f();
z(); // Returns: 1
z(); // Returns: 2
```

## Function Overloading

We can specify different ways to call a function by writing overloads. The specific method is to write multiple function headers with the same name but different signatures for the same function, followed by the function implementation.

```typescript
function foo(x: number): void;
/* First function definition */
function foo(x: string): void;
/* Second function definition */
function foo(x: number | string): void {
  /* Function implementation */
}
foo(123); //  OK, uses first definition
foo("aa"); // OK, uses second definition
```

Overloaded functions are not allowed to have the same name and parameter list, otherwise compilation errors will occur.
